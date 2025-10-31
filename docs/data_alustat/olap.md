# OLAP vs OLTP

## Perusteet

Tietokannat voidaan jakaa kahteen luokkaan käyttötarkoituksen mukaan: operatiivisiin kantoihin ja tietovarastoihin (eng. data warehouse). Operatiiviset transaktiokannat käsittelevät tilauksia, varauksia, ottoja ja muita tilamuutoksia pääasiassa rivi errallaan. Nämä tuotannon kannalta kriittiset kannat eivät tyypillisesti tallenna historiadataa vaan edustavat datan nykytilannetta. Tietovarastot toimivat hyvin päinvastaisesti: tietokannan haut käsittelevät useita tuhansia tai miljoonia rivejä kerrallaan, ja haettu tieto edustaa esimerkiksi keskimääräistä varausmäärää per tietty asiakassektori. Näille tiedon käsittelyn malleille on vakiintuneet lyhenteet: tietovarastokäyttöä vastaa lyhenne OLAP (Online Analytics Processing), ja operatiivisten kantojen käyttöä OLTP (Online Transaction Processing). Mallien eroavaisuudet on esitelty alla olevassa taulukossa.

![olap-varastosofta](../images/olap-varastosofta.svg)

**Kuvio 1**: *Eri applikaatioiden data virtaa usein eri kantoihin. Data saatetaan kerätysti yhteen tietovarastoon, jotta analyytikko, ylin johto, tai joku muu yrityksen sisäinen taho voi sitä tarkastella.*

|                       | OLTP                                                                                                                                       | OLAP                                                                                                             |
| --------------------- | ------------------------------------------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------- |
| Toiminta-ajatus       | Tallentaa ja palauttaa tietueita yrityksen päivittäisen toiminnan takaamiseksi.                                                            | Tukee päätöksentekoa ja mahdollistaa datan louhinnan ja analysoinnin.                                            |
| Tyypillinen haku      | Loppukäyttäjä sovelluksen käyttöliittymän kautta. Pieni määrä rivejä, jotka löydetään primääriavaimella tai muulla indeksoidulla kentällä. | Data-analyytikko SQL-kyselyllä. Usein aggregoitua dataa, jota on suodatettu tai ryhmitelty sarakearvojen mukaan. |
| Tyypillinen kirjoitus | Yksittäiseen riviin kohdistuva  kirjoitus tai muutos.                                                                                      | Suuri erä rivejä ajastetussa ETL-prosessissa.                                                                    |
| Kyselyn laajuus       | Yksittäinen tietue eli rivi, joka haetaan sen id:tä tai avainta vasten                                                                     | Useita tietueita tai niiden aggregaatti (esimerkiski keskiarvo).                                                 |
| Viive                 | Nano- tai millisekunteja.                                                                                                                  | Sekunteja tai tunteja.                                                                                           |

Erilaiset tiedon hakisen ja tallentamisen tarpeet vaikuttavat niin teknisiin ratkaisuihin kuin tiedon loogisen malliin.
