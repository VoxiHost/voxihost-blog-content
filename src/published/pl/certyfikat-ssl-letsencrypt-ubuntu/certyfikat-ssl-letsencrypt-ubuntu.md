---
image: /assets/images/blog/pl/certyfikat-ssl-letsencrypt-ubuntu/og-image.png
title: 'Jak skonfigurować SSL z Let''s Encrypt i Certbot na Ubuntu i Debian: Kompletny przewodnik'
description: Kompletny przewodnik dla początkujących do zabezpieczania serwera Nginx lub Apache z bezpłatnymi certyfikatami SSL/TLS od Let's Encrypt używając Certbota na Ubuntu i Debian.
date: '2026-03-25'
translationKey: setup-ssl-letsencrypt-certbot
category: Poradniki
tags:
  - ssl
  - lets encrypt
  - certbot
  - https
  - ubuntu
  - debian
  - nginx
  - apache
  - security
  - linux
  - vps
howto:
  name: Jak skonfigurować SSL z Let's Encrypt i Certbot na Ubuntu i Debian
  totalTime: PT10M
  yield: W pełni zabezpieczony serwer WWW z automatycznie odnawianymi certyfikatami SSL/TLS i wymuszonymi połączeniami HTTPS
  tool:
    - VPS lub dedykowany serwer z Ubuntu lub Debian
    - Zainstalowany serwer WWW (Nginx lub Apache) z skonfigurowanym blokiem serwera/Wirtualnym Hostem
    - Zarejestrowana nazwa domeny wskazująca na publiczny adres IP serwera
    - Konto użytkownika z uprawnieniami sudo
  steps:
    - name: Zainstaluj Certbot
      text: Uruchom sudo apt install certbot i albo python3-certbot-nginx lub python3-certbot-apache.
      url: krok-1-zainstaluj-certbot
    - name: Potwierdź ustawienia zapory
      text: Zapewnij że Twoja zapora zezwala na ruch HTTPS (port 443) używając ufw allow 'Nginx Full' lub 'Apache Full'.
      url: krok-2-potwierdz-ustawienia-zapory
    - name: Uzyskaj i zainstaluj certyfikat SSL
      text: Uruchom sudo certbot --nginx -d twoja_domena.com lub sudo certbot --apache -d twoja_domena.com, aby uzyskać i zainstalować certyfikaty SSL.
      url: krok-3-uzyskaj-i-zainstaluj-certyfikaty-ssl
    - name: Zweryfikuj automatyczne odnawianie
      text: Sprawdź czy timer certbot.timer jest aktywny, aby certyfikaty były odnawiane przed wygaśnięciem.
      url: krok-4-zweryfikuj-automatyczne-odnawianie
status: published
locale: pl
author:
  name: VoxiHost Team
  link: https://voxihost.pl/
contributors:
  - danielmarszalkowski
---

W nowoczesnym internecie, serwowanie strony internetowej przez zwykłe HTTP (port 80) nie jest już akceptowalne. Przeglądarki wyświetlą ostrzeżenia "Niebezpieczne", wyszukiwarki będą karać strony bez SSL, a użytkownicy stają się coraz bardziej świadomi bezpieczeństwa.

Dzięki projektowi **Let's Encrypt** i ich automatycznemu klientowi **Certbot**, możesz uzyskać enterprise'owej jakości, kryptograficznie bezpieczne certyfikaty SSL/TLS zupełnie za darmo w mniej niż dwie minuty.

## Wymagania wstępne

Zanim zaczniesz, musisz mieć dwie rzeczy na miejscu:
1. **Zainstalowany serwer WWW** z skonfigurowanym blokiem serwera/Wirtualnym Hostem dla Twojej domeny. Jeśli jeszcze tego nie zrobiłeś, sprawdź nasze przewodniki [instalacji Nginx](/pl/blog/instalacja-nginx-ubuntu-debian/) lub [instalacji Apache](/pl/blog/instalacja-apache-ubuntu-debian/).
2. **Prawidłowe ustawienia DNS** - Twoja domena (np., `twoja_domena.com` i `www.twoja_domena.com`) musi mieć aktywne rekordy `A` wskazujące na publiczny adres IP Twojego serwera. Certbot nie będzie w stanie zweryfikować Twojej domeny w przeciwnym razie.

## Krok 1: Zainstaluj Certbot

Certbot to narzędzie który łączy się z serwerami Let's Encrypt, weryfikuje że kontrolujesz domenę i automatycznie pobiera oraz instaluje certyfikaty.

Najpierw zaktualizuj indeks pakietów:

```bash
sudo apt update
```

Teraz zainstaluj Certbot wraz z wtyczką dla swojego serwera WWW:

**Dla Nginx:**

{% image "/assets/images/blog/pl/certyfikat-ssl-letsencrypt-ubuntu/H1.png", "Uruchamianie sudo apt install certbot python3-certbot-nginx -y na Ubuntu/Debian - wynik terminala", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo apt install certbot python3-certbot-nginx -y
```

**Dla Apache:**
```bash
sudo apt install certbot python3-certbot-apache -y
```

## Krok 2: Potwierdź ustawienia zapory

Certbot musi komunikować się przez HTTP (port 80) aby zweryfikować, że kontrolujesz domenę, zanim wystawi certyfikat SSL. Twoja zapora musi zezwalać na ruch HTTPS (port 443).

Jeśli śledziłeś nasz [Przewodnik Konfiguracji Zapory UFW](/pl/blog/konfiguracja-ufw-ubuntu-debian/), upewnij się że zezwoliłeś na odpowiedni ruch.

Sprawdź obecne reguły zapory:

**Dla Nginx:**
```bash
sudo ufw status
```

Jeśli widzisz tylko `Nginx HTTP` dozwolony, musisz uaktualnić profil:

```bash
sudo ufw allow 'Nginx Full'
sudo ufw delete allow 'Nginx HTTP'
```

**Dla Apache:**
```bash
sudo ufw status
```

Jeśli widzisz tylko `Apache` dozwolony, uaktualnij profil:

```bash
sudo ufw allow 'Apache Full'
sudo ufw delete allow 'Apache'
```

## Krok 3: Uzyskaj i zainstaluj certyfikaty SSL

Teraz uzyskaj i instaluj certyfikat SSL dla swojej domeny. Użyj polecenia Certbot z flagą `-d` (tryb odłączony), aby uzyskać certyfikat i jednocześnie skonfigurować swój serwer WWW.

**Uruchom Certbot dla Nginx:**

{% image "/assets/images/blog/pl/certyfikat-ssl-letsencrypt-ubuntu/H2.png", "Uruchamianie sudo certbot --nginx -d twoja_domena.com -d www.twoja_domena.com na Ubuntu/Debian - wynik terminala", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo certbot --nginx -d twoja_domena.com -d www.twoja_domena.com
```

**Dla Apache:**
```bash
sudo certbot --apache -d twoja_domena.com
```

Certbot połączy się z serwerami Let's Encrypt, zweryfikuje Twoją domenę, pobiera certyfikaty i automatycznie modyfikuje konfigurację Nginx lub Apache aby włączyć HTTPS.

Po zakończeniu, zobaczysz komunikat podobny do tego:
```
Successfully received certificate.
Certificate is saved at: /etc/letsencrypt/live/twoja_domena.com/fullchain.pem
Key is saved at:         /etc/letsencrypt/live/twoja_domena.com/privkey.pem
```

## Krok 4: Zweryfikuj automatyczne odnawianie

Let's Encrypt certyfikaty są ważne tylko przez 90 dni. Certbot instaluje timer systemowy który automatycznie odnawia certyfikaty zanim wygaśnięciem. Możesz zweryfikować czy timer jest aktywny uruchamiając:

{% image "/assets/images/blog/pl/certyfikat-ssl-letsencrypt-ubuntu/H3.png", "Uruchamianie sudo systemctl status certbot.timer na Ubuntu/Debian - wynik terminala", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo systemctl status certbot.timer
```

Powinieneś zobaczyć `"Active: active (waiting)"`.

Możesz też sprawdzić datę następnego odnawiania:

```bash
sudo systemctl list-timers certbot.timer
```

Aby ręcznie przetestować proces odnawiania bez czekania na następne odnawianie:

```bash
sudo certbot renew --dry-run
```

Jeśli test przejdzie bez błędów, wszystko działa poprawnie!

## Krok 4: Wymuś połączenia HTTPS

Twój serwer WWW jest teraz skonfigurowany tak aby obsługiwał bezpieczne połączenia HTTPS. Przeglądarki automatycznie przekierują ruch HTTP na HTTPS.

Otwórz przeglądarkę i nawiguj do: `https://twoja_domena.com`

Powinieneś zobaczyć ikonę kłódki obok paska adresu URL, co potwierdza że Twoja strona jest w pełni zabezpieczona!

Gratulacje! Pomyślnie zabezpieczyłeś swój serwer WWW za pomocą bezpłatnych, enterprise'owych certyfikatów SSL które automatycznie się odnawiają. Czy masz ciężki sklep e-commerce, forum czy blog o dużym ruchu gotowy do uruchomienia? Wdróż swoje bezpieczne aplikacje na niezwykle wydajnym [Premium VPS](/pl/premium-vps/), zainstaluj certyfikaty SSL i uruchom swoją ostateczną stronę do świata.