# ORC01: Apache Airflow

Apache Airflow on työnkulkujen orkestrointityökalu. Siinä työnkulku kuvataan DAGina (Directed Acyclic Graph): joukko tehtäviä (*task*), niiden väliset riippuvuudet ja ajastussääntö. Airflow'n Scheduler tunnistaa, milloin DAG on aika suorittaa, jonottaa tehtävät ja antaa ne Executor-prosessin suoritettavaksi. Web UI näyttää työnkulkujen tilan reaaliajassa ja tarjoaa käyttöliittymän manuaaliseen käynnistämiseen sekä lokien selaamiseen.

Tässä harjoituksessa ajetaan Airflow'ta paikallisesti Docker Composessa LocalExecutor-moodissa: Scheduler ja Web UI pyörivät samassa prosessissa eikä erillistä Worker-klusteria tarvita. Rakennat ETL-tyyppisen DAGin, ajat sen Web UI:sta, seuraat suorituksen etenemistä ja harjoittelet virheen paikantamista lokeista.

Voit tutustua myös [Airflow 101: Building Your First Workflow](https://airflow.apache.org/docs/apache-airflow/stable/tutorial/fundamentals.html)-tutoriaaliin, joka on tämän harjoituksen pohjaesimerkki ja lähtökohta. Tämä on yksinkertaistettu versio samasta. Tutustu myös [Quick Start](https://airflow.apache.org/docs/apache-airflow/stable/start.html)-ohjeeseen. Dockeriin liittyvä ohjeistus löytyy [Running Airflow in Docker](https://airflow.apache.org/docs/apache-airflow/stable/howto/docker-compose/index.html)-sivulta.

## Esivaatimukset

- Docker ja Docker Compose

## Valmistelut: Airflow-ympäristö

### Lataa Docker Compose -tiedosto

Luo hakemisto harjoitustasi varten, esimerkiksi `airflow-harjoitus/`. Luo sinne alihakemistot `dags/` ja `data/`. Lisää hakemistoon `docker-compose.yaml` (tai sama tiedosto tuoreemmalla `compose.yaml` nimellä). Tiedsoton sisältö on niin pitkä, etten listaa sitä alla kokonaisuudessaan. Lataa tiedosto alla olevalla komennolla, jos sinulla `curl` tai `wget` asennettuna:

```bash
# curl
curl -LfO 'https://airflow.apache.org/docs/apache-airflow/3.2.1/docker-compose.yaml'

# ...tai wget
wget 'https://airflow.apache.org/docs/apache-airflow/3.2.1/docker-compose.yaml'
```

Jos sinulla ei ole jostain syystä kumpaakaan, voit ladata selaimella navigoimalla yllä olevaan URL:iin. Korostan, että on suositeltavaa silmäillä sivussa Airflow:n omaa, virallista dokumentaatiota, jonka pohjalta tämä harjoitus on rakennettu. Se löytyy osoitteesta [Running Airflow in Docker](https://airflow.apache.org/docs/apache-airflow/stable/howto/docker-compose/index.html).


### Initialisoi konfiguraatio

Älä aja sokkona `docker compose up` -komentoa, vaan noudata ohjeistusta ja aja ensin initilisointi. Tarkat ohjeet löytyy Airflow:n dokumentaatiosta, mutta alla on TLDR:

```bash
# Jos sinulla on Linux, aja:
mkdir -p ./dags ./logs ./plugins ./config
echo -e "AIRFLOW_UID=$(id -u)" > .env

# Jos sinulla on jokin muu OS, voit luoda tyhjän .env-tiedoston, jotta ei tule herjoja
touch .env

# Ja sitten:
docker compose run airflow-cli airflow config list
```

Suosittelen lämpimästi tässä välissä:

1. Muokkaa `config/airflow.cfg`-tiedostoa seuraavan rivin osalta:
    
    * Oletus: `logging_level = INFO` 
    * Uusi arvo: `logging_level = WARNING`. 

    Tämä vähentää merkittävästi tulostuvan lokitiedon määrää.

2. Silmäile `docker-compose.yaml`-tiedoston sisältöä. Tutustu pintapuoleisesti eri palveluihin ja eri bind mountteihin.

### Initialisoi tietokanta

Tämä ohje pätee kaikkiin käyttöjärjestelmiin:

```bash
# Tämä ajetaan tasan kerran. Se luo PostgreSQL-tietokannan ja käyttäjätiedot, 
# joita Airflow käyttää metatietokantanaan.
docker compose up airflow-init
```

### Aja palvelu ylös

```bash
# Flägi -d on valinnainen; voit ajaa sen myös suoraan etualalla, 
# nähden kaikki lokit ja välivaiheet ilman lisäkomentoja.
docker compose up -d
```

Odota, että palvelu on ylhäällä, ja testaa se:

1. Avaa selaimella `localhost:8080`. Näet Airflow'n Web UI:n.
2. Kirjaudu sisään: `airflow / airflow` (käyttäjätunnus / salasana).

## Tehtävänanto

!!! warning

    Työvaiheet puuttuvat vielä. TODO! Work in progress.

## Videolla esitettävä

!!! warning

    Tämä lista on WORK IN PROGRESS. Tarkista se.

Tässä harjoituksessa videon tulee osoittaa vähintään seuraavat asiat:

1. Kerrot, kuinka monta tuntia käytit harjoitukseen.
2. Selität lyhyesti, mikä on Apache Airflow ja mikä on DAGin rooli (yleisönä toisen tiimin jäsen, ei datainsinööri).
3. Käynnistät Docker Compose -palvelun videolla (`docker compose up -d`).
4. Avaat Web UI:n (`localhost:8080`) ja näytät, kuinka `orc01_etl`-DAG ilmestyy listaan.
5. Ajat DAGin manuaalisesti ja näytät Graph-näkymästä, kuinka taskit etenevät `success`-tilaan.
6. Avaat yhden taskin lokitiedot Web UI:sta ja luet, mitä sieltä löytyy.
7. Näytät tahallisen virheen aiheuttaman epäonnistumisen ja sen paikantamisen lokeista.




