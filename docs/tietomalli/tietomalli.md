Tietomallilla tarkoitetaan sitä, kuinka tieto on tallennettu tauluihin ja miten taulut liittyvät toisiinsa. Tässä kurssimateriaalissa on jo esitelty kaksi tietomallia: One Big Table ja normalisoitui (1-3NF). Kertaa nämä [Historia](../data_alustat/historia.md)-osiossa.



## Tietokantojen ja tietovarastojen tietomallit

Operatiivisten tuotantokantojen tietomalli rakennetaan sovelluksen tarpeen mukaan, joten kahden eri tuotantokannan tietomallit voivat olla keskenään hyvinkin poikkeavia. Tyypillisin tietomalli on relaatiotietokanta, jossa data on tallennettu tauluihin ja tietokanta noudattaa vähintään ensimmäisen asteen normaalimuotoa, mutta käytössä on jatkuvasti enenevissä määrin muita tietokantatyyppejä, kuten dokumentti- tai graafitietokantoja. Näiden relaatiotietokannoista poikkeavien kantatyyppien yleisnimenä on NoSQL. Eräänlainen hybridi on kanta, joka on tavallinen relaatiokanta, mutta applikaation ymmärtämä data upotetaan BLOB tai JSON -kenttään, jolloin skeema voi muuttua ajan yli, vaikka itse taulun skeema pysyisi samana.

TODO: SQL-esimerkki vielä kertauksen vuoksi

TODO: noSQL-esimerkki

Tietovarastojen tietomallien diversiteetti on sen sijaan huomattavasti kapeampi; tietovarastot noudattavat lähes poikkeuksetta tähtiskeemaa ja mukailevat Kimballin dimensiomallia, ainakin ulommilla kerroksilla. Tietovarastojen tietomalli rakennetaan yrityksen prosessien ja tietovaraston käyttäjien tarpeiden mukaan. Näin ollen operatiivisten kantojen ja ietovarastojen tietomallit rakennetaan hyvin erilaisista lähtökohdista; kummassakin on sama informaatio, mutta eri tavoin järjesteltynä.

## Normalisoitu vai ei

Tietovarasto ei hyödy normalisoinnista, ainakaan samoissa määrin kuin operatiivinen kanta, koska tietovaraston käyttötarkoitus ei ole palvella sovellusta tai käyttäjää nopeilla rivikohtaisilla luku- ja kirjoitusoperaatioilla. Tietovarastoon tuotu data muokataan usein toisenlaiseen tietomalliin: tähtiskeemaan. On mainittavan arvoista, että ennen tähtiskeemaa tietovarastossa voi olla esimerkiksi Data Vault 2.0 -tyylinen kerros, joka on jossain määrin hybridi normalisoidun ja tähtimallin välillä, ja sisältää muun muassa rivien historiatietoa.

Tähtiskeema koostuu fakta- ja dimensiotauluista. Faktataulut mallintavat yrityksen prosesseja. Faktataulu on nimeltään esimerkiksi `fact_sales`, jonka yksittäinen rivi on myyntioperaatio tai kuittirivi sisältäen tuotteen senhetkisen hinnan ja alennushinnan. Tähän liittyvä dimensiotaulu on esimerkiksi `dim_product`, jonka  yksittäinen rivi kuvastaa tuotetta, tai dim_customer, jonka yksittäinen rivi kuvastaa asiakasta.  Yhteen faktatauluun liittyy monta dimensiota. Näin syntyy tähti, jossa keskiössä on faktataulu, ja  tähden sarakkeet ovat dimensiotauluja. Tähtiskeema sisältää redundanttia, toistuvaa tietoa, joten taulut ovat fyysisesti (tavuina mitattuna) huomattavasti suurempia kuin mitä niiden alkuperäiset tietolähteet eli normalisoidut relaatiotaulut. Dimensiotaulut ovat usein myös hyvin leveitä:  niissä voi kymmeniä tai satoja sarakkeita. Denormalisoitu dimensiotaulu saa ja voi sisältää toisistaan riippuvia sarakkeita, kuten: syntymäpäivä ja ikä; pituus, leveys ja pinta-ala; päivämäärä ja viikonpäivä. Normalisoidussa kannassa nämä tietoparit nähtäisiin redundantteina, koska niillä on riippuvuus; yhden arvon voi laskea toisesta.

## Avaintyypit

### Tekninen näkökulma

Pääavain ja viiteavain ovat siinä mielessä teknisen tason avaimia, että ne eivät välttämättä ole liiketoimintalähtöisiä. Niitä tarvitaan relaatioiden luomiseen ja tietojen yhdistämiseen, mutta ne eivät välttämättä kerro mitään liiketoiminnasta tai sen prosesseista. Teknisen tason avaimet ovat yleensä yksinkertaisia ja helposti ymmärrettäviä, mutta ne eivät *välttämättä* ole liiketoimintalähtöisiä.

Aavain voi koostua useammasta kuin yhdestä attribuutista. Tällöin puhutaan yhdistelmäavaimesta (composite key). 

#### Pääavain (primary key)

Kolme seuraavaa termiä on hyvä tuntea. Ne voi käsittää Venn-diagrammin tyyliin siten, että ylempi termi on aina seuraavan kattokäsite:

* Yliavain (superkey)
* Kandidaattiavain (candidate key)
* Pääavain (primary key)

Yliavain on yksi tai useampi kenttä, jotka yksilöivät tietueen taulussa. Kandidaattiavain on minimaalinen avain, eli suppein yhdistelmäavain, joka yhä yksilöi tietueen taulussa. Kandidaattiavaimista voidaan valita yksi avain, joka on taulun pääavain (primary key). Pääavaimen arvoa ei saa muuttaa eikä se saa sisältää tietoa, joka voi muuttua. Pääarvo ei myöskään voi olla NULL. [^grokking_relational]

#### Viiteavain (foreign key)

Viiteavain (foreign key) on attribuutti tai attribuuttien joukko, joka viittaa toisen taulun pääavaimeen. Viiteavain luo suhteen kahden taulun välille ja mahdollistaa tietojen yhdistämisen.

### Liiketoiminnan näkökulma

Pääavain ja viiteavain ovat teknisen tason avaimia, kun taas liiketoimintalähtöiset avaimet ovat liiketoimintalähtöisiä. Taulun tasolla ne voivat olla yhä samoja avaimia: primääriavain voi esimerkiksi olla sijaisavain tai luonnollinen avain. Jako on nimenomaan se, että onko *katsojan näkökulmasta* avain luonnollinen vai ei.

#### Sijaisavain (surrogate)

Sijaisavain on sellainen avain, jolla ei (alunperin) ole liiketoiminnallista tai reaalimaailman merkitystä. Tyypillisesti se on SQL-lausekkeella `AUTO_INCREMENT` tai `SERIAL` luotu kokonaisluku, joka on uniikki ja kasvaa automaattisesti aina kun tauluun lisätään uusi rivi. Alla simppeli esimerkki

| this_is_a_surrogate_key | name | age |
| ----------------------- | ---- | --- |
| 1                       | John | 25  |
| 2                       | Jane | 30  |
| 3                       | Jack | 35  |

#### Luonnollinen (natural) avain

Luonnollinen avain on yksi tai useampi attribuutti, joka yksilöi tietueen taulussa ja jolla on **liiketoiminnallinen merkitys**.

Luonnollisen avaimen (natural key) määritelmä riippuu hieman siitä, käsitelläänkö tietoa lähdejärjestelmän vai tietovaraston näkökulmasta **Tiukan määritelmän** mukaan luonnollinen avain esiintyy järjestelmän ulkopuolella sinällään, ja sillä on sisäänrakennettu merkitys. Esimerkkejä ovat henkilötunnus, kirjan ISBN, digitaalisen lähteen DOI, laitteen sarjanumero, sähköpostiosoite tai kojelaudalta löytyvä VIN. Jopa (firstname, surname, birthdate) -yhdistelmäavainta voidaan pitää luonnollisena avaimena, joskin riski on, että se ei ole uniikki. [^dataquality] 

Tietovaraston näkökulmasta voidaan käyttää **löyhempää määritelmää** siten, että esimerkiksi autoinkrementin tai satunnaisuuden avulla luotu **CustomerID** voi olla ==tietovaraston silmin== luonnollinen avain, *vaikka* se onkin keinotekoisesti luotu, ja lähdejärjestelmän näkökulmasta aivan selkeä sijaisavain. Tietovaraston näkökulmasta sillä on kuitenkin jo selkeä merkitys: se edustaa yksilöllisesti tunnistettavaa asiakasta tosimaailmassa eli siinä järjestelmässä, mistä tieto on haettu. Käyttäjä `123` on sama käyttäjä `123` nyt ja aina. Jos tietoa haetaan useista järjestelmistä, täytyy kuitenkin huomioida, että eri järjestelmissä voi olla päällekäisiä ID-numeroita.

Tietovarastossa voidaan luoda uusia surrogaattiavaimia jos sille on tarvetta. Yksi tarve on *hash surrogate key*, joka luodaan esimerkiksi silloin, jos lähdetaulun primääriavain koostuu useista kentistä, ja tietovarasto hyötyy yksittäisestä uniikista kentästä (esimerkiksi CDC-prosessissa). Se voidaan laskea myös bisnesavaimesta Data Vault -mallinnuksen yhteydessä (ks. alta).
 
#### Bisnesavain

Jos kahdesta eri järjestelmästä poimitut tietueet voidaan yhdistää valittujen attribuuttien avulla, voimme puhua *business key* -kentistä. Tällöin kentät ovat luonnollisesti *luonnollisia* avaimia. [^scalable_dw_with_data_vault]

**Esimerkki**: System A:n käyttäjä `123` ja System B:n käyttäjä `0000325` ovat sama henkilö, jotka tunnistetaan kentillä: `(firstname, surname, birthdate)`. Tähän termiin törmäät Data Vault -mallinnuksessa ja Master Data Management -yhteyksissä.

Parempi ratkaisu on luonnollisesti keskitetty käyttäjähallinta, SSO (single sign-on) tai jokin muu tapa, jolla käyttäjät tunnistetaan ja hallitaan keskitetysti. Tällöin käyttäjät voivat käyttää useita järjestelmiä yhdellä ja samalla tunnuksella, ja käyttäjätiedot ovat aina ajan tasalla, ilman että niitä tarvitsee synkronoida eri järjestelmien välillä. GDPR:n kannalta on ongelmallista, jos et tiedä, keitä sinun asiakkaasi ovat, tai jos päättelet omatoimisesti, kuka asiakas on yhdistelemällä useiden järjestelmien tietoja kertomatta asiakkaalle tästä prosessista.


## Yleisimmät mallinnusmenetelmät

### Inmon

TODO: Selitä.

### Kimball

TODO: Selitä.

### Data Vault

TODO: Selitä.


[^grokking_relational]: Hao, Q. % Tsikerdekis, M. *Grokking Relational Database Design*. Manning Publications. 2025.

[^dataquality]: Olson, J. *Data Quality*. Morgan Kaufmann. 2003.

[^scalable_dw_with_data_vault]: Linstedt, D. & Olschimke, M. *Building a Scalable Data Warehouse with Data Vault 2.0*. Morgan Kaufmann. 2015.
