# Tervetuloa kurssille

!!! warning

    Tämä materiaali on refaktoroinnin alla. Uusi toteutus käynnistyy syksyllä 2026.

Oppimateriaali on tarkoitettu Kajaanin Ammattikorkeakoulun toisen vuoden IT-alan opiskelijoille. Materiaalin ymmärtämisessä on eduksi, että ymmärtää SQL perusteet, tietokantojen perusteet, osaa Pythonia vähintään perusteiden verran, ja on käyttänyt Dockeria.

Tuleva rakenne on, että kurssi koostuu harjoituksista, jotka kukin arvostellaan [Arviointityökalun](https://arviointi.munpaas.com/) kriteeristöllä nimeltään ==videoitu demo==. Videot palautetaan Reppu-alustaan linkkeinä. Kustakin videosta saa painotetun määrän pisteitä. Jos harjoituksesta voi saada maksimissaan 20 pistettä, kaava on `{{ arviointityökalu }} * 4`. Huomaa, että harjoituksia ei ole numeroitu vaan kullakin on ID muotoa `ABC12`. Tämä on datanhallinnallinen valinta, jolla estää tilanne, jossa uuden harjoituksen lisääminen aiheuttaa numeroiden uudelleenjärjestelyä. Pistekertoimet on valittu siten, että pisteet pyrkivät mallintamaan niihin kuluvaa tuntimäärää: 20 pisteen harjoituksen kuvitellaan kestävän keskiverto-opiskelijalla 20 tuntia.

Esimerkki pisteistä:

| ID    | Otsikko      | Harjoituksen lyhyt kuvaus                                                                           | Max pisteet |
| ----- | ------------ | --------------------------------------------------------------------------------------------------- | ----------- |
| PUH01 | CSC Puhti    | Metaharjoitus, jossa opit ajamaan seuraavat harjoitukset oman koneen sijasta CSC Puhti -palvelussa. | 5           |
| LAK01 | Duck Lake    | Duck Lake -harjoitus, jossa kasataan minimaalinen Data Lakehouse.                                   | 10          |
| SRK01 | Apache Spark | Apache Spark -tutuksi lokaalin `pyspark`:n avulla.                                                  | 5           |
| CDC01 | Mealie CDC   | Mealie-sovelluksen CDC-tiedonkeruuharjoitus.                                                        | 20          |
| STR01 | Streaming    | Redpanda Streaming -harjoitus, jossa Kafka producer ja consumer Pythonilla.                         | 15          |
| AUR01 | Airflow      | Orkestraatioharjoitus, jossa käytetään Airflow-työkalua.                                            | 10          |
| KIM01 | Kimball      | Dimensionaalisen mallintamisen harjoitus.                                                           | 10          |
| CUB01 | OLAP-kuutio  | OLAP-kuutio rakentaminen Marimo Notebookissa.                                                       | 5           |
| BIG01 | Juna-alusta  | Digitrafficin Rautatieliikennedataa hyödyntävä End-to-End OLAP-alusta.                              | 30          |

Saamasi arvosana riippuu kokonaispisteitä, jotka olet kerännyt eri harjoitukset palauttamalla. Arvosanat määräytyvät seuraavasti:

| Pisteet | Arvosana |
| ------- | -------- |
| 0-49    | Hylätty  |
| 50-59   | 1        |
| 60-69   | 2        |
| 70-79   | 3        |
| 80-89   | 4        |
| 90-     | 5        |


## Faktavirheet

Mikäli oppimateriaali sisältää virheellistä tietoa, tee jompi kumpi:

* Forkkaa GitHubin repository ja tarjoa Pull Request, joka sisältää korjausehdotukset.
* Ota yhteyttä ylläpitoon ja esittele virheellisen tiedon korjaus.