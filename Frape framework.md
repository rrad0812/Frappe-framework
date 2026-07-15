# Projekat: Učenje Frappe Framework-a

## Faze projekta

</br>

- **Faza 0 – Priprema okruženja**

  - Izabrati verziju Ubuntu-a (24.04 LTS)
  - Kreirati novu VM
  - Instalirati Ubuntu Server
  - Ažurirati sistem
  - Instalirati osnovne alate
  - Napraviti snapshot Clean System

  Cilj: "Stabilna razvojna mašina.
  
</br>

- **Faza 1 – Instalacija Frappe okruženja**

  - Python
  - Node.js
  - Redis
  - PostgreSQL
  - Bench
  - Kreirati prvi Bench

  Ovde nećemo još praviti nijedan DocType.  
  Cilj je da razumemo: "Šta je Bench?"

</br>

- **Faza 2 – Anatomija Bencha**

  Ovde ne pišemo kod. Samo istražujemo.
  
  - Struktura direktorijuma
  - apps/
  - sites/
  - env/
  - logs/
  - config/
  
  Na kraju ove faze treba da možeš da pogledaš Bench i kažeš: "Znam čemu služi svaki direktorijum."

</br>

- **Faza 3 – Prvi Site**

  - napraviti Site
  - pokrenuti ga
  - otvoriti Desk
  - prijaviti se
  - pogledati šta je nastalo u PostgreSQL-u
  
  Ovde ćemo prvi put zaviriti u bazu.

</br>

- **Faza 4 – Anatomija Site-a**

  Opet bez programiranja.
  
  - site_config.json
  - private/
  - public/
  - assets/
  - logs/
  
  Na kraju: "Znam šta je Site."

</br>

- **Faza 5 – Prva aplikacija**

  Tek ovde,
  
  - new-app
  - instalacija aplikacije
  - modul
  - prvi DocType

</br>

- **Faza 6 – Desk**

  Ovo je deo koji si već pomenuo da ti je bio nejasan.
  
  Ovde ćemo odgovoriti na pitanja:
  
  - Kako Desk vidi moj DocType?
  - Kako se pojavljuje u Workspace-u?
  - Zašto ga nekad nema?
  - Kako ga organizovati?
  
  Mislim da će ova faza biti jedna od najzanimljivijih.

</br>

- **Faza 7 – DocType**

  Ovde konačno ulazimo u razvoj.
  
  - polja
  - validacija
  - child table
  - link
  - select
  - naming

</br>

- **Faza 8 – ORM**

  - Python.  
  - Ne JavaScript.  
  - Ne REST.  
  - Samo ORM.

</br>

- **Faza 9 – Hook-ovi**

  Šta se događa kada:
  
  - sačuvaš dokument
  - obrišeš dokument
  - submit
  - cancel
  
</br>

- **Faza 10 – JavaScript**

  - Client Script.
  - Form Script.
  - List Script.

</br>

- **Faza 11 – REST API

  Ovde ćemo ga uporediti sa uAdmin-om.  
  Mislim da će to biti veoma zanimljivo.

</br>

- **Faza 12 – Bezbednost**

  - Users
  - Roles
  - Permissions

</br>

- **Faza 13 – Deploy**

  Tek na kraju.
  
  - production
  - nginx
  - supervisor
  - backup

</br>

- **Faza 14 - Završna faza**

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

**Zašto Frappe ima Bench?**

Dodao bih i jednu "Fazu -1" koja se ne tiče instalacije nego razumevanja. Pre nego što instaliramo ijedan paket, voleo bih da odgovorimo na ovo pitanje!

Većina korisnika samo prihvati da postoji komanda bench, ali retko ko razume zašto nije jednostavno pip install frappe. Mislim da će nam upravo odgovor na to pitanje dati dobar uvod u arhitekturu frameworka i učiniti sve naredne korake logičnijim. Po mom mišljenju, to je pravo mesto odakle treba da počnemo.

## Faza 0 - Priprema okruženja

- **Preuzimnje ISO slike i instalacija**

  </br>

  - Sa adrese <https://releases.ubuntu.com/24.04.4/ubuntu-24.04.4-live-server-amd64.iso> preuzeti
    ISO za `Ubuntu server 24.04`.

  </br>
  
  - Izgradnja Ubuntu Server 24.04 VM  
    VM izgradnja na QUEMU/KVM root sesiji sa sledećim parametrima:

    - vCPU 2
    - RAM 8GB
    - SSD 60GB

</br>

- **Ažuriranje paketa distibucije**

  ```sh
  sudo apt update && sudo apt upgrade -y
  ```

</br>

- **Instalacija curl-a**

  ```sh
  sudo apt install curl
  ```

</br>

- **Aktiviranje firewall-a i dozvola ssh pristupa**

  </br>

  - Aktiviranje firewall-a
  
    ```sh
    sudo ufw enable
    ```
  
  </br>
  
  - Dozvola pristupa preko ssh
  
    ```sh
    sudo ufw allow OpenSSH
    ```
  
  </br>
  
  - Proba konekcije na VM ( samo sa localhost-a )
    Iz virtuelne mašine pokrenuti:

    ```sh
    ip a
    ```

    Vratiće, za KVM slučaj, nešto kao 192.168.122.X.
  
  </br>
  
  - Sa hosta ssh pristup na VM
  
    ```sh
    ssh username_na_VM@127.198.122.X
    ```
  
  - Reset VM
  
    ```sh
    sudo shutdown -r now
    ```

- **Promena vremenske zone i sync. vremena**

  ```sh
  sudo timedatectl set-timezone Europe/Belgrade
  sudo timedatectl set-ntp on
  ```

</br>

- **Promena locale**

  ```sh
  sudo dpkg-reconfigure locales
  ```

  </br>
  
  - Dodaj nove locale `sr_RS@latin UTF-8` i postavi ih za default.
  
  </br>
  
  - Reset VM
  
    ```sh
    sudo shutdown -r now
    ```

</br>

- **Instalacija osnovnih dev paketa**

  ```sh
  sudo apt python3-dev
  ```

</br>

- **Instalacija uv pajton paket i runtime managera**

  ```sh
  curl -LsSf https://astral.sh/uv/install.sh | sh
  ```
  
  </br>
  
  - Osveži sesiju:

    ```sh
    source "$HOME/.local/bin/env"
    ```

  </br>
  
  - Proveri instaliranost:

    ```sh
    uv --version
    ```

</br>

**Šta je urađeno?**

- Instaliran Ubuntu Server 24.04 sa OpenSSH serverom.
- Instalacija ažurirana na nove verzije paketa
- Promenjen status UFW na enable. Promenjen status OpenSSH na enable. Postignut
  ssh pristup sa localhosta na VM.
- Promenjena vremenska zona na Europe/Belgrade, uradjen sync. vremena
- Promenjen locale na sr_RS@latin UTF-8 i postavljen za podrazumevani.
- Instaliran uv Pajton paket i runtime manger.

## Faza 1 - Instalacija Frappe okruženja

</br>

- **Instalacija git**

  ```sh
  sudo apt install git -y
  ```

</br>

- **Instalacija Redisa**

  ```sh
  sudo apt install redis-server -y
  ```

</br>

- **Instalacija PostgreSQL**

  ```sh
  sudo apt install postgresql postgresql-contrib libpq-dev -y
  ```

  - Provera instalirane verzije
  
    ```sh
    psql --version
    ```

  - Prelazak na `postgres` nalog  

    To je podrazumevani admin korisnik PostgeSQL i instaliran je na sistem za vreme instalacije PostgrSQL-a.

    ```sh
    sudo -i -u postgres
    ```

  - Pokretanje PostgreSQL shela

    ```sh
    psql
    ```

  - Izlazak iz shela:

    ```sh
    \q
    ```

  - Na psql možeš doći kao `postgres` bez prelaska sa svog naloga:

    ```sh
    sudo -u postgres psql
    ```

  - Dodela passworda postgres korisniku:

    ```sql
    ALTER USER postgres WITH PASSWORD 'postgres_password';
    ```
  
- **Instalacija web servera - nginx**

  ```sh
  sudo apt install nginx
  ```

- **Instalacija wkhtmltopdf**
  
  - Instalacija zavisnosti

    ```sh
    sudo apt install xvfb libfontconfig
    ```

  - Preuzmi wkhtmltopdf paket sa <https://wkhtmltopdf.org/downloads.html>,
    potom pokreni sledeću komandu za instalaciju:

    ```sh
    sudo apt install ./wkhtmltox_file.deb
    ```
  
- **Instalacija frontend zavisnosti**

  - Instalacija **nvm**

    ```sh
    curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash
    ```

  - Instalacija **nodejs**

    ```sh
    nvm install 24
    ```

    - Provera instaliranosti
  
      ```sh
      node -v
      ```

  - Instalacija **yarn**

    ```sh
    npm install -g yarn
    ```
  
- **Instalacija Bench-a (Frappe v15)**  

  ```sh
  uv tool install frappe-bench --with setuptools
  ```

  **Napomena**:  
  Dodali smo sa `--with setuptools` jer Frappe u pozadini još uvek koristi neke starije Python mehanizme za pakete.
  
  - Sada proveri da li sistem vidi komandu:

    ```sh
    bench --version
    ```

- **Instalacija i inicijalizacija frappe-bench direktorijuma**

  Sada kada imamo bench alat, pravimo glavni folder gde će ti biti svi sajtovi i aplikacije (izaberi verziju 15):
  
  ```sh
  bench init frappe-bench --frappe-branch version-15
  ```
  
  Uđi u kreirani direktorijum:
  
  ```sh
  cd frappe-bench
  ```
  
  Sada se nalaziš u glavnom upravljačkom čvorištu svog projekta.

</br>

**Šta je urađeno?**

- Instaliran je git.
- Instaliran je Redis.
- Instaliran je PostgreSQL.
- Instaliran nginx.
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

</br>

- **Priprema PostgreSQL šablona baze podataka**

  Pokreni ovu komandu da omogućiš potrebne ekstenzije u podrazumevanom šablonu baze:
  
  ```sh
  sudo -u postgres psql -d template1 -c "CREATE EXTENSION IF NOT EXISTS pg_trgm; CREATE EXTENSION IF NOT EXISTS btree_gin;"
  ```
  
</br>

- **Kreiranje novog sajta**

  Sada, dok si unutar "frappe-bench" direktorijuma, pokreni komandu za kreiranje sajta. Nazvaćemo ga npr. "site1.local" (možeš staviti bilo koje ime koje se završava sa .localhost ili .local za lokalni razvoj):
  
  ```sh
  bench new-site site1.local --db-type postgres
  ```
  
  Tokom izvršavanja ove komande, Bench će te pitati za dve stvari:
  
  </br>

  - MySQL root password  
    Iako piše MySQL, pošto si stavio `--db-type postgres`, ovde unosiš lozinku za `postgres` korisnika koju si postavio u Fazi 1.
  
  - Administrator password  
    Ovo je lozinka za glavnog admina samog Frappe web interfejsa. Postavi neku lozinku po želji (npr. admin123 za lokalni rad) i zapamti je.
  
  </br>

  Kako da proveriš da li je uspelo?  
  Kada komanda završi, proveri da li baza zaista postoji u PostgreSQL-u:
  
  ```sh
  sudo -u postgres psql -c "\l"
  ```
  
  Trebalo bi da vidiš novu bazu sa čudnim, nasumičnim imenom koje počinje sa podvlakom (npr. _1a2b3c4d5e...), jer Frappe namerno generiše hešovana imena baza radi bezbednosti.
  
  </br>
  
- **Rešenje problema sa konekcijom na PostgreSQL**
  
  Moramo da dozvolimo svim korisnicima (`all`) da se povežu na sve baze (`all`) preko 127.0.0.1 (localhost-a) interfejsa koristeći lozinku.

  </br>

  Podrazumevano PostgeSQL je podešen na `peer` konkcije (`socket`) sa localhosta, i to je način na koji pristupa psql. Za Frappe pristup, preko mrežnog interfejsa potrebno je prepodesiti u `/etc/postgresql/16/main/pg_hba.conf` konfiguracionm fajlu IPv4 pravilo, tako da glasi:

  ```txt
  # IPv4 local connections:
  host    all             all             127.0.0.1/32            md5
  ```

  </br>

  Promenili smo tip protkola lozinke, jer se u praksi pokazalo da radi!

  </br>
  
  - **Restart PostgreSQL servisa**
  
    Posle promene konfig. fajllova, restartuj PostgreSQL servis:
  
    ```sh
    sudo systemctl restart postgresql
    ```

  </br>
  
- **Šta je urađeno**
  
  - U PostgreSQL `template1` šablon dodate ekstenzije: `pg_trgm` i `btree_gin`.
  - U fajlu `/etc/postgresql/16/main/pg_hba.conf` promenjena autentifikacija za IPv4 na `md5` za
    sve korisnike (`all all 127.0.0.1/32`), kako bi Frappe mogao mrežnim putem da kreira baze i korisnike.
  
  - Uspešno kreiran prvi sajt na PostgreSQL bazi podataka pod imenom `site1.  
    local` komandom:
  
    ```sh
    `bench new-site site1.local --db-type postgres`
    ```

## Faza 3 - Anatomija frappe-a

**`frappe-bench` dir**

Uradi u terminalu:

```bash
pwd
ls -lah
```

i pošalji izlaz.

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

</br>

- **apps/**  
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
  
  </br>
  
- **env/**  
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

  ```sh
  uv venv
  ```

  samo što Bench to radi sam.
  
</br>

- **sites/**  
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
  
  ```txt
  ERPNext
  ```
  
  može raditi na
  
  ```txt
  firmaA.local
  firmaB.local
  firmaC.local
  ```
  
  bez tri kopije ERPNext-a.
  
</br>

- **config/**  
  Ovo je Bench konfiguracija. Ovde se nalaze konfiguracioni fajlovi koje Bench generiše. Na primer:
  
  - Redis
  - Nginx
  - Supervisor
  - Procfile konfiguracije
  - razni JSON fajlovi
  
  Većinu vremena nećeš ručno menjati ove fajlove.
  
  Bench ih uglavnom generiše.
  
</br>

- **logs/**  
  Vrlo koristan direktorijum. Ako nešto ne radi... ovde prvo gledaš.
  
  Npr.
  
  ```sh
  web.log
  worker.log
  redis.log
  schedule.log
  ```
  
  Kasnije ćeš dosta vremena provoditi upravo ovde.

</br>

- **Procfile**  
  Ovo je zanimljiv fajl. Bench ne pokreće jedan proces. Pokreće ih više. Na primer:
  
  - web server
  - scheduler
  - worker
  - socketio
  - watch
  - ...
  
  `Procfile` govori Bench-u: "Ovo su procesi koje treba pokrenuti."
  
  Ako si radio sa Heroku ili Foreman, koncept će ti biti poznat.

</br>

- **patches.txt**  
  Ovo nije nešto što ćeš često dirati. Koristi se tokom migracija i nadogradnji kako bi Bench znao koje su zakrpe (patches) već primenjene.

</br>

**Globalna oraganizacija**  
Primeti da Bench veoma lepo odvaja tri stvari:

```sh
Code -> apps/
Data -> sites/
Enviroment -> env/
```

To je veoma elegantna organizacija.

</br>

**Pregled apps i sites dirova**  
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

</br>

- **apps/ dir**  
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

</br>

- **sites/ dir**
  Ovde se već nalazi mnogo zanimljivijih stvari.
  
  </br>

  **common_site_config.json**  
  Ovo je globalna konfiguracija Bench-a. Ona važi za sve sajtove. Na primer:
  
  ```txt
  site1.local
  site2.local
  demo.local
  ```
  
  Svi će koristiti ono što je definisano ovde, osim ako neki sajt ne prepiše (override) određenu vrednost.
  
  To je isti koncept koji postoji u mnogim frameworcima: globalna podešavanja + lokalna podešavanja.
  
  </br>

  **site1.local/**  

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

  </br>

  **assets/ dir**  

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

  </br>

  **apps.txt**  

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

  </br>

  **apps.json**  

  Ovo je noviji mehanizam koji Bench koristi za dodatne informacije o aplikacijama. U praksi ga retko menjaš ručno; Bench ga održava.

</br>

**Ono što treba da zapamtiš**  
Po mom mišljenju, ovo je najvažnija slika do sada:

```sh
apps/ -> Izvorni kod
sites/ -> Podaci o sajtovima
```

</br>

**Jedna aplikacija → više sajtova**  
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

</br>

**Predlog za sledeći korak**  
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
  site1.local/
  ├── frappe
  ├── erpnext
  ├── payments
  └── moja_aplikacija
  ```

  Sve četiri rade zajedno kao jedna celina.

</br>

**Kako to izgleda u praksi?**

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

</br>

**A šta svaka aplikacija donosi?**

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
  
**I gde se to čuva?**

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

**Jedna mala terminološka napomena**
Ovde je lako pomešati pojmove jer reč aplikacija u Frappe-u ne znači isto što i u mnogim drugim okvirima.

- **Frappe app** je modul ili paket funkcionalnosti (npr. ERPNext, CRM ili tvoja aplikacija).
- **Site** je instanca koja ima svoju bazu podataka, konfiguraciju i korisnike.

To znači da jedan site može biti sastavljen od više Frappe aplikacija, a ista aplikacija može biti korišćena na više različitih sajtova. Upravo ta fleksibilnost je jedan od razloga zašto je Frappe pogodan za razvoj većih poslovnih sistema i SaaS rešenja.

</br>

**sites/site1.local dir**  
Ovde ćemo videti sadržaj sites/site1.local dira

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

Zašto?  
Zato što je sav kod u `apps/`, a "site1.local" sadrži samo ono što pripada toj konkretnoj instanci.

To je jedna od najlepših ideja u Frappe-u.

</br>

- **site_config.json**  
  Ovo je, po mom mišljenju, najvažniji fajl jednog sajta.
  
  On odgovara na pitanja:
  
  - na koju bazu se povezujem?
  - koji Redis koristim?
  - gde su fajlovi?
  - koje su specifične postavke ovog sajta?
  
  Praktično, kada Frappe "otvori" sajt, prvo pročita ovaj fajl.
  
  Možemo ga detaljno analizirati za nekoliko minuta.

</br>

- **private/**  
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

</br>

- **public/**  
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

</br>

- **logs/**  
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

</br>

- **locks/**  
  Ovaj direktorijum mnogi ni ne primete. Koristi se za razne mehanizme zaključavanja (locking).
  
  Na primer:
  
  - da se dva procesa ne sudare tokom migracije,
  - da scheduler ne pokrene isti posao dva puta,
  - da se spreče paralelne operacije koje bi dovele do nekonzistentnog stanja.
  
  Većinu vremena će biti prazan. I to je potpuno normalno.

</br>

- **site_confg.json**  
  Predložio bih da pogledamo sadržaj:
  
  ```bash
  cat sites/site1.local/site_config.json
  ```
  
  To je možda svega desetak redova, ali iz njega ćemo moći da objasnimo:
  
  - kako Frappe pronalazi PostgreSQL bazu,
  - kako zna kojim Redis instancama da se poveže,
  - kako razlikuje jedan sajt od drugog,
  - koje su postavke globalne (`common_site_config.json`), a koje lokalne (`site_config.json`).
  
</br>

**Zanimljivost**  
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

</br>

**Jedna mala napomena**  
Do sada smo pričali o arhitekturi **Bench → Apps → Sites**, i mislim da je to bio pravi redosled. Međutim, od sledećeg koraka počećemo da povezujemo te delove u jednu celinu. Videćeš da Bench nije "aplikacija", već pre **orkestrator** koji upravlja Python okruženjem, aplikacijama i sajtovima.  

Kada to shvatiš, većina `bench` komandi će postati vrlo intuitivna, jer ćeš razumeti **šta** rade, a ne samo **kako** se koriste.
  
---

Evo izlaza

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

Odmah možemo da izvedemo nekoliko zaključaka:

</br>

- **Zašto baza nema ime `site1.local`?**  

  Verovatno si očekivao nešto poput: "site1.local", ili "site1_local". Međutim, Frappe radi drugačije.  On generiše ime baze: "_c9eb2d89e08e8728".

</br>

- **Gde je korisničko ime?**

  Primeti nešto zanimljivo. Ovde nema: "db_user": "...". Zašto? Zato što kod PostgreSQL-a Frappe koristi isto ime za bazu i korisnika.
  
  Drugim rečima: db_name: "_c9eb2d89e08e8728" i "role": "_c9eb2d89e08e8728" imaju isto ime.

</br>

- **Lozinka**

  Ovu lozinku nisi ti birao. Bench ju je napravio automatski. To je veoma dobra praksa. Svaki sajt dobija:
  
  - svog korisnika
  - svoju bazu
  - svoju nasumičnu lozinku

</br>

- **db_type**

  Ovde piše "db_type":"postgres". To znači da ostatak Frappe-a zna koji backend koristi. Da si radio sa MariaDB, ovde bi bilo drugačije.
  
</br>

**Kako Frappe zna da koristi za pristup PostgreSQL adresu 127.0.0.1 i port 5432**  
Ako ovde nema `db_host` i `db_port` definicije kako Frappe zna da koristi:  127.0.0.1, 5432?

Odgovor je: Ne zna iz ovog fajla. Te informacije dolaze iz drugih delova konfiguracije (globalnih podešavanja Bench-a i podrazumevanih vrednosti).

To znači da jedan `site_config.json` sadrži samo ono što je specifično za taj sajt.

To mi se veoma sviđa kao dizajn.

**Jedna mala digresija**  
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

**Sledeći korak**  
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

</br>

**Za početak Faze 4**  
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

Ono što je najlepše: **sve radi**.

Sada ćemo uraditi nešto što retko rade tutorijali. Umesto da kažemo "pokreni `bench start`", objasnićemo zašto se pokreće baš ovih sedam procesa.

</br>

**Bench nije server**
Ovo je prva važna stvar. Mnogi početnici misle: "Pokrenuo sam Bench."  
Ne. Pokrenuo si **orkestrator**.

`bench start` ne izvršava Frappe direktno. On pokreće više nezavisnih procesa i nadgleda ih.

Praktično radi nešto ovako:

```text
                bench start
                     │
     ┌───────────────┼────────────────┐
     │               │                │
   web.py       redis_cache      redis_queue
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

Pogledaj prvih nekoliko redova:

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
  
</br>

- **WEB**
  
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

</br>

- **SOCKETIO**

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

</br>

- **REDIS CACHE**
  
  Prvi Redis.
  
  ```text
  port 13000
  ```
  
  Ovaj Redis služi kao memorijski keš.
  
  Na primer:
  
  ```txt
  Korisnik -> Permissions -> Redis Cache -> sledeći zahtev -> ne čita bazu ponovo
  ```
  
  Time se štedi mnogo SQL upita.

</br>

- **REDIS QUEUE**
  
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
  
</br>

- **WORKER**
  
  Jedan od mojih omiljenih procesa.
  
  ```text
  worker.1
  ```
  
  Njegov posao je veoma jednostavan.  
  
  Beskonačna petlja.
  
  ```txt
  Ima li nešto u Queue? -> nema -> čekaj -> ima -> izvrši -> čekaj
  ```
  
  To je sve.  
  
  Ali zahvaljujući njemu browser ostaje brz.
  
</br>

- **SCHEDULER**
  
  Ovo je nešto kao cron.
  
  Na primer:
  
  ```txt
  svakih 5 minuta -> pokreni cleanup
  ```
  
  ili
  
  ```text
  svake noći -> backup
  ```
  
  ili
  
  ```text
  svakih sat vremena -> sync
  ```
  
  Scheduler ne izvršava posao.
  On samo kaže:
  
  "Vreme je."
  
  Posao zatim ubaci u Queue.
  Worker ga izvrši.
  Primeti kako se procesi lepo nadovezuju.

</br>

- **WATCH**
  
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

</br>

**Jedan primer**  
Recimo da klikneš u browseru
  
```txt
Create Customer
```

Šta se dešava?

Otprilike:

```txt
Browser -> HTTP -> WEB -> Python -> PostgreSQL -> vrati rezultat -> Browser
```

Ali ako taj Customer treba da pošalje email dobrodošlice:

```txt
WEB -> Queue -> Redis Queue -> Worker -> SMTP -> Email poslat
```

Browser u međuvremenu već prikazuje stranicu.

Ne čeka email.

</br>

**A ako email traje 20 sekundi?**  
Nema problema.
Worker radi.
Browser radi.
SocketIO može da kaže:

```txt
Email sent.
```

bez refresh-a.

</br>

**Šta je zapravo Frappe?**

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
 ┌───────┼──────────────────┐
 │               │          │
 ▼               ▼          ▼
PostgreSQL Redis Queue Redis Cache
           │
           ▼
         Worker
```

To je mnogo bliže stvarnoj slici nego jednostavno reći "Frappe je Python web framework".

</br>

**Dve zanimljive sitnice iz tvog izlaza**:

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

</br>

**Prva poseta sajtu**  
Sada prelazimo na ono što ja smatram prvim pravim susretom sa Frappe-om. Do sada smo bili "ispod haube". Sada ćemo prvi put pogledati kako izgleda sistem iz ugla korisnika, ali ćemo ga posmatrati iz ugla programera.

</br>

- **Otvori Frappe u browseru**

  Pošto radiš u VM-u, trebalo bi da možeš da otvoriš:
  
  ```txt
  http://192.168.122.74:8000
  ```
  
  ili, ako koristiš SSH tunel ili port forwarding, odgovarajuću adresu na hostu.
  
  Trebalo bi da dobiješ login ekran.
  
  Nemoj još ništa da istražuješ. Samo potvrdi da se stranica otvara.

</br>

- **Jedan HTTP zahtev**

  Pre nego što se uloguješ, hajde da ispratimo jedan jedini HTTP zahtev.
  
  - Browser šalje:
  
    ```txt
    GET /
    ```
  
  - Web server prima zahtev.
  - Frappe kaže: "Koji sajt je tražen?". Pošto imaš samo jedan sajt (`site1.
    local`), odgovor je jednostavan.
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

- **Kako Frappe zna da treba da koristi baš `site1.local`?**

  Ovo je odlično pitanje.
  
  Na produkcionom serveru odgovor je jednostavan:
  
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
  
  Pa kako onda zna?
  Hajde da pogledamo.
  
  Pošalji rezultat ove komande:
  
  ```sh
  cat sites/common_site_config.json
  ```
  
  To je sledeći važan fajl u Bench arhitekturi.
  
  Zašto baš sada?
  
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
  
  ```txt
  Browser -> bench start-> Web server -> common_site_config.json -> site1.local -> site_config.json -> PostgreSQL
  ```
  
  Kada razumeš ovu putanju, razumeš kako Frappe "pronalazi" i pokreće jedan sajt. Posle toga će nam mnogo lakše biti da pređemo na DocType-ove, aplikacije i razvoj sopstvenog koda.

</br>

**Zajednička definicija za sve sajtove**  
`common_site_config.json`

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

Po mom mišljenju, `common_site_config.json` je "kontrolni centar" Bench-a, dok je `site_config.json` lična karta jednog sajta.

Hajde da ga rastavimo.

</br>

- **Dva nivoa konfiguracije**

  Već sada možeš da vidiš da postoje dva nivoa:
  
  ```txt
  sites/
      common_site_config.json [ 1. -> (važi za sve sajtove)]
      site1.local/
          site_config.json    [ 2. -> (samo za site1.local)]
  ```
  
  To je veoma elegantan dizajn. Globalne stvari pišu se jednom. Specifične stvari pišu se po sajtu.
  
</br>

- **Redis**

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

</br>

- **Web server**

  Ovde stoji
  
  ```json
  "webserver_port": 8000
  ```
  
  To je upravo ono što si video u izlazu:
  
  ```sh
  Running on http://127.0.0.1:8000
  ```
  
  Dakle Bench nije "pogodio" port. On ga je pročitao odavde.

</br>

- **SocketIO**

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

</br>

- **File watcher**

  ```json
  "file_watcher_port": 6787
  ```
  
  Ovo koristi Watch proces.  
  Ne koristi ga browser.  
  Ne koristi PostgreSQL.  
  Koristi ga razvojni alat.

</br>

- **Background workers**

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

</br>

- **Gunicorn workers**

  ```json
  "gunicorn_workers": 5
  ```
  
  Ovo te možda zbuni. Jer si video Flask development server. Zašto onda Gunicorn?
  Odgovor je jednostavan. `bench start` koristi flask ako razvojni server.
  
  U produkciji Bench koristi Gunicorn. Dakle ova vrednost će biti važna kasnije kada pređeš na produkciono okruženje.
  
</br>

- **Najzanimljivija stavka**

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

</br>

- **Korisnik Bench-a**

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
  
</br>

**Jedna mala "Aha!" stvar**  
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

</br>

**Aplikacija Frappe**
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

Do sada smo posmatrali Bench kao grad. Sada ulazimo u jednu zgradu — aplikaciju Frappe.

</br>

**Gde se nalazi**  
Kod tebe:

```text
frappe-bench/
└── apps/
    └── frappe/
```

Ali... To još nije Python paket. Uđi u direktorijum i pogledaj sadržaj:

```bash
cd apps/frappe
ls -alh
```

Pošalji izlaz.

Zašto ne idem odmah na `tree`?  
Namerno.  Frappe ima više od 3000 fajlova.  Ako odmah uradimo:  
  
```bash
tree
```

dobićeš zid teksta.

Nećemo ništa naučiti.

Želim da prvo vidimo vrh strukture*, pa ćemo onda ulaziti samo u one delove koji su važni.

</br>

**Šta ćemo tražiti?**

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

</br>

**Mali cilj za ovu fazu**
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

Prvo, želim da razdvojimo dve stvari koje se često mešaju.

</br>

**Pregled frape direktorijuma**:

- **Git repozitorijum**

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

- **Python paket**

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

- **Šta je sve ostalo?**

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

**Sledeći korak u aplikaciji Frappe**
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
  
</br>

**Šta je urađeno**
Do sada smo praktično složili sledeću mentalnu mapu:

```txt
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
  
</br>

**Plan za sledeći nastavak**
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

</br>

**Frappe direktorijum**
Pregled frape/frape direktorijuma

---
Evo izlaza

```sh
tree -L 1 frappe
frappe
├── api
├── app.py
├── apps.py
├── auth.py
├── automation
├── boot.py
├── build.py
├── cache_manager.py
├── change_log
├── client.py
├── commands
├── config
├── contacts
├── core
├── coverage.py
├── custom
├── data
├── database
├── defaults.py
├── deferred_insert.py
├── desk
├── email
├── exceptions.py
├── frappeclient.py
├── geo
├── gettext
├── handler.py
├── hooks.py
├── __init__.py
├── installer.py
├── integrations
├── locale
├── middlewares.py
├── migrate.py
├── model
├── modules
├── modules.txt
├── monitor.py
├── oauth.py
├── onboarding.py
├── parallel_test_runner.py
├── patches
├── patches.txt
├── permissions.py
├── printing
├── public
├── pulse
├── push_notification.py
├── __pycache__
├── query_builder
├── rate_limiter.py
├── realtime.py
├── recorder.py
├── search
├── sessions.py
├── share.py
├── social
├── templates
├── test_runner.py
├── tests
├── translate.py
├── translations
├── twofactor.py
├── types
├── utils
├── website
├── workflow
└── www
```

---

Mnogi početnici pogledaju ovaj spisak i pomisle: *"Ovo je haos."* Međutim, kada se grupiše po nameni, postaje prilično logičan.

</br>

- **Prvo zapažanje**

  Ako pogledaš ovaj direktorijum, videćeš da **nije** organizovan kao Django.
  
  Na primer, Django ima nešto poput:
  
  ```sh
  django/
      db/
      forms/
      views/
      urls/
      templates/
  ```
  
  Frappe je drugačije organizovan.
  
  Ovde je logika:
  
  > **"Grupiši kod po funkcionalnosti (feature), a ne po tipu fajla."**
  
  Zbog toga vidiš foldere kao što su:
  
  ```sh
  desk/
  email/
  printing/
  workflow/
  website/
  oauth/
  contacts/
  ```
  
  Svaki od njih predstavlja jednu veću celinu sistema.

</br>

- **Ja ga obično podelim na 7 velikih grupa**

  </br>
  
  - **Core framework**
    Ovo je srce Frappe-a.
  
    ```sh
    model/
    database/
    query_builder/
    utils/
    permissions.py
    exceptions.py
    hooks.py
    ```
  
    Ovo je nešto poput motora automobila.
    Ako bi pravio svoj framework, upravo bi ovde završio najveći deo posla.
  
  </br>
  
  - **HTTP/Web**
    Ovde počinje svaki zahtev iz browsera.
  
    ```sh
    app.py
    handler.py
    middlewares.py
    api/
    www/
    website/
    templates/
    ```
  
    ```txt
    Browser -> Nginx -> Gunicorn -> app.py -> handler.py -> Python
    ```
  
    To je ceo tok.
  
  </br>
  
  - **Desk (ERP interfejs)**
  
    ```sh
    desk/
    boot.py
    client.py
    ```
  
    Ovo je ono što korisnik vidi nakon logovanja.
    Dakle:
  
    - Workspace
    - List View
    - Form View
    - Report View
    - Search
    - Notifications

    Sve to živi ovde.
  
  </br>
  
  - **ORM i DocType**
  
    Za mene je ovo najlepši deo Frappe-a.
  
    ```sh
    model/
    database/
    modules/
    custom/
    workflow/
    ```
  
    Ovde nastaje ono po čemu je Frappe poznat.
  
    ```txt
    DocType -> Python objekat -> ORM -> SQL
    ```
  
  </br>
  
  - **Poslovni moduli**
  
    Ovo nisu framework delovi.
    To su ugrađene funkcionalnosti.
  
    ```sh
    contacts/
    email/
    printing/
    geo/
    social/
    oauth/
    workflow/
    ```
  
    Drugim rečima:  
    Framework kaže: "Evo kako se pravi modul."  
    Ovi folderi su: "Evo modula napravljenih pomoću tog framework-a."
  
  </br>
  
  - **Administracija**
  
    ```sh
    installer.py
    migrate.py
    patches/
    commands/
    tests/
    ```
  
    Ovde Bench i Frappe rade:
  
    - instalaciju
    - migracije
    - update
    - testove
  
  </br>
  
  - **Statički sadržaj**
  
    ```sh
    public/
    locale/
    translations/
    gettext/
    ```
  
    CSS  
    JS  
    ikonice  
    prevodi  
    fontovi  

- **Šta je najvažnije**

  Po meni nisu svi ovi folderi jednako važni.
  
  Ja bih ih rangirao ovako:
  
  ```sh
  ⭐⭐⭐⭐⭐
  
  model/
  database/
  desk/
  ```
  
  ```sh
  ⭐⭐⭐⭐
  
  api/
  utils/
  permissions.py
  hooks.py
  ```
  
  ```sh
  ⭐⭐⭐
  
  website/
  www/
  templates/
  ```
  
  ```sh
  ⭐⭐
  
  email/
  printing/
  workflow/
  oauth/
  ```
  
  ```sh
  ⭐
  
  social/
  geo/
  pulse/
  ```

  Njih možeš proučavati i mnogo kasnije.

</br>

- **Šta bih ja voleo da prvo razumemo**

  Ako bismo učili Frappe kao da zajedno pravimo sopstveni framework, redosled bi bio:
  
  ```txt
  model/ -> database/ -> query_builder/ -> Document -> DocType -> Meta -> Desk -> API -> Website
  ```
  
  Zašto baš ovako?
  
  Zato što sve u Frappe-u počinje od `Document` klase. Bukvalno sve.
  
  Customer  
  Item  
  Sales Invoice  
  User  
  Role  
  Project  
  Task  
  
  ...sve su oni samo izvedene forme jednog osnovnog koncepta: `Document`.
  
</br>

**Frappe model**
Mislim da je sada pravi trenutak da ne "skačemo" po direktorijumima, već da pratimo jednu nit kroz framework.

Predložio bih sledeći put:

1. `frappe/model/`
2. Šta je `Document`.
3. Šta je `Meta`.
4. Kako jedan DocType postaje Python objekat.
5. Kako taj objekat završava u PostgreSQL bazi.

Po mom iskustvu, kada razumeš tih pet koraka, razumeo si oko 70% Frappe arhitekture. Sve ostalo (Desk, API, Website, Workflow...) uglavnom koristi iste osnovne mehanizme.

Meni se ovaj pristup čini prirodnijim nego da obilazimo direktorijume jedan po jedan, jer od početka povezuje strukturu projekta sa načinom na koji framework zaista funkcioniše.

Sada krećemo na deo koji ja smatram **srcem Frappe-a**.

I sada ću usporiti. Nećemo juriti kroz kod. Cilj nije da naučimo napamet gde je koji fajl, nego da razumemo zašto postoji.

- **Prvo jedno pitanje**

  Kada kažeš: "Napravio sam DocType `Customer`."
  
  šta se zapravo desilo?
  
  Da li je napravljen:
  
  - SQL tabela?
  - Python klasa?
  - HTML forma?
  - JavaScript?
  - REST API?
  
  Odgovor je... Sve od navedenog.
  
  I upravo zato Frappe deluje "magično".
  
  Ali nema magije. Samo dosta automatizacije.

**`model/`**

Molim te pošalji izlaz:

```bash
cd ~/frappe-bench/apps/frappe/frappe/model
tree -L 1
```

Ako nemaš `tree`:

```bash
ls -al
```

Nemoj još dublje. Samo prvi nivo.

</br>

- **Zašto baš `model/`**

  Zamisli da praviš sopstveni framework.
  
  Najpre bi morao da odgovoriš na pitanje: Kako predstavljam jedan red u bazi?
  
  Recimo tabela:
  
  | id | first_name | last_name |
  | -- | ---------- | --------- |
  | 1  | Marko      | Marković  |
  
  U čistom Pythonu mogao bi da napišeš:
  
  ```python
  class Person:
      def __init__(self):
          self.first_name = ""
          self.last_name = ""
  ```
  
  U Django-u bi napisao:
  
  ```python
  class Person(models.Model):
      first_name = models.CharField(...)
  ```
  
  U SQLAlchemy:
  
  ```python
  class Person(Base):
      ...
  ```
  
  A u Frappe-u ne pišeš skoro ništa.
  
  Zašto?
  
  Zato što će DocType definicija postati Python objekat u vreme izvršavanja (runtime).
  
  To je ogromna razlika.

</br>

- **Jedna analogija**

  Većina framework-a radi ovako:
  
  ```txt
  Python klasa -> SQL tabela
  ```
  
  Frappe radi obrnuto:
  
  ```txt
  DocType (meta podaci) -> Meta objekat -> Document objekat -> SQL tabela
  ```
  
  > Drugim rečima:
  >
  > U Frappe-u baza ne prati Python klase, već Python prati definiciju DocType-a.
  
  To je razlog zašto možeš da napraviš novi DocType iz Desk interfejsa, bez pisanja Python klase, a framework ipak zna kako da kreira, učita, sačuva i validira te dokumente.

- **Šta ćemo tražiti u `model/`**

  Kada dobijemo spisak fajlova i direktorijuma, nećemo odmah otvarati sve. Tražićemo odgovore na četiri pitanja:
  
  1. Gde je definisana osnovna `Document` klasa?
  2. Gde se čuvaju meta podaci o DocType-ovima?
  3. Kako Frappe učitava dokument iz baze?
  4. Kako ga pretvara nazad u SQL kada pozoveš `save()`?
  
  To je "kičma" celog framework-a. Kada to razumeš, sve ostalo — Desk, API, Workflow, čak i ERPNext — postaje mnogo lakše za praćenje.
  
  Pošalji izlaz `tree -L 1` za `frappe/model`, pa nastavljamo korak po korak.

---

Evo izlaza

```sh
tree -L 1 frappe/model
frappe/model
├── base_document.py
├── create_new.py
├── db_query.py
├── delete_doc.py
├── docfield.py
├── docstatus.py
├── document.py
├── dynamic_links.py
├── __init__.py
├── mapper.py
├── meta.py
├── naming.py
├── __pycache__
├── rename_doc.py
├── sync.py
├── utils
├── virtual_doctype.py
└── workflow.py
```

I sada dolazimo do dela gde bih voleo da radimo malo drugačije nego što to rade većina tutorijala. Nećemo ići redom po fajlovima. Umesto toga, pokušaćemo da odgovorimo na jedno pitanje:

> "Šta se desi kada napišem `frappe.get_doc(...)` ili kada kliknem `Save` na formi?"

Ako to ispratimo, videćemo gotovo ceo `model` direktorijum u prirodnom redosledu.

</br>

**Pogledaj ova imena**:

```text
base_document.py
document.py
meta.py
docfield.py
```

Da li primećuješ nešto?

Nisu nazvani:

- customer.py
- sales_invoice.py
- employee.py

nego:

- Document
- Meta
- DocField

To znači da Frappe ne razmišlja o "Customer-u". Razmišlja o bilo kom dokumentu. To je veoma važna filozofija.

</br>

**Postoje četiri glavna pojma**
Ja ih zamišljam ovako:

```txt
DocType -> Meta -> Document -> Database
```

Svaki od ova četiri ima svoju ulogu.

</br>

- **DocType**
  Ovo već poznaješ.  
  Na primer: Customer ili Item ili Task  
  DocType nije Python klasa.  
  DocType je opis.  
  Na primer:  
  
  ```txt
  Customer
    Fields:
      name
      email
      phone
      country
  ```
  
  To su samo podaci.

- **Meta**
  Ovde dolazi `meta.py`.  
  Meta odgovara na pitanje: "Kako izgleda Customer?"
  Ne: "Koji je Customer?" nego: "Šta Customer uopšte jeste?"  
  Drugim rečima:

  ```txt
  Customer
    field 1 = Data
    field 2 = Link
    field 3 = Check
    field 4 = Date
  ```

  Meta opisuje strukturu.  
  Ne sadržaj.

</br>

- **Document**
  Tek ovde dolazi jedan konkretan zapis.  
  Na primer:
  
  ```txt
  Customer
  Name: Petar
  Phone: 12345
  ```
  
  To više nije opis.  
  To je konkretan objekat.

- **Database**
  Na kraju se sve pretvori u SQL.

</br>

**Analogija**  
Recimo da praviš Excel.
  
Meta bi bila:

```txt
Kolona A
Kolona B
Kolona C
```

Document bi bio:

```txt
Marko
Beograd
25
```

Vidiš razliku?

Meta opisuje tabelu.  
Document predstavlja jedan red.

</br>

**Da pogledamo ova četiri fajla**:

- `base_document.py` -> Osnovna funkcionalnost.  
- `document.py` -> Pravi Document.  
- `meta.py` -> Opis DocType-a.  
- `docfield.py` -> Opis jednog polja.  

Već sada možeš da naslutiš hijerarhiju:

```txt
Meta
  ├── DocField
  ├── DocField
  ├── DocField
```

a zatim:

```txt
Document
  ├── value 1
  ├── value 2
  ├── value 3
```

To je veoma elegantna ideja.

</br>

**Ono što mene posebno fascinira**  
Većina ORM-ova radi ovako:

```py
class Customer(Model):
    name = CharField(...)
```

Kod je opis modela.

U Frappe-u je obrnuto.

Model se nalazi u bazi (kao DocType definicija), a Python ga učitava u `Meta` objekat.

To znači da framework može da radi sa DocType-om koji nije postojao kada je Frappe pokrenut.

Napraviš novi DocType u Desk-u.

Ne restartuješ server. Odmah radi.

To je moguće upravo zahvaljujući ovom sloju `Meta`.

</br>

**Šta ćemo sada?**  
Sada više nema smisla da obilazimo direktorijume.

Došli smo do mesta gde počinje "život" svakog dokumenta.

Predlažem da sledeći korak bude da otvorimo `document.py`, ali ne da čitamo ceo fajl (ima dosta koda), već da pronađemo:

- kako izgleda klasa `Document`,
- od koje klase nasleđuje,
- koje su njene najvažnije metode (`insert`, `save`, `delete`, `db_insert`, `db_update`...).

Na taj način ćemo pratiti životni ciklus jednog dokumenta od njegovog nastanka do upisa u PostgreSQL. Mislim da će ti to dati mnogo bolju sliku nego da analiziramo fajlove izolovano.
  
</br>

**Frappe iznutra**  
Ali pre nego što otvorimo `document.py`, želim da ti pokažem kako ja čitam veliki framework. To je metod koji koristim godinama kada upoznajem novi projekat (Linux kernel, Django, PostgreSQL, Qt...). Nikada ne krenem od prvog reda fajla.

Krenem od pitanja: Šta je glavna klasa ovog modula?

**Pronađi klasu**  
Pošalji izlaz:

```bash
grep "^class " ~/frappe-bench/apps/frappe/frappe/model/document.py
```

Nemoj još otvarati ceo fajl.

Želim prvo da vidimo koje klase postoje.

</br>

**Zašto?**  
Zamisli da si ušao u biblioteku od 3000 knjiga. Nećeš otvoriti prvu stranu prve knjige. Prvo pogledaš sadržaj. Kod je isti.
Kada vidimo klase, pronaći ćemo njihove metode.

Na primer:

```bash
grep "^ def " document.py
```

ili

```bash
grep "^    def " document.py
```

(zavisi da li koristi tabove ili razmake).

Time ćemo dobiti nešto poput:

```py
__init__
save
insert
delete
...
```

I odjednom će fajl od nekoliko hiljada linija postati mapa od dvadesetak metoda.

To je mnogo lakše za razumevanje.

</br>

**Zašto ovo radimo?**

Većina ljudi čita framework ovako:

```txt
linija 1
linija 2
linija 3
...
```

Posle 15 minuta više ne znaju šta čitaju.

Ja ga čitam ovako:

```txt
Modul -> Klase -> Metode -> Jedna metoda -> Pozivi drugih metoda
```

To je kao da prvo pogledaš mapu grada, pa tek onda kreneš ulicama.

</br>

**Jedna mala digresija**  
Ovo nije samo korisno za Frappe.

Ako jednog dana budeš čitao:

- Django,
- SQLAlchemy,
- Flask,
- Linux kernel,
- PostgreSQL izvorni kod,

isti pristup će ti uštedeti mnogo vremena.

Dakle, pošalji izlaz:

```bash
grep "^class " ~/frappe-bench/apps/frappe/frappe/model/document.py
```

Pa ćemo zajedno napraviti "mapu" `Document` klase pre nego što uđemo u njenu implementaciju. Mislim da će ti se ovaj način rada dopasti, jer ćemo umesto šume odmah videti drveće, a zatim polako prići svakom stablu.

---

Evo izlaza:

```sh
grep "^class " ~/frappe-bench/apps/frappe/frappe/model/document.py
class Document(BaseDocument):
```

---

E sad dolazimo do jednog lepog trenutka. Ovaj izlaz:

```python
class Document(BaseDocument):
```

govori mnogo više nego što izgleda na prvi pogled.

</br>

**Pitanje**
Zašto nije napisano:

```python
class Document:
```

nego

```python
class Document(BaseDocument):
```

To znači da je autor Frappe-a rekao: "Postoje stvari koje važe za svaki dokument, i postoje stvari koje važe samo za Frappe Document."

Drugim rečima:

```txt
BaseDocument
    ▲
    │
    │ nasleđivanje
    │
Document
```

Ovo je veoma uobičajen obrazac u velikim framework-ovima.

</br>

- **`BaseDocument`**

  Pre nego što uopšte otvorimo taj fajl, hajde da pogađamo.
  
  Po nazivu bih očekivao da sadrži stvari kao što su:
  
  - čuvanje atributa (`self.name`, `self.owner`, ...)
  - pristup poljima (`doc.email`)
  - pretvaranje u `dict`
  - iteraciju kroz polja
  - osnovne pomoćne metode
  
  Drugim rečima... ništa što ima veze sa bazom.

</br>

- **`Document`**

  E tu već očekujem Frappe logiku.
  
  Na primer:
  
  - `insert()`
  - `save()`
  - `delete()`
  - `submit()`
  - `cancel()`
  - validaciju
  - hook-ove
  - dozvole
  - komunikaciju sa bazom
  
  Drugim rečima:
  
  ```txt
  BaseDocument
      │
      ├── čuva podatke
      ├── ponaša se kao objekat
      └── pomoćne metode
  
  Document
      │
      ├── ORM
      ├── SQL
      ├── Hooks
      ├── Permissions
      └── Business Logic
  ```

</br>

**Zašto je ovo dobro projektovanje?**

Zamisli da sutra Frappe želi da uvede još jednu vrstu dokumenta.
Na primer:

```txt
BaseDocument
        │
   ┌────┴─────┐
   │          │
Document   VirtualDocument
```

Oba mogu koristiti iste osnovne mehanizme.

To je jedan od razloga zašto postoji `virtual_doctype.py`, koji si video u spisku fajlova.

Dakle, `BaseDocument` nije napravljen slučajno.

</br>

**Šta ćemo sada dalje?**

Sada bih voleo da ne otvorimo odmah `document.py`, već da proverimo da li je moja pretpostavka tačna.

Hajde da pogledamo koje metode ima `BaseDocument`.

Pošalji izlaz:

```bash
grep "^    def " ~/frappe-bench/apps/frappe/frappe/model/base_document.py
```

Ako ništa ne vrati (zbog drugačije indentacije), probaj:

```bash
grep "def " ~/frappe-bench/apps/frappe/frappe/model/base_document.py
```

</br>

**Zašto baš `BaseDocument` prvo?**
Zato što želim da razdvojimo dva sloja:

- Mehaniku Python objekta (`BaseDocument`),
- Frappe ORM ponašanje (`Document`).
Po mom iskustvu, to je mnogo lakše za razumevanje nego odmah uskočiti u `save()` i `insert()`, gde se odjednom pojavljuju dozvole, validacije, hook-ovi, transakcije i SQL.

</br>

**Mala napomena**
Ovo nije jedini mogući redosled učenja. Mogli bismo odmah analizirati `save()`. Ja biram ovaj put zato što gradi mentalni model od temelja ka složenijim slojevima. Kasnije ćeš mnogo lakše razumeti zašto `Document` radi to što radi, jer ćeš već znati šta je nasledio od `BaseDocument`.

---

Evo izlaza

```txt
grep "def " ~/frappe-bench/apps/frappe/frappe/model/base_document.py

def get_controller(doctype):
def import_controller(doctype):
def __init__(self, d):
def meta(self):
def permitted_fieldnames(self) -> set[str]:
def __getstate__(self):
def remove_unpicklable_values(self, state):
def update(self, d):
def update_if_missing(self, d):
def get_db_value(self, key):
def get(self, key, filters=None, limit=None, default=None):
def getone(self, key, filters=None):
def set(self, key, value, as_value=False):
def delete_key(self, key):
def append(self, key: str, value: D | dict | None = None, position: int = -1) -> D:
def parent_doc(self):
def parent_doc(self, value):
def parent_doc(self):
def extend(self, key, value):
def remove(self, doc):
def _init_child(self, value, key):
def _get_table_fields(self):
def get_valid_dict(
def init_child_tables(self):
def init_valid_columns(self):
def get_valid_columns(self) -> list[str]:
def is_new(self) -> bool:
def docstatus(self) -> DocStatus:
def docstatus(self, value) -> None:
def as_dict(
def as_json(self):
def get_table_field_doctype(self, fieldname):
def get_parentfield_of_doctype(self, doctype):
def db_insert(self, ignore_if_duplicate=False):
def db_update(self):
def db_update_all(self):
def show_unique_validation_message(self, e):
def get_field_name_by_key_name(self, key_name):
def get_label_from_fieldname(self, fieldname):
def update_modified(self):
def _fix_numeric_types(self):
def _get_missing_mandatory_fields(self):
def get_msg(df):
def has_content(df):
def get_invalid_links(self, is_submittable=False):
def get_msg(df, docname):
def set_fetch_from_value(self, doctype, df, values):
def _validate_selects(self):
def _validate_data_fields(self):
def _validate_constants(self):
def _validate_length(self):
def _validate_code_fields(self):
def _sync_autoname_field(self):
def throw_length_exceeded_error(self, df, max_length, value):
def _validate_update_after_submit(self):
def _sanitize_content(self):
def _save_passwords(self):
def get_password(self, fieldname="password", raise_exception=True):
def is_dummy_password(self, pwd):
def precision(self, fieldname, parentfield=None) -> int | None:
def get_formatted(
def is_print_hide(self, fieldname, df=None, for_print=True):
def __setup__(self):
def in_format_data(self, fieldname):
def reset_values_if_no_permlevel_access(self, has_access_to, high_permlevel_fields):
def get_value(self, fieldname):
def cast(self, value, df):
def _extract_images_from_text_editor(self):
def _filter(data, filters, limit=None):
```

---

I sada ću uraditi nešto što će ti, nadam se, pomoći da ne "utoneš" u more metoda.

</br>

- **Prvo zapažanje**

  Kada si pogledao spisak, da li si primetio nešto što nije odgovaralo mojoj pretpostavci?
  
  Ja sam rekao: "Verovatno `BaseDocument` nema ništa vezano za bazu."
  
  A onda vidimo:
  
  ```text
  db_insert()
  db_update()
  db_update_all()
  ```
  
  Dakle, moja pretpostavka je bila delimično pogrešna.
  
  To je važna lekcija kada čitamo tuđi kod: uvek pravimo hipotezu, pa je proverimo. Nekad je pogodimo, nekad ne. To nije greška – to je način na koji se uči arhitektura.

**Grupisanje metode**
Ovaj spisak od 60+ metoda izgleda zastrašujuće, ali u stvari može da se podeli u nekoliko logičkih grupa.

</br>

- **Konstrukcija objekta**:

  ```py
  __init__()
  update()
  update_if_missing()
  ```

  Ove metode služe da naprave i popune objekat.

</br>

- **Pristup podacima**:

  ```py
  get()
  getone()
  set()
  append()
  extend()
  remove()
  delete_key()
  get_value()
  ```
  
  Ovo je interfejs za rad sa poljima dokumenta.

</br>

  **Serijalizacija**:
  
  ```text
  as_dict()
  as_json()
  __getstate__()
  ```

  Pretvaranje objekta u rečnik, JSON ili stanje za pickle.

</br>

**Meta informacije**:

  ```text
  meta()
  get_valid_columns()
  get_table_field_doctype()
  ```
  
  Ovde se vidi veza sa `Meta` objektom o kome smo pričali.

</br>

**Validacija**:

  Ovde ih ima mnogo:
  
  ```text
  _validate_length()
  _validate_selects()
  _validate_data_fields()
  _validate_constants()
  ...
  ```
  
  To znači da se dosta validacije dešava na nivou samog dokumenta.

</br>

**Baza**:

  ```text
  db_insert()
  db_update()
  db_update_all()
  ```
  
  Ovo je zanimljivo.
  
  Obrati pažnju na prefiks `db_`.
  
  To mi govori da autor pravi razliku između:
  
  ```py
  save()
  i
  db_update()
  ```
  
  To je veoma dobra praksa.
  
  Zašto?
  
  Zato što `save()` verovatno radi mnogo više od običnog SQL `UPDATE`.
  
  Na primer:
  
  ```txt
  save() -> permissions -> hooks -> validate -> before_save -> db_update() -> 
  after_save 
  ```
  
  Dok:
  
  ```py
  db_update()
  ```
  
  verovatno radi samo posao sa bazom.
  
  To ćemo kasnije proveriti.

</br>

**Ono što mi je posebno zapalo za oko**:

Pogledaj ove metode:

```py
meta()
get_valid_columns()
permitted_fieldnames()
```

One mi odmah govore da `BaseDocument` zna šta je njegov DocType.

Drugim rečima:

```txt
Document -> Meta -> DocField
```

nije samo teorija.

Već sada vidimo tragove te veze u samom kodu.

</br>

**Hajde da proverimo jednu hipotezu**:  
Po meni, najvažnija metoda u celom fajlu nije `db_update()`. Nije ni `append()`.
Nije ni `get()`.
  
Ja mislim da je to:

```python
@property
def meta(...)
```

Zašto?

Jer bez `Meta` objekta `BaseDocument` ne zna:

- koja polja postoje,
- koji je tip svakog polja,
- koja su obavezna,
- koja predstavljaju child tabele,
- koja su `Link` polja,
- koja su `Date`, `Currency`, `Check`, itd.

Drugim rečima, `Meta` je "uputstvo za upotrebu" svakog dokumenta.

</br>

**Meta property**  
Umesto da odmah analiziramo `save()`, voleo bih da otvorimo baš tu `meta` osobinu. To je mali komad koda, ali mislim da će nam objasniti kako se `Document` povezuje sa `Meta`.

Možeš da pošalješ izlaz:

```bash
grep -A 8 "def meta" ~/frappe-bench/apps/frappe/frappe/model/base_document.py
```

ili, ako vidiš da je metoda duža, slobodno:

```bash
sed -n '40,70p' ~/frappe-bench/apps/frappe/frappe/model/base_document.py
```

(tačan opseg nije bitan; cilj je da obuhvatimo `@property` i telo metode).
**Jedna napomena za kraj**
Želim da uvedemo još jedno pravilo u naše proučavanje Frappe-a.

</br>

- **Nećemo analizirati ceo fajl odjednom.**

  Umesto toga, svaki put ćemo izabrati jednu važnu ideju (kao što je `Meta`, `Document`, `save()` ili `get_doc()`), razumeti je do kraja, pa tek onda nastaviti dalje. Po mom iskustvu, tako se veliki framework zaista nauči, umesto da ostane samo utisak da si "prošao kroz kod".
  
---

Evo izlaza

```sh
grep -A 8 "def meta" ~/frappe-bench/apps/frappe/frappe/model/base_document.py

def meta(self):
  return frappe.get_meta(self.doctype)

@cached_property
def permitted_fieldnames(self) -> set[str]:
  return set(get_permitted_fields(doctype=self.doctype, parenttype=getattr(self,  
    "parenttype", None)))

def __getstate__(self):
```

---

Pogledaj ponovo:

```python
def meta(self):
    return frappe.get_meta(self.doctype)
```

Na prvi pogled deluje gotovo razočaravajuće jednostavno.

Ali u stvari, ovde se krije jedna od najvažnijih ideja Frappe-a.

</br>

- **Šta znači `self.doctype`?**

  Zamisli da imaš objekat:
  
  ```python
  customer = frappe.get_doc("Customer", "CUST-0001")
  ```
  
  Taj objekat u sebi ima nešto poput:
  
  ```python
  customer.doctype == "Customer"
  ```
  
  Dakle, `doctype` nije Python klasa.
  
  To je običan string.

</br>

- **Šta se dešava kada pozoveš `customer.meta`?**

  Izvrši se:
  
  ```python
  frappe.get_meta("Customer")
  ```
  
  Dakle:
  
  ```txt
  Customer Document -> doctype = "Customer" -> frappe.get_meta("Customer") -> Meta objekat
  ```
  
  I sad dolazimo do ključne ideje.

</br>

- **Šta je zapravo Meta?**

  Nemoj ga zamišljati kao "fajl". Zamisli ga kao opis dokumenta u memoriji.
  
  Na primer:
  
  ```txt
  Meta
    doctype = Customer
      fields
        name
        email
        phone
        country
        customer_group
        territory
  ```
  
  To nije red iz baze.  
  To je opis strukture.

</br>

- **Zašto je ovo genijalno?**

  Pogledaj šta nigde ne postoji.
  
  Ne postoji:
  
  ```python
  class Customer:
  ```
  
  Ozbiljno. Frappe ne mora da ima Python klasu za svaki DocType.
  Umesto toga radi:
  
  ```txt
  Customer -> get_meta() -> Meta -> Document
  ```
  
  To je ogromna razlika u odnosu na Django.

</br>

- **Upoređenje sa Django**

  **U Django-u:**
  
  ```python
  class Customer(models.Model):
      email = ...
      phone = ...
  ```
  
  Klasa **opisuje** bazu.
  
  **U Frappe-u:**
  
  ```txt
  DocType -> Meta -> Document
  ```
  
  Klasa je ista za sve.
  
  Menja se samo Meta.
  
  To znači... Ako napraviš novi DocType u Desk-u:

  ```txt
  Vehicle
  ```
  
  Frappe ne generiše:

  ```py
  class Vehicle(Document):
  ```

  Nego samo napravi novi Meta zapis.

  Kasnije:

  ```py
  frappe.get_doc("Vehicle")
  ```

  koristi isti `Document`.

  To je veoma elegantan dizajn.

</br>

- **A šta radi `frappe.get_meta()`?**

  E sad dolazimo do pitanja koje će nas odvesti dalje.
  
  Ova metoda:
  
  ```python
  frappe.get_meta("Customer")
  ```
  
  Mora negde da:
  
  - pročita DocType,
  - pročita DocField,
  - napravi Meta objekat,
  - kešira ga,
  - vrati ga.
  
  Drugim rečima:
  
  ```txt
  Database -> DocType -> DocField -> Meta() -> cache -> return
  ```

</br>

- **Obrati pažnju na još jednu sitnicu**

  Metoda nije:
  
  ```py
  def get_meta(self):
  ```
  
  nego
  
  ```py
  @property
  def meta(self):
  ```
  
  To znači da pišeš:
  
  ```py
  doc.meta
  ```
  
  a ne
  
  ```python
  doc.get_meta()
  ```
  
  Zašto?
  
  Zato što se autor trudi da `meta` izgleda kao osobina dokumenta, a ne kao neka operacija.
  
  To lepo čitaš:
  
  ```python
  doc.meta.fields
  ```
  
  umesto:
  
  ```python
  doc.get_meta().fields
  ```
  
  To je mala stvar, ali doprinosi čitljivosti.

</br>

- **A odmah ispod...**

  Vidimo još nešto zanimljivo:
  
  ```python
  @cached_property
  def permitted_fieldnames(...)
  ```
  
  Ovo je odličan primer modernog Pythona.
  
  Prvi put kada pristupiš:
  
  ```python
  doc.permitted_fieldnames
  ```
  
  izračunaće skup dozvoljenih polja.
  
  Drugi put... neće ponovo računati. Vratiće vrednost iz keša. To je važno jer se dozvole mogu proveravati mnogo puta tokom jednog zahteva.

</br>

**Tok izvršenja u Frappeu**  
Mislim da je vreme da prestanemo da gledamo pomoćne metode i da krenemo da pratimo jedan konkretan tok izvršavanja.

Na primer:

```py
doc = frappe.get_doc("Customer", "CUST-0001")
```

To je savršena početna tačka.

Od tog jednog poziva možemo pratiti:

```txt
get_doc() -> Document -> Meta -> Database -> SQL -> Python objekat
```

To je, po mom mišljenju, mnogo uzbudljivije nego da analiziramo metodu po metodu. Počećemo da pratimo kako Frappe "oživljava" jedan dokument iz baze, a usput ćemo prirodno nailaziti na `Document`, `Meta`, ORM i ostale ključne delove framework-a. Mislim da će ti se taj način proučavanja mnogo više dopasti, jer prati stvarni životni ciklus jednog dokumenta umesto strukture fajlova.

E sada dolazimo do dela koji je meni bio "aha!" trenutak kada sam prvi put ozbiljno proučavao Frappe.

Do sada smo gledali **strukturu**. Sada ćemo pratiti **izvršavanje**.

</br>

- **Naš cilj**

  Želimo da odgovorimo na jedno jedino pitanje:  
  Šta se tačno dogodi kada napišem:**
  
  ```python
  doc = frappe.get_doc("Customer", "CUST-0001")
  ```
  
  Ne zanima nas još SQL. Ne zanima nas još Desk. Samo ovaj jedan poziv.
  
</br>

- **Kako ja analiziram framework**

  Nikada ne pokušavam da pročitam 3000 linija koda.
  
  Tražim **ulaznu tačku**.
  
  Ovde je to:
  
  ```python
  frappe.get_doc(...)
  ```
  
  Zato ćemo pratiti samo njega.

</br>

- **Gde se nalazi `get_doc`?**

  Prvo, razmisli logički.
  
  Da si ti pisao Frappe, gde bi stavio funkciju koja se zove:
  
  ```python
  frappe.get_doc(...)
  ```
  
  Da li u:
  
  ```sh
  database/
  ```
  
  ili
  
  ```sh
  model/
  ```
  
  ili
  
  ```sh
  __init__.py
  ```
  
  Ja bih rekao:
  
  > `frappe/__init__.py`
  
  Zašto?
  
  Zato što je korisnik poziva kao:
  
  ```python
  frappe.get_doc(...)
  ```
  
  To znači da mora biti izložena na nivou paketa `frappe`.

</br>

- **Hajde da proverimo hipotezu**

  Pošalji:
  
  ```bash
  grep -n "def get_doc" ~/frappe-bench/apps/frappe/frappe/__init__.py
  ```
  
  Ako je tamo, nećemo odmah čitati telo funkcije.
  
  Hoću prvo da vidimo koliko je velika.

</br>

- **Zašto?**

  U velikim framework-ovima često postoji obrazac:
  
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

</br>

- **Usput jedna zanimljiva stvar**

  Već smo videli:
  
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
  frappe.get_doc() -> model.get_doc() -> Document(...)
  ```
  
  ili postoji još neki sloj između.
  
  To nećemo nagađati – proverićemo u kodu.

</br>

- **Zašto mi se dopada ovaj način rada?**

  Zato što više ne učimo "fajlove". Učimo **tok izvršavanja**.
  
  Kasnije ćemo isto uraditi za:
  
  - `doc.save()`
  - `doc.insert()`
  - `doc.delete()`
  - `frappe.db.get_value()`
  
  i polako ćeš početi da vidiš Frappe kao sistem kroz koji "teče" izvršavanje, a ne kao gomilu direktorijuma i fajlova. To je, po mom iskustvu, trenutak kada veliki framework počne da ima smisla.
  
---

Evo izlaza

```sh
radosav@frappe:~/frappe-bench/apps/frappe$ grep -n "def get_doc" ~/frappe-bench/apps/frappe/frappe/__init__.py
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

Odlično! I sada smo naišli na nešto veoma zanimljivo. Ovo je jedan od razloga zašto volim da se framework čita polako.

</br>

- **Prvo što upada u oči**

  Pogledaj ovo:
  
  ```text
  1269:def get_doc(document: "Document", /) -> "Document":
  1274:def get_doc(doctype: str, /) -> _SingleDocument:
  1280:def get_doc(doctype: str, name: str, /, *, for_update: bool | None = None) -> "Document":
  1286:def get_doc(**kwargs: dict) -> "_NewDocument":
  1293:def get_doc(documentdict: dict) -> "_NewDocument":
  1299:def get_doc(*args, **kwargs):
  ```
  
  Prva reakcija je obično: "Kako može Python da ima šest funkcija sa istim imenom?". Odgovor je: ne može.
  
  Ovo nisu različite implementacije. One služe za **type hinting**.

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
  
  Na primer, IDE može da zaključi:
  
  ```python
  frappe.get_doc("Customer", "CUST-0001")
  ```
  
  vraća:
  
  ```python
  Document
  ```
  
  dok
  
  ```python
  frappe.get_doc({"doctype": "Customer"})
  ```
  
  vraća novi dokument.
  
  Dakle, ovo nije "preopterećenje funkcija" kao u C++ ili Javi, već način da se jednom implementacijom opiše više načina korišćenja.
  
</br>

- **Ovo je lep primer zašto ne treba stati na prvi pogled**

Da smo samo preleteli fajl, mogli bismo pogrešno zaključiti da Python podržava overload.

A zapravo podržava samo:

- jednu implementaciju,
- više deklaracija za potrebe tipova.

</br>

- Sada nas zanima samo jedna stvar

Hoću da vidimo **pravu implementaciju**.

Pošalji nekoliko linija oko nje:

```bash
sed -n '1299, 1315p' ~/frappe-bench/apps/frappe/frappe/__init__.py
```

Ako se poziv nastavlja dalje, slobodno i malo više, recimo:

```bash
sed -n '1299,1325p' ~/frappe-bench/apps/frappe/frappe/__init__.py
```

- **Šta očekujem?**

Imam hipotezu.

Mislim da će implementacija biti veoma kratka.

Nešto u stilu:

```python
def get_doc(*args, **kwargs):
    import frappe.model.document
    return frappe.model.document.get_doc(*args, **kwargs)
```

ili nešto veoma slično.

Ako se to potvrdi, naučićemo još jednu važnu stvar o Frappe-u:

- `frappe.__init__.py` je **javni API**,
- prava logika je smeštena u drugim modulima.

To je obrazac koji ćeš verovatno viđati na mnogo mesta u Frappe-u.

--

Evo izlaza

```sh
sed -n '1299,1325p' ~/frappe-bench/apps/frappe/frappe/__init__.py
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


def get_single_value(setting: str, fieldname: str, /, *, as_dict: bool = False):
  """Return the cached value associated with the given fieldname from single DocType.

 Usage:
    telemetry_enabled = frappe.get_single_value("System Settings", "telemetry_enabled")
  """
```

I sada mogu da ti pokažem jedan obrazac koji se ponavlja kroz ceo Frappe.

</br>

- **Pogledaj koliko je funkcija mala**

  ```python
  def get_doc(*args, **kwargs):
      ...
      import frappe.model.document
  
      return frappe.model.document.get_doc(*args, **kwargs)
  ```

  To je praktično sve.
  
  Dakle... `frappe.get_doc()` ne radi gotovo ništa.

Njegov posao je samo da kaže: "Idi u `frappe.model.document` i tamo odradi pravi posao."

</br>

- **Zašto ovako?**

  Ovo je veoma lepo projektovanje.
  
  Zamisli da nije ovako.
  
  Morali bismo da pišemo:
  
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

- **Ovo se zove "Facade"**

  Ne znam da li si ranije nailazio na taj termin.

  U objektno orijentisanom projektovanju postoji obrazac:

  ```txt
  Korisnik -> Facade -> Komplikovan sistem
  ```
  
  Ovde je:
  
  ```txt
  Programer -> frappe.get_doc() -> frappe.model.document.get_doc() -> Document -> Database
  ```
  
  Programer vidi samo jednu funkciju.
  
  Sve ostalo je sakriveno.

</br>

- **A zašto `import` nije na vrhu fajla?**

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

  - **Hipoteza 1 — Izbegavanje kružnih zavisnosti**

    Zamisli:

    ```sh
    __init__.py -> document.py -> base_document.py -> frappe -> __init__.py
    ```

    I odjednom:

    ```py
    ImportError
    ```

    Lokalni `import` često rešava ovakve probleme.

  </br>

  - **Hipoteza 2 — Brže pokretanje**
  
    Ako nikad ne pozoveš `get_doc()`, nikad se neće importovati `document.py`.
    To može malo ubrzati startovanje.

  </br>

  - **Koja hipoteza je tačna?**

    Ne znamo.

    I nećemo nagađati.

    To je još jedna stvar koju ćemo proveriti kad budemo bolje upoznali zavisnosti između modula.

</br>

- **Ono što smo upravo naučili**

  Već sada imamo malu mapu.
  
  ```txt
  Program -> frappe.get_doc() -> frappe.model.document.get_doc() -> ??? -> Document
  ```
  
  Sada treba da otkrijemo ono **???**
  
</br>

- **Sledeći korak**

  Sada više nema razloga da ostajemo u `__init__.py`.
  
  Pravi posao je ovde:
  
  ```sh
  frappe/model/document.py
  ```
  
  Hajde da pronađemo sledeću funkciju.
  
  Pošalji:
  
  ```bash
  grep -n "^def get_doc" ~/frappe-bench/apps/frappe/frappe/model/document.py
  ```

</br>

- **Zašto mislim da će sada postati baš zanimljivo**

  Do sada smo pratili samo prosleđivanje poziva.
  
  Ali očekujem da ćemo u `document.py` prvi put videti kako Frappe odlučuje:
  
  - da li otvara postojeći dokument,
  - da li pravi novi dokument,
  - da li radi sa `Single DocType`,
  - da li dobija `dict`,
  - koju klasu treba da instancira.
  
  Drugim rečima, mislim da ćemo sada doći do **prvog pravog "dispečera" (dispatcher)** u Frappe-u. Tu će jedna funkcija, na osnovu argumenata koje dobije, odlučiti kojim putem će dalje krenuti. To je trenutak kada će naš dijagram toka početi da se grana, umesto da bude samo niz poziva.
  
  Ako slučajno ne vrati ništa (što je moguće ako je funkcija dekorisana ili drugačije formatirana), probaj:
  
  ```bash
  grep -n "def get_doc" ~/frappe-bench/apps/frappe/frappe/model/document.py
  ```
  
  ili još sigurnije:
  
  ```bash
  grep -n "@simple_singledispatch\|def get_doc" ~/frappe-bench/apps/frappe/frappe/model/document.py
  ```
  
</br>

- **Jedna napomena**

  Postoji jedna stvar koju sam namerno nisam unapred rekao.
  
  Na osnovu onoga što smo do sada videli (`frappe.get_doc()` samo prosleđuje poziv), očekujem da ćemo u `document.py` naići na nešto malo neobično. Ako se moja pretpostavka potvrdi, to će nam objasniti kako Frappe uspeva da podrži sve ove pozive:
  
  ```python
  frappe.get_doc("Customer", "CUST-0001")
  frappe.get_doc({"doctype": "Customer"})
  frappe.get_doc(existing_doc)
  frappe.get_doc("System Settings")
  ```
  
  ...a da ipak ima jednu javnu funkciju `get_doc()`.
  
  Neću da prejudiciram dok ne vidimo kod. Jedna od stvari koje pokušavam da negujemo tokom ovog proučavanja jeste da zaključujemo iz izvornog koda, a ne iz pretpostavki.
  
  Pošalji izlaz, pa nastavljamo da pratimo tok izvršavanja korak po korak.
  
---

Evo izlaza

```h
grep -n "^def get_doc" ~/frappe-bench/apps/frappe/frappe/model/document.py

36:def get_doc(*args, **kwargs):
```

---

E sad dolazimo do trenutka kada bih malo promenio tempo.

Do sada smo radili ovako:

```py
frappe.get_doc() -> frappe.model.document.get_doc()
```

To je bilo lako.

Sada prvi put ulazimo u pravi kod.

I ovde želim da uvedem jedno pravilo koje sam naučio čitajući velike projekte.

**Ne analiziramo funkciju po imenu. Analiziramo njeno telo.**

Zato sada više nije dovoljno da znamo da postoji `get_doc()`.

Moramo videti šta radi.

</br>

- **Sledeći korak**

  Pošalji prvih dvadesetak-tridesetak linija te funkcije:
  
  ```bash
  sed -n '36,70p' ~/frappe-bench/apps/frappe/frappe/model/document.py
  ```
  
  Ako se vidi da je funkcija duža, možemo proširiti na:
  
  ```bash
  sed -n '36,90p' ~/frappe-bench/apps/frappe/frappe/model/document.py
  ```
  
  Nemoj više od toga za sada.

- **Zašto baš toliko?**

  Zato što očekujem da ćemo vrlo brzo naići na neku od sledećih stvari:
  
  - `match` (Python 3.10+),
  - `if isinstance(...)`,
  - `singledispatch`,
  - ili neku drugu tehniku za "dispečiranje" različitih tipova argumenata.
  
  To će nam objasniti kako jedna funkcija uspeva da podrži toliko različitih načina poziva.
  
</br>

- **Jedna mala opaska**

  Primećuješ li kako se naš pristup promenio?
  
  Na početku smo istraživali:
  
  ```sh
  Direktorijumi -> Fajlovi -> Klase -> Metode
  ```
  
  Sada više ne gledamo strukturu projekta.
  
  Sada pratimo **jedan konkretan tok izvršavanja**.
  
  To je, po mom mišljenju, trenutak kada framework prestaje da bude "gomila fajlova" i počinje da se ponaša kao živi sistem.
  
---
Evo izlaza

```py
sed -n '36,90p' ~/frappe-bench/apps/frappe/frappe/model/document.py

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

---

Sada smo konačno stigli do mesta gde Frappe počinje da pokazuje svoju filozofiju. I iskreno, ovo je mnogo elegantnije nego što mnogi očekuju.

Hajde da ga pročitamo kao da smo autori Frappe-a.

</br>

- **Prvi utisak**

  Mene je odmah iznenadilo nešto.
  
  `get_doc()` uopšte ne učitava dokument iz baze.
  
  Nema: `SELECT ...`. Nema: `frappe.db.sql(...)`. Nema: `db.get(...)`.
  Ništa.  

  </br>
  To znači da ova funkcija ima sasvim drugu ulogu. Šta ona zapravo radi?
  Ja bih je nazvao:
  
</br>

- **Fabrika (Factory)**
  
  Ona ne pravi SQL.  Ona odlučuje:  
  "Koju klasu treba napraviti?"
  
  Hajde redom
  
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

- **Controller**
  
  Ovde moram da napravim malu pauzu.

  Jer reč `controller` može da zavara.

  Ako dolaziš iz MVC sveta, očekuješ:

  - Model
  - View
  - Controller

  Ali ovde to nije to. Ovde je "controller" zapravo Python klasa koja predstavlja taj DocType.

  Na primer:

  ```txt
  Customer -> Customer(Document)
  ```

  ili

  ```txt
  User -> User(Document)
  ```

  Ako takva klasa postoji. Ako ne... videćemo šta Frappe radi.

  Pogledaj završetak funkcije.

  ```py  
  return controller(*args, **kwargs)
  ```

  Ovo je fantastično.

  Ne piše: `Customer(...)`. Ne piše: `User(...)`. Ne piše: `Task(...)`.
  Nego: `controller(...)`

  Drugim rečima:

  ```py
  get_doc() -> get_controller() -> ??? -> pozovi klasu
  ```

  To je čisti **Factory Pattern**.

</br>

- **Sada jedna veoma zanimljiva stvar**
  
  Sećaš se kada smo govorili:
  
  ```txt
  DocType -> Meta -> Document
  ```
  
  Sada možemo malo da dopunimo dijagram.
  
  ```txt
  frappe.get_doc() -> document.get_doc() -> get_controller() -> ??? -> Document(...)
  ```

  Odgovor na ono veliko ??? krije se upravo u `get_controller()`.
  
</br>

- **I evo pitanja koje sada možemo postaviti**
  
  Ako napišem: `frappe.get_doc("Customer")` kako `get_controller()` zna da vrati baš: "Customer" a ne: "User" ili: "Task".
  
  I još važnije... Šta ako "Customer.py" uopšte ne postoji?
  
  To je upravo ono po čemu je Frappe poseban.
  
</br>

- **Moj predlog**
  
  Mislim da je sledeći korak potpuno prirodan.  
  Nemojmo još otvarati Document.  
  Nemojmo još gledati SQL.  
  Pratimo ovu jednu liniju:

  ```py
  controller = get_controller(doctype)
  ```

  Pošto smo je već videli u `base_document.py`, sada ćemo konačno razumeti njenu ulogu.
  
  Pošalji:
  
  ```h
  grep -A 50 "^def get_controller" ~/frappe-bench/apps/frappe/frappe/model/base_document.py
  ```
  
  Nemoj se uplašiti ako bude malo duža. To je verovatno jedna od najvažnijih funkcija u celom mehanizmu Frappe-a, jer ona predstavlja most između naziva DocType-a (običnog stringa poput "Customer") i Python klase koja će na kraju biti instancirana.
  
---

Evo izlaza:

```sh
grep -A 50 "^def get_controller" ~/frappe-bench/apps/frappe/frappe/model/base_document.py

def get_controller(doctype):
  """
  Returns the locally cached **class** object of the given DocType.
  For `custom` type, returns `frappe.model.document.Document`.

  :param doctype: DocType name as string.
  """

  if frappe.local.dev_server or frappe.flags.in_migrate:
    return import_controller(doctype)

  site_controllers = frappe.controllers.setdefault(frappe.local.site, {})
  if doctype not in site_controllers:
    site_controllers[doctype] = import_controller(doctype)

  return site_controllers[doctype]


def import_controller(doctype):
```

---

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
  
  To znači da ova funkcija ne vraća Customer.
  Ona vraća:
  
  ```py
  Customer
  ```
  
  odnosno samu **klasu**.
  
  To je ogromna razlika.
  
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
  controller == Document
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
  
  Aha!
  
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

    Zašto?

    Jer u development-u menjaš kod. Ako bi klasa ostala keširana...  
    menjaš fajl... a Frappe i dalje koristi staru klasu.  
    To bi bilo veoma frustrirajuće.

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

  To ima potpuno smisla.

  Jer dva sajta mogu imati različite aplikacije.

</br>

- **Evo kako ja to zamišljam**

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

- **Hajde da zastanemo na trenutak**

  Voleo bih da obratiš pažnju na nešto.
  
  Pre desetak koraka imali smo dijagram:
  
  ```txt
  get_doc() -> Document
  ```
  
  Sada je on postao mnogo precizniji.
  
  ```txt
  get_doc() -> get_controller() -> import_controller() -> Python class -> Document instance
  ```
  
  Primećuješ?
  
  Naš mentalni model postaje sve detaljniji. Ne zato što učimo napamet funkcije.
  Nego zato što svaku "crnu kutiju" polako otvaramo.

</br>

- **Još jedna stvar koju želim da naglasim**

  Pogledaj kako je funkcija napisana. Nema 200 linija. Nema ogromnog `if`. Nema SQL-a. Radi samo jednu stvar:

  > pronađi odgovarajuću klasu i keširaj je.
  
  To je odličan primer principa **Single Responsibility Principle (SRP)** iz SOLID-a.
  
  Zato je lako razumeti šta radi i lako je testirati.
  
</br>

- **Moj predlog za nastavak**

  Mislim da smo sada došli do mesta gde treba otvoriti poslednju "crnu kutiju":
  
  ```python
  import_controller(doctype)
  ```
  
  Ali... Tu bih napravio malu promenu u našem načinu rada.
  
  Do sada smo pratili izvršavanje. Sada bih voleo da na trenutak stanemo i nacrtamo kompletnu arhitekturu onoga što smo do sada otkrili, pre nego što uđemo u `import_controller()`.
  
  Zašto?
  
  Zato što smo već sakupili dovoljno delova slagalice da možemo da vidimo celu sliku:
  
  ```txt
  Program -> frappe.get_doc(...) -> frappe.model.document.get_doc(...) -> get_controller(doctype) -> import_controller(doctype) -> Python klasa -> Instanca Document
  ```
  
  Mislim da je to dobar trenutak da zastanemo, proverimo da li nam je model jasan i tek onda otvorimo `import_controller()`, jer sumnjam da će baš tamo početi priča o tome:
  
  - kako Frappe pronalazi Python modul za dati DocType,
  - kako rade standardni i custom DocType-ovi i
  - kako se uključuju aplikacije (`apps`).
  
  To je već sledeći veliki deo arhitekture.
