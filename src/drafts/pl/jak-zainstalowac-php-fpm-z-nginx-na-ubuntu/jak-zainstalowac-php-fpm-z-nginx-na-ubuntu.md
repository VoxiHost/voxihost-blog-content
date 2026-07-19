---
image: "/assets/images/blog/pl/jak-zainstalowac-php-fpm-z-nginx-na-ubuntu/og-image.png"
title: "Jak zainstalować PHP-FPM z Nginx na Ubuntu"
description: "Dowiedz się jak zainstalować i skonfigurować PHP-FPM z Nginx na Ubuntu. Przewodnik obejmuje instalację, konfigurację serwera oraz testy na serwerze VoxiHost."
status: draft
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
contributors: []
howto:
  name: "Jak zainstalować PHP-FPM z Nginx na Ubuntu"
  steps:
    - name: "Krok 1: Instalacja Nginx i PHP-FPM"
      url: "krok-1-instalacja-nginx-i-php-fpm"
    - name: "Krok 2: Konfiguracja bloku serwera Nginx"
      url: "krok-2-konfiguracja-bloku-serwera-nginx"
    - name: "Krok 3: Utworzenie skryptu testowego PHP"
      url: "krok-3-utworzenie-skryptu-testowego-php"
    - name: "Krok 4: Weryfikacja konfiguracji i statusu usługi"
      url: "krok-4-weryfikacja-konfiguracji-i-statusu-usługi"
faq:
  - question: "Jak sprawdzić, która wersja PHP-FPM jest aktualnie uruchomiona?"
    answer: "Możesz wykonać polecenie <code>php-fpm8.3 -v</code> w terminalu, aby zobaczyć dokładną wersję zainstalowaną na serwerze VoxiHost."
  - question: "Dlaczego otrzymuję błąd 502 Bad Gateway na mojej stronie?"
    answer: "Błąd 502 zazwyczaj oznacza niezgodność między ścieżką gniazda <code>fastcgi_pass</code> w Nginx a rzeczywistą ścieżką zdefiniowaną w pliku konfiguracyjnym puli PHP-FPM."
  - question: "Czy muszę restartować PHP-FPM po każdej zmianie pliku konfiguracyjnego?"
    answer: "Tak, należy wykonać <code>sudo systemctl reload php8.3-fpm</code> za każdym razem, gdy modyfikujesz konfigurację puli, aby upewnić się, że zmiany zostały zastosowane."
  - question: "Jak zainstalować dodatkowe rozszerzenia PHP, takie jak php-mysql lub php-xml?"
    answer: "Użyj <code>sudo apt install php8.3-mysql php8.3-xml</code>, aby dodać wymagane moduły, a następnie przeładuj usługę PHP-FPM, aby załadować je do silnika."
  - question: "Gdzie powinienem umieścić pliki strony, aby zapewnić poprawne uprawnienia?"
    answer: "Standardową praktyką jest użycie katalogu <code>/var/www/html</code>, upewniając się, że użytkownik <code>www-data</code> posiada uprawnienia do odczytu i wykonywania plików."
---

## Wstęp

Uruchomienie wydajnego serwera WWW wymaga stosu technologicznego, który łączy szybkość z oszczędnością zasobów. Choć serwer Apache obsługuje ruch w sieci od dziesięcioleci, to połączenie Nginx z PHP-FPM stało się standardem w przypadku nowoczesnych aplikacji o dużym natężeniu ruchu. Dzięki przeniesieniu przetwarzania PHP do dedykowanego menedżera procesów FastCGI, zyskujesz znaczną poprawę w zakresie zużycia pamięci oraz obsługi żądań w porównaniu do starszych metod wykonywania skryptów.

Ten przewodnik skupia się na konfiguracji stabilnego, gotowego do pracy produkcyjnej środowiska na systemie Ubuntu. Przeprowadzimy proces instalacji Nginx oraz najnowszej stabilnej wersji PHP 8.3, dbając o to, aby poprawnie komunikowały się ze sobą za pośrednictwem gniazd Unix. Taka konfiguracja jest idealna dla programistów hostujących dynamiczne strony na serwerach <span class="text-white">Voxi</span><span class="text-amber-300">Host</span> [Premium VPS](/pl/premium-vps/), gdzie redukcja opóźnień ma kluczowe znaczenie.

Stawiamy na czystą i minimalistyczną instalację. Unikamy rozbudowanych pakietów na rzecz oficjalnych repozytoriów, które gwarantują, że aktualizacje bezpieczeństwa są dostępne na wyciągnięcie ręki poprzez polecenie `apt upgrade`. Pod koniec tego poradnika będziesz dysponować funkcjonalnym serwerem WWW, zdolnym do obsługi dynamicznych treści PHP, wraz z niezbędnymi poprawkami konfiguracji pozwalającymi uniknąć typowych wąskich gardeł. Niezależnie od tego, czy migrujesz istniejącą aplikację, czy uruchamiasz nowy projekt, ta konfiguracja stanowi solidny fundament dla Twojej infrastruktury.

{% image "/assets/images/blog/pl/jak-zainstalowac-php-fpm-z-nginx-na-ubuntu/H1.png", "Terminal pokazujący weryfikację statusu usług Nginx oraz PHP-FPM", "(max-width: 768px) 100vw, 800px" %}

## Wymagania wstępne

Przed rozpoczęciem instalacji należy upewnić się, że serwer spełnia podstawowe wymagania dla stabilnego środowiska produkcyjnego. Zalecamy posiadanie co najmniej 1 GB pamięci RAM oraz przynajmniej jednego dedykowanego rdzenia procesora, aby obsłużyć narzut serwera Nginx oraz procesów roboczych PHP-FPM bez spadku wydajności. Niniejszy przewodnik zakłada, że korzystasz ze świeżej instalacji systemu Ubuntu 24.04 LTS.

Konieczne jest posiadanie dostępu do konta root lub użytkownika z uprawnieniami `sudo` w celu wykonywania poleceń administracyjnych. Jeśli korzystasz z usługi <span class="text-white">Voxi</span><span class="text-amber-300">Host</span> [Budget VPS](/pl/budzetowy-vps/), upewnij się, że zapora sieciowa jest poprawnie skonfigurowana, aby zezwalać na ruch przychodzący na portach 80 oraz 443. Jeśli zapora nie została jeszcze skonfigurowana, zapoznaj się z naszym przewodnikiem dotyczącym [konfiguracji UFW](/pl/konfiguracja-ufw-ubuntu-debian/), aby zapobiec nieautoryzowanemu dostępowi do portów zarządzania.

Należy zweryfikować, czy lista pakietów jest aktualna, a system w pełni zaktualizowany, aby uniknąć konfliktów zależności podczas procesu instalacji. Mimo że w kolejnych krokach zainstalujemy określone pakiety, posiadanie czystego i zaktualizowanego środowiska pozwala zapobiec typowym błędom niezgodności bibliotek. Upewnij się, że jesteś zalogowany na serwerze przez SSH oraz masz przygotowaną domenę lub adres IP serwera w celu przetestowania finalnej konfiguracji.

{% image "/assets/images/blog/pl/jak-zainstalowac-php-fpm-z-nginx-na-ubuntu/H2.png", "Sesja terminala pokazująca aktualizację systemu i podstawowe sprawdzenie środowiska", "(max-width: 768px) 100vw, 800px" %}

## Krok 1: Instalacja Nginx i PHP-FPM

Aby rozpocząć konfigurację, należy pobrać najnowsze metadane pakietów z oficjalnych repozytoriów i zainstalować serwer WWW Nginx wraz z interpreterem PHP-FPM. Docelowo wybieramy PHP 8.3, który jest obecnie stabilnym wydaniem dla systemu Ubuntu 24.04 LTS.

Uruchom poniższe polecenia, aby zainicjować instalację:

```bash
## Aktualizacja lokalnego indeksu pakietów w celu pobrania najnowszych wersji
sudo apt update

## Instalacja Nginx oraz menedżera procesów PHP 8.3 FastCGI
sudo apt install nginx php8.3-fpm
```

Pakiet `php8.3-fpm` zawiera niezbędne pliki usługi systemd, które pozwalają zarządzać procesami roboczymi PHP niezależnie od serwera WWW. Po zakończeniu instalacji system automatycznie uruchamia usługę Nginx. Możesz zweryfikować, czy obie usługi są aktywne i działają poprawnie, sprawdzając ich status:

```bash
## Weryfikacja działania usługi Nginx
sudo systemctl status nginx

## Weryfikacja działania usługi PHP-FPM
sudo systemctl status php8.3-fpm
```

Jeśli w danych wyjściowych statusu widzisz "active (running)" w zielonym kolorze, oznacza to, że podstawowe usługi zostały skonfigurowane poprawnie. Nginx nasłuchuje teraz na domyślnym porcie HTTP, a gniazdo PHP-FPM jest gotowe do obsługi żądań. Możesz przejść do połączenia tych dwóch komponentów, aby umożliwić przetwarzanie dynamicznych plików PHP.

{% image "/assets/images/blog/pl/jak-zainstalowac-php-fpm-z-nginx-na-ubuntu/H3.png", "Dane wyjściowe terminala pokazujące pomyślną instalację oraz aktywny status usług Nginx i PHP-FPM", "(max-width: 768px) 100vw, 800px" %}

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
        fastcgi_pass unix:/var/run/php/php8.3-fpm.sock;
    }
}
```

Sprawdź dokładnie dyrektywę `fastcgi_pass`. Ścieżka `/var/run/php/php8.3-fpm.sock` musi być zgodna z rzeczywistym plikiem gniazda w Twoim systemie. W przypadku późniejszych błędów upewnij się, że numer wersji w ścieżce zgadza się z usługą `php8.3-fpm`, którą zainstalowano w poprzednim kroku. Zapisz zmiany, naciskając `Ctrl + O`, a następnie `Enter`, i wyjdź za pomocą `Ctrl + X`.

{% image "/assets/images/blog/pl/jak-zainstalowac-php-fpm-z-nginx-na-ubuntu/H4.png", "Widok edytora tekstu z domyślną konfiguracją Nginx pokazujący poprawnie skonfigurowany blok lokalizacji PHP", "(max-width: 768px) 100vw, 800px" %}

## Krok 3: Utworzenie skryptu testowego PHP

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

Jeśli test składni zakończy się pomyślnie, otwórz przeglądarkę internetową i przejdź pod adres `http://twoj_adres_ip_serwera/info.php`. Powinna wyświetlić się strona ze szczegółowymi informacjami o instalacji PHP 8.3. Jeśli zamiast wyrenderowanej strony widzisz surowy kod, sprawdź dokładnie, czy blok `location ~ \.php$` w konfiguracji Nginx zgadza się ze ścieżką do gniazda (socket). Po potwierdzeniu, że wszystko działa poprawnie, usuń plik testowy, aby uniemożliwić nieautoryzowanym użytkownikom wgląd w wrażliwe informacje o serwerze:

```bash
## Usuń skrypt testowy w celu zachowania bezpieczeństwa
sudo rm /var/www/html/info.php
```

{% image "/assets/images/blog/pl/jak-zainstalowac-php-fpm-z-nginx-na-ubuntu/H5.png", "Strona PHP info wyświetlona w przeglądarce, potwierdzająca poprawną integrację PHP-FPM z serwerem Nginx", "(max-width: 768px) 100vw, 800px" %}

## Krok 4: Weryfikacja konfiguracji i statusu usługi

Po potwierdzeniu integracji PHP za pomocą skryptu testowego należy przeprowadzić ostateczną kontrolę usług systemowych, aby upewnić się, że zarówno Nginx, jak i PHP-FPM działają bez błędów. Monitorowanie tych usług jest kluczowe, zwłaszcza w przypadku środowisk produkcyjnych uruchomionych na serwerze <span class="text-white">Voxi</span><span class="text-amber-300">Host</span> [Premium VPS](/pl/premium-vps/).

Uruchom poniższe polecenia, aby zweryfikować aktywny status obu demonów:

```bash
## Sprawdź status serwera WWW Nginx
sudo systemctl status nginx

## Sprawdź status usługi PHP 8.3 FPM
sudo systemctl status php8.3-fpm
```

> **Uwaga:** W danych wyjściowych tych poleceń należy szukać wskaźnika "active (running)". Jeśli usługa wyświetla status "failed" lub "inactive", sprawdź logi za pomocą polecenia `journalctl -u nginx` lub `journalctl -u php8.3-fpm`, aby zidentyfikować ewentualne niezgodności w konfiguracji.

Jeśli wprowadzono jakiekolwiek ostateczne poprawki w plikach konfiguracyjnych, zawsze wykonaj sprawdzenie składni przed przeładowaniem usługi, aby uniknąć przestojów:

```bash
## Sprawdź poprawność składni konfiguracji Nginx
sudo nginx -t

## Zastosuj wszelkie końcowe zmiany w konfiguracji
sudo systemctl reload nginx
```

Te kroki zapewniają, że stos technologiczny jest nie tylko funkcjonalny, ale również odporny na drobne błędy konfiguracyjne. Jeśli obsługujesz wiele witryn, rozważ okresowe sprawdzanie logów błędów znajdujących się w `/var/log/nginx/error.log`. Pomaga to wyłapać problemy z uprawnieniami lub brakującymi rozszerzeniami PHP, zanim wpłyną one na użytkowników końcowych.

{% image "/assets/images/blog/pl/jak-zainstalowac-php-fpm-z-nginx-na-ubuntu/H6.png", "Dane wyjściowe terminala pokazujące aktywny status usług Nginx oraz PHP-FPM", "(max-width: 768px) 100vw, 800px" %}

## Podsumowanie

Dysponujesz teraz w pełni funkcjonalnym stosem LEMP działającym na serwerze Ubuntu. Dzięki połączeniu Nginx z PHP-FPM 8.3 stworzyłeś wysokowydajną podstawę, zdolną do obsługi dynamicznego ruchu internetowego przy minimalnym obciążeniu zasobów. Niezależnie od tego, czy wdrażasz własną aplikację, czy system zarządzania treścią, ta konfiguracja zapewnia niezbędną kontrolę nad środowiskiem serwerowym.

Utrzymanie tego stosu to coś więcej niż tylko początkowa konfiguracja. W miarę rozwoju aplikacji może zajść potrzeba dostosowania procesów roboczych w konfiguracji Nginx lub dostrojenia ustawień puli PHP-FPM, aby lepiej odpowiadały przydzielonym zasobom. Jeśli kiedykolwiek zajdzie potrzeba przeprowadzenia aktualizacji systemu, pamiętaj, aby najpierw zatrzymać usługi w celu zapewnienia integralności danych:

```bash
## Bezpieczne zatrzymanie usług przed aktualizacją systemu
sudo systemctl stop nginx
sudo systemctl stop php8.3-fpm

## Aktualizacja pakietów systemowych
sudo apt update && sudo apt upgrade -y

## Uruchomienie usług po aktualizacji
sudo systemctl start nginx
sudo systemctl start php8.3-fpm
```

Dla osób zarządzających krytycznymi obciążeniami, kolejnym logicznym krokiem jest monitorowanie. Obserwuj wykorzystanie zasobów systemowych za pośrednictwem panelu <span class="text-white">Voxi</span><span class="text-amber-300">Host</span>, szczególnie jeśli skalujesz swoje środowiska [Premium VPS](/pl/premium-vps/) lub [Budget VPS](/pl/budget-vps/). Regularne sprawdzanie logów oraz aktualizowanie pakietów zapewni, że Twój serwer WWW pozostanie szybki i bezpieczny przed nowymi zagrożeniami.

Jeśli zdecydujesz się na dalszą rozbudowę infrastruktury, rozważ zapoznanie się z przewodnikiem [Jak skonfigurować certyfikat SSL z Let's Encrypt i Certbot na Ubuntu i Debianie: Kompletny poradnik](/pl/certyfikat-ssl-letsencrypt-ubuntu/), aby szyfrować ruch i poprawić pozycjonowanie w wyszukiwarkach.

{% image "/assets/images/blog/pl/jak-zainstalowac-php-fpm-z-nginx-na-ubuntu/H7.png", "Ukończony stos serwera WWW prezentujący poprawnie działające usługi Nginx i PHP-FPM", "(max-width: 768px) 100vw, 800px" %}
