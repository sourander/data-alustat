# SRK02: Delta Lake

!!! warning

    On äärimmäisen suositeltavaa suorittaa aiempi `SRK01`-harjoitus ennen tätä, jotta ymmärrät Spark Connectin ja Apache Sparkin peruskonseptit.

Delta Lake on avoin data lakehouse -kirjasto, joka tarjoaa ACID-takuita, time travel -ominaisuuden ja muita lisämausteita Parquet-tiedoston päälle. Delta Lake tallentaa datan parquet-muodossa ja ylläpitää metatietoa `_delta_log`-hakemistossa. Tämän hakemiston ja Delta Laken toimintaan tutustuminen on tämän harjoituksen pääasiallinen tavoite.

Ympäristö poikkeaa SRK01-harjoituksesta siten, että tässä kaikki pyörii Dockerin sisällä: sekä Marimo Notebookin Python-palvelin että PySpark (johon Delta Lake on integroitu). Käytämme bind mounttia, jotta voit tarkastella luotuja tiedostoja suoraan host-koneeltasi, mutta voit myös tutkia niitä Dockerin sisällä Linux-komentorivillä.

## Esivaatimukset

- Docker (Engine)
- Docker Compose

## Valmistelut: Spark + Delta Lake -ympäristö

Luo hakemisto harjoitustasi varten. Lisää sinne alla oleva `Dockerfile`:

```dockerfile
# ------------------------------------------------
# Dockerfile for SRK02: Delta Lake + Marimo
# ------------------------------------------------
# Note: this is heavily based on the delta-spark image, but with Marimo instead of Jupyter Lab
# For the original, see: https://github.com/delta-io/delta-docker/blob/main/Dockerfile
#
# The original file copyright notice is included below, but the modifications are licensed under the same Apache License 2.0.
# Copyright (2023) The Delta Lake Project Authors.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# You may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
# http://www.apache.org/licenses/LICENSE-2.0

ARG BASE_CONTAINER=apache/spark:4.1.1-scala2.13-java21-python3-ubuntu
FROM ${BASE_CONTAINER} AS spark
FROM spark AS delta

LABEL authors="Jani Sourander"

USER root

ARG DELTA_SPARK_VERSION="4.1.0"
ARG DELTALAKE_VERSION="1.4.2"
ARG MARIMO_VERSION="0.23.6"
ARG SPARK_SCALA_SUFFIX="4.1_2.13"

RUN pip install --quiet --no-cache-dir \
    delta-spark==${DELTA_SPARK_VERSION} \
    deltalake==${DELTALAKE_VERSION} \
    "marimo[recommended]==${MARIMO_VERSION}"

# Own addition begins ---------------------------
ENV IVY_HOME="/tmp/.ivy2"
ENV DELTA_PACKAGE="io.delta:delta-spark_${SPARK_SCALA_SUFFIX}:${DELTA_SPARK_VERSION}"

RUN mkdir -p ${IVY_HOME} && \
    ${SPARK_HOME}/bin/spark-shell \
        --packages ${DELTA_PACKAGE} \
        --conf "spark.jars.ivy=${IVY_HOME}"

RUN find ${IVY_HOME} -name "*.jar" -exec cp {} ${SPARK_HOME}/jars/ \;

RUN rm -rf ${IVY_HOME}
# Own addition ends ---------------------------

FROM delta AS startup
ARG NBuser=NBuser
ARG GROUP=NBuser
ARG WORKDIR=/opt/spark/work-dir


RUN groupadd -r ${GROUP} && useradd -r -m -g ${GROUP} ${NBuser}

RUN apt-get -qq update \
    && apt-get -qq -y install --no-install-recommends \
        vim nano curl tree \
    && rm -rf /var/lib/apt/lists/*

COPY --chown=${NBuser}:${GROUP} startup.sh "${WORKDIR}/"

RUN mkdir -p /opt/spark/logs && \
    chown -R ${NBuser}:${GROUP} /opt/spark/logs


RUN chown -R ${NBuser}:${GROUP} /home/${NBuser}/ \
    && chown -R ${NBuser}:${GROUP} ${WORKDIR}

USER ${NBuser}
WORKDIR ${WORKDIR}

ENTRYPOINT ["bash", "startup.sh"]
```

!!! tip "Versioiden yhteensopivuus"

    Huomaa, että Spark- ja Delta Lake -versioiden yhteensopivuus on tärkeää. Jos haluaisit päivittää jotakin tässä `ARG`-muuttujan arvoa eli versionumeroa, hyvä paikka aloittaa olisi etsiä tuorein [apache/spark Docker tag](https://hub.docker.com/r/apache/spark/tags) muotoa `apache/spark:X.Y.Z-scala2.13-java21-python3-ubuntu`, ja sitten tarkistaa [Delta Lake: Compatibility with Apache Spark](https://docs.delta.io/releases/)-sivun taulukosta, mikä versio tulee sen kanssa toimeen. Nämä eivät aina kulje käsi kädessä. Ajan kanssa myös Scala ja Java versiot voivat päivittyä ja aiheuttaa muutoksia. Näiden suhteen olisi hyvä lukaista `gh:delta-io/delta-docker/`-repositoryn Dockerfile esimerkkinä.

Tarvitset myös `startup.sh`-tiedoston, joka käynnistää Marimo Notebookin:

```bash title="startup.sh"
#!/bin/bash

exec marimo edit \
  --host 0.0.0.0 \
  --port 8888 \
  --no-token \
  /opt/spark/work-dir/notebooks/srk02_delta.py
```

Lisää samaan hakemistoon `compose.yml`:

```yaml title="compose.yml"
services:
  srk02-delta:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: srk02-delta
    ports:
      - "4040:4040"
      - "8888:8888"
    volumes:
      - ./notebooks:/opt/spark/work-dir/notebooks
      - ./data/lake:/opt/spark/work-dir/data/lake/
      - ./data/warehouse:/opt/spark/work-dir/spark-warehouse/
```

Käynnistä ympäristö:

```bash
docker compose up -d
```

**Käytettävissä olevat portit:**

- `localhost:8888` — Marimo Notebook (yksi tiedosto `srk02_delta.py`)
- `localhost:4040` — Spark Web UI (myöhemmin)

## Tehtävänanto

### (Valinnainen) Tutki Docker-kontin sisältöä

Käynnistä Docker-palvelu ja tarkastele, mitä hakemistoja on luotu:

```bash
docker compose exec srk02-delta bash
ls -la ls $SPARK_HOME/jars | grep delta
```

!!! tip

    Voit tarkastella koneen sisältöä pitkin tehtävää. Esimerkiksi hakemisto `/opt/spark/work-dir/data` on bind mountattu host-koneen työhakemiston `bind/data-lake/`-hakmeistoon, mutta Windowsin sijasta voit penkoa tiedostoja Linux-tyylisesti Dockerin sisällä. Tämä on hyvä paikka harjoitella Linuxin komentorivin käyttöä. Imageen on asennettu `tree`, `nano` ja `vim`, jotka auttavat tiedostojen tutkimisessa.


### 1. Yhdistä Marimo Notebookista

Tiedosto `notebooks/srk02_delta.py` on luotu jo valmiiksi sinulle (käynnistyskomennon `marimo edit /opt/spark/work-dir/notebooks/srk02_delta.py` toimesta), mutta sisältö luonnollisesti uupuu.

1. Avaa Marimo Notebook selaimessasi osoitteessa `http://localhost:8888`
2. Tab-täydennä ja aja solu, jossa lukee `import marimo as mo`
3. Lisää `setup cell`, jonka sisälle koodi:

```python title="srk02_delta.py"
import pyspark.sql.functions as F

from pyspark.sql import SparkSession
from delta.tables import DeltaTable

spark = (
    SparkSession.builder
    .appName("MyApp")
    .config("spark.sql.extensions", "io.delta.sql.DeltaSparkSessionExtension")
    .config("spark.sql.catalog.spark_catalog", "org.apache.spark.sql.delta.catalog.DeltaCatalog")
    .config(
        "spark.hadoop.javax.jdo.option.ConnectionURL",
        "jdbc:derby:;databaseName=/opt/spark/work-dir/catalog/metastore_db;create=true"
    )
    .config("spark.sql.catalogImplementation", "hive")
    .enableHiveSupport()
    .getOrCreate()
)

sc = spark.sparkContext
sc.setLogLevel("ERROR")
```

!!! tip

    Tuon solun ajamisen jälkeen portista `localhost:4040` löytyy Spark Web UI.

!!! bug

    On tyypillistä, että Spark -solujen ajaminen tulostaa kohtalaisen määrän Java-krääsää solun ulostuloon. Tavallisesta `WARN`-tason varoituksesta ei yleensä ole tarve huolestua. Lue teksti läpi oppimismielessä, mutta älä ylimieti virheilmoituksia. Jos olet epävarma, kysy opettajalta, tekoälyltä tai toisilta opiskelijoilta apua. Jos haluat nähdä vähemmän tietoja, voit lisätä seuraavat rivit yllä olevien perään:

    ```python title="srk02_delta.py"
    sc = spark.sparkContext
    sc.setLogLevel("ERROR")
    ```

    Huomaa, että tästä on apua vasta *seuraavissa* soluissa.

### 2. Tarkista Sparkin toimivuus

Aivan kuten Delta Laken omassa [gh:delta-io/delta-docker/quickstart.ipynb](https://github.com/delta-io/delta-docker/blob/main/quickstart.ipynb)-Notebookissa, testaa, että Spark toimii odotetusti:

```python title="srk02_delta.py"
_data = spark.range(0, 5)

(_data
    .write
    .format("delta")
    # .mode("overwrite")
    .save("/opt/spark/work-dir/data/lake/tablename")
)
```

Komento luo äärimmäisen yksinkertaisen 5-rivisen Delta-taulun hakemistoon `/tmp/delta-table`. Sen ei pitäisi nostaa virhettä. Jos yrität ajaa solun uusiksi, sen tulisi nostaa virhe (`[DELTA_PATH_EXISTS]`). Voit korjata tämän poistamalla kommenttimerkin `.mode("overwrite")`-metodikutsun edestä.

!!! tip

    Kannattaa nyt ja kaikissa tulevissakin komennoissa kurkata host-koneen `./data/lake/tablename`-hakemistoon. Uteliaalla insinöörillä olisi oletettavasti mielessään nyt ajatuksia:
    
    * Kuinka monta tiedostoa syntyi? Miksei vain 1? 
    * Mitä tarkoittaa `*.snappy.*` tiedostonimessä?
    * Miksi kutakin `*.parquet`-tiedostoa vasten on olemassa yksi `*.crc`-tiedosto?
    * Mitä `__delta_log/000.json`-tiedosto sisältää?

### 3. Aja sama solu uudestaan

Aja sama solu uudestaan poistettuasi kommenttimerkin `.mode("overwrite")`-metodikutsusta. Nyt sinulla tulisi herätä kysymyksiä, että:

* Ylikirjoittiko Spark (tai Delta Lake) samat tiedostot? Moodihan on `overwrite`.
* ... jos ei, niin mitä tapahtui?
* Mitä tiedosto `__delta_log/001.json` sisältää?

Voit ajaa solun niin monta kertaa kuin haluat.

### 4. Imuroi roskat pois

Kuvitellaan, että hyväksyt tilanteen, jossa paluuta vanhaan ei ole. Tuorein versio on hyvä. Tähän auttaa `VACUUM`-komento, jota voi luonnollisesti käyttää joko SQL-kielellä tai Python-syntaksin läpi. Tämä onnistuu seuraavalla tavalla:

```python title="srk02_delta.py"
_dt = DeltaTable.forPath(spark, "/opt/spark/work-dir/data/lake/tablename")
_dt.vacuum(0)
```

!!! warning "Virheilmoitus on odotettu!"

    Yllä oleva ei tule onnistumaan vaan kaatuu `IllegalArgumentException`-virheeseen. Keksi tapa korjata tämä. Vihje, tai jopa ratkaisu, löytyy virheilmoituksen tekstistä.

Kunhan olet saanut virheilmoituksen pois:

* Tutki `__delta_log/*.json`-tiedostoista kahta tuoreinta. Mitä löydät?
* Tarkista, mitä `*.parquet`-tiedostoja hakemistossa yhä on.

### 5. 10,000 riviä

Tämä vaihe on mukana auttamassa sinua ymmärtämään, miksi Parquet-tiedostona on aina useita. Tutki hakemiston sisältöä, kun ajat seuraavan solun tasan kerran. Laske `*.parquet`-tiedostot:

```python title="srk02_delta.py"
import multiprocessing

print("# of CPUs of Threads available: ", multiprocessing.cpu_count())

_data = spark.range(0, 10_000)
(_data
    .write
    .format("delta")
    .mode("overwrite")
    .save("/opt/spark/work-dir/data/lake/tenthousand")
)
```

Huomaa, että `spark.range()` luo oletuksena useita partitioita. Partitioiden määrä on tyypillisesti sidoksissa Sparkin oletusparallelismiin, joka yksinkertaisessa yhden noden Docker‑ympäristössä vastaa käytettävissä olevien CPU‑ytimien määrää. Kukin partitio toimii itsenäisesti, joten kukin luo oman tiedostonsa.

### 6. 10,000 riviä, kaksi tiedostoa


```python title="srk02_delta.py"
_data = spark.range(0, 10_000)
(_data
    .coalesce(2)
    .write
    .format("delta")
    .mode("overwrite")
    .save("/opt/spark/work-dir/data/lake/coalescetwo")
)
```

Näiden tutustumisen jälkeen sinulle pitäisi olla riittävä ymmärrys siitä, että eri RDD-partitiot (joita on tässä single-node ympäristössä `n_cpu_cores`) tuottavat kukin oman Parquet-tiedoston.

Jos haluat tutkia nykyisiä arvoja, voit ajaa seuraavan solun:

```python title="srk02_delta.py"
print("SparkContext default parallelism: ", spark.sparkContext.defaultParallelism)
print("Shuffle partition count: ", spark.conf.get("spark.sql.shuffle.partitions"))
```

## 7. Pingviinit tietovarastoon

```python title="srk02_delta.py"
from urllib.request import urlretrieve

# Choose table name
table_name = "d_penguins"

# Define schema with DDL syntax
schema = """
species STRING,
island STRING,
bill_length_mm DOUBLE,
bill_depth_mm DOUBLE,
flipper_length_mm INT,
body_mass_g INT,
sex STRING
"""

if not spark.catalog.tableExists(table_name):
    data_url = "https://raw.githubusercontent.com/mwaskom/seaborn-data/refs/heads/master/penguins.csv"
    urlretrieve(data_url, "/tmp/penguins.csv")

    df = (spark.read
          .schema(schema)
          .option("header", True)
          .option("inferSchema", True)
          .csv("/tmp/penguins.csv")
    )
    print("The CSV was read into N partitions:", df.rdd.getNumPartitions())

    # Add artificial ID column for later on MERGE reasons
    df = df.select(F.monotonically_increasing_id().alias("id"), "*")

    (df.write
       .format("delta")
       .mode("overwrite")  # safe because only runs once
       .saveAsTable(table_name)
    )

    print(f"Table '{table_name}' created.")
else:
    print(f"Table '{table_name}' already exists, skipping.")
```

!!! tip "Skeema käsin?"

    Spark osaa päätellä (engl. infer) skeeman CSV-tiedostosta, aivan kuten esimerkiksi Pandas tai Polars, mutta lähtökohtaisesti on hyvä määritellä skeema itse. Tietovarasto on aika hataralla pohjalla, jos sitä ohjaa autopilot eikä ihminen.

!!! tip "Tärkeää tietoa katalogista"

    Tähän asti olemme käyttäneet `save()`-metodia tarkan tiedostopolun kanssa. Tämä on tyypillistä **data lake**-työskentelyä, eli raakojen tiedostopolkujen kanssa pelaamista. 

    Nykyisin Sparkin ja Delta Laken kyljessä on jokin katalogi, kuten:

    * Hive Metastore + Derby (embedded-katalogi, kuin SQLite)
    * Hive Metastore + PostgreSQL-katalogi (`thrift://metastore-host:9083`)
    * AWS Glue Data Catalog
        * `spark.sql.catalog.glue_catalog.type=glue`
        * `spark.sql.catalog.glue_catalog.warehouse=s3a://deltalake/warehouse`
    * Databricks Unity Catalog

    Katalogi pitää kirjaa tauluista ja niiden sijainneista – muun metatiedon lomassa. Näin ei tarvitse koko ajan muistaa, missä mikäkin taulu sijaitsee, vaan riittää, että tietää taulun nimen. Vakiona meidän nykyisen katalogin materilisointi tapahtuu tiedostojärjestelmään, tarkemmin sanottuna hakemistoon `/opt/spark/work-dir/spark-warehouse/metadata_db`. Tämä mahdollistaa sen, että taulujen nimet pysyvät muistissa, vaikka sammuttaisit Dockerin ja käynnistäisit sen uudestaan.

    Voit kurkistaa katalogin sisältöön joko lokaalin hostin hakemistossa `data/catalog/` tai Dockerissa ajettavan Linuxin avulla seuraavasti:

    ```bash
    docker compose exec srk02-delta bash
    ls -la /opt/spark/work-dir/catalog
    ```


## 8. Tutki dataa

Tämä ==ei ole tuotannossa suositeltu tapa==, koska koko datasetin pitää mahtua driver-noden muistiin komennon suorittamiseksi, mutta tämän yksinkertaisen taulun kanssa voit halutessasi plärätä koko taulun sisältöä Marimo Notebookin interaktiivisella työkalulla. Tällöin siirrän taulun datat Marimo-prosessilla muistiin komennolla:

```python title="srk02_delta.py"
# Lataa KOKO DATA driverin muistiin
spark.sql("SELECT * FROM d_penguins").toArrow()
```

!!! tip

    Tuotannossa tutkisit vain aggredaatteja tai pientä samplea kerrallaan.

### 9 Aja tarvittavat SQL-kyselyt

Tutustu alla vaadittuihin asioihin, joiden tulee näkyä videolla. Aja tarvittavat SQL-kyselyt Marimo Notebookissa, jotta saat tarvittavat vaiheet tehtyä. Alla on esimerkkejä SQL-kyselyistä, joita muokkaamalla saat valtaosan tehtävästä tehtyä. Periaatteessa voit käyttää myös DataFrame-syntaksia, mutta koska meillä on tietovarasto ja katalogi, SQL on luonteva valinta.

Jos haluat datat aina kivasti näkyviin tekstimuodossa, aja komento formaatilla siten, että funktioketjun perässä on `.show()` tai `.show(truncate=False)`:

```python title="srk02_delta.py"
spark.sql("SELECT your_query_here").show()
```

Alla komentoja, joita voit muokata tarpeen mukaan. Huomaa, että komennot eivät välttämättä ole siinä järjestyksessä, jossa sinun ne tulisi ajaa. Nämä ovat rakennuspalikoita, joiden avulla ratkot oman tehtäväsi.

```sql
-- Datamäärä
SELECT COUNT(*) FROM d_penguins;

-- Selvitä kunkin sarakkeen NULL-arvon korvaajaksi sopiva arvo
SELECT
    mean(bill_length_mm) AS mean_bill_length,
    mean(bill_depth_mm) AS mean_bill_depth,
    mean(flipper_length_mm) AS mean_flipper_length,
    mean(body_mass_g) AS mean_body_mass,
    mode(sex)
FROM d_penguins
GROUP BY ALL;

-- Päivitä yhden sarakkeen NULL-arvot sopivalla korvaavalla arvolla
UPDATE d_penguins
SET bill_length_mm = 43.92
WHERE bill_length_mm IS NULL;

-- Laskee NULL arvot kussakin sarakkeessa
SELECT
    COUNT_IF(ISNULL(id)) AS null_id,
    COUNT_IF(ISNULL(species)) AS null_species,
    COUNT_IF(ISNULL(island)) AS null_island,
    COUNT_IF(ISNULL(bill_length_mm)) AS null_bill_length,
    COUNT_IF(ISNULL(bill_depth_mm)) AS null_bill_depth,
    COUNT_IF(ISNULL(flipper_length_mm)) AS null_flipper_length,
    COUNT_IF(ISNULL(body_mass_g)) AS null_body_mass,
    COUNT_IF(ISNULL(sex)) AS null_sex
FROM d_penguins;

-- Optimoi taulu
OPTIMIZE d_penguins;

-- Aja VACUUM
VACUUM d_penguins RETAIN 0 HOURS;

-- Näe wanha taulun versio
SELECT * FROM d_penguins VERSION AS OF 0;

-- Tuhoa taulu
DROP TABLE d_penguins;
```


!!! tip

    Jos tarvitset dokumentaatiota, niin Databricksin [SQL Language Reference](https://docs.databricks.com/aws/en/sql/language-manual) on mainio.

!!! tip

    Älä turhaan keuli SQL-taitojen kanssa siten, että rakennat 300-rivisen CTE/view/alikysely-himmelin tekoälyn avulla. Tämän harjoituksen kannalta on parempi, että irrallisia kyselyitä on useita, jotta lokihakemiston sisältö kasvaa komento komennolta.

    TUOTANNOSSA tietenkin kannattaa välttää miljoonaa pientä yksittäistä muutosta tietovarastoon, koska se aiheuttaa datan kopiointia ja tilan tuhlausta. Tämä on harjoitus, joten epäoptimaalinen toiminta sallittakoon.

Alla vielä erikseen ohjeet CDC-datan mergeä varten. Tässä on oletuksena, että jokin louhija on saanut pääteltyä, että kannassa on tehty Insert, Update ja Delete operaatioita. Tämä onnistuisi esimerkiksi BINLOG tai WAL dataa seuraamalla. Tästä aiheesta on erikseen harjoituksia, joten emme käsittele sitä sen tarkemmin. Hyväksy vain, että muutoshistoriaa on saatu noudattua lähdejärjestelmästä, ja haluamme varmistaa, että meidän dimensiotaulu pingiineistä on ajan tasalla.

```sql

-- Lisää uusi pingviini, jos ei ole jo olemassa preppaus:
-- VAIHE 1: Luoda CDC-dataa
-- Op ==> I=insert, U=update, D=delete
CREATE OR REPLACE TABLE cdc_updates
USING DELTA
AS
SELECT *
FROM VALUES
    ('I', 0,  'Duplicate', 'Event', 0.0, 0.0, 0, 0, 'unknown'),
    ('D', 1,  'Adelie',   'Torgersen', 39.5, 17.4, 186, 3800, 'MALE'),
    ('U', 2,  'Kana', 'Ahvenanmaa', 12.3, 12.3, 42, 1337, 'ROBOT'),
    ('I', 1000, 'Pulu',  'Ahvenanmaa', 39.1, 18.7, 181, 3750, 'MALE'),
    ('I', 1001, 'Lokki', 'Ahvenanmaa', 46.5, 17.9, 192, 3500, 'FEMALE')
AS v(
    Op,
    id,
    species,
    island,
    bill_length_mm,
    bill_depth_mm,
    flipper_length_mm,
    body_mass_g,
    sex
);

-- UPSERT
-- VAIHE 2: Merge.  
MERGE INTO d_penguins AS target
USING cdc_updates AS source
ON target.id = source.id
WHEN MATCHED AND source.Op = 'D' THEN
    DELETE
WHEN MATCHED AND source.Op = 'U' THEN
    UPDATE SET *
WHEN NOT MATCHED AND source.Op = 'I' THEN
    INSERT *;

-- Näytä lopputulos
-- VAIHE 3: Tarkista, että HALUTUT päivitykset ovat tietovarastossa.
SELECT * FROM d_penguins WHERE id IN (0, 1, 2, 1000, 1001);
```

Näiden komentojen dokumentaatioon on hyvä lähde Delta Lake -sivusto. Esimerkiksi artikkeli [Table deletes, updates, and merges](https://docs.delta.io/delta-update/) sekä [Change data feed](https://docs.delta.io/delta-change-data-feed/).

## Videolla esitettävä

Tässä harjoituksessa videon tulee osoittaa vähintään seuraavat asiat:

1. Kerrot, kuinka monta tuntia käytit harjoitukseen.
2. Selität lyhyesti, mitä Delta Lake on ja mikä on `__delta_log`-hakemiston
   rooli.
3. Käynnistät Docker Compose -palvelun videolla (`docker compose up -d`).
4. Avaat Marimo Notebookin ja näytät, kuinka yhdistät Spark Connectiin.
5. Näytät Delta-taulun luonnin ja tarkastelet `__delta_log`-hakemistoa.
6. Lisää/päivitä/poista dataa ja näytät, kuinka delta log elää.
7. Suorita myös MERGE-operaatio feikki CDC-datan avulla.
8. Näytät ja selität, mitkä CDC-operaatiot aiheuttivat pingviinitauluun muutoksia ja miksi.
9. Suoritat time travel -kyselyn ja näytät, kuinka sama taulu palauttaa eri datan eri versioissa.
10. Suoritat `VACUUM` ja näytät, kuinka vanhat tiedostot poistuvat.
