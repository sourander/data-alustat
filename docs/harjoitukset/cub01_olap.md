# CUB01: OLAP-kuutio

Aloita aiheeseen tutustuminen teoriasta. Alla on tiivistelmä aiheesta, mutta katso pohjalle Stanfordin yliopiston Jennifer Widomin [Introduction to Databases](https://youtube.com/playlist?list=PLroEs25KGvwzmvIxYHRhoGTz9w8LeXek0&si=7J6IkuXfXf2QIezF)-soittolistan videot, jotka käsittelevät OLAP-kuutioita. Tunnistat ne `olap`-sanasta otsikossa. Videoilla selitetään CUBE, DRILLDOWN ja ROLLUP -lausekkeiden merkitys ja käyttö todella selkeiden esimerkkien avulla. Alla on tekstimuotoisena vastaavaa sisältöä, mutta videot avaavat aihetta todennäköisesti paremmin.

!!! tip

    Videon katsominen antaa sinulle erityisen hyvän lähtökohdan selittää, miten "kuutio" liittyy koko asiaan. Tämä löytyy videon 16-01-olap.mp4 kohdasta 9:20 eteenpäin.

SQL:n perinteinen `GROUP BY` jakaa datan ryhmiin valittujen sarakkeiden arvojen perusteella, jolloin kullekin ryhmälle voidaan laskea koontiarvoja aggregaattifunktioilla, kuten `COUNT()`, `SUM()` tai `AVG()`. Raportoinnissa ja analytiikassa pelkkä yksittäinen ryhmittelytaso ei kuitenkaan aina riitä, vaan rinnalle tarvitaan välisummia (engl. _subtotals_) eri ulottuvuuksien yhdistelmille sekä kaikkien rivien kokonaissumma (engl. _grand total_). Ilman laajennuksia tämä vaatisi useiden erillisten `GROUP BY` -kyselyiden kirjoittamista ja niiden tulosten yhdistämistä päällekkäin `UNION`-operaatiolla [^viescasEffectiveSQL].

SQL-standardi tarjoaa tähän kolme `GROUP BY` -lauseen laajennusta, joiden avulla useat eri ryhmittelytasot saadaan laskettua yhdellä kyselyllä [^viescasEffectiveSQL]:

- **`ROLLUP (a, b)`** muodostaa ryhmittelyt hierarkkisesti oikealta vasemmalle: ensin yhdistelmälle `(a, b)`, sitten pelkälle `(a)` ja lopuksi kokonaissummalle `()`. Se soveltuu luontevasti hierarkkisiin ulottuvuuksiin (esimerkiksi _vuosi → kuukausi → päivä_), mutta ei laske välisummia pelkän `b`:n mukaan.
- **`CUBE (a, b)`** laskee aggregaatit **kaikille mahdollisille sarakeyhdistelmille**: `(a, b)`, `(a)`, `(b)` sekä kokonaissumman `()`. Tätä moniulotteista koontia kutsutaan _OLAP-kuutioksi_ (engl. _OLAP cube_).
- **`GROUPING SETS ((a), (b), ())`** antaa määrittää täsmälleen ne ryhmittelyjoukot, jotka tulokseen halutaan, ilman että kaikkia mahdollisia yhdistelmiä tarvitsee laskea. Tyhjä sulkupari `()` tarkoittaa koko aineiston kattavaa kokonaissummaa.

Tulostaulussa arvo `NULL` ryhmittelysarakkeessa ilmaisee, että kyseinen rivi on koonti kaikkien kyseisen ulottuvuuden arvojen yli [^viescasEffectiveSQL]. BI-työkaluissa valmiiksi laskettu kuutio mahdollistaa datan suodattamisen ja tarkastelun eri ulottuvuuksien mukaan (_slice and dice_, _drill-down_, _roll-up_) keventäen samalla kyselyaikaista laskentakuormaa.

Voit tutustua aiheeseen myös eri tietovarastojen dokumentaatioiden avulla. Esimerkiksi:

- [DuckDB: GROUPING SETS](https://duckdb.org/docs/2.0/sql/query_syntax/grouping_sets), joka sisältää oikean vastauksen siemenen alla olevaan tehtävään.
- [RedShift: Aggregation extensions](https://docs.aws.amazon.com/redshift/latest/dg/r_GROUP_BY_aggregation-extensions.html)
- [BigQuery: Group rows by CUBE](https://docs.cloud.google.com/bigquery/docs/reference/standard-sql/query-syntax)
- [Snowflake: GROUP BY CUBE](https://docs.snowflake.com/en/sql-reference/constructs/group-by-cube)
- [Databricks: cube function](https://docs.databricks.com/sql/language-manual/functions/cube.html)

## Valmistelut

### Data

Data on kuvitteellista pingviinidataa, joka myötäilee `seaborn-data/penguins.csv`-tiedostoa. Data on alustettu valmiiksi Marimo Notebookiin, joten sinun ei tarvitse huolehtia siitä. Se on alla Markdown-tauluna referenssiksi:

| species   | island    | sex    | body_mass_g |
| :-------- | :-------- | :----- | ----------: |
| Adelie    | Torgersen | female |        3730 |
| Adelie    | Torgersen | female |        3200 |
| Adelie    | Biscoe    | male   |        3850 |
| Adelie    | Dream     | female |        3150 |
| Adelie    | Dream     | male   |        3700 |
| Adelie    | Dream     | female |        3050 |
| Chinstrap | Dream     | male   |        3800 |
| Chinstrap | Dream     | female |        3150 |
| Chinstrap | Torgersen | male   |        3700 |
| Chinstrap | Torgersen | female |        3050 |
| Chinstrap | Biscoe    | male   |        3750 |
| Chinstrap | Biscoe    | female |        3100 |
| Gentoo    | Biscoe    | male   |        5250 |
| Gentoo    | Biscoe    | female |        4800 |
| Gentoo    | Dream     | male   |        5250 |
| Gentoo    | Dream     | female |        4850 |

### Notebook

Luo Marimo Notebook. Jos haluat päästä helpolla, ota käyttöön opettajan valmis pohja, jossa on alustettu data ja yksi SQL-solu.

``` python title="penguin_cube.py"
# /// script
# requires-python = ">=3.14"
# dependencies = [
#     "duckdb==2.0.0",
#     "marimo[sql]>=0.25.2",
#     "polars==2.1.0",
# ]
# ///

import marimo

__generated_with = "0.25.2"
app = marimo.App(width="medium")


@app.cell
def _():
    import marimo as mo
    import polars as pl
    import duckdb

    conn = duckdb.connect()
    return conn, mo, pl


@app.cell(hide_code=True)
def _(mo):
    mo.md(r"""
    # OLAP Cube of Penguins
    """)
    return


@app.cell
def _(pl):
    data = {
        "species": ["Adelie", "Adelie", "Adelie", "Adelie", "Adelie", "Adelie", "Chinstrap", "Chinstrap", "Chinstrap", "Chinstrap", "Chinstrap", "Chinstrap", "Gentoo", "Gentoo", "Gentoo", "Gentoo"],
        "island": ["Torgersen", "Torgersen", "Biscoe", "Dream", "Dream", "Dream", "Dream", "Dream", "Torgersen", "Torgersen", "Biscoe", "Biscoe", "Biscoe", "Biscoe", "Dream", "Dream"],
        "sex": ["female", "female", "male", "female", "male", "female", "male", "female", "male", "female", "male", "female", "male", "female", "male", "female"],
        "body_mass_g": [3730, 3200, 3850, 3150, 3700, 3050, 3800, 3150, 3700, 3050, 3750, 3100, 5250, 4800, 5250, 4850]
    }

    penguins = pl.DataFrame(data)
    return


@app.cell
def _(conn, mo):
    _df = mo.sql(
        f"""
        SELECT * FROM penguins
        """,
        engine=conn
    )
    return


if __name__ == "__main__":
    app.run()
```

Aja Notebook ylös näin:

``` bash
uvx marimo edit --sandbox penguin_cube.py
```

## Tehtävänanto

### 1: Tee OLAP-kuutio

Kuution voi luoda DuckDB:n SQL:n CUBE-lauseen avulla.

``` sql
SELECT 
    species, 
    island, 
    sex, 
    COUNT() as penguins
FROM penguins
GROUP BY CUBE (species, island, sex);
```

Vaihda SQL-solun alla oleva **Output variable** -asetus oletuksesta (`_df`) muotoon `reference`. Näin sinulla on jatkossa Polars DataFrame muuttujassa nimeltään `reference`, joka sisältää ns. oikean vastauksen.

### 2: Suunnittele oma kuutiosi

Luo oma kuutiosi irrallisina SQL-lausekkeina, jotka yhdessä muodostavat kuution. Tämä on ns. työläs metodi, jota ei tarvitse koskaan todennäköisesti tehdä -- paitsi nyt oppimisen vuoksi!

OLAP-kuution voi kasata yhdistämällä päällekäin (`UNION ALL`) useat eri queryt, joista kukin tuottaa joidenkin ulottuvuuksien määrän. Tarvitsemme yhteensä 8 querytä, jotka tuottavat seuraavat yhdistelmät:

- (`N/A`) - Grand total
- (`sex`) - Group by one dimension
- (`species`)
- (`island`)
- (`species`, `island`) - Group by two dimensions
- (`species`, `sex`)
- (`island`, `sex`)
- (`species`, `island`, `sex`) - Group by all three dimensions

Suunnittele, miten lähetyt tätä ongelmaa siten, että ns. perjantaidemossa on helppo esitellä, mistä kokonaisuuksista CUBE koostuu. Opettajan ehdotus on, että:

1. Teet väliotsikon Marimo Notebookiin kutakin queryä varten, esimerkiksi `### Grand Total`, `### Group by sex`, jne.
2. Kunkin väliotsikon alla teet querystä oman VIEW:n.
3. Kunkin väliotsikon alla tulostat outputin.

### 3: Luo kuutiosi

Aloita kuution rakentaminen tekemällä tarvittavat 8 lauseketta yksitellen. Jos olet täysin jumissa nyt, niin klikkaa alla oleva vinkki auki, josta näet ensimmäisen solun sisällön.

Muista tarkistaa kaikkien välivaiheiden tulokset. Koko taulukossa on vain 16 riviä, joten mikä tahansa query pitäisi olla helppo tarkistaa silmämääräisesti.

??? tip "Vinkki: Grand Total"

    Ensimmäisen SQL-solun sisältö voisi olla esimerkiksi seuraava:

    ``` sql
    CREATE OR REPLACE TABLE grand_total AS
        SELECT 
            NULL AS species, 
            NULL AS island, 
            NULL AS sex, 
            COUNT(*) AS penguins
        FROM penguins;

    SELECT * FROM grand_total;
    ```

    Vastauksen tulisi olla tuple (`NULL`, `NULL`, `NULL`, 16), koska pingviinejä on yhteensä 16 kappaletta.

### 4: Yhdistä kuutiosi

Kun osat ovat valmiit, yhdistä ne lopuksi `UNION ALL` -lauseella. Tallenna lopputulos uuteen tauluun nimeltä `my_cube`. Tulosta kuution sisältö.

### 5: Vertaa kuutioita

Vertaa luomaasi kuutiota DuckDB:n CUBE-lauseella luotuun kuutioon. Keksi itse, mikä on paras tapa verrata kahta taulua keskenään: tähän on useita tapoja!

??? "Vinkki: Vertailu"

    Edellinen on jo lisätty `reference`-muuttujaan, joten hyvä lähtökohta on lisätä tämä `diycube`-muuttujaan (Polars DataFrame). Kenties voit esimerkiksi järjestää kummatkin taulut ja käyttää `df.equals(another_df)`-metodia? Tai kenties Polarsin `assert_frame_equal`-funktiosta on apua? 
    
    Keksitkö tavan, joka hoituu täysin SQL:llä siten, että oikein toimiessaan palautuu tyhjä tulos (eli 0 riviä dataa)?

### 6: Do It Yourself BI-työkalu

Rakenna työkalu, joka sallii valita graafisella käyttöliittymällä (GUI) halutut ulottuvuudet ja näyttää valittujen ulottuvuuksien valmiiksi lasketut koontiarvot. Luo tätä varten uusi `CUBE`-kysely, jossa lasket `COUNT`-lukumäärän lisäksi myös pingviinien painon (`body_mass_g`) summan (`SUM`) ja keskiarvon (`AVG`).

Huomaa, että kuutiossa `NULL`-arvo tarkoittaa kaikkien kyseisen ulottuvuuden arvojen yli laskettua koontia. Korvaamalla `NULL`-arvot esimerkiksi merkkijonolla `"Any"` (Polarsin `.fill_null("Any")`-metodilla) saat sen suoraan mukaan valitsimen vaihtoehdoksi ja suodatettavaksi arvoksi.

Suositeltava keino toteuttaa valinnat on Marimon `multiselect`-valitsin, jonka avulla yhtä ulottuvuutta voi tarkastella joko koontina (`"Any"`), yksittäisenä arvona tai useamman arvon vertailuna:

``` python
# Listojen avulla
select = mo.ui.multiselect(
    options=["a", "b", "c"], value=["a"], label="choose"
)

# ... tai sarakkeen arvoilla
select = mo.ui.multiselect.from_series(df["column_name"])
```

...ja käyttää toisessa solussa valittuja arvoja näin, käyttäen Polarsin `filter`- ja `is_in`-metodeja:

``` python
df_filtered = df.filter(pl.col("column_name").is_in(select.value))
```

!!! info "Miksi teemme näin?"

    Modernit BI-työkalut toimivat nopeasti, koska ne eivät laske keskiarvoja ja summia sadoista miljoonista riveistä joka kerta, kun käyttäjä painaa nappia. Ne käyttävät taustalla ennalta laskettuja koosteita (OLAP-kuutioita). Tässä tehtävässä rakennat tällaisen rakenteen itse: `CUBE`-kysely laskee kaikki mahdolliset yhdistelmät valmiiksi, ja käyttöliittymä (GUI) ainoastaan suodattaa näitä valmiita vastauksia.

    Katso käytännön vastineeksi ensimmäiset 2 minuuttia Pragmatic Worksin videosta [Master Power BI Slicers in 10 minutes!](https://www.youtube.com/watch?v=dNy43CKekGY&t=81s).

## Videolla esitettävä

1. Kerrot, kuinka monta tuntia käytit harjoitukseen.
2. Selität omin sanoin, miten "kuutio" liittyy OLAP-käsitteeseen ja mitä `NULL`-arvot tarkoittavat kuution ryhmittelysarakkeissa.
3. Näytät Marimo Notebookista DuckDB:n `GROUP BY CUBE` -lausekkeella luodun `reference`-taulun.
4. Esittelet lyhyesti kahdeksan erillistä SQL-kyselyäsi, joista oma kuutiosi rakentuu, sekä `UNION ALL` -lausekkeella yhdistetyn `my_cube`-taulun (`diycube`).
5. Näytät, miten vertasit `reference`- ja `diycube`-tauluja keskenään ja osoitat, että ne sisältävät samat tulokset.
6. Esittelet rakentamasi DIY BI -työkalun: näytät `CUBE`-kyselyn (`COUNT`, `SUM`, `AVG`) ja demonstroit valitsimilla, miten valmiiksi laskettu kuutio suodattuu eri ulottuvuuksien ja mittareiden mukaan.

## Lähteet

[^viescasEffectiveSQL]: Viescas, J, Steele, D & Clothier, B. _Effective SQL: 61 Specific Ways to Write Better SQL, First Edition_. Addison-Wesley Professional. 2016.
