# Učenje Frappe Framework

[Sadržaj][00]

## 12 Životni ciklus dokumenta

Rekao bih da je **životni ciklus dokumenta** za Frappe isto ono što je **HTTP request lifecycle** za Django ili **middleware pipeline** za ASP.NET.

Ako ovo razumeš, razumećeš gde ide gotovo sav poslovni kod.

### insert()

Ne zanima nas (za sada) kako `insert()` radi iznutra.

Nego: Ako napišem:

```python
doc.insert()
```

šta će Frappe redom da pozove?

Jer od toga zavisi gde ćeš pisati svoj kod.

</br>

**Hajde da krenemo od programera**:

Zamisli da imaš:

```python
doc = frappe.new_doc("Service Order")
doc.customer = "ABC"
doc.insert()
```

Pitanje je: **Šta bi framework morao da uradi?**

Ja bih očekivao nešto ovako:
  
```text
Da li je dokument ispravan? -> Validacija ->
Da li treba nešto automatski popuniti? -> Upis u bazu ->
Obavesti ostale delove sistema ->
Kraj
```

To je intuitivno.

Ali... Koliko događaja postoji?

Kojim redom? Koji se izvršavaju samo kod `insert()`, a koji i kod `save()`?

To sada treba da otkrijemo.

Ovde ćemo malo promeniti metod rada:

Do sada smo pratili:

```py
get_doc() -> get_controller() 
...
```

Sada ćemo pratiti:

```sh
insert() -> šta poziva? -> šta ono poziva? ->
...
```

Ali nećemo ulaziti u svaku pomoćnu funkciju.

Interesuje nas samo redosled događaja.

</br>

**Prvi zadatak**:

Pronađimo `insert()`.
  
Pošalji:

```bash
grep -n "^    def insert" ~/frappe-bench/apps/frappe/frappe/model/document.py
```

Kada dobijemo broj linije, otvorićemo otprilike 100–150 linija oko nje.

</br>

**Šta očekujem?**
Pre nego što otvorimo kod, napraviću hipotezu (kao i do sada).

Mislim da će `insert()` izgledati otprilike ovako:

```txt
insert() -> before_insert -> validate -> before_save -> db_insert -> after_insert -> on_update -> notify
```

Ali ovo je samo pretpostavka.

Nećemo učiti napamet iz dokumentacije niti nagađati — proverićemo u izvornom kodu.

I još nešto važno:

Od ove teme pa nadalje, voleo bih da za svaku metodu koju sretnemo odgovorimo na **tri pitanja**:

| Metoda            | Kada se poziva? | Kada je programer koristi? |
| ----------------- | --------------- | -------------------------- |
| `validate()`      | ?               | ?                          |
| `before_insert()` | ?               | ?                          |
| `before_save()`   | ?               | ?                          |
| `on_update()`     | ?               | ?                          |
| `on_submit()`     | ?               | ?                          |

Na kraju ćemo imati jednu tabelu koja će, po mom mišljenju, vredeti više od desetina stranica dokumentacije. To će biti nešto što ćeš moći da pogledaš za 30 sekundi i odmah znati **gde ide koja vrsta logike**.

Mislim da će to biti jedan od najkorisnijih rezultata našeg zajedničkog proučavanja Frappe-a.

Evo izlaza

```sh
grep -n "def insert" ~/frappe-bench/apps/frappe/frappe/model/document.py
```

```sh
261:    def insert(
```

Evo izlaza

```sh
sed -n '261,380p' ~/frappe-bench/apps/frappe/frappe/model/document.py
```

```py
def insert(
    self,
        ignore_permissions=None,
        ignore_links=None,
        ignore_if_duplicate=False,
        ignore_mandatory=None,
        set_name=None,
        set_child_names=True,
    ) -> "Document":
        """Insert the document in the database (as a new document).
        This will check for user permissions and execute `before_insert`,
        `validate`, `on_update`, `after_insert` methods if they are written.

        :param ignore_permissions: Do not check permissions if True.
        :param ignore_links: Do not check validity of links if True.
        :param ignore_if_duplicate: Do not raise error if a duplicate entry exists.
        :param ignore_mandatory: Do not check missing mandatory fields if True.
        :param set_name: Name to set for the document, if valid.
        :param set_child_names: Whether to set names for the child documents.
        """
        if self.flags.in_print:
            return self

        self.flags.notifications_executed = []

        if ignore_permissions is not None:
            self.flags.ignore_permissions = ignore_permissions

        if ignore_links is not None:
            self.flags.ignore_links = ignore_links

        if ignore_mandatory is not None:
            self.flags.ignore_mandatory = ignore_mandatory

        self.set("__islocal", True)

        self._set_defaults()
        self.set_user_and_timestamp()
        self.set_docstatus()
        self.check_permission("create")
        self.check_if_latest()
        self._validate_links()
        self.run_method("before_insert")
        self.set_new_name(set_name=set_name, set_child_names=set_child_names)
        self.set_parent_in_children()
        self.validate_higher_perm_levels()

        self.flags.in_insert = True
        self.run_before_save_methods()
        self._validate()
        self.set_docstatus()
        self.flags.in_insert = False

        # run validate, on update etc.

        # parent
        if getattr(self.meta, "issingle", 0):
            self.update_single(self.get_valid_dict())
        else:
            self.db_insert(ignore_if_duplicate=ignore_if_duplicate)

        # children
        for d in self.get_all_children():
            d.db_insert()

        self.run_method("after_insert")
        self.flags.in_insert = True

        if self.get("amended_from"):
            self.validate_amended_from()
            self.copy_attachments_from_amended_from()

        relink_mismatched_files(self)
        self.run_post_save_methods()
        self.flags.in_insert = False

        # delete __islocal
        if hasattr(self, "__islocal"):
            delattr(self, "__islocal")

        # clear unsaved flag
        if hasattr(self, "__unsaved"):
            delattr(self, "__unsaved")

        if not (frappe.flags.in_migrate or frappe.local.flags.in_install or frappe.flags.in_setup_wizard):
            if frappe.get_cached_value("User", frappe.session.user, "follow_created_documents"):
                follow_document(self.doctype, self.name, frappe.session.user)
        return self

    def check_if_locked(self):
        if not self.creation or not self.is_locked:
            return

        # Allow unlocking if created more than 60 minutes ago
        primary_action = None
        if file_lock.lock_age(self.get_signature()) > DOCUMENT_LOCK_SOFT_EXPIRY:
            primary_action = {
                "label": "Force Unlock",
                "server_action": "frappe.model.document.unlock_document",
                "hide_on_success": True,
                "args": {
                    "doctype": self.doctype,
                    "name": self.name,
                },
            }

        frappe.throw(
            _(
                "This document is currently locked and queued for execution. Please try again after some time."
            ),
            title=_("Document Queued"),
            primary_action=primary_action,
            exc=frappe.DocumentLockedError,
        )

    def save(self, *args, **kwargs):
        """Wrapper for _save"""
        return self._save(*args, **kwargs)

    def _save(self, ignore_permissions=None, ignore_version=None) -> "Document":
```
  
Odmah mogu da kažem: sada ćemo prestati da "kopamo" i počećemo da gradimo mentalni model.

Mislim da se `insert()` može podeliti u **četiri logične faze**.

</br>

- **1 – Priprema**

  ```python
  self._set_defaults()
  self.set_user_and_timestamp()
  self.set_docstatus()
  self.check_permission("create")
  self.check_if_latest()
  self._validate_links()
  ```
  
  Ovo nisu događaji koje ti pišeš. Ovo framework radi da bi dokument bio spreman.

  Ja ovo zovem: **Infrastructure phase**.  

  Tu Frappe postavlja temelje.

</br>

- **2 – before_insert()**

  Ovde se prvi put pojavljuje nešto što programer piše.
  
  ```python
  self.run_method("before_insert")
  ```
  
  Ovo znači:
  
  Ako u svom kontroleru napišeš:
  
  ```python
  class ServiceOrder(Document):
  
      def before_insert(self):
          ...
  ```
  
  ovde će biti pozvano.
  
  I odmah jedna važna stvar.

  `before_insert()` se izvršava **samo jednom u životu dokumenta**.

   Nikada više.

   To ga razlikuje od mnogih drugih metoda.

</br>

- **2.1 - set_new_name**

  ```python
  self.set_new_name(...)
  ```
  
  Framework dodeljuje ime.
  
  Na primer:
  
  ```py
  SO-00001
  ```
  
  ili
  
  ```py
  CUST-00034
  ```
  
  ili šta god `Naming Rule` kaže.

</br>

- **2.2 - set_parent_in_children()**

  Posle toga:
  
  ```python
  self.set_parent_in_children()
  ```
  
  To smo praktično već videli kada smo proučavali `_init_child()`.

</br>

- **3 – Najvažniji deo**

  Po meni je ovo srce celog životnog ciklusa.
  
  ```python
  self.run_before_save_methods()
  self._validate()
  self.set_docstatus()
  ```
  
  E sad... Ovde nam se pojavljuje nešto veoma zanimljivo.
  
  **3.1 - run_before_save_methods()**
  
  To je, po mom mišljenju, sledeća funkcija koju treba otvoriti.
  
  Zašto?
  
  Jer se iz njenog imena vidi da ona verovatno poziva:
  
  - validate
  - before_save
  - možda još nešto
  
  Drugim rečima, mislim da se **pravi redosled događaja krije upravo tu**.

</br>

- **4 – Upis u bazu**

  Ovde više nema filozofije.
  
  ```python
  self.db_insert()
  ```
  
  i zatim
  
  ```python
  child.db_insert()
  ```
  
  Dakle:
  
  Prvo roditelj. Pa sva deca.
  
  To ima smisla.

- **Posle baze**

  Sada dolaze događaji koji se izvršavaju **nakon INSERT-a**.
  
  Prvo:
  
  ```python
  self.run_method("after_insert")
  ```
  
  Dakle:
  
  ```python
  class ServiceOrder(Document):
  
      def after_insert(self):
          ...
  ```
  
  će se izvršiti ovde.
  
  A onda:
  
  ```python
  self.run_post_save_methods()
  ```
  
  I evo opet funkcije koju moramo otvoriti.
  
  Po imenu očekujem da upravo ona poziva:
  
  - on_update
  - notify
  - hook-ove
  - možda Version
  - možda Assignment
  - možda Timeline
  
  Ne znamo.
  
  Ali gotovo sigurno se tu krije pola Frappe-a. 😊

Već imamo prvu verziju dijagrama.

```text
insert()
   │
   ├── priprema
   │
   ├── before_insert()
   │
   ├── naming
   │
   ├── run_before_save_methods()
   │
   ├── _validate()
   │
   ├── db_insert()
   │
   ├── child.db_insert()
   │
   ├── after_insert()
   │
   └── run_post_save_methods()
```

Po meni je ovo ogromno otkriće.

Zašto?

Zato što sada više ne razmišljamo: "Koju funkciju da otvorim?"

nego: "U kojoj fazi života dokumenta želim da se izvrši moj kod?"

To je sasvim druga perspektiva.

</br>

**Sada dolazi možda najvažniji deo**:

Primetio sam nešto što mi se jako sviđa u dizajnu Frappe-a.

Pogledaj:

```python
self.run_method("before_insert")
```

Framework **ne poziva direktno**:

```python
self.before_insert()
```

nego:

```python
run_method(...)
```

To znači da `run_method()` verovatno radi mnogo više od običnog poziva metode.

Moja pretpostavka je da:

- poziva metodu kontrolera (ako postoji),
- poziva `doc_events` hook-ove,
- možda Server Script,
- možda još neke ekstenzije.

Ako je to tačno, onda je **run_method()** centralna tačka sistema događaja u Frappe-u.

I ako mene pitaš, to je mnogo zanimljivije od `db_insert()`.

</br>

**Moj predlog za sledeći korak**:

Ja ne bih sada otvorio `_validate()`. Niti `db_insert()`.

Nego bih odmah krenuo na:

```python
run_before_save_methods()
```

jer mislim da ćemo tu prvi put videti:

- kada se poziva `validate()`,
- kada `before_save()`,
- kako Frappe razlikuje `insert()` od `save()`.

A odmah posle toga bih otvorio `run_method()`.

Imam osećaj da će nam te dve funkcije objasniti skoro ceo sistem događaja koji te zapravo zanima iz ugla programera.

### run_post_save_methods()

Koristi globalnu pretragu iz korena Frappe aplikacije:

```bash
grep -R -n "def run_before_save_methods" ~/frappe-bench/apps/frappe/frappe
```

Ako ništa ne vrati (što me baš čudi), onda probaj širu pretragu:

```bash
grep -R -n "run_before_save_methods" ~/frappe-bench/apps/frappe/frappe
```

Ona će pokazati i mesto gde je definisana i sva mesta gde se poziva.

</br>

**Zašto baš nju tražimo?**

Voleo bih da ti objasnim zašto sam izabrao baš tu funkciju.

Iz `insert()` se vidi da je redosled:

```text
before_insert() -> run_before_save_methods() -> _validate() -> db_insert() -> after_insert() -> run_post_save_methods()
```

Ovde postoje dve "kutije":

- `run_before_save_methods()`
- `run_post_save_methods()`

Po mom iskustvu sa framework-ovima, upravo su takve funkcije **orkestratori**. One ne rade mnogo same, nego pozivaju desetak drugih metoda odgovarajućim redosledom.

A upravo **redosled** je ono što tebe zanima.

</br>

**Mislim da smo konačno na pravom tragu**:

Sećaš se kada si pre nekoliko dana rekao: *"Mnogo me više zanima kada se šta poziva nego kako radi unutra."*

Mislim da je ovo tačno mesto gde počinje odgovor na to pitanje.

Moj cilj više nije da znamo svaku internu funkciju Frappe-a.

Moj cilj je da na kraju možemo da nacrtamo jednu tabelu poput ove (ali sa **100% tačnim** redosledom, potvrđenim iz koda):

| Programer pozove | Frappe zatim poziva                          | Ovde obično pišeš                                   |
| ---------------- | -------------------------------------------- | --------------------------------------------------- |
| `insert()`       | `before_insert()`                            | Inicijalizacija koja važi samo pri prvom kreiranju  |
|                  | `validate()`                                 | Provera i priprema podataka                         |
|                  | `before_save()`                              | Zajednička logika pre upisa                         |
|                  | `db_insert()`                                | (Framework)                                         |
|                  | `after_insert()`                             | Radnje koje se izvršavaju samo nakon prvog INSERT-a |
|                  | `on_update()` (ako ga poziva post-save faza) | Sinhronizacija, dodatne akcije                      |

Kada završimo ovu temu, imaćeš nešto što je mnogo praktičnije od običnog čitanja izvornog koda: mapu životnog ciklusa dokumenta iz ugla programera. Mislim da će to biti jedna od najvrednijih stvari u celom našem proučavanju Frappe-a.

Evo izlaza

```sh
grep -R -n "def run_before_save_methods" ~/frappe-bench/apps/frappe/frappe
```

```sh
/home/radosav/frappe-bench/apps/frappe/frappe/model/document.py:1125:    def run_before_save_methods(self):
```

Ovo potvrđuje nešto što sam ti rekao prošli put:

**Ne treba da pamtimo gde se šta nalazi. Treba da naučimo kako da pronađemo odgovor.**

Ovo (`grep -R`) ćeš koristiti stalno kada budeš proučavao Frappe ili bilo koji veliki projekat.

Do sada sam ti tražio po 100-150 linija.

Mislim da ovde to nije potrebno.

Pošalji samo ovu metodu, do početka sledeće:

```bash
sed -n '1125,1195p' ~/frappe-bench/apps/frappe/frappe/model/document.py
```

Ako nije cela, nastavićemo još 20-30 linija.

Mislim da smo sada stigli do **prvog zaista važnog mesta za Frappe programera**.

Pogledaj šta već znamo.

Programer napiše:

```python
doc.insert()
```

Framework uradi:

```text
before_insert() -> run_before_save_methods() -> _validate() -> INSERT -> after_insert() -> run_post_save_methods()
```

Sve što je do sada bilo jeste kostur. Sada ćemo prvi put videti meso.

Drugim rečima:

</br>

**Šta zapravo znači "pre save"**:

- **Imam jednu hipotezu**

  Pretpostavljam da ćemo videti nešto ovako:
  
  ```txt
  if insert:
      before_insert
  if save:
      before_save
  validate
  before_submit
  ...
  ```
  
  ili nešto veoma slično.
  
  Ako bude tako, onda će nam odjednom postati jasno:
  
  - zašto postoji `validate()`,
  - zašto postoji `before_insert()`,
  - zašto postoji `before_save()`,
  
  i kada se svaki od njih koristi.

I još jedna stvar. od sada ću pokušati da manje nagađam, a više da izvodimo zaključke direktno iz koda.

Do sada nam je to bilo korisno jer smo tek upoznavali arhitekturu.

Ali sada ulazimo u deo koji ćeš stvarno koristiti u razvoju, pa želim da ono što zapišemo u tvojoj dokumentaciji bude potvrđeno iz izvornog koda, a ne moja pretpostavka.

Evo izlaza:

```sh
sed -n '1125,1195p' ~/frappe-bench/apps/frappe/frappe/model/document.py
```

```py
    def run_before_save_methods(self):
        """Run standard methods before    `INSERT` or `UPDATE`. Standard Methods are:

        - `validate`, `before_save` for **Save**.
        - `validate`, `before_submit` for **Submit**.
        - `before_cancel` for **Cancel**
        - `before_update_after_submit` for **Update after Submit**

        Will also update title_field if set"""

        self.reset_seen()

        # before_validate method should be executed before ignoring validations
        if self._action in ("save", "submit"):
            self.run_method("before_validate")

        if self.flags.ignore_validate:
            return

        if self._action == "save":
            self.run_method("validate")
            self.run_method("before_save")
        elif self._action == "submit":
            self.run_method("validate")
            self.run_method("before_submit")
        elif self._action == "cancel":
            self.run_method("before_cancel")
        elif self._action == "update_after_submit":
            self.run_method("before_update_after_submit")

        self.set_title_field()

    def load_doc_before_save(self, *, raise_exception: bool = False):
        """load existing document from db before saving"""

        self._doc_before_save = None

        if self.is_new():
            return

        try:
            self._doc_before_save = frappe.get_doc(self.doctype, self.name, for_update=True)
        except frappe.DoesNotExistError:
            if raise_exception:
                raise

            frappe.clear_last_message()

    def run_post_save_methods(self):
        """Run standard methods after `INSERT` or `UPDATE`. Standard Methods are:

        - `on_update` for **Save**.
        - `on_update`, `on_submit` for **Submit**.
        - `on_cancel` for **Cancel**
        - `update_after_submit` for **Update after Submit**"""

        if self._action == "save":
            self.run_method("on_update")
        elif self._action == "submit":
            self.run_method("on_update")
            self.run_method("on_submit")
        elif self._action == "cancel":
            self.run_method("on_cancel")
            self.check_no_back_links_exist()
        elif self._action == "update_after_submit":
            self.run_method("on_update_after_submit")

        self.clear_cache()

        if self.flags.get("notify_update", True):
            self.notify_update()
```

Mislim da je ovo prvi put da smo otvorili deo koda koji je direktno koristan za svakodnevni razvoj u Frappe-u.

I odmah ću da ispravim jednu svoju grešku iz prethodnih razgovora.

</br>

- **Prvo ispravka**

  Ranije sam govorio o metodama kao da ih `insert()` poziva direktno.

  To nije tačno.

  Prava slika je:

  ```text
  insert() -> run_before_save_methods() -> run_method(...)
  ```

  i  

  ```text
  run_post_save_methods() -> run_method(...)
  ```

  Dakle, **run_before_save_methods()** i **run_post_save_methods()** su orkestratori životnog ciklusa.

</br>

- **Druga stvar**

  Pogledaj komentar.
  
  To nije običan komentar. To je praktično specifikacija Frappe-a.
  
  ```python
  Save
      validate
      before_save
  
  Submit
      validate
      before_submit
  
  Cancel
      before_cancel
  
  Update after submit
      before_update_after_submit
  ```
  
  Odmah ispod toga vidiš implementaciju.
  
  To mi se jako sviđa.

</br>

- **E sad dolazi nešto što je meni bilo novo**

  Pogledaj ovo.

  ```python
  if self._action in ("save", "submit"):
      self.run_method("before_validate")
  ```
  
  Dakle postoji:
  
  ```text
  before_validate
  ```
  
  Mi ga do sada uopšte nismo pominjali.
  
  To znači da je redosled:
  
  ```text
  before_validate -> validate -> before_save -> 
  ```
  
  a ne samo
  
  ```text
  validate -> before_save
  ```
  
  To je veoma važna razlika.

  </br>

  **Zašto postoji before_validate?**
  
  Po meni je logika sledeća. Recimo da korisnik nije uneo nešto što može automatski da se izračuna.
  
  Na primer:
  
  ```text
  full_name
  ```
  
  iz:
  
  ```text
  first_name
  last_name
  ```
  
  To možeš da uradiš ovde.
  
  ```python
  def before_validate(self):
  
      self.full_name = ...
  ```
  
  A onda:
  
  ```python
  validate()
  ```
  
  već proverava gotove podatke.
  
  To mi deluje veoma logično.

</br>

- **Sada pogledaj Save**

  ```python
  validate() -> before_save()
  ```
  
  Ovo je zanimljivo.
  
  Zašto prvo validate?
  
  Zato što `before_save()` više nije mesto za proveru podataka.
  
  Po meni:
  
  ```text
  validate = da li je dokument ispravan?
  ```
  
  a
  
  ```text
  before_save = sada kada znam da jeste, uradi završne pripreme
  ```

</br>

- **Submit**

  Ovde mi se posebno sviđa dizajn.
  
  ```text
  validate() -> before_submit()
  ```
  
  Dakle... Submit nije posebna planeta. On takođe prolazi kroz validaciju.
  To znači da pravila koja važe za dokument važe i prilikom submit-a.
  
  To je veoma elegantno.

</br>

- **A sada druga polovina**

  Posle baze.
  
  ```python
  run_post_save_methods()
  ```
  
  Ovde je sve mnogo jasnije.
  
  **Save**:
  
  ```text
  on_update
  ```
  
  **Submit**:
  
  ```text
  on_update -> on_submit
  ```
  
  E ovo je interesantno.
  
  `submit()` poziva i:
  
  ```text
  on_update
  ```
  
  i:
  
  ```text
  on_submit
  ```
  
  Dakle `submit()` je zapravo specijalan slučaj update-a.
  
  To mi ranije nije bilo očigledno.

- **Cancel**

Posle cancel-a:

```text
on_cancel
```

I odmah:

```python
check_no_back_links_exist()
```

To znači da framework odmah proverava referencijalni integritet.

Mislim da sada konačno možemo napraviti tabelu koju ćeš zaista koristiti.

| Događaj           | Kada se poziva                           | Tipična upotreba                                                      |
| ----------------- | ---------------------------------------- | --------------------------------------------------------------------- |
| `before_validate` | Pre svake validacije (`save`, `submit`)  | Popunjavanje ili normalizacija podataka pre provere.                  |
| `validate`        | Pre čuvanja i pre submit-a               | Provera poslovnih pravila, bacanje greške ako dokument nije ispravan. |
| `before_save`     | Samo kod običnog `save`                  | Završne pripreme pre upisa dokumenta.                                 |
| `before_submit`   | Neposredno pre `submit`                  | Provere koje važe samo pri predaji dokumenta.                         |
| `on_update`       | Posle uspešnog `save` (i tokom `submit`) | Reakcija na uspešno snimanje dokumenta.                               |
| `on_submit`       | Posle uspešnog `submit`                  | Radnje koje se izvršavaju samo jednom pri potvrđivanju dokumenta.     |
| `before_cancel`   | Pre otkazivanja                          | Provere pre otkazivanja.                                              |
| `on_cancel`       | Posle otkazivanja                        | Čišćenje, oslobađanje resursa, dodatne akcije.                        |

I sada dolazimo do pitanja koje si postavio pre nekoliko dana: **Kada koristim Controller, a kada Hook?** Mislim da sada imamo dovoljno znanja da damo precizan odgovor.

Controller ne pišeš zato što postoji `validate()`. Controller pišeš zato što je to prirodno mesto gde dokument definiše svoje ponašanje.

Na primer:

```python
class ServiceOrder(Document):

    def validate(self):
        ...

    def before_submit(self):
        ...

    def on_submit(self):
        ...
```

To je životni ciklus tog dokumenta.

Hook dolazi tek posle.

Hook kaže: "Kada ovaj događaj nastupi, moja aplikacija želi još nešto da uradi."

Drugim rečima:

```text
Document -> Controller metoda -> run_method() -> Hook-ovi
```

I upravo zbog toga bih voleo da sledeća funkcija koju otvorimo bude `run_method()`.

### run_method

Ne zato što želim da kopamo po kodu, nego zato što mislim da će nam ona odgovoriti na poslednje veliko pitanje:

</br>

**Kako Frappe spaja Controller, Hook-ove, Server Script-ove i ostala proširenja u jedan jedinstven sistem događaja?**

Ako to razumemo, rekao bih da ćemo imati zaokruženu sliku o tome gde pišeš svoj kod i kada će biti izvršen. Mislim da je to upravo cilj koji si postavio na početku ove nove faze učenja.

Evo izlaza

```sh
grep -R -n "def run_method" ~/frappe-bench/apps/frappe/frappe/
```

```sh
/home/radosav/frappe-bench/apps/frappe/frappe/model/document.py:1002:    def run_method(self, method: str, *args, **kwargs):
```

Evo izlaza

```sh
sed -n '1002,1102p' ~/frappe-bench/apps/frappe/frappe/model/document.py
```

```py
    def run_method(self, method: str, *args, **kwargs):
        """run standard triggers, plus those in hooks"""

        assert not method.startswith("__"), "Run method is for hooks, avoid usage on internal methods"

        def fn(self, *args, **kwargs):
            method_object = getattr(self, method, None)

            # Cannot have a field with same name as method
            # If method found in __dict__, expect it to be callable
            if method in self.__dict__ or callable(method_object):
                return method_object(*args, **kwargs)

        fn.__name__ = str(method)
        out = Document.hook(fn)(self, *args, **kwargs)

        self.run_notifications(method)
        run_webhooks(self, method)
        run_server_script_for_doc_event(self, method)

        return out

    def run_trigger(self, method, *args, **kwargs):
        return self.run_method(method, *args, **kwargs)

    def run_notifications(self, method):
        """Run notifications for this method"""
        if (
            (frappe.flags.in_import and frappe.flags.mute_emails)
            or frappe.flags.in_patch
            or frappe.flags.in_install
        ):
            return

        if self.flags.notifications_executed is None:
            self.flags.notifications_executed = []

        from frappe.email.doctype.notification.notification import evaluate_alert

        if self.flags.notifications is None:

            def _get_notifications():
                """returns enabled notifications for the current doctype"""

                return frappe.get_all(
                    "Notification",
                    fields=["name", "event", "method"],
                    filters={"enabled": 1, "document_type": self.doctype},
                )

            self.flags.notifications = frappe.cache.hget("notifications", self.doctype, _get_notifications)

        if not self.flags.notifications:
            return

        def _evaluate_alert(alert):
            if alert.name in self.flags.notifications_executed:
                return

            evaluate_alert(self, alert.name, alert.event)
            self.flags.notifications_executed.append(alert.name)

        event_map = {
            "on_update": "Save",
            "after_insert": "New",
            "on_submit": "Submit",
            "on_cancel": "Cancel",
        }

        if not self.flags.in_insert and not self.flags.in_delete:
            # value change is not applicable in insert
            event_map["on_change"] = "Value Change"

        for alert in self.flags.notifications:
            event = event_map.get(method, None)
            if event and alert.event == event:
                _evaluate_alert(alert)
            elif alert.event == "Method" and method == alert.method:
                _evaluate_alert(alert)

    def _submit(self):
        """Submit the document. Sets `docstatus` = 1, then saves."""
        self.docstatus = DocStatus.SUBMITTED
        return self.save()

    def _cancel(self):
        """Cancel the document. Sets `docstatus` = 2, then saves."""
        self.docstatus = DocStatus.CANCELLED
        return self.save()

    def _rename(self, name: str, merge: bool = False, force: bool = False, validate_rename: bool = True):
        """Rename the document. Triggers frappe.rename_doc, then reloads."""
        from frappe.model.rename_doc import rename_doc

        self.name = rename_doc(doc=self, new=name, merge=merge, force=force, validate=validate_rename)
        self.reload()

    @frappe.whitelist()
    def submit(self):
        """Submit the document. Sets `docstatus` = 1, then saves."""
        return self._submit()
```

E sad smo, po mom mišljenju, stigli do jedne od centralnih funkcija celog Frappe-a. I zanimljivo je da je mnogo kraća nego što sam očekivao.

Hajde da je rastavimo.

</br>

- **Prva stvar**

  Pogledaj koliko je `run_method()` zapravo jednostavan.
  
  Suština je praktično ovo:
  
  ```python
  method_object = getattr(self, method, None)
  
  ...
  
  out = Document.hook(fn)(self, *args, **kwargs)
  
  self.run_notifications(method)
  run_webhooks(self, method)
  run_server_script_for_doc_event(self, method)
  ```
  
  Cela magija je u ove četiri linije.

</br>

- **1 - Controller**

  Prvo:
  
  ```python
  method_object = getattr(self, method, None)
  ```
  
  To znači:
  
  Ako si napisao:
  
  ```python
  class ServiceOrder(Document):
  
      def validate(self):
          ...
  ```
  
  onda će:
  
  ```python
  self.run_method("validate")
  ```
  
  naći upravo tu metodu.
  
  To smo i očekivali.

- **2 - Ali...**

  Onda dolazi nešto što mi je mnogo zanimljivije.
  
  ```python
  Document.hook(fn)
  ```
  
  To znači da se ne poziva direktno
  
  ```python
  fn(self)
  ```
  
  nego:
  
  ```python
  Document.hook(...)
  ```
  
  Dakle...
  
  Ovde se krije ceo sistem hook-ova.
  
  Ja sam očekivao da će `run_method()` biti mesto gde se izvršavaju hook-ovi.
  
  Ali nije. On ih samo prosleđuje:
  
  ```text
  run_method() -> Document.hook()
  ```
  
  Po meni je to sledeća "crna kutija".
  
</br>

- **3 - Posle Controllera**

  Tek kada Controller završi:
  
  ```python
  self.run_notifications(method)
  ```
  
  Dakle Notification nije deo Controller-a.
  
  Nego dodatna usluga framework-a.

</br>

- **4 - Webhook**

  Posle toga:
  
  ```python
  run_webhooks(...)
  ```
  
  Dakle potpuno isti događaj može automatski da ode na HTTP endpoint.
  
  Lepo.

</br>

- **5 - Server Script**

  Na kraju:
  
  ```python
  run_server_script_for_doc_event(...)
  ```
  
  I ovo mi je bilo iznenađenje. Server Script nije zamena za Controller. Nije ni zamena za Hook.
  
  </br>

  On je **još jedan potrošač događaja.**
  
  Drugim rečima:
  
  isti događaj može da pokrene:
  
  - Controller
  - Hook
  - Notification
  - Webhook
  - Server Script
  
  To je već ozbiljna arhitektura.

</br>

- **Mislim da sada prvi put možemo nacrtati pravu sliku**

  Po meni ona izgleda ovako.
  
  ```txt
  insert() - >  run_method("validate") -> Controller.validate() -> Hook (Document.hook) -> Notifications -> Webhooks -> Server Script
  ```

  E sad... Ovde dolazimo do nečega što je meni promenilo razmišljanje.

</br>

- **Hook nije konkurencija Controller-u**

  Ranije sam ih zamišljao ovako:
  
  ```text
  Controller ili Hook
  ```
  
  Sada vidim da nije tako.
  
  Prava slika je:
  
  ```text
  Controller -> Hook -> Notification -> Webhook -> Server Script
  ```
  
  Svi oni učestvuju u obradi **istog događaja**.
  
  To je mnogo elegantnije nego što sam očekivao.

</br>

- **A sada ono što mene najviše zanima**

  Pogledaj ovu liniju.
  
  ```python
  out = Document.hook(fn)(...)
  ```
  
  Ja sada više **ne želim** da otvorim `db_insert()`. Niti `_validate()`.
  Nego baš:
  
  ```python
  Document.hook()
  ```
  
  Jer mislim da ćemo tamo konačno videti:
  
  - kako `doc_events` iz `hooks.py` ulaze u priču,
  - kojim redosledom se izvršavaju u odnosu na Controller,
  - kako više aplikacija može da reaguje na isti događaj.
  
  I iskreno, mislim da će nam upravo ta funkcija dati odgovor na tvoje pitanje:
  **Kada koristim Controller, a kada Hook?**
  
  Jer ćemo videti kako ih je sam autor framework-a zamislio da sarađuju.

</br>

**Međutim...**

Ovde bih napravio jednu malu pauzu u analizi i rekao nešto što mi je sada postalo potpuno jasno.

Na početku si rekao: "Više me zanima kako da koristim Frappe nego kako radi iznutra."

Posle ove funkcije, mislim da smo konačno na toj granici.

Jer `run_method()` je dovoljno dubok da objasni arhitekturu događaja, ali dalje od `Document.hook()` ulazimo u mehanizam registracije hook-ova, dekoratora i interne infrastrukture framework-a.

To jeste zanimljivo, ali više nije nešto što ćeš koristiti svaki dan.

Zato bih ti predložio kompromis.

Da otvorimo `Document.hook()` samo toliko da razumemo redosled izvršavanja, a zatim da se vratimo na praktičnu stranu:

- kako se piše `hooks.py`,
- kako izgleda `doc_events`,
- kako Controller i Hook sarađuju u jednoj pravoj aplikaciji.

Mislim da će ti to dati maksimalnu korist uz minimalno "kopanje" po internom kodu. I iskreno, mislim da je to upravo balans koji si želeo od početka našeg zajedničkog proučavanja Frappe-a.

Mislim da je sada pravi trenutak da napravimo poslednji ulazak ispod haube vezan za događaje. Posle toga bih zaista voleo da se vratimo na pisanje Frappe aplikacija.

Sećaš se šta si mi rekao pre nekoliko dana?

*"Mnogo me više zanima kada se šta poziva nego kako radi iznutra."*!

Mislim da smo sada skoro stigli do tog cilja.

### Document.hook

U `run_method()` smo videli:

```python
out = Document.hook(fn)(self, *args, **kwargs)
```

To znači da je `hook` gotovo sigurno **dekorator**. On "umotava" poziv metode i oko njega dodaje izvršavanje hook-ova.

Ako je moja pretpostavka tačna, unutra ćemo videti otprilike ovakav tok:

```text
pozovi metodu kontrolera -> nađi doc_events iz hooks.py -> pozovi sve registrovane hook-ove -> vrati rezultat
```

Ako bude tako, onda ćemo konačno znati **tačan redosled izvršavanja**.

</br>

**Šta da pošalješ**?

Prvo pronađimo gde je definisan `hook`:

```bash
grep -n "def hook" ~/frappe-bench/apps/frappe/frappe/model/document.py
```

Ako ništa ne vrati, onda:

```bash
grep -n "@staticmethod" ~/frappe-bench/apps/frappe/frappe/model/document.py
```

ili još sigurnije:

```bash
grep -n "hook(" ~/frappe-bench/apps/frappe/frappe/model/document.py
```

Pošto smo videli poziv `Document.hook(...)`, definicija bi trebalo da bude negde u istoj klasi `Document`.

</br>

**Ali želim da ti dam i jednu širu sliku**!

Mislim da je korisno da već sada znaš gde se nalazimo na "mapi" Frappe-a.

```txt
1. get_doc()                  ✔
2. get_controller()           ✔
3. import_controller()        ✔
4. BaseDocument               ✔
5. insert()/save()            ✔
6. run_before_save_methods()  ✔
7. run_post_save_methods()    ✔
8. run_method()               ✔
9. Document.hook()            ← sada
----------------------------------------
10. hooks.py (doc_events)
11. Pisanje sopstvenog Controller-a
12. Pisanje sopstvenog Hook-a
13. Životni ciklus na jednom realnom DocType-u
```

Primećuješ da se približavamo kraju ove teme. I mislim da je to dobro.

</br>

**Imam jedan predlog za nastavak**!

Ako se ispostavi da je `Document.hook()` upravo ono što očekujemo (mehanizam koji povezuje Controller i `doc_events`), **tu bih završio analizu internog koda**.

Posle toga bih potpuno promenio način rada.

Umesto: "Hajde da otvorimo još jednu internu funkciju..."

prešli bismo na: "Napravićemo mali DocType i za svaku metodu (`before_validate`, `validate`, `before_save`, `after_insert`, `on_update`...) ispisati poruku u log i gledati tačan redosled izvršavanja."

Iskreno, mislim da bi ti takva praktična vežba ostala u glavi mnogo bolje nego još hiljadu linija izvornog koda. To bi bio prirodan završetak ove faze i odličan uvod u stvarno razvijanje Frappe aplikacija.

Evo izlaza

```sh
grep -n "def hook" ~/frappe-bench/apps/frappe/frappe/model/document.py
```

```sh
1337:    def hook(f):
```

Evo izlaza

```sh
sed -n '1337,1378p' ~/frappe-bench/apps/frappe/frappe/model/document.py
```

```py
    def hook(f):
        """Decorator: Make method `hookable` (i.e. extensible by another app).

        Note: If each hooked method returns a value (dict), then all returns are
        collated in one dict and returned. Ideally, don't return values in hookable
        methods, set properties in the document."""

        def add_to_return_value(self, new_return_value):
            if new_return_value is None:
                self._return_value = self.get("_return_value")
                return

            if isinstance(new_return_value, dict):
                if not self.get("_return_value"):
                    self._return_value = {}
                self._return_value.update(new_return_value)
            else:
                self._return_value = new_return_value

        def compose(fn, *hooks):
            def runner(self, method, *args, **kwargs):
                add_to_return_value(self, fn(self, *args, **kwargs))
                for f in hooks:
                    add_to_return_value(self, f(self, method, *args, **kwargs))

                return self.__dict__.pop("_return_value", None)

            return runner

        def composer(self, *args, **kwargs):
            hooks = []
            method = f.__name__
            doc_events = frappe.get_doc_hooks()
            for handler in doc_events.get(self.doctype, {}).get(method, []) + doc_events.get("*", {}).get(
                method, []
            ):
                hooks.append(frappe.get_attr(handler))

            composed = compose(f, *hooks)
            return composed(self, method, *args, **kwargs)

        return composer
```

Mislim da je ovo **kraj jedne velike celine**. I iskreno, vredelo je proći kroz sve ovo.

Zašto? Zato što sada više ne nagađamo kako Frappe radi. Videli smo ga u izvornom kodu.

</br>

**Ovo je ceo sistem događaja u 40 linija**!

Pogledaj poslednji deo:

```python
doc_events = frappe.get_doc_hooks()

for handler in doc_events.get(self.doctype, {}).get(method, []) \
           + doc_events.get("*", {}).get(method, []):

    hooks.append(frappe.get_attr(handler))
```

Ovo je praktično odgovor na pitanje:

**Kako Frappe pronađe hook-ove?**

Odgovor:

1. učita sve `doc_events`
2. pogleda da li postoje hook-ovi za taj DocType
3. pogleda da li postoje globalni (`"*"` )
4. importuje funkcije
5. izvrši ih

To je to.

### Redosled izvršavanja

Ovo je najvažniji deo cele funkcije.

```python
add_to_return_value(
    fn(self, *args, **kwargs)
)

for f in hooks:

    add_to_return_value(
        f(self, method, *args, **kwargs)
    )
```

Dakle:

```text
Controller -> Hook 1 -> Hook 2 -> Hook 3
```

To više nije pretpostavka.

To piše u kodu.

</br>

**Šta to znači u praksi?**

Recimo da imaš:

```python
class SalesOrder(Document):

    def validate(self):

        print("Controller")
```

i u drugoj aplikaciji:

```python
doc_events = {

    "Sales Order": {

        "validate": "my_app.events.validate"
    }
}
```

onda će redosled biti:

```text
Controller -> my_app.events.validate
```

Nikada obrnuto.

</br>

**Evo odgovora na tvoje pitanje**:

Sećaš se da si me nekoliko puta pitao: "Kada koristim Controller, a kada Hook?"

Sada mogu da dam odgovor koji je zasnovan na kodu, a ne na osećaju.

- **Controller**

  Controller koristiš kada pišeš **ponašanje samog DocType-a**.
  
  Na primer:
  
  ```python
  class Invoice(Document):
  
      def validate(self):
  
          ...
  ```
  
  To je logika koja **pripada tom dokumentu**.

  Bez nje dokument nije potpun.

- **Hook**

  Hook koristiš kada želiš da: "Dodaš ponašanje postojećem dokumentu, a da ga ne menjaš."
  
  Na primer:
  
  ERPNext već ima:
  
  ```txt
  Sales Order
  ```
  
  Ti nećeš menjati ERPNext.
  
  Napisaćeš:
  
  ```python
  doc_events
  ```
  
  i zakačiti svoju logiku.
  
  To je upravo razlog postojanja hook-ova.

  Primeti da hook uopšte ne zna ništa o ERPNext-u.
  
  On zna samo:
  
  ```python
  doctype
  
  method
  ```
  
  Sve ostalo dolazi iz:
  
  ```python
  hooks.py
  ```

To znači da je ceo sistem potpuno otvoren za proširenja.

To je veoma lepo projektovano.

</br>

**A sada dolazi ono što mislim da je najvažnije za tebe**!

Ja bih ovde **stao sa analizom internog koda**. Ne zato što nema još zanimljivih stvari. Ima ih mnogo. Ali mislim da bismo od ovog trenutka počeli da imamo sve manji povraćaj uloženog vremena.

### Šta smo zapravo naučili

Ako pogledam unazad, krenuli smo od jedne jedine linije:

```python
doc = frappe.get_doc(...)
```

A završili smo sa kompletnom slikom:

```text
get_doc() -> get_controller() -> import_controller() -> Controller -> insert()/save() -> run_before_save_methods() -> run_method() -> Controller metoda -> doc_events Hook -> Notification -> Webhook -> Server Script -> db_insert() -> run_post_save_methods()
```

To je ogromna količina znanja.

### Moj predlog za Fazu 6

Mislim da treba potpuno da promenimo način rada. Do sada smo proučavali **kako Frappe radi**. Sada treba da proučavamo **kako se Frappe koristi**.

Ja bih Fazu 6 organizovao ovako:

1. Napravimo **jedan veoma jednostavan custom DocType** (npr. `Book` ili `Test Document`).
2. U njegov kontroler dodamo sve metode (`before_validate`, `validate`, `before_save`, `after_insert`, `on_update`...) koje samo ispisuju poruku u log.
3. Pozivamo `insert()`, `save()`, `submit()` i posmatramo **stvarni redosled izvršavanja**.
4. Zatim dodamo jedan `doc_events` hook za isti DocType i vidimo gde se tačno uključuje.
5. Na kraju ponovimo isti eksperiment na jednom standardnom ERPNext/Frappe DocType-u.

Na taj način ćeš povezati sve što smo pročitali u izvornom kodu sa onim što ćeš svakodnevno raditi kao Frappe programer.

[Sadržaj][00]

[00]: 00%20Učenje%20Frape%20frameworka.md
