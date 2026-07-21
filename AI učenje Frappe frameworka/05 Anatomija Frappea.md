
# Učenje Frappe Frameworka

[Sadržaj][00]

## 05 Anatomija Frappea

### `frappe-bench` direktorijum

> [!Note] AI Zahtev
>
> Uradi u terminalu:
>
> ```bash
> cd /home/radosav/frappe-bench
> ls -lah
> ```
>
> i pošalji izlaz.

Prva zanimljiva stvar je da je ovo mnogo "čistije" nego što ljudi obično očekuju. Mnogi misle da je `frappe-bench` neka ogromna struktura, a zapravo je vrlo pregledna.

> [!Info] Izlaz
>
> ```sh
> frappe-bench/
> ├── apps/
> ├── config/
> ├── env/
> ├── logs/
> ├── sites/
> ├── Procfile
> └── patches.txt
> ```

Hajde da prođemo kroz ovo, ali ne samo da nabrojimo direktorijume, već da razumemo njihovu ulogu.

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
  
  Drugim rečima,
  
  > [!Info]
  > **Aplikacija ≠ Sajt**.  
  
  </br>

  > [!Info]
  > **Jedna aplikacija može biti instalirana na više sajtova.**
  
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

> [!Info]
>
> ```sh
> Code -> apps/
> Data -> sites/
> Enviroment -> env/
> ```

To je veoma elegantna organizacija.

#### apps/ i sites/ direktorijumi

> [!Note] AI zahtev
>
> Hajde da sada "zavirimo" u dva direktorijuma koja su ključna za razumevanje
> Frappe-a:
>
> Prvo:
>
> ```bash
> ls -alh apps
> ```
>
> a zatim:
>
> ```bash
> ls -alh sites
> ```

Iz toga ćemo odmah videti:

- koje su aplikacije trenutno instalirane,
- kako Bench razlikuje globalnu konfiguraciju od konfiguracije pojedinačnog sajta,
- i napravićemo jasnu mentalnu sliku odnosa Bench → Apps → Sites.

Mislim da će posle toga cela arhitektura Frappe-a početi da leže na svoje mesto.

Sada ćemo napraviti jednu važnu mentalnu mapu. Ako je razumeš, mnogo ćeš lakše razumeti sve ostalo u Frappe-u.

> [!Info] Vizuelna slika najvažnijeg dela organizacije Frappe
>
> ```text
>                Bench
>                  │
>        ┌─────────┴─────────┐
>        │                   │
>     Applications         Sites
>     (izvorni kod)   (podaci i konfiguracija)
> ```

</br>

Idemo redom:

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

  Primeti jednu stvar: `apps/` ne zna ništa o `site1.local` sajtu.
  Tu nema:
  - konfiguracije sajta,
  - nema baze,
  - nema korisnika,
  - nema podataka.
  
  Samo kod. To je veoma lepo odvajanje odgovornosti.

</br>

- **sites/**  
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
  
  To je isti koncept koji postoji u mnogim frameworcima:
  
  **globalna podešavanja + lokalna podešavanja.**
  
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

#### Jedna aplikacija - više sajtova

Po mom mišljenju, ovo je najvažnija slika do sada:

> [!Info]
>
> ```sh
> apps/ -> Izvorni kod
> sites/ -> Podaci o sajtovima
> ```

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

Pre nego što pokrenemo razvojni server, voleo bih da pogledamo jedan konkretan sajt. To će nam dati jasnu sliku šta Frappe smatra "sajtom".

> [!Note] AI Zahtev
> Možeš da pošalješ izlaz:
>
> ```sh
> tree -L 2 sites/site1.local
> ```
>
> Ako nemaš `tree` instaliran:
>
> ```sh
> sudo apt install tree
> ```
>
> ili, bez instalacije:
>
> ```bash
> find sites/site1.local -maxdepth 2 | sort
> ```

Posle toga ćemo analizirati `site_config.json`, jer je to jedan od najvažnijih fajlova u celom Frappe ekosistemu. Tu ćeš prvi put videti kako se jedan sajt povezuje sa PostgreSQL bazom i ostatkom sistema.

> [!Note] Moje pitanje
> Pre nego pređemo na sledeći korak može pitanje: "Kažeš da jedna aplikacija može
> da bude na više sajtova. Da li jedan sajt može da ima više aplikacija?

</br>

> [!Info] AI odgovor
> Odgovor je: Da. Zapravo, to je jedan od osnovnih principa Frappe-a.
>
> U stvari, odnos je: **više aplikacija ↔ više sajtova**.  
> Možeš to posmatrati kao matricu.
>
> | Aplikacija | site1.local | firmaA.local | firmaB.local |
> | ---------- | :---------: | :----------: | :----------: |
> | frappe | ✅ | ✅ | ✅ |
> | erpnext | ✅ | ✅ | ❌ |
> | crm | ❌ | ✅ | ✅ |
> | warehouse | ✅ | ❌ | ✅ |
>
> Dakle:
>
> - jedan sajt može imati više aplikacija,
> - jedna aplikacija može biti instalirana na više sajtova.

**Frappe je uvek prva instalirana aplikacija na sajtu**  
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

Za svaki sajt postoji informacija koje su aplikacije na njemu instalirane.

To možeš čak odmah da proveriš:

```sh
bench --site site1.local list-apps
```

Pošto si napravio potpuno nov sajt, očekujem da će rezultat biti:

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

</br>

**Jedna mala terminološka napomena**  
Ovde je lako pomešati pojmove jer reč aplikacija u Frappe-u ne znači isto što i u mnogim drugim okvirima.

- **Frappe app** je modul ili paket funkcionalnosti (npr. ERPNext, CRM ili tvoja aplikacija).
- **Site** je instanca koja ima svoju bazu podataka, konfiguraciju i korisnike.

To znači da jedan site može biti sastavljen od više Frappe aplikacija, a ista aplikacija može biti korišćena na više različitih sajtova. Upravo ta fleksibilnost je jedan od razloga zašto je Frappe pogodan za razvoj većih poslovnih sistema i SaaS rešenja.

#### sites/site1.local/ direktorijum

Ovde ćemo videti sadržaj `sites/site1.local` direktorijuma.

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

> [!Note] AI Zahtev
>
> Predložio bih da pogledamo sadržaj:
>
> ```bash
> cat sites/site1.local/site_config.json
> ```

To je možda svega desetak redova, ali iz njega ćemo moći da objasnimo:

- kako Frappe pronalazi PostgreSQL bazu,
- kako zna kojim Redis instancama da se poveže,
- kako razlikuje jedan sajt od drugog,
- koje su postavke globalne (`common_site_config.json`), a
  koje lokalne (`site_config.json`).
  
</br>

**Zanimljivost**  
Ako pogledaš direktorijum `site1.local/` kao celinu on uopšte ne izgleda kao aplikacija. Više liči na profil jednog korisnika sistema.

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
  
> [!Info] Izlaz
>
> ```sh
> cat sites/site1.local/site_config.json
> ```
>
> ```json
> {
>  "db_name": "_c9eb2d89e08e8728",
>  "db_password": "Oq69qLS1RcrKcUxg",
>  "db_type": "postgres"
> }
> ```
  
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

**Kako Frappe pristupa PostgreSQL?**  

Ako ovde nema `db_host` i `db_port` definicije kako Frappe zna da koristi:  127.0.0.1, 5432?

Odgovor je: Ne zna iz ovog fajla. Te informacije dolaze iz drugih delova konfiguracije (globalnih podešavanja Bench-a i podrazumevanih vrednosti).

To znači da jedan `site_config.json` sadrži samo ono što je specifično za taj sajt.

[Sadržaj][00]

[00]: 00%20Učenje%20Frape%20frameworka.md
