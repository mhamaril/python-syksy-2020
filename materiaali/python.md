# Muutamia toteutukseen liittyviä vihjeitä

## Sovelluksen käyttöliittymä

Voit siis tehdä sovelluksellesi tekstikäyttöliittymän tai graafisen käyttöliittymän. Tekstikäyttöliittymän tekeminen on toki useimmiten huomattavasti helpompaa, mutta se voi olla hieman tylsää ja graafisen käyttöliittymän tekemättömyys saattaa [vaikuttaa arvosanaan](./arvosteluperusteet.md).

Pääasia on joka tapauksessa, että pyrit _eriyttämään mahdollisimman hyvin sovelluslogiikan käyttöliittymästä_. Käyttöliittymän roolin tulee siis olla ainoastaan käyttäjän kanssa tapahtuva interaktio, varsinaisen logiikan tulee tapahtua muissa oliossa.

### Tekstikäyttöliittymä

Tarkastellaan erästä mallia tekstikäyttöliittymän toteuttamiselle. Otetaan esimerkkisovellukseksi numerotiedostelu sovellus, jonka avulla käyttäjä voi mm. lisätä, poistaa ja hakea henkilöiden puhelinnumeroita. Näitä toimintoja varten on toteutettu joukko komentoriviltä annettavia komentoja. Tässä on käyttöliittymälle yksi potentiaalinen toteutustapa:

```python
from numero_ja_osoite_palvelu import NumeroJaOsoitePalvelu

KOMENNOT = {
    "x": "x lopeta",
    "1": "1 lisää numero",
    "2": "2 hae numerot",
    "3": "3 hae puhelinnumeroa vastaava henkilö",
    "4": "4 lisää osoite",
    "5": "5 hae henkilön tiedot",
    "6": "6 poista henkilön tiedot",
    "7": "7 filtteröity listaus",
}


class Numerotiedustelu:
    def __init__(self):
        self.lue = input
        self.tulosta = print
        self.palvelu = NumeroJaOsoitePalvelu()

    def kaynnista(self):
        self.tulosta("numerotiedustelu")
        self.tulosta_ohje()

        while True:
            self.tulosta("")
            komento = self.lue("komento: ")

            if not komento in KOMENNOT:
                self.tulosta("virheellinen komento")
                self.tulosta_ohje()
                continue

            if komento == "x":
                break
            elif komento == "1":
                self.lisaa_numero()
            elif komento == "2":
                self.hae_numerot()
            elif komento == "3":
                self.hae_henkilo()
            elif komento == "4":
                self.lisaa_osoite()
            elif komento == "5":
                self.hae_tiedot()
            elif komento == "6":
                self.poista_henkilo()
            elif komento == "7":
                self.listaus()

    def hae_numerot(self):
        nimi = self.tulosta("kenen: ")
        numerot = self.palvelu.hae_numerot(nimi)

        if len(numerot) == 0:
            self.tulosta("ei löytynyt")
            return

        for numero in numerot:
            self.tulosta(numero)

    def lisaa_numero(self):
        nimi = self.lue("kenelle: ")
        numero = self.lue("numero: ")

        self.palvelu.lisaa_numero(nimi, numero)

    # lisää käyttöliittymäfunktioita...
```

Käytössäolevat komennot on tallennettu `KOMENNOT`-nimiseen [dictionaryyn](https://docs.python.org/3/tutorial/datastructures.html#dictionaries), jonka avaimina toimivat komentojen nimet ja arvoina niiden kuvaukset. Käyttöliittymää varten on toteutettu `Numerotiedustelu`-luokka. Luokan konstruktori alustaa oliomuuttujat `lue`, `tulosta` ja `palvelu`. Metodit `lue` ja `tulosta` tallennetaan viittaukset tuttuihin [print](https://docs.python.org/3/library/functions.html#print)- ja [input](https://docs.python.org/3/library/functions.html#input)-funktioihin. Metodiin `palvelu` sen sijaan tallenetaan `NumeroJaOsoitePalvelu`-luokan olio, jonka avulla voimme tehdä puhelinnumeroihin liittyviä operaatioita. Luokan käyttö on tapa erottaa sovelluslogiikka käyttöliittymästä, joka on periaatteena erittäin tärkeä.

`Numerotiedustelu`-luokan `kaynnista`-metodi käynnistää käyttöliittymän. Metodi `tulosta_ohje` tulostaa käyttäjälle käytössäolevat komennot. Tämän jälkeen käyttäjältä aletaan pyytää komentoja `while True`-silmukassa.

Jos komentojen määrä kasvaa, voi harkita esimerkiksi [Command](https://en.wikipedia.org/wiki/Command_pattern)-suunnittelumallin käyttöä. Toteutuksessa komennot voisivat olla omia luokkiaan, kuten:

```python
class LisaaNumeroKomento:
    def __init__():
        self.lue = input
        self.tulosta = print
        self.palvelu = NumeroJaOsoitePalvelu()

    def tulosta_ohje():
        return "1 lisää numero"

    def suorita():
        nimi = self.lue("kenelle: ")
        numero = self.lue("numero: ")

        self.palvelu.lisaa_numero(nimi, numero)
```

Kaikilla komentoluokilla on siis metodit `tulosta_ohje` ja `suorita`. Komennot voi tallentaa dictionaryyn `Numerotiedustelu`-luokan konstruktorissa:

```python
class Numerotiedustelu:
    def __init__(self):
        self.lue = input
        self.tulosta = print

        self.komennot = {
            "x": LopetaKomento()
            "1": LisaaNumeroKomento()
            # ...
        }

    # ...
```

Tämä yksinkertaistaa `Numerotiedustelu`-luokan `kaynnista`-metodia:

```python
def kaynnista(self):
    self.tulosta("numerotiedustelu")
    self.tulosta_ohje()

    while True:
        self.tulosta("")
        komento = self.lue("komento: ")

        if not komento in self.komennot:
            self.tulosta("virheellinen komento")
            self.tulosta_ohje()
            continue

        if komento == "x":
            break

        komento_olio = self.komennot[komento]
        komento_olio.suorita()
```

### Graafinen käyttöliittymä

Graafinen käyttöliittymä eroaa tekstikäyttöliittymästä siinä, että komentoriviltä annettavien tekstimuotoisten komentojen sijaan käyttäjä voi antaa sovellukselle syötteitä erilaisten graafisten komponenttien kautta. Tämä voi tarkoittaa esimerkiksi tekstikenttiin kirjoittamista, tai painikkeiden painelua.

[TkInter](https://wiki.python.org/moin/TkInter)-kirjasto on Pythonissa jo standardiksi muodostonut tapa toteuttaa graaffisia käyttöliittymä. Koska aihe on jonkin verran tekstikäyttöliittymä laajempi, on sille kirjoitettu oma [ohjeensa](./tkinter.md).

## Riippuvuuksien injektointi

Edellä esitetyn `Numerotiedustelu`-luokan attribuutit `lue`, `tulosta` ja `palvelu` alustettiin suoraan konstruktorissa:

```python
class Numerotiedustelu:
    def __init__(self):
        self.lue = input
        self.tulosta = print
        self.palvelu = NumeroJaOsoitePalvelu()

# ...
```

Näitä attribuutteja voidaan pitää luokan _riippuvuuksina_, eli toiminnallisuuksina, joita luokka käyttää. Nämä riippuvuudet ovat tällä hetkellä varsin _konkreettisia_, koska tiedämme esimerkiksi täsmälleen, miten syötteet luetaan. Jos haluaisimme lukea syötteitä esimerkiksi tiedostosta, vaatisi se muutoksia luokan koodiin.

_Riippuvuuksien injektointi_ on ohjelmointitekniikka, jonka avulla pyritään eroon konkreettisista riippuvuuksista. Tekniikan perusideana on, että riippuvuudet annetaan konstruktorille, metodille, tai funktiolle sen kutsun yhteydessä. Tällöin riippuvuuksien käyttäjä ei tiedä riippuvuuksien toteutuksesta mitään, eli ne ovat _abstrakteja_. `Numerotiedustelu`-luokan tapauksessa `lue`-, `tulosta`- ja `palvelu`-attribuutit saisivat arvonsa konstruktorin parametrien kautta:

```python
class Numerotiedustelu:
    def __init__(lue, tulosta, palvelu):
        self.lue = lue
        self.tulosta = tulosta
        self.palvelu = palvelu

    # ...
```

Voimme alustaa `Numerotiedustelu`-olion seuraavasti:

```python
palvelu = NumeroJaOsoitePalvelu()
numerotiedustelu = Numerotiedustelu(input, print, palvelu)
numerotiedustelu.kaynnista()
```

Koska riippuvuudet eivät ole enää konkreettisia, voimme helposti vaihtaa riippuvuutena saadun palvelun toteutusta ilman, että se vaatisi muutoksia `Numerotiedustelu`-luokkaan. Sanotaan, että numerot tallennettaisiin ja luettaisiin tietokannasta seuraavan luokan avulla:

```python
class TietokantaNumeroJaOsoitePalvelu:
    def hae_numerot(self):
        # ...

    def lisaa_numero(self, nimi, numero):
        # ...

    # ...
```

Voimme yksinkertaisesti antaa `Numerotiedustelu`-luokalle riippuvuutena `TietokantaNumeroJaOsoitePalvelu`-luokan olion
`NumeroJaOsoitePalvelu`-luokan olion sijaan:

```python
palvelu = TietokantaNumeroJaOsoitePalvelu()
numerotiedustelu = Numerotiedustelu(input, print, palvelu)
numerotiedustelu.kaynnista()
```

Riippuvuuksien injektointi osoittaa myös erittäin hyödylliseksi testaamisessa. Pythonin `input`- ja `output`-funktioiden käyttö tekee testaamisesta erittäin hankalaa. Koska injektoimme riippuvuudet, voimme toteuttaa niille toteutukset, jotka ovat käyttökelpoisia testeissä:

```python
import unittest
from numerotiedustelu import Numerotiedustelu


class TestNumerotiedustelu(unittest.TestCase):
    def test_numeron_lisays_lisaa_tiedot_oikein(self):
        syotteet = ["1", "Kalle Ilves", "040-123456", "x"]
        tulosteet = []

        lue = lambda viesti: syotteet.pop(0)
        tulosta = lambda viesti: tulosteet.append(viesti)
        palvelu = NumeroJaOsoitePalvelu()
        numerotiedustelu = Numerotiedustelu(lue, tulosta, palvelu)
        numerotiedustelu.kaynnista()

        # varmista assert-lauseella että ohjelman tulostus oli halutun kaltainen
```

## Tietojen tallennus

Arvosteluperusteet [kannustavat](./arvosteluperusteet.md) siihen, että ohjelmasi käsittelisi johonkin muotoon pysyväistalletettua tietoa. Kannattaa kuitenkin pitää talletettavan tiedon määrä kohtuullisena, eeppisimmät tietoa käsittelevät aiheet sopivat paremmin kurssille Tietokantasovellus.

### Repository-suunnittelumalli

Riippumatta mihin tiedon tallennat, kannattaa tiedon haku ja tallentaminen eristää sovelluksen muista osista. Kun piilotamme näihin operaatioihin liittyvän koodin yksityiskohdat sovelluksen muulta koodilta, on esimerkiksi tiedon tallennustapaan helppo tehdä muutoksia ilman, että sillä on vaikutuksia muualla. Tämä onnistuu noudattamalla [repository](https://docs.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/infrastructure-persistence-layer-design)-suunnittelumallia.

Repository-suunnittelumallin perusidea on se, että jokaisella tietokohteella (kuten todo-sovelluksen tehävä) on oma repositorionsa (ei tule sekoittaa gitin repositorioihin). Repositorio tarjoaa tietokohteeseen erilaisia luku- ja kirjoitusoperaatioita. Käytetään esimerkkinä referenssisovelluksen `TodoRepository`-luokkaa:

```python
class TodoRepository:
    def __init__(self, file_path):
        self.file_path = file_path

    def ensure_file_exists(self):
        Path(self.file_path).touch()

    def read(self):
        todos = []

        self.ensure_file_exists()

        with open(self.file_path) as file:
            for row in file:
                row = row.replace("\n", "")
                parts = row.split(";")

                todo_id = parts[0]
                content = parts[1]
                done = parts[2] == "1"
                username = parts[3]

                user = user_repository.find_by_username(
                    username) if username else None

                todos.append(
                    Todo(content, done, user, todo_id)
                )

        return todos

    def write(self, todos):
        self.ensure_file_exists()

        with open(self.file_path, "w") as file:
            for todo in todos:
                done_string = "1" if todo.done else "0"
                username = todo.user.username if todo.user else ""

                row = f"{todo.id};{todo.content};{done_string};{username}"

                file.write(row+"\n")

    def find_all(self):
        return self.read()

    def find_by_username(self, username):
        todos = self.find_all()

        user_todos = filter(
            lambda todo: todo.user and todo.user.username == username, todos
        )

        return list(user_todos)

    def create(self, todo):
        todos = self.find_all()

        todos.append(todo)

        self.write(todos)

        return todo
```

`TodoRepository`-luokka tarjoaa metodit tiedon lukemista varten metodit `find_all` ja `find_by_username`. Nämä metodit hakevat tietoa CSV-tiedostosta ja muodostovat sen riveistä `Todo`-luokan olioita. Tiedon kirjoittamista varten luokka tarjoaa metodin `create`.

`TodoRepository`-luokasta voi tehdä olion seuraavasti:

```python
import os

dirname = os.path.dirname(__file__)

todo_repository = TodoRepository(os.path.join(dirname, "..", "data", "todos.csv"))
```

Annamme luokan konstruktorille polun CSV-tiedostolle. Tämän jälkeen voimme käyttää repositorion metodeja tietämättä, miten tieto tallennetaan tai haetaan:

```python
todo_repository.create(Todo("Learn the repository pattern"))

todos = todo_repository.find_all()

print(todos)
```

Huomaa, että luokan käyttäjä ei ole tietoinen, miten tieto haetaan. Tämä mahdollistaa tallennustapaan tehtävät muutokset vaivattomasti ilman muutoksia muuhun koodiin.

### SQLite-tietokannan käyttö

SQLite-tietokantaa kannattaa käyttää sovelluksessa [sqlite3](https://docs.python.org/3/library/sqlite3.html)-moduulin kautta. Mikä tekee SQLite-tietokannan käytöstä hieman hankalampaa perinteiseen tiedostoon verrattuna on se, että sen käyttö vaatii tietokantaulujen alustuksen.

Tietokantayhteys kannattaa muodostaa omassa moduulissaan esimerkiksi _src/database_connection.py_-tiedostossa:

```python
import os
import sqlite3

dirname = os.path.dirname(__file__)

connection = sqlite3.connect(os.path.join(__dirname, "..", "data", "database.sqlite"))
connection.row_factory = sqlite3.Row


def get_database_connection():
    return connection
```

Ennen tietokantaulujen alustusta kannattaa entiset tietokantaulut poistaa. Näin esimerkiksi uuden sarakkeen lisääminen tauluun onnistuu helposti. Tietokannan alustustoimenpiteitä varten kannattaa toteuttaa oma moduulinsa esimerkiksi _src/initiailize_database.py_ tiedostoon:

```python
from database_connection import get_database_connection


def drop_tables(connection):
    cursor = connection.cursor()

    cursor.execute('''
        drop table if exists mytable;
    ''')

    connection.commit()


def create_tables(connection):
    cursor = connection.cursor()

    cursor.execute('''
        create table mytable (
            id text primary key,
            mycolumn text
        );
    ''')

    connection.commit()


def initialize_database():
    connection = get_database_connection()

    drop_tables(connection)
    create_tables(connection)


if __name__ == "__main__":
    initialize_database()
```

Moduulin `initialize_database`-funktion voi suorittaa joko projektin toisesta tiedostosta:

```python
from initialize_database import initialize_database

initialize_database()
```

Tai komentoriviltä:

```bash
python3 src/initialize_database.py
```

Etenkin tietokantaoperaatioita testaavien testien kanssa funktiokutsun avulla tapahtuva alustaminen on erittäin hyödyllinen. Ennen kuin testit suoritetaan `pytest`-komennolla, pytest tarkistaa, onko testihakemistossa _conftest.py_-tiedostoa. Jos kyseinen tiedosto löytyy, se kutsuu tiedostossa määriteltyä `pytest_configure`-funktiota ennen testien suorittamista. Tästä syystä funktion sisällä onkin hyödyllistä tehdä tietokannan alustus:

```python
from initialize_database import initialize_database


def pytest_configure():
    initialize_database()
```

### Huomioita testaamisesta

[Testausohjeissa](./unittest.md) ohjeistettiin, että _kaikki testit tulee olla toisistaan riippumattomia_. Tämä tarkoittaa sitä, että tallennusta testaavan metodin tulee olettaa, ettei edellisiä tallennustietoja ole. Tämä onnistuu testiluokan `setUp`-metodissa kutsumalla repositorion metodia, joka tyhjentää tallennustiedot. Esimerkiksi referenssisovelluksessa toteutus on seuraava:

```python
class TestTodoRepository(unittest.TestCase):
    def setUp(self):
        todo_repository.delete_all()
        user_repository.delete_all()

    # ...
```

## Sovelluksen konfiguraatiot

Sovelluksen koodiin ei ole syytä kovakoodata mitään konfiguraatioita, kuten sen käyttämien tiedostojen tai tietokantojen nimiä. Eräs syy tähän on se, että jos konfiguraatiot ovat koodissa, ei ohjelman normaalin käyttäjän (jolla ei ole pääsyä koodiin) ole mahdollista tehdä muutoksia konfiguraatioihin.

Konfiguraatiot on syytä määritellä ohjelman ulkopuolella, esim. erillisissä konfiguraatiotiedostoissa. Ei siis välttämättä kannatta ottaa mallia edellisten esimerkkien tavasta kovakoodata tiedostojen polkuja esimerkiksi SQLite-tietokanna yhteyden muodostamisessa.

Pipenv [lataa automaattisesti](https://pipenv-fork.readthedocs.io/en/latest/advanced.html#automatic-loading-of-env) projektin juurihakemiston _.env_ tiedostosta niin kutsuttuja _ympäristömuuttujia_. Näihin ympäristömuuttujiin pääsee koodissa käsiksi mm. [os](https://docs.python.org/3/library/os.html)-moduulin [getenv](https://docs.python.org/3/library/os.html#os.getenv)-funktion avulla. Tiedoston _.env_ sisältö voisi olla esimerkiksi seuraava:

```
FOO=bar
LOREM=ipsum
```

Tiedostossa määritellään arvot `FOO` ja `LOREM` ympäristömuuttujille. Jokainen muuttuja määritellään omalla rivillään formaatissa `MUUTTUJAN_NIMI=muuttujan_arvo`. Voimme tulostaa muuttujien arvot seuraavalla tavalla:

```python
import os

print(os.getenv("FOO"))
print(os.getenv("LOREM"))
```

Jos ympäristömuuttajalle ei ole annettu arvoa, `getenv`-funktio palauttaa arvon `None`. Jotta koodi pysyisi selkeänä, kannattaa ympäristömuuttujien arvot lukea erillisessä moduulissa, joka toteutetaan esimerkiksi _src/config.py_ tiedostoon:

```python
import os

# jos os.getenv("FOO") palauttaa arvon None, FOO saa arvokseen "default bar"
FOO = os.getenv("FOO") or "default bar"
LOREM = os.getenv("LOREM") or "default ipsum"
```

Muuttujille on hyvä antaa oletusarvot, jos ympäristömuuttujalle ei ole annettu arvoa. Moduulissa alustetut muuttujat `FOO` ja `LOREM` voidaan importata projektin toisessa tiedostossa seuraavasti:

```python
from config import FOO, LOREM

print(FOO)
print(LOREM)
```

Testeille on usein käytössä eri konfiguraatio, kuin muulla koodilla. Esimerkiksi testien kannattaa käyttää SQLite-tietokannan kanssa eri tiedostoa. Tätä varten voimme tehdä projektin juurihakemistoon erillisen _.env.test_-tiedoston, jonne määritellään testien käyttämät ympäristömuuttujat.

Näiden ympäristömuuttujien lataaminen onnistuu pytestin [pytest-dotenv](https://pypi.org/project/pytest-dotenv/)-lisäosalla. Sen asentaminen onnistuu seuraavalla komennolla:

```bash
python3 -m pipenv install pytest-dotenv
```

Asentamisen lisäksi tulee projektin juurihakemistoon luoda _pytest.ini_-tiedosto, jossa kerrotaan, mistä tiedostosta ympäristömuuttujat ladataan. Tiedoston sisältö on seuraava:

```
[pytest]
env_override_existing_values = 1
env_files =
    .env.test
```

Nyt testit, jotka suoritetaan `pytest`-komennolla, käyttävät _.env.test_-tiedossa määriteltyjä ympäristömuuttujia.

## Uuden tekniikan harjoittelu ja käyttöönotto

Kun olet toteuttamassa jotain itsellesi uudella tekniikalla, esim. tkinter:illä, SQLite-tietokantaa hyödyntäen, tai teet ohjelmaasi laajennuksen hyödyntämällä kirjastoa, jota et vielä tunne, kannattaa ehdottomasti tehdä uudella tekniikalla erillisiä kokeiluja varsinaisen ohjelmasi ulkopuolella, omassa pienessä koesovelluksessa.

Jos yrität "montaa asiaa yhtä aikaa" eli ottaa esim. SQLite-tietokannan käyttöön omassa jo pitkälle edenneessä ohjelmassasi, on aika varmaa, että saat ainoastaan aikaan suuren määrän ongelmia. Silloin kun koodia ja liikkuvia osia on paljon, ei ole koskaan varmuutta missä ongelma on, ja sen takia on erittäin hyödyllistä, että teet harjoittelun ja kokeilut erillisessä "proof of concept"-sovelluksessa ja kun saat esim. SQLite-tietokannan toimimaan kokeilusovelluksessa, on usein sen jälkeen helppoa "copypasteta" koodi varsinaiseen sovellukseen.
