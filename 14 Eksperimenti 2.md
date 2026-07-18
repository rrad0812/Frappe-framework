# Učenje Frappe Framework

[Sadržaj][00]

## 14 Eksperimenti - II

Mislim da je sada pravi trenutak da krenemo da **pravimo nešto**, a ne da filozofiramo.

</br>

### Proverimo Module Def

Pošto u `modules.txt` imaš:

```text
Frappe Lab
```

proverimo da li postoji i na sajtu (što si već potvrdio):

```python
frappe.get_all("Module Def", fields=["name", "app_name"])
```

Postoji. ✔

To znači da možemo da koristimo modul **Frappe Lab**.

### Napravi prvi DocType

Uradi u Desk-u:

***Developer → DocType → New***

Popuni:

* **Module:** `Frappe Lab` -  Da bi DocType bio u tom modulu
* **Name:** `Book`
* **Custom:** ❌ (isključeno)
* **Is Submittable:** ❌
* **Is Single:** ❌

Dodaj samo dva polja:

| Label | Type     |
| ----- | -------- |
| Title | Data     |
| Price | Currency |

Sačuvaj.

Pošalji mi izlaz:

```bash
tree -L 5 ~/frappe-bench/apps/frappe_lab/frappe_lab
```

Hoću da vidiš : "šta je Frappe upravo napravio na disku".

To će biti naš prvi pravi susret sa vezom: "DocType u bazi ↔ Python fajlovi ↔ JSON definicija".

I odatle krećemo na lifecycle eksperimente. Mislim da će od tog trenutka sve postati mnogo konkretnije.

---

Evo izlaza

```sh
tree -L 5 ~/frappe-bench/apps/frappe_lab/frappe_lab
```

```sh
/home/radosav/frappe-bench/apps/frappe_lab/frappe_lab
├── config
│   └── __init__.py
├── frappe_lab
│   ├── doctype
│   │   ├── book
│   │   │   ├── book.js
│   │   │   ├── book.json
│   │   │   ├── book.py
│   │   │   ├── __init__.py
│   │   │   ├── __pycache__
│   │   │   │   ├── book.cpython-312.pyc
│   │   │   │   └── __init__.cpython-312.pyc
│   │   │   └── test_book.py
│   │   ├── __init__.py
│   │   └── __pycache__
│   │       └── __init__.cpython-312.pyc
│   ├── __init__.py
│   └── __pycache__
│       └── __init__.cpython-312.pyc
├── hooks.py
├── __init__.py
├── modules.txt
├── patches
│   └── __init__.py
├── patches.txt
├── public
│   ├── css
│   └── js
├── __pycache__
│   ├── hooks.cpython-312.pyc
│   └── __init__.cpython-312.pyc
├── templates
│   ├── includes
│   ├── __init__.py
│   └── pages
│       └── __init__.py
└── www

17 directories, 21 files
```

---

Sada smo konačno na "živom" Frappe-u. I odmah možeš da primetiš jednu vrlo zanimljivu stvar. Ranije smo pričali o `Document` klasi, a sada vidi šta je Frappe napravio:

```text
book/
├── book.json
├── book.py
├── book.js
└── test_book.py
```

Svaki od ovih fajlova ima svoju ulogu.

* `book.json` → **model** (definicija DocType-a: polja, dozvole, opcije...)
* `book.py` → **server-side kontroler** (naslednik `Document`)
* `book.js` → **client-side logika** (forma u browseru)
* `test_book.py` → testovi

Već sada vidiš kako se spajaju baza, Python i JavaScript.

### Prvi mali eksperiment

Otvori:

```bash
nano ~/frappe-bench/apps/frappe_lab/frappe_lab/frappe_lab/doctype/book/book.py
```

ili editor koji koristiš.

Pošalji njegov sadržaj.

Hoću da vidimo šta je Frappe automatski generisao pre nego što bilo šta menjamo.

To će biti prvi fajl koji ćemo "rastaviti" i povezati sa svim onim što smo čitali u `document.py`. Mislim da će ti biti poznat već na prvi pogled.

---

Evo izlaza

```bash
nano ~/frappe-bench/apps/frappe_lab/frappe_lab/frappe_lab/doctype/book/book.py
```

```py
# Copyright (c) 2026, rrad and contributors
# For license information, please see license.txt

# import frappe
from frappe.model.document import Document


class Book(Document):
        pass
```

---

Pogledaj samo ovo:

```python
from frappe.model.document import Document

class Book(Document):
    pass
```

Sećaš se koliko smo vremena proveli u:

```text
frappe/model/document.py
```

E pa sada **`self`** u svim onim metodama koje smo čitali jeste upravo instanca **`Book`**.

Drugim rečima:

```python
doc = frappe.new_doc("Book")
```

napraviće:

```python
Book(Document)
```

a kada pozoveš:

```python
doc.insert()
```

izvršava se **isti `Document.insert()`** koji smo detaljno analizirali, samo što će usput pozivati metode iz tvoje klase `Book`.

### Eksperiment 1 - Prvi pravi eksperiment

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

Zato što smo ih sreli dok smo čitali `Document.insert()`. Sada ćemo proveriti da li se naše razumevanje poklapa sa stvarnim izvršavanjem. To će biti prvi eksperiment u kome ćemo moći da kažemo: "Aha! Ovo je upravo ono što smo pročitali u izvornom kodu."

Mislim da će ti ovo biti mnogo zanimljivije od daljeg čitanja `document.py`. 😊

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

**Ono što očekujem da vidiš**:

Neću još da kažem kojim redom. 😊

Hoću da mi prepišeš izlaz iz terminala, pa ćemo ga uporediti sa onim što smo ranije pročitali u `Document.insert()`.

To će biti naš prvi eksperiment iz Faze 6.

**Hajde jedan mali test**:

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

---

Evo izlaza

```py
In [1]: doc = frappe.new_doc("Book")
   ...: doc.title = "Test"
   ...: doc.price = 100
   ...: doc.insert()
>>> before_insert
>>> before_validate
>>> validate
>>> before_save
>>> after_insert
>>> on_update
Out[1]: <Book: qs827pdgad>

Ovaj ulaz se ne vidi iz http-a?
```

---

Ovo nam govori dve važne stvari.

1. `book.py` je ispravno učitan. ✅
2. Redosled koji smo očekivali se zaista dešava. ✅

Sada na tvoje pitanje: "Ovaj izlaz se ne vidi iz http-a?" rebalo bi da se vidi. Ali samo kada se zaista izvrši `insert()`.

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

**Jedna zanimljiva stvar za domaći zadatak**:

Da li primećuješ nešto "čudno"? Na primer: Zašto se **`on_update`** poziva odmah posle **`after_insert`**?

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

**Zašto se onda zove `on_update`?**

Sećaš se da smo čitali nešto ovako (parafraziram):

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

**Zašto su to uradili?**

Po mom mišljenju (i mislim da je to bila namera autora), zato što ogromna većina poslovne logike ne zanima kako je dokument stigao u trenutno stanje.

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

**E, sada dolazimo do zanimljivog dela Faze 6**:

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

Posle toga ćemo dodati **`doc_events` hook** i posmatrati gde se on tačno "umeće" u isti tok.

To će biti prvi put da vidiš kako se prepliću:

* metoda u `Book` klasi,
* `Document` klasa,
* i `hooks.py`.

Mislim da će to biti jedan od najkorisnijih eksperimenata koje ćemo uraditi.

Mislim da smo sada na tački gde treba da radimo male eksperimente, a ne da čitamo kod.

### Eksperiment 2 – Kompletan lifecycle

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

**Zatim**:

Napravi **novi** Book dokument (ne menjaj postojeći).

Nemoj još da mi kažeš rezultat. 😊

Hoću da **sam** pokušaš da predvidiš redosled izvršavanja.

Možeš čak i da ga zapišeš na papir.

**Šta očekujem da primetiš**:

Posebno obrati pažnju na:

* da li `before_naming` dolazi pre ili posle `validate`,
* kada se poziva `autoname`,
* da li se `on_change` poziva pri `insert()` ili samo pri `save()` postojećeg dokumenta.

To su pitanja na koja nismo eksplicitno odgovorili čitajući kod.

**Posle toga**:

Uradi još jedan eksperiment:

1. Otvori **isti** Book.
2. Promeni samo `Price`.
3. Klikni **Save**.

Uporedi redosled događaja sa prvim snimanjem.

Ako sve bude kako očekujem, dobićemo dva veoma lepa dijagrama:

* **Insert lifecycle**
* **Update lifecycle**

I tek tada ćemo uvesti `hooks.py`, jer ćeš odmah videti gde se hook "ubacuje" u već poznati tok. To je mnogo lakše nego učiti hook-ove napamet.

---

Evo izlaza za INSERT

```sh
22:28:11 web.1         | >>> before_insert
22:28:11 web.1         | >>> before_naming
22:28:11 web.1         | >>> autoname
22:28:11 web.1         | >>> before_validate
22:28:11 web.1         | >>> validate
22:28:11 web.1         | >>> before_save
22:28:11 web.1         | >>> after_insert
22:28:11 web.1         | >>> on_update
22:28:11 web.1         | >>> on_change
```

before_naming ide pre autoname, obadva pre before_validate.

Evo izlaza za UPDATE

```sh
22:31:24 web.1         | >>> before_validate
22:31:24 web.1         | >>> validate
22:31:24 web.1         | >>> before_save
22:31:24 web.1         | >>> on_update
22:31:24 web.1         | >>> on_change
```

U ovom slučaju nema poziva povezanih sa:

before_insert, before_naming, autoname, after_insert.

Sada mi deluje da je on_update namenjen svemu što treba promeniti ali u GUI posle INSERT, isto tako on_change ali posle UPFATE.

Što se tiče DB, tu su ostali metodi tipa _insert ili \_save.

---

**Ono što si dokazao**:

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

---

**Jedna mala ispravka**:

Napisao si: "on_update je namenjen svemu što treba promeniti u GUI..."

Ne.

`on_update` nema veze sa GUI-jem.

On se izvršava na serveru.

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

Odgovor neću odmah dati. 😊

Predlažem da ga sami otkrijemo sledećim eksperimentom.

### Eksperiment 3 - Najzanimljiviji do sada

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

**Zašto baš ovo?**

Zato što ćemo prvi put videti kada se tačno poziva hook u odnosu na metode klase `Book`.

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
        "*": "frappe_lab.book_hooks.trace"
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

To će biti prvi put da se vide sva tri sloja zajedno. Mislim da će biti baš zanimljivo.

---

Evo izlaza:

```ssh
22:53:17 web.1         | >>> before_insert
22:53:17 web.1         | >>> before_naming
22:53:17 web.1         | >>> autoname
22:53:17 web.1         | >>> before_validate
22:53:17 web.1         | >>> validate
22:53:17 web.1         | >>> before_save
22:53:17 web.1         | >>> after_insert
22:53:17 web.1         | >>> on_update
22:53:17 web.1         | >>> on_change
22:53:17 web.1         | 192.168.122.1 - - [18/Jul/2026 22:53:17] "POST /api/method/frappe.desk.form.save.savedocs HTTP/1.1" 200 -
22:53:32 web.1         | >>> before_validate
22:53:32 web.1         | >>> validate
22:53:32 web.1         | >>> before_save
22:53:32 web.1         | >>> on_update
22:53:32 web.1         | >>> on_change
22:53:32 web.1         | 192.168.122.1 - - [18/Jul/2026 22:53:32] "POST /api/method/frappe.desk.form.save.savedocs HTTP/1.1" 200 -
```

Bojim se da nešto nije baš kako treba. Ja ovde ne vidim sve što si rekao da treba da se vidi a namestio sam book_hooks.py u frappe_lab/frappe_lab (drugi po dubini)

---

Dobro si primetio. I zapravo si našao važan detalj. Nije problem u lokaciji `book_hooks.py`. Ta lokacija je dobra:

```text
/home/radosav/frappe-bench/apps/frappe_lab/frappe_lab/book_hooks.py
```

jer odgovara importu:

```python
frappe_lab.book_hooks.trace
```

Ali fali nam jedan korak.

Zašto se hook ne vidi? Najverovatnije zato što `hooks.py` nije ponovo učitan.
`doc_events` se učitava kroz Frappe hook mehanizam i kešira se.

Uradi:

```bash
bench --site site1.local clear-cache
```

zatim:

```bash
bench restart
```

(Ako radiš samo `bench start`, prekini ga i ponovo pokreni.)

U `bench --site site1.console console` proveri:

```python
frappe.get_hooks("doc_events")
```

Trebao bi da vidiš nešto slično:

```python
{
    "Book": {
        "*": [
            "frappe_lab.book_hooks.trace"
        ]
    }
}
```

Ako nema `Book`, Frappe još ne zna za hook.

Tvoj izlaz:

```text
>>> before_insert
...
...
>>> on_change
```

pokazuje samo metode iz `Book` klase.

Kada hook proradi, očekujemo dodatno:

```text
HOOK ---> before_insert
HOOK ---> before_validate
...
```

ali ne nužno za svaki događaj koji postoji u klasi.

Zašto? Zato što `doc_events` i `Document` lifecycle nisu ista stvar.

Hook-ovi se pozivaju kroz drugi mehanizam (`run_doc_method` / `run_method` lanac), i lista dostupnih događaja nije identična listi metoda koje možeš napisati u kontroleru.

Dakle sada samo proveri prvo:

```python
frappe.get_hooks("doc_events")
```

To će nam odmah reći da li je problem:

* učitavanje hook-a,
* putanja modula,
* ili nešto treće.

Ovo je zapravo dobar eksperiment, jer upravo učimo kada Frappe čita hooks.py, a ne samo gde se napiše.

---

Book je prisutan na listi kao poslednji :

'Book': {'*': ['frappe_lab.book_hooks.trace']}}

---

To znači da je hook registrovan i učitan.

Dakle problem nije:

* ❌ putanja `book_hooks.py`
* ❌ import string
* ❌ `hooks.py`
* ❌ cache

Sada imamo drugi zaključak: naša očekivanja o redosledu su bila pogrešna.

`doc_events` hook sa:

```python
"Book": {
    "*": "frappe_lab.book_hooks.trace"
}
```

ne znači: "pozovi me za svaki interni `Document` lifecycle metod", nego: "pozovi me za Frappe document event-e koji prolaze kroz `run_doc_event` mehanizam."

Drugim rečima, događaji kao:

```text
before_insert
validate
on_update
after_insert
```

mogu da se pojave i u jednom i u drugom kontekstu, ali nisu isti pozivni putevi.

Hajde da sada uradimo malu proveru.

Promeni `book_hooks.py`:

```python
import frappe

def trace(doc, method):
    frappe.logger().info(f"HOOK ---> {method}")
    print(f"HOOK ---> {method}")
```

I onda u `bench console`:

```python
doc = frappe.get_doc("Book", "qs827pdgad")
doc.save()
```

(umesto ovog imena stavi stvarno ime nekog postojećeg Book zapisa)

Očekujem da sada vidimo nešto kao:

```text
HOOK ---> validate
HOOK ---> on_update
```

ili slično.

I još jedna stvar: moram da ispravim jednu stvar iz prethodne poruke. Rekao sam da očekujemo hook za `before_insert` itd. To je bilo previše široko rečeno. `doc_events` ne treba posmatrati kao "drugu kopiju" svih metoda klase `Document`, nego kao spoljašnji mehanizam za proširivanje postojećeg lifecycle-a.

Ovo je zapravo lep trenutak za ono praktično pitanje: Kada staviti kod u `Book.validate()`, a kada u `doc_events["Book"]["validate"]`?

Tu ćemo doći odmah posle ovog testa.

---

Nema izlaza, nikakvog!

---

Dobro. To je zapravo koristan rezultat. Znači da smo sada eliminisali još jednu pretpostavku.

Ako:

```python
frappe.get_hooks("doc_events")
```

vraća:

```python
'Book': {'*': ['frappe_lab.book_hooks.trace']}
```

a pri:

```python
doc.save()
```

nema:

```text
HOOK ---> ...
```

onda hook jeste registrovan, ali se ne izvršava preko ovog wildcard zapisa kako smo očekivali.

Hajde da ne nagađamo. Uradimo najprostiji test.

Promeni `hooks.py`:

umesto:

```python
doc_events = {
    "Book": {
        "*": "frappe_lab.book_hooks.trace"
    }
}
```

stavi:

```python
doc_events = {
    "Book": {
        "on_update": "frappe_lab.book_hooks.trace"
    }
}
```

Znači samo jedan događaj.

Zatim:

```bash
bench --site site1.local clear-cache
```

pa ponovi:

```python
doc = frappe.get_doc("Book", "IME_NEKOG_POSTOJEĆEG_BOOKA")
doc.save()
```

Očekujemo:

```text
HOOK ---> on_update
```

Ako ni tada nema ništa, sledeća stvar koju proveravamo je da li se učitava **ista aplikacija** koju menjamo (dešava se kod više bench/site kombinacija).

Ali prvo ovaj test. Ovo je inače dobra lekcija: `doc_events` nije samo "dodaj funkciju u hooks.py i ona se magično pozove"; postoji još jedan sloj kroz koji Frappe prolazi.

---

Odstupio sam i pokušao iz interfejsa da updejtujem.

ModuleNotFoundError: No module named 'frappe_lab.book_hooks'

---

E, ovo je zapravo **dobar trag**. Sada znamo šta se dešava.

Prethodni test iz `bench --site site1.local console` nije pokazao ništa zato što hook verovatno nije bio pozvan. Ali GUI sada jeste pokušao da ga pozove i dobili smo:

```text
ModuleNotFoundError: No module named 'frappe_lab.book_hooks'
```

To znači:

* `hooks.py` je pročitan ✅
* `doc_events` je pronađen ✅
* Frappe je pokušao:

```python
import frappe_lab.book_hooks
```

ali Python ne vidi taj fajl.

Dakle sada proveravamo putanju.

Molim te pošalji:

```bash
ls -l ~/frappe-bench/apps/frappe_lab/frappe_lab/
```

i:

```bash
ls -l ~/frappe-bench/apps/frappe_lab/frappe_lab/frappe_lab/
```

Jer ovde postoji jedna česta zabuna.

Tvoja struktura trenutno ima:

```text
frappe_lab/
├── frappe_lab/
│   ├── doctype/
│   │   └── book/
│   ├── hooks.py
│   ├── modules.txt
│   └── ...
```

Dakle Python paket je:

```text
frappe_lab/frappe_lab/
```

i tu treba da bude:

```text
frappe_lab/frappe_lab/book_hooks.py
```

Ali postoji mogućnost da si ga slučajno stavio u:

```text
frappe_lab/frappe_lab/frappe_lab/book_hooks.py
```

(ono što smo ranije zvali "treći nivo").

Još jedna brza provera:

U `bench --site site1.local console`:

```python
import frappe_lab
print(frappe_lab.__file__)
```

i:

```python
import frappe_lab.book_hooks
```

Druga linija mora da prođe bez greške.

Ovo je sada običan Python import problem, ne Frappe problem. I dobro je što se pojavio — upravo smo došli do mesta gde se vidi razlika između strukture aplikacije i Python paketa.

---

```sh
ls -l ~/frappe-bench/apps/frappe_lab/frappe_lab/
```

```sh
total 48
drwxrwxr-x 2 radosav radosav 4096 jul 17 23:16 config
drwxrwxr-x 4 radosav radosav 4096 jul 18 23:44 frappe_lab
-rw-rw-r-- 1 radosav radosav 6182 jul 18 23:44 hooks.py
-rw-rw-r-- 1 radosav radosav   22 jul 17 23:16 __init__.py
-rw-rw-r-- 1 radosav radosav   10 jul 17 23:16 modules.txt
drwxrwxr-x 2 radosav radosav 4096 jul 17 23:16 patches
-rw-rw-r-- 1 radosav radosav  290 jul 17 23:16 patches.txt
drwxrwxr-x 4 radosav radosav 4096 jul 17 23:16 public
drwxrwxr-x 2 radosav radosav 4096 jul 18 23:45 __pycache__
drwxrwxr-x 4 radosav radosav 4096 jul 17 23:16 templates
drwxrwxr-x 2 radosav radosav 4096 jul 17 23:16 www
```

```sh
ls -l ~/frappe-bench/apps/frappe_lab/frappe_lab/frappe_lab/
```

```sh
total 12
-rw-rw-r-- 1 radosav radosav  120 jul 18 23:37 book_hooks.py
drwxrwxr-x 4 radosav radosav 4096 jul 18 19:03 doctype
-rw-rw-r-- 1 radosav radosav    0 jul 17 23:16 __init__.py
drwxrwxr-x 2 radosav radosav 4096 jul 17 23:44 __pycache__
```

---

Evo ga. Našli smo problem.

I ovde moram da ispravim sebe: ranije sam te naveo na pogrešan nivo direktorijuma.

Tvoja struktura:

```text
/home/radosav/frappe-bench/apps/frappe_lab/
└── frappe_lab/                  ← Python paket aplikacije
    ├── hooks.py
    ├── modules.txt
    ├── __init__.py
    └── frappe_lab/              ← Frappe modul "Frappe Lab"
        ├── doctype/
        └── book_hooks.py   ❌
```

A u `hooks.py` smo napisali:

```python
"frappe_lab.book_hooks.trace"
```

To znači:

```python
import frappe_lab.book_hooks
```

Python traži:

```text
/home/radosav/frappe-bench/apps/frappe_lab/frappe_lab/book_hooks.py
```

a ti ga trenutno imaš ovde:

```text
/home/radosav/frappe-bench/apps/frappe_lab/frappe_lab/frappe_lab/book_hooks.py
```

Dakle samo jedan nivo previsoko.

Uradi:

```bash
mv ~/frappe-bench/apps/frappe_lab/frappe_lab/frappe_lab/book_hooks.py \
   ~/frappe-bench/apps/frappe_lab/frappe_lab/book_hooks.py
```

Posle proveri:

```bash
ls -l ~/frappe-bench/apps/frappe_lab/frappe_lab/
```

Treba da vidiš:

```text
book_hooks.py
hooks.py
modules.txt
...
```

Zatim:

```bash
bench --site site1.local clear-cache
```

i u browseru uradi ponovo **Save** postojećeg Book-a.

Sada očekujemo:

```text
>>> before_validate
>>> validate
>>> before_save
HOOK ---> on_update
>>> on_update
>>> on_change
```

(tačan redosled između hook-a i metode proverićemo, to je sada baš eksperiment).

Inače, ovo je bila veoma dobra greška za učenje. Upravo smo praktično dokazali razliku između:

* Python paketa aplikacije
* Frappe modula
* DocType direktorijuma

što nas je ranije i bunilo.

---

```sh
>>> before_validate
23:54:14 web.1         | >>> validate
23:54:14 web.1         | >>> before_save
23:54:14 web.1         | >>> on_update
23:54:14 web.1         | HOOK ---> on_update
23:54:14 web.1         | >>> on_change
23:54:15 web.1         | 192.168.122.1 - - [18/Jul/2026 23:54:15] "POST /api/method/frappe.desk.form.save.savedocs HTTP/1.1" 200 -
```

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

Sledeći logičan korak bih ja uradio: **isti eksperiment na standardnom Frappe DocType-u**.

Na primer ne bih odmah uzimao ERPNext (jer ga nemaš instaliranog), nego neki Frappe-ov standardni DocType, recimo `User`.

Napravili bismo hook na `User` i videli kako se proširuje tuđi kod bez menjanja Frappe izvora.

To je praktično ono što se radi svaki dan u Frappe razvoju.

Danas si praktično prešao jednu veliku granicu:

Pre nekoliko dana si gledao:

```text
frappe/model/document.py
```

i pokušavao da razumeš šta znače:

* `run_method`
* lifecycle događaji
* `insert`
* `save`
* hook sistem

A danas si napravio svoj DocType i video uživo:

```text
Document -> Book controller -> doc_events hook
```

sa stvarnim redosledom izvršavanja.

Sledeća stanica bi bila:

* još malo `doc_events` (kada se koristi, kada ne),
* pa onda prelazak na standardni Frappe DocType,
* i poređenje: **moj kod u controlleru vs proširenje preko hook-a**.

---

[Sadržaj][00]

[00]: 00%20Učenje%20Frape%20frameworka.md
