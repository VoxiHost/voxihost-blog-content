---
image: /assets/images/blog/pl/instalacja-wordpress-docker-compose/og-image.png
title: "Jak zainstalować WordPressa na VPS przy użyciu Docker Compose"
description: "Kompletny poradnik krok po kroku, jak uruchomić stronę na WordPressie z dedykowaną bazą MySQL przy użyciu Docker Compose na serwerze Linux VPS."
date: '2026-06-02'
translationKey: install-wordpress-docker-compose
locale: pl
category: Poradniki
tags:
  - wordpress
  - docker
  - docker-compose
  - vps
  - mysql
status: draft
author:
  name: VoxiHost Team
  link: https://voxihost.pl/
contributors:
  - sl0ikkk
howto:
  name: "Jak zainstalować WordPressa na VPS przy użyciu Docker Compose"
  totalTime: "PT15M"
  yield: "Serwer z działającą stroną na WordPressie i bazą MySQL przez Docker Compose"
  tool:
    - "Serwer VPS lub dedykowany"
    - "Klient SSH (np. terminal, PuTTY)"
    - "Zainstalowany Docker i Docker Compose"
  steps:
    - name: "Przygotowanie środowiska"
      text: "Stworzenie folderu na projekt."
      url: "krok-1-przygotowanie-srodowiska"
    - name: "Tworzenie konfiguracji"
      text: "Pisanie pliku docker-compose.yml."
      url: "krok-2-tworzenie-konfiguracji"
    - name: "Uruchomienie i instalacja"
      text: "Start kontenerów i kreator WordPressa."
      url: "krok-3-uruchomienie-i-instalacja"
---

## Wstęp

WordPress napędza ponad 40% całego internetu. Choć można go zainstalować tradycyjnie, ręcznie konfigurując serwery Apache/Nginx i bazy danych, użycie **Docker Compose** jest powszechnie uważane za znacznie nowocześniejsze i bezpieczniejsze podejście. Docker izoluje Twoją stronę i bazę danych w osobnych kontenerach, co sprawia, że kopie zapasowe, aktualizacje i migracje stają się banalnie proste.

W tym poradniku uruchomimy oficjalny obraz WordPressa w połączeniu z dedykowaną bazą MySQL.

> **Wymagania wstępne:** Potrzebujesz serwera VPS z systemem Linux, dostępu SSH z kontem użytkownika z uprawnieniami `sudo` oraz zainstalowanego Dockera z Docker Compose. Sprawdź nasz [poradnik konfiguracji Docker Compose](/pl/blog/jak-skonfigurowac-docker-compose/) jeśli potrzebujesz pomocy.

## Krok 1: Przygotowanie środowiska

Upewnij się, że masz zainstalowanego Dockera oraz Docker Compose na swoim serwerze VPS.

Na początek stwórzmy dedykowany folder dla naszej nowej strony. Będzie on przechowywał plik konfiguracyjny oraz wszystkie trwałe dane strony (jak wgrywane obrazki, motywy i wtyczki).

{% image "/assets/images/blog/pl/instalacja-wordpress-docker-compose/H1.png", "Tworzenie folderu projektu", "(max-width: 768px) 100vw, 800px" %}

```bash
mkdir ~/moj-wordpress
cd ~/moj-wordpress
```

## Krok 2: Tworzenie konfiguracji

W Docker Compose cała infrastruktura serwerowa jest definiowana w jednym pliku.

{% image "/assets/images/blog/pl/instalacja-wordpress-docker-compose/H2.png", "Tworzenie pliku docker-compose.yml", "(max-width: 768px) 100vw, 800px" %}

```bash
nano docker-compose.yml
```

Wklej poniższą konfigurację w formacie YAML:

```yaml
services:
  db:
    image: mysql:latest
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: twoje_silne_haslo_root
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wp_user
      MYSQL_PASSWORD: twoje_silne_haslo_wp
    volumes:
      - db_data:/var/lib/mysql

  wordpress:
    image: wordpress:latest
    restart: always
    ports:
      - "8080:80"
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: wp_user
      WORDPRESS_DB_PASSWORD: twoje_silne_haslo_wp
      WORDPRESS_DB_NAME: wordpress
    volumes:
      - wp_data:/var/www/html
    depends_on:
      - db

volumes:
  db_data:
  wp_data:
```

### Co robi ten plik:
1. **Serwis db**: Pobiera oficjalny obraz MySQL Latest, ustawia hasła dostępu i montuje wirtualny dysk `db_data`, dzięki czemu Twoja baza danych nie zniknie po restarcie serwera.
2. **Serwis wordpress**: Pobiera najnowszego WordPressa, przekierowuje publiczny port `8080` do wnętrza kontenera na port 80 i łączy się z bazą danych używając zdefiniowanych haseł. Podłącza też dysk `wp_data`, by pliki strony były bezpieczne.

*Ważne: Zmień `twoje_silne_haslo_root` i `twoje_silne_haslo_wp` na prawdziwe, skomplikowane hasła.*

Zapisz i zamknij plik.

## Krok 3: Uruchomienie i instalacja

Aby włączyć swoją stronę internetową, po prostu wpisz:

{% image "/assets/images/blog/pl/instalacja-wordpress-docker-compose/H3.png", "Uruchamianie Dockera komendą up -d", "(max-width: 768px) 100vw, 800px" %}

```bash
docker compose up -d
```

Docker automatycznie pobierze WordPressa oraz MySQL, połączy je wewnętrzną siecią i uruchomi w tle.

Gdy komenda zakończy działanie, otwórz przeglądarkę i wejdź na adres IP swojego serwera z portem 8080:

`http://ip_twojego_serwera:8080`

{% image "/assets/images/blog/pl/instalacja-wordpress-docker-compose/H4.png", "Kreator instalacji WordPress", "(max-width: 768px) 100vw, 800px" %}

Pojawi się słynny, 5-minutowy instalator WordPressa. Wybierz język, nadaj tytuł swojej stronie i stwórz konto administratora.

---

## Podsumowanie

Twoja strona na WordPressie działa teraz w kontenerach Docker na Twoim VPS-ie. Cały stos — WordPress i MySQL — jest zdefiniowany w jednym pliku `docker-compose.yml`, co sprawia, że tworzenie kopii zapasowych, migracje i aktualizacje są banalnie proste.

W następnym kroku zdecydowanie zalecamy umieszczenie strony za Nginx Proxy Manager, aby przypisać do niej własną domenę i włączyć bezpłatny HTTPS przez Let's Encrypt. Sprawdź nasz poradnik: [jak skonfigurować Nginx Proxy Manager na VPS](/pl/blog/konfiguracja-nginx-proxy-manager-vps/).

Jeśli potrzebujesz niezawodnego serwera VPS dla swojej strony WordPress, sprawdź nasze [plany Premium VPS](/pl/premium-vps/) lub [Budget VPS](/pl/budget-vps/) — oba oferują szybkie dyski NVMe SSD i natychmiastowe wdrożenie.
