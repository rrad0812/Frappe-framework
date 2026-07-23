# Učenje Frappe Frameworka

[Sadržaj][00]

## 01 Faze projekta

</br>

### Priprema okruženja

- Izabrati verziju Ubuntu-a (24.04 LTS)
- Kreirati novu VM
- Instalirati Ubuntu Server
- Ažurirati sistem
- Instalirati osnovne alate
- Napraviti snapshot Clean System

  Cilj: "Stabilna razvojna mašina.
  
</br>

### Instalacija Frappe okruženja

- Python
- Node.js
- Redis
- PostgreSQL
- Bench
- Kreirati prvi Bench

  Ovde nećemo još praviti nijedan DocType.  
  Cilj je da razumemo: "Šta je Bench?"

</br>

### Anatomija Bencha

  Ovde ne pišemo kod. Samo istražujemo.
  
- Struktura direktorijuma
- apps/
- sites/
- env/
- logs/
- config/
  
  Na kraju ove faze treba da možeš da pogledaš Bench i kažeš: "Znam čemu služi svaki direktorijum."

</br>

### Prvi Site

- napraviti Site
- pokrenuti ga
- otvoriti Desk
- prijaviti se
- pogledati šta je nastalo u PostgreSQL-u
  
  Ovde ćemo prvi put zaviriti u bazu.

</br>

### Anatomija Site-a

  Opet bez programiranja.
  
- site_config.json
- private/
- public/
- assets/
- logs/
  
  Na kraju: "Znam šta je Site."

</br>

### Prva aplikacija

  Tek ovde,
  
- new-app
- instalacija aplikacije
- modul
- prvi DocType

</br>

### Desk

  Ovo je deo koji si već pomenuo da ti je bio nejasan.
  
  Ovde ćemo odgovoriti na pitanja:
  
- Kako Desk vidi moj DocType?
- Kako se pojavljuje u Workspace-u?
- Zašto ga nekad nema?
- Kako ga organizovati?
  
  Mislim da će ova faza biti jedna od najzanimljivijih.

</br>

### DocType

  Ovde konačno ulazimo u razvoj.
  
- polja
- validacija
- child table
- link
- select
- naming

</br>

### ORM

- Python.  
- Ne JavaScript.  
- Ne REST.  
- Samo ORM.

</br>

### Hook-ovi

  Šta se događa kada:
  
- sačuvaš dokument
- obrišeš dokument
- submit
- cancel
  
</br>

### JavaScript

- Client Script.
- Form Script.
- List Script.

</br>

### REST API

  Ovde ćemo ga uporediti sa uAdmin-om.  
  Mislim da će to biti veoma zanimljivo.

</br>

### Bezbednost

- Users
- Roles
- Permissions

</br>

### Deploy

  Tek na kraju.
  
- production
- nginx
- supervisor
- backup

</br>

### Završna faza

  Napravićemo jednu ozbiljniju aplikaciju. Ne "ToDo", ne "Student". Nego nešto što ima smisla.
  
[Sadržaj][00]

[00]: 00%20Učenje%20Frape%20frameworka.md
