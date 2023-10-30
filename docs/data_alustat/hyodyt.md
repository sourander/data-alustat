Datan tallentaminen maksaa. Mikäli yritys siirtää tiedot operatiivisista kannoista analyyttisiin kantoihin (warehouse, lake, lakehouse), datan säilytyksen ja käsittelyn kulut luonnollisesti kasvavat. Mikäli dataa säilötään mikropalveluista vastaavien tiimien omien kantojen ulkopuolella, tästä aiheutuu myös uusia tarpeita kommunikaatiolle. Kommunikaatio on lähtökohtaisesti vaikeaa. Lisäksi jokainen uusi järjestelmä, jossa tietoa säilötään, lisää myös tietoturvariskiä. Lisääntyneet kulut eivät välttämättä edes kuittaa itseään: vain murto-osa (~15%) big data projekteista onnistuu tuottamaan selkeää bisnesarvoa tavoitteiden mukaisesti. Tästä herää kysymys: ==mitä hyötyä siitä on businekselle?==

## Hyödyt

Hyötyjä siihen tilanteeseen verrattuna, että tietoalustaa ei ole, ovat muiden muassa alla listatut. Vaihtoehto tietoalustan puuttumiselle on Excel-tiedostot ja muut kopiot tai dumpit.

* **Datan laadun paraneminen.** Loppukäyttäjät uskaltavat luottaa dataan, joka on tallennettu yhteen paikkaan. Tämä vähentää datan duplikaatiota ja parantaa datan laatua.
* **Tietosuojan paraneminen**. Datan hallinta on keskitettyä, jolloin tietosuojan hallinta on helpompaa.
* **Datakulttuurin kehittyminen**. Datakulttuuri kehittyy, kun dataa käytetään liiketoiminnan päätöksenteossa ja työnteon arjessa.
* **Siilojen ja heimotietouden purku**. Data-alustan käyttöönotto edellyttää yhteistyötä eri tiimien välillä. Tämä purkaa siiloja ja heimotietoutta.

!!! tip

    Jos termi heimostietous on vieras, tutustu asiaan. Englanniksi termi on *tribal knowledge*.

!!! question "Tehtävä"

    Etsi esimerkkejä yrityksistä, jotka ovat hyötyneet big data -projekteista. Mitä hyötyjä he ovat saaneet? Löydätkö sekä sellaisia esimerkkejä, jotka näkyvät suoraan asiakkaalle (esim. recommendation engine) että sellaisia, jotka näkyvät yrityksen sisällä (esim. työntekjöiden tyytyväisyyden parantaminen.)

## Projektien kaatumisen syyt

Big data -projektien kaatuminen on yleinen aihe esimerkiksi Medium-sivuston postauksissa, LinkedIn-alustan keskusteluissa sekä konsulttitalojen julkaisemissa artikkeleissa. Joitakin yleisiä, kirjoituksesta toiseen toistuvia syitä ovat:

* **Datan laadun tärkeys aliarvioidaan.** Garbage in, gargabe out.
* **Tavoitteiden hataruus.** Tavoitteet ovat vaikeasti mitattavissa, eivät ole liiketoiminnan kannalta merkittäviä tai realistia, tai tavoitteet ovat liian kauaskantoisia.
* **Projektin väärä omistajuus.** Big data -alusta nähdään teknologisena ratkaisuna, ei liiketoiminnallisena ratkaisuna. Mikäli data-alusta on "IT-hanke", se kaatuu lähes varmasti.
* **Raakadatan kerääminen ilman tarkoitusta.** Pelkällä datan keräämisellä ei ole liiketoiminnallista arvoa. Tässä on kenties taustalla FOMO (Fear of Missing Out) eli *"koska muutkin, niin meidänkin pitää."*
* **Kulttuurin luominen epäonnistuu.** Jos työntekijät jatkavat työskentelyä vanhoilla tavoilla, ei datakulttuuria synny. Data-alustasta ei ole hyötyä, jos sitä ei käytetä.
* **Oikea osaaminen puuttuu**. Yritykseltä puuttuu oikea osaaminen, työntekijöiltä puuttuu resurssit kehittää omaa osaamistaan, ja kenties rekryssä etsitään yksisarvisia.

!!! tip

    Jos data-alustan yksi hyöty (ja samalla menestyksen vaatimus) on datakulttuurin ja datanlukutaidon kehittyminen, mieti, kuinka data-alustan luomisen voi ulkoistaa. Millainen ulkoistetun projektin tulee olla, jotta se kehittää tilaajayrityksen sisäistä datakulttuuria?

    Vihje: vertaa tätä tietoturvan kulttuuriin. Tietoturva on yrityksen kaikkien työntekijöiden vastuulla. Yksi tiimi ei voi "hoitaa hommaa". Tietoturva on myös jatkuvaa työtä, ei yksittäinen projekti.

## Onnistumisen edellytykset

Yllä mainitut syyt projektien kaatumiselle voi käännetään myös onnistumisen edellytyksiksi. Mikäli oletetaan aiempi lista riittävän kattavaksi, niin tarkistuslista onnistumisen tielle voisi olla:

### Bisnestavoitteet

Data-alustalle asetetut tavoitteet ovat bisnesvetoisia ja mitattavissa. Tavoitteet ovat realistisia ja niiden saavuttamiseksi on olemassa suunnitelma. Ensimmäinen tavoite on matalan kynnyksen ==Proof of Concept== (lyhyesti PoC), jonka tulisi tuottaa ensimmäisiä realistisia tuloksia hyvinkin lyhyessä ajassa (kuukausi tai kaksi). Näin data-alustajan kasaajat ja liiketoiminnan edustajat saavat käsiinsä jotain konkreettista, jonka avulla on helppo puhua samoista asioista.

Projektia käynnistäessä on tärkeää, että teknologia tai lopputuote ei ole itseisarvo, vaan tavoitteet ovat liiketoiminnallisia. Ensimmäisissä palavereissa on hyvä vältellä turhia muotisanoja (eng. buzzword, kuten LLM, NLP, semantic layer, data vault) sekä nimettyjä teknologioita tai palveluntarjoajia (Apache Spark, Apache Kafka, Snowflake, AWS EMR, jne.) Myöskään pelkkään lopputulokseen, kuten "me halutaan tämmönen interaktiivinen Dashboard", ei kannata takertua. Ethän siis tee datatiedettä ihan vain datatieteen takia - tai siksi että muutkin - vaan tarpeen määrittelemänä.

Tavoitteita laatiessa on tärkeää tavoittaa ja kuulla kaikkia niitä, joita **projektissa ratkaistava ongelma koskettaa**. Ota heidät mukaan kokoukseen. Kun tiedät, ketä ongelma koskettaa, voit kokeilla seuraavia:

1. Rautakoodaa tai tee muutoin itse PoC, joka ratkaisee ongelman.
2. Kysy loppukäyttäjältä, onko ratkaisu käyttökelpoinen, ja kuinka se muuttaisi hänen toimintaansa.

!!! tip

    Kuvittele, että olet myymäläpäällikkö Kajaanin Tilpehööri Oy:ssä. Aulassanne on Hype2Reality Oy -konsulttitalon toteuttavat asiakastyytyväisyysmittaus, joka perustuu viiteen hymiöön: :rage:, :frowning:, :neutral_face:, :slight_smile:, :laughing:, edustaen arvosanoja 1-5.

    Vastauksien määrä ja hajonta pysyy samana, mutta keskiarvona tulokset ovat:

    Viikko 33: :slight_smile: (# 4) <br /> 
    Viikko 34: :neutral_face: (# 3)

    Kuinka korjaat tämän negatiivisen kehityssuunnan?

Jos loppukäyttäjä ei osaa kertoa, kuinka hänen edessään näkyvä ratkaisu muuttaisi hänen toimintaansa, on todennäköistä, että ratkaisu ei ole liiketoiminnallisesti merkittävä. Toki tietoalustan voi myös kasata *"If you build it, they will come"*-periaatteella, jossa luotetaan siihen, että mahdollisuus luo käyttötarkoituksia.

!!! question "Tehtävä"

    Keksi esimerkkejä ratkaisuista, jotka olisi mahdollista rakentaa ja jotka aiheuttavat Wow-efektin, mutta joista ei kaikesta hienoudesta huolimatta ole minkään sortin hyötyä.

    Pohjana voit käyttää: "Ammattikorkeakoulun sisäänkäyntiin tuodaan Info-näyttö, jossa näkyy xxxxxx. Siihen kerätään automaattisesti dataa yyyyyy ja zzzzzzz."

!!! question "Tehtävä"

    Tee yllä oleva uusiksi, mutta yritä löytää oikea ongelma, ja ratkaisu siihen. Mitä dataa käytät? Kuinka se kerätään?

### Osaamisen kehittäminen

Data-alustat ja niihin liittyvä ekosysteemi kehittyy nopeasti. Tämä tarkoittaa, että osaamisen kehittäminen on jatkuvaa, ja työntekijöiden etsiminen "5+ vuoden kokemuksella" on naiivia. Aiemmassa kohdassa mainitun PoC:n voi hyvin toteuttaa muutamalla eri työkalulla, jotta tulee kokeiltua, mitkä työkalut istuvat juuri tähän caseen parhaiten. On äärimmäisen todennäköistä, että kukaan yrityksen työntekijöistä (tai edes hakijoista) ei ole käyttänyt kaikkia työkaluja - jos mitään. Iteratiivinen kehitys, jossa työkaluja, käytänteitä, datakulttuuria, osaamista ja liiketoimintaa kehitetään yhdessä, on todennäköisesti paras tapa edetä. Osaamista ja jossain määrin valmiiksi koeponnistettuja ratkaisuita voi hankkia myös konsulttitaloilta, mutta tässäkin tapauksessa pitäisi onnistua varmistamaan, että oman talon datakulttuuri kehittyy.

### Kulttuurin muutos

Datavetoisen päätöksenteon kulttuurin luominen on pitkäjänteistä työtä. Kulttuurin muutos ei tapahdu yhdessä yössä, eikä se tapahdu, jos data-alustan käyttöönotto on pelkkä IT-projekti. Dataan tottuminen vaatii samankaltaista muutosta kuin digitalisaatio joitakin vuosia sitten.

!!! tip

    Mieti, kuinka onnistuneesti kivijalkakaupan digitalisaatioprojekti etenee, jos yrityksen johto tilaa IT-osastolta firmalle nettisivut. Verkkosivuilta löytyy kaupan yhteystiedot ja esitteen voi ladata Word-dokumenttina. Pohdi, miten tämä analogia istuu data-alustan käyttöönottoon ja datakulttuurin synnyttämiseen.

### Datan laatu

Ilman laadukasta dataa tietoalustan onnistumisen mahdollisuudet ovat todella heikot. Datan laatua ei voi myöskään kokonaisuudessaan ulkoistaa data engineereille, jotka saattavat tiedon Bronze tai Silver tasolle. Osa vastuusta on dataa tuottavilla tiimeillä. Vastuu jakautuu myös liiketoiminnan suuntaan: he määrittelevät bisnestermistön. Kaiken perustana pitäisi olla ==datastrategia==, joka läpäisee koko yrityksen toiminnan.

!!! warning

    Universaalia, kaikki datan laadun ongelmat poistavaa, järjestelmää ei ole olemassa. Datan laatu ei siis hoidu sillä, että "ostaa kerralla hyvän tietoalustan."
    
    Datan laatu on aina liiketoiminnan ja datan kontekstista riippuvaista. On täysin mahdollista, että datan laatuun liittyvän ongelman ratkaisemiseksi tarvitaan samaan neuvotteluun ei-tekninen liiketoiminnan edustaja, data engineer, substanssiosaaja (esim. tutkija) ja mikropalvelun lead developer.
