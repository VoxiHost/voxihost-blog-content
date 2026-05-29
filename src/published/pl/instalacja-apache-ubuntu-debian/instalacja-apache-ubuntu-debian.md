---
image: /assets/images/blog/pl/instalacja-apache-ubuntu-debian/og-image.png
title: 'Jak zainstalować Apache na Ubuntu i Debian: Kompletny przewodnik serwera'
description: Kompletny przewodnik krok po kroku do instalacji serwera WWW Apache na Ubuntu i Debian. Naucz się konfigurować UFW, zarządzać usługą apache2 i tworzyć Wirtualne Hosty.
date: '2026-03-25'
translationKey: install-apache-ubuntu-debian
category: Poradniki
tags:
  - apache
  - apache2
  - ubuntu
  - debian
  - web server
  - linux
  - vps
  - server administration
  - virtual host
  - ufw
howto:
  name: Jak zainstalować Apache na Ubuntu i Debian
  totalTime: PT15M
  yield: W pełni funkcjonalny serwer WWW Apache działający na Ubuntu lub Debian, gotowy do hostowania stron internetowych
  tool:
    - VPS lub dedykowany serwer z Ubuntu lub Debian
    - Klient SSH (np. terminal, PuTTY)
    - Konto użytkownika z uprawnieniami sudo
  steps:
    - name: Zaktualizuj indeks pakietów i zainstaluj Apache
      text: Uruchom sudo apt update a następnie sudo apt install apache2 aby zainstalować serwer WWW.
      url: step-1-install-apache
    - name: Dostosuj zaporę
      text: Uruchom sudo ufw allow 'Apache Full' aby otworzyć ruch HTTP (80) i HTTPS (443).
      url: step-2-adjust-the-firewall
    - name: Sprawdź swój serwer WWW
      text: Nawiguj przeglądarką do adresu IP swojego serwera aby zobaczyć domyślną stronę Apache.
      url: step-3-check-your-web-server
    - name: Zarządzaj procesem Apache
      text: Użyj poleceń systemctl (start, stop, restart, reload) aby kontrolować usługę apache2.
      url: step-4-manage-the-apache-process
    - name: Skonfiguruj Wirtualny Host
      text: Utwórz niestandardowy plik konfiguracyjny w /etc/apache2/sites-available/ aby hostować swoje unikalne domeny.
      url: step-5-set-up-virtual-hosts
status: published
locale: pl
author:
  name: VoxiHost Team
  link: https://voxihost.pl/
contributors:
  - danielmarszalkowski
---

Apache to legendarny, sprawdzony w boju serwer WWW który pomógł zbudować internet. Pozostaje jednym z najczęściej używanych serwerów WWW na świecie, znany ze swojej mocy, niezawodności i ogromnego ekosystemu modułów (jak `mod_rewrite` szeroko używany przez WordPress i inne platformy CMS).

Na systemach Ubuntu i Debian, pakiet i usługa Apache są określane jako `apache2`. Konfiguracja zajmuje tylko kilka minut. Oto jak to zrobić.

## Krok 1: Zainstaluj Apache

Apache to podstawowy pakiet dostępny bezpośrednio z domyślnych repozytoriów oprogramowania Ubuntu i Debian, co oznacza że możemy go zainstalować czysto używając menedżera pakietów `apt`.

Jak zawsze przed rozpoczęciem, zaktualizuj swój lokalny indeks pakietów aby zapewnić otrzymanie najnowszej dostępnej wersji:

```bash
sudo apt update
```

Zainstaluj pakiet `apache2`:

{% image "/assets/images/blog/pl/instalacja-apache-ubuntu-debian/H1.png", "Uruchamianie sudo apt install apache2 -y na Ubuntu aby zainstalować serwer WWW Apache", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo apt install apache2 -y
```

Podobnie jak Nginx, proces `apt` automatycznie konfiguruje usługę Apache tak aby uruchamiała się automatycznie natychmiast po instalacji i zapewnia że działa automatycznie gdy serwer startuje.

## Krok 2: Dostosuj zaporę

Przed przetestowaniem Apache, musisz zmodyfikować reguły zapory aby zezwolić na zewnętrzny dostęp do domyślnych portów WWW. Jeśli śledziłeś nasz polecany [Przewodnik Konfiguracji Zapory UFW](/pl/blog/konfiguracja-ufw-ubuntu-debian/), użyjesz UFW aby bezproblemowo to obsłużyć.

Apache rejestruje kilka profili aplikacji z UFW przy instalacji. Wyświetl je wpisując:

{% image "/assets/images/blog/pl/instalacja-apache-ubuntu-debian/H2.png", "Uruchamianie sudo ufw app list na Ubuntu pokazujące profile aplikacji zapory Apache", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo ufw app list
```

Powinieneś zobaczyć:
```text
Available applications:
  Apache
  Apache Full
  Apache Secure
  OpenSSH
```

- **Apache**: Otwiera tylko port 80 (normalny, niezaszyfrowany ruch HTTP).
- **Apache Secure**: Otwiera tylko port 443 (zaszyfrowany ruch HTTP TLS/SSL).
- **Apache Full**: Otwiera zarówno port 80 jak i port 443.

Mocno zalecamy zezwolenie na oba porty od razu aby być gotowym do skonfigurowania szyfrowania SSL/TLS (HTTPS) w przyszłości.

{% image "/assets/images/blog/pl/instalacja-apache-ubuntu-debian/H3.png", "Uruchamianie sudo ufw allow 'Apache Full' aby otworzyć porty HTTP i HTTPS przez zaporę UFW na Ubuntu", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo ufw allow 'Apache Full'
```

Sprawdź status aby zweryfikować zmianę:

```bash
sudo ufw status
```

## Krok 3: Sprawdź swój serwer WWW

W tym momencie Apache działa i twoja zapora zezwala na ruch. Upewnijmy się że wszystko działa prosząc Apache o stronę.

Skieruj swoją ulubioną przeglądarkę internetową na publiczny adres IP swojego serwera. Jeśli musisz sobie przypomnieć adres IP serwera szybko z terminala:

{% image "/assets/images/blog/pl/instalacja-apache-ubuntu-debian/H4.png", "Uruchamianie curl -4 icanhazip.com na Ubuntu aby wyświetlić publiczny adres IP serwera w terminalu", "(max-width: 768px) 100vw, 800px" %}

```bash
curl -4 icanhazip.com
```

Wpisz ten adres IP w pasek URL przeglądarki:
`http://your_server_ip`

{% image "/assets/images/blog/pl/instalacja-apache-ubuntu-debian/H5.png", "Domyślna strona powitalna Apache2 w przeglądarce potwierdzająca pomyślną instalację na serwerze Ubuntu", "(max-width: 768px) 100vw, 800px" %}

Powinieneś natychmiast zobaczyć domyślną **Stronę Domyślną Ubuntu Apache2** (lub jej odpowiednik dla Debiana). Ta strona potwierdza że oprogramowanie serwera WWW jest poprawnie zainstalowane i dostępne przez internet w sposób bezpieczny.

## Krok 4: Zarządzaj procesem Apache

Zarządzanie Apache opiera się na systemd poprzez polecenie `systemctl`. Oto podstawowe polecenia zarządzania których będziesz potrzebować do codziennej administracji serwerem WWW nad usługą `apache2`.

Aby całkowicie zatrzymać serwer WWW:

{% image "/assets/images/blog/pl/instalacja-apache-ubuntu-debian/H6.png", "Uruchamianie sudo systemctl stop apache2 aby zatrzymać serwer WWW Apache na Ubuntu", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo systemctl stop apache2
```

Aby uruchomić serwer WWW gdy jest zatrzymany:

{% image "/assets/images/blog/pl/instalacja-apache-ubuntu-debian/H7.png", "Uruchamianie sudo systemctl start apache2 aby uruchomić serwer WWW Apache na Ubuntu lub Debian", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo systemctl start apache2
```

Aby zatrzymać i uruchomić usługę w jednym poleceniu, użyj restart (przydatne jeśli ciężko zepsułeś zależności konfiguracyjne):

{% image "/assets/images/blog/pl/instalacja-apache-ubuntu-debian/H8.png", "Uruchamianie sudo systemctl restart apache2 aby zrestartować Apache i przeładować konfigurację na Ubuntu", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo systemctl restart apache2
```

Jednakże, jeśli po prostu zmieniłeś plik konfiguracyjny lub dodałeś nowego Wirtualnego Hosta, możesz zmusić Apache do przeładowania swojej konfiguracji bez rozłączania połączeń kogokolwiek aktualnie przeglądającego stronę:

{% image "/assets/images/blog/pl/instalacja-apache-ubuntu-debian/H9.png", "Uruchamianie sudo systemctl reload apache2 aby przeładować konfigurację Apache bez rozłączania połączeń", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo systemctl reload apache2
```

## Krok 5: Skonfiguruj Wirtualne Hosty

Jeśli planujesz hostować więcej niż jedną stronę internetową (nazwę domeny) na swoim serwerze, musisz skonfigurować **Wirtualne Hosty** (równoważne Server Blocks w Nginx).

Apache dostarcza domyślną konfigurację wirtualnego hosta od razu z pudełka (`000-default.conf`) która serwuje dokumenty z katalogu `/var/www/html`. To jest w porządku dla pojedynczej strony, ale szybko staje się koszmarem jeśli chcesz hostować wiele różnych domen.

Zamiast nadpisywać domyślną stronę, stworzymy całkowicie nową strukturę katalogów i plik konfiguracyjny dla twojej nowej domeny. Dla tego przykładu, użyjemy `your_domain.com`.

### 1. Utwórz katalog
Stwórz dedykowany folder dla twojej domeny wewnątrz `/var/www/`:

{% image "/assets/images/blog/pl/instalacja-apache-ubuntu-debian/H10.png", "Uruchamianie sudo mkdir -p aby utworzyć nowy katalog strony internetowej w /var/www na serwerze Apache Ubuntu", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo mkdir -p /var/www/your_domain.com/html
```

### 2. Dostosuj własność
Nadaj swojemu kontu użytkownika własność folderu głównego WWW abyś mógł edytować lub przesyłać pliki bez ciągłej potrzeby uprawnień root:

{% image "/assets/images/blog/pl/instalacja-apache-ubuntu-debian/H11.png", "Uruchamianie sudo chown aby przypisać własność użytkownika głównemu katalogowi WWW /var/www na Ubuntu", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo chown -R $USER:$USER /var/www/your_domain.com/html
```

### 3. Utwórz stronę testową
Umieść przykładowy plik w miejscu aby upewnić się że wszystko poprawnie się łączy.

{% image "/assets/images/blog/pl/instalacja-apache-ubuntu-debian/H12.png", "Tworzenie testowej strony index.html za pomocą edytora nano w głównym katalogu WWW Apache na Ubuntu", "(max-width: 768px) 100vw, 800px" %}

```bash
nano /var/www/your_domain.com/html/index.html
```

Dodaj ten przykładowy blok HTML:

```html
<html>
    <head>
        <title>Witaj na your_domain.com!</title>
    </head>
    <body>
        <h1>Sukces! Wirtualny host your_domain.com działa!</h1>
    </body>
</html>
```

Zapisz i wyjdź.

### 4. Utwórz plik konfiguracyjny Wirtualnego Hosta
Apache na Ubuntu izoluje konfigurację w czysto zorganizowane foldery. Wirtualne hosty powinny być zdefiniowane w `/etc/apache2/sites-available`.

{% image "/assets/images/blog/pl/instalacja-apache-ubuntu-debian/H13.png", "Tworzenie nowego pliku konfiguracyjnego wirtualnego hosta Apache za pomocą nano dla niestandardowej domeny na Ubuntu", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo nano /etc/apache2/sites-available/your_domain.com.conf
```

Wklej następującą konfigurację podstawową do pliku, która mówi Apache jakiej domeny nasłuchiwać i gdzie znaleźć pliki źródłowe:

```apache
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    ServerName your_domain.com
    ServerAlias www.your_domain.com
    DocumentRoot /var/www/your_domain.com/html
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

Zapisz i zamknij plik.

### 5. Włącz Wirtualny Host
Apache zawiera wbudowane narzędzia do łatwego zarządzania konfiguracjami. Użyjemy narzędzia `a2ensite` (Apache 2 Enable Site) aby jawnie włączyć nowego wirtualnego hosta:

{% image "/assets/images/blog/pl/instalacja-apache-ubuntu-debian/H14.png", "Uruchamianie sudo a2ensite aby włączyć konfigurację wirtualnego hosta Apache dla niestandardowej domeny", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo a2ensite your_domain.com.conf
```

Skoro już przy tym, wyłączmy domyślną stronę lądowania która przychodzi z Apache (aby żądania przeznaczone dla naszej domeny nie były myląco mapowane z powrotem do domyślnego fallback):

{% image "/assets/images/blog/pl/instalacja-apache-ubuntu-debian/H15.png", "Uruchamianie sudo a2dissite 000-default.conf aby wyłączyć domyślną stronę Apache na Ubuntu", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo a2dissite 000-default.conf
```

### 6. Przetestuj konfigurację i zrestartuj
Za każdym razem gdy edytujesz pliki konfiguracyjne Apache, najlepszą praktyką jest przetestowanie ich pod kątem błędów składni przed pełnym restartem. Zapewnia to że nie zamkniesz przypadkowo swojego serwera WWW z powodu literówki.

```bash
sudo apache2ctl configtest
```

Powinieneś zobaczyć wynik który mówi **Syntax OK**. (Może pojawić się nieszkodliwe ostrzeżenie serwera 'Could not reliably determine the server's fully qualified domain name', ale walidacja składni jest tutaj kluczowa).

Teraz przeładuj Apache aby zainicjalizować nowego wirtualnego hosta:

```bash
sudo systemctl reload apache2
```

Apache powinien teraz aktywnie serwować twoją nazwę domeny. Dopóki twoje rekordy DNS dla `your_domain.com` wskazują na adres IP twojego VPS, możesz wpisać ją w przeglądarkę aby zobaczyć swój ciąg sukcesu.

Dla niezawodnego środowiska wspierającego szybkie prototypowanie projektów z warstwami WWW Apache, sprawdź naszą wysokowydajną linię przystępnych środowisk [Budget VPS](/pl/budget-vps/) już dziś.