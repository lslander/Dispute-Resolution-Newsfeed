# Dispute Resolution Newsfeed

Uutisvirta riidanratkaisua koskevista ratkaisuista: oikeuskäytäntö,
välimiesmenettely ja prosessi. Sama putki kuin tietosuojaseurannassa
(oikeusfeed), tätä varten omilla lähteillä ja avainsanoilla. Python-skripti
hakee aineiston, suodattaa sen avainsanoilla ja kirjoittaa staattisen sivun.
GitHub Actions ajaa haun joka aamu ja GitHub Pages julkaisee sivun.

Ei palvelinta, ei tietokantaa, ei kuukausimaksua.

## Mikä eroaa tietosuojafeedistä

Koodi (`fetchers.py`, `filtering.py`, `main.py`, `render.py`) on sama. Erot:

- `sources.yaml` on kokonaan oma: DR-lähteet ja DR-avainsanat.
- `render.py`:n sivun esittelyteksti ja käsin tarkistettavien lähteiden lista
  on muutettu riidanratkaisulle.
- Workflow ajaa joka aamu (`cron: "10 5 * * *"`), ei vain arkisin.

## Lähteet (10)

| Ryhmä | Lähteet |
|---|---|
| Kansallinen oikeuskäytäntö | KKO, hovioikeudet, markkinaoikeus, työtuomioistuin, vakuutusoikeus (tuomioistuimet.fi:n ratkaisusivut) sekä tuomioistuinlaitoksen ajankohtaissyöte, joka tuo käräjäoikeuksien tiedotteet |
| EU | Unionin tuomioistuin (SPARQL, oma require_any rajat ylittäviin riitoihin), curia-lehdistötiedotteet |
| Muut | Edilex-uutiset, Suomen Asianajajaliitto |

Käräjäoikeuksilla ei ole omaa ratkaisulistaa, joten ne näkyvät vain tiedotteina
ajankohtaissyötteestä. Välitystuomiot (esim. FAI) eivät ole julkisia, joten
niitä ei voi hakea koneellisesti. Nämä näkyvät sivun lähdepaneelissa käsin
tarkistettavina.

## Suodatus

Riidanratkaisu on vaikea rajata avainsanoilla, koska lähes jokainen siviilijuttu
on riita. `must_any` on siksi laaja prosessi-, sopimus-, vahingonkorvaus- ja
täytäntöönpanotermistö, ja `never` karsii pois alueet, jotka tulevat läpi
yleisen prosessitermin takia mutta eivät kuulu kaupalliselle DR-tiimille:
julkiset hankinnat, rikos- ja säilöönottoasiat sekä perhe- ja perintöjutut.
Jos haluat jonkin näistä takaisin, poista rivi `never`-listalta.

Säädä aina `--dry-run`:lla ennen julkaisua:

```bash
pip install -r requirements.txt
python main.py --dry-run     # näyttää osumat, ei kirjoita tiedostoja
python main.py               # kirjoittaa docs-kansion
python main.py --only kko-ratkaisut,hovi-ratkaisut --dry-run
```

`must_any` on portti, `boost` nostaa jutun tärkeäksi, `never` pudottaa pois
(katsotaan vain otsikosta ja asiasanoista). `require_any` on lähdekohtainen
lisäportti, jota EU-lähde käyttää.

## Käyttöönotto

1. Luo GitHubiin uusi julkinen repo, nimeltä `Dispute-Resolution-Newsfeed`, ja
   työnnä nämä tiedostot sinne.
2. **Vaihda** `.github/workflows/update.yml`:n `--base-url` oikeaan muotoon
   `https://KAYTTAJATUNNUS.github.io/REPON-NIMI/`. Nyt siinä on oletus
   `https://lslander.github.io/Dispute-Resolution-Newsfeed/`, joka toimii vain jos
   repon nimi on täsmälleen tämä.
3. Settings -> Pages: Source *Deploy from a branch*, Branch `main`, kansio
   `/docs`. Pages toimii ilmaistilillä vain julkisessa repossa.
4. Actions-välilehti -> valitse työnkulku -> Run workflow. Ensimmäinen ajo
   kestää pari minuuttia, koska EU-kysely on hidas.
5. Lisää sivun osoite puhelimen aloitusnäytölle tai tilaa `feed.xml`
   RSS-lukijaan.

## Tiedostot

| Tiedosto | Tehtävä |
|---|---|
| `sources.yaml` | DR-lähteet, avainsanat ja aiheluokat. Tätä muokkaat normaalisti. |
| `fetchers.py` | Lähdekohtaiset hakijat. Sama kuin tietosuojafeedissä. |
| `filtering.py` | Avainsanasuodatus ja järjestys. |
| `render.py` | HTML-sivu, RSS ja JSON. DR-tekstit muutettu. |
| `main.py` | Käynnistys. |
| `docs/` | Valmis sivu. GitHub Pages näyttää tämän kansion. |
| `state.json` | Muistaa nähdyt ratkaisut, jotta uudet saavat "uusi"-merkin. |

Seuranta on apuväline. Tarkista ratkaisu aina alkuperäisestä lähteestä ennen
kuin nojaat siihen.
