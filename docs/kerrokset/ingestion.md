Tietolähteitä on monenlaisia. Huomaa, että kaikkia näitä voidaan tuoda tietoalustaan ETL tai ELT -työnkulkua noudattaen. Kirjaimet edustavat sanoja Extract, Transform, Load. Tiivistetysti ETL:n ja ELT:n ero on siinä, että ETL:ssä dataa muokataan ennen tietovarastoon vientiä, kun taas ELT:ssä dataa muokataan vasta tietovarastoon viennin jälkeen.

Nykyisin ELT on yleisempi lähestymistapa. Syitä tälle on monia, kuten:

* ETL:n T on vaikea määritellä pysyvästi. Jos T:n määritelmä muuttuu, data joudutaan hakemaan alusta alkaen uusiksi tietovarastoon.
* Raakadata itsessään voi olla hyödyllistä koneoppimisen näkökulmasta.

Ajoittain törmää myös termiin EtLT, jolla korostetaan sitä, että Extractin ja Loadin välillä pienimuotoista datan käsittelyä. Dataa ei siis välttämättä muokata lopulliseen tietovaraston tarvitsemaa muotoon, mutta sitä saatetaan käsitellä esimerkiksi anonymisoinnin tai pseudonymisoinnin vuoksi, surrogaattiavainten laskemiseksi, tai muiden syiden vuoksi. 

## SaaS vs. DIY

Tietoalustan rakentamiseen on useita eri lähestymistapoja. Yksi on rakentaa kaikki itse. Toinen on ostaa kaikki valmiina palveluna. Kolmas on yhdistää näitä kahta. Mikäli käytät ELT-lähestymistapaa, ingestion tool lataa tiedon joko Staging tai Bronze tasolle, riippuen miten olet halunnut määritellä sen. Tietoalustasi vastaa T-kirjaimen toteutuksesta eli eri tietolähteiden tiedojen mallintamisesta ja yhdistämisestä. Huomaa, että tämä on täysin modularisoitavissa. Sinulla voi olla rinnakkain useita eri ELT-työtyökaluja, joista kukin vastaa jostakin/joistakin tietolähteistä.

!!! question "Tehtävä"

    Tutustu seuraaviin SaaS-palveluihin. Mitä tietolähteitä ne tukevat?

    * [Fivetran](https://www.fivetran.com/connectors)
    * [Airbyte (Cloud)](https://airbyte.com/connectors)
    * [AWS AppFlow](https://aws.amazon.com/appflow/integrations/)
    * [Microsoft Fabric Data Factory](https://learn.microsoft.com/en-us/fabric/data-factory/connector-overview)
    * [Confluent](https://www.confluent.io/product/connectors/)

Yksittäisiin työkaluihin tutustuminen on kuitenkin tämän kurssin skoopin ulkopuolella. Mikäli sinulla on oikea yrityksen case, kannattaa vertailuttaa useita eri tarjoajia. Valittuun tarjoajaan voi vaikuttaa myös yrityksesi jo valmiiksi käyttämä hyperscaler (AWS, Azure, Google.)

## Tietolähteet

Alla on käsiteltynä tyypillisiä tietolähteitä alkaen tietokannoita ja päättyen web-sivuihin, joista tieto ladataan web-scrapingin avulla.

### Tietokannat

#### SQL ja noSQL

Eri SQL-kantojen (MySQL, MariaDB, Postgresql) ja noSQL-kantojen (MongoDB, Cassandra) dataa voi ladata eri strategioita hyödyntäen. SQL-kannoista palautuu lista monikkoja (`list[tuple]`), kun taas noSQL-kannoista palautuu JSON string tai lista sanakirjoja (`list[dict]`). Koska ingestion huolehtii vain latauksesta, ja T eli Transform tehdään vasta myöhemmissä vaiheissa, SQL ja noSQL saavat hyvin samankaltaisen kohtelun tässä vaiheessa: dumppaa data stagingiin ja murehdi myöhemmin.

* non-CDC: Full Load joka yö (naiivi)
* CDC: Inkrementaalista kenttää hyötyntäen
* CDC: Muutoshistorian lukeminen

!!! question "Tehtävä"

    Lataukseen hyödynnetään tietokannan ajuria (JDBC/OBDC), joka mahdollistaa tietokannan lukemisen Pythonilla. Tyypillisesti nämä voi asentaa `pip`:n avulla. Tutustu seuraaviin paketteihin pintapuoleisesti: `pymysql`, `psycopg2`.

!!! tip

    Termi CDC tarkoittaa Change Data Capture. Sillä tarkoitetaan lyhesti ottaen sitä, että lataustyökalu tai -skripti pyrkii hakemaan sen, mikä on uutta sitten edellisen haun. Tämä onnistuu esimerkiksi timestamp-kentän avulla, jolloin lataustyökalu tietää, että seuraavalla kerralla pitää hakea kaikki rivit, joiden timestamp on suurempi kuin edellisellä kerralla.

#### Timestamp CDC

Aikakoodikenttään tai monotonisesti kasvavaan ID-kenttään perustuva inkrementaalinen lataus on äärimmäisen yksinkertainen. Ensimmäisellä latauskerralla haetaan aivan kaikki rivit. Jatkossa haetaan vain rivit, joissa inkrementaalinen kenttä on suurempi kuin edellisellä kerralla.

Suuret `SELECT * FROM table`-tyyliset kyselyt ovat tietokantapalvelimelle muistin- tai kiintolevynkäytön kannalta erittäin raskaita. Ethän aja vastaavia kyselyitä tuotantokantaa vasten. Kriittiset kannat on suositeltavaa replikoida erilliseen tietokantaan, josta lataus voidaan tehdä. Jos ELT-skriptin kuorma kaataa palvelimen tai täyttää sen levytilan, tuotanto ei kärsi. Tämä on kallista, mutta turvallisempaa kuin tuotantokannan kuormittaminen.

Äärimmäisen naiivi latausskriptin alku voisi olla seuraavanlainen:

```python
# Import pymysql library
import pymysql
from imaginery_library import write_as_parquet

# Connect to the database
connection = pymysql.connect(host='localhost',
                             user='user',
                             password='password',
                             db='database'
)

if FULL_LOAD:
    query = "SELECT * FROM table"
elif CDC:
    query = f"SELECT * FROM table WHERE timestamp > {last_timestamp}"

# Execute the query and dump to staging
write_as_csv(connection.execute(query).fetchall())
```

!!! warning

    Timestamppiin perustuva inkrementaalinen lataus ei toteuta tietokannan poistoja laisinkaan. Tämä on potentiaalinen GDPR-rike. Tämän takia on suositeltavaa lukea muutoshistoriaa, jolloin poistotkin saadaan mukaan.

#### Binary Log CDC

Binary log on tietokantapalvelimen alunperin sisäiseen käyttöön tarkoitettu lokitiedosto, joka sisältää kaikki tietokannan muutokset. Binary logia voi lukea myös ulkopuolinen sovellus, jolloin se toimii CDC:n tavoin. Tietokannat käyttävät lokitiedostoa muun muassa homogeeniseen replikointii (esim. MySQL => MySQL) sekä vikaantumistilanteiden varalle. Binary log pitää erikseen olla aktivoituna tietokannassa; se ei välttämättä ole oletuksena päällä. Lisäksi binlogia ei säilytetä ikuisesti, joten sinulla on n päivää replikoida muutokset ennen kuin ne katoavat. Itse muutokset sisältävät operaation tyypin (write, update, delete) sekä rivin vanhan ja uuden tilan.

Mikäli CDC:n haluaa toteuttaa muutoshistoriaan nojaten, voi olla järkevintä unohtaa kotikutoinen Python-skripti, ja ottaa käyttöön jokin kaupallinen palvelu tai avoimen lähdekoodin työkalu (esim. Apache Kafka + Debezium tai Airbyte.)

Jos kuitenkin haluat tutustua konseptiin Pythonin avulla, tämä onnistuu esimerkiksi `python-mysql-replication`-kirjastolla.

```python
from pymysqlreplication import BinLogStreamReader
from pymysqlreplication.row_event import WriteRowsEvent, UpdateRowsEvent, DeleteRowsEvent

# Start from log_file position log_pos.
# Examples: log_file='mysql-bin.000003', log_pos=4
# Keep only events that affect rows. Ignore e.g. DDL events.
stream = BinLogStreamReader(
    connection_settings=mysql_settings,
    log_file=log_file, 
    log_pos=log_pos, 
    resume_stream=True,
    only_events=[
        WriteRowsEvent, 
        UpdateRowsEvent, 
        DeleteRowsEvent
    ]
)

# Process the binary log events
for binlog_event in stream:
    # Process
    pass

# Close connection
stream.close()
```

### Tiedostot

TODO: Tiedon jäsentyneisyyden tasot: Structured, Semi-structured, Unstructured.

### API

TODO: REST, GraphQL

### IoT

TODO: MQTT

### Streaming

TODO: Pub/sub

### Web pages

TODO: Scraping