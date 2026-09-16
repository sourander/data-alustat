# Yleisohjeet

!!! warning

    Käytäthän opettajan sinulle antamaasi repositoriota, jotta opettajalla on automaattisesti pääsy sinun työhösi. Varmista, että repositoryn juuressa on `README.md`-tiedosto, josta alkaen käyttäjä kuljetetaan eri ohjeiden äärelle. Jos dokumentaatio on repositorion punainen lanka, niin `README.md` on lankakerän pää. Siitä on hyvä linkittää kaikkiin ohjeisiin, jotta opettaja löytää ne helposti.

## Repositorio

Käytä repositoriossa ns. monorepo-rakennetta, jossa kaikki harjoitukset sijaitsevat samassa repositoriossa. Jokaisella harjoituksella on oma kansio. Esimerkiksi:

```
.
├── README.md
├── cdc01
│   └── compose.yml
├── csc01
│   ├── MEMO.md
│   └── compose.yml
├── lak01
│   ├── main.py
│   └── compose.yml
├── orc01
│   ├── dags/
│   └── compose.yaml
└── <ID>
    ├── ...
    └── ...
```

Tarkka repositorion rakenne on kuitenkin sinun päätettävissä. Varmista kuitenkin, että se on selkeä siten, että opettaja löytää `README.md`-tiedoston ja sitä kautta muut harjoitukset. On esimerkiksi täysin sallittua antaa juokseva järjestysnumero tekemisille harjoituksilla, kuten `01_CDC01`, `02_CSC01` jne.

!!! tip "CLI-työskentely"

    Kun ajat harjoituksia, siirry kyseiseen hakemistoon, ja aja komennot siellä. Esimerkiksi jos teet CDC01-harjoitusta, niin:

    ```bash
    cd cdc01
    docker compose up -d
    ```

## Videot

Videon tekemisen ohjeistus on ulkoistettu [HedgeDoc: Videotehtävän yleisohjeistus](https://gitlab.dclabra.fi/wiki/s/WX2xszsjJe) -dokumenttiin, koska sama ohjeistus on käytössä useilla kursseilla.
