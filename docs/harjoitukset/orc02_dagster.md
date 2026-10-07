# ORC02: Dagster

!!! warning

    On suositeltavaa suorittaa aiempi `ORC01`-harjoitus. Tässä materiaalissa verrataan Dagsterin ja Airflow:n paradigmoja, joten aiemin kokemus Airflow:sta auttaa. Tämä ei kuitenkaan ole välttämätön vaatimus.

Tämän tutoriaalin pohjana toimii Dagster-dokumentaation [Build your first Dagster pipeline](https://docs.dagster.io/getting-started/quickstart) -opas.

Airflow ja Dagster ovat kumpikin workflow orchestrator -työkaluja. Ennen Airflow 3 -versiota näiden välinen ero oli nykyistä suurempi. Dagster on _asset-centric_ ja Airflow _task-centric_. Tämä ero on kuitenkin kaventunut; jos teit `ORC01`-harjoituksen, olet jo käyttänyt Airflow:n asset-ominaisuutta. Lyhyt selitys näistä lähestymistavoista on:

- **Task-centric**: keskittyy suoritettaviin tehtäviin (tasks). Määritellään, mitä tehtäviä suoritetaan ja miten ne suoritetaan (eli DAG).
- **Asset-centric**: keskittyy tuotettaviin artefakteihin (assets). Dagsterissa määritellään, mitä dataa tuotetaan ja miten se tuotetaan.

Se, kumpi lähestymistapa on parempi, riippuu käyttötapauksesta. Asset-keskeisyys on hyödyllistä, kun syntynyt data on ns. _first-class citizen_, ja koko workflow edustaa ajatusta _"a graph of things that should exist"_. Kun data on keskiössä, on mahdollista suunnitella työkalu siten, että dataan liittyvät ominaisuudet kuten _data lineage_ eivät vaadi irrallista työkalua, ja nimenomaan dataan liittyvät työkalut kuten `dbt` integroituvat natiivisti yhteen. Kaikki maailman työnkulut eivät kuitenkaan tuota selkeää data-artefaktia. Olisi siis kovin väärin väittää, että jompi kumpi näistä lähestymistavoista tai mainituista työkaluista olisi parempi kuin toinen. Molemmilla on paikkansa, ja molempia käytetään laajasti.

Kumpikin työkalu toki itse mainostaa itseään kilpailijaa paremmaksi. Airflow:ta kaupallistavan Astronomer-yhtiön kirjoituksen [Task-based vs. asset-based orchestration (2026)](https://www.astronomer.io/airflow/astro-vs-dagster/) tiivistä asian: _"Dagster might continue to be a strong choice for greenfield, dbt-centric analytics teams. Airflow remains the standard for production data platforms at scale."_ Sen sijaan Navid kirjoittaa Dagsterin blogipostauksessa [The Case for Dagster: Moving Beyond Airflow in the Modern Data Stack™](https://dagster.io/blog/moving-beyond-airflow-in-the-modern-data-stack), että _"Yes, it only took them three years to catch up to where Dagster was in 2021."_

Datainsinöörinä sinun tehtävä on tunnistaa yrityksen ongelmat ja tarpeet ja sovittaa näihin sopiva työkalu. Joskus tarvitset monta työkalua rinnakkain. Älä ole _fangirl_ tai _fanboy_; valitse työkalu tarpeen mukaan. Tällä kertaa tutustutaan harjoituksessa Dagsteriin.

## Esivaatimukset

- uv

## Tehtävänanto

### 1: Luo Dagster-projekti

Muistathan, että tämä on vain uudelleenmuotoiltu [Quickstart](https://docs.dagster.io/getting-started/quickstart) ja [Tutorial](https://docs.dagster.io/dagster-basics-tutorial). Kannattaa silmäillä noita samalla. Lisäksi, jos Dagster on merkittävästi päivittynyt, voi olla että joudut kaivaa alkuperäisestä ohjeesta tuoreemman komennon. Tällöin myös opettajaa kannattaa huomauttaa päivittämään materiaalia.

Dagster tarjoaa oman init-tyylisen komennon, jolla voi luoda projektin templaatin avulla. Aja siis:

```bash
cd to/your/workspace
uvx create-dagster@latest project orc02
```

Asennus kysyy, että haluatko käyttää `uv.lock`-tiedostoa virtuaaliympäristön määrittelyyn. Vastaa kyllä. Tämän jälkeen voit siirtyä luotuun projektiin:

```bash
cd orc02
```

Jatkossa voit lisätä tarvitsemiasi riippuvuuksia `uv add`-komennolla, jolloin se lisätään `pyproject.toml`-tiedostoon riippuvuutena, esimerkiksi:

```bash
uv add polars
```

### 2: Tutustu projektin rakenteeseen

Tässä välissä kannattaa rauhassa pysähtyä hetkeksi ja tutustua hakemistorakenteen sisältöön. Aja komento `tree` ja/tai avaa tiedostot valitsemassa editorissa, kuten Visual Studio Codessa.

### 3: Luo assets.py

Käytä Dagsterin tutoriaalin mukaisesti komentoa `defs`. Minä en turhaan aktivoi virtuaaliympäristöä vaan ajan joka komennon `uv run`-etuliitteellä, jolloin `uv` käyttää oikeaa virtuaaliympäristöä. Aja siis:

```bash
uv run dg scaffold defs dagster.asset assets.py
```

!!! tip

    Nämä definitionit kuuluvat hyvinkin tiettyyn paikkaan, mistä Dagster niitä etsii. Sen määrittää nämä rivit koodia `definitions.py`-tiedostossa:

    ```python
    @definitions
    def defs():
        return load_from_defs_folder(path_within_project=Path(__file__).parent)
    ```

    Dagster etsiis siis `definitions.py`-tiedoston hakemistosta hakemistoa `defs/`, josta se lataa kaikki assetit. Tässä tapauksessa uusi definition syntyy tiedostoon `src/orc02/defs/assets.py`.

### 4: Lataa pingviinidata

Käytetään samaa pingviinidataa kuin `ORC01`-harjoituksessa. Lataa se siis uudelleen:

```bash
PENGUINS="https://raw.githubusercontent.com/mwaskom/seaborn-data/master/penguins.csv"

mkdir src/orc02/defs/data
cd src/orc02/defs/data
curl -O $PENGUINS
```

### 5: Määrittele processed_penguins_csv assetti

Muokkaa aiemmin mainittua `src/orc02/defs/assets.py`-tiedostoa. Lisää sinne seuraava koodi. Assetti tuottaa `processed_penguins.csv`-tiedoston:

```python title="src/orc02/defs/assets.py"
import polars as pl
import dagster as dg

penguins_file = "src/orc02/defs/data/penguins.csv"
processed_penguins_file = "src/orc02/defs/data/processed_penguins.csv"

@dg.asset
def processed_penguins_csv():
    # Read data from the CSV
    df = pl.read_csv(penguins_file)

    # Add a body_mass_group column based on body mass
    df = df.with_columns(
        pl.col("body_mass_g")
        .cut(
            breaks=[3500, 4500],
            labels=["Light", "Medium", "Heavy"],
        )
        .alias("body_mass_group")
    )

    # Save processed data
    df.write_csv(processed_penguins_file)

    return "Penguin data processed successfully into discrete body mass groups"
```

### 6: Tarkista definitionit

Aja lisäksi tutoriaalissa mainitut komennot ja tutustu outputtiin:

- `uv run dg list defs`
- `uv run dg check defs`

### 7: Käynnistä Dagster UI

Aja komento:

```bash
uv run dg dev
```

Tämän jälkeen löydät `localhost:3000`-osoitteesta Dagsterin käyttöliittymän. Tutustu siihen.

### 8: Materialisoi assetti ja tarkista tulos

Aja assetin pipeline web-käyttöliittymästä:

1. Mene Assets-välilehdelle klikkaamalla vasemmasta laidasta "Catalog"
2. Klikkaa oikeasta ylälaidasta "View lineage"
3. Klikkaa "Materialize"-painiketta.

!!! tip

    Dagsterin CLI ja sen dokumentaatio on kovin vahvasti Cloud-versioon kallellaan. Huomannet, että `--help` ei yleensä tarjoa mitään hyödyllistä. Dagsterin tutoriaalin perusteella seuraava komento kuitenkin toimii:

    ```bash
    uv run dg launch --assets "processed_penguins_csv"
    ```

    Ainakaan opettajan kokeilulla tämä ei kuitenkaan tuota täysin haluttua tulosta. Web UI:ssa "Materilized"-kenttä ei päivity assetin kohdalla, kun sen ajaa CLI:stä. Kenties Dagster on kirjoitushetkellä jossain limbo-siirtymävaiheessa, jossa CLI ja web UI eivät ole täysin synkronissa. Joka tapauksessa, web UI:n kautta ajaminen toimii.

Avaa `src/orc02/defs/data/processed_penguins.csv`-tiedosto valitsemallasi ohjelmalla. Huomaat, että sarake `body_mass_group` on lisätty ja sen arvot ovat "Light", "Medium" tai "Heavy" riippuen pingviinin painosta. Voit tarkistaa sisällön esimerkiksi näin:

```bash
head src/orc02/defs/data/processed_penguins.csv
```

### 9: Lisää DuckDB-riippuvuus

Aja seuraava komento:

```bash
uv add dagster-duckdb duckdb
```

Jatkossa voimme käyttää DuckDB:tä Dagsterin assettien kanssa.

### 10: Tallenna käsitelty data DuckDB-tauluun

Lisää uusi assetti, joka lukee käsitellyn CSV-tiedoston ja tallentaa sen DuckDB-tauluun. Tämä vaatii kaksi palasta: DuckDB-yhteyden määrittelevän resurssin ja assetin, joka käyttää sitä. Alla on vihje resurssista ja assetin rungosta, mutta ei koko uutta koodia.

Dagsterin tutoriaalin tapaan resurssi kannattaa sijoittaa omaan `resources.py`-tiedostoon. Voit luoda sen komennolla:

```bash
uv run dg scaffold defs dagster.resources resources.py
```

Täydennä tiedosto seuraavasti:

```python title="src/orc02/defs/resources.py"
import dagster as dg
from dagster_duckdb import DuckDBResource

database_resource = DuckDBResource(database="src/orc02/defs/data/penguins.duckdb")


@dg.definitions
def resources():
    return dg.Definitions(resources={"duckdb": database_resource})
```

Assetissa riittää, että määrittelet riippuvuuden `processed_penguins_csv`-assettiin, otat resurssin vastaan `duckdb`-nimisenä parametrina ja avaat yhteyden:

```python title="src/orc02/defs/assets.py"
@dg.asset(deps=[processed_penguins_csv])
def processed_penguins(duckdb: DuckDBResource):
    with duckdb.get_connection() as conn:
        # Lue processed_penguins.csv ja luo siitä taulu
        ...
```

!!! tip

    DuckDB osaa lukea CSV-tiedoston suoraan `read_csv_auto`-funktiolla, joten voit luoda taulun aiemmin kirjoitetusta `processed_penguins.csv`-tiedostosta. Käytä taulun luonnissa lausetta `CREATE OR REPLACE TABLE`, niin voit ajaa assetin uudelleen ilman virhettä.

### 11: Luo riippuva penguins_summary assetti

Lisää uusi DuckDB-assetti, joka riippuu edellisestä taulusta, muotoa...

```title="src/orc02/defs/assets.py"
@dg.asset(deps=[processed_penguins])
def penguins_summary(...):
    ...
```

Taulun sisällön tulee olla jokin yksinkertainen aggregaatti sarakkeista (esim. `species`), käyttäen sopivaa aggregaatiota (esim. `count`, `mean`, `sum`, `min`, `max`). Tallenna tulos DuckDB-tietokantaan.

### 12: Lisää asset check

Lisää `penguins_summary`-assettiin check, joka tarkistaa, että taulussa on vähintään 3 riviä. Dagsterin tutoriaalin vaihe [Step 1: Define an asset check](https://dagster.io/docs/dagster-basics-tutorial/asset-checks#step-1-define-an-asset-check) tarjoaa ohjeet.

### 13: Tutki lineagea ja tietokannan sisältöä

1. Materialisoi koko asset-ketju
2. Avaa UI:ssa lineage-näkymä
3. Tunnista ketju: `processed_penguins_csv` → `processed_penguins` → `penguins_summary`

### 14: Tutustu ulkoisten palveluiden orkestrointiin

Tähän asti Dagster ja varsinainen datan käsittely ovat toimineet samalla tietokoneella. Tuotantoympäristössä orkestrointipalvelu voi kuitenkin ohjata muualla tapahtuvaa työtä, aivan kuten myös esimerkiksi Airflow:n kanssa. Dagster toimii tällöin orkestraattorina, joka ohjaa ulkoisia palveluita. Käsitellään kuviteellisen yrityksen data-alustaa, johon kuuluvat:

- **Airbyte Cloud** siirtää datan lähdejärjestelmistä (ingestion)
- **Snowflake** tallentaa ja käsittelee datan (warehouse)
- **Dagster** määrittelee riippuvuudet, käynnistää työvaiheet ja seuraa niiden tilaa (orchestration)

Dagster ei tässä mallissa siirrä tai käsittele suurta datamäärää itse. Se kutsuu Airbyten ja Snowflaken rajapintoja ja seuraa työn etenemistä. Tutustu tähän liittyen seuraaviin dokumentaatiosivuihin:

- [Dagster & Airbyte](https://dagster.io/docs/integrations/libraries/airbyte)
- [Dagster & Snowflake](https://dagster.io/docs/integrations/libraries/snowflake)

Vastaukseen ei tarvitse sisällyttää toimivaa koodia. Voit halutessasi piirtää yksinkertaisen arkkitehtuurikuvan.

!!! note "BIG01 harjoitusta suunnitteleville"

    Dagster tukee myös dbt-projektien orkestrointia. Dagster voi tulkita yksittäiset dbt-mallit asseteiksi ja yhdistää ne samaan lineageen muiden työkalujen kanssa. BIG01-harjoituksessa rakennettavaa DuckDB- ja dbt-kokonaisuutta voisi siis myöhemmin orkestroida Dagsterilla. Laita tämä mahdollisuus korvan taakse.
    
    Datatyökalut ovat usein hieman LEGO-palikoiden kaltaisia: Airbyte voi vastata datan siirrosta, Snowflake tai DuckDB tallennuksesta ja käsittelystä, dbt muunnoksista sekä Dagster kokonaisuuden orkestroinnista. Työkalut valitaan bisnesongelman ja ympäristön tarpeiden mukaan.

## Videolla esitettävä

1. Kerrot, kuinka monta tuntia käytit harjoitukseen.
2. Selität lyhyesti, mitä asset tarkoittaa Dagsterissa.
3. Käynnistät Dagsterin komennolla uv run dg dev.
4. Näytät Catalog- tai Assets-näkymästä assetit processed_penguins_csv, processed_penguins ja penguins_summary.
5. Näytät lineage-näkymästä assettien väliset riippuvuudet.
6. Materialisoit koko asset-ketjun.
7. Avaat yhden ajon lokit ja kerrot, mitä siinä tapahtui.
8. Näytät DuckDB:n penguins_summary-taulun sisällön.
9. Näytät asset checkin tuloksen.
10. Selität lyhyesti, mitä hyötyä asset checkistä on verrattuna siihen, että pipeline vain suoritetaan onnistuneesti.
11. Esittelet lyhyesti suunnittelemasi Airbyte Cloud–Snowflake–Dagster-kokonaisuuden ja selität, mikä työkalu suorittaa datan siirron, mikä datan käsittelyn ja mikä orkestroi kokonaisuutta.