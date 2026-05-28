---
image: /assets/images/blog/instalacja-lemp-ubuntu-debian/og-image.png
title: Jak skonfigurować stos LEMP (Linux, Nginx, MariaDB, PHP) na Ubuntu i Debian
description: Kompletny przewodnik krok po kroku do instalacji nowoczesnego stosu LEMP (Linux, Nginx, MariaDB, PHP) na świeżym serwerze Ubuntu lub Debian.
date: '2026-03-25'
translationKey: setup-lemp-stack-ubuntu-debian
category: Poradniki
tags:
  - lemp
  - nginx
  - mariadb
  - php
  - ubuntu
  - debian
  - linux
  - vps
  - web server
howto:
  name: Jak zainstalować stos LEMP na Ubuntu lub Debian
  totalTime: PT15M
  yield: W pełni funkcjonujący serwer WWW zdolny do renderowania dynamicznych aplikacji PHP wspierany przez bazę danych MariaDB
  tool:
    - VPS lub dedykowany serwer z Ubuntu lub Debian
    - Klient SSH (np. terminal, PuTTY)
    - Konto użytkownika z uprawnieniami sudo
  steps:
    - name: Zainstaluj Nginx (serwer WWW)
      text: Uruchom sudo apt install nginx i zezwól na ruch przez swoją zaporę.
      url: step-1-install-nginx-the-web-server
    - name: Zainstaluj MariaDB (baza danych)
      text: Uruchom sudo apt install mariadb-server i zabezpiecz ją używając wbudowany skrypt mysql_secure_installation.
      url: step-2-install-mariadb-the-database
    - name: Zainstaluj PHP (język przetwarzania)
      text: Uruchom sudo apt install php-fpm php-mysql aby zainstalować PHP8 z FPM i rozszerzenie MySQL.
      url: step-3-install-php-the-processing-language
    - name: Skonfiguruj Nginx aby używał PHP
      text: Edytuj blok serwera Nginx aby przekazywać pliki .php do gniazda PHP-FPM.
      url: step-4-configure-nginx-to-use-php
    - name: Przetestuj stos LEMP
      text: Utwórz plik info.php aby zweryfikować że Nginx i PHP-FPM komunikują się poprawnie.
      url: step-5-test-php-processing-on-nginx
status: published
locale: pl
author:
  name: VoxiHost Team
  link: https://voxihost.pl/
contributors:
  - danielmarszalkowski
---

**Stos LEMP** to nowoczesny, wysokowydajnościowy fundament dla milionów stron internetowych na całym świecie. Jest to akronim reprezentujący cztery krytyczne kawałki oprogramowania wymagane do hostowania dynamicznych, opartych na bazie danych aplikacji (jak WordPress czy Laravel).

LEMP oznacza cztery komponenty:
- **L**inux: System operacyjny (Ubuntu lub Debian).
- **E**Nginx (wymawiane *Engine-X*): Błyskawiczny szybki serwer WWW.
- **M**ariaDB: Społecznościowy, wysoko zoptymalizowany fork MySQL, który jest standardem na nowoczesnych dystrybucjach Linuksa.
- **P**HP: Język skryptowy po stronie serwera obsługujący logikę backendową.

W porównaniu do starszego stosu LAMP (Apache), LEMP jest wysoko faworyzowany w środowiskach obsługujących duży, współbieżny ruch dzięki asynchronicznej architekturze Nginx.

## Krok 1: Zainstaluj Nginx (Serwer WWW)

Najpierw zaktualizuj swój lokalny indeks pakietów aby zapewnić pobranie najnowszych poprawek bezpieczeństwa.

```bash
sudo apt update
sudo apt upgrade -y
```

Teraz zainstaluj serwer WWW Nginx (pakiet nosi się `nginx` na systemach Debian/Ubuntu):

{% image "/assets/images/blog/instalacja-lemp-ubuntu-debian/H1.png", "Uruchamianie sudo apt install nginx -y na Ubuntu lub Debian aby rozpocząć instalację stosu LEMP", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo apt install nginx -y
```

Jeśli śledziłeś nasz [Przewodnik Konfiguracji Zapory UFW](/pl/blog/konfiguracja-ufw-ubuntu-debian/), musisz zezwolić na ruch Nginx przez swoją zaporę. Otwórz zarówno HTTP (Port 80) jak i HTTPS (Port 443):

```bash
sudo ufw allow 'Nginx Full'
```

Możesz zweryfikować że Nginx działa wpisując adres IP swojego serwera do ulubionej przeglądarki (`http://your_server_ip`). Powinieneś zobaczyć domyślną stronę *"Witaj w nginx!"*.

## Krok 2: Zainstaluj MariaDB (Baza danych)

Teraz gdy masz serwer WWW, potrzebujesz systemu bazy danych do przechowywania i zarządzania danymi aplikacji. **MariaDB** to wysoko zoptymalizowany, w pełni opensource'owy fork MySQL, który jest standardem na większości nowoczesnych dystrybucji Linuksa.

Zainstaluj serwer MariaDB:

{% image "/assets/images/blog/instalacja-lemp-ubuntu-debian/H2.png", "Uruchamianie sudo apt install mariadb-server mariadb-client -y na Ubuntu lub Debian aby zainstalować MariaDB jako część stosu LEMP", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo apt install mariadb-server mariadb-client -y
```

Po zakończeniu instalacji, baza danych działa ale jest całkowicie niezabezpieczona. Musisz ją zablokować używając wbudowany skrypt bezpieczeństwa:

{% image "/assets/images/blog/instalacja-lemp-ubuntu-debian/H3.png", "Uruchamianie sudo mysql_secure_installation na Ubuntu aby zabezpieczyć instalację MariaDB usuwając użytkowników anonimowych, wyłączając zdalne logowanie root i przeładowując tabele uprawnień", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo mysql_secure_installation
```

Zostaniesz poproszony o serię pytań aby skonfigurować profil bezpieczeństwa:
1. **Wtyczka Validate Password**: Wpisz `y` jeśli chcesz aby MariaDB aktywnie blokowała słabe hasła, lub `n` aby pominąć.
2. **Usuń użytkowników anonimowych**: Wpisz `y`.
3. **Zabroń logowanie root zdalnie**: Wpisz `y` (root powinien mieć dostęp do bazy danych tylko z *wewnątrz* serwera).
4. **Usuń bazę danych testową**: Wpisz `y`.
5. **Przeładuj tabele uprawnień**: Wpisz `y`.

Twoja baza danych jest teraz bezpieczna.

## Krok 3: Zainstaluj PHP (Język przetwarzania)

Nginx jest niesamowicie szybki w serwowaniu statycznych plików (HTML, obrazy, CSS), ale nie może przetwarzać kod dynamiczny PHP w sposób natywny jak Apache. Aby to zrobić, potrzebujemy zainstalować **PHP-FPM** (FastCGI Process Manager) wraz z rozszerzeniem MySQL.

Zainstaluj PHP8 z FPM i rozszerzeniem MySQL:

{% image "/assets/images/blog/instalacja-lemp-ubuntu-debian/H4.png", "Uruchamianie sudo apt install php-fpm php-mysql -y na Ubuntu lub Debian aby zainstalować PHP8 z FPM i rozszerzenie MySQL jako część stosu LEMP", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo apt install php-fpm php-mysql -y
```

> **Uwaga:** W zależności od dokładnej wersji Debiana/Ubuntu, `apt` automatycznie zainstaluje poprawną wersję PHP (np., `php8.1-fpm` lub `php8.3-fpm`). Zapamiętaj jaką wersję instalujesz, ponieważ będziesz jej potrzebować w następnym kroku.

## Krok 4: Skonfiguruj Nginx aby używał PHP

Musimy jawnie powiedzieć Nginx jak obsługiwać pliki PHP. Zrobimy to edytując domyślny blok serwera Nginx.

Otwórz domyślną konfigurację bloku serwera w edytorze nano:

{% image "/assets/images/blog/instalacja-lemp-ubuntu-debian/H5.png", "Tworzenie nowego pliku konfiguracyjnego bloku serwera Nginx w sites-available dla niestandardowej domeny w konfiguracji stosu LEMP na Ubuntu", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo nano /etc/nginx/sites-available/default
```

Znajdź dyrektywę `index` i dodaj `index.php` na samym początku listy:

```nginx
# Dodaj index.php przed index.html
index index.php index.html index.htm index.nginx-debian.html;
```

Następnie przewiń w dół do sekcji `location ~ \.php$` i odkomentuj (usuń `#`) istniejące linie tak aby wyglądało tak:

```nginx
location ~ \.php$ {
    include snippets/fastcgi-php.conf;
    fastcgi_pass unix:/var/run/php/php8.3-fpm.sock;
}
```

> **Ważne:** Upewnij się że ścieżka `fastcgi_pass` pasuje do wersji PHP którą zainstalowałeś w Kroku 3! Jeśli zainstalowałeś `php8.1-fpm`, użyj `php8.1-fpm.sock`.

Zapisz plik i wyjdź.

Przetestuj konfigurację Nginx pod kątem błędów składni przed włączeniem strony:

{% image "/assets/images/blog/instalacja-lemp-ubuntu-debian/H6.png", "Uruchamianie sudo nginx -t aby przetestować konfigurację bloku serwera Nginx pod kątem błędów składni przed włączeniem strony LEMP na Ubuntu", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo nginx -t
```

Jeśli zgłasza "syntax is ok", przeładuj Nginx aby zastosować zmiany:

```bash
sudo systemctl reload nginx
```

## Krok 5: Przetestuj stos LEMP

Aby udowodnić że Nginx pomyślnie przekazuje pliki PHP do PHP-FPM, stworzymy klasyczny skrypt informacyjny PHP.

Utwórz nowy plik w głównym katalogu WWW:

{% image "/assets/images/blog/instalacja-lemp-ubuntu-debian/H7.png", "Tworzenie pliku testowego info.php w /var/www/html za pomocą edytora nano w celu zweryfikowania komunikacji między Nginx a PHP-FPM w konfiguracji stosu LEMP", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo nano /var/www/html/info.php
```

Wklej standardową funkcję inicjalizacyjną PHP:

```php
<?php
phpinfo();
?>
```

Zapisz plik i wyjdź.

Otwórz przeglądarkę i nawiguj do: `http://your_server_ip/info.php`

{% image "/assets/images/blog/instalacja-lemp-ubuntu-debian/H8.png", "Strona phpinfo() wyświetlająca się w przeglądarce potwierdzająca że Nginx i PHP-FPM komunikują się poprawnie w konfiguracji stosu LEMP na Ubuntu", "(max-width: 768px) 100vw, 800px" %}

Powinieneś zobaczyć ogromną, szczegółową tabelę fioletowo-szarą opisującą dokładną konfigurację twojego serwera. To dowód że twój kompletny stos LEMP działa idealnie!

> **Krytyczne ostrzeżenie bezpieczeństwa:** Strona `info.php` ujawnia niezwykle szczegółowe informacje o konfiguracji twojego serwera. Pozostawienie tego pliku publicznie jest masowym ryzykiem bezpieczeństwa. Gdy potwierdzisz że wszystko działa, **usuń plik natychmiast**:
```bash
sudo rm /var/www/html/info.php
```

Gratulacje! Pomyślnie zbudowałeś sprawdzony, przetestowany i solidny fundament do hostowania dynamicznych aplikacji na Linuksie. Czy masz ciężki sklep e-commerce, forum czy blog o dużym ruchu gotowy do uruchomienia? Połącz swój nowy stos LEMP z jednym z naszych środowisk [Premium VPS](/pl/premium-vps/), zainstaluj [bezpłatne SSL przez Let's Encrypt](/pl/blog/certyfikat-ssl-letsencrypt-ubuntu/) i wdróż swoją ostateczną aplikację do świata.