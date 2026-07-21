# Učenje Frappe Framework

[Sadržaj][00]

## 10 Instaciranje klase DocTypea

Ja bih prvo voleo da pratimo šta se dešava kada se ta klasa instancira, odnosno kako `Document.__init__()` (i ono što nasleđuje iz `BaseDocument`) pretvara prosleđene argumente u pravi objekat. Time ćemo zatvoriti ceo životni ciklus `frappe.get_doc()` pre nego što pređemo na čuvanje u bazu. Mislim da će to biti prirodan završetak ove prve velike teme o nastanku dokumenta.

Mislim da sada ulazimo u deo koji će ti se posebno dopasti, jer više ne pričamo o "pronalaženju klase", nego o nastanku živog Document objekta.

I ovde bih opet voleo da nastavimo našim pravilom:

**Pratimo tok izvršavanja, ne fajlove.**

Došli smo do ove linije:

```python
controller(*args, **kwargs)
```

Pretpostavimo da je `controller` zapravo:

```python
Customer
```

koji nasleđuje:

```text
Customer ->
Document ->
BaseDocument
```

Pošto `Document` nema svoj `__init__()`, Python će pozvati:

```python
BaseDocument.__init__()
```

A njega već imamo ispred sebe:

```python
def __init__(self, d):
    if d.get("doctype"):
        self.doctype = d["doctype"]

    self._table_fieldnames = {
        df.fieldname for df in self._get_table_fields()
    }

    self.update(d)
    self.dont_update_if_missing = []

    if hasattr(self, "__setup__"):
        self.__setup__()
```

</br>

**Hajde da ga čitamo veoma pažljivo**:

</br>

- **Prva linija**

  ```python
  if d.get("doctype"):
      self.doctype = d["doctype"]
  ```
  
  Ovde se dešava nešto sasvim očekivano.
  
  Ako si napravio:
  
  ```python
  frappe.get_doc({
      "doctype": "Customer",
      "customer_name": "Marko"
  })
  ```
  
  objekat odmah dobija:
  
  ```python
  self.doctype = "Customer"
  ```
  
  To je važno jer smo već videli:
  
  ```python
  self.meta
  ```
  
  radi:
  
  ```python
  frappe.get_meta(self.doctype)
  ```
  
  Dakle doctype je identitet objekta.

</br>

- **Čitanje tables polja**

  ```python
  self._table_fieldnames = {
      df.fieldname
      for df in self._get_table_fields()
  }
  ```

  Ovde se ne čitaju obična polja.
  
  Čitaju se **Table** polja.
  
  Drugim rečima, Frappe odmah želi da zna: Koja polja predstavljaju child tabele?
  
  Na primer:
  
  ```text
  Sales Invoice
  
  items
  taxes
  payments
  ```
  
  nisu obična polja.
  
  To su liste drugih dokumenata.
  
  Mislim da zato ovo radi odmah u konstruktoru.

</br>

- **Po meni najvažnija linija**

  ```python
  self.update(d)
  ```
  
  Ovde sam skoro siguran da počinje pravo "punjenje" objekta.
  
  Drugim rečima:
  
  iz ovog:
  
  ```python
  {
      "doctype": "Customer",
      "customer_name": "Marko"
  }
  ```
  
  nastaje:
  
  ```python
  doc.customer_name
  ```
  
  ali... neću da nagađam.

  Hoću da vidimo kako `update()` radi.

  Ovo je zanimljivo

  ```python
  self.dont_update_if_missing = []
  ```
  
  Za sada ne znamo čemu služi.
  
  Ne bih još dirao.

- **Poslednja linija**

  ```python
  if hasattr(self, "__setup__"):
      self.__setup__()
  ```
  
  Ovo mi se mnogo dopada.
  
  To znači: ako kontroler želi dodatnu inicijalizaciju... ne mora da dira `__init__()`. Samo napiše:
  
  ```python
  def __setup__(self):
      ...
  ```
  
  To je lep obrazac.

  Po meni se sada pojavljuje nova "crna kutija".
  
  ```python
  self.update(d)
  ```
  
  Mislim da se upravo tu:
  
  - pune atributi,
  - prave child dokumenti,
  - pretvaraju dict-ovi u objekte,
  - možda rade i konverzije tipova.
  
  Ako je moja pretpostavka tačna... onda je `update()` jedna od najvažnijih metoda u `BaseDocument`.

</br>

Nemojmo odmah otvarati 200 linija.

> [!Note] AI Zahtev
>
> Pošalji samo ovu metodu:
>
> ```bash
> grep -A 60 "^ def update" ~/frappe-bench/apps/frappe/frappe/model/
> base_document.py
> ```
>
> ili ako `grep` ne uhvati zbog razmaka, onda:
>
> ```bash
> grep -n "def update" ~/frappe-bench/apps/frappe/frappe/model/base_document.py
> ```

pa ćemo preko `sed` otvoriti tačan opseg.

Počinjem da uočavam jednu zanimljivu osobinu Frappe-a.

Do sada skoro svaka funkcija koju smo otvorili radi **jednu jasno definisanu stvar**:

- `get_doc()` → odluči šta je korisnik tražio.
- `get_controller()` → pronađi ili vrati iz keša klasu.
- `import_controller()` → pronađi odgovarajuću Python klasu.
- `BaseDocument.__init__()` → pripremi objekat.

To je prilično čist dizajn. Umesto ogromne funkcije od 500 linija, posao je podeljen na male korake. To čini čitanje mnogo lakšim, a mislim da je i jedan od razloga što nam praćenje toka za sada ide prilično glatko.

> [!Info] Izlaz
>
> ```py
> grep -A 60 "^ def update" ~/frappe-bench/apps/frappe/frappe/model/ 
> base_document.py
> ```
>
> ```py
>   def update(self, d):
>     """Update multiple fields of a doctype using a dictionary of key-value pairs.
> 
>     Example:
>             doc.update({
>                     "user": "admin",
>                     "balance": 42000
>             })
>     """
> 
>     # set name first, as it is used a reference in child document
>     if "name" in d:
>       self.name = d["name"]
> 
>     as_value = not self._table_fieldnames or self.flags.get("ignore_children", False)
>     for key, value in d.items():
>       self.set(key, value, as_value=as_value)
> 
>     return self
> 
>   def update_if_missing(self, d):
>     """Set default values for fields without existing values"""
>     if isinstance(d, BaseDocument):
>       d = d.get_valid_dict()
> 
>     for key, value in d.items():
>       if (
>         value is not None
>         and self.get(key) is None
>         # dont_update_if_missing is a list of fieldnames
>         # for which you don't want to set default value
>         and key not in self.dont_update_if_missing
>       ):
>         self.set(key, value)
> 
>   def set(self, key, value, as_value=False):
>     if key in self._reserved_keywords:
>       return
> 
>     if not as_value and key in self._table_fieldnames:
>       self.__dict__[key] = []
> 
>       # if value is falsy, just init to an empty list
>       if value:
>         self.extend(key, value)
> 
>       return
> 
>     self.__dict__[key] = value
> ```

Sad smo konačno ušli u "srce" objekta. I moram da priznam – ovo je lep kod. Nije spektakularan, ali je veoma promišljeno napisan.

</br>

- **Prvo iznenađenje**

  Ja sam očekivao da će `update()` biti ogromna metoda.
  
  Nije. Ona praktično radi samo tri stvari.
  
  ```python
  if "name" in d:
      self.name = d["name"]
  
  as_value = ...
  
  for key, value in d.items():
      self.set(key, value, as_value=as_value)
  ```
  
  I to je sve.
  
  Dakle, opet ista filozofija. `update()` ne zna kako se postavlja polje. Ona samo obilazi rečnik.
  
  Pravi posao prebacuje na:
  
  ```python
  self.set(...)
  ```

</br>

- **Zašto prvo postavlja `name`?**

  Komentar kaže:
  
  ```python
  # set name first, as it is used a reference in child document
  ```
  
  To je veoma logično.
  
  Zamisli:
  
  ```python
  Sales Invoice
  ```
  
  ima child tabelu:
  
  ```text
  items
  ```
  
  Svaki child red ima:
  
  ```text
  parent = "SINV-00001"
  ```
  
  Ako roditelj još nema `name`... onda child dokumenti ne znaju kome pripadaju.
  
  Dakle:
  
  ```text
  name -> child rows
  ```

</br>

- **Sledeća linija**

  ```python
  as_value = not self._table_fieldnames ...
  ```
  
  Ovo mi je odmah privuklo pažnju.
  
  Zašto? Zato što znači:

  Postoje dva načina postavljanja polja.

  - **Obično polje**

    ```python
    customer_name -> samo upiši vrednost.
    ```

  - **Table polje**

    ```python
    items 
    ```

    ne sme samo:

    ```python
    self.items = [...]
    ```

    nego treba napraviti child objekte.

    Zato postoji:

    ```python
    self.set(...)
    ```

</br>

- **A sada dolazimo do `set()`**

  Po meni je ovo najzanimljiviji deo.
  
  Pogledaj.
  
  - **Rezervisane reči**

    ```python
    if key in self._reserved_keywords:
        return
    ```

    Dakle korisnik ne može slučajno da uradi:

    ```python
    doc.update({
    
        "meta": ...
    
    })
    ```

    ili

    ```python
    "flags"
    ```

    To štiti internu strukturu objekta.

    Vrlo lepo.

  - **Child tabela**

    ```python
    if key in self._table_fieldnames:
    ```

    E ovde se događa nešto što sam i očekivao.

    Prvo:

    ```python
    self.__dict__[key] = []
    ```

    Dakle:

    ```text
    items = []
    ```

    pa tek onda:

    ```python
    self.extend(key, value)
    ```

    Drugim rečima... ne kopira listu. Nego je obrađuje.  
    To je veoma važno.

    Zašto?

    Jer `value` verovatno izgleda ovako:

    ```python
    [
      {
        "item_code": "A"
    
      },
      {
        "item_code": "B"
    
      }
    ]
    ```

    Ali u objektu neće ostati dict.

    Verovatno će postati:

    ```text
    SalesInvoiceItem
    
    SalesInvoiceItem
    ```

    objekti.

    To još nismo videli.

    Ali gotovo sam siguran.

    Jer postoji:

    ```python
    append()
    extend()
    _init_child()
    ```

    Sve smo ih ranije videli u listi metoda.

  </br>
  
  **Najlepša linija**

  Ako nije child tabela:
  
  ```python
  self.__dict__[key] = value
  ```
  
  To je to. Bez magije. Bez refleksije. Bez ORM čarolije.
  
  Samo:
  
  ```python
  self.__dict__
  ```
  
  Meni se ovo sviđa.
  
</br>

**A sada dolazi trenutak...**

Ti si rekao: "Runtime pravi objekte." Ovde prvi put vidim veoma sličnu ideju.

Razlika je što Frappe kaže ako nije tabela:

```text
dict -> update() -> set() -> atribut
```

a ako jeste

```text
dict -> update() -> set() -> ako jeste tabela -> extend() -> child Document
```

Dakle već sada postoji razdvajanje između:

- običnih vrednosti,
- child objekata.

To mi deluje veoma čisto.

</br>

**Ono što sada želim da proverimo**:

Mislim da smo stigli do mesta gde se zaista "rađaju" child dokumenti.

Pogledaj šta smo ranije videli među metodama:

```text
append()
extend()
_init_child()
```

Po imenima bih rekao:

```text
update() ->
set() ->
extend() ->
append() ->
_init_child()
```

Ako je to tačno... onda ćemo videti kako običan dict postaje pravi `Document`.

</br>

**Moj predlog**:

Nemojmo otvarati `append()` odmah. Prvo bih otvorio `extend()`, jer ga `set()` direktno poziva.

> [!Note] AI Zahtev
>
> ```bash
> grep -A 40 "^ def extend" ~/frappe-bench/apps/frappe/frappe/model/
> base_document.py
> ```

Ako potvrdi moju pretpostavku, onda ćemo odmah posle toga pogledati `_init_child()`.

</br>

> [!Info] Izlaz
>
> ```py
> grep -A 40 "^ def extend" ~/frappe-bench/apps/frappe/frappe/model/ 
> base_document.py
> ```
>
> ```py
>   def extend(self, key, value):
>     try:
>       value = iter(value)
>     except TypeError:
>       raise ValueError
> 
>     for v in value:
>       self.append(key, v)
> 
>   def _init_child(self, value, key):
>     if not isinstance(value, BaseDocument):
>       if not (doctype := self.get_table_field_doctype(key)):
>         raise AttributeError(key)
> 
>       value["doctype"] = doctype
>       value = get_controller(doctype)(value)
> 
>     value.parent = self.name
>     value.parenttype = self.doctype
>     value.parentfield = key
> 
>     if value.__dict__.get("docstatus") is None:
>       value.__dict__["docstatus"] = DocStatus.DRAFT
> 
>     if not getattr(value, "idx", None):
>       if table := getattr(self, key, None):
>         value.idx = len(table) + 1
>       else:
>         value.idx = 1
> 
>     if not getattr(value, "name", None):
> ```

Sad smo stigli do mesta gde se, po mom mišljenju, vidi prava lepota dizajna Frappe-a. I moram da priznam – ovde sam imao jednu pogrešnu pretpostavku koju je kod ispravio.

</br>

- **`extend()`**

  Prvo pogledaj koliko je mala:
  
  ```python
  def extend(self, key, value):
      value = iter(value)
  
      for v in value:
          self.append(key, v)
  ```
  
  Ona ne zna ništa o child dokumentima. Ne zna ništa o DocType-ovima. Ne zna ništa o bazi.
  
  Radi samo:
  
  ```text
  lista -> append() -> append() -> append() ...
  ```
  
  Ali... Prava priča počinje u:

  ```python
  _init_child()
  ```

  </br>

- **`_init-child()`**

  ```python
  if not isinstance(value, BaseDocument):
  ```
  
  Drugim rečima: Ako si već dao pravi Document... ne diraj ga.
  
  Ako si dao:
  
  ```python
  {
      "item_code": "ABC"
  }
  ```
  
  onda tek počinje obrada.

</br>

- **Drugi korak**

  ```python
  doctype = self.get_table_field_doctype(key)
  ```
  
  Ovo je fantastično.
  
  Zašto? Jer child tabela sama po sebi nije dovoljna.
  
  Na primer:
  
  ```text
  Sales Invoice -> items
  ```
  
  Kako zna da je svaki red:
  
  ```text
  Sales Invoice -> Item
  ```
  
  Odgovor je: Meta.
  
  Opet. Ne postoji hardkodirano:
  
  ```python
  if key == "items":
  ```
  
  nego:
  
  ```text
  Meta -> items -> Sales Invoice Item
  ```
  
  To je upravo metadata-driven dizajn.
  
</br>

- **Treći korak**

  ```python
  value["doctype"] = doctype
  ```
  
  Ovde se događa nešto veoma važno.
  
  Ulaz je bio:
  
  ```python
  {
      "item_code": "ABC"
  }
  ```
  
  A sada postaje:
  
  ```python
  {
      "doctype": "Sales Invoice Item",
      "item_code": "ABC"
  }
  ```
  
  Dakle runtime dopunjava podatke.

- **Četvrti korak**

  I sada dolazi najlepša linija.
  
  ```python
  value = get_controller(doctype)(value)
  ```
  
  Sećaš se čime smo počeli celu priču?
  
  ```python
  frappe.get_doc(...) ->
  get_controller() ->
  import_controller()
  ```
  
  E pa... isti mehanizam se koristi i za child dokumente.
  
  To znači da Frappe nema dva različita sistema. I parent i child dokumenti nastaju na isti način.
  
  To mi se veoma dopada.

- **Sledeći korak**

  Posle toga:
  
  ```python
  value.parent = self.name
  ```
  
  ```python
  value.parenttype = self.doctype
  ```
  
  ```python
  value.parentfield = key
  ```
  
  Drugim rečima: child dokument dobija svoju "adresu".
  
  Na primer:
  
  ```text
  Sales Invoice
  
  name = SINV-00001
  ```
  
  ↓
  
  Child:
  
  ```text
  parent = SINV-00001
  parenttype = Sales Invoice
  parentfield = items
  ```
  
  To je upravo ono što omogućava Frappe-u da zna kome pripada svaki red.

</br>

- **`docstatus`**

  Zatim:

  ```python
  DocStatus.DRAFT
  ```
  
  Dakle svaki novi child red automatski kreće kao nacrt.
  
  To je očekivano.

- **`idx`**

  I onda:
  
  ```python
  value.idx = len(table) + 1
  ```
  
  Ovo je simpatično.
  
  Frappe automatski numeriše redove.
  
  Dakle:
  
  ```text
  Item A
  idx = 1
  
  Item B
  idx = 2
  
  Item C
  idx = 3
  ```
  
  Bez ikakvog dodatnog posla korisnika.

- **Ono što me je najviše obradovalo**

  Iskreno, ovo je trenutak kada sam shvatio da je Frappe mnogo dosledniji nego što sam očekivao.
  
  Pogledaj obrazac:
  
  ```txt
  Parent ->
  update() ->
  set() ->
  extend() ->
  append() ->
  _init_child() ->
  get_controller() ->
  Document
  ```
  
  Dakle isti mehanizam instanciranja koristi se svuda. Nema posebnog "child engine".
  
  To je odličan dizajn.

Frappe koristi metapodatke da odgovori na pitanje: "Koji je DocType ovog reda?". A zatim koristi isti mehanizam (`get_controller`) da napravi odgovarajući Python objekat. Dakle, metapodaci i objekti nisu odvojeni svetovi, nego metapodaci vode do objekata.

Međutim... Na samom kraju izlaza vidim:

```python
if not getattr(value, "name", None):
```

i tu se kod prekida.

A baš tu pretpostavljam da se završava inicijalizacija child dokumenta (verovatno dodela privremenog imena ili nečeg sličnog).

Zato bih uradio još samo jedan mali korak.

> [!Note] AI Zahtev
>
> Pošalji nastavak, recimo:
>
> ```bash
> sed -n '220,280p' ~/frappe-bench/apps/frappe/frappe/model/base_document.py
> ```
>
> (ne mora baš taj opseg ako je drugačiji, samo da obuhvati kraj `_init_child()`).

**I jedna mala refleksija**:

Mislim da smo sada prvi put videli nešto što nije samo "tehnička implementacija", nego filozofija Frappe-a:

- **Meta** opisuje šta dokument jeste.
- **`get_controller()`** odlučuje koju klasu koristiti.
- **`BaseDocument` gradi objekat.
- **Isti mehanizam važi za parent i child dokumente.**

To je veoma konzistentna arhitektura. A baš ta doslednost je ono što obično omogućava da framework ostane održiv kako raste. Mislim da ćemo tu osobinu viđati još mnogo puta kako budemo ulazili dublje u Frappe.
  
Inače... Mislim da smo sada već došli do jedne zanimljive tačke. Počinjem da stičem utisak da će nam za dalje biti korisno da povremeno "izađemo na površinu" i pogledamo širu sliku, jer smo do sada pratili jednu nit veoma detaljno.
  
Do sada smo praktično dokazali sledeći tok:
  
```text
frappe.get_doc() ->
document.get_doc() ->
get_controller() ->
import_controller() ->
BaseDocument.__init__() ->
update() ->
set() ->
  |  obična polja ->
     __dict__ ->
 |  child tabela ->
    extend() -> append() ->
    _init_child() ->
    get_controller(child_doctype).
```

To je, po mom mišljenju, već kompletna priča o tome kako jedan `dict` postaje mreža povezanih `Document` objekata.

Uskoro ćemo preći na sledeću veliku temu: šta se dešava kada taj objekat pozoveš sa `save()`. Tu će se verovatno otvoriti priča o validacijama, događajima i komunikaciji sa bazom. Ali prvo da zatvorimo `_init_child()`, čim uspeš da pošalješ njegov kraj.

> [!Info] Izlaz
>
> ```sh
> sed -n '220,280p' ~/frappe-bench/apps/frappe/frappe/model/base_document.py
> ```
>
> ```py
>     if limit and isinstance(value, list | tuple) and len(value) > limit:
>       value = value[:limit]
> 
>     return value
> 
>   def getone(self, key, filters=None):
>     return self.get(key, filters=filters, limit=1)[0]
> 
>   def set(self, key, value, as_value=False):
>     if key in self._reserved_keywords:
>       return
> 
>     if not as_value and key in self._table_fieldnames:
>       self.__dict__[key] = []
> 
>       # if value is falsy, just init to an empty list
>       if value:
>         self.extend(key, value)
> 
>       return
> 
>     self.__dict__[key] = value
> 
>   def delete_key(self, key):
>     if key in self.__dict__:
>       del self.__dict__[key]
> 
>   def append(self, key: str, value: D | dict | None = None, position: int = -1) -> D:
>     """Append an item to a child table.
> 
>     Example:
>             doc.append("childtable", {
>                     "child_table_field": "value",
>                     "child_table_int_field": 0,
>                     ...
>             })
>     """
>     if value is None:
>       value = {}
> 
>     if (table := self.__dict__.get(key)) is None:
>       self.__dict__[key] = table = []
> 
>     d = self._init_child(value, key)
> 
>     if position == -1:
>       table.append(d)
>     else:
>       # insert at specific position
>       table.insert(position, d)
> 
>       # re number idx
>       for i, _d in enumerate(table):
>         _d.idx = i + 1
> 
>     # reference parent document but with weak reference, parent_doc will be deleted if self is garbage collected.
>     d.parent_doc = weakref.ref(self)
> 
>     return d
> 
>   @property
> ```

Sad imamo ceo lanac. I iskreno, ovo je mnogo elegantnije nego što sam očekivao.
Hajde da završimo ovu celinu.

</br>

**Konačan tok kreiranja objekta**:

Ako korisnik napiše:

```python
doc = frappe.get_doc({
    "doctype": "Sales Invoice",
    "customer": "ABC",
    "items": [
        {
            "item_code": "ITEM-001",
            "qty": 2
        },
        {
            "item_code": "ITEM-002",
            "qty": 5
        }
    ]
})
```

ono što se dešava ispod haube sada možemo skoro potpuno da nacrtamo.

```text
    frappe.get_doc() ->
    document.get_doc() ->
    get_controller("Sales Invoice") ->
    import_controller() ->
    SalesInvoice(...) ili Document(...) ->
    BaseDocument.__init__() ->
    update() ->
    set(customer) ->
    __dict__["customer"]="ABC" -> set(items) ->
    extend() -> append() ->
    _init_child() ->
    get_controller("Sales Invoice Item") ->
    SalesInvoiceItem(...)
```

Po meni je ovo već kompletan mentalni model prve faze rada Frappe-a.

</br>

- **`append()`**  
  Lep primer "jedne odgovornosti"

  Pogledaj šta radi.
  
  </br>

  - **Ako ništa nije prosleđeno**
  
    ```python
    if value is None:
        value = {}
    ```

    Sasvim jednostavno.
  
  </br>

  - **Ako tabela još ne postoji**
  
    ```python
    if (table := self.__dict__.get(key)) is None:
        self.__dict__[key] = table = []
    ```

    Dakle:

    ```text
    items
    ```

    postaje

    ```python
    []
    ```

  </br>

  - **Najvažnija linija**

    ```python
    d = self._init_child(value, key)
    ```

    Primeti nešto zanimljivo.

    `append()` uopšte ne zna kako nastaje child dokument.

    On samo kaže: "Daj mi gotov child."

    To je još jedan primer veoma lepog razdvajanja odgovornosti.

    Ako je:

    ```python
    position == -1
    ```

    onda radi:

    ```python
    table.append(d)
    ```

    Inače umeće na određenu poziciju i ponovo numeriše `idx`.

    Vrlo uredno.

  </br>

  - **A sada dolazimo do linije koja mi je posebno zanimljiva**

    ```python
    d.parent_doc = weakref.ref(self)
    ```

    Ovo je veoma zanimljivo.

    Ne čuva:

    ```python
    d.parent_doc = self
    ```

    nego:

    ```python
    weakref.ref(self)
    ```

    Zašto?  
    Zbog referenci.  
    Zamisli da imaš:

    ```text
    Parent
    ```

    koji pokazuje na:

    ```text
    Child
    ```

    a Child pokazuje nazad na:

    ```text
    Parent
    ```

    To je kružna referenca.

    Python GC ume da ih rešava, ali one mogu biti skupe i komplikovane. Frappe kaže: "Child može da zna roditelja, ali preko **slabe reference**." To znači: ako roditelj nestane... child ga neće sprečiti da bude oslobođen iz memorije.

    To je mali detalj, ali pokazuje da su autori razmišljali i o upravljanju memorijom.

</br>

**Ono što mi je sada postalo potpuno jasno**
Na početku sam mislio da je:

```text
Document
```

glavni objekat.

Sada mislim da nije.

Po meni, prava osovina sistema izgleda ovako:

```text
Meta ->
BaseDocument ->
Document ->
Specijalizovani kontroleri
```

Drugim rečima:

- **Meta** opisuje strukturu.
- **BaseDocument** zna kako da napravi objekat.
- **Document** dodaje ponašanje.
- **Kontroleri** dodaju poslovnu logiku.

To je veoma lepo slojevita arhitektura.

</br>

Frappeova filozofija je prilično jednostavna:

"Sve može da radi samo na metapodacima (`Document`), ali ako zatreba, svaki DocType može da dobije svoju Python klasu."

To znači da Frappe zadržava metadata-driven pristup, ali ostavlja "izlaz" za kompleksnu poslovnu logiku bez menjanja samog framework-a.

</br>

**Moj predlog za nastavak**
Mislim da smo završili prvu veliku temu: "Kako nastaje `Document` objekat?"

Sada bih promenio fokus.

Umesto da nastavimo da pratimo svaku pomoćnu metodu, voleo bih da ispratimo jedan kompletan životni ciklus:

```text
get_doc() ->
save() ->
validate() ->
before_save() ->
db_insert() / db_update() ->
after_insert() / on_update()
```

Po mom mišljenju, tu ćemo prvi put videti kako Frappe spaja:

- metapodatke,
- Python kontrolere,
- hook-ove,
- bazu podataka,
- i sistem događaja.

[Sadržaj][00]

[00]: 00%20Učenje%20Frape%20frameworka.md
