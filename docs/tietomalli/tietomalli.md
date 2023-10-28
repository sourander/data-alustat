Tietomallilla tarkoitetaan sitä, kuinka tieto on tallennettu tauluihin ja miten taulut liittyvät toisiinsa. Tässä kurssimateriaalissa on jo esitelty kaksi tietomallia: One Big Table ja normalisoitui (1-3NF). Kertaa nämä [Historia](../data_alustat/historia.md)-osiossa.



## Tietokantojen ja tietovarastojen tietomallit

Operatiivisten tuotantokantojen tietomalli rakennetaan sovelluksen tarpeen mukaan, joten kahden eri tuotantokannan tietomallit voivat olla keskenään hyvinkin poikkeavia. Tyypillisin tietomalli on relaatiotietokanta, jossa data on tallennettu tauluihin ja tietokanta noudattaa vähintään ensimmäisen asteen normaalimuotoa, mutta käytössä on jatkuvasti enenevissä määrin muita tietokantatyyppejä, kuten dokumentti- tai graafitietokantoja. Näiden relaatiotietokannoista poikkeavien kantatyyppien yleisnimenä on NoSQL. Eräänlainen hybridi on kanta, joka on tavallinen relaatiokanta, mutta applikaation ymmärtämä data upotetaan BLOB tai JSON -kenttään, jolloin skeema voi muuttua ajan yli, vaikka itse taulun skeema pysyisi samana.

TODO: SQL-esimerkki vielä kertauksen vuoksi

TODO: noSQL-esimerkki

Tietovarastojen tietomallien diversiteetti on sen sijaan huomattavasti kapeampi; tietovarastot noudattavat lähes poikkeuksetta tähtiskeemaa ja mukailevat Kimballin dimensiomallia, ainakin ulommilla kerroksilla. Tietovarastojen tietomalli rakennetaan yrityksen prosessien ja tietovaraston käyttäjien tarpeiden mukaan. Näin ollen operatiivisten kantojen ja ietovarastojen tietomallit rakennetaan hyvin erilaisista lähtökohdista; kummassakin on sama informaatio, mutta eri tavoin järjesteltynä.



## Normalisoitu vai ei

Tietovarasto ei hyödy normalisoinnista, ainakaan samoissa määrin kuin operatiivinen kanta, koska tietovaraston käyttötarkoitus ei ole palvella sovellusta tai käyttäjää nopeilla rivikohtaisilla luku- ja kirjoitusoperaatioilla. Tietovarastoon tuotu data muokataan usein toisenlaiseen tietomalliin: tähtiskeemaan. On mainittavan arvoista, että ennen tähtiskeemaa tietovarastossa voi olla esimerkiksi Data Vault 2.0 -tyylinen kerros, joka on jossain määrin hybridi normalisoidun ja tähtimallin välillä, ja sisältää muun muassa rivien historiatietoa.

Tähtiskeema koostuu fakta- ja dimensiotauluista. Faktataulut mallintavat yrityksen prosesseja. Faktataulu on nimeltään esimerkiksi  fact_sales, jonka yksittäinen rivi on myyntioperaatio tai kuittirivi sisältäen tuotteen senhetkisen hinnan ja alennushinnan. Tähän liittyvä dimensiotaulu on esimerkiksi dim_product, jonka  yksittäinen rivi kuvastaa tuotetta, tai dim_customer, jonka yksittäinen rivi kuvastaa asiakasta.  Yhteen faktatauluun liittyy monta dimensiota. Näin syntyy tähti, jossa keskiössä on faktataulu, ja  tähden sarakkeet ovat dimensiotauluja. Tähtiskeema sisältää redundanttia, toistuvaa tietoa, joten taulut ovat fyysisesti (tavuina mitattuna) huomattavasti suurempia kuin mitä niiden alkuperäiset tietolähteet eli normalisoidut relaatiotaulut. Dimensiotaulut ovat usein myös hyvin leveitä:  niissä voi kymmeniä tai satoja sarakkeita. Denormalisoitu dimensiotaulu saa ja voi sisältää toisistaan riippuvia sarakkeita, kuten: syntymäpäivä ja ikä; pituus, leveys ja pinta-ala; päivämäärä ja  viikonpäivä. Normalisoidussa kannassa nämä tietoparit nähtäisiin redundantteina, koska niillä on riippuvuus; yhden arvon voi laskea toisesta.

## Dimensiomalli

TODO: Selitä tähtimalli ja dimensiomalli Kimballin tyyliin.