---
image: /assets/images/blog/pl/jak-skonfigurowac-docker-compose/og-image.png
title: 'Jak skonfigurować Docker Compose: Kompletny przewodnik do zarządzania aplikacji wielokontenerowych'
description: Kompletny przewodnik krok po kroku do konfiguracji i używania Docker Compose V2 na Ubuntu, Debian, AlmaLinux, Rocky Linux, CentOS i Fedorze.
date: '2026-03-25'
translationKey: setup-docker-compose
category: Poradniki
tags:
  - docker compose
  - docker
  - containers
  - linux
  - vps
  - server administration
  - ubuntu
  - almalinux
  - yaml
howto:
  name: Jak skonfigurować i używać Docker Compose
  totalTime: PT15M
  yield: Aplikacja wielokontenerowa działająca płynnie przez plik docker-compose.yml
  tool:
    - VPS lub dedykowany serwer działający na dystrybucji Linux
    - Docker Engine oficjalnie zainstalowany
    - Konto użytkownika z uprawnieniami do uruchamiania poleceń Docker
  steps:
    - name: Zweryfikuj że wtyczka Docker Compose jest zainstalowana
      text: Uruchom docker compose version aby upewnić że V2 jest aktywnie działające w twoim systemie.
      url: step-1-verify-docker-compose-is-installed
    - name: Utwórz katalog projektu
      text: Użyj mkdir my-project i cd do niego aby utrzymać swoje wdrożenie zorganizowane.
      url: step-2-set-up-a-project-directory
    - name: Napisz swój plik docker-compose.yml
      text: Utwórz konfigurację YAML opisującą swoją usługę WWW, bazę danych i porty.
      url: step-3-create-the-docker-compose-yml-file
    - name: Wdróż swój stos
      text: Uruchom docker compose up -d aby uruchomić kontenery w tle.
      url: step-4-spin-it-up
    - name: Zarządzaj swoim stosem
      text: Użyj docker compose logs, pause, stop lub down aby zarządzać działającymi środowiskami.
      url: step-5-manage-your-environment
status: published
locale: pl
author:
  name: VoxiHost Team
  link: https://voxihost.pl/
contributors:
  - danielmarszalkowski
---

Docker Engine jest fantastyczny do uruchamiania pojedynczych, odizolowanych kontenerów. Ale nowoczesne aplikacje rzadko istnieją w całkowitej izolacji. Zazwyczaj masz serwer WWW, API backend, bazę danych (jak MySQL lub PostgreSQL) i może warstwę buforującą (jak Redis).

Próba uruchomienia wszystkich tych kontenerów ręcznie i ręczne łączenie ich wewnętrznych sieci przez wyczerpujące, milowe polecenia `docker run` jest frustrujące i poważnie podatne na błędy ludzkie.

Wprowadź **Docker Compose**. Pozwala ci zadeklarować cały stos aplikacji w pojedynczym, czystym pliku `.yml` (YAML). Z jednym centralnym poleceniem, Docker buduje wewnętrzne sieci, pobiera wszystkie niezbędne obrazy i uruchamia cały stos sekwencyjnie.

## Krok 1: Zweryfikuj że Docker Compose jest zainstalowany

Jeśli śledziłeś nasze przewodniki instalacji Dockera dla [Ubuntu/Debian](/pl/blog/instalacja-docker-ubuntu-debian/) lub [AlmaLinux/Rocky/Fedora](/pl/blog/instalacja-docker-centos-rhel/), w rzeczywistości już masz zainstalowany Docker Compose!

Nowoczesne dystrybucje Dockera przeszły od starego samodzielnego pliku binarnego `docker-compose` (napisanego w Pythonie) do natywnej **wtyczki Docker Compose V2** (napisanej w Go) osadzonej bezpośrednio w CLI Dockera.

Zweryfikuj że jest zainstalowany sprawdzając jego wersję:

{% image "/assets/images/blog/pl/jak-skonfigurowac-docker-compose/H1.png", "Uruchamianie docker compose version w terminalu na Ubuntu aby zweryfikować że Docker Compose V2 jest zainstalowany", "(max-width: 768px) 100vw, 800px" %}

```bash
docker compose version
```

*(Zwróć uwagę na spację między `docker` a `compose`, nie myślnik!)* 

Powinieneś zobaczyć wynik podobny do tego:
`Docker Compose version v2.32.x`

Jeśli otrzymasz błąd "command not found", musisz zainstalować wtyczkę przez swój menedżer pakietów:
- **Ubuntu/Debian**: `sudo apt install docker-compose-plugin`
- **AlmaLinux/RHEL**: `sudo dnf install docker-compose-plugin`

## Krok 2: Skonfiguruj katalog projektu

Docker Compole opiera się absolutnie na kontekście katalogu w którym uruchamiasz polecenie. Szuka pliku o nazwie `docker-compose.yml` w jakimkolwiek folderze w którym obecnie jesteś.

Najpierw stwórzmy dedykowany dom dla swojego nowego projektu tak aby pliki pozostały zorganizowane:

{% image "/assets/images/blog/pl/jak-skonfigurowac-docker-compose/H2.png", "Uruchamianie mkdir my-webapp i cd my-webapp aby utworzyć i wejść do dedykowanego katalogu projektu Docker Compose na Ubuntu", "(max-width: 768px) 100vw, 800px" %}

```bash
mkdir my-webapp
cd my-webapp
```

## Krok 3: Utwórz plik docker-compose.yml

Teraz stwórzmy funkcjonalny, realistyczny przykład. Będziemy wdrażać oficjalny obraz WordPress i podłączyć go do bezpiecznej bazy danych MySQL backend, czysto konfigurując wszystko przez Compose.

Otwórz nowy plik za pomocą `nano`:

{% image "/assets/images/blog/pl/jak-skonfigurowac-docker-compose/H3.png", "Uruchamianie nano docker-compose.yml aby otworzyć plik konfiguracyjny Docker Compose do edycji na Ubuntu", "(max-width: 768px) 100vw, 800px" %}

```bash
nano docker-compose.yml
```

Wklej następujący blok YAML w całości:

```yaml
services:
  database:
    image: mysql:8.0
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: secure_root_password
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wp_user
      MYSQL_PASSWORD: secure_wp_password
    volumes:
      - db_data:/var/lib/mysql

  wordpress:
    image: wordpress:latest
    restart: always
    ports:
      - "8080:80"
    environment:
      WORDPRESS_DB_HOST: database
      WORDPRESS_DB_USER: wp_user
      WORDPRESS_DB_PASSWORD: secure_wp_password
      WORDPRESS_DB_NAME: wordpress
    depends_on:
      - database

volumes:
  db_data:
```

### Wyjaśnienie konfiguracji:
- **services**: Zdefiniowaliśmy dwa kontenery: `database` i `wordpress`.
- **image**: Mówi Dockerowi jaki szablon kontenera pobrać z Docker Hub.
- **environment**: Wstrzykuje bezpieczne zmienne (jak hasła i nazwy użytkowników) automatycznie do kontenerów tak aby konfigurowały się cicho przy pierwszym uruchomieniu. Spójrz jak kontener `wordpress` wie że jego host to `database` (dokładna nazwa innej usługi). Docker Compose automatycznie zbudował wewnętrzną, niewidoczną sieć dla nich aby mogły ze sobą rozmawiać bezproblemowo!
- **ports**: Mapuje port `8080` na publicznym internecie bezpośrednio do portu `80` (Standardowy HTTP) wewnątrz odizolowanego kontenera WordPress.
- **volumes**: W standardowych kontenerach, gdy usuwasz kontener, wszystkie dane znikają z nim. Zmapowaliśmy przechowywanie bazy danych do woluminu dysku twardego tak aby twoje dane przetrwały nawet jeśli zrestartujesz lub usuniesz kontener!
- **depends_on**: Zapewnia że WordPress nie próbuje uruchomić dopóki baza danych nie działa pomyślnie.

Zapisz i wyjdź.

## Krok 4: Uruchom to

Z jednym plikiem, cała twoja infrastruktura jest zadeklarowana. Aby ją uruchomić, użyj polecenia `up`. Flaga `-d` mówi mu aby działał "odłączony" w tle tak abyś mógł nadal używać konsoli terminala:

{% image "/assets/images/blog/pl/jak-skonfigurowac-docker-compose/H4.png", "Uruchamianie docker compose up -d aby uruchomić wszystkie kontenery zdefiniowane w docker-compose.yml w trybie odłączonym na Ubuntu", "(max-width: 768px) 100vw, 800px" %}

```bash
docker compose up -d
```

Docker automatycznie:
1. Stworzy dedykowaną wewnętrzną sieć dla `my-webapp`.
2. Pobierze ciężkie obrazy MySQL i WordPress.
3. Uruchomi bazę danych, przypisze hasła i zbuduje trwałą partycję przechowywania.
4. Uruchomi serwer WordPress, podłączy go do sieci i zmapuje port 8080.

Gdy skończone, otwórz swoją przeglądarkę internetową i nawiguj do adresu IP swojego serwera połączonego z portem 8080:
`http://your_server_ip:8080`

> **Ostrzeżenie bezpieczeństwa:** Docker zarządza swoimi własnymi regułami sieciowymi przez `iptables`. Gdy używasz mapowanie `ports:` w swoim pliku `docker-compose.yml`, Docker będzie **całkowicie omijać twoją zaporę UFW**. Aby utrzymać usługę prywatną, zmapuj ją do `127.0.0.1:8080` zamiast tylko `8080`.

{% image "/assets/images/blog/pl/jak-skonfigurowac-docker-compose/H5.png", "Ekran kreatora instalacji WordPress w przeglądarce po uruchomieniu stosu WordPress plus MySQL Docker Compose na VPS Ubuntu", "(max-width: 768px) 100vw, 800px" %}

Natychmiast trafisz słyny ekran instalacji WordPress!

## Krok 5: Zarządzaj swoim środowiskiem

Oto kluczowe polecenia do zapamiętania gdy działasz w katalogu zawierającym twój plik `docker-compose.yml`:

Zobacz co aktywnie działa w tym projekcie:

{% image "/assets/images/blog/pl/jak-skonfigurowac-docker-compose/H6.png", "Uruchamianie docker compose ps aby wyświetlić wszystkie działające kontenery i ich porty w bieżącym projekcie Docker Compose", "(max-width: 768px) 100vw, 800px" %}

```bash
docker compose ps
```

Sprawdź głęboko szczegółowe logi systemowe w tle (przydatne jeśli aplikacja ulegnie awarii aby zobaczyć dlaczego umarła):

```bash
docker compose logs
# Dodaj -f aby śledzić logi na żywo
docker compose logs -f
```

Zatrzymaj kontenery tymczasowo bez usuwania ich:

{% image "/assets/images/blog/pl/jak-skonfigurowac-docker-compose/H7.png", "Uruchamianie docker compose stop aby tymczasowo zatrzymać wszystkie działające kontenery w projekcie Docker Compose bez usuwania ich", "(max-width: 768px) 100vw, 800px" %}

```bash
docker compose stop
```

*(Możesz je ponownie uruchomić używając `docker compose start`)*

Zniszcz cały projekt w dół (zatrzymuje kontenery, usuwa je i usuwa wewnętrzne sieci):

{% image "/assets/images/blog/pl/jak-skonfigurowac-docker-compose/H8.png", "Uruchamianie docker compose down aby zatrzymać i usunąć wszystkie kontenery, sieci i woluminy w projekcie Docker Compose", "(max-width: 768px) 100vw, 800px" %}

```bash
docker compose down
```

*(Domyślnie, to NIE usuwa twojego woluminu bazy danych, więc twoje posty WordPress są całkowicie bezpieczne przy ponownym wdrożeniu).*

Aby wdrażać stosy aplikacji o wysokiej dostępności używając architektury kontenerowej, nie ma nic lepszego niż wysokowydajna infrastruktura backendowa ją wspierająca. Uruchom [Premium VPS](/pl/premium-vps/), zainstaluj Docker, skopiuj swoje konfiguracje YAML i uruchom swoje złożone, wielowarstwowe infrastruktury do produkcji bez wysiłku.