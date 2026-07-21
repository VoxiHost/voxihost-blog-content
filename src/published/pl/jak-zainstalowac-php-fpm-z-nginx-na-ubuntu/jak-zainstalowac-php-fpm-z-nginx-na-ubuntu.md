---
image: "/assets/images/blog/pl/jak-zainstalowac-php-fpm-z-nginx-na-ubuntu/og-image.png"
title: "Jak zainstalować PHP-FPM z Nginx na Ubuntu"
description: "Dowiedz się jak zainstalować i skonfigurować PHP-FPM z Nginx na Ubuntu. Przewodnik obejmuje instalację, konfigurację serwera oraz testy na serwerze VoxiHost."
status: published
category: Poradniki
tags:
  - php
  - php-fpm
  - nginx
  - ubuntu
  - vps
  - server
date: '2026-07-20'
locale: pl
translationKey: install-php-fpm-nginx-ubuntu
author:
  name: VoxiHost Team
  link: https://voxihost.pl/
contributors: [ "danielmarszalkowski" ]
howto:
  name: "Jak zainstalować PHP-FPM z Nginx na Ubuntu"
  steps:
    - name: "Krok 1: Instalacja Nginx i PHP-FPM"
      url: "krok-1-instalacja-nginx-i-php-fpm"
    - name: "Krok 2: Konfiguracja bloku serwera Nginx"
      url: "krok-2-konfiguracja-bloku-serwera-nginx"
    - name: "Krok 3: Utworzenie skryptu testowego PHP i weryfikacja działania"
      url: "krok-3-utworzenie-skryptu-testowego-php-i-weryfikacja-dzialania"
faq:
  - question: "Jak sprawdzić, która wersja PHP-FPM jest aktualnie uruchomiona?"
    answer: "Możesz wykonać polecenie <code>php -v</code> w terminalu, aby zobaczyć dokładną wersję PHP zainstalowaną na serwerze VoxiHost."
  - question: "Dlaczego otrzymuję błąd 502 Bad Gateway na mojej stronie?"
    answer: "Błąd 502 zazwyczaj oznacza niezgodność między ścieżką gniazda <code>fastcgi_pass</code> w Nginx a rzeczywistą ścieżką zdefiniowaną w pliku konfiguracyjnym puli PHP-FPM."
  - question: "Czy muszę restartować PHP-FPM po każdej zmianie pliku konfiguracyjnego?"
    answer: "Tak, należy wykonać <code>sudo systemctl reload php8.x-fpm</code> (zastępując <code>8.x</code> zainstalowaną wersją PHP, np. <code>8.5</code> lub <code>8.3</code>) za każdym razem, gdy modyfikujesz konfigurację puli, aby upewnić się, że zmiany zostały zastosowane."
  - question: "Jak zainstalować dodatkowe rozszerzenia PHP, takie jak php-mysql lub php-xml?"
    answer: "Użyj <code>sudo apt install php-mysql php-xml</code>, aby dodać wymagane moduły, a następnie przeładuj usługę PHP-FPM (np. <code>sudo systemctl reload php8.x-fpm</code>), aby załadować je do silnika."
  - question: "Gdzie powinienem umieścić pliki strony, aby zapewnić poprawne uprawnienia?"
    answer: "Standardową praktyką jest użycie katalogu <code>/var/www/html</code>, upewniając się, że użytkownik <code>www-data</code> posiada uprawnienia do odczytu i wykonywania plików."
---

## Wstęp

Uruchomienie wydajnego serwera WWW wymaga stosu technologicznego, który łączy szybkość z oszczędnością zasobów. Choć serwer Apache obsługuje ruch w sieci od dziesięcioleci, to połączenie Nginx z PHP-FPM stało się standardem w przypadku nowoczesnych aplikacji o dużym natężeniu ruchu. Dzięki przeniesieniu przetwarzania PHP do dedykowanego menedżera procesów FastCGI, zyskujesz znaczną poprawę w zakresie zużycia pamięci oraz obsługi żądań w porównaniu do starszych metod wykonywania skryptów.

Ten przewodnik skupia się na konfiguracji stabilnego, gotowego do pracy produkcyjnej środowiska na systemie Ubuntu 26.04 LTS. Przeprowadzimy proces instalacji Nginx oraz domyślnej wersji PHP 8.5 (PHP-FPM) dostępnej w systemie, dbając o to, aby poprawnie komunikowały się ze sobą za pośrednictwem gniazd Unix. Taka konfiguracja jest idealna dla programistów hostujących dynamiczne strony na serwerach <span class="text-white">Voxi</span><span class="text-amber-300">Host</span> [Premium VPS](/pl/premium-vps/), gdzie redukcja opóźnień ma kluczowe znaczenie.

Stawiamy na czystą i minimalistyczną instalację. Unikamy rozbudowanych pakietów na rzecz oficjalnych repozytoriów, które gwarantują, że aktualizacje bezpieczeństwa są dostępne na wyciągnięcie ręki poprzez polecenie `apt upgrade`. Pod koniec tego poradnika będziesz dysponować funkcjonalnym serwerem WWW, zdolnym do obsługi dynamicznych treści PHP, wraz z niezbędnymi poprawkami konfiguracji pozwalającymi uniknąć typowych wąskich gardeł. Niezależnie od tego, czy migrujesz istniejącą aplikację, czy uruchamiasz nowy projekt, ta konfiguracja stanowi solidny fundament dla Twojej infrastruktury.

## Wymagania wstępne

Przed rozpoczęciem instalacji należy upewnić się, że serwer spełnia podstawowe wymagania dla stabilnego środowiska produkcyjnego. Zalecamy posiadanie co najmniej 1 GB pamięci RAM oraz przynajmniej jednego dedykowanego rdzenia procesora, aby obsłużyć narzut serwera Nginx oraz procesów roboczych PHP-FPM bez spadku wydajności. Niniejszy przewodnik zakłada, że korzystasz ze świeżej instalacji systemu Ubuntu 26.04 LTS.

Konieczne jest posiadanie dostępu do konta root lub użytkownika z uprawnieniami `sudo` w celu wykonywania poleceń administracyjnych. Jeśli korzystasz z usługi <span class="text-white">Voxi</span><span class="text-amber-300">Host</span> [Budget VPS](/pl/budzetowy-vps/), upewnij się, że zapora sieciowa jest poprawnie skonfigurowana, aby zezwalać na ruch przychodzący na portach 80 oraz 443. Jeśli zapora nie została jeszcze skonfigurowana, zapoznaj się z naszym przewodnikiem dotyczącym [konfiguracji UFW](/pl/blog/konfiguracja-ufw-ubuntu-debian/), aby zapobiec nieautoryzowanemu dostępowi do portów zarządzania.

Należy zweryfikować, czy lista pakietów jest aktualna, a system w pełni zaktualizowany, aby uniknąć konfliktów zależności podczas procesu instalacji. Mimo że w kolejnych krokach zainstalujemy określone pakiety, posiadanie czystego i zaktualizowanego środowiska pozwala zapobiec typowym błędom niezgodności bibliotek. Upewnij się, że jesteś zalogowany na serwerze przez SSH oraz masz przygotowaną domenę lub adres IP serwera w celu przetestowania finalnej konfiguracji.

## Krok 1: Instalacja Nginx i PHP-FPM

Aby rozpocząć konfigurację, należy pobrać najnowsze metadane pakietów z oficjalnych repozytoriów i zainstalować serwer WWW Nginx wraz z interpreterem PHP-FPM. Zainstalujemy domyślny pakiet `php-fpm`, który automatycznie zainstaluje PHP 8.5 na systemie Ubuntu 26.04 LTS (lub inną wersję, jeśli korzystasz z innej wersji systemu Ubuntu).

Uruchom poniższe polecenia, aby zainicjować instalację:

```bash
## Aktualizacja lokalnego indeksu pakietów w celu pobrania najnowszych wersji
sudo apt update

## Instalacja Nginx oraz domyślnego menedżera procesów PHP FastCGI
sudo apt install nginx php-fpm
```

{% image "/assets/images/blog/pl/jak-zainstalowac-php-fpm-z-nginx-na-ubuntu/H1.png", "Terminal pokazujący polecenia apt update i apt install nginx php-fpm", "(max-width: 768px) 100vw, 800px" %}

Po zakończeniu instalacji warto sprawdzić, która wersja PHP została zainstalowana jako domyślna w systemie. Określa to dokładną nazwę usługi oraz ścieżkę do gniazda, których użyjesz w kolejnych krokach:

```bash
## Sprawdzenie zainstalowanej wersji PHP
php -v
```

Przykładowo, jeśli dane wyjściowe wskazują na `PHP 8.5.x`, nazwa Twojej usługi PHP-FPM to `php8.5-fpm`, a plik gniazda to `php8.5-fpm.sock` (w przypadku Ubuntu 24.04 LTS będzie to odpowiednio `PHP 8.3.x` z usługą `php8.3-fpm` i gniazdem `php8.3-fpm.sock`). Zapamiętaj tę wersję — użyjesz jej w konfiguracji Nginx w kolejnym kroku.

## Krok 2: Konfiguracja bloku serwera Nginx

Domyślnie Nginx nie jest skonfigurowany do przetwarzania plików PHP. Należy zmodyfikować domyślny blok serwera, aby przekazywał żądania dotyczące plików `.php` do gniazda (socket) PHP-FPM. Otwórz plik konfiguracyjny za pomocą edytora tekstu:

```bash
## Otwórz domyślny plik konfiguracyjny witryny do edycji
sudo nano /etc/nginx/sites-available/default
```

Znajdź blok `location /` i upewnij się, że dyrektywa `index` zawiera `index.php`. Następnie odkomentuj lub dodaj blok `location ~ \.php$ { ... }`, aby obsłużyć żądania PHP. Twoja konfiguracja powinna przypominać poniższą strukturę:

```nginx
server {
    listen 80 default_server;
    listen [::]:80 default_server;

    root /var/www/html;
    index index.php index.html index.htm index.nginx-debian.html;

    server_name _;

    location / {
        try_files $uri $uri/ =404;
    }

    # Przekaż skrypty PHP do gniazda PHP-FPM
    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.x-fpm.sock;
    }
}
```

Sprawdź dokładnie dyrektywę `fastcgi_pass`. Ścieżka `/var/run/php/php8.x-fpm.sock` (gdzie `8.x` to Twoja wersja PHP, np. `8.5` na Ubuntu 26.04 LTS lub `8.3` na Ubuntu 24.04 LTS) musi być zgodna z rzeczywistym plikiem gniazda w Twoim systemie. W przypadku późniejszych błędów upewnij się, że numer wersji w ścieżce zgadza się z usługą `php8.x-fpm`, którą zweryfikowano w poprzednim kroku. Zapisz zmiany, naciskając `Ctrl + O`, a następnie `Enter`, i wyjdź za pomocą `Ctrl + X`.

{% image "/assets/images/blog/pl/jak-zainstalowac-php-fpm-z-nginx-na-ubuntu/H3.png", "Widok edytora tekstu z domyślną konfiguracją Nginx pokazujący poprawnie skonfigurowany blok lokalizacji PHP", "(max-width: 768px) 100vw, 800px" %}

## Krok 3: Utworzenie skryptu testowego PHP i weryfikacja działania

Aby zweryfikować, czy Nginx poprawnie przekazuje żądania do procesora PHP-FPM, należy utworzyć prostą stronę informacyjną. Skrypt ten wykorzystuje funkcję `phpinfo()`, która wyświetla bieżącą konfigurację PHP, załadowane moduły oraz zmienne środowiskowe.

Przejdź do domyślnego katalogu głównego serwera WWW i utwórz nowy plik o nazwie `info.php`:

```bash
## Utwórz plik testowy w domyślnym katalogu głównym serwera WWW
sudo nano /var/www/html/info.php
```

Wklej do pliku następujący kod PHP:

```php
<?php
phpinfo();
?>
```

Zapisz plik, naciskając `Ctrl + O`, a następnie `Enter`, i wyjdź za pomocą `Ctrl + X`.

Przed przetestowaniem pliku w przeglądarce należy sprawdzić konfigurację Nginx, aby upewnić się, że w poprzednim kroku nie pojawiły się błędy składniowe. Następnie zastosuj zmiany, przeładowując usługę:

```bash
## Sprawdź składnię konfiguracji
sudo nginx -t

## Zastosuj zmiany w usłudze Nginx
sudo systemctl reload nginx
```

Jeśli test składni zakończy się pomyślnie, otwórz przeglądarkę internetową i przejdź pod adres `http://twoj_adres_ip_serwera/info.php`. Powinna wyświetlić się strona ze szczegółowymi informacjami o instalacji PHP (w przypadku nowej instalacji na Ubuntu 26.04 LTS będzie to PHP 8.5). Jeśli zamiast wyrenderowanej strony widzisz surowy kod, sprawdź dokładnie, czy blok `location ~ \.php$` w konfiguracji Nginx zgadza się ze ścieżką do gniazda (socket). Po potwierdzeniu, że wszystko działa poprawnie, usuń plik testowy, aby uniemożliwić nieautoryzowanym użytkownikom wgląd w wrażliwe informacje o serwerze:

```bash
## Usuń skrypt testowy w celu zachowania bezpieczeństwa
sudo rm /var/www/html/info.php
```

{% image "/assets/images/blog/pl/jak-zainstalowac-php-fpm-z-nginx-na-ubuntu/H4.png", "Strona PHP info wyświetlona w przeglądarce, potwierdzająca poprawną integrację PHP-FPM z serwerem Nginx", "(max-width: 768px) 100vw, 800px" %}

Po potwierdzeniu integracji PHP za pomocą skryptu testowego należy przeprowadzić ostateczną kontrolę usług systemowych, aby upewnić się, że zarówno Nginx, jak i PHP-FPM działają bez błędów. Uruchom poniższe polecenia, aby zweryfikować status obu demonów:

```bash
## Sprawdź status serwera WWW Nginx
sudo systemctl status nginx

## Sprawdź status usługi PHP-FPM (zastąp 8.x swoją wersją PHP)
sudo systemctl status php8.x-fpm
```

Upewnij się, że obie usługi wyświetlają status "active (running)". Jeśli usługa wyświetla status "failed" lub "inactive", sprawdź logi systemowe za pomocą polecenia `journalctl -u nginx` lub `journalctl -u php8.x-fpm` (zastępując `8.x` swoją wersją PHP). Ponadto zaleca się okresowe sprawdzanie logów błędów Nginx w `/var/log/nginx/error.log` w celu szybkiego wykrywania problemów z uprawnieniami lub brakującymi modułami PHP.

{% image "/assets/images/blog/pl/jak-zainstalowac-php-fpm-z-nginx-na-ubuntu/H2.png", "Dane wyjściowe terminala pokazujące wynik php -v oraz aktywny status usług Nginx i PHP-FPM", "(max-width: 768px) 100vw, 800px" %}

## Podsumowanie

Dysponujesz teraz w pełni funkcjonalnym stosem LEMP działającym na serwerze Ubuntu. Dzięki połączeniu Nginx z PHP-FPM 8.5 stworzyłeś wysokowydajną podstawę, zdolną do obsługi dynamicznego ruchu internetowego przy minimalnym obciążeniu zasobów. Niezależnie od tego, czy wdrażasz własną aplikację, czy system zarządzania treścią, ta konfiguracja zapewnia niezbędną kontrolę nad środowiskiem serwerowym.

Utrzymanie tego stosu to coś więcej niż tylko początkowa konfiguracja. W miarę rozwoju aplikacji może zajść potrzeba dostosowania procesów roboczych w konfiguracji Nginx lub dostrojenia ustawień puli PHP-FPM, aby lepiej odpowiadały przydzielonym zasobom. Jeśli kiedykolwiek zajdzie potrzeba przeprowadzenia aktualizacji systemu, pamiętaj, aby najpierw zatrzymać usługi w celu zapewnienia integralności danych:

```bash
## Bezpieczne zatrzymanie usług przed aktualizacją systemu (zastąp 8.x swoją wersją PHP)
sudo systemctl stop nginx
sudo systemctl stop php8.x-fpm

## Aktualizacja pakietów systemowych
sudo apt update && sudo apt upgrade -y

## Uruchomienie usług po aktualizacji (zastąp 8.x swoją wersją PHP)
sudo systemctl start nginx
sudo systemctl start php8.x-fpm
```

Dla osób zarządzających krytycznymi obciążeniami, kolejnym logicznym krokiem jest monitorowanie. Obserwuj wykorzystanie zasobów systemowych za pośrednictwem panelu <span class="text-white">Voxi</span><span class="text-amber-300">Host</span>, szczególnie jeśli skalujesz swoje środowiska [Premium VPS](/pl/premium-vps/) lub [Budget VPS](/pl/budzetowy-vps/). Regularne sprawdzanie logów oraz aktualizowanie pakietów zapewni, że Twój serwer WWW pozostanie szybki i bezpieczny przed nowymi zagrożeniami.

Jeśli zdecydujesz się na dalszą rozbudowę infrastruktury, rozważ zapoznanie się z przewodnikiem [Jak skonfigurować certyfikat SSL z Let's Encrypt i Certbot na Ubuntu i Debianie: Kompletny poradnik](/pl/blog/certyfikat-ssl-letsencrypt-ubuntu/), aby szyfrować ruch i poprawić pozycjonowanie w wyszukiwarkach.
