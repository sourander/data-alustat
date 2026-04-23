# Yleisohjeet

## Tehtävien rakenne

Tässä neuvotaan yleisohjeet, kuten:

* Valmiiksi annetut materiaalit, kuten mahdolliset Dockerfile-tiedostot, löytyvät repositorion juuresta alkaen `handout/`-hakemistosta.
* TODO: kirjoita ohjeet

!!! warning

    Käytäthän opettajan sinulle antamaasi repositoriota, jotta opettajalla on automaattisesti pääsy sinun työhösi. Varmista, että repositoryn juuressa on `README.md`-tiedosto, josta alkaen käyttäjä kuljetetaan eri ohjeiden äärelle. Jos dokumentaatio on repositorion punainen lanka, niin `README.md` on lankakerän pää. Siitä on hyvä linkittää kaikkiin ohjeisiin, jotta opettaja löytää ne helposti.

## Videon tekninen toteutus

### Jakaminen

Palautus hoidetaan lisäämällä linkki Reppu-palvelun palautuslaatikkoon. Linkin tulee olla luotu siten, että se toimii opettajan koneella. Pikaohje tähän on:

* **Jos YouTube**, valitse Visibility: *Unlisted*.
* **Jos OneDrive**, jaa linkkinä siten, että kaikilla linkin saaneilla (ainakin KamiT AD:ssa) on lupa nähdä se.

OneDrive-kohtaan on alla tarkempi ohje.

![](images/onedrive-share.png)

**Kuva** *Tiedoston jakaminen OneDrivessä vaatii Share-ominaisuuden käyttöä. Ethän siis kopioi URL:ia selaimen osoiteriviltä vaan käytä Share-ominaisuutta. Kohdasta Link Settings pitäisi löytyä kuvassa näkyvä menu.*

### Tallennus

Voit tallentaa videon esimerkiksi seuraavilla työkaluilla:

* [Microsoft Clipchamp](https://m365.cloud.microsoft/launch/Clipchamp/). Työkalu on käytettävissä suoraan selaimesta ja video tallentuu pilvipalveluun (KamiT OneDriveen).
* [OBS Studio](https://obsproject.com/). Avoimen lähdekoodin videotallennin ja striimaustyökalu. Tiedosto tallentuu lokaalisti todella hyvälaatuisena ja on täten helposti leikattavissa.

Jos haluat leikata videota, siihen soveltuu ainakin:

* [Microsoft Clipchamp](https://m365.cloud.microsoft/launch/Clipchamp/). Online-editori. En ole käyttänyt, mutta se vaikuttaa helpolta.
* [Shotcut](https://shotcut.org/). Avoimen lähdekoodin videoeditointiohjelma, joka on saatavilla useille alustoille.
* [DaVinci Resolve](https://www.blackmagicdesign.com/products/davinciresolve/). Tehokas videoeditointiohjelma, joka tarjoaa laajat ominaisuudet ilmaisversiossaan.

Suosittelen opiskelijoita jakamaan keskenään vinkkejä helpoista editoreista. Opettajalla on kokemusta pääasiassa Premierestä tai DaVinci Resolvea, jotka eivät ole aivan helpoimmasta päästä.



## Videon sisällölliset ohjeet

Palautat **lyhyt video**. Älä tee tunnin luentoa vaan ns. ==perjantaidemo==, jonka voisi esittää yrityksen perjantaisessa videopalaverisessiossa. Harjoituksesta riippuen 5-15 minuuttia on hyvä kesto. Videolla:

* Kerrot, kuinka monta tuntia käytit tehtävään. [^1]
* Selität sen osan terminologiasta, joka on oleellista ymmärtää, jotta ymmärtäisi, mitä harjoituksessa tapahtuu. Älä selitä opettajalle, ja ajattele, että *opettaja kyllä tietää*. Selitä kuin yleisössä olisi yrityksen toisten tiimien jäseniä, jotka eivät välttämättä täysin tiedä, mitä työkaluja ovat esimerkiksi Airflow, DuckDB tai Apache Spark.
* Esittelet harjoituksessa luodun kokonaisuuden.
    * Palvelu nostetaan pystyyn videolla (esim. `docker compose up`)
    * Palvelu health status on todistettu. Jos palveluun liittyy web-käyttöliittymä, näytä se. Jos ei, näytä vaikka `docker status`.
    * Palvelun tyypillinen toiminnallisuus näkyy videolla. Esimerkiksi siihen syötetään dataa, näytetään ja selitetään datan kulku järjestelmässä, ja selitetään, mitä ja miksi tulee tuloskseksi.

Huomaa, että tarkat ohjeet riippuvat harjoituksesta. Osa sinun osaamistasi on tunnistaa, mikä juuri kyseisessä tehtävässä on oleellista.

Opettaja arvioi työsi [Arviointityökalulla](https://arviointi.munpaas.com/). Valitse vetovalikosta *Videoitu demo*, ja voit kokeilla, kuinka arvioisit itse oman työsi.

[^1]: Tämä on tärkeää, jotta voin optimoida pisteytysmallia tulevaisuudessa. Et saa enempää tai vähempää pisteitä siitä, kuinka paljon käytit aikaa, mutta **arvosanasi laskee, jos et ilmoita käyttämääsi aikaa**.