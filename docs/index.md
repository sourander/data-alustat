# Tervetuloa kurssille

!!! warning

    Tämä materiaali on refaktoroinnin alla. Uusi toteutus käynnistyy syksyllä 2026.

## Kenelle?

Oppimateriaali on tarkoitettu Kajaanin Ammattikorkeakoulun toisen vuoden IT-alan opiskelijoille. Materiaalin ymmärtämisessä on eduksi, että ymmärtää SQL perusteet, tietokantojen perusteet, osaa Pythonia vähintään perusteiden verran, ja on käyttänyt Dockeria.

## Pisteytysmalli

Kurssi koostuu harjoituksista, jotka kukin arvostellaan [Arviointityökalun](https://arviointi.munpaas.com/) kriteeristöllä nimeltään ==videoitu demo==. Videot palautetaan Reppu-alustaan linkkeinä. 

Kustakin videosta saa `raakapistemäärän`, joka on välillä 0–5. Tämä pistemäärä kerrotaan harjoitukselle määritellyllä kertoimella, jolloin saadaan harjoituksesta saatavat kurssipisteet. Alla oleva dummy-koodi toivon mukaan selventää logiikkaa:

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

Kertoimet on valittu siten, että *max pisteet* kuvastaa *keskimääräisiä* opiskelijan tehtävään kuluttamia tunteja. Kerään tilastotietoa harjoituksiin käyttämästänne ajasta, jotta voin optimoida tätä pisteytysmallia tulevaisuudessa. Sinun tulee siis kertoa tehtävään käyttämäsi aika videodemossa.

Alla listattuna kurssin harjoitukset (==HUOM! TÄMÄ ON YHÄ TODO eli WORK IN PROGRESS==):

| ID    | Otsikko      | Harjoituksen lyhyt kuvaus                                                                            | Max pisteet (≈tunnit) |
| ----- | ------------ | ---------------------------------------------------------------------------------------------------- | --------------------- |
| CSC01 | cPouta       | Metaharjoitus, jossa opit ajamaan seuraavat harjoitukset oman koneen sijasta CSC cPouta -palvelussa. | 5                     |
| LAK01 | Duck Lake    | Duck Lake -harjoitus, jossa kasataan minimaalinen Data Lakehouse.                                    | 10                    |
| SRK01 | Apache Spark | Apache Spark -tutuksi lokaalin `pyspark`:n avulla.                                                   | 5                     |
| SRK02 | Spark Delta  | Apache Spark -harjoitus, jossa käytetään Delta Lakea.                                                | 10                    |
| CDC01 | Mealie CDC   | Mealie-sovelluksen CDC-tiedonkeruuharjoitus.                                                         | 20                    |
| STR01 | Streaming    | Redpanda Streaming -harjoitus. Kafka producer ja consumer toteutetaan Pythonilla.                    | 15                    |
| AUR01 | Airflow      | Orkestraatioharjoitus, jossa käytetään Airflow-työkalua.                                             | 10                    |
| KIM01 | Kimball      | Dimensionaalisen mallintamisen harjoitus.                                                            | 10                    |
| CUB01 | OLAP-kuutio  | OLAP-kuutio rakentaminen Marimo Notebookissa.                                                        | 5                     |
| BIG01 | Juna-alusta  | Digitrafficin Rautatieliikennedataa hyödyntävä End-to-End OLAP-alusta.                               | 30                    |

Saamasi arvosana riippuu kokonaispisteitä, jotka olet kerännyt eri harjoitukset palauttamalla. Arvosanat määräytyvät seuraavasti:

| Pisteet | Arvosana |
| ------- | -------- |
| 0–59    | Hylätty  |
| 60–69   | 1        |
| 70–79   | 2        |
| 80–89   | 3        |
| 90–99   | 4        |
| 100—    | 5        |


## Faktavirheet

Mikäli oppimateriaali sisältää virheellistä tietoa, tee jompi kumpi:

* Forkkaa GitHubin repository ja tarjoa Pull Request, joka sisältää korjausehdotukset.
* Ota yhteyttä ylläpitoon ja esittele virheellisen tiedon korjaus.