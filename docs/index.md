# Tervetuloa kurssille

!!! warning

    Tämä materiaali on refaktoroinnin alla. Uusi toteutus käynnistyy syksyllä 2026.

## Kenelle?

Oppimateriaali on tarkoitettu Kajaanin Ammattikorkeakoulun toisen vuoden IT-alan opiskelijoille. Materiaalin ymmärtämisessä on eduksi, että ymmärtää SQL perusteet, tietokantojen perusteet, osaa Pythonia vähintään perusteiden verran, ja on käyttänyt Dockeria.

## Pisteytysmalli

Kurssi koostuu harjoituksista, jotka kukin arvostellaan [Arviointityökalun](https://arviointi.munpaas.com/) kriteeristöllä nimeltään ==videoitu demo==. Videot palautetaan Reppu-alustaan linkkeinä. 

Kustakin videosta saa `raakapistemäärän`, joka on välillä 0–5. Tämä pistemäärä kerrotaan harjoitukselle määritellyllä kertoimella, jolloin saadaan harjoituksesta saatavat kurssipisteet. Opiskelijana voit käytännössä unohtaa tämän kaavan ja seurata harjoituksesta saatuja loppupisteitä. Jos aihe kuitenkin kiinnostaa, avaa alla olevat admonition-laatikot.

??? info "Miten kurssipisteet lasketaan? Python edition"

    ```python
    # Lista, johon kerätään kaikkien harjoitusten kurssipisteet
    tehtyjen_harjoitusten_pisteet = []

    for harjoitus in tehdyt_harjoitukset:
        # Arviointityökalun antamat pisteet videodemon perusteella (0.0–5.0) * harjoituksen kerroin
        raaka = arviointi(harjoitus.demo_linkki)
        kerroin = harjoitusten_kertoimet[harjoitus.id]
        painotettu = raaka * kerroin
        tehtyjen_harjoitusten_pisteet.append(painotettu)

    # Kurssin kokonaispistemäärä on kaikkien harjoitusten pisteiden summa
    kurssin_kokonaispistemaara = sum(tehtyjen_harjoitusten_pisteet)
    ```

??? info "Miten kurssipisteet lasketaan? Moodle edition"

    Moodlen Grabook:ssa laskukaava on Course Totalin osalta:

    ```
    =sum([[CSC01]] * 2, [[LAK01]] * 4, [[SRK01]] * 2, ..., [[BIG01]] * 8)
    ```

Kertoimet on valittu siten, että *max pisteet* kuvastaa *keskimääräisiä* opiskelijan tehtävään kuluttamia tunteja. Kerään tilastotietoa harjoituksiin käyttämästänne ajasta, jotta voin optimoida tätä pisteytysmallia tulevaisuudessa. Sinun tulee siis kertoa tehtävään käyttämäsi aika videodemossa.

Alla listattuna kurssin harjoitukset (==HUOM! TÄMÄ ON YHÄ TODO eli WORK IN PROGRESS==):

| ID                                      | Otsikko      | Harjoituksen lyhyt kuvaus                                                                            | Max Pisteet (≈tunnit) | Deadline |
| --------------------------------------- | ------------ | ---------------------------------------------------------------------------------------------------- | --------------------- | -------- |
| [CSC01](harjoitukset/csc01_cpouta.md)   | cPouta       | Metaharjoitus, jossa opit ajamaan seuraavat harjoitukset oman koneen sijasta CSC cPouta -palvelussa. | 10                    | A        |
| [LAK01](harjoitukset/lak01_ducklake.md) | Duck Lake    | Duck Lake -harjoitus, jossa kasataan minimaalinen Data Lakehouse.                                    | 20                    | A        |
| [SRK01](harjoitukset/srk01_spark.md)    | Apache Spark | Apache Spark -tutuksi kontitetun `Spark`:n ja `spark-client`:n avulla.                               | 10                    | A        |
| SRK02                                   | Spark Delta  | Apache Spark -harjoitus, jossa käytetään Delta Lakea.                                                | 10                    | A        |
| [CDC01](harjoitukset/cdc01_mealie.md)   | Mealie CDC   | Mealie-sovelluksen CDC-tiedonkeruuharjoitus.                                                         | 20                    | A        |
| STR01                                   | Streaming    | Redpanda Streaming -harjoitus. Kafka producer ja consumer toteutetaan Pythonilla.                    | 10                    | B        |
| ORC01                                   | Airflow      | Orkestraatioharjoitus, jossa käytetään Airflow-työkalua.                                             | 10                    | B        |
| ORC02                                   | Dagster      | Orkestraatioharjoitus, jossa käytetään Dagster-työkalua.                                             | 10                    | B        |
| KIM01                                   | Kimball      | Dimensionaalisen mallintamisen harjoitus.                                                            | 10                    | B        |
| CUB01                                   | OLAP-kuutio  | OLAP-kuutio rakentaminen Marimo Notebookissa.                                                        | 10                    | B        |
| BIG01                                   | Juna-alusta  | Digitrafficin Rautatieliikennedataa hyödyntävä End-to-End OLAP-alusta.                               | 40                    | B        |

??? question "Mitä tarkoittaa Deadline?"

    Harjoitukset on jaettu ryhmiin A ja B deadlinen mukaan. Ryhmän A harjoitukset palautetaan noin kurssin puolivälissä, ryhmän B harjoitukset viimeistään kurssin lopuksi. Näin varmistetaan, että opiskelijat saavat palautetta harjoituksistaan ennen kuin kurssi päättyy, ja ettei työ kasaannu viimeiselle viikolle.

??? question "Entä lopullinen arvosana?"

    Saamasi arvosana riippuu kokonaispisteitä, jotka olet kerännyt eri harjoitukset palauttamalla. Arvosanat määräytyvät seuraavasti:

    | Pisteet | Arvosana |
    | ------- | -------- |
    | 0–59    | Hylätty  |
    | 60–69   | 1        |
    | 70–79   | 2        |
    | 80–89   | 3        |
    | 90–99   | 4        |
    | 100—    | 5        |

    Moodlen Gradebook:ssa tämä lasketaan seuraavalla kaavalla, jossa `TOTAL` edustaa yllä esiteltyjen tehtävien kumulatiivista pistemäärää:

    ```
    =min(5, max(0, floor(([[TOTAL]] - 50) / 10)))
    ```

## Faktavirheet

Mikäli oppimateriaali sisältää virheellistä tietoa, tee jompi kumpi:

* Forkkaa GitHubin repository ja tarjoa Pull Request, joka sisältää korjausehdotukset.
* Ota yhteyttä ylläpitoon ja esittele virheellisen tiedon korjaus.