# BIG01: Juna-alusta

Tämä harjoitus poikkeaa muista harjoituksista siten, että alustan toteutukseen löytyy YouTube-soittolista avuksi. Soittolistan videoissa kasataan lähes identtinen alusta siitä, mitä sinä kasaat. Erona on eri data, mikä vaikuttaa pienissä määrin tiedon lataukseen, tiedon mallintamiseen ja visualisointiin.

* Videot: [YouTube Playlist: Data-alustat BIG101 demo](https://youtube.com/playlist?list=PL7AbISYtmmfgzyf95jbLMWoVp5DqzEJSl)
* Esimerkkitoteutuksen data: [Jäätelöauto API](https://gitlab.dclabra.fi/jani-public/jaateloauto)
* Esimerkkitoteutuksen koodi: [data-alustat-demo](https://gitlab.dclabra.fi/jani-public/data-alustat-demo/)

Soittolistan videoilla kasataan alusta, joka:

* Tuo feikkidataa jaateloauto-API:sta
* Edustaa [Data-alustat/Arkkitehtuuri](https://sourander.github.io/data-alustat/aloituspaketti/arkkitehtuurit/) mukaista yhden koneen ja yhden tietolähteen arkkitehtuuria
* Mallintaa datan Kimballin tähtimallia muotoilevaksi Silver-kerrokseksi
* Mallinta datan one big table -tyyppiseksi aggregaattitauluksi Gold-kerrokselle
* Tarjoaa loppukäyttäjille Evidence BI-työkalun, jolla valittu bisnesongelma visualisoidaan

Sinun tehtäväsi on luoda mahdollisimman automatisoitu putki, joka hyödyntää dbt:tä, DuckDB:tä ja Evidenceä työkaluina, ja DuckDB-tietovarastoon tehdään medaljonkiarkkitehtuuri (bronze, silver, gold).

!!! warning "Muokkaa monorepoksi"

    Vuoden 2025 toteutuksessa tämä oli kurssin loppupuoliskon käytännön toteutus, ja täten se eli yksin omassa repositoriossaan. Vuodesta 2026 alkaen toteutuksessa on käytetty monorepoa. Älä siis toista laput silmillä opettajan komentoja vanhasta videosta: tee samat asiat nykyisen `etunimisukunimi/` repositoriosi sisään, esimerkiksi hakemistoon `etunimisukunimi/big01/`.

    Voi olla kannattavaa avata tuo hakemisto omaan VS Code -ikkunaan. Osa VS Coden ominaisuuksista, erityisesti Python virtuaaliympäristöön liittyvät, toimivat parhaiten kun ne ovat VS Coden näkökulmasta projektin juurihakemistossa.

## Videon vaiheet

Soittolistan videoissa näkyy TODO-lista, jota opettaja seuraa. Alla on sama lista tarjottuna sinulle, siltä varalta, että se helpottaa tehtävän tekemistä tai videoiden seuraamista. Jos siitä ei ole apua, hyppää yli.

Demossa valittu bisnesongelma on: Jäätelöautojen viikoittainen kumulatiivinen sekä keskimääräinen (p50 ja p90) pysäkiltä myöhässä lähteminen (eli *"departure lateness"*).

### Luento 1: Jaateloauto REST API to staging

#### Part 1/3: Projektin aloitus

- [x] Valitse bisnesongelma
    - [x] Tutustu REST API:n dokumentaatioon ja varmista että valitsemasi ongelmaa vastaava data on olemassa. Jos ei, vaihda bisnesongelma.
- [x] Alusta repositorio
- [x] Asenna uv
    - [x] Asenna Python 3.12 (`uv python install 3.12`)
    - [x] Pinnaa versio (`uv python pin 3.12`)
    - [x] Alusta uv-projekti (`uv init --lib --name "ope-data-alusta" .`) 
        - [x] `--lib` selitetään myöhemmin hakemistorakenteessa.
    - [x] Luo virtuaaliympäristö (`uv sync`)
    - [x] ... ja varmista, että VS Code käyttää sitä Python Interpreterinä.
- [x] Päätä projektihakemiston rakenne, esim:
    - [x] Data menee aina `./data/`
        - [x] Raakadata louhitaan `./data/lake/*/*`
        - [x] Tietovarasto sijoitetaan `./data/warehouse/jotain.db`
    - [x] Koodi sijoitetaan `./src/` (uv huolehti jo tästä)
    - [x] Automaatioskriptit sijoitetaan `./scripts/`
    - [x] Tee tyhjistä hakemistoista pysyviä `.gitkeep`-tiedostojen avulla. 
    - [x] Puske GitLabiin (muista `git status -u` !!!)

**HOX!** Oikeaa tehtävää tehdessä tässä välissä olisi hyvä pitää dokumentaatio kunnossa. Onhan sinulla README.md-tiedosto vähintään jo otsikkotasolla päivitetty?

#### Part 2/3: Jaateloauto REST API

**HOX!** Tähän väliin käytännön vinkki: teille on täysin sallittua sijoittaa repositorioon `MEMO.md` tai `./notes/*.md` tai ylläpitää muistiinpanoja Notionissa tai käyttää fyysistä muistivihkoa. Minä en sitä tee videoilla, mutta opiskellesssa suosittelen tekemään muistiinpanoja. Ethän aja itseäsi tilanteeseen, jossa *"Ajoin komennon, jota en muista, ja sit tuli virhe, jota en muista, mutta nyt tämä ei enää toimi."* Tätä kannattaa käyttää myös työelämässä.

**HOX!** Tee muistiinpanoja myös kurssipalautteeseen liittyvistä asioita. Näin osaat antaa parempaa palautetta kurssin lopuksi, kun ongelmakohdat eivät ole muistin varassa. Voit käytännössä copy-pasteta palautteen intran kaavakkeeseen. Kukaan meistä ei ole täydellinen, mutta me kaikki voimme kehittyä. Palaute auttaa tässä.

- [x] Kloonaa [Jäätelöauto API](https://gitlab.dclabra.fi/jani-public/jaateloauto/)
    - [x] Lue README.md ja aja palvelu ylös.
    - [x] Tutustu Swagger UI:iin (`localhost:8000`)
    - [x] Tutustu [Rautatieliikenne](https://www.digitraffic.fi/rautatieliikenne/) API:iin
        - [x] Swagger
        - [x] (Optional) GraphQL
        - [x] Tsekkaa myös Keskusteluryhmä. Näet aitoa bugi-ilmoittelua!
- [x] Tutustu REST:n olemukseen [JSONPlaceholder](https://jsonplaceholder.typicode.com/) avulla.
    - [x] Selain
    - [x] HTTPie
    - [x] Testaa samat vielä Jäätelöauto API:n datalla
    - [x] Tsekkaa vielä [HTTP verbit](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Methods)
- [x] Python Notebook Playground
    - [x] Asenna IPykernel ja requests (`uv add ipykernel requests`)
    - [x] Tee hakemisto Notebookeille (esim. `./notebooks/`)
    - [x] Luo uusi Notebook, `rest_tutuksi.ipynb`
    - [x] Kokeile `requests`-kirjastoa (`requests.get('url').json()`
- [x] Bonus: kurkataan huvikseen myös [Vanhat junat zip-paketteina](https://www.digitraffic.fi/rautatieliikenne/#vanhat-junat-zip-paketteina) -vaihtoehtoa. Tämä voi olla houkutteleva, mikäli jonkun bisnesongelma käsittelee pidemmän ajanjakson ongelmaa useampaan junaan liittyen. Keksitkö tavan automatisoida kaikkien näiden S3:lla olevien zip-tiedostojen lataamisen (esimerkiksi `./data/lake/staging/vr-zip/yyyy-mm-dd.zip`) ja purkamisen (`./data/lake/staging/vr-data/yyyy/mm/dd/???.zip`).
- [x] Puske GitLabiin

**HOX!** Varsinaiset skriptit voit kehittää `.ipynb`-Notebookissa, jos kaipaat interaktiivisuutta kehittäessä, mutta luo niistä kuitenkin lopulta `.py`-skriptit, jotta niiden parametrisointi ja automatisointi on helppoa myöhemmin.

**HOX!** Get-metodin palauttaman objektin metodi `.json()` on kiva plärätessä, mutta jos haluat Bronzen todella olevan **as-raw-as-possible**, kirjoita palautuneet bytet sellaisina kuin ne ovat saapuneet ilman turhia enkoodauksia, joita ko. metodi väkisinkin ujuttaa mukaan.

**HOX!** Huomaa, että sinun tulee aina lukea käyttämäsi API:n käyttöehdot ja -ohjeet. Esimerkiksi Fintrafficilla on oma [Tuki > Ohjeita ja lisätietoa rajapintojen käyttöön](https://www.digitraffic.fi/tuki/ohjeita/) -ohje, jossa neuvotaan pakolliset headerit (eli `'Digitraffic-User: Junamies/FoobarApp 1.0'`) sekä kuvaukset rajoituksista, jotka rajoittavat kyselyiden tiehyttä (default: 60 kpl/min).

#### Part 3/3: Ingestion Tool

- [x] Koodaa dummy ingestion tool.
    - [x] Jupyter Notebook (`./notebooks/jaateloauto_ingestion.ipynb`)
    - [x] Hetken miete datan/skeeman tarkistukselle (e.g. Polars schema)
    - [x] Hetken miete JSON koskemattomana vai esim. JSON Lines (`ELT` vs `EtLT`)
    - [x] Hetken miete, saako tässä välissä tiputtaa tarpeettomia kenttiä pois
- [x] (Advanced:) Koodaa helpommin ajettava ingestion tool
    - [x] Python (`./src/ingestor/jaateloauto.py`) ja `argparse`
    - [x] Immutability!
- [x] Laita `data/lake/staging/` gitignoreen. Data ei kuulu versionhallintaan.
- [x] Puske GitLabiin.

📅 Tähän pisteeseen teidän pitäisi päästä ensimmäisellä viikolla! Kirjoittakaa jäätelöautodatan kylkeen skriptit, jotka louhivat kyseisen dummydatan sijasta aitoa raideliikennedataa.

### Luento 2: Staging to Bronze using dbt

#### Part 1/3: dbt ja DuckDB

- [x] Asenna dbt 
    - [x] Aloita asentamalla [DuckDB CLI](https://duckdb.org/docs/installation/). (⚠️ Huomaa mahdollinen vaatimus C++ kirjastojen asentamiselle Windowsissa! ⚠️).
    - [x] Tämän jälkeen asenna dbt-duckdb (`uv add dbt-duckdb`), jonka riippuvuuksina asentuvat myös `dbt-core` ja `duckdb`.
- [x] Luo dbt-projekti
    - [x] Päätä: voit luoda joko jäätelöautolle ja junadatalle yhteisen tai kummallekin erikseen.
    - [x] `uv run dbt init`
    - [x] `cd <projektin_nimi>`
    - [x] Luo lokaali `profiles.yml` (katso malli alta)
- [x] Tutustu seuraaviin tiedostoihin:
    - [x] `profiles.yml` (configuration precedence: Project, User, System)
    - [x] `dbt_project.yml`
    - [x] `models/**/*.yml`
    - [x] `models/**/*.sql`
        - [x] Ja niissä `{{ jotain }}` -syntaksi



Luo lokaali `profiles.yml`:

```yaml
dbt_warehouse:
  target: dev
  outputs:
    dev:
      type: duckdb
      path: '../data/warehouse/warehouse.duckdb'
      schema: main
```



- [x] Aja ensimmäinen malli
    - [x] `uv run dbt run`
    - [x] `uv run dbt test` <= 🚧 Noteeraa virhe 🚧
    - [x] Tutki tiedostoja, etsi virhe, korjaa se.
- [x] Tutustu DuckDB UI:in
    - [x] Aja `duckdb -ui polku/sinun/warehouseen.duckdb`. Selaimeen aukeaa etäisesti Jupyter Notebookin oloinen SQL-client.
    - [x] Huomaa, että se luo Notebookit ja muun datan kotikansioosi `~/.duckdb/extension_data/ui/`
- [x] Tutustu docsiin
    - [x] `uv run dbt docs generate`
    - [x] `uv run dbt docs serve `
- [x] Puske GitLabiin



#### Part 2/3: dbt ja Fake CSV

- [x] Luo tiedostot
    - [x] `data/lake/staging/csvtest/customers.csv` (id, name, email)
    - [x] `.../orders.csv` (id, customer_id, product, quantity)
- [x] Luo datamalli Bronzelle
    - [x] Hakemisto `<projektin_nimi>/models/bronze/csvtest`
    - [x] Tiedostot `csvtest_customers.sql` ja `csvtest_orders.sql`
        - [x] **HOX**! Harjoittele SQL:ää DuckDB UI:ssa!
        - [x] Tutustu [DuckDB:n CSV Importtiin](https://duckdb.org/docs/stable/data/csv/overview.html)
    - [x] Lisää `dbt_project.yml` tiedostoon model
        - [x] materialized: table
        - [x] schema: bronze (muokkaa profilesiin `target: dev`, jotta DuckDB schemaksi tulee `dev_bronze`)
    - [x] Kirjoita myös `schema.yml` (tai `whatever.yml`)
        - [x] (Optional:) Käytä `docs.md` tiedostoa ja tuo se näin: `'{{ doc("customers_desc") }}'`
- [x] Puske GitLabiin



#### Part 3/3: dbt Jaatelo Bronzelle

- [x] Tarkastele tiedostoja
    - [x] `data/lake/staging/jaatelo/<junanro>/<date>.json`
    - [x] Tiedosta, että REST API saa olla nyt alhaalla.
- [x] Luo datamalli Bronzelle
    - [x] Hakemisto `models/bronze/jaatelo`
    - [x] Tiedosto `jaateloauto_api.sql`
        - [x] Suunnittele, älä hutki!
        - [x] Luo primääriavain `sk_jaateloauto` (truckId + departureDate + operatorId)
        - [x] ... käytä myös concat_ws, coalesce tai ifnull ja md5 hash
        - [x] ... ja dbt:ssä käytä [dbt utils](https://hub.getdbt.com/dbt-labs/dbt_utils/latest/) (`packages.yml` ja `uv run dbt deps`)
    - [x] Lisää `dbt_project.yml` tiedostoon modelseihin
    - [x] Dokumentoi `schema.yml`
- [x] Puske GitLabiin

### Luento 3: Silver and Gold modelling

#### Part 1/3: Silver Timetable

- [x] Luo datamalli Silverille

    - [x] **Suunnittele ennen kuin teet!** Sinun pitää tietää, mikä yksi rivi yhdessä taulussa edustaa!

    - [x] Tästä alkaen työ on mekaanista toistoa yllä olevasta, mutta kaikki tehdään `models/silver` ja `gold` kansioon

    - [x] SQL kannattaa taas harjoitella interaktiivisessa DuckDB UI -tilassa ja myöhemmin liittää `.sql`-tiedostoon.

    - [x] Luotavat 🥈 Silver taulut:

        - [x] `jaateloauto_timetable` : one row per an event (arrive, departure) that has ever happened. In other words, this is the unnested JSON data from Bronze. Primary key needs to be a **surrogate key**.
            - [x] `timetable_row -> '$.station' as method_a` ([JSON Functions](https://duckdb.org/docs/stable/data/json/json_functions.html)) (json_extract) sopii kaikille paitsi stringeille.
            - [x] `timetable_row --> '$.stopName' as method_b` (json_extract_string) parsii pois hipsukat stringeistä.
            - [x] Yhdistä castin kanssa: `json_extract(ttr, '$stopId')::INT`
            - [x] Jos kenttä on yhä nestattu: `json_extract(ttr, '$.latenessCauses')::JSON[]`
            - [x] PK/SK: `sk_jaateloauto_timetable` : (sk_jaateloauto + stop_id + stop_direction)
            - [x] Lisää modeliksi ja dokumentoi taulu.

    - [x] Puske GitLabiin


#### Part 2/3: Silver Fact and Dim

Luotavat 🥈 Silver taulut hakevat kumpikin tiedot yllä tehdystä aputaulusta.

- [x] `f_stop` : A fact table representing one row per stop - including arrival and departure. The ice cream trucks stops to sell ice cream. It will include columns for arrives lateness and depature lateness. In short, the data is nearly the same as in the one above, but granularity has been reduced to stop (instead of stop-event). The moment of a truck stopping to sell ice cream has been chosen as a business event that is tracked. Primary key needs to be a **surrogate key**, since we don't get a unifying UUID from source system of other suitable id.
    - [x] PK/SK: `sk_stop` (sk_jaateloauto + stop_id) using 

- [x] `d_truck` : One row per truck. In real life, we would most likely enrich this data with other information such as route length, amount of stops on the truck's route et cetera. Primary key is **truck_id**, which is essentially a natural key, since customer's would see this truck id in the time schedule.
    - [x] PK: `truck_id`
    - [x] Huomaa, että koska jäätelöauton perustiedot pysyvät päivästä toiseen samana, voimme noutaa kaiken tiedon kyseisen trukin uusimmasta rivistä, ja voimme myös pitää esimerkiksi pelkästään departuren.
- [x] Puske GitLabiin


#### Part 3/3: Gold

Luotavat 🥇 Gold taulut:

- [x] `jaateloauto_weekly_lateness` : one row per `week + truck_id + operator_name` combo. We will focus on departure times. Aggregated fields field be e.g. `total_departure_lateness`,  `p50_departure_lateness` (alias median) and `p90_departure_lateness`.
    - [x] Tsekkaa [Aggregate Functions: quantile cont](https://duckdb.org/docs/stable/sql/functions/aggregates.html)
- [x] Tarkista, että dbt docsissa lineage näkyy oikein.
- [x] Puske GitLabiin


### Luento 4: Evidence

!!! info

    Jos haluat, voit korvata Evidencen jollakin toisella BI-työkalulla. Vaihtoehtoja ovat esimerkiksi:

    * Marimo Notebook + Altair
    * Streamlit + Matplotlib/Plotly/Altair/etc.
    * Power BI tai Tableau
    * JavaScript + D3.js

    Evidencen setup on aiheuttanut joillakin Windows-käyttäjillä päänvaivaa, joten sallin myös vaihtoehtoiset ratkaisut. BI-kerros ei ole tämän harjoituksen pääpointti, mutta se pitää kuitenkin olla olemassa, jotta Gold-tason tauluille on jokin merkitys olemassa.

#### Part 1/2

- [x] Tutustu [Evidencen dokumentaatioon](https://docs.evidence.dev/)
    - [x] Ja [gh:evidence-dev/docker-devenv](https://github.com/evidence-dev/docker-devenv/tree/main) esimerkkiin
    - [x] Ja minun vastaaviin Docker Compose ja Dockerfile tiedostoihin
- [x] Aja ensin init
    - [x] `docker compose -f docker-compose.init.yml up --build`
    - [x] `docker compose -f docker-compose.init.yml down`
    - [x] ... jotta saat `bi/workspace` kansion.
- [x] Lisää GitLabiin


Tässä välissä, jos haluat, voit tutkia mitä uusi volume sisältää.

```bash
# Huomaat, että sinulle on uusi volume evidence_node_modules
docker volume ls

# Luo tilapäinen kontti, johon volume on mountattu
docker run --rm -it \
  -v evidence_node_modules:/mnt \
  ubuntu bash
  
# Kontin sisällä voit ajaa seuraavat komennot.
# Ctrl + D sulkee ja tuhoaa kontin (mutta ei volumea)
cd /mnt
ls
```

- [x] Aja sitten palvelu ylös
    - [x] Tarpeen mukaan `docker compose build`
    - [x] `docker compose up --watch`
    - [x] Kokeile muokata index.md:tä. Sivun pitäisi päivittyä.
- [x] Puske GitLabiin

Tässä välissä vaihdan **Windowsiin**, jotta homma olisi hieman cross OS -testattua:

- [x] Aja ensin `dbt`-komennot, jotta `warehouse.duckdb` on ajan tasalla.
- [x] Testaa kummatkin tilanteet:
    - [x] on ensin ajettu `rm -r bi/workspace` ja sitten ajetaan kummatkin `docker`-komennot. Kontin pitäisi päivittyä kun `index.md`:tä muokataan.
    - [x] workspace on valmiiksi olemassa, mutta tämän koneen Docker-ympäristöstä puuttuu `evidence_node_modules`. Muista tuhota se ensin yllä olevan testin jäljiltä. Muutoin testi on sama kuin yllä.

#### Part 2/2

- [x] Kirjoita `update_data.sh` skripti, joka ajaa tarvittavat `dbt`-rimpsut, jotta warehouse on ajan tasalla, ja lopulta kopioi sen `bi/workspaces/sources/warehouse/warehouse.duckdb` tiedoston päälle.
- [x] Tutustu [Evidence Docs: Build Your First App](https://docs.evidence.dev/build-your-first-app/)
- [x] Luo source:
    - [x] HOX! Koska Compose Watch:n sync on yksisuuntainen `host -> container`, et voi luoda tietolähteitä Evidence GUI:ssa. Tee tämä ihan vain luomalla tiedostoja.
    - [x] Luo `connection.yaml` ja `jaateloauto_weekly_lateness.sql` esimerkkinä toimivan `needful_things` kaltaisesti.
    - [x] Testaa Evidence SQL Consolessa `select * from warehouse.jaateloauto_weekly_lateness`
- [x] Luo itse sivu:
    - [x] Muokkaa `index.md`-tiedostoa.
    - [x] Luo perus SQL code block, kuten ohjeissa neuvotaan, jotta saat datan aliakseen.
    - [x] Kokeile ihan vain `<DataTable data={code_block_nimi_ylta} />`
    - [x] Lisää `<BarChart />`, jonka pitäisi istua tähän käyttöön varsin hyvin.
        - [x] `data=`
        - [x] `x=week_monday`
        - [x] `y=p50_departure_lateness`
        - [x] `y2=p90_depatrure_lateness`
        - [x] `xFmt=shortdate`
        - [x] `xAxisTitle=Week`
        - [x] `yAxisTitle=Minutes`
    - [x] Ja toinen, jossa on lisäksi `series=truck_id` ja `type=grouped` tai `=stacked`
- [x] Puske GitLabiin

## Videolla esitettävä

Tässä harjoituksessa videon tulee osoittaa alustasi ja datarakenteesi keskeisimmät oivallukset. Koska teit tässä useita vaiheita, tiivistä presentaatio näyttämään prosessin kulku. Videolla näkyy **vähimmillään** seuraavat vaiheet:

1. **Aloitus ja tavoite:** Kerro valitsemasi junadatan bisnesongelma (esim. joidenkin tiettyjen junien myöhästymiset) ja näytä, mihin REST API -päätepisteeseen olet yhdistänyt.
2. **Raakadatan nouto:** Näytä Python-skriptisi (ingestion tool), joka hakee datan, ja demonstroi, että raakadata tallentuu onnistuneesti `.json` tai `.csv` -muodossa Datalaken staging-kansioon.
3. **Bronze (dbt & DuckDB):** Esittele, miten dbt tekee raakadatasta DuckDB-tietovaraston Bronze-tason taulun (näytä esim. SQL-kyselysi tai DuckDB UI -näkymä).
4. **Silver & Gold -mallinnus (dbt):** Esittele tekemäsi dimensionalinen mallinnus (Silver-tason ratkaisut) sekä lopputuloksena syntyvä Gold-tason aggregaatiotaulu.
5. **dbt Docs & Lineage:** Avaa `dbt docs` selaimessa ja näytä projektisi data lineage -graafi varmistaaksesi putken oikeellisuuden.
6. **BI:** Esittele lopuksi dashboard, joka hyödyntää DuckDB:n Gold-tasoa ja vastaa alussa asettamaasi bisnesongelmaan. Voit tehdä kerroksen valitsemallasi työkaulla; ei ole pakko käyttää Evidenceä.

