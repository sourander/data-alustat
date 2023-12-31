Alla useita kurssin aikana käytettyjä termejä keskitetysti.

#### ACID

Neljän tekijän (atomisuus, eheys, eristyvyys ja pysyvyys) kokonaisuus. ACID-periaatteita noudattava kanta on ihanteellisessa maailmassa virheenkestävä jopa vikatilanteissa tai suorittaessa useita kilpailevia kirjoitusoperaatiota yhtä aikaa

#### CDC

Muutostiedon kaappaus (englanniksi change data capture). Prosessi, jossa tunnistetaan lähdekannan muutokset joko lähes reaaliaikaisesti tai ajastetusti.

#### Data Lakehouse

Arkkitehtuuri, joka yhdistää tietovaraston (eng. data warehouse) sekä tietoaltaan (eng. data lake) toiminnallisuuksia

#### Delta Lake

Avoimen lähdekoodin tiedontallennuksen kerros, joka lisää ACID-periaatteiden mukaisen toiminnallisuuden tiedostopohjaiseen tietoaltaaseen.

#### ELT

Alemman akronyymin eli ETL:n mukaelma, jossa tiedon muokkaus ja tiedon lataus kohdekantaan suoritetaan käänteisessä järjestyksessä. Tietomallia muokataan vasta määränpäässä, joka voi olla esimerkiksi tietovarasto tai jokin moderni tietoalusta

#### ETL

Prosessi, jossa tieto kaapataan lähdekannasta (extract), muunnetaan  kohdekannan tietomallin mukaiseksi (transform) ja ladataan kohdekantaan (load), joka on tyypillisesti tietovarasto

#### Modern Data Stack

Modernin vastakohtana on legacy, jolla viitataan menneiden vuosikymmenien monoliittisiin tietoalustoihin. Modern Data Stack ei ole yksittäisen palveluntarjoajan myymä tietoalusta vaan kokoelma erilaisia työkaluja, joiden avulla dataa voidaan käsitellä ja analysoida. Tyypillisesti moderni data stack koostuu tietoaltaasta, tietovarastosta, tietovirrasta ja BI-työkalusta. Siihen kuuluu tyypillisesti myös orkestrointityökaluja, infraan hallitsemiseen liittyviä työkaluja sekä datakatalogi. Moderni data stack on usein pilvipohjainen. Yksittäiset komponentit ovat modulaarisia, joten niitä voidaan päivittää tarpeen mukaan. Sekä pilvipohjaisuus että modulaarisuus mahdollistavat skaalautuvuuden, joka on yksi modernin data stackin keskeisistä ominaisuuksista.

#### OLAP

Online Analytics Processing. Tietovarastojen tiedon hakemisen ja tallentamisen malli.

#### OLTP

Online Transaction Processing. Operatiivisten kantojen tiedon hakemisen ja tallentamisen malli.

#### Tietoallas

Suurten tietomassojen tallennukseen tarkoitettu arkkitehtuuri. Mahdollistaa strukturoimattoman tiedon tallennuksen ja käsittelyn. Tallennuskapasiteetti on usein hajautettua ja tiedostopohjaista, mikä mahdollistaa edullisen horisontaalisen skaalauksen. Englanniksi data lake.

#### Tietovarasto

Järjestelmä, johon tuodaan ETL tai ELT-prosessin avulla useista eri tietokannoista tietoa, jotta hajallaan olevaa dataa voidaan käsitellä ja analysoida keskitetysti. Englanniksi data warehouse
