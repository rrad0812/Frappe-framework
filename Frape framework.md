# Projekat: Učenje Frappe Framework-a

## Faze projekta

- **Faza 0 – Priprema okruženja**

  - Izabrati verziju Ubuntu-a (24.04 LTS)
  - Kreirati novu VM
  - Instalirati Ubuntu Server
  - Ažurirati sistem
  - Instalirati osnovne alate
  - Napraviti snapshot Clean System

  Cilj: "Stabilna razvojna mašina.
  
- **Faza 1 – Instalacija Frappe okruženja**

  - Python
  - Node.js
  - Redis
  - PostgreSQL
  - Bench
  - Kreirati prvi Bench

  Ovde nećemo još praviti nijedan DocType.
  Cilj je da razumemo: "Šta je Bench?"

- **Faza 2 – Anatomija Bencha**

  Ovde ne pišemo kod. Samo istražujemo.
  
  - Struktura direktorijuma
  - apps/
  - sites/
  - env/
  - logs/
  - config/
  
  Na kraju ove faze treba da možeš da pogledaš Bench i kažeš: "Znam čemu služi svaki direktorijum."
  
- **Faza 3 – Prvi Site**

  - napraviti Site
  - pokrenuti ga
  - otvoriti Desk
  - prijaviti se
  - pogledati šta je nastalo u PostgreSQL-u
  
  Ovde ćemo prvi put zaviriti u bazu.

- **Faza 4 – Anatomija Site-a**

  Opet bez programiranja.
  
  - site_config.json
  - private/
  - public/
  - assets/
  - logs/
  
  Na kraju: "Znam šta je Site."

- **Faza 5 – Prva aplikacija**

  Tek ovde,
  
  - new-app
  - instalacija aplikacije
  - modul
  - prvi DocType

- **Faza 6 – Desk**

  Ovo je deo koji si već pomenuo da ti je bio nejasan.
  
  Ovde ćemo odgovoriti na pitanja:
  
  - Kako Desk vidi moj DocType?
  - Kako se pojavljuje u Workspace-u?
  - Zašto ga nekad nema?
  - Kako ga organizovati?
  
  Mislim da će ova faza biti jedna od najzanimljivijih.

- **Faza 7 – DocType**

  Ovde konačno ulazimo u razvoj.
  
  - polja
  - validacija
  - child table
  - link
  - select
  - naming

- **Faza 8 – ORM**

  - Python.  
  - Ne JavaScript.  
  - Ne REST.  
  - Samo ORM.

- **Faza 9 – Hook-ovi**

  Šta se događa kada:
  
  - sačuvaš dokument
  - obrišeš dokument
  - submit
  - cancel
  
- **Faza 10 – JavaScript**

  - Client Script.
  - Form Script.
  - List Script.

- **Faza 11 – REST API**

  Ovde ćemo ga uporediti sa uAdmin-om.
  Mislim da će to biti veoma zanimljivo.

- **Faza 12 – Bezbednost**

  - Users
  - Roles
  - Permissions

- **Faza 13 – Deploy**

  Tek na kraju.
  
  - production
  - nginx
  - supervisor
  - backup

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

- **Jedna dopuna**

  Dodao bih i jednu "Fazu -1" koja se ne tiče instalacije nego razumevanja. Pre nego što instaliramo ijedan paket, voleo bih da odgovorimo na jedno pitanje:
  
  Zašto Frappe uopšte ima Bench?  
  Većina korisnika samo prihvati da postoji komanda bench, ali retko ko razume zašto nije jednostavno pip install frappe. Mislim da će nam upravo odgovor na to pitanje dati dobar uvod u arhitekturu frameworka i učiniti sve naredne korake logičnijim. Po mom mišljenju, to je pravo mesto odakle treba da počnemo.

## Faza 0 - Priprema okruženja

- **Preuzimnje ISO slike i instalacija Ubuntu Server 24.04**

  Sa adrese <https://releases.ubuntu.com/24.04.4/ubuntu-24.04.4-live-server-amd64.iso> preuzeti ISO za `Ubuntu server 24.04`.
  
  </br>

- **Izgradnja Ubuntu Server 24.04 VM**:  

  VM izgradnja na QUEMU/KVM root sesiji sa sledećim parametrima:
  
  - vCPU 2
  - RAM 8GB
  - SSD 60GB
  
  </br>
- **Ažuriranje paketa distibucije**:

  ```sh
  sudo apt update && sudo apt upgrade -y
  ```
  
  </br>
- **Instalacija curl-a**

  ```sh
  sudo apt install curl
  ```
  
  </br>
- **Aktiviranje firewall-a i dozvola ssh pristupa**:

  ```sh
  sudo ufw enable
  ```
  
  Dozvola pristupa preko ssh:

  ```sh
  sudo ufw allow OpenSSH
  ```

  Proba konekcije na VM ( samo sa localhost-a )  
  Iz virtuelne mašine pokrenuti:

  ```sh
  ip a
  ```

  Vratiće, za KVM slučaj, nešto kao 192.168.122.X.

  Sa hosta ssh pristup na VM kao:

  ```sh
  ssh username_na_VM@127.198.122.X
  ```
  
  Reset VM:

  ```sh
  sudo shutdown -r now
  ```

  </br>
- **Promena vremenske zone i sync. vremena**

  ```sh
  sudo timedatectl set-timezone Europe/Belgrade
  sudo timedatectl set-ntp on
  ```

- **Promena locale**:

  ```sh
  sudo dpkg-reconfigure locales
  ```
  
  Dodaj nove locale `sr_RS@latin UTF-8` i postavi ih za default.

  Reset VM:

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

  Osveži sesiju:
  
  ```sh
  source "$HOME/.local/bin/env"
  ```
  
  Proveri instaliranost:
  
  ```sh
  uv --version
  ```

- **Šta je urađeno u Fazi 0**

  - Instaliran Ubuntu Server 24.04 sa OpenSSH serverom.
  - Instalacija ažurirana na nove verzije paketa
  - Promenjen status UFW na enable. Promenjen status OpenSSH na enable. Postignut
    ssh pristup sa localhosta na VM.
  - Promenjena vremenska zona na Europe/Belgrade, uradjen sync. vremena
  - Promenjen locale na sr_RS@latin UTF-8 i postavljen za podrazumevani.
  - Instaliran uv Pajton paket i runtime manger.

## Faza 1 - Instalacija Frappe okruženja

- **Instalacija git**:

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

  Provera instalirane verzije

  ```sh
  psql --version
  ```

  Prelazak na `postgres` nalog**  
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

  Na psql možete doći kao `postgres` bez prelaska sa svog naloga:

  ```sh
  sudo -u postgres psql
  ```

  Dodela passworda postgres korisniku:

  ```sql
  ALTER USER postgres WITH PASSWORD 'postgres_password';
  ```

  </br>
- **Instalacija wkhtmltopdf**
  
  ```sh
  sudo apt install xvfb libfontconfig
  ```

  Preuzmi wkhtmltopdf paket sa <https://wkhtmltopdf.org/downloads.html>, potom pokreni sledeću komandu za instalaciju:
  
  ```sh
  sudo dpkg -i wkhtmltox_file.deb
  ```

  </br>
- **Instalacija frontend zavisnosti**

  - Instalacija nvm

    ```sh
    curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash
    ```

  - Instalacija nodejs

    ```sh
    nvm install 24
    ```

  - Provera instaliranosti

    ```sh
    node -v
    ```

  - Instalacija yarn

    ```sh
    npm install -g yarn
    ```

  </br>
- **Instalacija Bench-a (Frappe v15)**  

  ```sh
  uv tool install frappe-bench --with setuptools
  ```

  **Napomena**:  
  Dodali smo `--with setuptools` jer Frappe u pozadini još uvek koristi neke starije Python mehanizme za pakete.
  
  Sada proveri da li sistem vidi komandu:
  
  ```sh
  bench --version
  ```

  </br>
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

- **Šta je urađeno u Fazi 1**

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
  
  - MySQL root password  
  Iako piše MySQL, pošto si stavio `--db-type postgres`, ovde unosiš lozinku za `postgres` korisnika koju si postavio u Fazi 1.
  
  - Administrator password  
  Ovo je lozinka za glavnog admina samog Frappe web interfejsa. Postavi neku lozinku po želji (npr. admin123 za lokalni rad) i zapamti je.

  - Kako da proveriš da li je uspelo?  
    Kada komanda završi (ovaj put uspešno), proveri da li baza zaista postoji u PostgreSQL-u:

    ```sh
    sudo -u postgres psql -c "\l"
    ```

    Trebalo bi da vidiš novu bazu sa čudnim, nasumičnim imenom koje počinje sa podvlakom (npr. _1a2b3c4d5e...), jer Frappe namerno generiše hešovana imena baza radi bezbednosti.

  </br>
- **Rešenje problema sa konekcijom na PostgreSQL**  
  Moramo da dozvolimo svim korisnicima (`all`) da se povežu na sve baze (`all`) preko 127.0.0.1 (localhost-a) interfejsa koristeći lozinku.

  Podrazumevano PostgeSQL je podešen na `peer` konkcije (`socket`) sa localhosta, i to je način na koji pristupa psql. Za Frappe pristup, preko mrežnog interfejsa potrebno je prepodesiti u '/etc/postgresql/16/main/pg_hba.conf' konfiguracionm fajlu IPv4 pravilo, tako da glasi:
  
  ```txt
  # IPv4 local connections:
  host    all             all             127.0.0.1/32            md5
  ```
  
  Promenili smi i tip protkola lozinke, jer se u praksi pokazalo da radi!

  </br>
- **Restart PostgreSQL servisa**  
  Posle promene konfig. fajllova, restartuj PostgreSQL servis:
  
  ```sh
  sudo systemctl restart postgresql
  ```
  
  </br>
- **Šta je urađeno u Fazi 2**

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

---

Odlično. Ovo je upravo ono što sam želeo da vidim.

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

  ```bash
  uv venv
  ```

  samo što Bench to radi sam.

</br>

- **sites/**

  Ovo je direktorijum koji početnicima pravi najveću zabunu. Ovde **nije izvorni kod**. Ovde su **podaci i konfiguracija sajtova**. Na primer:
  
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

  Drugim rečima, **aplikacija ≠ sajt.**
  To je veoma važna razlika.
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
  Ako si radio sa **Heroku** ili **Foreman**, koncept će ti biti poznat.

</br>

- **patches.txt**

  Ovo nije nešto što ćeš često dirati. Koristi se tokom migracija i nadogradnji kako bi Bench znao koje su zakrpe (patches) već primenjene.

</br>

- **Ono što mi se posebno sviđa**  
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
- i napravićemo jasnu mentalnu sliku odnosa **Bench → Apps → Sites**.

Mislim da će posle toga cela arhitektura Frappe-a početi da "leže" na svoje mesto.

---

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

Odlično. Ovo nam već govori dosta.

Sada ćemo napraviti jednu važnu mentalnu mapu. Ako je razumeš, mnogo ćeš lakše razumeti sve ostalo u Frappe-u.

```text
               Bench
                 │
       ┌─────────┴─────────┐
       │                   │
    Applications        Sites
    (izvorni kod)    (podaci i konfiguracija)
```

</br>

- **apps/**

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

- **sites/**

  Ovde se već nalazi mnogo zanimljivijih stvari.
  
  - **common_site_config.json**
  
    Ovo je **globalna konfiguracija Bench-a**. Ona važi za sve sajtove. Na primer:

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
  
</br>

- **assets/**

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

- **apps.txt**

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

- **apps.json**

  Ovo je noviji mehanizam koji Bench koristi za dodatne informacije o aplikacijama. U praksi ga retko menjaš ručno; Bench ga održava.
  
</br>

- **Ono što treba da zapamtiš**  
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

- **Predlog za sledeći korak**  

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

Odlično pitanje.

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

</br>

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

</br>

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
  
</br>

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

  Ime govori dosta.   Ovde završavaju stvari koje nisu javno dostupne.  
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

- **Zanimljivost**
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
  - Jedna mala napomena  
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
  
  - **Zašto baza nema ime `site1.local`?**  
    Verovatno si očekivao nešto poput: "site1.local", ili "site1_local". Međutim, Frappe radi drugačije.  On generiše ime baze: "_c9eb2d89e08e8728".

  - **Gde je korisničko ime?**  
    Primeti nešto zanimljivo. Ovde nema: "db_user": "...". Zašto? Zato što kod PostgreSQL-a Frappe koristi isto ime za bazu i korisnika.

    Drugim rečima: db_name: "_c9eb2d89e08e8728" i "role": "_c9eb2d89e08e8728"
    imaju isto ime.

  - **Lozinka**
    Ovu lozinku nisi ti birao. Bench ju je napravio automatski. To je veoma dobra praksa. Svaki sajt dobija:

    - svog korisnika
    - svoju bazu
    - svoju nasumičnu lozinku
  
  - **db_type**
    Ovde piše "db_type":"postgres". To znači da ostatak Frappe-a zna koji backend koristi. Da si radio sa MariaDB, ovde bi bilo drugačije.
  
  - **Ali sada dolazi pitanje...**
    Ako ovde nema: db_host i db_port definicije kako Frappe zna da koristi:
    127.0.0.1, 5432?

    Odgovor je: Ne zna iz ovog fajla. Te informacije dolaze iz drugih delova konfiguracije (globalnih podešavanja Bench-a i podrazumevanih vrednosti).

    To znači da jedan `site_config.json` sadrži samo ono što je specifično za taj sajt.

    To mi se veoma sviđa kao dizajn.
  
  - **Jedna mala digresija**
  
    Sećaš se koliko smo juče jurili grešku?
  
    ```sh
    password authentication failed
      ```

    pa zatim

    ```sh
    no pg_hba.conf entry
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
  
- **Sledeći korak**
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
  