# Učenje Frappe Framework

[Sadržaj][00]

## 14 Eksperimenti - II

### Prvi pravi eksperiment

Zameni `pass` ovim:

```python
from frappe.model.document import Document

class Book(Document):

    def before_insert(self):
        print(">>> before_insert")

    def before_validate(self):
        print(">>> before_validate")

    def validate(self):
        print(">>> validate")

    def before_save(self):
        print(">>> before_save")

    def after_insert(self):
        print(">>> after_insert")

    def on_update(self):
        print(">>> on_update")
```

Nemoj još dodavati više metoda.

**Zašto baš ovih šest?**

Zato što smo ih sreli dok smo čitali `Document.insert()`. Sada ćemo proveriti da li se naše razumevanje poklapa sa stvarnim izvršavanjem.

**Šta dalje**:

1. Sačuvaj izmene u:

   ```text
   book.py
   ```

2. Pokreni:

   ```bash
   bench restart
   ```

   Ako koristiš `bench start`, dovoljno je da ga prekineš (`Ctrl+C`) i ponovo pokreneš:

   ```bash
   bench start
   ```

3. U browseru otvori **Book** i napravi jedan novi zapis.

   Na primer:

   * Title: `Frappe Internals`
   * Price: `100`

4. Posmatraj terminal u kome radi `bench start`.

Hoću da mi prepišeš izlaz iz terminala, pa ćemo ga uporediti sa onim što smo ranije pročitali u `Document.insert()`.

To će biti naš prvi pravi eksperiment iz Faze 6.

Ili hajde jedan mali test:

U `bench --site site1.local console`:

```python
doc = frappe.new_doc("Book")
doc.title = "Test"
doc.price = 100
doc.insert()
```

Ako je `book.py` učitan, trebalo bi odmah da vidiš naše poruke.

Ako ih nema, onda ćemo istražiti zašto.

Dakle, sledeći korak je samo ovaj mali eksperiment u `bench console`. Ako tu prorade `before_insert`, `validate` itd., znaćemo da je sve u redu i možemo da nastavimo sa proučavanjem životnog ciklusa `Document`.

> [!Info] Izlaz:
>
> ```py
> In [1]: doc = frappe.new_doc("Book")
>    ...: doc.title = "Test"
>    ...: doc.price = 100
>    ...: doc.insert()
> >>> before_insert
> >>> before_validate
> >>> validate
> >>> before_save
> >>> after_insert
> >>> on_update
> Out[1]: <Book: qs827pdgad>
> 
> Ovaj ulaz se ne vidi iz http-a?
> ```

Ovo nam govori dve važne stvari.

1. `book.py` je ispravno učitan. ✅
2. Redosled koji smo očekivali se zaista dešava. ✅

Sada na tvoje pitanje:

"Ovaj izlaz se ne vidi iz http-a?" Trebalo bi da se vidi. Ali samo kada se zaista izvrši `insert()`.

---

Otprilike sam otkrio šta se dešava.

Kada sam uneo četvrti zapis, "nije" prihvatio prvi klik na Save button, tek drugi, i pojavio se očekivani tekst u redosledu pozvanih događaja. Mislim da prvi klik samo da fokus formi!

---

Mislim da si veoma verovatno u pravu.

To nije ponašanje `Document` klase, već **Desk UI**.

Desk ponekad "pojede" prvi klik ako se u tom trenutku dešava nešto kao:

* završava se editovanje poslednjeg polja,
* gubi se fokus sa polja,
* izvršava se client-side validacija,
* ili se forma tek prebacuje u stanje spremno za snimanje.

Tek drugi klik zaista pošalje zahtev:

```txt
Browser
    ↓
POST /api/method/frappe.desk.form.save.savedocs
    ↓
frappe.get_doc(...)
    ↓
Document.insert()
    ↓
Book.before_insert()
...
```

To je upravo ono što smo želeli da vidimo.

**Ono što je meni zanimljivije**:

Pogledaj redosled:

```text
  before_insert
  before_validate
  validate
  before_save
  after_insert
  on_update
```

Sećaš se koliko smo analizirali `insert()` u `document.py`?

Sada više nije apstraktan kod.

Kada vidiš u izvornom kodu:

```python
self.run_method("before_insert")
...
self.run_before_save_methods()
...
self.db_insert()
...
self.run_method("after_insert")
...
self.run_post_save_methods()
```

ti sada znaš šta će izaći na ekranu.

To je bio glavni cilj cele prethodne faze.

Jedna zanimljiva stvar za domaći zadatak:

Da li primećuješ nešto "čudno"? Zašto se `on_update` poziva odmah posle `after_insert`?

Na prvi pogled čovek bi očekivao: "Pa nisam radio update, radio sam insert."
Ali ipak se `on_update` izvršava.

To nije greška. To je dizajn Frappe-a.

---

Verovatno prvo doda prazan zapis, a onda ga update-uje sa pravim vrednostima.
Na taj način zadržava približno isti scenario add / update!

---

Odlično razmišljanje. To bi bio sasvim razuman dizajn u nekim ORM-ovima, ali u Frappe-u nije tako. U stvari, zapis se ne insertuje pa update-uje.

Ostaje samo jedan SQL:

```sql
INSERT INTO tabBook (...)
VALUES (...)
```

Nema naknadnog:

```sql
UPDATE tabBook ...
```

Zašto se onda zove `on_update`? Sećaš se da smo čitali nešto ovako (parafraziram):

```python
insert()
    ...
    db_insert()
    ...
    after_insert()
    ...
    run_post_save_methods()
```

A u `run_post_save_methods()` se nalazi poziv:

```python
self.run_method("on_update")
```

Dakle, Frappe kaže: "Dokument je sada uspešno sačuvan u bazi."

i tada poziva:

```python
on_update()
```

bez obzira da li je to bio:

* prvi `INSERT`, ili
* deseti `UPDATE`.

Zašto su to uradili? Po mom mišljenju (i mislim da je to bila namera autora), zato što ogromna većina poslovne logike ne zanima kako je dokument stigao u trenutno stanje.

Na primer:

```python
def on_update(self):
    self.update_customer_balance()
    self.recalculate_totals()
    self.send_notifications()
```

Ta logika treba da se izvrši:

1. posle kreiranja dokumenta,
2. posle svake izmene.

Ne želiš da pišeš isto dva puta:

```python
after_insert()
    ...

on_update()
    ...
```

**A `after_insert`?**

On postoji baš za ono što se dešava samo jednom.

Na primer:

```python
def after_insert(self):
    create_default_tasks()
```

To ne želiš da radiš pri svakom snimanju.

Sada dolazimo do zanimljivog dela Faze 6.

Predlažem da sledeći eksperiment bude da dodamo još nekoliko metoda:

```python
before_naming
autoname
before_validate
validate
before_save
before_insert
after_insert
on_update
on_change
```

i da vidimo ceo tok izvršavanja.

Posle toga ćemo dodati `doc_events` hook i posmatrati gde se on tačno "umeće" u isti tok.

To će biti prvi put da vidiš kako se prepliću:

* metoda u `Book` klasi,
* `Document` klasa,
* i `hooks.py`.

Mislim da će to biti jedan od najkorisnijih eksperimenata koje ćemo uraditi.

Mislim da smo sada na tački gde treba da radimo male eksperimente, a ne da čitamo kod.

### Kompletan lifecycle

Proširi `Book` ovako:

```python
from frappe.model.document import Document


class Book(Document):

    def before_naming(self):
        print(">>> before_naming")

    def autoname(self):
        print(">>> autoname")

    def before_insert(self):
        print(">>> before_insert")

    def before_validate(self):
        print(">>> before_validate")

    def validate(self):
        print(">>> validate")

    def before_save(self):
        print(">>> before_save")

    def after_insert(self):
        print(">>> after_insert")

    def on_update(self):
        print(">>> on_update")

    def on_change(self):
        print(">>> on_change")
```

Zatim:

Napravi **novi** Book dokument (ne menjaj postojeći).

Hoću da **sam** pokušaš da predvidiš redosled izvršavanja.

Možeš čak i da ga zapišeš na papir.

Posebno obrati pažnju na:

* da li `before_naming` dolazi pre ili posle `validate`,
* kada se poziva `autoname`,
* da li se `on_change` poziva pri `insert()` ili samo pri `save()` postojećeg dokumenta.

To su pitanja na koja nismo eksplicitno odgovorili čitajući kod.

Posle toga, uradi još jedan eksperiment:

1. Otvori **isti** Book.
2. Promeni samo `Price`.
3. Klikni **Save**.

Uporedi redosled događaja sa prvim snimanjem.

Ako sve bude kako očekujem, dobićemo dva veoma lepa dijagrama:

* **Insert** lifecycle
* **Update** lifecycle

I tek tada ćemo uvesti `hooks.py`, jer ćeš odmah videti gde se hook "ubacuje" u već poznati tok. To je mnogo lakše nego učiti hook-ove napamet.

> [Info] Izlaz
>
> Evo izlaza za INSERT
>
> ```sh
> 22:28:11 web.1         | >>> before_insert
> 22:28:11 web.1         | >>> before_naming
> 22:28:11 web.1         | >>> autoname
> 22:28:11 web.1         | >>> before_validate
> 22:28:11 web.1         | >>> validate
> 22:28:11 web.1         | >>> before_save
> 22:28:11 web.1         | >>> after_insert
> 22:28:11 web.1         | >>> on_update
> 22:28:11 web.1         | >>> on_change
> ```
>
> `before_naming` ide pre `autoname`, obadva pre `before_validate`.
>
> Evo izlaza za UPDATE
>
> ```sh
> 22:31:24 web.1         | >>> before_validate
> 22:31:24 web.1         | >>> validate
> 22:31:24 web.1         | >>> before_save
> 22:31:24 web.1         | >>> on_update
> 22:31:24 web.1         | >>> on_change
> ```
>
> U ovom drugom slučaju nema poziva povezanih sa: `before_insert`,
> `before_naming`, `autoname`, `after_insert`.

Ono što si dokazao:

**INSERT**:

```text
before_insert
before_naming
autoname
before_validate
validate
before_save
------------------
INSERT u bazu
------------------
after_insert
on_update
on_change
```

**UPDATE**:

```text
before_validate
validate
before_save
------------------
UPDATE u bazu
------------------
on_update
on_change
```

To je praktično ceo životni ciklus koji jedan Frappe programer koristi svakodnevno.

</br>

`on_update` se izvršava na serveru.

Na primer:

```python
def on_update(self):
    self.recalculate_stock()
    self.create_gl_entries()
    self.update_customer_balance()
```

To je čista poslovna logika.

A `on_change`?

E tu dolazimo do zanimljivog pitanja.

Po čemu se razlikuju:

```python
on_update()
```

i

```python
on_change()
```

Kada si pogledao eksperiment, vidiš da se oba pozivaju.

Zašto onda postoje oba?

To je jedno od najčešćih pitanja kod Frappe početnika.

### Detalji lifecycle-a

Dodaj u `hooks.py`:

```python
doc_events = {
    "Book": {
        "*": "frappe_lab.book_hooks.trace"
    }
}
```

Zatim napravi novi fajl, na primer:

```text
frappe_lab/book_hooks.py
```

sa sadržajem:

```python
def trace(doc, method):
    print(f"HOOK ---> {method}")
```

I onda ponovi oba eksperimenta:

* INSERT
* UPDATE

Zašto baš ovo? Zato što ćemo prvi put videti kada se tačno poziva hook u odnosu na metode klase `Book`.

To je nešto što se ne vidi lako ni čitanjem koda, ali se fenomenalno vidi eksperimentom.

Mislim da će nam to biti najbolji uvod u `hooks.py`, koji je sledeća velika tema. I usput ćemo dobiti odgovor na pitanje koje mnogi postavljaju:

"Kada koristiti metodu u `Document` klasi, a kada `doc_events` hook?"

To je, po mom mišljenju, jedna od najvažnijih praktičnih lekcija u razvoju Frappe aplikacija.

Po tvojoj strukturi aplikacije to je: `hooks.py`** već postoji:

```text
/home/radosav/frappe-bench/apps/frappe_lab/frappe_lab/hooks.py
```

U njega dodaj:

```python
doc_events = {
    "Book": {
        "on_update": "frappe_lab.book_hooks.trace"
    }
}
```

Napravi novi fajl `book_hooks.py`:

```text
/home/radosav/frappe-bench/apps/frappe_lab/frappe_lab/book_hooks.py
```

sa sadržajem:

```python
def trace(doc, method):
    print(f"HOOK ---> {method}")
```

**Zašto je putanja baš takva?**

Seti se da Python import izgleda ovako:

```python
frappe_lab.book_hooks.trace
```

To znači:

```text
frappe_lab                 ← Python paket
    book_hooks.py          ← modul
        trace()            ← funkcija
```

što odgovara putanji:

```text
/home/radosav/frappe-bench/apps/frappe_lab/frappe_lab/book_hooks.py
```

Posle toga ćemo napraviti jedan INSERT i jedan UPDATE i dobiti kompletan dijagram izvršavanja:

* `Document`
* `Book`
* `doc_events`

To će biti prvi put da se vide sva tri sloja zajedno.

> [!Info] Izlaz
>
>
> ```sh
> 23:54:14 web.1         | before_validate
> 23:54:14 web.1         | >>> validate
> 23:54:14 web.1         | >>> before_save
> 23:54:14 web.1         | >>> on_update
> 23:54:14 web.1         | HOOK ---> on_update
> 23:54:14 web.1         | >>> on_change
> 23:54:15 web.1         | 192.168.122.1 - - [18/Jul/2026 23:54:15] "POST /api/method/frappe.desk.form.save.savedocs HTTP/1.1" 200 -
> ```

E, sada smo dobili ono što smo hteli.

I pogledaj kako je lep rezultat:

```text
   before_validate
   validate
   before_save
         ↓
   on_update
         ↓
   HOOK ---> on_update
         ↓
   on_change
```

Sada možemo da izvučemo nekoliko važnih zaključaka.

**`doc_events` nije zamena za metodu klase**

Imaš:

```python
class Book(Document):

    def on_update(self):
        print(">>> on_update")
```

i u `hooks.py`:

```python
doc_events = {
    "Book": {
        "on_update": "frappe_lab.book_hooks.trace"
    }
}
```

To su dve različite tačke proširenja.

Prvo se izvršava metoda dokumenta:

```text
on_update
```

a zatim spoljašnji hook:

```text
HOOK ---> on_update
```

**Zašto ovo postoji?**

Zamisli da imaš standardni Frappe/ERPNext DocType:

```text
Sales Invoice
```

Ne želiš da menjaš:

```python
erpnext/accounts/doctype/sales_invoice/sales_invoice.py
```

jer će ti sledeći update pregaziti izmene.

Umesto toga u svojoj aplikaciji kažeš:

```python
doc_events = {
    "Sales Invoice": {
        "on_update": "my_app.invoice_hooks.after_update"
    }
}
```

i dodaš svoju logiku.

To je jedan od glavnih razloga postojanja hook sistema.

**Još jedan lep detalj**:

Primetio si ranije: "on_update je za sve"

I sada vidiš zašto.

I pri:

```text
INSERT
```

dobio si:

```text
after_insert
on_update
```

A pri:

```text
UPDATE
```

dobio si:

```text
on_update
```

Dakle `on_update` u Frappe terminologiji zapravo više znači: "dokument je uspešno sačuvan i sada mogu da reagujem" a ne strogo SQL UPDATE.

Mislim da smo ovim završili prvi veliki deo Faze 6:

✅ napravili App  
✅ napravili DocType  
✅ napisali controller  
✅ pratili lifecycle  
✅ dodali `doc_events` hook  
✅ videli odnos controller ↔ hook  

### Moje mišljenje o Frappe ocenama

Ne bih se mnogo obazirao na broj zvezdica ili pojedinačne komentare.

Frappe je pomalo "čudna zver".

Ljudi koji očekuju:

* Django
* Flask
* FastAPI

često ga ne vole.

Ljudi koji očekuju:

* Odoo
* Oracle APEX
* Power Apps
* Mendix

često ga obožavaju.

Zašto?

Jer Frappe nije prvenstveno web framework. To je **application framework** sa ugrađenim RAD (Rapid Application Development) alatima.

Zbog toga ima:

* ORM
* permissions
* workflow
* report engine
* print engine
* scheduler
* background jobs
* REST
* realtime
* Desk
* Website
* Jinja
* migrations
* metadata
* role system...

...što je zaista retko u jednom open-source projektu.

Njegova mana nije što je siromašan, nego što je **veliki**.

### Hajde da ubrzamo

Predlažem ovakav plan.

#### `doc_events`

Već si praktično naučio 90%.

Zapamti samo ovo:

**Controller**:

Koristiš kada pišeš logiku **svog DocType-a**.

```python
class Book(Document):
    def validate(self):
        ...
```

**doc_events**:

Koristiš kada želiš da se "prikačiš" na:

* tuđi DocType
* ili svoj DocType bez menjanja controller-a.

  ```python
  doc_events = {
      "User": {
          "validate": "moja_app.user_hooks.validate"
      }
  }
  ```

To je cela filozofija.

Ne bih više trošio vreme na `doc_events`.

#### Desk

Po meni ovo je sledeća velika tema.

Tu bih prošao:

* Workspace
* Module
* DocType
* Report
* Page
* Dashboard
* Print Format
* Web Form

bez ulaženja u detalje.

Cilj: "Šta sve Desk može da napravi?"

#### Deploy

Ovo me posebno zanima da zajedno prođemo.

Jer postoji razlika između:

Development

```text
bench start
```

i

Production

```text
Supervisor
Nginx
Redis
SocketIO
Workers
```

Kako se aplikacija pakuje.  
Kako ide na Git.  
Kako ide na drugi server.  
Kako se instalira.  
Kako rade migracije.

To je veoma važna tema.

Ja bih ovde stao sa Frappe internals. Mislim da tada već znaš dovoljno.

Posle toga bih radio ono što rade pravi programeri: Napravimo malu aplikaciju.

Recimo:

* Library
* Inventory
* Help Desk
* TODO

i usput učimo:

* Query Builder
* Child Tables
* REST
* Background Jobs
* Print Formats
* Permissions

onoliko koliko nam zatreba.

**Mislim da je sada pravi trenutak da promenimo način učenja**:

Do sada smo učili: Kako Frappe radi?

Od sada bih učio: Kako se pravi aplikacija u Frappe-u?

To je, po mom mišljenju, mnogo zanimljiviji i korisniji deo. A kada naiđemo na nešto iz internala, vratićemo se na izvorni kod samo za tu konkretnu temu, umesto da ga čitamo redom. Mislim da ćeš tako mnogo brže steći osećaj za ceo framework.

**`docs_events`**:

**Ko piše `doc_events`?**

Programer aplikacije. Ne korisnik, ne administrator, ne Desk.  
Ti.

**Gde se piše?**

U svakoj Frappe aplikaciji postoji:

```text
apps/
└── frappe_lab/
    └── frappe_lab/
        └── hooks.py
```

U njemu napišeš:

```python
doc_events = {
    "Book": {
        "on_update": "frappe_lab.book_hooks.trace"
    },
    "User": {
        "validate": "frappe_lab.user_hooks.validate_user"
    },
    "*": {
        "after_insert": "frappe_lab.audit.log_insert"
    }
}
```

**Ko ga izvršava?**

Ne pozivaš ga ti.

Kada Frappe izvršava:

```python
doc.save()
```

ili

```python
doc.insert()
```

interno proverava:  

Da li neka instalirana aplikacija ima `doc_events` za ovaj DocType i ovaj događaj? Ako ima, pozove ih.

Zato se i zove `hook` – "zakačiš" svoju funkciju na događaj.

**Zašto je koristan?**

Najčešći slučaj:

Imaš standardni DocType:

```text
User
```

Ne smeš da menjaš:

```text
apps/frappe/frappe/core/doctype/user/user.py
```

jer će sledeći update Frappe-a pregaziti izmene.

Umesto toga napišeš u svojoj aplikaciji:

```python
doc_events = {
    "User": {
        "validate": "moja_aplikacija.user_hooks.validate_user"
    }
}
```

i gotovo.

**Kada ga JA koristim?**

Ja imam vrlo jednostavno pravilo:

* **Pišem svoj DocType** → koristim metode u kontroleru (`validate()`, `on_update()`, ...).
* **Proširujem tuđi DocType** (Frappe ili ERPNext) → koristim `doc_events`.

To pravilo pokriva ogromnu većinu slučajeva.

Zapravo, mislim da je ovo sve što treba da znaš o `doc_events` za sada. Kasnije ćemo naići na još hook-ova (`override_doctype_class`, `override_whitelisted_methods`, `scheduler_events`, `fixtures`...), ali filozofija je ista: `hooks.py` je mesto gde aplikacija govori Frappe-u kako želi da proširi njegovo ponašanje.

[Sadržaj][00]

[00]: 00%20Učenje%20Frape%20frameworka.md
