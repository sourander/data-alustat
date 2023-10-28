On tärkeää muistaa, että data-alusta ei ole yksittäinen tuote vaan kollaasi eri työkaluja. Loppukäyttäjän näkökulmasta alusta voi olla _"se meiän firman power bi"_, mutta todellisuudessa kokonaisuus koostuu useista eri työkaluista. Lisäksi eri yritysten data-alustat voivat olla keskenään hyvin erilaisia. Yrityksellä voi olla yksi monoliittinen, keskitetty tietoalusta, tai hyvinkin hajautettu data-alustojen patteristo.

## Yleisnäkymä

### 50,000 jalan näkymä

![architecture-50000-foot](../images/architecture-50000-foot.svg)

**Kuvio 1:** _Karkea 50,000 jalan näkymä data-alustan arkkitehtuuriin._

Yllä olevan graafin (Kuvio 1) komponentit ovat vasemmalta lukien:

- **Sininen**: Lähdejärjestelmät (source system)
    - Datan alkuperäiset lähteet, kuten operatiiviset tietokannat.
- **Hopea**: Sisääntuonnin työkalut (ingestion tools)
    - Kerros, joka louhii tai vastaanottaa datan tietoalustaan.
- **Oranssi**: Toistaiseksi mustana laatikkona nähty kokonaisuus, joka sisältää tallennuksen ja laskennan (eng. storage & compute.)
- **Violetti**: Datan tarjoilukerros (eng. serving layer) on loppukäyttäjän näkymä dataan. Tyypillinen esimerkki on BI-työkalu kuten Power BI, Qlik tai Tableau, mutta kyseessä voi hyvin olla myös käyttötarpeeseen räätälöity verkkosivusto (esim. Plotly, JavaScript D3, React) tai suoraa kantayhteyttä muistuttava connector (esim. ODBC connector). Tietoalusta voi myös puskea dataa ulos, jolloin sitä ei ylipäätänsä noudeta, vaan se syötetään järjestelmään X.
- **Ihmisfiguuri**: Tietoalustan loppukäyttäjä. Heidän tarpeitaan varten koko alusta luodaan. Loppukäyttäjiä voi olla useita erilaisia, kuten myynti, ylin johto, tutkimus ja tuotekehitys, ohjelmistonkehittäjät, tietoturva-asiantuntijat, ja niin edelleen.

### 30,000 jalan näkymä

![architecture-30000-foot](../images/architecture-30000-foot.svg)

**Kuvio 2**: _Edellistä kuviota yksityiskohtaisempi 30,000 jalan näkymä data-alustan arkkitehtuuriin._

Huomaathan, että yllä olevassa Kuvio 2:ssa on jo valittu tiettyjä palveluita esimerkeiksi. Data-alustan **ei ole pakko** käyttää medaljonkiarkkitehtuuria (bronze, silver, gold) eikä mitään muutakaan yksittäistä valittua tuotetta. Työkalut ovat esillä liiallisen abstraktion välttelemiseksi.

## Laskennan hajautus

### Anti Big Data: DuckDB

Kaikki eivät niele täysin purematta big datan tarpeellisuutta kaikissa maailman yrityksissä. Tästä 2023 vuoden näkyvimpänä esimerkkinä on MotherDuck-yrityksen yhden noden kolumnaarinen OLAP-engine nimeltään [DuckDB](https://duckdb.org/). DuckDB ei siis ole tietokanta vaan Apache Sparkin kaltainen laskenta-engine. Tietokanta voi kuitenkin olla tiedostossa (esim. `db.duckdb`.)

!!! question "Tehtävä"

    Lue MotherDuckin Jordan Tiganin kirjoitus [Big Data is Dead](https://motherduck.com/blog/big-data-is-dead/). Mihin Jordan perustaa väitteensä?

!!! tip
    DuckDB on oikeasti olemassa oleva projekti, joka on julkaistu avoimen lähdekoodin lisenssillä. DuckDB on SQL-tietokanta, joka on suunniteltu toimimaan yhdellä tietokoneella. DuckDB on myös yksi esimerkki siitä, että data-alustan ei tarvitse olla monoliittinen, vaan se voi koostua useista eri työkaluista.

Alla esimerkki DuckDB-tyylisestä alusta, jossa kaikki data mahtuu yhden tietokoneen rammiin vähintään paloiteltuna. Huomaa, että arkkitehtuuri ei loogisella tasolla eroa kovinkaan paljon esimerkiksi yllä olevasta 30,000 jalan näkymästä, vaikka fyysisellä tasolla ero onkin merkittävämpi (yksi tietokone vs. monta hajautetun laskennan klusteria.) Kuvan alusta on jotakin, mikä on täysin tehtävissä 3 opintopisteen kurssin aikana.

![yhden-koneen-alusta](../images/yhden-koneen-data-alusta.png)

**Kuvio 3**: *Yhden koneen ja yhden tietolähteen data-alusta. Alustaa on helppo jatkaa suuremmaksi. Siihen voi tuoda uusia tietolähteitä, siirtää laskennan pilveen, siirtää tallennuksen pilveen, vaihtaa BI-työkalun, ottaa orkestrointiin dbt Cloudin ja/tai Airflow:n. Oikein käytettynä alustasta voisi olla hyvinkin realistista bisneshyötyä.*

!!! question "Tehtävä"

    Vertaa yllä esiteltyä alustaa Christophe Blefarin [Fancy Data Stackiin](https://www.blef.fr/the-fancy-data-stack/). Kuinka paljon yhtäläisyyksiä ja eroja löydät?


### Pro Big Data: Muut

Mikäli voidaan olettaa, että dataa on paljon, ja että dataa on useista eri lähteistä, on todennäköistä, että dataa ei voida tallentaa yhdelle koneelle (esim. `db.duckdb`-tiedostoon). Vaikka data hajautettaisiin pilvipalvelun tarjoamaan loputtamasti skaalautuvaan tallennustilaan eri tiedostoihin partitioituina, senprosessointi yhdellä koneella ei välttämättä ole realistista. Tällöin tarvitaan hajautettua laskentaa.

TODO! Tähän tulee myöhemmin lisää kontenttia.