TODO

## ACID

| Kirjain | Termi      | Kuvaus                                                                                                                                                                                                                                                  |
| ------- | ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| A       | Atomisuus  | Transaktiot suoritetaan atomisesti eli kokonaan tai ei ollenkaan.  Vian ilmaantuessa kirjoitusoperaatio peruutetaan.                                                                                                                                    |
| C       | Eheys      | Monimerkityksellinen ja siten ongelmallinen termi, joka viittaa muun muassa siihen, ovatko primääri- ja viiteavaimet pätevässä tilassa, ja siihen, että vastataanko kyselyihin aina tuoreimmalla datalla vai onko käytössä eventual consistency -malli. |
| I       | Eristyvyys | Kirjoitusoperaatiot suoritetaan siten, että ne tapahtuvat toisistaan erillään. Tyypillinen esimerkki on, että kaksi eri applikaatiota yrittävät muokata saman sarakkeen arvoa samalla hetkellä.                                                         |
| D       | Pysyvyys   | Onnistuneiden transaktioiden pysyvyys pyritään säilyttämään vikatilanteissa.                                                                                                                                                                            |

TODO: Esittele Delta Lake, Hudi, Iceberg, jotka mahdollista omalta osaltaan ACID:n object storageen.

## Ihmisluettavat vs. binääriset tiedostot

TODO: Esittele JSON Lines vs. Parquet tai ORC.

## Row vs Columnar

Esittele Parquet.

