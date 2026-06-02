---
image: /assets/images/blog/pl/aktualizacja-wersji-php-linux/og-image.png
title: Jak zarządzać i aktualizować wersje PHP na Linux (Ubuntu i Debian)
description: Kompletny przewodnik do bezpiecznej instalacji, aktualizacji i zarządzania wieloma wersjami PHP i PHP-FPM na swoim serwerze Linux używając oficjalnego PPA Ondřeja Surýego.
date: '2026-03-25'
updated: '2026-06-02'
translationKey: manage-update-php-versions-linux
category: Poradniki
tags:
  - php
  - php-fpm
  - ubuntu
  - debian
  - linux
  - vps
  - server administration
  - nginx
  - apache
howto:
  name: Jak zainstalować i zarządzać wieloma wersjami PHP na Ubuntu/Debian
  totalTime: PT10M
  yield: Serwer zdolny do uruchamiania różnych wersji PHP jednocześnie, z możliwością przełączania domyślnych w locie
  tool:
    - VPS lub dedykowany serwer z Ubuntu lub Debian
    - Klient SSH (np. terminal, PuTTY)
    - Konto użytkownika z uprawnieniami sudo
  steps:
    - name: Dodaj repozytorium PPA Ondřeja Surýego
      text: Uruchom sudo add-apt-repository ppa:ondrej/php aby uzyskać dostęp do wszystkich najnowszych wersji PHP.
      url: krok-1-dodaj-zaufane-repozytorium-php
    - name: Zainstaluj konkretną wersję PHP
      text: Uruchom sudo apt install php8.3 php8.3-fpm php8.3-mysql aby zainstalować dokładną wersję której potrzebujesz.
      url: krok-2-zainstaluj-wiele-wersji-php
    - name: Przełącz domyślną wersję CLI
      text: Uruchom sudo update-alternatives --config php aby wybrać która wersja PHP odpowiada na polecenie 'php' w terminalu.
      url: krok-3-przelacz-domyslna-wersje-wiersza-polecen-cli
    - name: Skonfiguruj swój serwer WWW
      text: Wskaż Nginx lub Apache do konkretnego gniazda PHP-FPM którego chcesz użyć dla swojej strony internetowej.
      url: krok-4-powiedz-swojemu-serwerowi-www-ktorej-wersji-uzyc
faq:
  - question: "Dlaczego powinienem dodać repozytorium Ondřeja Surégo?"
    answer: "Domyślne repozytoria Ubuntu/Debiana zawierają tylko jedną, przestarzałą wersję PHP z dnia premiery systemu. Repozytorium Ondřeja Surégo (oficjalnego dewelopera Debiana) to standard branżowy, który umożliwia bezpieczne instalowanie wielu nowszych wersji PHP obok siebie."
  - question: "Czy różne wersje PHP mogą działać na serwerze jednocześnie?"
    answer: "Tak. Każda wersja PHP działa jako oddzielna usługa systemowa w tle (np. <code>php8.1-fpm</code> oraz <code>php8.3-fpm</code>) i nasłuchuje na osobnym gnieździe Unix socket, przez co nie wchodzą sobie w drogę."
  - question: "Jak zainstalować rozszerzenie PHP dla konkretnej wersji?"
    answer: "Musisz wskazać wersję PHP w nazwie pakietu rozszerzenia. Na przykład, aby zainstalować rozszerzenie curl dla PHP 8.3, należy uruchomić polecenie <code>sudo apt install php8.3-curl</code>."
  - question: "Jak zmienić wersję PHP używaną przez serwer Nginx?"
    answer: "Nginx komunikuje się z PHP za pomocą socketów FPM. Aby zmienić wersję, edytuj plik konfiguracyjny swojej strony i zmień ścieżkę w dyrektywie <code>fastcgi_pass</code> (np. z <code>/var/run/php/php8.1-fpm.sock</code> na <code>/var/run/php/php8.3-fpm.sock</code>), a potem przeładuj Nginx."
  - question: "Dlaczego moje zmiany wprowadzone w php.ini nie działają?"
    answer: "Po edycji pliku <code>/etc/php/VERSION/fpm/php.ini</code> musisz zrestartować właściwą usługę FPM (np. <code>sudo systemctl restart php8.3-fpm</code>). Upewnij się też, że edytujesz plik konfiguracyjny FPM, a nie CLI (wersji konsolowej)."
status: published
locale: pl
author:
  name: VoxiHost Team
  link: https://voxihost.pl/
contributors:
  - danielmarszalkowski
---

Gdy instalujesz PHP bezpośrednio z domyślnych repozytoriów Ubuntu lub Debian przez `apt install php`, jesteś zablokowany w pojedynczej wersji PHP która została dostarczona z systemem operacyjnym w dniu jego wydania.

Jeśli aktualizujesz aplikację jak WordPress, Laravel, czy Magento, może nagle wymagać PHP 8.2 lub 8.3. Z drugiej strony, jeśli migrujesz starą aplikację, może agresywnie awariować na czymkolwiek nowszym niż PHP 7.4.

Aby to rozwiązać, potrzebujesz możliwości instalacji wielu wersji PHP jednocześnie, uruchamiania ich obok siebie używając **PHP-FPM**, i mówienia swojemu serwerowi WWW dokładnie której wersji użyć dla której strony internetowej.

## Krok 1: Dodaj zaufane repozytorium PHP

Ponieważ domyślne repozytoria OS zawierają tylko jedną statyczną wersję PHP, musimy dodać zaufane repozytorium stron trzecich.

Przez prawie dekadę, globalny standard dla tego był utrzymywany przez **Ondřeja Surýego**, wysoko szanowanego dewelopera Debiana który zarządza oficjalnymi pakietami PHP.

Najpierw zainstaluj pakiet właściwości oprogramowania (jeśli nie jest już tam):

{% image "/assets/images/blog/pl/aktualizacja-wersji-php-linux/H1.png", "Uruchamianie sudo apt install software-properties-common ca-certificates lsb-release apt-transport-https aby przygotować Ubuntu do dodania repozytorium PHP", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo apt update
sudo apt install software-properties-common ca-certificates lsb-release apt-transport-https -y
```

Teraz dodaj repozytorium:

**Dla Ubuntu:**

{% image "/assets/images/blog/pl/aktualizacja-wersji-php-linux/H2.png", "Uruchamianie sudo add-apt-repository ppa:ondrej/php na Ubuntu aby dodać repozytorium PHP Ondřeja Surýego z wieloma wersjami PHP", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo add-apt-repository ppa:ondrej/php
sudo apt update
```

**Dla Debian:**
```bash
sudo curl -sSLo /usr/share/keyrings/deb.sury.org-php.gpg https://packages.sury.org/php/apt.gpg
sudo sh -c 'echo "deb [signed-by=/usr/share/keyrings/deb.sury.org-php.gpg] https://packages.sury.org/php/ $(lsb_release -sc) main" > /etc/apt/sources.list.d/php.list'
sudo apt update
```

## Krok 2: Zainstaluj wiele wersji PHP

Z repozytorium dodanym, masz teraz dostęp do dosłownie każdej wersji PHP (od ultra-prastarej 5.6 do najnowocześniejszej 8.3 i dalej).

Możesz bezpiecznie zainstalować wiele wersji. Nie nadpiszą się ani nie będą ze sobą konfliktować.

Zainstalujmy **PHP 8.3** (dla nowoczesnej aplikacji) i **PHP 8.1** (dla starszej aplikacji):

{% image "/assets/images/blog/pl/aktualizacja-wersji-php-linux/H3.png", "Instalowanie PHP 8.3 i PHP 8.1 z php-fpm obok siebie na Ubuntu używając apt do zarządzania wieloma wersjami PHP", "(max-width: 768px) 100vw, 800px" %}

```bash
# Zainstaluj PHP 8.3 i jego procesor FPM
sudo apt install php8.3 php8.3-fpm php8.3-mysql php8.3-xml php8.3-curl -y

# Zainstaluj PHP 8.1 i jego procesor FPM 
sudo apt install php8.1 php8.1-fpm php8.1-mysql php8.1-xml php8.1-curl -y
```

Po zainstalowaniu, obie usługi PHP-FPM automatycznie zaczną działać w tle niezależnie. Możesz to zweryfikować sprawdzając ich status:

{% image "/assets/images/blog/pl/aktualizacja-wersji-php-linux/H4.png", "Uruchamianie sudo systemctl status php8.3-fpm aby zweryfikować że PHP-FPM działa po instalacji na Ubuntu", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo systemctl status php8.3-fpm
sudo systemctl status php8.1-fpm
```

## Krok 3: Przełącz domyślną wersję wiersza poleceń (CLI)

Jeśli wpiszesz `php -v` w terminalu teraz, wyświetli wersję która była zainstalowana *ostatnia*.

{% image "/assets/images/blog/pl/aktualizacja-wersji-php-linux/H5.png", "Uruchamianie php -v w terminalu aby sprawdzić obecną domyślną wersję PHP na Ubuntu Linux", "(max-width: 768px) 100vw, 800px" %}

Jeśli używasz narzędzi terminalowych jak `composer` czy WP-CLI, musisz zapewnić że domyślna wersja wiersza poleceń PHP pasuje do twojego projektu.

Aby łatwo przełączać domyślną wersję CLI, Linux używa narzędzia o nazwie `update-alternatives`:

{% image "/assets/images/blog/pl/aktualizacja-wersji-php-linux/H6.png", "Uruchamianie sudo update-alternatives --config php aby interaktywnie wybrać domyślną wersję CLI PHP na Ubuntu", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo update-alternatives --config php
```

Zobaczysz wynik podobny do tego:
```text
There are 2 choices for alternative php (providing /usr/bin/php).

  Selection    Path             Priority   Status
------------------------------------------------------------
* 0            /usr/bin/php8.3   83        auto mode
  1            /usr/bin/php8.1   81        manual mode
  2            /usr/bin/php8.3   83        manual mode

Press <enter> to keep the current choice[*], or type selection number:
```

Po prostu wpisz numer `Selection` który chcesz (np., `1` dla PHP 8.1) i naciśnij Enter. Zweryfikuj zmianę wpisując `php -v` ponownie.

## Krok 4: Powiedz swojemu serwerowi WWW której wersji użyć

Wersja CLI nie ma wpływu na twój serwer WWW. [Nginx](/pl/blog/instalacja-nginx-ubuntu-debian/) i [Apache](/pl/blog/instalacja-apache-ubuntu-debian/) nie obchodzi jaka jest twoja domyślna terminalowa PHP; obchodzi im tylko które **gniazdo FPM** które jawnie wskażesz im w swoich plikach konfiguracyjnych.

### Jeśli używasz Nginx (LEMP Stack):

Otwórz swój Blok Serwera Nginx (np., `/etc/nginx/sites-available/default`) aby znaleźć blok lokalizacji PHP. Zmień linię `fastcgi_pass` aby wskazywała na dokładną wersję gniazda której chcesz użyć dla tej konkretnej strony internetowej.

Aby użyć **PHP 8.3**:
```nginx
location ~ \.php$ {
    include snippets/fastcgi-php.conf;
    fastcgi_pass unix:/var/run/php/php8.3-fpm.sock;
}
```

Aby użyć **PHP 8.1** na innym Bloku Serwera, po prostu powiąż je z gniazdem 8.1:
```nginx
location ~ \.php$ {
    include snippets/fastcgi-php.conf;
    fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
}
```

Zapisz plik i przeładuj Nginx:
```bash
sudo systemctl reload nginx
```

### Jeśli używasz Apache (LAMP Stack):

Apache obsługuje FPM nieco inaczej przez `a2enconf`.

Najpierw wyłącz ogólny moduł PHP:
```bash
sudo a2dismod php*
```

Teraz włącz konkretną konfigurację PHP-FPM której chcesz użyć globalnie:
```bash
# Włącz wymagane moduły jeśli jeszcze nie masz
sudo a2enmod proxy_fcgi setenvif

# Włącz PHP 8.3 FPM
sudo a2enconf php8.3-fpm

# LUB całkowicie przełącz na PHP 8.1 FPM
# sudo a2disconf php8.3-fpm
# sudo a2enconf php8.1-fpm
```

Zrestartuj Apache aby zmiany weszły w życie:
```bash
sudo systemctl restart apache2
```

## Krok 5: Edytowanie właściwego php.ini

Gdy musisz zwiększyć swoje `upload_max_filesize` czy `memory_limit`, musisz edytować plik `php.ini` który koreluje dokładnie do wersji i środowiska którego używasz.

Dla Nginx/Apache obsługującego ruch WWW przez PHP 8.3:
```bash
sudo nano /etc/php/8.3/fpm/php.ini
```

Dla Wiersza Poleceń (CLI) obsługującego PHP 8.3:
```bash
sudo nano /etc/php/8.3/cli/php.ini
```

Za każdym razem gdy edytujesz plik `fpm/php.ini`, musisz zrestartować tę konkretną usługę FPM aby nowe limity pamięci weszły w życie:
```bash
sudo systemctl restart php8.3-fpm
```

Hostowanie wielu aplikacji z drastycznie różnymi wymaganiami dziedzictwa i nowoczesności PHP wymaga serwera z RAM i CPU aby płynnie obsługiwać odmienne pule demonów FPM. Chwyć masowo niezawodny [Premium VPS](/pl/premium-vps/), zainstaluj tyle wersji PHP ilu wymagają twoi klienci i uruchamiaj je jednocześnie bez wycierania potu.