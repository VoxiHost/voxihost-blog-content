---
image: /assets/images/blog/pl/konfiguracja-nginx-proxy-manager-vps/og-image.png
title: "Jak skonfigurować Nginx Proxy Manager na VPS"
description: "Dowiedz się, jak łatwo hostować wiele stron, podpinać domeny i zarządzać certyfikatami SSL za pomocą Nginx Proxy Manager i Docker Compose na serwerze VPS."
date: '2026-06-02'
translationKey: setup-nginx-proxy-manager-vps
locale: pl
category: Poradniki
tags:
  - nginx
  - nginx-proxy-manager
  - docker
  - docker-compose
  - vps
  - ssl
status: published
author:
  name: VoxiHost Team
  link: https://voxihost.pl/
contributors:
  - sl0ikkk
howto:
  name: "Jak skonfigurować Nginx Proxy Manager na VPS"
  totalTime: "PT15M"
  yield: "Serwer VPS z działającym Nginx Proxy Manager i dostępem do webowego panelu zarządzania"
  tool:
    - "Serwer VPS (np. Ubuntu lub Debian)"
    - "Klient SSH (np. terminal, PuTTY)"
    - "Zainstalowany Docker i Docker Compose"
  steps:
    - name: "Czym jest Nginx Proxy Manager?"
      text: "Zrozumienie reverse proxy."
      url: "krok-1-czym-jest-nginx-proxy-manager"
    - name: "Instalacja NPM"
      text: "Uruchomienie przez Docker Compose."
      url: "krok-2-instalacja-npm"
    - name: "Dostęp do Panelu"
      text: "Logowanie i pierwsza konfiguracja."
      url: "krok-3-dostep-do-panelu"
---

## Wstęp

Kiedy hostujesz kilka aplikacji na jednym serwerze VPS (np. WordPressa, Nextcloud i panel do gier), szybko napotykasz problem: wszystkie nie mogą korzystać z domyślnego portu 80. Kończy się to dostępem do usług przez niewygodne porty, np. `http://twoje-ip:8080`.

Aby przypisać prawdziwe domeny (`chmura.twojadomena.pl`) do tych aplikacji i zabezpieczyć je darmowym certyfikatem HTTPS, potrzebujesz tzw. **Reverse Proxy**. **Nginx Proxy Manager (NPM)** to najprostszy i najbardziej estetyczny sposób na jego konfigurację.

> **Wymagania wstępne:** Potrzebujesz serwera VPS z systemem Ubuntu lub Debian, dostępu SSH z kontem użytkownika z uprawnieniami `sudo` oraz zainstalowanego Dockera z Docker Compose. Jeśli jeszcze tego nie zrobiłeś, sprawdź nasz [poradnik konfiguracji Docker Compose](/pl/blog/jak-skonfigurowac-docker-compose/).

## Krok 1: Czym jest Nginx Proxy Manager?

Nginx Proxy Manager to graficzny interfejs (panel webowy) sterujący serwerem Nginx. Zamiast pisać skomplikowane pliki konfiguracyjne ręcznie w terminalu, otrzymujesz przejrzysty panel. Wystarczy kilka kliknięć, aby przekierować ruch z konkretnej domeny na wybrany kontener Docker i wygenerować darmowy certyfikat Let's Encrypt.

## Krok 2: Instalacja NPM

NPM działa jako kontener, więc jego uruchomienie to kwestia sekund, o ile posiadasz już Docker Compose.

Stwórzmy nowy folder dla NPM:

{% image "/assets/images/blog/pl/konfiguracja-nginx-proxy-manager-vps/H1.png", "Tworzenie katalogu npm", "(max-width: 768px) 100vw, 800px" %}

```bash
mkdir ~/npm-server
cd ~/npm-server
```

Utwórz plik `docker-compose.yml`:

{% image "/assets/images/blog/pl/konfiguracja-nginx-proxy-manager-vps/H2.png", "Edycja pliku docker-compose.yml", "(max-width: 768px) 100vw, 800px" %}

```bash
nano docker-compose.yml
```

Wklej domyślną, oficjalną konfigurację:

```yaml
services:
  app:
    image: 'jc21/nginx-proxy-manager:latest'
    restart: unless-stopped
    ports:
      - '80:80' # Publiczny port HTTP
      - '81:81' # Port panelu admina
      - '443:443' # Publiczny port HTTPS
    volumes:
      - ./data:/data
      - ./letsencrypt:/etc/letsencrypt
```

Zapisz i wyjdź. Zauważ, że NPM przejmuje porty `80` i `443` na całym serwerze VPS. Będzie on teraz przechwytywał cały ruch z internetu i decydował, do której wewnętrznej aplikacji go skierować!

Uruchom środowisko:

{% image "/assets/images/blog/pl/konfiguracja-nginx-proxy-manager-vps/H3.png", "Uruchamianie Nginx Proxy Manager komendą docker compose up", "(max-width: 768px) 100vw, 800px" %}

```bash
docker compose up -d
```

## Krok 3: Dostęp do Panelu

Gdy tylko kontenery wystartują, możesz wejść do panelu administratora za pomocą przeglądarki, wchodząc na port `81`.

`http://ip_twojego_serwera:81`

{% image "/assets/images/blog/pl/konfiguracja-nginx-proxy-manager-vps/H4.png", "Ekran logowania do Nginx Proxy Manager", "(max-width: 768px) 100vw, 800px" %}

**Domyślne dane logowania:**
- Email: `admin@example.com`
- Hasło: `changeme`

Zaraz po pierwszym zalogowaniu system wymusi na Tobie zmianę tych danych ze względów bezpieczeństwa.

Możesz teraz przejść do zakładki **"Proxy Hosts"** -> **"Add Proxy Host"**, aby bezboleśnie przypisać swoje domeny do dowolnych portów działających na VPS. Koniec z edycją plików Nginx w terminalu!

---

## Podsumowanie

Nginx Proxy Manager działa teraz na Twoim VPS i jest w pełni dostępny przez przeglądarkę. Możesz przekierowywać dowolną liczbę usług na własne domeny i wystawiać bezpłatne certyfikaty Let's Encrypt w zaledwie kilka kliknięć — bez ręcznej edycji plików konfiguracyjnych Nginx.

Połącz NPM z innymi usługami działającymi na tym samym VPS-ie, takimi jak [WordPress](/pl/blog/instalacja-wordpress-docker-compose/) czy [Nextcloud](/pl/blog/instalacja-nextcloud-vps-docker-compose/), aby zbudować kompletny, produkcyjny stos self-hosted.

Jeśli szukasz niezawodnego serwera pod taki stos, sprawdź nasze [plany Premium VPS](/pl/premium-vps/) lub [Budget VPS](/pl/budget-vps/) — oba oferują szybkie dyski NVMe SSD i port sieciowy o dużej przepustowości.