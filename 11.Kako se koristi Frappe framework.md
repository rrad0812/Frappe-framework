# Učenje Frappe Framework

[Sadržaj][00]

## 11 Korišćenje Frappe frameworka od strane programera

---

Ne znam da li je ovo najblji način za jedno običnog programera da shvati kako treba da koristi FF ali sam ja izgubio najvažniju nit:

Koji način koristim kada kreiram dokument u kojoj situaciji. Nisi mi baš najjasniji bio do kraja.

---

Dobro je što si ovo rekao, jer mislim da si pogodio dve stvari u centar.

Prvo, mislim da sam napravio grešku u načinu na koji vodim ovo proučavanje.

Mi smo se toliko fokusirali na pitanje **kako Frappe radi?** da smo izgubili pitanje koje je za programera mnogo važnije:

**Šta ja kao programer treba da koristim?**

A to je zapravo ono zbog čega učiš Frappe.

Hajde da vratimo fokus**!

Da sam danas seo pored nekoga ko tek uči Frappe, rekao bih mu da postoje četiri osnovna scenarija rada sa dokumentima.

### Kreiranje novog dokumenta

**Koristiš**:

```python
doc = frappe.new_doc("Customer")
```

ili

```python
doc = frappe.get_doc({
    "doctype": "Customer",
    "customer_name": "ABC Ltd"
})
```

**Kada?**

Kada praviš potpuno novi dokument koji još ne postoji u bazi.

Razlika između ova dva je uglavnom u praktičnosti:

```py
doc = frappe.new_doc("Customer")
doc.customer_name = "ABC Ltd"
doc.customer_group = "Commercial"
```

naspram:

```python
doc = frappe.get_doc({
    "doctype": "Customer",
    "customer_name": "ABC Ltd",
    "customer_group": "Commercial"
})
```

Na kraju oba završe sa:

```python
doc.insert()
```

### Učitavanje postojećeg dokumenta

**Koristiš:**

```python
doc = frappe.get_doc("Customer", "CUST-00001")
```

Ovo je potpuno druga priča.

Ovde Frappe odlazi u bazu, pročita podatke i vrati objekat.

### Brzo čitanje podataka

Ako ti ne treba ceo objekat:

```python
name = frappe.db.get_value(...)
```

ili

```python
rows = frappe.get_all(...)
```

Zašto?

Zato što je mnogo brže.

Ne pravi `Document`, ne pokreće dodatnu logiku.

### Direktan SQL

Tek kada ORM nije dovoljan.

### Hijerarhija pristupa

Ja bih to danas nacrtao ovako:

```text
Treba mi dokument?

│
├── Ne
│      │
│      └── get_all()
│          get_value()
│          SQL
│
└── Da
       │
       ├── već postoji?
       │        │
       │        └── get_doc()
       │
       └── novi?
                │
                ├── new_doc()
                └── get_doc({...})
```

### Filozofija Frappea

- sve je `Document`,
- svaki `Document` ima `Meta`,
- ponašanje dolazi iz kombinacije metapodataka i Python klase,
- gotovo sve prolazi kroz isti životni ciklus (`insert`, `save`, `submit`, `cancel`...),
- hook-ovi i događaji omogućavaju proširenje bez menjanja jezgra framework-a.

To je već sasvim zaokružen model.

</br>

**Mislim da bih promenio i način rada**:

Imam jedan predlog koji će, po mom mišljenju, biti mnogo korisniji za nastavak.

Umesto da nastavimo da čitamo funkciju po funkciju, hajde da svaku novu temu počnemo pitanjem:

"Šta programer želi da uradi?"

Na primer:

- Želim da napravim novi dokument.
- Želim da učitam postojeći.
- Želim da dodam red u child tabelu.
- Želim da presretnem `save()`.
- Želim da napravim svoj DocType.
- Želim da napišem API.
- Želim da napravim Workflow.
  
Tek onda ćemo pratiti kako Frappe to interno ostvaruje.
  
Mislim da ćeš tako mnogo lakše povezati API koji koristiš svaki dan sa kodom koji smo već proučili.
  
</br>

**O Fazi 5**:

Mislim da "Faza 5" više nije jedna tema. Ona već prirodno sadrži nekoliko poglavlja:

- **5.1 Arhitektura Frappe-a**
- **5.2 Kako nastaje `Document`**
- **5.3 Rad sa DocType kontrolerima**
- **5.4 Životni ciklus dokumenta**
- **5.5 Hook-ovi i događaji**

Ako ostane kao jedna celina od nekoliko hiljada linija, za šest meseci će i tebi biti teško da pronađeš ono što tražiš. Ako je razbiješ na manje logičke celine, dobićeš nešto što će zaista ličiti na lični priručnik za Frappe.

I mislim da bi od sledećeg susreta trebalo da promenimo ugao gledanja:

- prvo "šta kao programer radim"
- pa tek onda kako Frappe to izvodi ispod haube.

Mislim da si sada pogodio **pravo pitanje**. Zapravo, mislim da je ovo pitanje važnije od svega što smo do sada čitali.

I da budem samokritičan: tu sam stvarno ostao nedorečen. Mi smo proučavali **mehanizam**, a nismo odgovorili na pitanje **"gde pišem svoj kod?"**

To je ono što programera zanima.

Hajde da zaboravimo na implementaciju na trenutak. Zamisli da praviš aplikaciju za servis računara.

Pitanje nije:

Kako radi `get_controller()`?

Pitanje je:

</br>

**Gde pišem poslovnu logiku?**

Odgovor je: na više mesta, i svako ima svoju namenu.

- **Controller (DocType klasa)**

  Ovo je prirodno mesto za logiku jednog DocType-a.
  
  Na primer:
  
  ```python
  class ServiceOrder(Document):
  
      def validate(self):
          ...
  
      def before_save(self):
          ...
  
      def on_submit(self):
          ...
  ```
  
  Ovde pišeš sve što je sastavni deo tog DocType-a.
  
  Na primer:
  
  - izračunavanje ukupne cene,
  - provera da li je serijski broj unet,
  - automatsko postavljanje statusa,
  - validacija datuma.
  
  Drugim rečima: Ako ta logika pripada tom dokumentu, piše se u kontroleru.

</br>

- **Hooks**

  E sad dolazi ono što je mene dugo bunilo kada sam prvi put gledao Frappe.
  
  Hook nije zamena za kontroler.
  
  Hook kaže: "Ne mogu ili ne želim da menjam postojeći DocType, ali želim da reagujem na njegove događaje."
  
  Na primer.
  
  ERPNext već ima:
  
  ```text
  Sales Invoice
  ```
  
  Ti nećeš da menjaš ERPNext.
  
  Ali želiš: "Posle svakog submit-a pošalji podatke u eksterni ERP."
  
  Ne diraš:
  
  ```python
  SalesInvoice(Document)
  ```
  
  nego u `hooks.py` napišeš:
  
  ```python
  doc_events = {
      "Sales Invoice": {
          "on_submit": "my_app.api.export_invoice"
      }
  }
  ```
  
  To je potpuno druga filozofija.

</br>

- **Controller i Hook nisu konkurenti**

  Mislim da je ovo najvažnija rečenica.
  
  Ja sam ih dugo posmatrao kao:
  
  ```text
  Controller
  
  ILI
  
  Hook
  ```
  
  A sada mislim da je ispravno:
  
  ```text
  Controller + Hook
  ```
  
  Controller je **vlasnik DocType-a**.
  
  Hook je **posmatrač**.
  
  - **Override**
  
  Treća mogućnost.
  
  Ako kažeš:
  
  ```python
  override_doctype_class = {
      "Customer": ...
  }
  ```
  
  onda praktično kažeš: Nemoj koristiti Frappe-ov ili ERPNext-ov kontroler.
  
  Koristi moj.
  
  Ovo je ozbiljna intervencija.
  
  Koristi se retko.
  
</br>

- **Server Script**

  Ako želiš malu logiku bez pravljenja aplikacije.
  
  Na primer:
  
  - automatski popuni polje,
  - zabrani nešto,
  - pošalji mail.
  
  To nije zamena za ozbiljan razvoj, ali je zgodno za manje prilagođavanje.
  
  - **Ja bih to danas nacrtao ovako**
  
  ```text
  Da li je ovo logika mog DocType-a?
  
          │
          ├── DA
          │      │
          │      └── Controller
          │
          └── NE
                 │
                 ├── samo reagujem?
                 │         │
                 │         └── Hook
                 │
                 ├── menjam tuđi DocType?
                 │         │
                 │         └── Override
                 │
                 └── mala administrativna logika
                           │
                           └── Server Script
  ```

</br>

### U kom trenutku izvršenja koda se šta poziva?

E to je, po mom mišljenju, sledeća velika tema.

Ne interesuje te samo gde pišeš kod.

Nego: **Kada će taj kod biti pozvan?**?

Na primer, ako uradiš:

```python
doc.insert()
```

šta se desi?

Otprilike:

```text
insert() -> before_insert -> before_validate -> validate -> before_save -> INSERT INTO database -> after_insert -> on_update -> on_change
```

**Napomena**: ovo je pojednostavljen redosled. Sledeći put ćemo ga proveriti u izvornom kodu i napraviti tačan dijagram.

Mislim da je upravo taj redosled ono što će ti najviše koristiti u svakodnevnom radu.

Jer tada više nećeš razmišljati: "Da li ovo ide u validate?" nego: "Treba mi da se izvrši pre nego što ode u bazu." ili "Treba mi da se izvrši samo kada je dokument prvi put napravljen."

To je mnogo prirodniji način razmišljanja.

### Moj predlog

Mislim da je vreme da **prestanemo da pratimo sitne pomoćne metode** i da sledeću sesiju posvetimo jednoj temi:

> **Kompletan životni ciklus jednog dokumenta.**

Ne samo `save()`, nego:

- kada se poziva svaki događaj,
- kada se izvršava kontroler,
- kada se izvršavaju hook-ovi,
- kojim redosledom,
- šta je bezbedno raditi u kom događaju,
- i, što je meni najvažnije, kada kao programer biraš jedno mesto umesto drugog.

Iskreno, mislim da će ti ta tema dati mnogo više praktičnog znanja nego narednih 500 linija `BaseDocument` koda. To je ono što ćeš koristiti svaki dan dok budeš razvijao Frappe aplikacije.

[Sadržaj][00]

[00]: 00%20Učenje%20Frape%20frameworka.md
