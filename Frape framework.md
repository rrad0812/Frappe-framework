# Projekat: Učenje Frappe Framework-a

## Faze projekta

### Faza 0 – Priprema okruženja

- Izabrati verziju Ubuntu-a (24.04 LTS)
- Kreirati novu VM
- Instalirati Ubuntu Server
- Ažurirati sistem
- Instalirati osnovne alate
- Napraviti snapshot Clean System

Cilj: "Stabilna razvojna mašina.
  
### Faza 1 – Instalacija Frappe okruženja

- Python
- Node.js
- Redis
- PostgreSQL
- Bench
- Kreirati prvi Bench

Ovde nećemo još praviti nijedan DocType.
Cilj je da razumemo: "Šta je Bench?"

### Faza 2 – Anatomija Bencha

Ovde ne pišemo kod. Samo istražujemo.

- Struktura direktorijuma
- apps/
- sites/
- env/
- logs/
- config/

Na kraju ove faze treba da možeš da pogledaš Bench i kažeš: "Znam čemu služi svaki direktorijum."

### Faza 3 – Prvi Site

- napraviti Site
- pokrenuti ga
- otvoriti Desk
- prijaviti se
- pogledati šta je nastalo u PostgreSQL-u

Ovde ćemo prvi put zaviriti u bazu.

### Faza 4 – Anatomija Site-a

Opet bez programiranja.

- site_config.json
- private/
- public/
- assets/
- logs/

Na kraju: "Znam šta je Site."

### Faza 5 – Prva aplikacija

Tek ovde,

- new-app
- instalacija aplikacije
- modul
- prvi DocType

### Faza 6 – Desk

Ovo je deo koji si već pomenuo da ti je bio nejasan.

Ovde ćemo odgovoriti na pitanja:

- Kako Desk vidi moj DocType?
- Kako se pojavljuje u Workspace-u?
- Zašto ga nekad nema?
- Kako ga organizovati?

Mislim da će ova faza biti jedna od najzanimljivijih.

### Faza 7 – DocType

Ovde konačno ulazimo u razvoj.

- polja
- validacija
- child table
- link
- select
- naming

### Faza 8 – ORM

- Python.  
- Ne JavaScript.  
- Ne REST.  
- Samo ORM.

### Faza 9 – Hook-ovi

Šta se događa kada:

- sačuvaš dokument
- obrišeš dokument
- submit
- cancel
  
### Faza 10 – JavaScript

- Client Script.
- Form Script.
- List Script.

### Faza 11 – REST API

Ovde ćemo ga uporediti sa uAdmin-om.  
Mislim da će to biti veoma zanimljivo.

### Faza 12 – Bezbednost

- Users
- Roles
- Permissions

### Faza 13 – Deploy

Tek na kraju.

- production
- nginx
- supervisor
- backup

### Faza 14 - Završna faza

Napravićemo jednu ozbiljniju aplikaciju. Ne "ToDo", ne "Student". Nego nešto što ima smisla.

Ono što bih dodao, voleo bih da uz svaku fazu imamo i status. Recimo ovako:

| Faza | Status |
| ---- | ------ |
| 0. Priprema okruženja | ⬜ |
| 1. Instalacija | ⬜ |
| 2. Bench | ⬜ |
| 3. Site | ⬜ |
| 4. Anatomija Site-a | ⬜ |
| 5. Aplikacija | ⬜ |
| 6. Desk | ⬜ |
| 7. DocType | ⬜ |
| 8. ORM | ⬜ |
| 9. Hook-ovi | ⬜ |
| 10. JavaScript | ⬜ |
| 11. REST API | ⬜ |
| 12. Bezbednost | ⬜ |
| 13. Deploy | ⬜ |
| 14. Završni projekat | ⬜ |

### Jedna dopuna

Dodao bih i jednu "Fazu -1" koja se ne tiče instalacije nego razumevanja. Pre nego što instaliramo ijedan paket, voleo bih da odgovorimo na jedno pitanje:

- **Zašto Frappe uopšte ima Bench?**  
  Većina korisnika samo prihvati da postoji komanda bench, ali retko ko razume zašto nije jednostavno pip install frappe. Mislim da će nam upravo odgovor na to pitanje dati dobar uvod u arhitekturu frameworka i učiniti sve naredne korake logičnijim. Po mom mišljenju, to je pravo mesto odakle treba da počnemo.

## Faza 0 - Priprema okruženja

### Preuzimnje ISO slike i instalacija

Sa adrese <https://releases.ubuntu.com/24.04.4/ubuntu-24.04.4-live-server-amd64.iso> preuzeti ISO za `Ubuntu server 24.04`.
  
### Izgradnja Ubuntu Server 24.04 VM

VM izgradnja na QUEMU/KVM root sesiji sa sledećim parametrima:

- vCPU 2
- RAM 8GB
- SSD 60GB

### Ažuriranje paketa distibucije

```sh
sudo apt update && sudo apt upgrade -y
```

### Instalacija curl-a

```sh
sudo apt install curl
```

### Aktiviranje firewall-a i dozvola ssh pristupa

Aktiairanje firewall-a

```sh
sudo ufw enable
```

Dozvola pristupa preko ssh

  ```sh
  sudo ufw allow OpenSSH
  ```

Proba konekcije na VM ( samo sa localhost-a**

Iz virtuelne mašine pokrenuti:
  
```sh
ip a
```

Vratiće, za KVM slučaj, nešto kao 192.168.122.X.

Sa hosta ssh pristup na VM

```sh
ssh username_na_VM@127.198.122.X
```

Reset VM

```sh
sudo shutdown -r now
```

### Promena vremenske zone i sync. vremena

```sh
sudo timedatectl set-timezone Europe/Belgrade
sudo timedatectl set-ntp on
```

### Promena locale

```sh
sudo dpkg-reconfigure locales
```

Dodaj nove locale `sr_RS@latin UTF-8` i postavi ih za default.

Reset VM

```sh
sudo shutdown -r now
```

### Instalacija osnovnih dev paketa

```sh
sudo apt python3-dev
```

### Instalacija uv pajton paket i runtime managera

```sh
curl -LsSf https://astral.sh/uv/install.sh | sh
```

Osveži sesiju:
  
```sh
source "$HOME/.local/bin/env"
```
  
Proveri instaliranost:
  
```sh
uv --version
```

### Šta je urađeno u Fazi 0

- Instaliran Ubuntu Server 24.04 sa OpenSSH serverom.
- Instalacija ažurirana na nove verzije paketa
- Promenjen status UFW na enable. Promenjen status OpenSSH na enable. Postignut
  ssh pristup sa localhosta na VM.
- Promenjena vremenska zona na Europe/Belgrade, uradjen sync. vremena
- Promenjen locale na sr_RS@latin UTF-8 i postavljen za podrazumevani.
- Instaliran uv Pajton paket i runtime manger.

## Faza 1 - Instalacija Frappe okruženja

### Instalacija git

```sh
sudo apt install git -y
```

### Instalacija Redisa

```sh
sudo apt install redis-server -y
```

### Instalacija PostgreSQL

```sh
sudo apt install postgresql postgresql-contrib libpq-dev -y
```

Provera instalirane verzije

 ```sh
 psql --version
 ```

Prelazak na `postgres` nalog  

To je podrazumevani admin korisnik PostgeSQL i instaliran je na sistem za vreme instalacije PostgrSQL-a.

```sh
sudo -i -u postgres
```

Pokretanje PostgreSQL shela

```sh
psql
```

Izlazak iz shela:

```sh
\q
```

Na psql možeš doći kao `postgres` bez prelaska sa svog naloga:

```sh
sudo -u postgres psql
```

Dodela passworda postgres korisniku:

```sql
ALTER USER postgres WITH PASSWORD 'postgres_password';
```
  
### Instalacija web servera - nginx

```sh
sudo apt install nginx
```

### Instalacija wkhtmltopdf
  
```sh
sudo apt install xvfb libfontconfig
```

Preuzmi wkhtmltopdf paket sa <https://wkhtmltopdf.org/downloads.html>, potom pokreni sledeću komandu za instalaciju:

```sh
sudo apt install ./wkhtmltox_file.deb
```
  
### Instalacija frontend zavisnosti

Instalacija **nvm**

```sh
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash
```

Instalacija **nodejs**

```sh
nvm install 24
```

Provera instaliranosti

```sh
node -v
```

Instalacija **yarn**

```sh
npm install -g yarn
```
  
### Instalacija Bench-a (Frappe v15)  

```sh
uv tool install frappe-bench --with setuptools
```

**Napomena**:  
Dodali smo sa `--with setuptools` jer Frappe u pozadini još uvek koristi neke starije Python mehanizme za pakete.

Sada proveri da li sistem vidi komandu:

```sh
bench --version
```

### Instalacija i inicijalizacija frappe-bench direktorijuma

Sada kada imamo bench alat, pravimo glavni folder gde će ti biti svi sajtovi i aplikacije (izaberi verziju 15):

```sh
bench init frappe-bench --frappe-branch version-15
```

Uđi u kreirani direktorijum:

```sh
cd frappe-bench
```

Sada se nalaziš u glavnom upravljačkom čvorištu svog projekta.

### Šta je urađeno u Fazi 1

- Instaliran je git.
- Instaliran je Redis.
- Instaliran je PostgreSQL.
- Instaliran je wkhtmltopdf.
- Instalirane su front-end zavisnosti:
  - npm
  - nodejs
  - yarn
- Instaliran je bench.
- Instaliran i inicijalizovan je frappe-bench direktorijum.

## Faza 2 - Kreiranje prvog Frappe sajta

Pošto koristimo PostgreSQL, moramo da uradimo jednu brzu pripremu pre same komande za kreiranje sajta.

Frappe u PostgreSQL-u zahteva posebne ekstenzije (poput `pg_trgm` i `btree_gin`). Da bi ih Frappe-ov korisnik uspešno kreirao tokom instalacije, najsigurnije je da ih tvoj `postgres` superuser ima već aktivne na sistemskom nivou (u šablonu `template1` iz kojeg se prave sve nove baze).

### Priprema PostgreSQL šablona baze podataka

Pokreni ovu komandu da omogućiš potrebne ekstenzije u podrazumevanom šablonu baze:

  ```sh
  sudo -u postgres psql -d template1 -c "CREATE EXTENSION IF NOT EXISTS pg_trgm; CREATE EXTENSION IF NOT EXISTS btree_gin;"
  ```

### Kreiranje novog sajta

Sada, dok si unutar "frappe-bench" direktorijuma, pokreni komandu za kreiranje sajta. Nazvaćemo ga npr. "site1.local" (možeš staviti bilo koje ime koje se završava sa .localhost ili .local za lokalni razvoj):

```sh
bench new-site site1.local --db-type postgres
```

Tokom izvršavanja ove komande, Bench će te pitati za dve stvari:

- MySQL root password  
Iako piše MySQL, pošto si stavio `--db-type postgres`, ovde unosiš lozinku za `postgres` korisnika koju si postavio u Fazi 1.

- Administrator password  
Ovo je lozinka za glavnog admina samog Frappe web interfejsa. Postavi neku lozinku po želji (npr. admin123 za lokalni rad) i zapamti je.

Kako da proveriš da li je uspelo?

Kada komanda završi (ovaj put uspešno), proveri da li baza zaista postoji u PostgreSQL-u:

```sh
sudo -u postgres psql -c "\l"
```

Trebalo bi da vidiš novu bazu sa čudnim, nasumičnim imenom koje počinje sa podvlakom (npr. _1a2b3c4d5e...), jer Frappe namerno generiše hešovana imena baza radi bezbednosti.

### Digresija: Rešenje problema sa konekcijom na PostgreSQL

Moramo da dozvolimo svim korisnicima (`all`) da se povežu na sve baze (`all`) preko 127.0.0.1 (localhost-a) interfejsa koristeći lozinku.

Podrazumevano PostgeSQL je podešen na `peer` konkcije (`socket`) sa localhosta, i to je način na koji pristupa psql. Za Frappe pristup, preko mrežnog interfejsa potrebno je prepodesiti u '/etc/postgresql/16/main/pg_hba.conf' konfiguracionm fajlu IPv4 pravilo, tako da glasi:

```txt
# IPv4 local connections:
host    all             all             127.0.0.1/32            md5
```

Promenili smi i tip protkola lozinke, jer se u praksi pokazalo da radi!

### Restart PostgreSQL servisa

Posle promene konfig. fajllova, restartuj PostgreSQL servis:

```sh
sudo systemctl restart postgresql
```
  
### Šta je urađeno u Fazi 2

- U PostgreSQL `template1` šablon dodate ekstenzije: `pg_trgm` i `btree_gin`.
- U fajlu `/etc/postgresql/16/main/pg_hba.conf` promenjena autentifikacija za IPv4 na `md5` za sve korisnike (`all all 127.0.0.1/32`), kako bi Frappe mogao mrežnim putem da kreira baze i korisnike.

- Uspešno kreiran prvi sajt na PostgreSQL bazi podataka pod imenom `site1.  
  local` komandom:

  ```sh
  `bench new-site site1.local --db-type postgres`
  ```

## Faza 3 - Anatomija frappe-a

### `frappe-bench` dir

Uradi u terminalu:

```bash
pwd
ls -lah
```

i pošalji izlaz.

Ne zato što ne znam kako izgleda Bench, nego zato što želim da pričamo o tvom okruženju. Različite verzije Bench-a i Frappe-a umeju da imaju sitne razlike, a usput ćemo videti i da li je sve nastalo kako očekujemo.

Posle toga ćemo uzeti jedan direktorijum po jedan i objasniti čemu služi. Tek kada budeš imao mentalnu mapu Bench-a, preći ćemo na `bench start`, procese, logove, konfiguraciju i dalje na razvoj aplikacija.

Mislim da će ti se taj pristup dopasti više nego da odmah krenemo sa novim komandama. Razumevanje strukture na početku mnogo olakšava sve što sledi.

---

Evo izlaza:

```sh
ls -alh  
total 36K  
drwxrwxr-x 7 radosav radosav 4,0K jul 11 00:35 .  
drwxr-x--- 8 radosav radosav 4,0K jul 11 00:37 ..  
drwxrwxr-x 3 radosav radosav 4,0K jul 11 00:35 apps  
drwxrwxr-x 3 radosav radosav 4,0K jul 11 00:37 config  
drwxrwxr-x 6 radosav radosav 4,0K jul 11 00:36 env  
drwxrwxr-x 2 radosav radosav 4,0K jul 11 01:04 logs  
-rw-rw-r-- 1 radosav radosav  346 jul 11 00:35 patches.txt  
-rw-rw-r-- 1 radosav radosav  278 jul 11 00:35 Procfile  
drwxrwxr-x 4 radosav radosav 4,0K jul 11 01:04 sites  
```

---

Prva zanimljiva stvar je da je ovo mnogo "čistije" nego što ljudi obično očekuju. Mnogi misle da je `frappe-bench` neka ogromna struktura, a zapravo je vrlo pregledna.

Hajde da prođemo kroz ovo, ali ne samo da nabrojimo direktorijume, već da razumemo njihovu ulogu.

```sh
frappe-bench/
├── apps/
├── config/
├── env/
├── logs/
├── sites/
├── Procfile
└── patches.txt
```

#### apps/

Ovo je verovatno najvažniji direktorijum. Ovde živi izvorni kod aplikacija. Na primer:

```sh
apps/
    frappe/
    erpnext/
    payments/
    crm/
    moja_aplikacija/
```

Svaka aplikacija je praktično jedan Git projekat. To znači da ćeš kasnije moći da uradiš nešto poput:

```sh
apps/
    frappe/
        .git

    moja_aplikacija/
        .git
```

i svaka će imati svoju istoriju. To je jedna od lepih osobina Frappe-a.

#### env/  

Ovo je Python virtual environment. Drugim rečima:

```sh
python
pip
bench
frappe
gunicorn
psycopg
...
```

sve živi ovde. Zbog toga ne zagađuješ sistemski Python. To je potpuno isto kao kada napraviš

```bash
python -m venv
```

ili

```bash
uv venv
```

samo što Bench to radi sam.

#### sites/

Ovo je direktorijum koji početnicima pravi najveću zabunu. Ovde nije izvorni kod. Ovde su podaci i konfiguracija sajtova. Na primer:

```sh
sites/
  site1.local/
    site_config.json
    private/
    public/
    locks/
    ...
  assets/
  apps.txt
  common_site_config.json
```

Drugim rečima, **aplikacija ≠ sajt**. To je veoma važna razlika.

Jedna aplikacija može biti instalirana na više sajtova.

Na primer:

```sh
ERPNext
```

može raditi na

```sh
firmaA.local
firmaB.local
firmaC.local
```

bez tri kopije ERPNext-a.

#### config/

Ovo je Bench konfiguracija. Ovde se nalaze konfiguracioni fajlovi koje Bench generiše. Na primer:

- Redis
- Nginx
- Supervisor
- Procfile konfiguracije
- razni JSON fajlovi

Većinu vremena nećeš ručno menjati ove fajlove.

Bench ih uglavnom generiše.

#### logs/

Vrlo koristan direktorijum. Ako nešto ne radi... ovde prvo gledaš.

Npr.

```sh
web.log
worker.log
redis.log
schedule.log
```

Kasnije ćeš dosta vremena provoditi upravo ovde.

#### Procfile

Ovo je zanimljiv fajl. Bench ne pokreće jedan proces. Pokreće ih više. Na primer:

- web server
- scheduler
- worker
- socketio
- watch
- ...

`Procfile` govori Bench-u: "Ovo su procesi koje treba pokrenuti."

Ako si radio sa Heroku ili Foreman, koncept će ti biti poznat.

#### patches.txt

Ovo nije nešto što ćeš često dirati. Koristi se tokom migracija i nadogradnji kako bi Bench znao koje su zakrpe (patches) već primenjene.

#### Globalna oraganizacija

Primeti da Bench veoma lepo odvaja tri stvari:

```sh
Code -> apps/
Data -> sites/
Enviroment -> env/
```

To je veoma elegantna organizacija.

### Pregled apps i sites dirova

Voleo bih da sada "zavirimo" u dva direktorijuma koja su ključna za razumevanje Frappe-a:

Prvo:

```bash
ls -alh apps
```

a zatim:

```bash
ls -alh sites
```

Iz toga ćemo odmah videti:

- koje su aplikacije trenutno instalirane,
- kako Bench razlikuje globalnu konfiguraciju od konfiguracije pojedinačnog sajta,
- i napravićemo jasnu mentalnu sliku odnosa Bench → Apps → Sites.

Mislim da će posle toga cela arhitektura Frappe-a početi da "leže" na svoje mesto.

---

Evo izlaza:

```sh
ls -alh apps
total 12K
drwxrwxr-x  3 radosav radosav 4,0K jul 11 00:35 .
drwxrwxr-x  7 radosav radosav 4,0K jul 11 00:35 ..
drwxrwxr-x 10 radosav radosav 4,0K jul 11 00:37 frappe

ls -alh sites
total 28K
drwxrwxr-x 4 radosav radosav 4,0K jul 11 01:04 .
drwxrwxr-x 7 radosav radosav 4,0K jul 11 00:35 ..
-rw-rw-r-- 1 radosav radosav  192 jul 11 00:37 apps.json
-rw-rw-r-- 1 radosav radosav    6 jul 11 00:37 apps.txt
drwxrwxr-x 5 radosav radosav 4,0K jul 11 00:37 assets
-rw-rw-r-- 1 radosav radosav  492 jul 11 00:35 common_site_config.json
drwxrwxr-x 6 radosav radosav 4,0K jul 11 01:04 site1.local
```

---

Sada ćemo napraviti jednu važnu mentalnu mapu. Ako je razumeš, mnogo ćeš lakše razumeti sve ostalo u Frappe-u.

```text
               Bench
                 │
       ┌─────────┴─────────┐
       │                   │
    Applications         Sites
    (izvorni kod)   (podaci i konfiguracija)
```

#### apps/ dir

Kod tebe trenutno postoji samo jedna aplikacija:

```text
apps/
└── frappe/
```

To znači da si instalirao samo Frappe Framework. To je potpuno očekivano.
Da si instalirao ERPNext, izgledalo bi otprilike ovako:

```text
apps/
├── frappe/
└── erpnext/
```

A kasnije, kada budeš pravio svoju aplikaciju:

```text
apps/
├── frappe/
├── erpnext/
└── moja_aplikacija/
```

Primeti jednu stvar: `apps/` ne zna ništa o `site1.local`.

Tu nema:

- konfiguracije sajta,
- nema baze,
- nema korisnika,
- nema podataka.

Samo kod.  

To je veoma lepo odvajanje odgovornosti.

#### sites/ dir

Ovde se već nalazi mnogo zanimljivijih stvari.

</br>

- **common_site_config.json**  
  Ovo je globalna konfiguracija Bench-a. Ona važi za sve sajtove. Na primer:

  ```text
  site1.local
  site2.local
  demo.local
  ```

  Svi će koristiti ono što je definisano ovde, osim ako neki sajt ne prepiše (override) određenu vrednost.

  To je isti koncept koji postoji u mnogim frameworcima: globalna podešavanja + lokalna podešavanja.

</br>

- **site1.local/**  
  Ovo je jedan konkretan Frappe sajt. Vrlo je važno da ga ne posmatraš kao "projekat". On je više nalik instanci aplikacije. Na primer:
  - ima svoju bazu,
  - svoje korisnike,
  - svoje dokumente,
  - svoje fajlove,
  - svoja podešavanja.
  
  Ako sutra napraviš:
  
  ```bash
  bench new-site firma2.local
  ```

  dobićeš još jedan direktorijum:

  ```text
  sites/
  ├── site1.local/
  └── firma2.local/
  ```

  Oba će koristiti isti kod iz `apps/frappe/`, ali će imati potpuno odvojene podatke.  

  To je jedna od najvećih prednosti Frappe arhitekture.

#### assets/ dir

Ovo često zbuni početnike. Ovde Bench smešta izgrađene (built) statičke resurse. Ne originalni JavaScript. Ne originalni CSS. Već ono što frontend alat napravi nakon build procesa. Drugim rečima:

```sh
apps/
    ... source JS ...
  ↓
bench build
  ↓
sites/assets/
```

Ako dolaziš iz sveta Vite-a, Webpack-a ili Rollup-a, ovo će ti biti poznato.

#### apps.txt

Ovaj fajl izgleda bezazleno. Verovatno sadrži samo:

```text
frappe
```

ali je veoma važan. On govori Bench-u: "Ove aplikacije postoje u ovom Bench okruženju."

Kasnije ćeš ovde videti i:

```text
frappe
erpnext
moja_aplikacija
```

#### apps.json

Ovo je noviji mehanizam koji Bench koristi za dodatne informacije o aplikacijama. U praksi ga retko menjaš ručno; Bench ga održava.
  
#### Ono što treba da zapamtiš

Po mom mišljenju, ovo je najvažnija slika do sada:

```sh
apps/ -> IZVORNI KOD
sites/ -> PODACI
```

### Jedna aplikacija → više sajtova

Zamisli sledeće:

```sh
apps/
    frappe/
    erpnext/
    warehouse_app/
```

i

```sh
sites/
    firmaA.local
    firmaB.local
    firmaC.local
```

Sve tri firme koriste:

- isti Frappe,
- isti ERPNext,
- istu tvoju aplikaciju.

Ali svaka ima:

- svoju PostgreSQL bazu,
- svoje korisnike,
- svoje fakture,
- svoje dokumente.

To je veoma elegantan način za **multi-tenant** arhitekturu.

### Predlog za sledeći korak

Pre nego što pokrenemo razvojni server, voleo bih da pogledamo jedan konkretan sajt. To će nam dati jasnu sliku šta Frappe smatra "sajtom".

Možeš da pošalješ izlaz:

```bash
tree -L 2 sites/site1.local
```

Ako nemaš `tree` instaliran:

```bash
sudo apt install tree
```

ili, bez instalacije:

```bash
find sites/site1.local -maxdepth 2 | sort
```

Posle toga ćemo analizirati `site_config.json`, jer je to jedan od najvažnijih fajlova u celom Frappe ekosistemu. Tu ćeš prvi put videti kako se jedan sajt povezuje sa PostgreSQL bazom i ostatkom sistema.

---

Pre nego pređemo na sledeći korak pitanje, Kažeš da jedna aplikacija može da bude na više sajtova. Da li jedan sajt može da ima više aplikacija?

---

Odgovor je: Da. Zapravo, to je jedan od osnovnih principa Frappe-a.

U stvari, odnos nije 1 aplikacija ↔ 1 sajt, već: više aplikacija ↔ više sajtova.
Možeš to posmatrati kao matricu.

| Aplikacija | site1.local | firmaA.local | firmaB.local |
| ---------- | :---------: | :----------: | :----------: |
| frappe | ✅ | ✅ | ✅ |
| erpnext | ✅ | ✅ | ❌ |
| crm | ❌ | ✅ | ✅ |
| warehouse | ✅ | ❌ | ✅ |

Dakle:

- jedan sajt može imati više aplikacija,
- jedna aplikacija može biti instalirana na više sajtova.
- **Frappe je uvek prva aplikacija**  
  Svaki sajt mora imati instaliranu aplikaciju `frappe`. Ona je osnova svega.
  Na nju se "kače" ostale aplikacije. Na primer:

  ```txt
  site1.local
  
  ├── frappe
  ├── erpnext
  ├── payments
  └── moja_aplikacija
  ```

  Sve četiri rade zajedno kao jedna celina.

- **Kako to izgleda u praksi?**

  Recimo da razvijaš sistem za proizvodnju.
  Napraviš aplikaciju:
  
  ```txt
  manufacturing
  ```
  
  Posle godinu dana razviješ:
  
  ```txt
  quality_control
  ```
  
  Kasnije:
  
  ```txt
  barcode
  ```
  
  Na jednom sajtu možeš imati:
  
  ```txt
  frappe
  erpnext
  manufacturing
  quality_control
  barcode
  ```

  Sve te aplikacije zajedno doprinose funkcionalnosti tog sajta.

- **A šta svaka aplikacija donosi?**

  Svaka može da doda:
  
  - nove DocType-ove,
  - nove stranice,
  - nove API-je,
  - nove izveštaje,
  - nove Workspaces,
  - nove hook-ove,
  - nove JavaScript fajlove,
  - nove Python module,
  - nove migracije.
  
  Drugim rečima, aplikacije se "ugrađuju" u isti Frappe sistem.

  Jedna lepa analogija:
  
  Ja to često zamišljam ovako.
  
  1. Bench je operativni sistem.
  
     ```txt
     Bench
     ```
  
  2. Na njemu postoje aplikacije:
  
     ```text
     Frappe
     ERPNext
     CRM
     Payments
     Moja aplikacija
     ```
  
  3. A jedan sajt je kao jedna instalacija sistema koja bira koji će aplikacije
     koristiti.
  
- **I gde se to pamti?**

  Tu dolazimo do jednog zanimljivog detalja koji ćemo uskoro videti. Za svaki sajt postoji informacija koje su aplikacije na njemu instalirane.
  
  To možeš čak odmah da proveriš:
  
  ```sh
  bench --site site1.local list-apps
  ```
  
  Pošto si napravio potpuno nov sajt, očekujem da će rezultat biti otprilike:
  
  ```txt
  frappe
  ```
  
  Kasnije, kada instaliraš ERPNext:
  
  ```sh
  bench --site site1.local install-app erpnext
  ```
  
  onda će:
  
  ```sh
  bench --site site1.local list-apps
  ```
  
  vratiti:
  
  ```txt
  frappe
  erpnext
  ```
  
  A kada jednog dana napraviš svoju aplikaciju:
  
  ```txt
  frappe
  erpnext
  moja_aplikacija
  ```

- **Jedna mala terminološka napomena**

  Ovde je lako pomešati pojmove jer reč aplikacija u Frappe-u ne znači isto što i u mnogim drugim okvirima.
  
  - **Frappe app** je modul ili paket funkcionalnosti (npr. ERPNext, CRM ili tvoja aplikacija).

  - **Site** je instanca koja ima svoju bazu podataka, konfiguraciju i korisnike.
  
  To znači da jedan site može biti sastavljen od više Frappe aplikacija, a ista aplikacija može biti korišćena na više različitih sajtova. Upravo ta fleksibilnost je jedan od razloga zašto je Frappe pogodan za razvoj većih poslovnih sistema i SaaS rešenja.
  
### Sadržaj sites/site1.local dira

---

Evo izlaza:

```sh
tree -L 2 sites/site1.local  
sites/site1.local  
├── locks  
├── logs  
│   ├── database.log  
│   └── database.log.1  
├── private  
│   ├── backups  
│   └── files  
├── public  
│   └── files  
└── site_config.json  
```

---

Odlično. Ovo izgleda potpuno očekivano za sveže kreiran Frappe sajt.

Hajde da ga "rastavimo" na delove.

```sh
site1.local/
├── locks
├── logs
├── private
├── public
└── site_config.json
```

Odmah možeš da primetiš jednu zanimljivu stvar.

Ovde nema Python koda. Nema:

- `.py`
- `.js`
- `.html`
- DocType definicija

Zašto? Zato što je sav kod u `apps/`, a "site1.local" sadrži samo ono što pripada toj konkretnoj instanci.

To je jedna od najlepših ideja u Frappe-u.

#### site_config.json

Ovo je, po mom mišljenju, najvažniji fajl jednog sajta.

On odgovara na pitanja:

- na koju bazu se povezujem?
- koji Redis koristim?
- gde su fajlovi?
- koje su specifične postavke ovog sajta?

Praktično, kada Frappe "otvori" sajt, prvo pročita ovaj fajl.

Možemo ga detaljno analizirati za nekoliko minuta.

#### private/

Ime govori dosta.
Ovde završavaju stvari koje nisu javno dostupne.  

Na primer:

```txt
private/
  backups/
  files/
```

Kasnije ćeš ovde imati:

- privatne priloge
- backup fajlove
- eksportovane podatke
- razne interne fajlove

Ovo nikada ne treba direktno servirati preko web servera.

#### public/

Suprotno od `private`. Ovde završavaju fajlovi koji mogu biti javno dostupni.
Najčešće:

```text
public/files
```

Na primer:

- logo firme
- slike proizvoda
- PDF koji je dozvoljeno preuzeti
- slike koje korisnici vide

#### logs/ dir

Ovo mi je posebno drago da vidim.

```txt
database.log
database.log.1
```

To znači da svaki sajt ima svoje logove.

Dakle, ako sutra imaš:

```text
site1.local
firmaA.local
firmaB.local
```

svaki može imati sopstvene logove.

To mnogo olakšava administraciju.

#### locks/

Ovaj direktorijum mnogi ni ne primete. Koristi se za razne mehanizme zaključavanja (locking).

Na primer:

- da se dva procesa ne sudare tokom migracije,
- da scheduler ne pokrene isti posao dva puta,
- da se spreče paralelne operacije koje bi dovele do nekonzistentnog stanja.

Većinu vremena će biti prazan. I to je potpuno normalno.

#### Zanimljivost

Ako pogledaš ovaj direktorijum kao celinu:

```sh
site1.local/
```

on uopšte ne izgleda kao aplikacija. Više liči na... profil jednog korisnika sistema.

I to je upravo ono što jeste. Kod je negde drugde.

Ovde su samo:

- konfiguracija,
- podaci,
- fajlovi,
- logovi.

#### site_confg.json

Predložio bih da pogledamo sadržaj:

```bash
cat sites/site1.local/site_config.json
```

To je možda svega desetak redova, ali iz njega ćemo moći da objasnimo:

- kako Frappe pronalazi PostgreSQL bazu,
- kako zna kojim Redis instancama da se poveže,
- kako razlikuje jedan sajt od drugog,
- koje su postavke globalne (`common_site_config.json`), a koje lokalne (`site_config.json`).
- **Jedna mala napomena**
  Do sada smo pričali o arhitekturi **Bench → Apps → Sites**, i mislim da je to bio pravi redosled. Međutim, od sledećeg koraka počećemo da povezujemo te delove u jednu celinu. Videćeš da Bench nije "aplikacija", već pre **orkestrator** koji upravlja Python okruženjem, aplikacijama i sajtovima.  
  Kada to shvatiš, većina `bench` komandi će postati vrlo intuitivna, jer ćeš razumeti **šta** rade, a ne samo **kako** se koriste.

  ---

  ```json
  cat sites/site1.local/site_config.json
  {
   "db_name": "_c9eb2d89e08e8728",
   "db_password": "Oq69qLS1RcrKcUxg",
   "db_type": "postgres"
  }
  ```
  
  ---

  Odlično. Ovo je veoma zanimljiv trenutak jer sada vidiš jednu od stvari koje su te zbunile juče kada smo rešavali PostgreSQL.
  
  Odmah možemo da izvedemo nekoliko zaključaka.
  
##### Zašto baza nema ime `site1.local`?

Verovatno si očekivao nešto poput: "site1.local", ili "site1_local". Međutim, Frappe radi drugačije.  On generiše ime baze: "_c9eb2d89e08e8728".

##### Gde je korisničko ime?

Primeti nešto zanimljivo. Ovde nema: "db_user": "...". Zašto? Zato što kod PostgreSQL-a Frappe koristi isto ime za bazu i korisnika.

Drugim rečima: db_name: "_c9eb2d89e08e8728" i "role": "_c9eb2d89e08e8728" imaju isto ime.

##### Lozinka

Ovu lozinku nisi ti birao. Bench ju je napravio automatski. To je veoma dobra praksa. Svaki sajt dobija:

- svog korisnika
- svoju bazu
- svoju nasumičnu lozinku
  
  </br>

##### db_type

Ovde piše "db_type":"postgres". To znači da ostatak Frappe-a zna koji backend koristi. Da si radio sa MariaDB, ovde bi bilo drugačije.
  
##### Ali sada dolazi pitanje

Ako ovde nema: db_host i db_port definicije kako Frappe zna da koristi:  127.0.0.1, 5432?

Odgovor je: Ne zna iz ovog fajla. Te informacije dolaze iz drugih delova konfiguracije (globalnih podešavanja Bench-a i podrazumevanih vrednosti).

To znači da jedan `site_config.json` sadrži samo ono što je specifično za taj sajt.

To mi se veoma sviđa kao dizajn.
  
##### Jedna mala digresija
  
Sećaš se koliko smo juče jurili grešku?
  
```sh
password authentication failed
```

pa zatim

```sh
pg_hba.conf entry
```

Sada možeš da rekonstruišeš šta se tada dešavalo.
Bench je radio otprilike ovo:

```sh
postgres
      │
      │  (superuser)
      ▼
CREATE ROLE _c9eb2d89e08e8728
CREATE DATABASE _c9eb2d89e08e8728
GRANT ...
      │
      ▼
Disconnect
Reconnect
user = _c9eb2d89e08e8728
database = _c9eb2d89e08e8728
```

I upravo je na tom drugom povezivanju PostgreSQL rekao: "Ne dozvoljavam ovom korisniku pristup."

Kada smo sredili `pg_hba.conf`, sve je proradilo.

Sada cela priča ima smisla.
  
</br>

#### Sledeći korak

Po mom mišljenju, sada smo završili upoznavanje sa strukturom na disku. Mislim da je vreme da upoznaš žive procese.
  
Odnosno:

```sh
bench start
```

Ali ne da ga samo pokreneš.

Hoću da zajedno rastumačimo:

- zašto se pokreće 6–7 procesa,
- šta radi svaki od njih,
- zašto postoje Redis Queue i Redis Cache,
- šta radi Scheduler,
- šta radi SocketIO,
- šta radi Watch,
- šta zapravo sluša na portu 8000.

Po meni, to je trenutak kada Frappe prestaje da bude "gomila direktorijuma" i postaje živi sistem. I upravo tada ćeš dobiti potpunu sliku kako se svi delovi koje smo do sada upoznali uklapaju u jednu funkcionalnu celinu.
  
## Faza 4 - Pokretanje razvojnog okruženja

U ovoj fazi nećemo samo izvršiti:

```bash
bench start
```

nego ćemo razumeti:

- zašto se pokreće više procesa;
- čemu služi svaki od njih;
- kako međusobno komuniciraju;
- šta je Redis Cache, a šta Redis Queue;
- šta radi Scheduler;
- šta radi Socket.IO;
- šta radi Watch proces;
- koji proces sluša na kom portu;
- kako browser na kraju dobije HTML stranicu.

To je jedna od najvažnijih lekcija u Frappe-u, jer kada jednom razumeš tu arhitekturu, mnoge kasnije stvari (background jobs, realtime obaveštenja, build frontend-a, migracije...) postaju mnogo logičnije.

### Za početak

Nemoj još ništa da objašnjavaš niti da tumačiš izlaz. Samo iz `frappe-bench` direktorijuma pokreni:

```bash
bench start
```

Sačekaj da se svi procesi pokrenu (ili da neki prijavi grešku), pa mi pošalji ceo izlaz terminala.

Na osnovu njega ćemo analizirati proces po proces. Nećemo ih posmatrati kao "gomilu logova", već ćemo ih povezati sa arhitekturom sistema. Mislim da će ti se taj deo posebno dopasti, jer ćeš prvi put videti kako Bench zapravo orkestrira ceo Frappe ekosistem.

---

Evo izlaza:

```sh
/frappe-bench$ bench start
15:31:35 system        | redis_cache.1 started (pid=1419)
15:31:35 system        | redis_queue.1 started (pid=1422)
15:31:35 system        | socketio.1 started (pid=1429)
15:31:35 redis_cache.1 | 1420:C 13 Jul 2026 15:31:35.033 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
15:31:35 redis_cache.1 | 1420:C 13 Jul 2026 15:31:35.033 # Redis version=7.0.15, bits=64, commit=00000000, modified=0, pid=1420, just started
15:31:35 redis_cache.1 | 1420:C 13 Jul 2026 15:31:35.033 # Configuration loaded
15:31:35 redis_cache.1 | 1420:M 13 Jul 2026 15:31:35.033 * Increased maximum number of open files to 10032 (it was originally set to 1024).
15:31:35 redis_cache.1 | 1420:M 13 Jul 2026 15:31:35.033 * monotonic clock: POSIX clock_gettime
15:31:35 system        | web.1 started (pid=1432)
15:31:35 system        | worker.1 started (pid=1441)
15:31:35 redis_cache.1 | 1420:M 13 Jul 2026 15:31:35.042 * Running mode=standalone, port=13000.
15:31:35 redis_cache.1 | 1420:M 13 Jul 2026 15:31:35.042 # Server initialized
15:31:35 redis_cache.1 | 1420:M 13 Jul 2026 15:31:35.042 # WARNING Memory overcommit must be enabled! Without it, a background save or replication may fail under low memory condition. Being disabled, it can can also cause failures without low memory condition, see https://github.com/jemalloc/jemalloc/issues/1328. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
15:31:35 redis_cache.1 | 1420:M 13 Jul 2026 15:31:35.042 * Ready to accept connections
15:31:35 system        | watch.1 started (pid=1427)
15:31:35 system        | schedule.1 started (pid=1431)
15:31:35 redis_queue.1 | 1426:C 13 Jul 2026 15:31:35.053 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
15:31:35 redis_queue.1 | 1426:C 13 Jul 2026 15:31:35.053 # Redis version=7.0.15, bits=64, commit=00000000, modified=0, pid=1426, just started
15:31:35 redis_queue.1 | 1426:C 13 Jul 2026 15:31:35.053 # Configuration loaded
15:31:35 redis_queue.1 | 1426:M 13 Jul 2026 15:31:35.053 * Increased maximum number of open files to 10032 (it was originally set to 1024).
15:31:35 redis_queue.1 | 1426:M 13 Jul 2026 15:31:35.053 * monotonic clock: POSIX clock_gettime
15:31:35 redis_queue.1 | 1426:M 13 Jul 2026 15:31:35.054 * Running mode=standalone, port=11000.
15:31:35 redis_queue.1 | 1426:M 13 Jul 2026 15:31:35.054 # Server initialized
15:31:35 redis_queue.1 | 1426:M 13 Jul 2026 15:31:35.054 # WARNING Memory overcommit must be enabled! Without it, a background save or replication may fail under low memory condition. Being disabled, it can can also cause failures without low memory condition, see https://github.com/jemalloc/jemalloc/issues/1328. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
15:31:35 redis_queue.1 | 1426:M 13 Jul 2026 15:31:35.057 * Ready to accept connections
15:31:36 watch.1       | 
15:31:36 web.1         | /home/radosav/frappe-bench/env/lib/python3.12/site-packages/passlib/utils/__init__.py:854: DeprecationWarning: 'crypt' is deprecated and slated for removal in Python 3.13
15:31:36 web.1         |   from crypt import crypt as _crypt
15:31:36 socketio.1    | Realtime service listening on: ws://0.0.0.0:9000
15:31:36 watch.1       | yarn run v1.22.22
15:31:36 watch.1       | $ node esbuild --watch --live-reload
15:31:37 web.1         | WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
15:31:37 web.1         |  * Running on all addresses (0.0.0.0)
15:31:37 web.1         |  * Running on http://127.0.0.1:8000
15:31:37 web.1         |  * Running on http://192.168.122.74:8000
15:31:37 web.1         | Press CTRL+C to quit
15:31:37 web.1         |  * Restarting with stat
15:31:37 web.1         | /home/radosav/frappe-bench/env/lib/python3.12/site-packages/passlib/utils/__init__.py:854: DeprecationWarning: 'crypt' is deprecated and slated for removal in Python 3.13
15:31:37 web.1         |   from crypt import crypt as _crypt
15:31:37 web.1         |  * Debugger is active!
15:31:37 web.1         |  * Debugger PIN: 972-324-418
15:31:43 watch.1       | Browserslist: caniuse-lite is outdated. Please run:
15:31:43 watch.1       |   npx update-browserslist-db@latest
15:31:43 watch.1       |   Why you should do it regularly: https://github.com/browserslist/update-db#readme
15:31:54 watch.1       | Watching for changes...
```

---

Savršeno! Ovo je upravo izlaz koji sam želeo da vidim. I ono što je najlepše: *sve radi.

Sada ćemo uraditi nešto što retko rade tutorijali. Umesto da kažemo "pokreni `bench start`", objasnićemo zašto se pokreće baš ovih sedam procesa.

### Bench nije server

Ovo je prva važna stvar. Mnogi početnici misle: "Pokrenuo sam Bench."  
Ne. Pokrenuo si **orkestrator**.

`bench start` ne izvršava Frappe direktno. On pokreće više nezavisnih procesa i nadgleda ih.

Praktično radi nešto ovako:

```text
                bench start
                     │
     ┌───────────────┼────────────────┐
     │               │                │
 web.py         redis_cache      redis_queue
     │
 socketio
     │
 scheduler
     │
 worker
     │
 watch
```

Ako jedan proces padne, Bench to vidi i ispisuje u terminal. To je razlog zbog kog su svi logovi objedinjeni na jednom mestu.

#### Pogledaj prvih nekoliko redova

  ```text
  redis_cache
  redis_queue
  socketio
  web
  worker
  watch
  schedule
  ```
  
  To je praktično mapa Frappe-a.
  Svaki od procesa rešava jedan veoma specifičan problem.
  
#### WEB
  
Ovo je najlakši.

```txt
web.1
```

Kasnije vidiš

```txt
Running on
127.0.0.1:8000
192.168.122.74:8000
```

To je HTTP server.  
Browser priča sa njim.  
Ako otvoriš <http://192.168.122.74:8000> ili preko port forwardinga sa hosta,
prvo se javlja upravo **web proces**.  
Ali... web ne radi sve. On samo prima zahtev.

#### SOCKETIO

```txt
Realtime service listening
ws://0.0.0.0:9000
```

Ovo je sasvim drugi server.
Ne HTTP. Već **WebSocket**.
Njegova svrha je: "server → browser" bez refresh-a.

Na primer:

- notifikacije
- chat
- progress bar
- live dashboard
- background job završen

Sve to dolazi preko SocketIO.

#### REDIS CACHE
  
Prvi Redis.

```text
port 13000
```

Ovaj Redis služi kao memorijski keš.

Na primer:

```txt
Korisnik
   ↓
Permissions
   ↓
Redis Cache
   ↓
sledeći zahtev
   ↓
ne čita bazu ponovo
```

 Time se štedi mnogo SQL upita.

#### REDIS QUEUE
  
Drugi Redis.

```txt
port 11000
```

Ovo je potpuno druga uloga.  
Ovde se ne čuvaju podaci.  
Ovde se čuvaju zadaci.  

Na primer:

```text
Pošalji 500 emailova.
```

Browser neće čekati.  
Web kaže: "Stavi ovo u Queue".  
Redis Queue zapamti posao.  
Worker ga kasnije izvrši.
  
#### WORKER
  
Jedan od mojih omiljenih procesa.

```text
worker.1
```

Njegov posao je veoma jednostavan.  

Beskonačna petlja.

```txt
Ima li nešto u Queue?
↓
nema
↓
čekaj
↓
ima
↓
izvrši
↓
čekaj
```

To je sve.  

Ali zahvaljujući njemu browser ostaje brz.
  
#### SCHEDULER
  
Ovo je nešto kao cron.

Na primer:

```txt
svakih 5 minuta
↓
pokreni cleanup
```

ili

```text
svake noći
↓
backup
```

ili

```text
svakih sat vremena
↓
sync
```

Scheduler ne izvršava posao.
On samo kaže:

"Vreme je."

Posao zatim ubaci u Queue.
Worker ga izvrši.
Primeti kako se procesi lepo nadovezuju.

#### WATCH
  
Ovo koriste programeri.

Kod tebe se lepo vidi

```txt
esbuild --watch
```

i kasnije

```text
Watching for changes
```

Šta to znači?

Ako promeniš

```txxt
apps/
frappe/
...    
...
...
some.js
```

Watch odmah vidi izmenu.  
Automatski pokrene build.  
Ne moraš ručno.
To je ogromna ušteda vremena.

#### Jedan primer

Recimo da klikneš u browseru

```txt
Create Customer
```

Šta se dešava?

Otprilike:

```txt
Browser
↓
HTTP
↓
WEB
↓
Python
↓
PostgreSQL
↓
vrati rezultat
↓
Browser
```
  
Ali ako taj Customer treba da pošalje email dobrodošlice:

```txt
WEB
↓
Queue
↓
Redis Queue
↓
Worker
↓
SMTP
↓
Email poslat
```

Browser u međuvremenu već prikazuje stranicu.

Ne čeka email.

#### A ako email traje 20 sekundi?**

Nema problema.
Worker radi.
Browser radi.
SocketIO može da kaže:

```txt
Email sent.
```

bez refresh-a.

#### Šta je zapravo Frappe?

Ja bih odgovorio ovako:

```text
                 Browser
                     │
                HTTP/WebSocket
                     │
         ┌───────────┴───────────┐
         │                       │
       Web                  SocketIO
         │
         ▼
      Frappe Framework
         │
 ┌───────┼────────┐
 │       │        │
 ▼       ▼        ▼
PostgreSQL Redis Queue Redis Cache
           │
           ▼
         Worker
```

To je mnogo bliže stvarnoj slici nego jednostavno reći "Frappe je Python web framework".

#### Dve zanimljive sitnice iz tvog izlaza

- Prva je upozorenje:

  ```txt
  Memory overcommit must be enabled!
  ```
  
  Ovo nije greška. Redis samo preporučuje da Linux ima:
  
  ```bash
  vm.overcommit_memory = 1
  ```
  
  Za razvoj možeš da ga ignorišeš. Za produkciju ga je dobro podesiti.

- Druga je:

  ```text
  passlib
  DeprecationWarning: crypt is deprecated
  ```
  
  To dolazi od Python-a 3.12 i biblioteke `passlib`. Nije do tvog sistema niti do Frappe konfiguracije. Očekujem da će biti rešeno u novijim verzijama zavisnosti.

Sada si prvi put video Frappe kao celovit sistem, a ne samo kao skup direktorijuma i komandi. Sledeći korak bi bio da otvorimo aplikaciju u pregledaču i ispratimo jedan zahtev od klika u browseru, preko web procesa i baze, pa nazad do odgovora. To će lepo povezati sve što smo danas objasnili.

### Prva poseta

Sada prelazimo na ono što ja smatram prvim pravim susretom sa Frappe-om. Do sada smo bili "ispod haube". Sada ćemo prvi put pogledati kako izgleda sistem iz ugla korisnika, ali ćemo ga posmatrati iz ugla programera.

</br>

- **Korak 1 - Otvori Frappe u browseru**

  Pošto radiš u VM-u, trebalo bi da možeš da otvoriš:
  
  ```txt
  http://192.168.122.74:8000
  ```
  
  ili, ako koristiš SSH tunel ili port forwarding, odgovarajuću adresu na hostu.
  
  Trebalo bi da dobiješ login ekran.
  
  Nemoj još ništa da istražuješ. Samo potvrdi da se stranica otvara.

</br>

- **Šta se upravo dogodilo?**

  Pre nego što se uloguješ, hajde da ispratimo jedan jedini HTTP zahtev.
  
  - Browser šalje:
  
    ```txt
    GET /
    ```
  
  - Web server prima zahtev.
  - Frappe kaže: "Koji sajt je tražen?". Pošto imaš samo jedan sajt (`site1.
  local`), odgovor je jednostavan.->  
  - Frappe učitava:  
  
    ```txt
    sites/site1.local/site_config.json
    ```
  
  - Povezuje se na PostgreSQL.  
  - Pronalazi da korisnik nije prijavljen.  
  - Generiše HTML login stranice.  
  - Browser je prikazuje.  
  
  To je ceo prvi ciklus.

</br>

- **Ali... kako Frappe zna da treba da koristi baš `site1.local`?**

  Ovo je odlično pitanje.
  
  </br>

  - **Na produkcionom serveru odgovor je jednostavan**
  
    ```sh
    erp.firma.rs
    ```

    - Host zaglavlje (HTTP Host header)
    - site1.local ili firma.rs ili erp.example.com
    - Svaki domen odgovara jednom sajtu.
  
    Ali ti nemaš domen.
    I nemaš Nginx.
  
    Imaš samo:
  
    ```txt
    <http://192.168.122.74:8000>
    ```
  
  </br>

  - **Pa kako onda zna?**

    Hajde da pogledamo.
  
    Pošalji rezultat ove komande:
  
    ```sh
    cat sites/common_site_config.json
    ```

    To je sledeći važan fajl u Bench arhitekturi.

  </br>

  - **Zašto baš sada?**

    Do sada smo videli:

    ```sh
    site_config.json
    ```

    (samo jedan sajt)

    Sada ćemo videti:

    ```sh
    common_site_config.json
    ```

    (globalna konfiguracija Bench-a)

    Kada ih uporedimo, dobićeš kompletnu sliku kako Bench odlučuje:

    - koji Redis koristi,
    - koje portove koristi,
    - kako pronalazi sajt,
    - koje vrednosti važe za sve sajtove.

</br>

- **Ono što ćemo uskoro povezati**

  Na kraju ove celine imaćeš veoma jasnu mentalnu mapu:
  
  ```txxt
  Browser
        │
        ▼
  bench start
        │
        ▼
  Web server
        │
        ▼
  common_site_config.json
        │
        ▼
  site1.local
        │
        ▼
  site_config.json
        │
        ▼
  PostgreSQL
  ```
  
  Kada razumeš ovu putanju, razumeš kako Frappe "pronalazi" i pokreće jedan sajt. Posle toga će nam mnogo lakše biti da pređemo na DocType-ove, aplikacije i razvoj sopstvenog koda.

### Zajednička definicija za sve sajtove

---

Evo izlaza:

```sh
cat sites/common_site_config.json
{
 "background_workers": 1,
 "file_watcher_port": 6787,
 "frappe_user": "radosav",
 "gunicorn_workers": 5,
 "live_reload": true,
 "rebase_on_pull": false,
 "redis_cache": "redis://127.0.0.1:13000",
 "redis_queue": "redis://127.0.0.1:11000",
 "redis_socketio": "redis://127.0.0.1:13000",
 "restart_supervisor_on_update": false,
 "restart_systemd_on_update": false,
 "serve_default_site": true,
 "shallow_clone": true,
 "socketio_port": 9000,
 "use_redis_auth": false,
 "webserver_port": 8000
}
```

---

Sada imamo praktično kompletnu sliku kako Bench funkcioniše.

Po mom mišljenju, **common_site_config.json je "kontrolni centar" Bench-a**, dok je **site_config.json lična karta jednog sajta**.

Hajde da ga rastavimo.

#### Dva nivoa konfiguracije

Već sada možeš da vidiš da postoje dva nivoa:

```text
sites/
    common_site_config.json [ 1. -> (važi za sve sajtove)]
    site1.local/
        site_config.json    [ 2. -> (samo za site1.local)]
```

To je veoma elegantan dizajn. Globalne stvari pišu se jednom. Specifične stvari pišu se po sajtu.

#### Redis

Odmah vidiš tri Redis konekcije:

```json
"redis_cache": "redis://127.0.0.1:13000",
"redis_queue": "redis://127.0.0.1:11000",
"redis_socketio": "redis://127.0.0.1:13000"
```

Odmah možemo da povežemo sa onim što smo videli juče.

```txt
Redis Cache -> port:13000
Redis Queue -> port: 11000
SocketIO -> koristi isti Redis kao Cache.
```

Zašto?  
Zato što SocketIO koristi Redis kao **message broker** između procesa.  
To ćemo detaljnije videti kasnije kada budemo pričali o realtime događajima.

#### Web server

Ovde stoji

```json
"webserver_port": 8000
```

To je upravo ono što si video u izlazu:

```sh
Running on http://127.0.0.1:8000
```

Dakle Bench nije "pogodio" port. On ga je pročitao odavde.

#### SocketIO

Ovde piše

```json
"socketio_port": 9000
```

A u logovima si video

```sh
Realtime service listening
ws://0.0.0.0:9000
```

Opet se sve poklapa.

#### File watcher

```json
"file_watcher_port": 6787
```

Ovo koristi Watch proces.  
Ne koristi ga browser.  
Ne koristi PostgreSQL.  
Koristi ga razvojni alat.

#### Background workers

```json
"background_workers": 1
```

To znači da trenutno imaš jednog Worker-a.

Kasnije možeš imati

```json
"background_workers": 4
```

ili

```json
"background_workers": 8
```

u zavisnosti od opterećenja.

#### Gunicorn workers

```json
"gunicorn_workers": 5
```

Ovo te možda zbuni. Jer si video Flask development server. Zašto onda Gunicorn?
Odgovor je jednostavan. `bench start` koristi flask ako razvojni server.

U produkciji Bench koristi Gunicorn. Dakle ova vrednost će biti važna kasnije kada pređeš na produkciono okruženje.

#### Najzanimljivija stavka

  Po meni je ovo:
  
  ```json
  "serve_default_site": true
  ```
  
  Sećaš se da si pitao:
  
  > Kako Frappe zna da otvori baš `site1.local` kada u URL-u pišem samo IP adresu?
  
  Evo odgovora.
  
  Pošto je
  
  ```json
  serve_default_site = true
  ```
  
  Bench kaže:
  
  > Ako ne mogu da odredim sajt na osnovu HTTP Host zaglavlja, posluži podrazumevani sajt.
  
  Pošto imaš samo jedan sajt, to je upravo `site1.local`.

  Kasnije, kada budeš imao:
  
  ```text
  site1.local
  firmaA.local
  firmaB.local
  ```
  
  više neće biti dovoljno da pristupiš preko IP adrese.
  
  HTTP Host zaglavlje ili Nginx će odlučivati koji sajt treba otvoriti.

#### Korisnik Bench-a

Ovde piše

```json
"frappe_user": "radosav"
```

To znači da je ceo Bench napravljen pod tvojim Linux korisnikom.

To je razlog zašto nikada ne radimo:

```bash
sudo bench ...
```

Bench treba da radi kao običan korisnik.

To je veoma dobra praksa.

#### Jedna mala "Aha!" stvar

Pogledaj sada zajedno oba fajla.

</br>

- **Globalno - common_site_config.json**
  - redis
  - portovi
  - worker
  - socketio

</br>

- **Lokalno - site1.local/site_config.json**
  - db_name
  - db_password
  - db_type

Odjednom postaje jasno zašto su odvojeni.

## Faza 5 - Frappe aplikacija

Do sada smo videli:

- gde je kod,
- gde su podaci,
- gde je konfiguracija,
- koji procesi rade.

Ali još nismo videli šta je zapravo Frappe aplikacija.

U `apps/` trenutno postoji samo:

```txt
frappe/
```

### Aplikacija Frappe

To je ogromna aplikacija.

Moje mišljenje je da je sledeći logičan korak da je "otvorimo" i upoznamo njenu unutrašnju strukturu.

Ne da čitamo hiljade fajlova, već da razumemo kako je organizovana.

To će nam dati odgovor na pitanja poput:

- Gde žive DocType-ovi?
- Gde je Python kod?
- Gde su JavaScript fajlovi?
- Gde su HTML/Jinja templejti?
- Gde su API endpointi?
- Šta su `hooks.py`, `modules.txt`, `patches.txt`?

To je trenutak kada ćeš prvi put videti kako je Frappe kao framework sastavljen iznutra, a odatle ćemo vrlo prirodno preći na pravljenje sopstvene aplikacije i prvog DocType-a. Po mom iskustvu, taj redosled omogućava da kasnije mnogo lakše razumeš zašto Frappe radi baš na način na koji radi.

Odlično. Sada ulazimo u ono što bih nazvao "Anatomija Frappe aplikacije".

Do sada smo posmatrali Bench kao grad. Sada ulazimo u jednu zgradu — aplikaciju **frappe**.

#### Gde se nalazi?

Kod tebe:

```text
frappe-bench/
└── apps/
    └── frappe/
```

Ali...

To još nije Python paket.

Uđi u direktorijum i pogledaj sadržaj:

```bash
cd apps/frappe
ls -alh
```

Pošalji izlaz.

Zašto ne idem odmah na `tree`?  
Namerno.  
Frappe ima više od 3000 fajlova.  
Ako odmah uradimo:  
  
```bash
tree
```

dobićeš zid teksta.

Nećemo ništa naučiti.

Želim da prvo vidimo vrh strukture*, pa ćemo onda ulaziti samo u one delove koji su važni.

#### Šta ćemo tražiti?

Kada dobijemo izlaz, pokazaću ti:

- šta je Git repozitorijum,
- gde počinje pravi Python paket,
- zašto postoji direktorijum koji se takođe zove `frappe`,
- gde su dokumenti,
- gde su testovi,
- gde su hook-ovi,
- gde su frontend fajlovi.

To ume da zbuni skoro svakoga prvi put, jer ćeš vrlo verovatno videti nešto poput:

```text
apps/
└── frappe/
    └── frappe/
```

i pomisliti:

> "Zašto postoje dva `frappe` dira?"

Odgovor je veoma logičan kada se jednom objasni, i posle toga ćeš isti obrazac prepoznavati i u drugim Python projektima.

#### Mali cilj za ovu fazu
  
Ne želim da zapamtiš nazive svih direktorijuma.

Želim da na kraju možeš da odgovoriš na jedno pitanje:

  > "Ako želim da pronađem deo Frappe koda koji implementira neku funkcionalnost, gde bih uopšte počeo da tražim?"

Kada to budeš znao, prelazak sa "korisnika Frappe-a" na "programera Frappe-a" biće mnogo prirodniji.

--

Evo izlaza:

```sh
ls -alh
total 332K
drwxrwxr-x  10 radosav radosav 4,0K jul 11 00:37 .
drwxrwxr-x   3 radosav radosav 4,0K jul 11 00:35 ..
-rw-rw-r--   1 radosav radosav 1,5K jul 11 00:35 attributions.md
-rw-rw-r--   1 radosav radosav  741 jul 11 00:35 babel_extractors.csv
-rw-rw-r--   1 radosav radosav 1,2K jul 11 00:35 codecov.yml
-rw-rw-r--   1 radosav radosav 3,2K jul 11 00:35 CODE_OF_CONDUCT.md
-rw-rw-r--   1 radosav radosav  220 jul 11 00:35 CODEOWNERS
-rw-rw-r--   1 radosav radosav  394 jul 11 00:35 commitlint.config.js
-rw-rw-r--   1 radosav radosav  386 jul 11 00:35 .coveragerc
-rw-rw-r--   1 radosav radosav  444 jul 11 00:35 crowdin.yml
drwxrwxr-x   6 radosav radosav 4,0K jul 11 00:35 cypress
-rw-rw-r--   1 radosav radosav  701 jul 11 00:35 cypress.config.js
-rw-rw-r--   1 radosav radosav  406 jul 11 00:35 .editorconfig
drwxrwxr-x   2 radosav radosav 4,0K jul 11 00:35 esbuild
-rw-rw-r--   1 radosav radosav  278 jul 11 00:35 .eslintignore
-rw-rw-r--   1 radosav radosav 2,5K jul 11 00:35 .eslintrc
drwxrwxr-x  36 radosav radosav 4,0K jul 11 00:37 frappe
-rw-rw-r--   1 radosav radosav  568 jul 11 00:35 generate_bootstrap_theme.js
drwxrwxr-x   8 radosav radosav 4,0K jul 11 00:35 .git
-rw-rw-r--   1 radosav radosav 1,3K jul 11 00:35 .git-blame-ignore-revs
drwxrwxr-x   5 radosav radosav 4,0K jul 11 00:35 .github
-rw-rw-r--   1 radosav radosav 2,5K jul 11 00:35 .gitignore
drwxrwxr-x   2 radosav radosav 4,0K jul 11 00:35 .greptile
-rw-rw-r--   1 radosav radosav  890 jul 11 00:35 hooks.md
-rw-rw-r--   1 radosav radosav 1,1K jul 11 00:35 LICENSE
-rw-rw-r--   1 radosav radosav 1,8K jul 11 00:35 .mergify.yml
drwxrwxr-x 418 radosav radosav  20K jul 11 00:37 node_modules
-rw-rw-r--   1 radosav radosav 2,0K jul 11 00:35 node_utils.js
-rw-rw-r--   1 radosav radosav 2,5K jul 11 00:35 package.json
-rw-rw-r--   1 radosav radosav 2,2K jul 11 00:35 .pre-commit-config.yaml
-rw-rw-r--   1 radosav radosav 4,4K jul 11 00:35 pyproject.toml
-rw-rw-r--   1 radosav radosav 3,2K jul 11 00:35 README.md
drwxrwxr-x   4 radosav radosav 4,0K jul 11 00:35 realtime
-rw-rw-r--   1 radosav radosav  597 jul 11 00:35 .releaserc
-rw-rw-r--   1 radosav radosav  556 jul 11 00:35 SECURITY.md
-rw-rw-r--   1 radosav radosav    0 jul 11 00:35 .semgrepignore
-rw-rw-r--   1 radosav radosav   37 jul 11 00:35 sider.yml
-rw-rw-r--   1 radosav radosav   23 jul 11 00:35 socketio.js
-rw-rw-r--   1 radosav radosav 162K jul 11 00:35 yarn.lock
```

---

Odlično! Ovo je jedan od onih trenutaka kada se iz strukture direktorijuma može mnogo naučiti.

Prvo, želim da razdvojimo dve stvari koje se često mešaju.

#### Pregled frape direktorijuma

##### Nivo 1 - Git repozitorijum

Nalaziš se ovde:

```text
apps/frappe/
```

Ovo je ceo Git repozitorijum Frappe-a.  

To potvrđuje:

```text
.git/
```

Dakle:

```sh
git clone https://github.com/frappe/frappe.git
```

bi napravio upravo ovakav direktorijum.

Unutra je sve:

- izvorni kod
- dokumentacija
- testovi
- frontend
- konfiguracija
- Git istorija

##### Nivo 2 - Python paket

Sada pogledaj ovu stavku:

```sh
frappe/
```

Ona nije isto što i spoljni direktorijum.

Imamo:

```text
apps/
└── frappe/          ← Git repozitorijum
    └── frappe/      ← Python paket
```

Ovo je veoma čest obrazac u Python svetu.

Na primer Django izgleda slično:

```text
django/
    django/
```

Flask:

```text
flask/
    flask/
```

SQLAlchemy:

```text
sqlalchemy/
    sqlalchemy/
```

Nije ništa neobično.

##### Šta je sve ostalo?

Pogledajmo redom.

</br>

- `.git`  
  - Očekivano.  
  - Git istorija.  
  - Branch-evi.  
  - Commit-i.  
  - Tagovi.  
  - Ništa Frappe-specifično.  

</br>

- `README.md`

  - Opis projekta.  
  - Kako se instalira.  
  - Kako se razvija.

</br>

- `pyproject.toml`
  - Ovo je veoma važan fajl.  
  - Danas skoro svaki ozbiljan Python projekat koristi upravo njega.  

  - On govori:
    - kako se paket zove
    - koje su zavisnosti
    - koji build sistem koristi
    - koje Python verzije podržava
  
    Drugim rečima: "Ja sam Python paket."

</br>

- `package.json`
  - E ovo je zanimljivo.  
  - Odmah vidiš da Frappe nije samo Python.  
  - To je istovremeno i Node projekat.  
  - Frontend koristi:
    - JavaScript
    - Node
    - Yarn
    - esbuild
  
  Zbog toga si instalirao Node.js.

</br>

- `node_modules`  
  
  - Ogroman direktorijum.  
  - Tu su svi frontend paketi.  
  - Nema potrebe da ga ikada ručno diraš.  

</br>

- `esbuild`
  - Frontend build.  
  - JavaScript.  
  - CSS.  
  - Bundle.  

</br>

- **`realtime`**
  - Odmah vidiš da postoji poseban direktorijum.  
  - To je SocketIO server.  
  - Kasnije ćemo ga otvoriti.  

</br>

- **`cypress`**
  - Automatski testovi.
  - Browser testovi.

</br>

- **Šta mi nedostaje?**
  - Zapravo, najvažniji direktorijum još nismo otvorili.
  - To je upravo ovaj:
  
    ```sh
    apps/frappe/frappe/
    ```

    Tu počinje framework.

</br>
 
- **Sledeći korak**

  Hajde da otvorimo samo prvi nivo tog direktorijuma.
  
  Pošalji:
  
  ```bash
  tree -L 1 frappe
  ```
  
  ili, ako nemaš `tree` pri ruci:
  
  ```bash
  ls -alh frappe
  ```
  
  To je možda **najvažniji direktorijum u celom framework-u**.
  
### Šta je urađeno?

Do sada smo praktično složili sledeću mentalnu mapu:

```text
frappe-bench/
├── apps/
│   └── frappe/          ← Git repozitorijum
│       └── frappe/      ← Python paket (framework)
│
├── sites/
│   ├── common_site_config.json
│   └── site1.local/
│       └── site_config.json
│
├── env/                 ← Python virtuelno okruženje
├── logs/
└── config/
```

A takođe smo razjasnili i odnose:

- **Bench** upravlja okruženjem.
- **App** predstavlja funkcionalnost (za sada samo `frappe`).
- **Site** predstavlja jednu instalaciju sa sopstvenom bazom.
- Jedna **aplikacija** može biti instalirana na više sajtova.
- Jedan **sajt** može imati više aplikacija.
- Svaki sajt ima svoju PostgreSQL bazu (ili MariaDB, u zavisnosti od konfiguracije).

To je zapravo temelj cele Frappe arhitekture.
  
### Plan za sledeći nastavak
  
Sledeći put ćemo konačno "otvoriti motor", odnosno `apps/frappe/frappe/`, i upoznati najvažnije delove framework-a:

- `model/` — gde žive DocType-ovi i ORM.
- `database/` — sloj za rad sa bazom.
- `desk/` — backend za Desk interfejs.
- `www/` — web stranice.
- `api/` — API endpointi.
- `utils/` — pomoćne funkcije.
- `hooks.py` — jedan od najvažnijih fajlova u svakoj Frappe aplikaciji.

Cilj neće biti da zapamtimo sve direktorijume, već da stekneš osećaj:  
**"Ako tražim određenu funkcionalnost, gde bih je potražio?"**
