---
image: /assets/images/blog/pl/instalacja-nginx-ubuntu-debian/og-image.png
title: 'Jak zainstalować Nginx na Ubuntu i Debian: Kompletny przewodnik serwera'
description: Kompletny przewodnik krok po kroku do instalacji Nginx na serwerach Ubuntu i Debian. Naucz się instalować serwer WWW, konfigurować UFW, zarządzać procesem Nginx i konfigurować swój pierwszy blok serwera.
date: '2026-03-25'
translationKey: install-nginx-ubuntu-debian
category: Poradniki
tags:
  - nginx
  - ubuntu
  - debian
  - web server
  - linux
  - vps
  - server administration
  - server block
  - ufw
howto:
  name: Jak zainstalować Nginx na Ubuntu i Debian
  totalTime: PT15M
  yield: W pełni funkcjonalny serwer WWW Nginx działający na Ubuntu lub Debian, gotowy do hostowania stron internetowych
  tool:
    - VPS lub dedykowany serwer z Ubuntu lub Debian
    - Klient SSH (np. terminal, PuTTY)
    - Konto użytkownika z uprawnieniami sudo
  steps:
    - name: Zaktualizuj indeks pakietów i zainstaluj Nginx
      text: Uruchom sudo apt update a następnie sudo apt install nginx aby zainstalować serwer WWW.
      url: krok-1-zainstaluj-nginx
    - name: Skonfiguruj zaporę
      text: Uruchom sudo ufw allow 'Nginx Full' aby otworzyć ruch HTTP (80) i HTTPS (443).
      url: krok-2-dostosuj-zapore
    - name: Zweryfikuj że Nginx działa
      text: Sprawdź adres IP swojego serwera w przeglądarce aby zobaczyć domyślną stronę powitalną Nginx.
      url: krok-3-sprawdz-swoj-serwer-www
    - name: Zarządzaj procesem Nginx
      text: Naucz się używać systemctl aby uruchamiać, zatrzymywać, restartować i przeładowywać Nginx.
      url: krok-4-zarzadzaj-procesem-nginx
    - name: Skonfiguruj Blok Serwera (Wirtualny Host)
      text: Utwórz niestandardowy plik konfiguracyjny w /etc/nginx/sites-available/ aby hostować własną domenę.
      url: krok-5-skonfiguruj-blok-serwera-wirtualny-host
faq:
  - question: "Co to jest Nginx i dlaczego jest często wybierany zamiast Apache?"
    answer: "Nginx korzysta z asynchronicznej, sterowanej zdarzeniami architektury, która znacznie lepiej radzi sobie z obsługą wielu jednoczesnych połączeń w porównaniu do procesowego modelu Apache. Jest idealny do serwowania statycznych treści oraz jako reverse proxy."
  - question: "Jaka jest różnica między katalogami sites-available a sites-enabled?"
    answer: "Katalog <code>sites-available</code> zawiera pliki konfiguracyjne wszystkich stworzonych bloków serwera (stron), natomiast <code>sites-enabled</code> zawiera dowiązania symboliczne (symlinki) do tych konfiguracji, które Nginx ma rzeczywiście załadować."
  - question: "Jak sprawdzić poprawność składni plików konfiguracyjnych Nginx?"
    answer: "Przed każdym przeładowaniem usługi Nginx należy uruchomić polecenie testowe <code>sudo nginx -t</code>, co pozwala uniknąć błędów składniowych mogących wyłączyć serwer."
  - question: "Jak przekierować cały ruch HTTP na HTTPS w Nginx?"
    answer: "W bloku serwera HTTP (nasłuchującym na porcie 80) dodaj dyrektywę przekierowania: <code>return 301 https://$host$request_uri;</code>, aby automatycznie kierować użytkowników na bezpieczny protokół HTTPS."
  - question: "Gdzie znajdują się domyślne pliki dziennika (logi) serwera Nginx?"
    answer: "Wszystkie zdarzenia i błędy są domyślnie zapisywane w katalogu <code>/var/log/nginx/</code>, w plikach <code>access.log</code> (logi dostępu) oraz <code>error.log</code> (logi błędów)."
status: published
locale: pl
author:
  name: VoxiHost Team
  link: https://voxihost.pl/
contributors:
  - danielmarszalkowski
---

Nginx to jeden z najpopularniejszych serwerów WWW na świecie. Oryginalnie napisany aby rozwiązać "problem C10k" (obsługę 10,000 jednoczesnych połączeń), wyewoluował w domyślny wybór dla stron internetowych wysokiej wydajności. Czy to serwujesz statyczne pliki HTML, uruchamiasz API Node.js, czy konfigurujesz odwrotne proxy dla złożonego backendu, Nginx jest dokładnie tym czego potrzebujesz.

Instalacja Nginx na dowolnym serwerze Linux działającym na Ubuntu lub Debian zajmuje mniej niż 15 minut.

## Krok 1: Zainstaluj Nginx

Ponieważ Nginx jest dostępny w domyślnych repozytoriach Ubuntu i Debian, jest prosty do zainstalowania używając menedżera pakietów `apt`.

Przed instalacją jakiegokolwiek nowego oprogramowania, odśwież swój lokalny indeks pakietów:

```bash
sudo apt update
```

Teraz zainstaluj Nginx:

{% image "/assets/images/blog/pl/instalacja-nginx-ubuntu-debian/H1.png", "Uruchamianie sudo apt install nginx -y na Ubuntu aby zainstalować serwer WWW Nginx z apt", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo apt install nginx -y
```

Ubuntu i Debian automatycznie uruchamiają usługę Nginx natychmiast po zakończeniu instalacji.

## Krok 2: Dostosuj zaporę

Jeśli włączyłeś zaporę UFW (jak mocno zalecamy w naszym [Przewodniku Konfiguracji UFW](/pl/blog/konfiguracja-ufw-ubuntu-debian/)), musisz zezwolić na połączenia do Nginx tak aby świat zewnętrzny mógł dotrzeć do twoich stron internetowych.

Podczas instalacji, Nginx rejestruje się z UFW aby dostarczyć kilka automatycznych profili aplikacji. Możesz je zobaczyć wpisując:

{% image "/assets/images/blog/pl/instalacja-nginx-ubuntu-debian/H2.png", "Uruchamianie sudo ufw app list na Ubuntu aby wyświetlić dostępne profile aplikacji zapory Nginx", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo ufw app list
```

Powinieneś zobaczyć:
```text
Available applications:
  Nginx Full
  Nginx HTTP
  Nginx HTTPS
  OpenSSH
```

- **Nginx HTTP**: Otwiera port 80 (normalny, niezaszyfrowany ruch WWW).
- **Nginx HTTPS**: Otwiera port 443 (zaszyfrowany ruch TLS/SSL).
- **Nginx Full**: Otwiera zarówno port 80 jak i 443.

Dla ogromnej większości przypadków, ostatecznie będziesz chciał certyfikat SSL, więc najlepiej zezwolić na oba od razu:

{% image "/assets/images/blog/pl/instalacja-nginx-ubuntu-debian/H3.png", "Uruchamianie sudo ufw allow 'Nginx Full' aby otworzyć port HTTP 80 i HTTPS 443 przez UFW na Ubuntu", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo ufw allow 'Nginx Full'
```

Zweryfikuj że zmiana została pomyślnie zastosowana:

```bash
sudo ufw status
```

## Krok 3: Sprawdź swój serwer WWW

W tym momencie, twój serwer WWW jest w pełni operacyjny. Aby to zweryfikować, otwórz swoją ulubioną przeglądarkę internetową i nawiguj do publicznego adresu IP swojego serwera.

Jeśli nie znasz publicznego adresu IP swojego serwera, możesz go znaleźć uruchamiając:

{% image "/assets/images/blog/pl/instalacja-nginx-ubuntu-debian/H4.png", "Uruchamianie curl -4 icanhazip.com na Ubuntu aby pobrać publiczny adres IP serwera", "(max-width: 768px) 100vw, 800px" %}

```bash
curl -4 icanhazip.com
```

Wpisz ten adres IP w przeglądarce: `http://your_server_ip`

{% image "/assets/images/blog/pl/instalacja-nginx-ubuntu-debian/H5.png", "Domyślna strona powitalna Nginx w przeglądarce potwierdzająca pomyślną instalację na serwerze Ubuntu", "(max-width: 768px) 100vw, 800px" %}

Powinieneś zobaczyć domyślną stronę lądowania **"Welcome to nginx!"**. To potwierdza że oprogramowanie działa poprawnie i że twoja zapora zezwala na ruch.

## Krok 4: Zarządzaj procesem Nginx

Teraz gdy masz swój serwer WWW uruchomiony i działający, musisz wiedzieć jak nim zarządzać. Będziesz używać poleceń `systemctl` do kontrolowania usługi Nginx.

Aby zatrzymać serwer WWW:

{% image "/assets/images/blog/pl/instalacja-nginx-ubuntu-debian/H6.png", "Uruchamianie sudo systemctl stop nginx aby zatrzymać proces serwera WWW Nginx na Ubuntu", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo systemctl stop nginx
```

Aby uruchomić serwer WWW gdy jest zatrzymany:

{% image "/assets/images/blog/pl/instalacja-nginx-ubuntu-debian/H7.png", "Uruchamianie sudo systemctl start nginx aby uruchomić serwer WWW Nginx na Ubuntu lub Debian", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo systemctl start nginx
```

Aby zatrzymać a następnie całkowicie zrestartować usługę:

{% image "/assets/images/blog/pl/instalacja-nginx-ubuntu-debian/H8.png", "Uruchamianie sudo systemctl restart nginx aby całkowicie zrestartować Nginx i zastosować nową konfigurację na Ubuntu", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo systemctl restart nginx
```

Jeśli tylko dokonałeś zmian konfiguracyjnych (jak dodanie nowej domeny), Nginx może przeładować bez rozłączania aktywnych połączeń. To jest polecenie którego będziesz używać najczęściej:

{% image "/assets/images/blog/pl/instalacja-nginx-ubuntu-debian/H9.png", "Uruchamianie sudo systemctl reload nginx aby przeładować konfigurację Nginx bez przerywania aktywnych połączeń", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo systemctl reload nginx
```

## Krok 5: Skonfiguruj Blok Serwera (Wirtualny Host)

Domyślnie, Nginx na Ubuntu ma jeden blok serwera włączony który serwuje dokumenty z katalogu `/var/www/html`. Podczas gdy to działa dobrze dla pojedynczej strony, staje się niezarządzalne jeśli hostujesz wiele stron.

Zamiast modyfikować domyślną konfigurację, powinieneś stworzyć **Blok Serwera** dla swojej domeny (w świecie Apache, te są nazywane Wirtualnymi Hostami).

Dla tego przykładu, użyjemy `your_domain.com`.

### 1. Utwórz katalog dla swojej domeny

{% image "/assets/images/blog/pl/instalacja-nginx-ubuntu-debian/H10.png", "Uruchamianie sudo mkdir -p aby utworzyć nowy katalog główny dokumentów strony internetowej w /var/www na Ubuntu z Nginx", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo mkdir -p /var/www/your_domain.com/html
```

### 2. Przypisz własność katalogu

Przypisz własność do obecnego użytkownika niestandardowego (tak abyś mógł łatwo przesyłać pliki później):
```bash
sudo chown -R $USER:$USER /var/www/your_domain.com/html
```

### 3. Utwórz przykładową stronę index.html

{% image "/assets/images/blog/pl/instalacja-nginx-ubuntu-debian/H11.png", "Tworzenie przykładowej strony testowej index.html za pomocą edytora nano wewnątrz /var/www dla Nginx na Ubuntu", "(max-width: 768px) 100vw, 800px" %}

```bash
nano /var/www/your_domain.com/html/index.html
```

Wklej ten prosty HTML do pliku:
```html
<html>
    <head>
        <title>Witaj na your_domain.com!</title>
    </head>
    <body>
        <h1>Sukces! Blok serwera your_domain.com działa!</h1>
    </body>
</html>
```
Zapisz i zamknij plik.

### 4. Utwórz konfigurację bloku serwera Nginx

Teraz musimy powiedzieć Nginx aby serwował ten folder gdy ktoś odwiedza `your_domain.com`. Utwórz nowy plik konfiguracyjny w katalogu `sites-available`:

{% image "/assets/images/blog/pl/instalacja-nginx-ubuntu-debian/H12.png", "Tworzenie nowego pliku konfiguracyjnego bloku serwera Nginx dla niestandardowej domeny w sites-available na Ubuntu", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo nano /etc/nginx/sites-available/your_domain.com
```

Wklej następującą bazę konfiguracyjną:
```nginx
server {
    listen 80;
    listen [::]:80;

    root /var/www/your_domain.com/html;
    index index.html index.htm index.nginx-debian.html;

    server_name your_domain.com www.your_domain.com;

    location / {
        try_files $uri $uri/ =404;
    }
}
```
Zapisz i zamknij plik.

### 5. Włącz plik poprzez linkowanie go do sites-enabled

Nginx ignoruje pliki w `sites-available` chyba że mają dowiązanie symboliczne umieszczone w katalogu `sites-enabled`. Utwórz to dowiązanie teraz:

{% image "/assets/images/blog/pl/instalacja-nginx-ubuntu-debian/H13.png", "Uruchamianie sudo ln -s aby utworzyć dowiązanie symboliczne z sites-available do sites-enabled aby aktywować wirtualny host Nginx", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo ln -s /etc/nginx/sites-available/your_domain.com /etc/nginx/sites-enabled/
```

### 6. Przetestuj konfigurację i przeładuj

Przed restartem Nginx, zawsze testuj konfigurację aby upewnić się że nie ma błędów składni:

{% image "/assets/images/blog/pl/instalacja-nginx-ubuntu-debian/H14.png", "Uruchamianie sudo nginx -t aby przetestować składnię konfiguracji Nginx przed przeładowaniem serwera WWW na Ubuntu", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo nginx -t
```

Jeśli wynik mówi `syntax is ok` i `test is successful`, przeładuj Nginx aby zastosować zmiany:

```bash
sudo systemctl reload nginx
```

Nginx powinien teraz serwować twoją nazwę domeny. Zakładając że zaktualizowałeś rekordy DNS-A swojej domeny aby wskazywały na adres IP twojego VPS, nawigacja do `http://your_domain.com` pokaże stronę sukcesu którą właśnie stworzyłeś.

Jeśli szukasz wysoce niezawodnego i błyskawicznie szybkiego środowiska do hostowania swojego nowego serwera Nginx, uruchom jedną z naszych instancji [Premium VPS](/pl/premium-vps/) i wdróż swoje projekty już dziś.