Prosessointi on ETL/ELT-lyhenteessä T-kirjain. Data luetaan jostain, prosessoidaan tavalla tai toisella, ja kirjoitetaan johonkin. Käsite on sen verran ympäripyöreä, että käsitellään se kahden esimerkin avulla. Kummassakin esimerkissä oletaan, että tieto-alusta perustuu Databricksiin, jolloin laskennasta vastaa Apache Spark.

!!! tip

    Apache Spark on koodattu Scala-ohjelmointikielellä, mutta driver node tarjoaa Python API:n (kuten myös SQL API:n). Suorituskyvyn kannalta on täysin sama, kirjoitetaanko koodi SQL:nä, Pythonina, Scalana vai R:nä.

## Case: Batch process 1 TB

Data on AWS:n S3-bucketissa (`s3://stage-bucket/staging/taulu/partition=yyyy-mm-dd/*.parquet`). Luot batch- eli eräajoon perustuvan prosessin, joka lukee datan stagingilta (Extract), prosessoi (Transform) sen tarvitulla tavalla, ja lataa (Load) sen määränpäähän. Dataa käsitellään leikisti noin 1000 gigatavua joka yö. Data kirjoitetaan Bronzelle. Myös Bronze koostuu Parquet-tiedostoista, jotka on tallennettu Delta Lake -kerroksen kera S3:een (`s3://medaljonki/tables/table=uuid-afasdf-afasf/`). 

!!! tip

    Huomaa, että pronssitaulun polussa taulun tunne on UUID. Tämä esimerkki mallintaa tilannetta, jossa tauluja hallinnoidaan Unity Catalogin avulla. Käyttäjä vastaa taulujen loogisesta osuudesta, Databricks huolehtii fyysisen materilisoinnin asiakkaan S3-buckettiin.

Kokeilet ajaa vertailun vuoksi prosessin muutamalla eri klusterikoolla. Kaikissa driver on sama (esim. `c5d.2xlarge`), ja sen hinta jätetään lyhyemmän esimerkin toiveissa huomioitta (joka on noin 6 €/kk jos prosessi kestää tunnin). Käytät samaa Pythonilla kirjoitettua pyspark-koodia kaikissa klustereissa. Koodi on seuraavanlainen:

```python
def fancy_process_thingy(df:DataFrame) -> DataFrame:
    # Some narrow transformations. No joins. No aggregations.
    return df

df = spark.read.parquet("s3://stage-bucket/staging/taulu/partition=yyyy-mm-dd/*.parquet")
df_out = fancy_process_thingy(df)
df_out.write.format("delta").save("s3://medaljonki/tables/table=uuid-afasdf-afasf/")
```

!!! tip

    Voit kokeilla hintalaskuria itse: [Databricksin laskuri](https://www.databricks.com/product/pricing/product-pricing/instance-types)

Alla mahdollinen hintataulukko:

| Cluster | executor_count | executor_type | Jobin kesto | Hinta per kk |
| ------- | -------------- | ------------- | ----------- | ------------ |
| A       | 4              | c5d.9xlarge   | 55 min      | 88           |
| B       | 8              | c5d.4xlarge   | 60 min      | 87           |
| C       | 16             | c5d.2xlarge   | 60 min      | 87           |
| D       | 32             | c5d.xlarge    | 60 min      | 88           |

Huomaat, että tässä tapauksessa on käytännössä sama, teetkö työn 4 isolla executorilla vai 32 pienellä. Mikäli sinulla olisi kiire, voisit tehdä jobin esimerkiksi 16x `c5d.4xlarge`:lla, ja se kestäisi noin 30 minuuttia. Tämä maksaisi yhä samat 88 dollaria. Kone olisi toki tuplasti kalliimpi kuin B-klusterin kone, mutta se olisi myös tuplasti nopeampi.

!!! warning

    Huomaa, että hintalaskuri ottaa huomioon vain Databricksin osuuden. Databricks luo koneet AWS:n palveluun, joten lopulta saat samasta tietokoneesta kaksi laskua: Databricksiltä ja AWS:ltä omansa. Kone c5d.4xlarge maksaa Databricksissä $0.09 per tunti. AWS laskuttaa koneesta $0.872 per tunti. Eräajoissa on mahdollista käyttää niin sanottuja spot-hinnastoja (vrt. pörssisähkö), jolloin AWS:n osuus voi olla merkittävästi pienempi. 

    Kaikissa yllä olevissa esimerkissä on oletettu, että Databricksissä kone on yöllä ajastettu jobi (Job Compute) ja yrityksellä on Premium-tilaus. Hinnat ovat lokakuulta 2023.

!!! question "Tehtävä"

    c5d.xlarge on Intel Xeon (Cascade Lake) -pohjainen virtuaalikone, jossa on NVMe SSD. X-large kokoluokan koneessa on 4 vCPU:ta (corea) ja 4 Gigaa muistia. CPU:RAM suhde on siis 1:1. Kone .4xlarge on nelinkertaisesti suurempi (16 vCPU ja 16 GB muistia).

    Suurin mahdollinen c5d-virtuaalikone on .24xlarge, jossa on 96 prosessoria (96 / 24 == 4) ja 192 GB rammia. Tämä on AWS:n palvelinkaapista yksi kokonainen kone; 24x:ää pienemmät koneet ovat siitä virtuaalisesti lohkottuja osia.

    Tutustu c5d-instansseihin [AWS:n dokumentaatiossa](https://aws.amazon.com/ec2/instance-types/c5/) sekä niiden hinnastoon [Amazon EC2 On-Demand Pricing](https://aws.amazon.com/ec2/pricing/on-demand/).


## Case: Aggregate and join process 1 TB

Myöhemmin prosessoit dataa Silveriltä Goldille. Teet uuden testin. Tällä kertaa data on useissa eri tauluissa, jotka sinun tulee joinata, ja lisäksi aggegoida dataa useiden eri sarakkeiden avulla. Silverillä olisi luontevaa käyttää SQL-kieltä, mutta pysyttäydytään tässä esimerkissä Pythonissa, jotta koodi pysyy mahdollisimman samanlaisena kuin edellisessä esimerkissä. Koodi on seuraavanlainen:

```python
def fancy_process_thingy(a, b, c) -> DataFrame:
    # Some wide transformations. Many joins. Such aggregations. Wow.
    return df

df_a = spark.read.format("delta").load("s3://medaljonki/tables/table=uuid-afasdf-afasf/")
df_b = spark.read.format("delta").load("s3://medaljonki/tables/table=uuid-ff00b1-12a00/")
df_c = spark.read.format("delta").load("s3://medaljonki/tables/table=uuid-727ed1-12345/")

df_out = do_your_thing(df_a, df_b, df_c)
```

| Cluster | executor_count | executor_type | Jobin kesto | Hinta per kk |
| ------- | -------------- | ------------- | ----------- | ------------ |
| A       | 4              | c5d.9xlarge   | 55 min      | 88           |
| B       | 8              | c5d.4xlarge   | 72 min      | 105          |
| C       | 16             | c5d.2xlarge   | 120 min     | 174          |
| D       | 32             | c5d.xlarge    | inf min     | N/A          |

Huomaat, että tässä tapauksessa **ei ole laisinkaan sama**, teetkö työn 4 isolla executorilla vai 32 pienellä. Pienin kone ei suoriutunut työstä laisinkaan; suuremmat executorit selviytyivät "wide"-tyylisistä operaatioista tehokkaammin.

## Todellisuuden monimutkaisuus

Todellisuudessa Apache Sparkin optimointi vaatii yllättävän monen seikan huomioon ottamista. Usein suurimmat optimoinnit syntyvät hyvinkin yksinkertaisilla koodiin tehtävillä ratkaisuilla. Isomman koneen ottaminen käyttöön ei ole suinkaan ensimmäinen tapa korjata ongelmalliset jobit.

!!! question "Tehtävä"

    Tämä Databricksin video näyttää käytännössä, miltä optimointiin tarjottu Spark UI sekä pikkuhiljaa vanhaksi käydä Ganglia näyttävät: [Data Collab Lab: Notes from the perf lab with fish and job (54 min)](https://www.youtube.com/live/JsY9UP3SI5c). Katso tai vähintäänkin silmäile video läpi.

    Kannattaa lukaista myös Databricksin [Best practices: Cluster configuration](https://docs.databricks.com/en/clusters/cluster-config-best-practices.html) sekä kilpailevan yrityksen Snowflaken [Warehouse Considerations](https://docs.snowflake.com/en/user-guide/warehouses-considerations).