
# Učenje Frappe Framework

[Sadržaj][00]

## 09 Tok izvršenja u Frappeu

Mislim da je vreme da prestanemo da gledamo pomoćne metode i da krenemo da pratimo jedan konkretan tok izvršavanja.

**Gde se nalazi `get_doc`?**

Ja bih rekao: `frappe/__init__.py`

Zašto? Zato što je korisnik poziva kao:

```python
frappe.get_doc(...)
```

To znači da mora biti izložena na nivou paketa `frappe`.

**AI Zahtev**:  

Pošalji:

```bash
grep -n "def get_doc" ~/frappe-bench/apps/frappe/frappe/__init__.py
```
  
Nećemo odmah čitati telo funkcije. Hoću prvo da vidimo koliko je velika.

Zašto? U velikim framework-ovima često postoji obrazac:
  
```python
def get_doc(...):
    return model.get_doc(...)
```

ili

```python
def get_doc(...):
    return local.get_doc(...)
```

Drugim rečima, javni API je često veoma tanak sloj koji prosleđuje posao dalje.

Ako je tako i u Frappe-u, onda ćemo odmah pratiti gde nas vodi.

Usput jedna zanimljiva stvar, već smo videli:

```python
doc.meta
```

poziva:

```python
frappe.get_meta(...)
```

Sada ćemo istražiti:

```python
frappe.get_doc(...)
```

Mene baš zanima da li će se i ovde ponoviti isti obrazac:

```py
frappe.get_doc() -> model.get_doc()
```

ili postoji još neki sloj između.

To nećemo nagađati – proverićemo u kodu.

**Izlaz**:

```sh
grep -n "def get_doc" ~/frappe-bench/apps/frappe/frappe/__init__.py
```

```py
1223:def get_document_cache_key(doctype: str, name: str):
1269:def get_doc(document: "Document", /) -> "Document":
1274:def get_doc(doctype: str, /) -> _SingleDocument:
1280:def get_doc(doctype: str, name: str, /, *, for_update: bool | None = None) -> "Document":
1286:def get_doc(**kwargs: dict) -> "_NewDocument":
1293:def get_doc(documentdict: dict) -> "_NewDocument":
1299:def get_doc(*args, **kwargs):
1571:def get_doc_hooks():
2299:def get_doctype_app(doctype):
```

I sada smo naišli na nešto veoma zanimljivo. Ovo je jedan od razloga zašto volim da se framework čita polako.

Pogledaj ovo:

```text
1269:def get_doc(document: "Document", /) -> "Document":
1274:def get_doc(doctype: str, /) -> _SingleDocument:
1280:def get_doc(doctype: str, name: str, /, *, for_update: bool | None = None) -> "Document":
1286:def get_doc(**kwargs: dict) -> "_NewDocument":
1293:def get_doc(documentdict: dict) -> "_NewDocument":
1299:def get_doc(*args, **kwargs):
```

Prva reakcija je obično: "Kako može Python da ima šest funkcija sa istim imenom?". Odgovor je: ne može. Ovo nisu različite implementacije. One služe za **type hinting**.

Pogledaj pažljivo poslednju:

```python
def get_doc(*args, **kwargs):
```

To je jedina prava implementacija.

Sve iznad su opisi za:

- IDE (PyCharm, VS Code),
- type checker-e (`mypy`, `pyright`),
- autocomplete,
- dokumentaciju.

To je obrazac koji se često koristi u modernom Pythonu.

Dakle, ovo nije "preopterećenje funkcija" kao u C++ ili Javi, već način da se jednom implementacijom opiše više načina korišćenja.

Ovo je lep primer zašto ne treba stati na prvi pogled, da smo samo preleteli fajl, mogli bismo pogrešno zaključiti da Python podržava overload.

A zapravo podržava samo:

- jednu implementaciju,
- više deklaracija za potrebe tipova.

Sada nas zanima samo jedna stvar

Hoću da vidimo pravu implementaciju.

**AI Zahtev**:  

Pošalji nekoliko linija oko nje:

```bash
sed -n '1299, 1315p' ~/frappe-bench/apps/frappe/frappe/__init__.py
```

Ako se poziv nastavlja dalje, slobodno i malo više, recimo:

```bash
sed -n '1299,1325p' ~/frappe-bench/apps/frappe/frappe/__init__.py
```

Šta očekujem? Imam hipotezu. Mislim da će implementacija biti veoma kratka.

Nešto u stilu:

```python
def get_doc(*args, **kwargs):
    import frappe.model.document
    return frappe.model.document.get_doc(*args, **kwargs)
```

ili nešto veoma slično.

Ako se to potvrdi, naučićemo još jednu važnu stvar o Frappe-u:

- `frappe.__init__.py` je javni API,
- prava logika je smeštena u drugim modulima.

To je obrazac koji ćeš verovatno viđati na mnogo mesta u Frappe-u.

**Izlaz**:

```sh
sed -n '1299,1325p' ~/frappe-bench/apps/frappe/frappe/__init__.py
```

```py
def get_doc(*args, **kwargs):
  """Return a `frappe.model.document.Document` object of the given type and name.

  :param arg1: DocType name as string **or** document JSON.
  :param arg2: [optional] Document name as string.

  Examples:
            # insert a new document
          todo = frappe.get_doc({"doctype":"ToDo", "description": "test"})
          todo.insert()
            # open an existing document
          todo = frappe.get_doc("ToDo", "TD0001")
  """
  import frappe.model.document
  return frappe.model.document.get_doc(*args, **kwargs)
```

I sada mogu da ti pokažem jedan obrazac koji se ponavlja kroz ceo Frappe.

Pogledaj koliko je funkcija mala:

```python
def get_doc(*args, **kwargs):
    ...
    import frappe.model.document

    return frappe.model.document.get_doc(*args, **kwargs)
```

To je praktično sve.

Dakle... `frappe.get_doc()` ne radi gotovo ništa.

Njegov posao je samo da kaže: "Idi u `frappe.model.document` i tamo odradi pravi posao."

Zašto ovako?. Ovo je veoma lepo projektovanje. Zamisli da nije ovako. Morali bismo da pišemo:
  
```python
from frappe.model.document import get_doc

doc = get_doc(...)
```
  
ili

```python
import frappe.model.document

doc = frappe.model.document.get_doc(...)
```

Umesto toga možeš jednostavno:

```python
doc = frappe.get_doc(...)
```

To je mnogo prijatniji javni API.

</br>

**Facade**:

Ne znam da li si ranije nailazio na taj termin. U objektno orijentisanom projektovanju postoji obrazac:

```txt
Korisnik ->
Facade ->
Komplikovan sistem
```

Ovde je:

```txt
Programer ->
frappe.get_doc() ->
frappe.model.document.get_doc() ->
Document ->
Database
```

Programer vidi samo jednu funkciju.

Sve ostalo je sakriveno.

A zašto `import` nije na vrhu fajla?

Ovo je još zanimljivije.

Pogledaj:

```python
def get_doc(...):
    import frappe.model.document
```

Većina Python programa radi:

```python
import ...
import ...
import ...
```

na početku fajla.

Ovde nije.

Zašto?

</br>

- Hipoteza 1 — Izbegavanje kružnih zavisnosti

  Zamisli:

  ```sh
  __init__.py ->
  document.py ->
  base_document.py ->
  frappe ->
  __init__.py
  ```

  I odjednom:

  ```py
  ImportError
  ```

  Lokalni `import` često rešava ovakve probleme.

</br>

- Hipoteza 2 — Brže pokretanje
  
    Ako nikad ne pozoveš `get_doc()`, nikad se neće importovati `document.py`.
    To može malo ubrzati startovanje.

  </br>

- Koja hipoteza je tačna?

  Ne znamo.

  I nećemo nagađati.

  To je još jedna stvar koju ćemo proveriti kad budemo bolje upoznali zavisnosti između modula.

Ono što smo upravo naučili:

Već sada imamo malu mapu.

```txt
Program ->
frappe.get_doc() ->
frappe.model.document.get_doc() ->
??? ->
Document
```

Sada treba da otkrijemo ono **???**
  
Sada više nema razloga da ostajemo u `__init__.py`.

Pravi posao je ovde:

```sh
frappe/model/document.py
```

Hajde da pronađemo sledeću funkciju.

**AI Zahtev**:  

Pošalji:

```bash
grep -n "^def get_doc" ~/frappe-bench/apps/frappe/frappe/model/document.py
```

</br>

Postoji jedna stvar koju sam namerno nisam unapred rekao.

Na osnovu onoga što smo do sada videli (`frappe.get_doc()` samo prosleđuje poziv), očekujem da ćemo u `document.py` naići na nešto malo neobično. Ako se moja pretpostavka potvrdi, to će nam objasniti kako Frappe uspeva da podrži sve ove pozive:

```python
frappe.get_doc("Customer", "CUST-0001")
frappe.get_doc({"doctype": "Customer"})
frappe.get_doc(existing_doc)
frappe.get_doc("System Settings")
```

...a da ipak ima jednu javnu funkciju `get_doc()`.

Pošalji izlaz, pa nastavljamo da pratimo tok izvršavanja korak po korak.
  
**Izlaz**:  

Evo izlaza

```sh
grep -n "^def get_doc" ~/frappe-bench/apps/frappe/frappe/model/document.py
```

```py
36:def get_doc(*args, **kwargs):
```

Do sada smo radili ovako:

```py
frappe.get_doc() -> frappe.model.document.get_doc()
```

Sada više nije dovoljno da znamo da postoji `get_doc()`.

Moramo videti šta radi.

**AI Zahtev**:  

Pošalji prvih dvadesetak-tridesetak linija te funkcije:

```bash
sed -n '36,70p' ~/frappe-bench/apps/frappe/frappe/model/document.py
```

Ako se vidi da je funkcija duža, možemo proširiti na:

```bash
sed -n '36,90p' ~/frappe-bench/apps/frappe/frappe/model/document.py
```

Zašto baš toliko? Zato što očekujem da ćemo vrlo brzo naići na neku od sledećih stvari:

- `match` (Python 3.10+),
- `if isinstance(...)`,
- `singledispatch`,
- ili neku drugu tehniku za "dispečiranje" različitih tipova argumenata.

To će nam objasniti kako jedna funkcija uspeva da podrži toliko različitih načina poziva.
  
Evo izlaza

```sh
sed -n '36,90p' ~/frappe-bench/apps/frappe/frappe/model/document.py
```

```py
def get_doc(*args, **kwargs):
  """returns a frappe.model.Document object.

  :param arg1: Document dict or DocType name.
  :param arg2: [optional] document name.
  :param for_update: [optional] select document for update.

  There are multiple ways to call `get_doc`

          # will fetch the latest user object (with child table) from the database
          user = get_doc("User", "test@example.com")

          # create a new object
          user = get_doc({
                  "doctype":"User"
                  "email_id": "test@example.com",
                  "roles: [
                          {"role": "System Manager"}
                  ]
          })

          # create new object with keyword arguments
          user = get_doc(doctype='User', email_id='test@example.com')

          # select a document for update
          user = get_doc("User", "test@example.com", for_update=True)
  """
  if args:
    if isinstance(args[0], BaseDocument):
      # already a document
      return args[0]
    elif isinstance(args[0], str):
      doctype = args[0]

    elif isinstance(args[0], dict):
      # passed a dict
      kwargs = args[0]

    else:
      raise ValueError("First non keyword argument must be a string or dict")

  if len(args) < 2 and kwargs:
    if "doctype" in kwargs:
      doctype = kwargs["doctype"]
    else:
      raise ValueError('"doctype" is a required key')

  controller = get_controller(doctype)
  if controller:
    return controller(*args, **kwargs)

  raise ImportError(doctype)
```

Sada smo konačno stigli do mesta gde Frappe počinje da pokazuje svoju filozofiju. I iskreno, ovo je mnogo elegantnije nego što mnogi očekuju.

Hajde da ga pročitamo kao da smo autori Frappe-a.

Mene je odmah iznenadilo nešto.

`get_doc()` uopšte ne učitava dokument iz baze.

Nema:

- `SELECT ...`.
- `frappe.db.sql(...)`,
- `db.get(...)`.

Ništa.  

To znači da ova funkcija ima sasvim drugu ulogu. Šta ona zapravo radi?

Ja bih je nazvao:

</br>

**Fabrika (Factory)**:

Ona ne pravi SQL.  Ona odlučuje: "Koju klasu treba napraviti?"

Hajdemo redom

</br>

- **Prvi slučaj**

  ```py
  if isinstance(args[0], BaseDocument):
    return args[0]
  ```

  Ako joj pošalješ već gotov dokument:

  ```py
  doc = frappe.get_doc(existing_doc)
  ˙˙˙
  ```

  ona kaže: "Već imaš Document." i vrati ga.

  To je elegantno.

</br>

- **Drugi slučaj**
  
  ```py
  elif isinstance(args[0], str):
      doctype = args[0]
  ```

  Ovo je najčešći poziv.

  Na primer:

  ```py
  frappe.get_doc("Customer", "CUST-0001")
  ```

  Ovde samo zapamti:

  ```py
  doctype = "Customer"
  ```

  Još ništa nije učitano.

</br>

- **Treći slučaj**
  
  ```py
  elif isinstance(args[0], dict):
  ```

  Ovde praviš novi dokument.

  Na primer:

  ```py
  frappe.get_doc({
      "doctype": "Customer",
      "customer_name": "Marko"
  })
  ```

  Dakle ista funkcija podržava i:
  - otvaranje postojećeg,
  - pravljenje novog.
  A onda... Dolazimo do najvažnije linije.

  ```py
  controller = get_controller(doctype)
  ```
  
  Po meni, ovo je ključ cele funkcije.
  
  Jer odjednom više nije važno:

  - da li je dokument novi,
  - da li dolazi iz baze,
  - da li je Customer,
  - da li je Item.

  Sve se svodi na jedno pitanje: Ko upravlja ovim DocType-om?
  
</br>

### Controller
  
"Controller" je zapravo Python klasa koja predstavlja taj DocType.

Na primer:

```txt
Customer
Customer(Document)
```

Ako takva klasa postoji. Ako ne... videćemo šta Frappe radi.

Pogledaj završetak funkcije.

```py  
return controller(*args, **kwargs)
```

Ovo je fantastično. Ne piše:

- `Customer(...)`.
- `User(...)`.
- `Task(...)`.

Nego: `controller(...)`.

Drugim rečima:

```py
    get_doc() ->
    get_controller() ->
    ??? ->
    pozovi klasu
```

To je čisti **Factory Pattern**.

</br>

**Sada jedna veoma zanimljiva stvar**:
  
Sećaš se kada smo govorili:

```txt
    DocType ->
    Meta ->
    Document
```

Sada možemo malo da dopunimo dijagram.

```txt
    frappe.get_doc() ->
    document.get_doc() ->
    get_controller() ->
    ??? ->
    Document(...)
```

Odgovor na ono veliko ??? krije se upravo u `get_controller()`.
  
I evo pitanja koje sada možemo postaviti: "Ako napišem: `frappe.get_doc("Customer")` kako `get_controller()` zna da vrati baš: "Customer" a ne: "User" ili: "Task".

I još važnije... Šta ako "Customer.py" uopšte ne postoji?

To je upravo ono po čemu je Frappe poseban.
  
Mislim da je sledeći korak potpuno prirodan. Nemojmo još otvarati Document.  
Nemojmo još gledati SQL. Pratimo ovu jednu liniju:

```py
controller = get_controller(doctype)
```

Pošto smo je već videli u `base_document.py`, sada ćemo konačno razumeti njenu ulogu.

> [!Note]
> **AI Zahtev**  
>
> Pošalji:
>
> ```sh
> grep -A 50 "^def get_controller" ~/frappe-bench/apps/frappe/frappe/model/
> base_document.py
> ```

Nemoj se uplašiti ako bude malo duža. To je verovatno jedna od najvažnijih funkcija u celom mehanizmu Frappe-a, jer ona predstavlja most između naziva DocType-a (običnog stringa poput "Customer") i Python klase koja će na kraju biti instancirana.
  
> [!Info] Izlaz
>
> ```sh
> grep -A 50 "^def get_controller" ~/frappe-bench/apps/frappe/frappe/model/
> base_document.py
> ```
>
>```py
>def get_controller(doctype):
>  """
>  Returns the locally cached **class** object of the given DocType.
>  For `custom` type, returns `frappe.model.document.Document`.
>
>  :param doctype: DocType name as string.
>  """
>
>  if frappe.local.dev_server or frappe.flags.in_migrate:
>    return import_controller(doctype)
>
>  site_controllers = frappe.controllers.setdefault(frappe.local.site, {})
>  if doctype not in site_controllers:
>    site_controllers[doctype] = import_controller(doctype)
>
>  return site_controllers[doctype]
>```

I moram da kažem – ovo je jedna od najlepših funkcija koje smo do sada videli. Nije velika, ali otkriva dosta o arhitekturi Frappe-a.

</br>

- **Prvo što upada u oči**

  Pogledaj docstring:
  
  ```python
  Returns the locally cached **class** object of the given DocType.
  ```
  
  Obrati pažnju na jednu reč: **class**.
  
  Ne kaže:
  
  - object
  - document
  - record
  
  nego: **class object**.
  
  To znači da ova funkcija ne vraća Customer objekat.

  Ona vraća:
  
  ```py
  Customer
  ```
  
  odnosno sam objekat klase. To je ogromna razlika.
  
  Dakle... Do sada smo imali:
  
  ```py
  controller = get_controller("Customer")
  ```
  
  Sada znamo da je rezultat nešto poput:
  
  ```py
  controller == Customer
  ```
  
  ili možda:
  
  ```py
  controller == Document # Zato što je Customer nasleđen iz Document
  ```
  
  zavisno od DocType-a.
  
  Posle toga:
  
  ```py
  return controller(*args, **kwargs)
  ```
  
  znači:
  
  ```py
  Customer(*args, **kwargs)
  ```
  
  ili
  
  ```py
  Document(*args, **kwargs)
  ```
  
</br>

- **Druga zanimljiva stvar**

  Pogledaj ovo:
  
  ```python
  if frappe.local.dev_server or frappe.flags.in_migrate:
      return import_controller(doctype)
  ```
  
  Ovde se vidi da Frappe razlikuje dva režima rada.

  - **Normalan rad**

    Koristi keš.

  - **Development**

    Ne koristi keš.

    Zašto? Jer u development-u menjaš kod. Ako bi klasa ostala keširana...  menjaš fajl... a Frappe i dalje koristi staru klasu. To bi bilo veoma frustrirajuće.

</br>

- **Treća stvar**

  Ovaj deo mi se posebno sviđa.
  
  ```python
  site_controllers = frappe.controllers.setdefault(
      frappe.local.site,
      {}
  )
  ```
  
  Ovde prvi put vidimo nešto veoma važno.
  
  Frappe nije zamišljen kao:

  ```txt
  jedan server
  jedna baza
  ```

  nego kao:

  ```txt
  jedan Bench
      │
      ├── site1.local
      ├── site2.local
      ├── companyA
      └── companyB
  ```

  Dakle... **keš nije globalan.**. Keš je: **po sajtu**.

  To ima potpuno smisla. Jer dva sajta mogu imati različite aplikacije.

  Evo kako ja to zamišljam:
  
  Negde u memoriji postoji nešto ovako:
  
  ```python
  controllers = {
      "site1.local": {
          "Customer": Customer,
          "Item": Item,
          "User": User,
      },
      "companyB": {
          "Customer": Customer,
          "Vehicle": Vehicle,
      }
  }
  ```

Naravno, pojednostavljeno, ali mislim da je mentalni model dobar.
  
</br>

- **Četvrta stvar**

  Pogledaj ovo:
  
  ```python
  if doctype not in site_controllers:
  ```
  
  Dakle prvi put:
  - Customer
  - učitaj klasu
  - stavi u keš
  
  Drugi put:
  - Customer
  - vrati iz keša
  
  Bez ponovnog importa.

</br>

- **A sada dolazimo do najveće misterije**

  Videli smo:
  
  ```python
  return import_controller(doctype)
  ```
  
  To je sada "crna kutija".
  
  Tu se krije odgovor na pitanje: **Kako string `"Customer"` postaje Python klasa?**
  
  To je upravo ono što smo tražili još od početka.

</br>

Voleo bih da obratiš pažnju na nešto.

Pre desetak koraka imali smo dijagram:

```txt
    get_doc() -> 
    Document
```

Sada je on postao mnogo precizniji.

```txt
    get_doc() ->
    get_controller() ->
    import_controller() ->
    Python class ->
    Document instance
```

Primećuješ? Naš mentalni model postaje sve detaljniji. Ne zato što učimo napamet funkcije, nego zato što svaku "crnu kutiju" polako otvaramo.

Pogledaj kako je funkcija napisana. Nema 200 linija. Nema ogromnog `if`. Nema SQL-a. Radi samo jednu stvar: "Pronađi odgovarajuću klasu i keširaj je."

To je odličan primer principa **Single Responsibility Principle (SRP)** iz SOLID-a.

Zato je lako razumeti šta radi i lako je testirati.

Mislim da smo sada došli do mesta gde treba otvoriti poslednju "crnu kutiju":

```python
import_controller(doctype)
```

Ali... Tu bih napravio malu promenu u našem načinu rada.

Do sada smo pratili izvršavanje. Sada bih voleo da na trenutak stanemo i nacrtamo kompletnu arhitekturu onoga što smo do sada otkrili, pre nego što uđemo u `import_controller()`.

Zašto? Zato što smo već sakupili dovoljno delova slagalice da možemo da vidimo celu sliku:

```txt
    Program ->
    frappe.get_doc(...) ->
    frappe.model.document.get_doc(...) ->
    get_controller(doctype) ->
    import_controller(doctype) ->
    Python klasa ->
    Instanca Document
```

Mislim da je to dobar trenutak da zastanemo, proverimo da li nam je model jasan i tek onda otvorimo `import_controller()`, jer sumnjam da će baš tamo početi priča o tome:

- kako Frappe pronalazi Python modul za dati DocType,
- kako rade standardni i custom DocType-ovi i
- kako se uključuju aplikacije (`apps`).

Mislim da smo napravili dobar izbor što **ne jurimo**. Da smo krenuli "kako se pravi DocType", već bismo napravili deset aplikacija, a ne bismo razumeli zašto jedna jedina linija:

```python
doc = frappe.get_doc(...)
```

pokrene pola framework-a.

Sada već imamo kontinuitet, pa idemo dalje.

```python
def get_doc(*args, **kwargs):

    ...

    controller = get_controller(doctype)

    return controller(*args, **kwargs)
```

i zatim:

```python
def get_controller(doctype):

    ...

    return import_controller(doctype)
```

Dakle sada ulazimo u poslednju "crnu kutiju".

**Šta očekujem od `import_controller()`?**  

Očekujem da radi otprilike ovo:

```sh
DocType ->
Kojoj aplikaciji pripada? ->
Koji modul? ->
Import Python modula ->
Nađi klasu ->
Vrati klasu
```

**Hajde sada da proverimo**  

**AI Zahtev**:  

Pošalji izlaz:

```bash
grep -A 80 "^def import_controller" ~/frappe-bench/apps/frappe/frappe/model/
base_document.py
```

**Izlaz**:  

```sh
grep -A 80 "^def import_controller" ~/frappe-bench/apps/frappe/frappe/model/
base_document.py
```

```py
def import_controller(doctype):
  from frappe.model.document import Document
  from frappe.utils.nestedset import NestedSet

  module_name = "Core"
  if doctype not in DOCTYPES_FOR_DOCTYPE:
    doctype_info = frappe.db.get_value("DocType", doctype, ("module", 
       "custom", "is_tree"), as_dict=True)
    if doctype_info:
      if doctype_info.custom:
        return NestedSet if doctype_info.is_tree else Document
      module_name = doctype_info.module

  module_path = None
  class_overrides = frappe.get_hooks("override_doctype_class")
  if class_overrides and class_overrides.get(doctype):
    import_path = class_overrides[doctype][-1]
    module_path, classname = import_path.rsplit(".", 1)
    module = frappe.get_module(module_path)

  else:
    module = load_doctype_module(doctype, module_name)
    classname = doctype.replace(" ", "").replace("-", "")
```

- **Prvi utisak**  
  Moram odmah da kažem jednu stvar. Moja hipoteza od prošlog puta bila je skoro tačna, ali ne potpuno.  
  Ja sam očekivao nešto poput:
  
  ```text
  DocType ->
  nađi modul ->
  import ->
  vrati klasu
  ```
  
  Ali Frappe radi još nekoliko veoma zanimljivih stvari usput.
  
</br>

- **Prvo se definišu podrazumevane klase**

  ```python
  from frappe.model.document import Document
  from frappe.utils.nestedset import NestedSet
  ```
  
  Već na početku vidimo dve moguće "osnovne" klase.
  
  To znači da Frappe već zna da postoje najmanje dve porodice DocType-ova:
  
  ```sh
  Document
  NestedSet
  ```
  
  `NestedSet` ćemo ostaviti za kasnije (koristi se za hijerarhijske strukture poput stabala), ali je zanimljivo da se već ovde pojavljuje.

</br>

- **Prvi odlazak u bazu**  

  Ovo je prvi put da naš tok izvršavanja zaista odlazi u bazu.
  
  ```python
  doctype_info = frappe.db.get_value(
      "DocType",
      doctype,
      ("module", "custom", "is_tree"),
      as_dict=True
  )
  ```

  E ovo je veoma važno. Primeti šta čita.
  
  Ne čita:
  
  - Customer
  - User
  - Sales Invoice
  
  nego čita:
  
  ```text
  DocType
  ```
  
  Drugim rečima: Prvo se učitavaju metapodaci o DocType-u.
  
  To se savršeno uklapa u ono što smo do sada zaključili.

- **Najveće iznenađenje**  
  
  Pogledaj ovo:
  
  ```python
  if doctype_info.custom:
      return NestedSet if doctype_info.is_tree else Document
  ```
  
  Ovo potvrđuje jednu veoma važnu stvar.

  - **Custom DocType**  
    Frappe ne traži Python klasu.

    Odmah kaže:

    ```txt
    custom DocType -> Document
    ```

    To znači da je za običan custom DocType sasvim dovoljno:

    - definicija u DocType,
    - definicija polja,
    - metapodaci.

    Nema potrebe za `customer.py`.

  - **Standardni DocType**

    Ako nije custom:

    ```python
    module_name = doctype_info.module
    ```

    Sada već ima dovoljno informacija da kaže:

    ```txt
    Customer -> Module = Selling -> User -> Module = Core
    ```

  - **Override DocType**

    Dolazimo do nečega što mi se posebno dopada.

    ```python
    class_overrides = frappe.get_hooks("override_doctype_class")
    ```

    Ovo pokazuje koliko je Frappe otvoren za proširenja. Ne kaže: "Koristi ovu klasu." Nego prvo pita: "Da li je neko u hook-ovima rekao da želi drugu klasu?"

    To znači da aplikacija može da zameni implementaciju nekog DocType-a bez menjanja izvornog koda Frappe-a.

    To je veoma moćan mehanizam.

- **Tek sada dolazi import**  
  
  Ako nema override-a:
  
  ```py
  module = load_doctype_module(...)
  ```
  
  Dakle moja prethodna hipoteza:
  
  ```py
  import modul
  ```
  
  jeste tačna. Ali nije prvi korak.

</br>

**Kako pronalazi klasu?**  
Veoma jednostavno.

```python
classname = doctype.replace(" ", "").replace("-", "")
```

Dakle:

```txt
Sales Invoice
```

postaje

```txt
SalesInvoice
```

a zatim:

```python
getattr(module, classname)
```

To je potpuno standardan Python.

</br>

**Provera**:

Još jedna stvar koja mi se dopada.

```python
issubclass(class_, BaseDocument)
```

Drugim rečima, ako napišeš:

```python
class Customer:
    ...
```

Frappe će reći: Ne.  

Mora da bude:

```python
class Customer(Document):
```

ili neka druga izvedena klasa `BaseDocument`.

To čuva konzistentnost celog framework-a.

**Sada dolazimo do najzanimljivijeg dela**:

Ako pogledamo ceo tok koji smo do sada ispratili, to izgleda ovako:

```text
frappe.get_doc(...) ->
document.get_doc(...) ->
get_controller(doctype) ->
(import iz keša ili import_controller) ->
DocType tabela  -> prvi odlazak u bazu ->
    | -> custom -> Document          |\
    |-> hook overide? -> druga klasa |  ->
    |-> load_doctype_module          |/
getattr(...) -> Python klasa -> controller(*args)
```

Po mom mišljenju, ovo je jedan od najvažnijih dijagrama koje smo do sada napravili.

Frappe kaže:

- za **custom DocType** → meta model je dovoljan (`Document`),
- za **standardne ili naprednije DocType-ove** → možeš dodati Python kontroler,
- a čak možeš i **zameniti kontroler** preko hook-ova.

To nije bolji ili lošiji pristup — to je drugačiji kompromis između fleksibilnosti i jednostavnosti.

Mislim da smo završili jednu veliku celinu.

Prešli smo put od javnog API-ja do trenutka kada Frappe dobije odgovarajuću Python klasu.

Po mom mišljenju, sada ne treba odmah da skočimo na `save()`.

[Sadržaj][00]

[00]: 00%20Učenje%20Frape%20frameworka.md
