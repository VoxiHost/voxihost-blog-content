---
image: /assets/images/blog/pl/najszybszy-serwer-lustrzany-apt-ubuntu/og-image.png
title: 'Jak znaleźć i używać najszybszego serwera lustrzanego APT dla Ubuntu'
description: Przeprowadzaj aktualizacje systemu i instalacje oprogramowania szybciej, konfigurując serwer Ubuntu do automatycznego korzystania z najszybszego regionalnego serwera lustrzanego APT.
date: '2026-06-17'
translationKey: fastest-apt-mirror-ubuntu
locale: pl
category: Poradniki
tags:
  - ubuntu
  - linux
  - apt
  - performance
status: draft
author:
  name: Anduin
  link: https://github.com/Anduin2017
contributors:
  - Anduin2017
  - danielmarszalkowski
howto:
  name: Znajdowanie i ustawianie najszybszego serwera lustrzanego APT na Ubuntu
  totalTime: PT10M
  yield: Szybsze pobieranie pakietów przez apt update i apt install dzięki optymalnemu serwerowi lustrzanemu.
  tool:
    - VPS z systemem Ubuntu
    - Dostęp SSH z uprawnieniami sudo
  steps:
    - name: "Krok 1: Utworzenie skryptu wyboru serwera lustrzanego"
      text: Utwórz skrypt bash testujący czas reakcji wielu globalnych serwerów lustrzanych.
      url: krok-1-utworzenie-skryptu-wyboru-serwera-lustrzanego
    - name: "Krok 2: Nadanie uprawnień do wykonania skryptu"
      text: Nadaj skryptowi uprawnienia do wykonywania.
      url: krok-2-nadanie-uprawnien-do-wykonania-skryptu
    - name: "Krok 3: Uruchomienie skryptu"
      text: Uruchom skrypt, aby przetestować serwery i zaktualizować listę źródeł APT.
      url: krok-3-uruchomienie-skryptu
faq:
  - question: "W jaki sposób skrypt wyznacza najszybszy mirror?"
    answer: "Skrypt wysyła zapytania za pomocą narzędzia <code>curl</code> do pliku <code>Release</code> każdego z serwerów lustrzanych. Zostaje wybrany ten serwer, który wykazuje najkrótszy całkowity czas połączenia (najniższe opóźnienia)."
  - question: "Czym jest nowoczesny format źródeł APT (ubuntu.sources)?"
    answer: "Począwszy od wersji Ubuntu 24.04 LTS, systemy Ubuntu domyślnie korzystają z formatu DEB822 (w pliku <code>/etc/apt/sources.list.d/ubuntu.sources</code>) zamiast tradycyjnego pliku <code>/etc/apt/sources.list</code>."
  - question: "Czy skrypt tworzy kopię zapasową starych konfiguracji APT?"
    answer: "Tak, skrypt automatycznie wykonuje kopię zapasową dotychczasowych źródeł jako <code>/etc/apt/sources.list.bak</code> przed naniesieniem jakichkolwiek modyfikacji."
---

## Wprowadzenie

Po wdrożeniu nowego serwera z systemem Ubuntu domyślna konfiguracja menedżera pakietów APT zazwyczaj wskazuje na główne, globalne serwery lustrzane (mirrory). W zależności od fizycznej lokalizacji Twojego serwera VPS w <span class="text-white">Voxi</span><span class="text-amber-300">Host</span>, może to spowolnić pobieranie pakietów podczas operacji `apt update` oraz `apt install`.

Aby uzyskać najlepszą przepustowość, warto skonfigurować APT do korzystania z najszybszego serwera lustrzanego w Twoim regionie. W tym poradniku udostępniamy prosty skrypt, który automatycznie przetestuje kilkanaście globalnych mirrorów, wybierze ten o najniższych opóźnieniach i zaktualizuje pliki konfiguracyjne (wspierając tradycyjny format `sources.list` oraz nowoczesny standard `ubuntu.sources`).

---

## Krok 1: Utworzenie skryptu wyboru serwera lustrzanego

Do przetestowania czasów odpowiedzi poszczególnych serwerów lustrzanych za pomocą narzędzia `curl` użyjemy skryptu bash.

Utwórz nowy plik o nazwie `fastest-mirror.sh` za pomocą wybranego edytora tekstu:

```bash
nano fastest-mirror.sh
```

Wklej do pliku następujący kod skryptu:

```bash
#!/bin/bash

# Sprawdzenie bieżącego formatu źródeł APT
check_apt_format() {
    local old_format=false
    local new_format=false
    
    if [ -f "/etc/apt/sources.list" ]; then
        if grep -v '^#' /etc/apt/sources.list | grep -q '[^[:space:]]'; then
            old_format=true
        fi
    fi
    
    if [ -f "/etc/apt/sources.list.d/ubuntu.sources" ]; then
        if grep -v '^#' /etc/apt/sources.list.d/ubuntu.sources | grep -q '[^[:space:]]'; then
            new_format=true
        fi
    fi
    
    if $old_format && $new_format; then
        echo "both"
    elif $old_format; then
        echo "old"
    elif $new_format; then
        echo "new"
    else
        echo "none"
    fi
}

# Znajdowanie najszybszego serwera lustrzanego
find_fastest_mirror() {
    echo "Testowanie prędkości mirrorów..." >&2
    codename=$(lsb_release -cs)
    
    mirrors=(
        "http://archive.ubuntu.com/ubuntu/"
        "https://mirror.i3d.net/pub/ubuntu/"
        "https://mirroronet.pl/pub/mirrors/ubuntu/"
        "http://us.archive.ubuntu.com/ubuntu/"
        "http://uk.archive.ubuntu.com/ubuntu/"
        "http://de.archive.ubuntu.com/ubuntu/"
        "https://ftp.uni-stuttgart.de/ubuntu/"
        "https://mirror.ubuntu.ikoula.com/"
    )
    
    declare -A results
    
    for mirror in "${mirrors[@]}"; do
        echo "Testowanie $mirror ..." >&2
        response="$(curl -o /dev/null -s -w "%{http_code} %{time_total}\n" \
                  --connect-timeout 2 --max-time 3 "${mirror}dists/${codename}/Release")"
        
        http_code=$(echo "$response" | awk '{print $1}')
        time_total=$(echo "$response" | awk '{print $2}')
        
        if [ "$http_code" -eq 200 ]; then
            results["$mirror"]="$time_total"
        else
            results["$mirror"]="9999"
        fi
    done
    
    sorted_mirrors="$(
        for url in "${!results[@]}"; do
            echo "$url ${results[$url]}"
        done | sort -k2 -n
    )"
    
    fastest_mirror="$(echo "$sorted_mirrors" | head -n 1 | awk '{print $1}')"
    
    if [[ "$fastest_mirror" == "" || "${results[$fastest_mirror]}" == "9999" ]]; then
        fastest_mirror="http://archive.ubuntu.com/ubuntu/"
    fi
    
    echo "$fastest_mirror"
}

# Generowanie pliku źródeł w nowym formacie (ubuntu.sources)
generate_new_format() {
    local mirror="$1"
    local codename="$2"
    
    echo "Generowanie nowej listy źródeł /etc/apt/sources.list.d/ubuntu.sources"
    
    sudo tee /etc/apt/sources.list.d/ubuntu.sources >/dev/null <<EOF
Types: deb
URIs: $mirror
Suites: $codename $codename-updates $codename-backports $codename-security
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg
EOF
}

# Generowanie pliku źródeł w starym formacie (sources.list)
generate_old_format() {
    local mirror="$1"
    local codename="$2"
    
    echo "Generowanie starej listy źródeł /etc/apt/sources.list"
    
    sudo tee /etc/apt/sources.list >/dev/null <<EOF
deb $mirror $codename main restricted universe multiverse
deb $mirror $codename-updates main restricted universe multiverse
deb $mirror $codename-backports main restricted universe multiverse
deb $mirror $codename-security main restricted universe multiverse
EOF
}

main() {
    sudo apt update
    sudo apt install -y curl lsb-release
    
    format=$(check_apt_format)
    codename=$(lsb_release -cs)
    
    fastest_mirror=$(find_fastest_mirror)
    echo "Znaleziono najszybszy mirror: $fastest_mirror"
    
    case "$format" in
        "old"|"none")
            generate_old_format "$fastest_mirror" "$codename"
            ;;
        "new"|"both")
            if [ "$format" == "both" ]; then
                sudo mv /etc/apt/sources.list /etc/apt/sources.list.bak
            fi
            generate_new_format "$fastest_mirror" "$codename"
            ;;
    esac
    
    sudo apt update
    echo "Optymalizacja źródeł APT zakończona pomyślnie!"
}

main
```

> **Uwaga:** W powyższym skrypcie celowo skróciliśmy listę mirrorów dla przejrzystości kodu, skupiając się na głównych serwerach europejskich i amerykańskich (idealnych dla lokalizacji [Premium VPS](/pl/premium-vps/)). Możesz łatwo dopisać do tablicy `mirrors` dodatkowe regionalne adresy URL, jeśli Twój serwer znajduje się w innej części świata.

{% image "/assets/images/blog/pl/najszybszy-serwer-lustrzany-apt-ubuntu/H1.png", "Terminal pokazujący uruchomienie skryptu sprawdzającego opóźnienia serwerów lustrzanych", "(max-width: 768px) 100vw, 800px" %}

---

## Krok 2: Nadanie uprawnień do wykonania skryptu

Zapisz i zamknij plik, a następnie nadaj skryptowi uprawnienia do uruchamiania:

```bash
chmod +x fastest-mirror.sh
```

---

## Krok 3: Uruchomienie skryptu

Uruchom przygotowany skrypt. Automatycznie zainstaluje on narzędzia `curl` i `lsb-release` (jeśli nie są jeszcze obecne w systemie), przetestuje czasy odpowiedzi serwerów, wybierze najszybszy z nich i nadpisze pliki konfiguracyjne APT w formacie odpowiednim dla używanej wersji Ubuntu.

```bash
./fastest-mirror.sh
```

Na ekranie powinien pojawić się komunikat zbliżony do poniższego:

```
Testowanie prędkości mirrorów...
Testowanie http://archive.ubuntu.com/ubuntu/ ...
Testowanie https://mirroronet.pl/pub/mirrors/ubuntu/ ...
Znaleziono najszybszy mirror: https://mirroronet.pl/pub/mirrors/ubuntu/
Generowanie nowej listy źródeł /etc/apt/sources.list.d/ubuntu.sources
Hit:1 https://mirroronet.pl/pub/mirrors/ubuntu noble InRelease
...
Optymalizacja źródeł APT zakończona pomyślnie!
```

{% image "/assets/images/blog/pl/najszybszy-serwer-lustrzany-apt-ubuntu/H2.png", "Konsola przedstawiająca zaktualizowany plik ubuntu.sources wskazujący na najszybszy mirror", "(max-width: 768px) 100vw, 800px" %}

---

## Podsumowanie

Twój serwer pobiera teraz pakiety z najszybszego regionalnego serwera lustrzanego, co znacząco przyspieszy instalowanie programów oraz codzienne aktualizacje bezpieczeństwa.

Szukasz wydajnego hostingu w Europie? Uruchom serwer [<span class="text-white">Voxi</span><span class="text-amber-300">Host</span> Budget VPS](/pl/budget-vps/) już dzisiaj i przekonaj się, jak szybko działa połączenie pamięci NVMe z optymalną ścieżką routingu sieciowego.
