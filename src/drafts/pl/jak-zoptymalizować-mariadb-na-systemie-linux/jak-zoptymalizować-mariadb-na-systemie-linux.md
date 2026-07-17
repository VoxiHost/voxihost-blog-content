---
image: /assets/images/blog/pl/jak-zoptymalizować-mariadb-na-systemie-linux/og-image.png
title: "Optymalizacja wydajności MariaDB na Linux"
description: "Dowiedz się jak zoptymalizować MariaDB na systemie Linux poprzez tuning ustawień, wyłączenie Transparent Huge Pages oraz konfigurację silnika InnoDB na VPS."
status: draft
category: Poradniki
tags:
  - mariadb
  - linux
  - mysql
  - server
  - vps
  - hosting
date: '2026-07-23'
locale: pl
translationKey: optimize-mariadb-performance-linux
author:
  name: VoxiHost Team
  link: https://voxihost.pl/
contributors: []
howto:
  name: "Optymalizacja wydajności MariaDB na Linux"
  steps:
    - name: "Krok 1: Instalacja MariaDB z oficjalnego repozytorium"
      url: "krok-1-instalacja-mariadb-z-oficjalnego-repozytorium"
    - name: "Krok 2: Optymalizacja wydajności na poziomie systemu"
      url: "krok-2-optymalizacja-wydajności-na-poziomie-systemu"
    - name: "Krok 3: Wyłączenie Transparent Huge Pages"
      url: "krok-3-wyłączenie-transparent-huge-pages"
    - name: "Krok 4: Konfiguracja ustawień InnoDB w MariaDB"
      url: "krok-4-konfiguracja-ustawień-innodb-w-mariadb"
faq:
  - question: "Dlaczego MariaDB wymaga restartu po zmianie innodb_buffer_pool_size?"
    answer: "Parametr <code>innodb_buffer_pool_size</code> definiuje obszar pamięci, w którym przechowywane są dane i indeksy. Ponieważ pamięć ta jest alokowana podczas uruchamiania, VoxiHost zaleca restart usługi, aby baza danych poprawnie zainicjowała nową alokację."
  - question: "Jak sprawdzić, czy Transparent Huge Pages są wyłączone na serwerze?"
    answer: "Status można zweryfikować uruchamiając <code>cat /sys/kernel/mm/transparent_hugepage/enabled</code>. Jeśli wynik wskazuje <code>[never]</code>, funkcja jest pomyślnie wyłączona."
  - question: "Jaka jest zalecana wartość swappiness dla serwera bazodanowego?"
    answer: "Dla środowisk produkcyjnych zalecamy wartość <code>swappiness</code> na poziomie <strong>10</strong>. Wymusza to na jądrze systemu utrzymywanie danych w fizycznej pamięci RAM zamiast przenoszenia ich na dysk."
  - question: "Czy zmiana innodb_snapshot_isolation wpływa na starsze aplikacje?"
    answer: "Tak, zmiana domyślnego ustawienia na <code>ON</code> w MariaDB 12.3 może zmienić zachowanie transakcji w starszym oprogramowaniu. Zawsze testuj aplikację w środowisku stagingowym na Premium VPS przed wdrożeniem zmian na produkcję."
  - question: "Jak sprawdzić, czy moja usługa MariaDB korzysta z najnowszego wydania LTS?"
    answer: "Wersję można sprawdzić poleceniem <code>mariadb --version</code>. Jeśli instalacja została przeprowadzona z oficjalnego repozytorium, polecenia <code>apt update && apt upgrade</code> zapewnią utrzymanie najnowszej stabilnej gałęzi LTS."
---

Wydajność bazy danych często wyznacza górną granicę szybkości działania aplikacji. Chociaż świeża instalacja MariaDB działa od razu po uruchomieniu, domyślne konfiguracje są zaprojektowane pod kątem kompatybilności, a nie surowej przepustowości. Na serwerze <span class="text-white">Voxi</span><span class="text-amber-300">Host</span> [Premium VPS](/pl/premium-vps/) dysponujesz mocą obliczeniową pozwalającą na obsługę tysięcy jednoczesnych zapytań, jednak nieefektywne zarządzanie pamięcią lub wąskie gardła na poziomie systemu mogą szybko zniwelować tę przewagę.

Optymalizacja MariaDB wymaga podejścia wielowarstwowego. Nie wystarczy dostosować kilku ustawień bufora, aby uzyskać oczekiwane rezultaty; należy dostroić system operacyjny, aby nie stanowił przeszkody dla silnika bazy danych. Obejmuje to zarządzanie sposobem, w jaki jądro przydziela pamięć, oraz zapewnienie, że limity deskryptorów plików są wystarczająco wysokie, aby obsłużyć duże obciążenie połączeniami.

Wprowadzimy optymalizację systemu nastawioną na wydajność, wyłączymy konfliktowe funkcje jądra, takie jak Transparent Huge Pages, oraz zainstalujemy najnowszą stabilną wersję MariaDB bezpośrednio z oficjalnych repozytoriów. Twoja baza danych zostanie skonfigurowana do obsługi wymagających obciążeń produkcyjnych przy znacznie niższych opóźnieniach i zwiększonej stabilności. Niezależnie od tego, czy uruchamiasz aplikację internetową o dużym natężeniu ruchu, czy wymagającą usługę danych, te zmiany stworzą fundament dla sprawnego i wydajnego środowiska bazodanowego.

{% image "/assets/images/blog/pl/jak-zoptymalizować-mariadb-na-systemie-linux/H1.png", "Okno terminala pokazujące sprawdzenie statusu usługi MariaDB na serwerze Linux", "(max-width: 768px) 100vw, 800px" %}

## Wymagania wstępne

Zanim rozpoczniemy proces optymalizacji, należy upewnić się, że środowisko spełnia niezbędne założenia techniczne. Serwer powinien działać pod kontrolą systemu Ubuntu 22.04 lub Debian 12. Choć MariaDB może działać na słabszych konfiguracjach, zdecydowanie zalecamy posiadanie co najmniej 2 GB pamięci RAM oraz 2 rdzeni procesora, aby uniknąć rywalizacji o zasoby w okresach dużego obciążenia.

Dostęp do serwera musi odbywać się za pośrednictwem konta użytkownika z uprawnieniami sudo. Jeśli korzystasz z serwera <span class="text-white">Voxi</span><span class="text-amber-300">Host</span> [Premium VPS](/pl/premium-vps/) lub [Budget VPS](/pl/budget-vps/), prawdopodobnie masz już taką konfigurację. Należy upewnić się, że zapora sieciowa jest skonfigurowana w taki sposób, aby zezwalać na połączenia z bazą danych tylko z zaufanych źródeł. Jeśli instancja nie została jeszcze zabezpieczona, warto przejrzeć nasz poradnik dotyczący [zabezpieczania zapory sieciowej serwera Linux](/pl/jak-zabezpieczyc-ssh-ubuntu-debian/) przed otwarciem jakichkolwiek portów bazy danych dla ruchu publicznego.

Na koniec należy zweryfikować, czy system jest aktualny. Do wykonania kroków konfiguracyjnych zawartych w tym przewodniku wymagane będą podstawowe narzędzia administracyjne. Jeśli serwer jest „czysty”, należy upewnić się, że lista pakietów została odświeżona, a zegar systemowy jest zsynchronizowany, aby zapobiec problemom z logowaniem transakcji.

{% image "/assets/images/blog/pl/jak-zoptymalizowac-mariadb-na-systemie-linux/H2.png", "Dane wyjściowe terminala wyświetlające sprawdzenie wymagań systemowych dla serwera bazy danych Linux", "(max-width: 768px) 100vw, 800px" %}

## Krok 1: Instalacja MariaDB z oficjalnego repozytorium

Aby zapewnić dostęp do najnowszych wydań LTS oraz poprawek bezpieczeństwa, pobierzemy MariaDB bezpośrednio z oficjalnego repozytorium. Korzystanie z domyślnych repozytoriów systemowych często skutkuje instalacją nieaktualnej wersji, co może prowadzić do problemów z kompatybilnością nowoczesnych funkcji wydajnościowych.

Zespół MariaDB udostępnia skrypt instalacyjny, który automatycznie wykrywa dystrybucję i konfiguruje odpowiednie pliki repozytorium. Należy wykonać poniższe polecenia, aby dodać oficjalne repozytorium i zainstalować niezbędne pakiety bazodanowe:

```bash
## Dodaj oficjalne repozytorium MariaDB i zaktualizuj listy pakietów
curl -LsS https://downloads.mariadb.com/MariaDB/mariadb_repo_setup | sudo bash
sudo apt update

## Zainstaluj pakiety serwera i klienta MariaDB
sudo apt install -y mariadb-server mariadb-client
```

Po zakończeniu instalacji usługa bazy danych uruchomi się automatycznie. Można zweryfikować, czy usługa jest aktywna i działa poprawnie, sprawdzając jej status:

```bash
## Potwierdź, że usługa MariaDB jest obecnie aktywna
systemctl status mariadb
```

W celu zabezpieczenia instalacji należy uruchomić dostarczony skrypt bezpieczeństwa. Skrypt ten usuwa anonimowych użytkowników, wyłącza zdalne logowanie dla użytkownika root oraz usuwa testowe bazy danych:

```bash
## Uruchom skrypt zabezpieczający instalację
sudo mysql_secure_installation
```

{% image "/assets/images/blog/pl/jak-zoptymalizować-mariadb-na-systemie-linux/H3.png", "Dane wyjściowe terminala pokazujące pomyślną instalację i wykonanie skryptu zabezpieczającego serwer MariaDB", "(max-width: 768px) 100vw, 800px" %}

## Krok 2: Optymalizacja wydajności na poziomie systemu

Przed dostosowaniem plików konfiguracyjnych MariaDB należy zająć się sposobem, w jaki system operacyjny zarządza pamięcią i zasobami plików. Domyślne ustawienia systemu Linux są często zbyt ogólne dla obciążeń bazodanowych, co może prowadzić do niepotrzebnego swapowania na dysk lub problemów z limitem deskryptorów plików, gdy serwer <span class="text-white">Voxi</span><span class="text-amber-300">Host</span> odnotowuje wysoki ruch.

Następnie należy dostosować zachowanie pamięci wymiany (swap) jądra systemu oraz zwiększyć limity deskryptorów plików. Obniżając wartość `vm.swappiness`, nakazujemy systemowi priorytetowe utrzymywanie bazy danych w pamięci RAM, zamiast przenoszenia jej na dysk. Zwiększamy również limit otwartych plików, aby zapewnić, że serwer obsłuży dużą liczbę jednoczesnych połączeń bez osiągania limitów systemowych.

```bash
## Ustawienie swappiness na 10, aby ograniczyć użycie partycji wymiany
echo "vm.swappiness=10" | sudo tee /etc/sysctl.d/99-mariadb-tuning.conf
sudo sysctl --system

## Zwiększenie limitów deskryptorów plików dla lepszej obsługi współbieżności
echo -e "* soft nofile 65536\n* hard nofile 65536" | sudo tee -a /etc/security/limits.conf
```

Powyższe zmiany gwarantują, że podstawowe środowisko Linux jest zoptymalizowane pod kątem obsługi wydajnych operacji bazodanowych. Dzięki zmniejszeniu prawdopodobieństwa zrzucania pamięci przez jądro do pliku wymiany, baza danych pozostanie responsywna nawet przy dużym obciążeniu zapytaniami.

{% image "/assets/images/blog/pl/jak-zoptymalizowac-mariadb-na-systemie-linux/H4.png", "Dane wyjściowe terminala pokazujące zastosowanie optymalizacji sysctl oraz limitów deskryptorów plików", "(max-width: 768px) 100vw, 800px" %}

## Krok 3: Wyłączenie Transparent Huge Pages

Musimy skupić się konkretnie na Transparent Huge Pages (THP). Choć THP ma na celu uproszczenie zarządzania pamięcią w aplikacjach na dużą skalę, jej agresywna strategia przydzielania pamięci często koliduje ze sposobem, w jaki InnoDB zarządza swoją pulą buforów.

Gdy THP jest aktywne, jądro systemu Linux może powodować znaczne skoki opóźnień podczas operacji bazodanowych, ponieważ system próbuje scalać strony pamięci w ogromne strony (huge pages). W przypadku serwera bazy danych działającego na <span class="text-white">Voxi</span><span class="text-amber-300">Host</span> [Premium VPS](/pl/premium-vps/), wyłączenie tej funkcji jest standardową praktyką, która zapewnia przewidywalną wydajność i minimalizuje wewnętrzne konflikty o pamięć.

Aby ta konfiguracja była trwała po ponownym uruchomieniu, tworzymy plik usługi systemd:

```bash
## Utwórz usługę systemd, aby wyłączyć THP przy starcie systemu
cat <<EOF | sudo tee /etc/systemd/system/disable-thp.service
[Unit]
Description=Disable Transparent Huge Pages (THP)
After=sysinit.target

[Service]
Type=oneshot
ExecStart=/bin/sh -c 'echo never > /sys/kernel/mm/transparent_hugepage/enabled && echo never > /sys/kernel/mm/transparent_hugepage/defrag'

[Install]
WantedBy=multi-user.target
EOF

## Włącz i uruchom usługę
sudo systemctl daemon-reload
sudo systemctl enable --now disable-thp.service
```

{% image "/assets/images/blog/pl/jak-zoptymalizowac-mariadb-na-systemie-linux/H5.png", "Terminal pokazujący tworzenie usługi systemd w celu wyłączenia transparent huge pages", "(max-width: 768px) 100vw, 800px" %}

## Krok 4: Konfiguracja ustawień InnoDB w MariaDB

Sercem wydajności MariaDB jest silnik pamięci masowej InnoDB. Zmienna `innodb_buffer_pool_size` jest najważniejszym parametrem, który należy dostosować. Określa ona, ile pamięci RAM jest przydzielane na buforowanie danych oraz indeksów. W przypadku dedykowanego serwera bazy danych, przyjętą praktyką jest ustawienie tej wartości na 70-80% dostępnej pamięci RAM systemu.

Otwórz plik konfiguracyjny, aby rozpocząć proces optymalizacji:

```bash
## Otwórz główny plik konfiguracyjny serwera MariaDB
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
```

Znajdź sekcję `[mysqld]` i dodaj lub dostosuj poniższe dyrektywy. Upewnij się, że zastąpisz `[RAM_SIZE]` wartością odpowiednią dla Twojej instancji <span class="text-white">Voxi</span><span class="text-amber-300">Host</span> [Premium VPS](/pl/premium-vps/):

```ini
[mysqld]
## Ustaw pulę bufora na 75% całkowitej pamięci RAM systemu
innodb_buffer_pool_size = 6G
## Zapewnij wydajne zapisywanie logów na dysk
innodb_flush_log_at_trx_commit = 2
## Zoptymalizuj liczbę wątków wejścia/wyjścia (I/O)
innodb_write_io_threads = 8
innodb_read_io_threads = 8
```

> **Ostrzeżenie:** Zmiana parametru `innodb_buffer_pool_size` wymaga pełnego restartu usługi, aby zmiany weszły w życie. Ustawienie zbyt wysokiej wartości niesie ze sobą ryzyko wywołania mechanizmu OOM (Out of Memory) killer w systemie, co spowoduje awarię bazy danych.

Po zapisaniu zmian za pomocą `Ctrl+O` i `Enter` oraz wyjściu z edytora przy użyciu `Ctrl+X`, zrestartuj usługę, aby zastosować nową konfigurację:

```bash
## Zrestartuj MariaDB, aby zastosować nowe ustawienia pamięci InnoDB
sudo systemctl restart mariadb
```

Ustawienie `innodb_flush_log_at_trx_commit = 2` zapewnia znaczny wzrost wydajności w aplikacjach intensywnie zapisujących dane, poprzez zrzucanie logów do pamięci podręcznej systemu operacyjnego co sekundę, a nie przy każdym pojedynczym zatwierdzeniu transakcji (commit). Jest to bezpieczne rozwiązanie dla większości aplikacji internetowych, jednak należy przywrócić wartość `1`, jeśli obciążenie Twojej bazy wymaga ścisłej zgodności z zasadami ACID dla każdej transakcji.

{% image "/assets/images/blog/pl/jak-zoptymalizowac-mariadb-na-systemie-linux/H6.png", "Edycja pliku konfiguracyjnego serwera MariaDB w celu dostosowania ustawień puli bufora InnoDB", "(max-width: 768px) 100vw, 800px" %}

## Podsumowanie

Optymalizacja MariaDB to proces iteracyjny. Wyłączenie Transparent Huge Pages, dostrojenie parametrów systemowych, takich jak swappiness, oraz właściwe dobranie rozmiaru InnoDB buffer pool pozwala na stworzenie solidnych fundamentów pod wydajną pracę bazy danych. Te zmiany zapewniają, że Twój serwer <span class="text-white">Voxi</span><span class="text-amber-300">Host</span> zarządza zasobami efektywnie, co redukuje opóźnienia i zapobiega powstawaniu wąskich gardeł przy dużym obciążeniu.

Warto pamiętać, że wydajność bazy danych rzadko bywa stała. W miarę wzrostu zbioru danych lub zmiany wzorców użytkowania aplikacji, należy okresowo monitorować metryki za pomocą narzędzi takich jak `htop` lub polecenia `SHOW ENGINE INNODB STATUS;` wewnątrz klienta MariaDB. Utrzymywanie serwera w aktualnym stanie jest równie istotne; zawsze należy sprawdzać najnowsze stabilne wydania, aby korzystać z bieżących poprawek wydajności wprowadzanych przez twórców oprogramowania.

Regularna konserwacja Twojego serwera [Premium VPS](/pl/premium-vps/) sprawi, że usługi będą działać płynnie. W przypadku zarządzania bardziej złożonymi środowiskami webowymi, przydatny może okazać się nasz poradnik dotyczący tego, [jak zainstalować stos LEMP (Linux, Nginx, MariaDB, PHP) na systemach Ubuntu i Debian](/pl/instalacja-lemp-ubuntu-debian/), co pozwoli na integrację zoptymalizowanej bazy danych z solidnym serwerem WWW. Prawidłowa konfiguracja na tym etapie pozwala zaoszczędzić znaczną ilość czasu, który w przeciwnym razie musiałby zostać poświęcony na rozwiązywanie problemów w przyszłości.

{% image "/assets/images/blog/pl/jak-zoptymalizowac-mariadb-na-systemie-linux/H7.png", "Terminal pokazujący udany restart usługi MariaDB po optymalizacji wydajności", "(max-width: 768px) 100vw, 800px" %}
