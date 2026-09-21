# STR01: Redpanda Streaming

Redpanda on Kafka-yhteensopiva streaming-alusta. Opetuskäytössä Redpanda on helpompi ja kevyempi vaihtoehto Kafkaan verrattuna, ja se tarjoaa samat perusominaisuudet, kuten producerit, consumerit, ja topicit. Tässä harjoituksessa tutustutaan Redpandan perusteisiin ja toteutetaan yksinkertainen Kafka producer ja consumer Pythonilla. 

## Esivaatimukset

- Docker ja Docker Compose

## Valmistelut: Redpanda-ympäristö

### Luo Docker Compose -tiedosto

Luo projektikansioon `compose.yml` -tiedosto, jossa määritellään Redpanda-palvelu. Alla oleva tiedoston sisälty perustuu vahvasti Redpandan viralliseen [Start a Single Redpanda Broker with Redpanda Console in Docker](https://docs.redpanda.com/labs/docker-compose/single-broker/)-ohjeeseen.

Voit ladata tiedoston näin:

```bash
URL='https://docs.redpanda.com/labs/docker-compose/_attachments/single-broker/docker-compose.yml'
curl -o compose.yml $URL
```

??? info "Backup koodista alla"

    Jos alkuperäisen ohjeen versio ei ole enää saatavilla, voit kopioida leikepöydälle alta sisällön, jota opettaja käytti.

    ```yaml title="compose.yml"
    name: redpanda-quickstart-one-broker
    networks:
    redpanda_network:
        driver: bridge
    volumes:
    redpanda-0: null
    services:
    redpanda-0:
        command:
        - redpanda
        - start
        - --kafka-addr internal://0.0.0.0:9092,external://0.0.0.0:19092
        # Address the broker advertises to clients that connect to the Kafka API.
        # Use the internal addresses to connect to the Redpanda brokers'
        # from inside the same Docker network.
        # Use the external addresses to connect to the Redpanda brokers'
        # from outside the Docker network.
        - --advertise-kafka-addr internal://redpanda-0:9092,external://localhost:19092
        - --pandaproxy-addr internal://0.0.0.0:8082,external://0.0.0.0:18082
        # Address the broker advertises to clients that connect to the HTTP Proxy.
        - --advertise-pandaproxy-addr internal://redpanda-0:8082,external://localhost:18082
        - --schema-registry-addr internal://0.0.0.0:8081,external://0.0.0.0:18081
        # Redpanda brokers use the RPC API to communicate with each other internally.
        - --rpc-addr redpanda-0:33145
        - --advertise-rpc-addr redpanda-0:33145
        # Mode dev-container uses well-known configuration properties for development in containers.
        - --mode dev-container
        # Tells Seastar (the framework Redpanda uses under the hood) to use 1 core on the system.
        - --smp 1
        - --default-log-level=info
        image: docker.redpanda.com/redpandadata/redpanda:v26.1.9
        container_name: redpanda-0
        volumes:
        - redpanda-0:/var/lib/redpanda/data
        networks:
        - redpanda_network
        ports:
        - 18081:18081
        - 18082:18082
        - 19092:19092
        - 19644:9644
    console:
        container_name: redpanda-console
        image: docker.redpanda.com/redpandadata/console:v3.7.4
        networks:
        - redpanda_network
        entrypoint: /bin/sh
        command: -c 'echo "$$CONSOLE_CONFIG_FILE" > /tmp/config.yml; /app/console'
        environment:
        CONFIG_FILEPATH: /tmp/config.yml
        CONSOLE_CONFIG_FILE: |
            kafka:
            brokers: ["redpanda-0:9092"]
            schemaRegistry:
            enabled: true
            urls: ["http://redpanda-0:8081"]
            redpanda:
            adminApi:
                enabled: true
                urls: ["http://redpanda-0:9644"]
        ports:
        - 8080:8080
        depends_on:
        - redpanda-0
    ```

### Käynnistä Redpanda

Nyt voit ajaa tuttuun tapaa komennon:

```bash
docker compose up -d
```

Jos ajat komennon `docker ps`, huomaa seuraavan:

```
245286dd5b55   docker.redpanda.com/redpandadata/redpanda:v26.1.9   "/entrypoint.sh redp…"    15 seconds ago   Up 15 seconds   0.0.0.0:18081-18082->18081-18082/tcp, [::]:18081-18082->18081-18082/tcp, 0.0.0.0:19092->19092/tcp, [::]:19092->19092/tcp, 0.0.0.0:19644->9644/tcp, [::]:19644->9644/tcp   redpanda-0
```

Tästä voi päätellä, että sinulla on seuraavat API:t olemassa:

| API                 | Portti | Käyttö                                                                                                                                              |
| ------------------- | ------ | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| Kafka API           | 19092  | Producerit ja consumerit käyttävät tätä porttia lähettäessään viestejä Redpandaan ja lukiessaan niitä Redpandasta Kafka-protokollalla.              |
| HTTP Proxy          | 18082  | Pandaproxy, joka tukee [subsettiä kaikista Kafka API:n toiminnallisuuksista](https://docs.redpanda.com/api/doc/http-proxy/).                        |
| Schema Registry API | 18081  | Schema Registry tarjoaa REST API:n, jonka kautta voit hallita Kafka-viestien skeemoja (Avro, JSON Schema, Protobuf, etc.)                           |
| Admin API           | 19644  | Admin API tarjoaa REST API:n, jonka kautta voit hallita Redpanda-klusteriasi, kuten tarkastella brokerien tilaa, hallita topiceja, ja paljon muuta. |
| Redpanda Console    | 8080   | Redpanda Console on web-käyttöliittymä, jonka kautta voit hallita Redpanda-klusteriasi visuaalisesti. Se käyttää Admin API:a taustalla.             |



## Tehtävänanto

### 1: Luo Producer

Luo Python-skripti, joka toimii Kafka producerina. Alla on aihio, josta sinun tulee lähtä liikkeelle:

```python title="producer.py"
# /// script
# dependencies = [
#     "confluent-kafka==2.14.2",
# ]
# requires-python = ">=3.11"
# ///

import argparse
from confluent_kafka import Producer


def delivery_report(err, msg):
    if err is not None:
        print(f"Delivery failed: {err}")
    else:
        print(f"Delivered to {msg.topic()} [{msg.partition()}] @ offset {msg.offset()}")


def main():
    parser = argparse.ArgumentParser()
    parser.add_argument("--bootstrap", default="localhost:19092")
    parser.add_argument("--topic", default="quickstart")
    parser.add_argument("--key", default=None)
    parser.add_argument("--value", required=True)

    args = parser.parse_args()

    producer = Producer({
        "bootstrap.servers": args.bootstrap,
    })

    producer.produce(
        topic=args.topic,
        key=args.key,
        value=args.value,
        callback=delivery_report,
    )

    producer.flush()


if __name__ == "__main__":
    main()
```

Tiedosto hyödyntää PEP 517 -tyylistä metatietoa, joka kertoo riippuvuuksista ja Python-versiosta. 

### 2: Testaa Producer

Voit ajaa tiedostoa `uv`:n avulla helposti:

```bash
uv run producer.py --value "Hello, Redpanda"
```

### 3: Tutustu Redpanda Consoleen

Yllä oleva komento loi uuden viestin. Etsi se Consolesta. Suuntima on `Topics > quickstart`.

### 4: Luo Consumer

Producer lähettää viestejä topiciin, mutta tarvitsemme myös consumerin, joka lukee viestit topicista. Luo projektikansioon tiedosto `consumer.py`, joka toimii Kafka consumerina. Alla on aihio, josta sinun tulee lähteä liikkeelle:

```python title="consumer.py"
# /// script
# dependencies = [
#     "confluent-kafka==2.14.2",
# ]
# requires-python = ">=3.11"
# ///

import argparse
from confluent_kafka import Consumer, KafkaError, KafkaException


def decode(value):
    if value is None:
        return None

    return value.decode("utf-8")


def main():
    parser = argparse.ArgumentParser()
    parser.add_argument("--bootstrap", default="localhost:19092")
    parser.add_argument("--topic", default="quickstart")
    parser.add_argument("--group", default="quickstart-consumers")
    parser.add_argument(
        "--offset-reset",
        choices=["earliest", "latest"],
        default="earliest",
    )

    args = parser.parse_args()

    consumer = Consumer({
        "bootstrap.servers": args.bootstrap,
        "group.id": args.group,
        "auto.offset.reset": args.offset_reset,
        "enable.auto.commit": True,
    })

    consumer.subscribe([args.topic])

    print(f"Kuunnellaan topicia '{args.topic}'.")
    print(f"Consumer group: '{args.group}'.")
    print("Lopeta consumer näppäinyhdistelmällä Ctrl+C.")

    try:
        while True:
            message = consumer.poll(timeout=1.0)

            if message is None:
                continue

            if message.error():
                if message.error().code() == KafkaError._PARTITION_EOF:
                    continue

                raise KafkaException(message.error())

            key = decode(message.key())
            value = decode(message.value())

            print(
                f"topic={message.topic()} "
                f"partition={message.partition()} "
                f"offset={message.offset()} "
                f"key={key!r} "
                f"value={value!r}"
            )

    except KeyboardInterrupt:
        print("\nConsumer pysäytetään.")

    finally:
        consumer.close()


if __name__ == "__main__":
    main()
```

Consumerin tärkeimmät asetukset ovat:

* `bootstrap.servers` kertoo Redpandan osoitteen.
* `group.id` määrittää consumer groupin, johon consumer kuuluu.
* `auto.offset.reset` määrittää, mistä consumer aloittaa lukemisen, jos consumer groupille ei ole vielä tallennettua offsetia.
* `enable.auto.commit` ottaa käyttöön automaattisen offsettien tallentamisen.
* `subscribe()` tilaa yhden tai useamman topicin.
* `poll()` odottaa seuraavaa viestiä brokerilta.
* `close()` poistuu consumer groupista hallitusti ja vapauttaa consumerin resurssit.

Consumer groupin tallentamien offsettien avulla consumer voi jatkaa käsittelyä edellisestä kohdasta uudelleenkäynnistyksen jälkeen. Saman group ID:n consumerit jakavat topicin partitionit keskenään.

### 5: Käynnistä Consumer

Avaa uusi terminaali-istunto projektikansioon ja käynnistä consumer:

```bash
uv run consumer.py
```

Jos vaiheessa 2 lähettämäsi viesti on edelleen topicissa, consumerin pitäisi tulostaa jotakin seuraavan kaltaista:

```plaintext title="stdout"
Kuunnellaan topicia 'quickstart'.
Consumer group: 'quickstart-consumers'.
Lopeta consumer näppäinyhdistelmällä Ctrl+C.
topic=quickstart partition=0 offset=0 key=None value='Hello, Redpanda'
```

!!! tip "Kokeile sulkea ja avata uusiksi"

    Kokeile lopettaa consumer process painamalla ++ctrl+c++ ja käynnistää se uudelleen. Toistuuko sama `Hello, Redpanda`-viesti? Jos ei, niin miksi?

### 6: Lähetä useampi viesti

Anna consumerin olla käynnissä ja avaa uusi terminaali-istunto. Lähetä useampi viesti producerilla:

```bash
uv run producer.py --value "Viesti 1"
uv run producer.py --value "Viesti 2"
uv run producer.py --value "Viesti 3"
```

Consumerin pitäisi vastaanottaa jokainen viesti lähes välittömästi.

### 7: Toinen Consumer Group

Voit antaa vanhan consumerin olla käynnissä tai sulkea sen -- ne ovat itsenäisiä. Luo uusi consumer toisella group ID:llä:

```bash
uv run consumer.py --group toinen-consumer-group
```

Huomaat, että tämä toinen consumer noutaa kaikki viestit uusiksi. Tämä on eri consumer group, joten se ei jaa offsetteja ensimmäisen consumerin kanssa. Jokainen consumer group saa oman kopion topicin viesteistä. Huomaa, että consumer groupin voi luoda myös siten, että se aloittaa lukemisen viimeisimmästä viestistä, jolloin se ei saa vanhoja viestejä. Tätä varten on olemassa tämä argumentti:

```python title="consumer.py"
# ...
    parser.add_argument(
        "--offset-reset",
        choices=["earliest", "latest"],
        default="earliest",
    )
# ...
```

### 8: Uusi topic ja skeema

Tähän asti topiciin on voinut lähettää mitä tahansa merkkijonoja. Käytännön järjestelmissä halutaan kuitenkin usein varmistaa, että viestit noudattavat ennalta sovittua rakennetta. Tätä varten Redpanda tarjoaa Schema Registryn. Aloitetaan tutustuminen siihen luomalla uusi topic ja siihen kohdistuva skeema.

**Luo uusi topic**:

```bash
docker exec -it redpanda-0 rpk topic create cars --partitions 1
``` 

!!! tip

    Voisit tehdä saman myös Redpanda Consolen kautta. Suuntima on `Topics > Create Topic`. Valitse topicin nimeksi `cars` ja partitionien määräksi 1.

**Luo uusi skeema**:

Tämän voisi tehdä `docker exec`-lähetymistavalla, mutta komento olisi turkasen pitkä, joten käytetään Redpanda Consolea. Siirry kohtaan: `Schema Registry → Create Schema`. Valitse `cars`-topic ja määritä skeemaksi seuraava JSON:

```json
{
   "type": "record",
   "name": "car",
   "fields": [
      {
         "name": "model",
         "type": "string"
      },
      {
         "name": "year",
         "type": "float"
      }
   ]
}
```

Valitse `Schema applies to`: kohdasta `[x] Value`. Huomaa, että tämä asetetaan nimenomaan `uv run producer.py --value 'TÄHÄN'`, eli me luomme skeeman viestin arvon (value) osalle. Avain (key), jota käytetään partitiointiin, on edelleen vapaa.

### 9: Lähetä virheellinen viesti

Lähetä viesti, joka ei todellakaan noudata skeemaa. Esimerkiksi:

```bash
uv run producer.py --topic cars --value '{"lorem": "ipsum"}'
```

Huomaat, että Redpanda ei hylkää viestiä. On tärkeä huomata, että Schema Registry ei yksin takaa datan laatua, vaan asiakkaiden (producer/consumer) täytyy käyttää sitä oikein.

### 10: Validoi

Luo uusi Python-skripti, joka validoi viestin skeeman mukaan. Käytä `fastavro` kirjastoa validointiin. Alla on skripti, jota voit käyttää:

```python title="validate.py"
# /// script
# dependencies = [
#     "requests",
#     "fastavro",
# ]
# requires-python = ">=3.11"
# ///

import argparse
import json
import requests
from fastavro.validation import validate, ValidationError
from fastavro import parse_schema


def main():
    parser = argparse.ArgumentParser()
    parser.add_argument(
        "--registry",
        default="http://localhost:18081",
        help="Schema Registry URL",
    )
    parser.add_argument(
        "--subject",
        default="cars-value",
        help="Schema Registry subject",
    )
    parser.add_argument(
        "--value",
        required=True,
        help="JSON viesti",
    )

    args = parser.parse_args()

    # Hae viimeisin skeema
    response = requests.get(
        f"{args.registry}/subjects/{args.subject}/versions/latest"
    )
    response.raise_for_status()

    schema_response = response.json()

    # Registry palauttaa skeeman merkkijonona
    schema = json.loads(schema_response["schema"])
    schema = parse_schema(schema)

    # Käyttäjän syöte
    message = json.loads(args.value)

    try:
        validate(message, schema)
        print("✅ Viesti on skeeman mukainen.")
    except ValidationError as e:
        print("❌ Viesti ei noudata skeemaa.")
        print(f"Virhe: {e}")

if __name__ == "__main__":
    main()
```

Nyt jos lähetät virheellisen viestin, esimerkiksi näin:

```bash
uv run validate.py --value '{"lorem":"ipsum"}'
```

...saat vastauksen:

```plaintext title="stdout"
❌ Viesti ei noudata skeemaa.
Virhe: [
  "Field(car.model) is None expected string"
]
```

Tuotannossa tätä ei tietenkään tehtäisi käsin jokaiselle viestille, vaan producerin koodi tarkistaisi skeeman ennen viestin lähettämistä ja/tai consumerin koodi tarkistaisi skeeman vastaanotetulle viestille. Skeemat on mahdollista versioida, joten viestityyppiin on mahdollista lisätä myöhemmin uusia kenttiä. Producerin ja consumerin täytyy tietää, mitä skeemaversiota käsiteltävä viesti käyttää (tai niiden pitää olla aina taaksepäin yhteensopivia, jolloin voi käyttää aina uusinta versiota).

### 11: Validoi onnistunut viesti

Tämä jää sinun tehtäväksesi. Luo JSON-viesti, joka noudattaa skeemaa ja validoi se. Eli siis:

```bash
uv run validate.py --value 'MITÄ TULEE TÄHÄN?'
```

### 12: Lopeta Redpanda

Aja lopuksi palvelu alas ja tuhoa mennessäsi volumet:

```bash
docker compose down -v
```

## Videolla esitettävä

1. Kerrot, kuinka monta tuntia käytit harjoitukseen.
2. Selität lyhyesti, mikä on Redpanda ja mihin sitä käytetään.
3. Käynnistät ympäristön videolla (`docker compose up -d`).
4. Avaat selaimeen Redpanda Consolen ja näytät, että klusteri on käynnissä.
5. Suoritat producerin, jolla lähetät viestin quickstart-topiciin.
6. Näytät Consolesta, että viesti löytyy topicista.
7. Käynnistät consumerin ja näytät, että se vastaanottaa viestejä.
8. Lähetät vähintään yhden uuden viestin producerilla consumerin ollessa käynnissä.
9. Luot tai näytät valmiiksi luodun skeeman Schema Registryssä, joka kohdistuu `cars`-topiciin.
10. Näytät, että virheellisen JSON-viestin lähettäminen cars-topiciin onnistuu, vaikka skeema on rekisteröity.
11. Suoritat `validate.py`-skriptin virheellisellä JSON-viestillä ja näytät validointivirheen.
12. Suoritat `validate.py`-skriptin skeeman mukaisella JSON-viestillä ja näytät onnistuneen validoinnin.
13. Selität omin sanoin, miksi pelkkä Schema Registryyn tallennettu skeema ei estä virheellisten viestien lähettämistä.
