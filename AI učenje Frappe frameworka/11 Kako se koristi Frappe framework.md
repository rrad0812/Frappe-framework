# Učenje Frappe Framework

[Sadržaj][00]

## 11 Korišćenje Frappe frameworka od strane programera

---

Ne znam da li je ovo najblji način za jedno običnog programera da shvati kako treba da koristi FF ali sam ja izgubio najvažniju nit:

Koji način koristim kada kreiram dokument u kojoj situaciji. Nisi mi baš najjasniji bio do kraja.

---

Dobro je što si ovo rekao, jer mislim da si pogodio dve stvari u centar.

Prvo, mislim da sam napravio grešku u načinu na koji vodim ovo proučavanje. Mi smo se toliko fokusirali na pitanje **kako Frappe radi?** da smo izgubili pitanje koje je za programera mnogo važnije:

### Šta programer treba da koristi

A to je zapravo ono zbog čega učiš Frappe.

Hajde da vratimo fokus**!

Da sam danas seo pored nekoga ko tek uči Frappe, rekao bih mu da postoje četiri osnovna scenarija rada sa dokumentima.

1. **Kreiranje novog dokumenta**

   Koristiš:

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

   Kada?: Kada praviš potpuno novi dokument koji još ne postoji u bazi.

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

2. **Učitavanje postojećeg dokumenta**

   Koristiš:

   ```python
   doc = frappe.get_doc("Customer", "CUST-00001")
   ```

   Ovo je potpuno druga priča. Ovde Frappe odlazi u bazu, pročita podatke i vrati objekat.

3. **Brzo čitanje podataka**

   Ako ti ne treba ceo objekat:

   ```python
   name = frappe.db.get_value(...)
   ```

   ili

   ```python
   rows = frappe.get_all(...)
   ```

   Zašto? Zato što je mnogo brže. Ne pravi `Document`, ne pokreće dodatnu logiku.

4. **Direktan SQL**

   Tek kada ORM nije dovoljan.

### Hijerarhija pristupa

Treba mi dokument?

Ja bih to danas nacrtao ovako:

```text
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

Hajde da zaboravimo na implementaciju na trenutak. Zamisli da praviš aplikaciju za servis računara.

Pitanje nije: Kako radi `get_controller()`?  
Pitanje je: Gde pišem poslovnu logiku?  
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

  Hook nije zamena za kontroler. Hook kaže: "Ne mogu ili ne želim da menjam postojeći DocType, ali želim da reagujem na njegove događaje."
  
  Na primer, ERPNext već ima:
  
  ```text
  Sales Invoice
  ```
  
  Ti nećeš da menjaš ERPNext. Ali želiš: "Posle svakog submit-a pošalješ podatke u eksterni ERP."
  
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
  Controller ili Hook
  ```
  
  A sada mislim da je ispravno:
  
  ```text
  Controller + Hook
  ```
  
  Controller je vlasnik DocType-a. Hook je posmatrač.
  
  - **Override**
  
  Treća mogućnost.
  
  Ako kažeš:
  
  ```python
  override_doctype_class = {
      "Customer": ...
  }
  ```
  
  onda praktično kažeš: Nemoj koristiti Frappe-ov ili ERPNext-ov kontroler Koristi moj. Ovo je ozbiljna intervencija.
  
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

... Prvo, hvala što si podelio repozitorijum. Pogledaću ga sa zadovoljstvom:

[rrad0812/Frappe-framework na GitHub-u](https://github.com/rrad0812/Frappe-framework?utm_source=chatgpt.com)

To mi je korisno iz dva razloga:

- moći ću da vidim kako organizuješ beleške i dokumentaciju,
- a još važnije, videću šta smo već obradili da ne bismo ponavljali ili preskakali stvari.

Umesto da sledeću temu bude: "Hajde da čitamo `save()`.", predlažem da bude:

#### Kako izgleda razvoj jedne Frappe aplikacije iz ugla programera

Drugim rečima, svaku temu ćemo posmatrati kroz tri pitanja:

1. **Šta želim da uradim?**
2. **Koji Frappe API koristim?**
3. **Kada se izvršava moj kod?**

Na primer, za dokumente:

| Želim da...                            | Koristim                                       | Moj kod pišem u... |
| -------------------------------------- | ---------------------------------------------- | ------------------ |
| Napravim novi dokument                 | `frappe.new_doc()` ili `frappe.get_doc({...})` | Controller         |
| Učitam postojeći                       | `frappe.get_doc()`                             | Controller         |
| Reagujem na događaj drugog DocType-a   | `hooks.py` (`doc_events`)                      | Hook               |
| Zamenim ponašanje postojećeg DocType-a | `override_doctype_class`                       | Novi Controller    |
| Samo pročitam podatke                  | `frappe.db.get_value()`, `frappe.get_all()`    | Nema Controller-a  |

Po mom mišljenju, ovakva tabela je vrednija za svakodnevni rad nego deset stranica analize `BaseDocument`.

Na kraju prošlog razgovora došli smo do ovoga:  Kada koristim Controller, a kada Hook?

Ja bih voleo da sledeću celinu posvetimo upravo tome, ali ne teorijski, nego kroz jedan konkretan primer.

Recimo da pravimo DocType:

```text
Service Order
```

i onda rešavamo pitanja jedno po jedno:

- Gde računam ukupnu cenu?
- Gde proveravam da li je serijski broj obavezan?
- Gde šaljem e-mail nakon `submit`?
- Gde sinhronizujem podatke sa drugim sistemom?
- Gde dodajem logiku ako ne smem da menjam ERPNext?

Mislim da će posle takve vežbe razlika između Controller-a, **Hook-ova, Server Script-ova i Override-a postati potpuno prirodna.

#### Prva velika ideja Frappe-a

Ako bih morao da objasnim Frappe u jednoj rečenici, rekao bih:  

Frappe je framework u kome se skoro sve vrti oko Document objekta i njegovog životnog ciklusa.

Dakle, nije centar:

- tabela,
- SQL,
- ORM,
- ili HTTP.

Centar je:

```text
Document
```

Sve ostalo se vrti oko njega.

</br>

Kao programer, šta zapravo radiš? Većinu vremena radićeš jednu od ovih stvari:

- Praviš novi Document

  ```python
  doc = frappe.new_doc("Customer")
  doc.customer_name = "ABC"
  doc.insert()
  ```
  
  ili
  
  ```python
  doc = frappe.get_doc({
      "doctype": "Customer",
      "customer_name": "ABC"
  })
  doc.insert()
  ```

</br>

- Menjaš postojeći

  ```python
  doc = frappe.get_doc("Customer", "CUST-0001")
  
  doc.mobile_no = "064..."
  
  doc.save()
  ```

</br>

- Čitaš podatke

  ```python
  frappe.get_all(...)
  ```
  
  ili
  
  ```python
  frappe.db.get_value(...)
  ```

</br>

- Reaguješ na događaje

  Gde pišem kod?
  
  Ja bih danas nacrtao ovu sliku.
  
  ```text
                      Želim da napišem kod
  
                             │
            ┌────────────────┴────────────────┐
            │                                 │
        Logika pripada                  Logika NE pripada
        ovom DocType-u                 ovom DocType-u
            │                                 │
            ▼                                 ▼
       Controller                        Hook
  ```
  
  To je osnovno pravilo.
  
  </br>

  **Primer**
  
  Napravio si DocType:
  
  ```text
  Service Order
  ```
  
  Sada postavimo nekoliko pitanja.
  
  </br>

  - **Treba izračunati ukupnu cenu.**  
    Gde? U:
  
    ```python
    class ServiceOrder(Document):
    ```
  
    Zašto? Jer je to **sastavni deo Service Order-a**.
    Bez toga dokument nije ispravan.

  </br>

  - **Treba poslati e-mail**
    Posle Submit-a.
    Ovde već postoji dilema. Ako je slanje e-maila deo samog poslovnog procesa Service Order-a, može ići u kontroler.  Ali... Ako praviš aplikaciju koja šalje SMS za više različitih DocType-ova:

    - Sales Invoice
    - Purchase Invoice
    - Service Order
      onda je to već:

      ```text
      Hook
      ```

      Zašto? Jer više nije logika jednog dokumenta.
      To je logika aplikacije.

  </br>

  - **Pravilo**
    Controller odgovara na pitanje: **Kako se ponaša ovaj dokument?**.  
    Hook odgovara na pitanje: **Šta moja aplikacija želi da uradi kada se nešto dogodi?**  

  </br>

  - **Jedan primer**
  
    **Controller:**

    ```python
    class ServiceOrder(Document):
    
        def validate(self):
    
            if self.total == 0:
                frappe.throw(...)
    ```

    To je prirodno.

    **Hook:**

    ```python
    doc_events = {
    
        "Service Order": {
    
            "on_submit": "my_app.sms.send_sms"
    
        }
    
    }
    ```

    Ovo nije deo dokumenta. Ovo je dodatna funkcionalnost aplikacije.

  </br>

  - **A šta je Override?**

    Override kaže: "Ne sviđa mi se postojeći kontroler." Koristi moj.
    To je ozbiljna intervencija.

    Većina projekata ga koristi veoma retko.

### Redosled evaulacije koda

Do sada smo pratili:

```text
    get_doc() ->
    get_controller() ->
...
```

To je bilo korisno jednom. Ali više ne. Od sada ćemo pratiti:

```text
    Programer napiše insert() ->
    Framework pozove before_insert ->
    validate ->
    before_save ->
    SQL ->
    after_insert ->
    ...
```

To je ono što programer zaista mora da zna.

### Predlažem novu "Fazu 5"

Ako bih sada organizovao nastavak našeg učenja, ona bi izgledala ovako:

1. **Životni ciklus dokumenta** (insert, save, submit, cancel)
2. **Controller metode** (šta radi koja i kada se poziva)
3. **Hook-ovi** (kada ih koristiti umesto kontrolera)
4. **Server Script** (kada je dovoljan, kada nije)
5. **Override mehanizmi**
6. **Klijent–server tok** (Desk → API → Document → DB → odgovor)

### Za sledeći korak

Ja bih već sledeći put uzeo jedan poziv:

```python
doc.insert()
```

i odgovorio na pitanje koje si postavio pre nekoliko dana: U kom trenutku izvršenja se šta poziva?

Ne kroz teoriju, nego kroz tačan dijagram poziva. Mislim da će to biti jedna od najkorisnijih lekcija koje ćemo napraviti, jer ćeš posle nje tačno znati gde se piše koja vrsta logike i zašto.

[Sadržaj][00]

[00]: 00%20Učenje%20Frape%20frameworka.md
