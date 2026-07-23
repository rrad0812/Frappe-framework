# Učenje Frappe Framework

[Sadržaj][00]

## 13 Eksperimenti - I

Mislim da je ovo odličan trenutak da promenimo perspektivu.

Do sada smo radili ono što radi vrlo malo ljudi – čitali smo izvorni kod Frappe-a i pratili kako se izvršava `Document`. To je bilo korisno jer sada znaš zašto se nešto dešava, a ne samo šta se dešava.

Sada je mnogo korisnije da krenemo "odozgo nadole":

* prvo napišemo kod kao Frappe programeri,
* posmatramo šta se dešava,
* pa tek onda, ako nešto nije jasno, vratimo se u izvorni kod.

Mislim da ćeš tada mnogo lakše povezati sve što smo čitali u `document.py`.

### Predlog Faze 6

Ja bih je malo proširio.

#### Faza 6.1 — Napraviti sopstveni DocType

Napravićemo novu aplikaciju (ako već nemaš neku za eksperimente) ili ćemo koristiti postojeću.

Napravićemo:

```txt
Book
```

sa nekoliko polja, na primer

```txt
title
author
price
published
```

Ništa komplikovano.

#### Faza 6.2 — Instrumentacija životnog ciklusa

U `book.py` ćemo implementirati skoro sve hook metode.

Na primer:

```python
def before_insert(self):
    frappe.logger().info("before_insert")

def validate(self):
    frappe.logger().info("validate")

def before_save(self):
    frappe.logger().info("before_save")
...
```

ili još bolje

```python
print("before_insert")
```

pošto je Bench u `development modu` i odmah ćemo videti izlaz u konzoli.

Cilj nije da rade nešto korisno. Cilj je da vidimo: kojim redom se zaista pozivaju.

#### Faza 6.3 — Eksperimenti

Radićemo:

```py
doc.insert()
```

zatim

```py
doc.save()
```

zatim

```py
doc.submit()
```

zatim

```py
doc.cancel()
```

zatim

```py
doc.delete()
```

i svaki put zapisati redosled.

Na kraju ćemo imati tabelu poput:

| Operacija | Redosled metoda |
| --------- | --------------- |
| insert()  | ...             |
| save()    | ...             |
| submit()  | ...             |
| cancel()  | ...             |

To je nešto što retko gde postoji ovako jasno dokumentovano, a izuzetno je korisno za svakodnevni razvoj.

#### Faza 6.4 — Hook-ovi

Zatim isti DocType povezujemo preko `hooks.py`.

Na primer:

```python
doc_events = {
    "Book": {
        "validate": "...",
        "before_save": "...",
        "after_insert": "..."
    }
}
```

Opet posmatramo:

* da li se prvo izvrši kontroler,
* pa hook,
* ili obrnuto.

To smo delimično videli u `Document.hook()`, ali sada ćemo potvrditi u praksi.

#### Faza 6.5 — Više aplikacija

Napravićemo još jednu malu aplikaciju.

U njoj ćemo registrovati isti `doc_event`.

Videćemo:

* kojim redom se izvršavaju hook-ovi iz više aplikacija,
* od čega taj redosled zavisi.

To je veoma praktično kada radiš na većim Frappe sistemima sa više instaliranih aplikacija.

#### Faza 6.6 — Standardni DocType

Tek onda uzimamo nešto iz Frappe-a ili ERPNext-a.

Na primer:

```txt
ToDo
```

ili

```txt
User
```

ili neki jednostavan standardni DocType.

Dodamo `doc_event`.

Posmatramo da li je tok isti.

Time potvrđujemo da ne postoji neka "magija" između standardnih i custom DocType-ova – svi prolaze kroz isti mehanizam `Document` klase.

#### Faza 6.7 — Povezivanje sa izvornim kodom

Tek na kraju vraćamo se u `document.py`.

Sada će svaka metoda koju smo ranije čitali imati konkretan smisao.

Na primer:

```txt
    insert() ->
    run_method("before_insert") ->
    run_before_save_methods() - >
    _validate() ->
    db_insert() ->
    run_method("after_insert")
```

To više neće biti samo kod koji smo pročitali, već nešto što smo videli kako radi u praksi.

---

Ok, slažem se uz dve napomene:

* bench nije u dev mode i
* nemam aplikaciju izgrađenu tj pridruženu site1.local sajtu.

---

Odlično, obe napomene su važne i zapravo nam pomažu da Fazu 6 uradimo "kako treba", a ne prečicama.

#### 1. Bench nije u development modu

To nije nikakav problem.

Imamo nekoliko mogućnosti za praćenje izvršavanja:

1. **`print()` u konzoli** – radi samo ako je proces koji izvršava kod pokrenut iz terminala i stdout nije preusmeren. Nije najpouzdanije.
2. **`frappe.logger()`** – profesionalniji pristup, zapis ide u log fajlove.
3. **`frappe.log_error()`** – koristan za pojedinačne događaje, ali nije namenjen za ovakvo praćenje.

Ja bih koristio **`frappe.logger()`**, jer je to način koji ćeš koristiti i kasnije u realnim aplikacijama.

#### 2. Nemaš svoju aplikaciju

Po mom mišljenju, ovo je zapravo idealno. Ne bih pravio DocType u samom Frappe-u. To nije dobra praksa.

Ja bih Fazu 6 podelio na još dva mala koraka.

### Prva Frappe aplikacija

Pre nego što počnemo sa hook-ovima, napravićemo svoju aplikaciju.

Ne zato što nam odmah treba, već zato što će nam ona služiti tokom celog ostatka učenja.

Na primer:

```sh
frappe_lab
```

Ja čak više volim nazive poput **`playground`** ili **`lab`**, jer odmah govore da je to eksperimentalna aplikacija.

Posle toga:

```sh
bench --site site1.local install-app frappe-lab
```

Time dobijaš potpuno odvojeno mesto za eksperimente.

#### Prvi Custom DocType

Tek tada pravimo

```txt
Book
```

ili

```txt
TestDocument
```

unutar svoje aplikacije.

To je mnogo bliže načinu na koji ćeš kasnije razvijati stvarne projekte.

Pošto si već dosta upoznao unutrašnjost Frappe-a, mislim da od ovog trenutka više ne treba da radimo u stilu: "Evo narednih 15 komandi, izvrši ih."

Radije bih radio ovako:

1. objasnim zašto nešto radimo,
2. dam jednu ili dve komande,
3. ti izvršiš,
4. pogledamo rezultat,
5. pa odlučimo šta je sledeće.

Tako ćemo mnogo više obraćati pažnju na ono što Frappe radi, a mnogo manje na "odrađivanje liste komandi".

Mislim da smo sada na prelazu između dve faze učenja:

* **Faze 1–5**: "Kako je Frappe napravljen?"
* **Faze 6 nadalje**: "Kako se Frappe koristi za razvoj aplikacija?"

To je prirodan nastavak projekta i mislim da će od sada tempo biti nešto brži, jer već imaš solidno razumevanje unutrašnje arhitekture.

---

Dobro, ali sam mislio da ćeš da mi kažeš kako da prebacim frappe u dev mode!

---

Toliko sam razmišljao o planu da sam preskočio ono što si zapravo pitao.

Međutim, tu postoji jedna važna stvar kod Frappe-a: "development mode" nije osobina Bench-a, već pojedinačnog Site-a.

Dakle, ne prebacuješ bench u development mode, već `site1.local`.

Najpre proveri trenutno stanje:

```bash
cd ~/frappe-bench

bench --site site1.local console
```

U Python konzoli:

```python
frappe.conf.developer_mode
```

Ako dobiješ:

```python
0
```

ili

```python
False
```

onda development mode nije uključen.

Izađi iz konzole:

```python
exit()
```

ili

```python
quit()
```

Zatim uključi development mode:

```bash
bench --site site1.local set-config developer_mode 1
```

To će upisati u:

```sh
sites/site1.local/site_config.json
```

stavku:

```json
{
    "developer_mode": 1
}
```

Posle toga je dobro da restartuješ procese Bench-a.

Ako koristiš:

```bash
bench start
```

dovoljno je da prekineš (`Ctrl+C`) i ponovo pokreneš:

```bash
bench start
```

Ako koristiš production setup (`bench setup supervisor` / systemd), onda treba restartovati odgovarajuće servise, ali pošto smo mi u razvojnoj VM, verujem da koristiš `bench start`.

#### developer mode

Za ono što planiramo u Fazi 6 – **da**.

Ne zbog `print()` funkcije, već zato što developer mode omogućava stvari koje su namenjene razvoju, između ostalog:

* kreiranje i izmenu Custom DocType-ova i drugih razvojnih objekata na očekivan način,
* izvoz promena u aplikaciju kada budemo imali svoju aplikaciju (`bench export-fixtures` i druge razvojne tokove),
* razvoj koji je usklađen sa načinom na koji Frappe očekuje da se razvijaju aplikacije.

Dakle, ja bih ga uključio pre nego što napravimo našu prvu aplikaciju.

Usput, predlažem da odmah na početku Faze 6 uradimo i jednu malu proveru: pogledaćemo zajedno šta se konkretno menja kada je `developer_mode = 1`, a šta ostaje potpuno isto. To je jedna od onih stvari koje se često pogrešno razumeju kod Frappe-a, pa vredi da razjasnimo pre nego što nastavimo.

---

Problemi.
Uspešno sam svičovao developer_mode.
Uspešno sam izgradio aplikaciju frappe-lab sa `bench new-app frappe-lab`.
Ali onda:

```sh
bench --site site1.local install-app frappe-lab
```

```sh
Could not find app "frappe-lab": 
No module named 'frappe-lab'
An error occurred while installing frappe-lab: No module named 'frappe-lab'
```

Ideja?

---

Problem je vrlo verovatno u nazivu aplikacije. Hajde prvo da utvrdimo šta je Bench zaista napravio:

Pošalji izlaz sledećih komandi.

```bash
ls -l ~/frappe-bench/apps
```

</br>

Šta Bench vidi kao instalirane aplikacije:

```bash
bench version
```

U pravu si, napravio sam je kao frappe_lab. Uspešno sam je poveza0 sa site1.local posle

`bench --site site1.local install-app frappe_lab`

---

Razlog je jednostavan:

* Naziv direktorijuma / Python paketa mora biti validan Python identifikator, dakle:

  ```text
  frappe_lab
  ```

* Kada radiš:

  ```bash
  bench --site site1.local install-app ...
  ```

  uvek koristiš Python naziv aplikacije, a ne "lep" naziv sa crticama.

Iz traceback-a se lepo videlo:

```python
importlib.import_module("frappe-lab.hooks")
```

što Python nikada ne može da učita, jer `frappe-lab` nije validan naziv modula.

Po mom mišljenju, upravo smo završili ono što bih nazvao Faza 6.0:

* ✅ `developer_mode = 1`
* ✅ napravljena aplikacija `frappe_lab`
* ✅ aplikacija instalirana na `site1.local`

To je odlična polazna tačka.

Ja ne bih odmah pravio DocType. Pre toga bih voleo da upoznaš strukturu aplikacije koju je Bench napravio.

Dakle, da odgovorimo na pitanja:

* Zašto postoje **dva** direktorijuma `frappe_lab`?
* Šta je `modules.txt`?
* Čemu služi `hooks.py`?
* Šta je `patches.txt`?
* Zašto postoji `config/desktop.py`?
* Šta je `www/`?
* Šta je `public/`?
* Šta je `templates/`?

To je otprilike 20–30 minuta rada, a posle toga će ti biti potpuno jasno gde ćemo smeštati DocType, hook-ove, API-je, Jinja templejte, statičke fajlove itd.

Pošto si već proveo dosta vremena proučavajući unutrašnjost Frappe-a, mislim da zaslužuješ da i svoju prvu aplikaciju razumeš od temelja, umesto da je samo koristiš kao "kutiju" u koju ubacuješ kod.

A onda krećemo na **Book** DocType i eksperimente sa životnim ciklusom `Document` klase. Mislim da će ti tada sve što smo proučavali u `document.py` "kliknuti" na svoje mesto.

Od sada menjamo stil rada:

* **kratko objašnjenje** (2–5 minuta),
* **malo prakse**,
* **malo diskusije**,
* pa sledeći korak.

#### Struktura aplikacije

> [!Note]
> **AI Zahtev**  
> Pošalji izlaz
>
> ```bash
> tree -L 2 ~/frappe-bench/apps/frappe_lab
> ```
>
> Ako nemaš `tree`:
>
> ```bash
> sudo apt install tree
> ```

</br>

> [!Info] Izlaz
>
> ```sh
> tree -L 2 ~/frappe-bench/apps/frappe_lab
> ```
>
> ```sh
> /home/radosav/frappe-bench/apps/frappe_lab
> ├── frappe_lab
> │   ├── config
> │   ├── frappe_lab
> │   ├── hooks.py
> │   ├── __init__.py
> │   ├── modules.txt
> │   ├── patches
> │   ├── patches.txt
> │   ├── public
> │   ├── __pycache__
> │   ├── templates
> │   └── www
> ├── license.txt
> ├── pyproject.toml
> └── README.md
> 
> 9 directories, 7 files
> ```

Ovo je sasvim standardna struktura. Hajde da označimo šta nam je **bitno sada**, a šta **kasnije**.

```sh
frappe_lab/
├── pyproject.toml        ⭐
├── README.md             😴
├── license.txt           😴
└── frappe_lab/
    ├── __init__.py       ⭐
    ├── hooks.py          ⭐⭐⭐⭐⭐
    ├── modules.txt       ⭐⭐⭐
    ├── config/           ⭐⭐
    ├── frappe_lab/       ⭐⭐⭐⭐
    ├── patches/          ⏳
    ├── patches.txt       ⏳
    ├── public/           ⏳
    ├── templates/        ⏳
    └── www/              ⏳
```

Za sada su nam važna samo četiri mesta

</br>

* **`hooks.py`**

  Srce aplikacije. Ovde se registruje skoro sve:
  
  * `doc_events`
  * scheduler
  * override-i
  * fixtures
  * Jinja
  * itd.
  
  Tu ćemo provesti dosta vremena.

</br>

* **`modules.txt`**

  Kaže Frappe-u koje module aplikacija ima. Kasnije će ovde stajati, recimo:
  
  ```sh
  Frappe Lab
  ```
  
  i taj modul će se pojaviti na Desk-u.

</br>

* **`frappe_lab/frappe_lab/`**

  Ovo je glavni Python paket tvoje aplikacije. Ovde će nastajati:
  
  * DocType-ovi
  * API funkcije
  * Business logika
  * itd.

</br>

* **`config/`**

  Za sada samo zapamti da je vezan za Desk/module konfiguraciju.
  Ne diramo još.

</br>

Primećuješ li nešto zanimljivo ovde?

```sh
apps/
└── frappe_lab/
    └── frappe_lab/
        └── frappe_lab/
```

Dakle imamo **tri puta** `frappe_lab`.

To nije greška.

Možeš li da pogodiš zašto postoje tri nivoa istog imena? To će nam biti lep uvod u narednih pet minuta, jer iza toga stoji logika Bench-a i Python paketa.

---

Prvi je aplikacija. Drugi je python paket, za ovu web aplikaciju, a treći je python paket ukome će biti frape kod.

---

Blizu si, ali treći nije baš to.

```sh
~/frappe-bench/apps/
└── frappe_lab/          ← 1. Git projekat (repozitorijum aplikacije)
    └── frappe_lab/      ← 2. Glavni Python paket aplikacije
        └── frappe_lab/  ← 3. Modul (business module) u Frappe-u
```

</br>

* **`apps/frappe_lab`**

  Ovo je običan Git projekat.
  
  Tu su:
  
  * `README.md`
  * `pyproject.toml`
  * `.git`
  * itd.
  
  To Python uopšte ne zanima.

* **`apps/frappe_lab/frappe_lab`**

  Ovo je **Python paket**.
  
  Zato tu postoji:
  
  ```sh
  __init__.py
  hooks.py
  modules.txt
  ...
  ```
  
  Kada Python izvrši:
  
  ```python
  import frappe_lab
  ```
  
  uvozi upravo ovaj paket.

* **`apps/frappe_lab/frappe_lab/frappe_lab`**

  Ovo nije novi Python paket zbog Pythona, već **Frappe modul**.
  
  U njemu će kasnije biti nešto poput:
  
  ```sh
  doctype/
  page/
  report/
  dashboard/
  ...
  ```
  
  Drugim rečima, ovo predstavlja jedan **Module Def** u Frappe-u.
  
  Da jednog dana napraviš još jedan modul, dobio bi:
  
  ```sh
  frappe_lab/
      accounting/
      crm/
      warehouse/
  ```
  
  Svi oni bi bili moduli iste aplikacije.

* **Zašto je ovo urađeno ovako?**

  Zato što jedna aplikacija može imati više Frappe modula.
  
  Na primer ERPNext je jedna aplikacija, ali ima module:
  
  * Accounts
  * Buying
  * Selling
  * HR
  * CRM
  * Manufacturing
  * ...
  
  Svaki od njih izgleda upravo kao ovaj tvoj treći direktorijum.

Mislim da je ovo dovoljno za danas iz strukture aplikacije.

Sada prelazimo na nešto mnogo zanimljivije:

**Napravićemo prvi modul i prvi DocType**, pa ćemo konačno početi da pratimo životni ciklus `Document` klase u praksi.

---

Ok, ali mi nije jasno kako se prave ti novi moduli. U stvari, pošto sam ranije čuo pojam modul mislio sam da se radi o aplikaciji?

---

Odlično pitanje. I tu si uhvatio jednu od najvećih Frappe terminoloških zamki. 😄

U Frappe svetu reč **module** se koristi za dve različite stvari, pa ljudi često pomešaju.

Da razdvojimo:

```sh
Aplikacija (App)
    |
    +-- Modul (Module)
            |
            +-- DocType
            +-- Report
            +-- Page
            +-- ...
```

</br>

**App (aplikacija)**:

To je ono što smo upravo napravili:

```text
frappe_lab
```

App je Python/Git projekat.

Pravi se:

```sh
bench new-app frappe_lab
```

App može da se instalira na site:

```bash
bench --site site1.local install-app frappe_lab
```

Jedan site može imati:

```sh
frappe
erpnext
frappe_lab
moja_druga_app
```

više aplikacija.

</br>

**Module (Frappe modul)**:

Modul je **organizacija sadržaja unutar aplikacije**.

Na primer ERPNext:

```sh
erpnext (App)
|
+-- Accounts (Module)
|
+-- Selling (Module)
|
+-- Buying (Module)
|
+-- Stock (Module)
```

Dakle: ERPNext nije 10 aplikacija. To je **jedna aplikacija sa mnogo modula**.

  </br>
  
* **Kako se pravi novi modul?**
  
  Modul se pravi kroz:
  
  ```bash
  bench --site site1.local new-module
  ```
  
  ili češće kroz Desk:
  
  ```txt
  Developer > Module Def > New
  ```
  
  Dobiješ nešto poput:
  
  ```sh
  frappe_lab
  |
  +-- Accounting
  |
  +-- Library
  |
  +-- Whatever
  ```
  
  Za naš slučaj, ja bih napravio:
  
  ```sh
  frappe_lab (App)
  |
  +-- Lab (Module)
        |
        +-- Book (DocType)
  ```
  
  Jer je logično:
  
  * **App*+ = naša aplikacija za učenje
  * **Module** = oblast unutar aplikacije
  * **DocType** = konkretan poslovni objekat
  
  Samo Frappe koristi istu reč za oba koncepta, Frappe modul i Python modul.
  
Sledeći mali korak bih predložio:
  
* pogledamo postojeći `modules.txt`
* napravimo naš prvi **Lab Module**
* onda napravimo **Book DocType**
  
  Tu će se konačno spojiti: `App → Module → DocType → Document lifecycle`.

---

Ovo je totalno čudno.

`bench --site site1.local new-module`  

Ovako ispada da je modul vezan za sajt a ne za aplikaciju, i osim toga kako je on deo frappe_lab app?

---

Da. I dobro si primetio problem — moj prethodni odgovor je tu bio previše pojednostavljen i napravio je zabunu.

U stvari, Frappe **modul** nije isto što i **modul aplikacije** u klasičnom smislu.

Hajde da ispravimo model.

**Ključna stvar**:

`Module Def` je zapis u bazi jednog Site-a.

Zato komanda:

```bash
bench --site site1.local new-module
```

ima smisla.

Ona pravi `Module Def` DocType zapis u `site1.local` bazi.

Dakle:

```sh
site1.local (baza)
|
+-- Module Def
       |
       +-- Lab
```

To nije odmah Python direktorijum.

**A gde je onda aplikacija?**

Aplikacija dobija vezu kroz sadržaj koji napraviš.

Na primer:

```sh
frappe_lab (app)
|
+-- frappe_lab
    |
    +-- lab
        |
        +-- doctype
```

Tu dolaze stvari kao:

```sh
module = "Lab"
```

u DocType JSON fajlovima.

Znači:

```sh
DocType
    |
    +-- module: Lab
```

govori Frappe-u:

"Ovaj DocType pripada modulu Lab."

**Zašto onda postoji `modules.txt`?**

Tu je druga zabuna.

U tvojoj aplikaciji imaš:

```sh
frappe_lab/modules.txt
```

On sadrži listu modula koje aplikacija obezbeđuje.

Na primer:

```sh
Frappe Lab
```

To nije Python import lista. To je više metadata za Frappe.

**Praktično za nas**:

Mi ćemo verovatno uraditi ovako:

* Imamo app:

  ```txt
  frappe_lab
  ```

* Napravimo modul:

  ```txt
  Lab
  ```

* Kreiramo DocType:

  ```txt
  Book
  ```

  i on će imati:
  
  ```json
  {
      "module": "Lab"
  }
  ```

* Taj DocType se čuva u aplikaciji:

  ```sh
  frappe_lab/
      frappe_lab/
          lab/
              doctype/
                  book/
  ```

Ovo je baš dobra tačka da stanemo sa "pravljenjem" i prvo pogledamo konkretno.

Predlažem sledeće:

> [!Note]
> **AI Zahtev**  
>
> ```bash
> cat ~/frappe-bench/apps/frappe_lab/frappe_lab/modules.txt
> ```
>
> i :
>
> ```bash
> bench --site site1.local console
> ```
>
> pa u konzoli:
>
> ```python
> frappe.get_all("Module Def", fields=["name", "app_name"])
> ```

Mislim da će ti ovaj rezultat odmah "složiti sliku".

> [!Info] Izlaz
>
> ```sh
> cat ~/frappe-bench/apps/frappe_lab/frappe_lab/modules.txt
> ```
>
> ```sh
> Frappe Lab
> ```
>
> ```sh
> bench --site site1.local console
> Apps in this namespace:
> frappe, frappe_lab
> ```
>
> ```py
> In [1]: frappe.get_all("Module Def", fields=["name", "app_name"])
> Out[1]: 
> [{'name': 'Frappe Lab', 'app_name': 'frappe_lab'},
>  {'name': 'Automation', 'app_name': 'frappe'},
>  {'name': 'Social', 'app_name': 'frappe'},
>  {'name': 'Contacts', 'app_name': 'frappe'},
>  {'name': 'Printing', 'app_name': 'frappe'},
>  {'name': 'Integrations', 'app_name': 'frappe'},
>  {'name': 'Desk', 'app_name': 'frappe'},
>  {'name': 'Geo', 'app_name': 'frappe'},
>  {'name': 'Custom', 'app_name': 'frappe'},
>  {'name': 'Email', 'app_name': 'frappe'},
>  {'name': 'Workflow', 'app_name': 'frappe'},
>  {'name': 'Website', 'app_name': 'frappe'},
>  {'name': 'Core', 'app_name': 'frappe'}]
> ```

Dobro to je način organizacije delova app na način shodan korisniku, a ne samo po app.

Module Def je logička organizacija funkcionalnosti unutar aplikacije.

Ne služi Python-u.
Ne služi Git-u.  
Ne služi Bench-u.  

Služi Frappe-u i korisniku."

Na primer:

```txt
ERPNext (app)

Accounts (Module Def)
    Journal Entry 
    Payment Entry
    GL Entry

Selling (Module Def)
    Sales Order
    Sales Invoice
    Customer

Buying (Module Def)
    Purchase Order
    Supplier
```

Korisnik ne razmišlja: "Ovo je u aplikaciji ERPNext." Nego razmišlja: "Radim u modulu Selling." To je organizacija **poslovnih funkcionalnosti**.

I pogledaj kako se lepo slažu podaci koje si izvukao:

| Module Def | App Name |
| ------ | --- |
| Frappe Lab | frappe_lab |
| Core | frappe |
| Desk | frappe |
| Email | frappe |
| Website | frappe |

Svaki Module Def zna:

* kome pripada (`app_name`)
* kako se zove (`name`)

To je razlog zašto se pravi preko **site**-a i zato što je Module Def običan DocType.

Iskreno, trebalo je i meni dosta vremena da prestanem da razmišljam u terminima Django-a ili Flask-a.

U Frappe-u skoro sve važno je **DocType**.

Čak i:

* Module Def
* DocType
* Workspace
* Role
* User
* Print Format

...sve su to zapisi u bazi.

Framework se onda na osnovu tih zapisa organizuje i generiše ponašanje.

Zato se ponekad čini da je "sve u bazi", a fajlovi na disku samo prate ono što je definisano u DocType-ovima.

</br>

#### Proverimo Module Def

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

> [!Note]
> **AI Zahtev**
> Pošalji mi izlaz:
>
> ```bash
> tree -L 5 ~/frappe-bench/apps/frappe_lab/frappe_lab
> ```

Hoću da vidiš : "šta je Frappe upravo napravio na disku".

To će biti naš prvi pravi susret sa vezom: "DocType u bazi ↔ Python fajlovi ↔ JSON definicija".

I odatle krećemo na lifecycle eksperimente. Mislim da će od tog trenutka sve postati mnogo konkretnije.

> [!Info]
> Izlaz
>
> ```sh
> tree -L 5 ~/frappe-bench/apps/frappe_lab/frappe_lab
> ```
>
> ```sh
> /home/radosav/frappe-bench/apps/frappe_lab/frappe_lab
> ├── config
> │   └── __init__.py
> ├── frappe_lab
> │   ├── doctype
> │   │   ├── book
> │   │   │   ├── book.js
> │   │   │   ├── book.json
> │   │   │   ├── book.py
> │   │   │   ├── __init__.py
> │   │   │   ├── __pycache__
> │   │   │   │   ├── book.cpython-312.pyc
> │   │   │   │   └── __init__.cpython-312.pyc
> │   │   │   └── test_book.py
> │   │   ├── __init__.py
> │   │   └── __pycache__
> │   │       └── __init__.cpython-312.pyc
> │   ├── __init__.py
> │   └── __pycache__
> │       └── __init__.cpython-312.pyc
> ├── hooks.py
> ├── __init__.py
> ├── modules.txt
> ├── patches
> │   └── __init__.py
> ├── patches.txt
> ├── public
> │   ├── css
> │   └── js
> ├── __pycache__
> │   ├── hooks.cpython-312.pyc
> │   └── __init__.cpython-312.pyc
> ├── templates
> │   ├── includes
> │   ├── __init__.py
> │   └── pages
> │       └── __init__.py
> └── www
> 
> 17 directories, 21 files
> ```

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

> [!Note]
> **AI Zahtev**
>
> ```bash
> nano ~/frappe-bench/apps/frappe_lab/frappe_lab/frappe_lab/doctype/book/book.py
> ```

ili editor koji koristiš.

Pošalji njegov sadržaj.

Hoću da vidimo šta je Frappe automatski generisao pre nego što bilo šta menjamo.

To će biti prvi fajl koji ćemo "rastaviti" i povezati sa svim onim što smo čitali u `document.py`. Mislim da će ti biti poznat već na prvi pogled.

> [!Info]
> Izlaz  
>
> ```bash
> nano ~/frappe-bench/apps/frappe_lab/frappe_lab/frappe_lab/doctype/book/book.py
> ```
>
> ```py
> # Copyright (c) 2026, rrad and contributors
> # For license information, please see license.txt
> 
> # import frappe
> from frappe.model.document import Document
> 
> 
> class Book(Document):
>         pass
> ```

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

E pa sada **`self`** u svim onim metodama koje smo čitali jeste instanca `Book`.

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

izvršava se isti `Document.insert()` koji smo detaljno analizirali, samo što će usput pozivati metode iz tvoje klase `Book`.

[Sadržaj][00]

[00]: 00%20Učenje%20Frape%20frameworka.md
