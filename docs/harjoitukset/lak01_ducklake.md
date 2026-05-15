# LAK01: Duck Lake

DuckLake on avoin Data Lakehouse -ratkaisu. Se yhdistää DuckDB:n suorituskyvyn Data Lake -tyyppisen tiedostopohjaiseen tallennukseen. Tiedot tallennetaan Parquet-tiedostoihin ja muutokset kirjataan erilliseen metatietokantaan (*catalog database*). Tässä harjoituksessa katalogitietokantana toimii PostgreSQL, joka pyörii Docker Composessa. Duck Lake toimii vastaavalla tavalla kuin esimerkiksi Databricksin käyttämä Delta Lake tai muut lakehouse-ratkaisut.

## Esivaatimukset

- Docker ja Docker Compose
- Marimo Notebook (`marimo[recommended]`).
- ... tai DuckDB CLI ja/tai DuckDB UI.

## Valmistelut: PostgreSQL-katalogi

Luo hakemisto harjoitustasi varten, esimerkiksi `ducklake-harjoitus/`. Lisää sinne alla oleva `compose.yaml`. Et tarvitse `volume`:a, koska katalogitietokanta on tarkoitettu tilapäiseen käyttöön harjoituksen aikana.

```yaml
services:
  postgres:
    image: postgres:17
    environment:
      POSTGRES_USER: ducklake
      POSTGRES_PASSWORD: ducklake
      POSTGRES_DB: ducklake_catalog
    ports:
      - "5432:5432"
```

Käynnistä palvelu:

```bash
docker compose up -d
```

Jos haluat tuhota sisällön ja aloittaa alusta, aja yksinkertaisesti `docker compose down; rm -r data_files/` ja sitten taas `docker compose up -d`.

## Tehtävänanto

Seuraa [DuckLaken virallista Getting Started -ohjetta](https://ducklake.select/docs/stable/duckdb/introduction) sekä [PostgreSQL-katalogiohjetta](https://ducklake.select/docs/stable/duckdb/usage/choosing_a_catalog_database.html) ja rakenna alla kuvattu kokonaisuus.

### 1. Asenna laajennukset ja yhdistä katalogiin

Avaa DuckDB CLI (`duckdb`) ja asenna tarvittavat laajennukset. Liitä sitten DuckLake-tietokanta käyttäen PostgreSQL-katalogina edellä käynnistämääsi tietokantaa. Muista myös asettaa `DATA_PATH` luomallesi `data_files/`-hakemistolle.

!!! tip "Data Inlining"

    Oletuksena DuckLake materialisoi datat levylle vasta 10 rivin muutoksen jälkeen. Jos haluat lisää läpinäkyvyyttä prosessiin, mikä on opiskellessa suositeltavaa, voit asettaa `DATA_INLINING_ROW_LIMIT`-parametrin nollaksi, jolloin kaikki muutokset materialisoidaan välittömästi. Lopulta sinulla tulee olemaan seuraavanlainen `ATTACH`-lause:

    ```sql
    ATTACH 'ducklake:postgres:
    dbname=ducklake_catalog 
    host=localhost 
    user=ducklake 
    password=ducklake' 
    AS my_ducklake (
    	DATA_PATH 'data_files/',
    	DATA_INLINING_ROW_LIMIT 0
    );
    ```

### 2. Luo kaksi taulua

Luo vähintään kaksi taulua, joilla on selkeä suhde toisiinsa. Voit esimerkiksi mallintaa yksinkertaisen tuote-myynti-asetelman:

- `tuotteet` — sisältää vähintään sarakkeet `id`, `nimi`, ja `hinta`
- `myynnit` — sisältää vähintään sarakkeet `id`, `tuote_id`, `maara`, ja `pvm`

Lisää kumpaankin tauluun muutama testitietue `INSERT`-lauseella.

### 3. Tarkastele katalogia

Tutki, miten katalogi muuttuu jokaisen operaation jälkeen. DuckLake tarjoaa `snapshots()`-funktion, joka palauttaa kaikki tehdyt muutokset aikajärjestyksessä:

```sql
FROM my_ducklake.snapshots();
```

Tutki myös, mitä Parquet-tiedostoja on syntynyt `data_files/`-hakemistoon.

### 4. Tee muutoksia ja seuraa vaikutuksia

Tee `UPDATE`- ja `DELETE`-operaatioita kumpaankin tauluun. Tarkastele `snapshots()`-tulosta jokaisen muutoksen jälkeen ja selitä, mitä katalogiin on kirjattu. Muutos voi olla esimerkiksi:

```sql
UPDATE tuotteet SET hinta = hinta * 4.20 WHERE id IN (1, 3)
```

### 5. Time travel

DuckLake tukee time travel -kyselyjä, joilla voidaan hakea taulujen data tietyn snapshottinumeron mukaan:

```sql
SELECT * FROM tuotteet AT (VERSION => 4);
```

Hae dataa niistä versioista, jotka edeltävät ja seuraavat tekemääsi `UPDATE`-operaatiota.

### 6. Tuhoa wanhat

Olisi sekä teknisistä että tietosuojallisista syistä ongelmallista, jos kaikki vanhat rivit, mukaan lukien tuhottu data, pysyisi loputtomasti Data Lakessa. Kokeile tuhota dataa! Tähän löytyy `ducklake_cleanup_old_files()`-funktio. Tulet tarvisemaan myös muita komentoja, joilla merkkaat tietyt snapshot-versiot tuhottaviksi.

!!! tip

    Voit kokeilla myös *compaction*-operaatiota, joka yhdistää pirstoutuneet Parquet-tiedostot yhdeksi (tai useammaksi) suuremmaksi tiedostoksi. Tästä voi olla hyötyä IO-suorituskyvyn kannalta. Tämä tunnetaan nimellä `ducklake_merge_adjacent_files()`.

## Videolla esitettävä

Tässä harjoituksessa videon tulee osoittaa vähintään seuraavat asiat:

1. Kerrot, kuinka monta tuntia käytit harjoitukseen.
2. Selität lyhyesti, mitä DuckLake on ja mikä on katalogin rooli (kuka on yleisössä: toisen tiimin jäsen, ei datainsinööri).
3. Käynnistät Docker Compose -palvelun videolla (`docker compose up -d`).
4. Avaat DuckDB CLI:n, asennat laajennukset ja liität DuckLake-tietokannan PostgreSQL-katalogiin.
5. Näytät, että kaksi taulua on luotu ja niihin on lisätty dataa.
6. Suoritat `snapshots()`-kyselyn ja selität, mitä se palauttaa.
7. Teet `UPDATE`- tai `DELETE`-operaation ja näytät `snapshots()`-kyselyn uudelleen — uusi snapshot on syntynyt.
8. Näytät, että `data_files/`-hakemistoon on syntynyt Parquet-tiedostoja (esim. globbing-kysely DuckDB:stä tai `ls`-komento terminaalista).
9. Teet time travel -kyselyn (`AT (VERSION => N)`) ja näytät, kuinka sama taulu palautti eri datan eri versiossa.
