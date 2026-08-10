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

> 245286dd5b55   docker.redpanda.com/redpandadata/redpanda:v26.1.9   "/entrypoint.sh redp…"    15 seconds ago   Up 15 seconds   0.0.0.0:18081-18082->18081-18082/tcp, [::]:18081-18082->18081-18082/tcp, 0.0.0.0:19092->19092/tcp, [::]:19092->19092/tcp, 0.0.0.0:19644->9644/tcp, [::]:19644->9644/tcp   redpanda-0

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

### 4: TODO! Jatka harjoitusta

Tämä artikkeli on kesken. Opettajan pittää jatkaa.