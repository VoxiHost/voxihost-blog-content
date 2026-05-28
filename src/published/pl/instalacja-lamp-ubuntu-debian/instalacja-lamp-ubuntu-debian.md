---
image: /assets/images/blog/instalacja-lamp-ubuntu-debian/og-image.png
title: Jak skonfigurować stos LAMP (Linux, Apache, MySQL, PHP) na Ubuntu i Debian
description: Kompleksowy przewodnik dla początkujących do instalacji sprawdzonego stosu LAMP (Linux, Apache2, MySQL, PHP) na Ubuntu i Debian.
date: '2026-03-25'
translationKey: setup-lamp-stack-ubuntu-debian
category: Poradniki
tags:
  - lamp
  - apache
  - mysql
  - php
  - ubuntu
  - debian
  - linux
  - vps
  - web server
howto:
  name: Jak zainstalować stos LAMP na Ubuntu lub Debian
  totalTime: PT15M
  yield: W pełni funkcjonujący serwer WWW zdolny do renderowania dynamicznych aplikacji PHP używając Apache i MySQL
  tool:
    - VPS lub dedykowany serwer z Ubuntu lub Debian
    - Klient SSH (np. terminal, PuTTY)
    - Konto użytkownika z uprawnieniami sudo
  steps:
    - name: Zainstaluj Apache (serwer WWW)
      text: Uruchom sudo apt install apache2 i skonfiguruj UFW aby zezwolić na ruch Apache Full.
      url: step-1-install-apache-the-web-server
    - name: Zainstaluj MySQL (baza danych)
      text: Uruchom sudo apt install mysql-server i zabezpiecz go używając wbudowanego skryptu bezpieczeństwa MySQL.
      url: step-2-install-mysql-the-database
    - name: Zainstaluj PHP
      text: Uruchom sudo apt install php libapache2-mod-php php-mysql aby zainstalować PHP i powiązać go bezpośrednio z Apache.
      url: step-3-install-php
    - name: Skonfiguruj indeks katalogów Apache
      text: Edytuj dir.conf aby priorytetyzować index.php nad standardowymi plikami index.html.
      url: step-4-configure-apache-index-priorities
    - name: Przetestuj przetwarzanie PHP
      text: Utwórz plik info.php w /var/www/html aby zweryfikować swoją konfigurację.
      url: step-5-test-the-lamp-stack
status: published
locale: pl
author:
  name: VoxiHost Team
  link: https://voxihost.pl/
contributors:
  - danielmarszalkowski
---

**Stos LAMP** jest niekwestionowanym pra-dziadem open-source hostingu stron internetowych. Przez dekady był najbardziej niezawodny, dokładnie udokumentowany i szeroko rozpowszechniony fundament dla hostowania dynamicznych aplikacji jak WordPress, Drupal i Joomla.

LAMP oznacza cztery komponenty:
- **L**inux: System operacyjny (Ubuntu lub Debian).
- **A**pache: Niesamowicie wytrzymały, wysoce konfigurowalny serwer WWW.
- **M**ySQL: Najpopularniejszy system zarządzania relacyjnymi bazami danych.
- **P**HP: Język skryptowy po stronie serwera obsługujący logikę backendową.

W porównaniu do nowszych stosów LEMP (Nginx), LAMP pozostaje uniwersalnie uwielbiany ponieważ Apache przetwarza PHP dynamicznie natywnie (nie potrzebu konfigurowania zewnętrznych gniazd FPM) i opiera się na wysoko elastycznych plikach `.htaccess` dla łatwej, katalogowej konfiguracji zastępczej.

## Krok 1: Zainstaluj Apache (Serwer WWW)

Przed instalacją jakiegokolwiek oprogramowania, zawsze aktualizuj swoje lokalne indeksy pakietów aby pobierać najnowsze poprawki bezpieczeństwa.

```bash
sudo apt update
sudo apt upgrade -y
```

Teraz zainstaluj serwer WWW Apache (pakiet nosi się `apache2` na systemach Debian/Ubuntu):

{% image "/assets/images/blog/instalacja-lamp-ubuntu-debian/H1.png", "Uruchamianie sudo apt install apache2 -y aby rozpocząć instalację serwera WWW Apache na Ubuntu", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo apt install apache2 -y
```

Jeśli używasz zapory `ufw` (co powinieneś robić zgodnie z naszym [Przewodnikiem Konfiguracji Zapory UFW](/pl/blog/konfiguracja-ufw-ubuntu-debian/)), musisz zezwolić na ruch Apache aby mógł przechodzić. Chcesz otworzyć zarówno HTTP (Port 80) jak i HTTPS (Port 443).

```bash
sudo ufw allow 'Apache Full'
```

Aby zweryfikować że twój serwer WWW żyje, wpisz adres IP swojego serwera do ulubionej przeglądarki (`http://your_server_ip`). Powinieneś zobaczyć domyślną stronę *"Apache2 Ubuntu Default Page"*.

## Krok 2: Zainstaluj MySQL (Baza danych)

Twój serwer może teraz serwować statyczne HTML, ale aby przechowywać dane aplikacji (jak konta użytkowników, posty na blogu i ustawienia), potrzebujesz bazy danych.

Zainstaluj oficjalny serwer MySQL:

{% image "/assets/images/blog/instalacja-lamp-ubuntu-debian/H2.png", "Uruchamianie sudo apt install mysql-server -y aby rozpocząć instalację serwera bazy danych MySQL na Ubuntu", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo apt install mysql-server -y
```

Gdy instalacja się zakończy, baza danych działa ale jej domyślna konfiguracja jest niebezpiecznie otwarta. Musisz ją zablokować używając interaktywnego skryptu bezpieczeństwa:

{% image "/assets/images/blog/instalacja-lamp-ubuntu-debian/H3.png", "Uruchamianie sudo mysql_secure_installation aby zabezpieczyć instalację MySQL", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo mysql_secure_installation
```

Zostaniesz poproszony o kilka pytań aby skonfigurować profil bezpieczeństwa:
1. **Wtyczka Validate Password**: Wpisz `y` jeśli chcesz aby MySQL aktywnie blokował słabe hasła, lub `n` aby pominąć.
2. **Usuń użytkowników anonimowych**: Wpisz `y`.
3. **Zabroń logowania root zdalnie**: Wpisz `y` (root powinien mieć dostęp do bazy danych tylko z *wewnątrz* serwera).
4. **Usuń bazę danych testową**: Wpisz `y`.
5. **Przeładuj tabele uprawnień**: Wpisz `y`.

MySQL jest teraz bezpieczny.

## Krok 3: Zainstaluj PHP

Masz serwer WWW i bazę danych, ale nie mogą się jeszcze ze sobą komunikować ani przetwarzać kod dynamiczny. Potrzebujesz PHP.

Dla Apache, instalacja PHP wymaga trzech głównych pakietów: podstawowy pakiet `php`, rozszerzenie `php-mysql` pozwalające skryptom PHP rozmawiać z twoją bazą danych, i kluczowy pakiet `libapache2-mod-php` który magicznie wiąże przetwarzanie PHP bezpośrednio w runtime Apache.

{% image "/assets/images/blog/instalacja-lamp-ubuntu-debian/H4.png", "Uruchamianie sudo apt install php libapache2-mod-php php-mysql -y aby zainstalować PHP i powiązać je z Apache i MySQL na Ubuntu", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo apt install php libapache2-mod-php php-mysql -y
```

## Krok 4: Skonfiguruj priorytetów indeksu Apache

Gdy użytkownik odwiedza katalog na twojej stronie (jak `twojstrona.com/blog/`), Apache automatycznie szuka domyślnego pliku "index" do serwowania. Domyślnie szuka `index.html` jako pierwszy, a jeśli go nie znajdzie, ostatecznie szuka `index.php`.

Dla aplikacji dynamicznych, chcemy aby Apache priorytetyzowało `index.php`.

Otwórz plik konfiguracyjny modułu `dir`:

{% image "/assets/images/blog/instalacja-lamp-ubuntu-debian/H5.png", "Uruchamianie sudo nano /etc/apache2/mods-enabled/dir.conf aby otworzyć plik konfiguracyjny priorytetów indeksu Apache", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo nano /etc/apache2/mods-enabled/dir.conf
```

Otworzy się on wygląda tak:

```apache
<IfModule mod_dir.c>
    DirectoryIndex index.html index.cgi index.pl index.xhtml index.htm
</IfModule>
```

Przesuń ciąg `index.php` na początek listy bezpośrednio po `DirectoryIndex`, aby wyglądało tak:

```apache
<IfModule mod_dir.c>
    DirectoryIndex index.php index.html index.cgi index.pl index.xhtml index.htm
</IfModule>
```

Zapisz i wyjdź.

Za każdym razem gdy modyfikujesz moduły Apache, musisz zrestartować serwer WWW aby zmiany weszły w życie:

```bash
sudo systemctl restart apache2
```

## Krok 5: Przetestuj stos LAMP

Twoje środowisko jest kompletne! Jednakże złotą zasadą administracji systemowej jest weryfikowanie swojej pracy. Napiszemy mały skrypt PHP aby udowodnić że Apache może przetwarzać kod dynamiczny.

Utwórz nowy plik w domyślnym katalogu głównym WWW Apache:

```bash
sudo nano /var/www/html/info.php
```

Wklej standardową funkcję inicjalizacyjną:

```php
<?php
phpinfo();
?>
```

Zapisz plik. Otwórz przeglądarkę i nawiguj do `http://your_server_ip/info.php`.

{% image "/assets/images/blog/instalacja-lamp-ubuntu-debian/H6.png", "Uruchamianie sudo systemctl status apache2 na Ubuntu aby zweryfikować że Apache jest aktywne po zakończeniu pełnej instalacji stosu LAMP", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo systemctl status apache2
```

Jeśli instalacja się powiodła, zostaniesz powitany ogromną, szczegółową tabelę opisującą twoją wersję PHP, zainstalowane moduły, limity pamięci i ustawienia integracji Apache.

> **Krytyczne ostrzeżenie bezpieczeństwa:** Strona `info.php` zawiera obszerną mapę twojej wewnętrznej architektury serwera. Pozostawienie tego pliku publicznego jest masowym ryzykiem bezpieczeństwa. Gdy potwierdzisz że stos działa, **usuń plik natychmiast**:
```bash
sudo rm /var/www/html/info.php
```

Gratulacje! Pomyślnie zbudowałeś sprawdzony, przetestowany i solidny fundament do hostowania dynamicznych aplikacji na Linuksie. Czy masz ciężki sklep e-commerce, forum czy blog o dużym ruchu gotowy do uruchomienia? Połącz swój nowy stos LAMP z jednym z naszych środowisk [Premium VPS](/pl/premium-vps/) lub wysoko efektywnych kosztowo [Budget VPS](/pl/budget-vps/), zainstaluj [bezpłatny SSL przez Certbot](/pl/blog/certyfikat-ssl-letsencrypt-ubuntu/) i zbuduj ostateczne doświadczenie WWW.