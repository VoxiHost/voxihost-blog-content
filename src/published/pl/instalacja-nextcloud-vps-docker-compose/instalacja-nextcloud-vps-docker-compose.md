---
image: /assets/images/blog/pl/instalacja-nextcloud-vps-docker-compose/og-image.png
title: "Jak zainstalować Nextcloud na serwerze VPS za pomocą Docker Compose"
description: "Dowiedz się, jak uruchomić własną, prywatną chmurę Nextcloud na serwerze VPS przy użyciu Docker Compose i bazy MariaDB, bez płacenia abonamentów."
date: '2026-06-15'
translationKey: install-nextcloud-vps-docker-compose
locale: pl
category: Poradniki
tags:
  - nextcloud
  - docker
  - docker-compose
  - vps
  - chmura
status: published
author:
  name: VoxiHost Team
  link: https://voxihost.pl/
contributors:
  - sl0ikkk
  - danielmarszalkowski
howto:
  name: "Jak zainstalować Nextcloud na serwerze VPS za pomocą Docker Compose"
  totalTime: "PT10M"
  yield: "Własna, prywatna chmura na pliki działająca na serwerze VPS"
  tool:
    - "Serwer VPS z systemem Linux"
    - "Klient SSH (np. terminal, PuTTY)"
    - "Zainstalowany Docker i Docker Compose"
  steps:
    - name: "Wymagania wstępne"
      text: "Upewnij się, że masz zainstalowanego Dockera."
      url: "krok-1-wymagania-wstepne"
    - name: "Przygotowanie środowiska"
      text: "Utworzenie katalogu i pliku konfiguracyjnego."
      url: "krok-2-przygotowanie-srodowiska"
    - name: "Uruchomienie Nextcloud"
      text: "Start kontenerów i pierwsza konfiguracja."
      url: "krok-3-uruchomienie-nextcloud"
faq:
  - question: "Jakie są wymagania sprzętowe do uruchomienia Nextcloud w Dockerze?"
    answer: 'Wymagany jest serwer VPS z minimum 1 GB pamięci RAM (zalecamy 2 GB dla optymalnej wydajności bazy danych MariaDB) oraz zainstalowanym Dockerem.'
  - question: "Jak zabezpieczyć Nextcloud za pomocą certyfikatu HTTPS?"
    answer: 'Zalecamy uruchomienie serwera proxy, takiego jak <a href="/pl/blog/konfiguracja-nginx-proxy-manager-vps/">Nginx Proxy Manager</a>, aby przekierować domenę z darmowym certyfikatem SSL na port 8080.'
  - question: "Czy mogę użyć innej bazy danych niż MariaDB?"
    answer: 'Tak, możesz zmodyfikować plik <code>docker-compose.yml</code>, aby korzystał z PostgreSQL lub PostgreSQL Alpine zamiast obrazu MariaDB.'
  - question: "Jak mogę zwiększyć maksymalny rozmiar przesyłanych plików w Nextcloud?"
    answer: 'Możesz to zrobić ustawiając zmienne środowiskowe <code>PHP_UPLOAD_LIMIT</code> oraz <code>PHP_MEMORY_LIMIT</code> w pliku <code>docker-compose.yml</code>.'
  - question: "Jak wykonać kopię zapasową danych z własnej chmury Nextcloud?"
    answer: 'Należy utworzyć kopię zapasową wolumenu bazy danych (za pomocą <code>mysqldump</code>) oraz wolumenu <code>nextcloud_data</code>, w którym przechowywane są wgrane pliki.'
---

## Wstęp

Masz dość płacenia abonamentów za Google Drive, Dropboxa czy iCloud? Posiadając serwer VPS, możesz z łatwością stworzyć własną, całkowicie prywatną chmurę na pliki korzystając z **Nextcloud**. To potężna platforma open-source, która pozwala na przechowywanie plików, synchronizację kontaktów i kalendarzy, dając Ci 100% kontroli nad własnymi danymi.

W tym poradniku uruchomimy Nextcloud za pomocą **Docker Compose**, co gwarantuje czystą, odizolowaną i łatwą w zarządzaniu instalację.

> **Wymagania wstępne:** Potrzebujesz serwera VPS z systemem Linux, dostępu SSH z kontem użytkownika z uprawnieniami `sudo` oraz zainstalowanego Dockera z Docker Compose. Jeśli jeszcze tego nie zrobiłeś, sprawdź nasz [poradnik konfiguracji Docker Compose](/pl/blog/jak-skonfigurowac-docker-compose/).

## Krok 1: Wymagania wstępne

Zanim zaczniemy, upewnij się, że na Twoim VPS zainstalowany jest Docker oraz Docker Compose. Jeśli jeszcze tego nie zrobiłeś, sprawdź nasz [poradnik o podstawach Docker Compose](/pl/blog/jak-skonfigurowac-docker-compose/).

Sprawdź, czy narzędzia są poprawnie zainstalowane:

{% image "/assets/images/blog/pl/instalacja-nextcloud-vps-docker-compose/H1.png", "Weryfikacja wersji Docker Compose w terminalu", "(max-width: 768px) 100vw, 800px" %}

```bash
docker compose version
```

## Krok 2: Przygotowanie środowiska

Potrzebujemy wydzielonego katalogu na pliki konfiguracyjne oraz dane chmury.

```bash
mkdir ~/nextcloud-server
cd ~/nextcloud-server
```

Teraz utwórzmy plik `docker-compose.yml`:

{% image "/assets/images/blog/pl/instalacja-nextcloud-vps-docker-compose/H2.png", "Otwieranie pliku docker-compose.yml w edytorze nano", "(max-width: 768px) 100vw, 800px" %}

```bash
nano docker-compose.yml
```

Wklej poniższą konfigurację. Uruchamia ona oficjalny obraz Nextcloud oraz osobną bazę danych MariaDB dla optymalnej wydajności.

```yaml
services:
  db:
    image: mariadb:latest
    restart: always
    command: --transaction-isolation=READ-COMMITTED --log-bin=binlog --binlog-format=ROW
    volumes:
      - db_data:/var/lib/mysql
    environment:
      - MYSQL_ROOT_PASSWORD=twoje_silne_haslo_root
      - MYSQL_PASSWORD=twoje_silne_haslo_bazy
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud

  app:
    image: nextcloud
    restart: always
    ports:
      - 8080:80
    links:
      - db
    volumes:
      - nextcloud_data:/var/www/html
    environment:
      - MYSQL_PASSWORD=twoje_silne_haslo_bazy
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud
      - MYSQL_HOST=db

volumes:
  db_data:
  nextcloud_data:
```

*Uwaga: Pamiętaj, aby przed zapisaniem zmienić `twoje_silne_haslo_root` i `twoje_silne_haslo_bazy` na prawdziwe, bezpieczne hasła.*

Zapisz plik i wyjdź z edytora (`CTRL+X`, następnie `Y` i `Enter`).

## Krok 3: Uruchomienie Nextcloud

Gdy konfiguracja jest gotowa, możemy uruchomić naszą chmurę. Wykonaj poniższe polecenie:

{% image "/assets/images/blog/pl/instalacja-nextcloud-vps-docker-compose/H3.png", "Uruchamianie kontenerów Nextcloud komendą docker compose up -d", "(max-width: 768px) 100vw, 800px" %}

```bash
docker compose up -d
```

Docker automatycznie pobierze wymagane obrazy i uruchomi serwisy w tle. Może to potrwać dłuższą chwilę.

Gdy proces się zakończy, otwórz przeglądarkę internetową i wejdź na adres IP swojego serwera z dopiskiem portu 8080:

`http://ip_twojego_serwera:8080`

{% image "/assets/images/blog/pl/instalacja-nextcloud-vps-docker-compose/H4.png", "Ekran pierwszej konfiguracji Nextcloud w przeglądarce", "(max-width: 768px) 100vw, 800px" %}

Pojawi się instalator Nextcloud. Utwórz konto administratora. Nextcloud automatycznie połączy się z bazą danych MariaDB, którą zdefiniowaliśmy w pliku konfiguracyjnym.

---

## Podsumowanie

Gratulacje! Twoja prywatna chmura Nextcloud działa teraz bezpiecznie w kontenerach Docker na Twoim serwerze VPS. Pliki przechowywane są wyłącznie na Twoim sprzęcie, całkowicie bez abonamentów i bez dostępu osób trzecich do Twoich danych.

W następnym kroku zalecamy umieszczenie Nextcloud za własną domeną z bezpłatnym certyfikatem HTTPS, używając [Nginx Proxy Manager](/pl/blog/konfiguracja-nginx-proxy-manager-vps/), dzięki czemu Twoja chmura będzie dostępna z każdego miejsca po szyfrowanym adresie.

Jeśli potrzebujesz niezawodnego serwera do hostowania swojej prywatnej chmury, sprawdź nasze [plany Premium VPS](/pl/premium-vps/) lub [Budget VPS](/pl/budget-vps/), które oferują szybkie dyski NVMe SSD i wysoką przepustowość sieci.