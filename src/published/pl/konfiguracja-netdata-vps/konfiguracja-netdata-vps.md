---
image: /assets/images/blog/pl/konfiguracja-netdata-vps/og-image.png
title: Jak skonfigurować Netdata do monitorowania VPS w czasie rzeczywistym
description: Kompletny przewodnik krok po kroku do instalacji Netdata na swoim serwerze Linux VPS. Uzyskaj wysoko szczegółowe, pięknie wygenerowane wykresy metryk dla CPU, RAM, Sieci i Dysku w minutach.
date: '2026-03-25'
translationKey: setup-netdata-vps
category: Poradniki
tags:
  - netdata
  - monitoring
  - linux
  - vps
  - server administration
  - dashboard
  - metrics
howto:
  name: Jak skonfigurować Netdata do monitorowania serwera
  totalTime: PT10M
  yield: Kompleksowy, pięknie wygenerowany panel monitorowania w czasie rzeczywistym dostępny przez przeglądarkę internetową
  tool:
    - VPS lub dedykowany serwer działający na dowolnej głównej dystrybucji Linux
    - Klient SSH (np. terminal, PuTTY)
    - Konto użytkownika z uprawnieniami sudo
  steps:
    - name: Zainstaluj Netdata
      text: 'Uruchom oficjalny skrypt kickstart: wget -O /tmp/netdata-kickstart.sh https://get.netdata.cloud/kickstart.sh && sh /tmp/netdata-kickstart.sh.'
      url: step-1-install-netdata-using-the-kickstart-script
    - name: Skonfiguruj zaporę
      text: Zezwól na port 19999 przez swoją zaporę (np., sudo ufw allow 19999/tcp).
      url: step-2-configure-the-firewall
    - name: Dostęp do panelu
      text: Otwórz przeglądarkę i nawiguj do http://your_server_ip:19999.
      url: step-3-access-your-dashboard
status: published
locale: pl
author:
  name: VoxiHost Team
  link: https://voxihost.pl/
contributors:
  - danielmarszalkowski
---

Narzędzia wiersza poleceń jak [`htop`](/pl/blog/monitorowanie-vps-htop-df-free/) i [`df`](/pl/blog/monitorowanie-vps-htop-df-free/) są doskonałe do szybkiego diagnozowania problemów serwera gdy jesteś zalogowany przez SSH. Ale co jeśli chcesz zobaczyć historię wykresów tego jak twój CPU reagował gdy nagły wzrost ruch na twojej stronie 2 godziny temu?

Dla tego potrzebujesz pełnego pakietu monitoringu.

Podczas gdy zespoły enterprise polegają na złożonych stosach jak Prometheus i Grafana (które są uciążliwe w konfiguracji i trudne w ustawieniu), istnie radykalnie prostsza, natychmiast piękna alternatywa: **Netdata**.

Netdata instaluje się w jednym poleceniu, automatycznie wykrywa wszystkie działające usługi (jak Nginx, Apache, MySQL, Docker) i natychmiast generuje tysiące metryk w czasie rzeczywistym przedstawione w oszałamowym panelu internetowym.

## Krok 1: Zainstaluj Netdata używając skrypt Kickstart

Netdata dostarcza oficjalny, uniwersalnie wspierany skrypt "kickstart" który obsługuje identyfikację architektury, pobiera wymagane zależności i instaluje agenta monitorującego idealnie.

Najpierw pobierz skrypt do folderu tymczasowego i wykonaj go:

{% image "/assets/images/blog/pl/konfiguracja-netdata-vps/H1.png", "Pobieranie i wykonywanie skryptu instalacyjnego Netdata za pomocą wget na serwerze Linux VPS", "(max-width: 768px) 100vw, 800px" %}

```bash
wget -O /tmp/netdata-kickstart.sh https://get.netdata.cloud/kickstart.sh && sh /tmp/netdata-kickstart.sh
```

Skrypt poprosi o potwierdzenie. Wpisz `Y` aby zatwierdzić.

Skrypt obsługuje wszystko niewidzialnie w tle. Gdy instalacja się zakończy, Netdata automatycznie rejestruje się jako usługa systemd, uruchamia swoje demony i konfigurując się do automatycznego uruchamienia przy każdym starcie serwera.

Sprawdź czy działa poprawnie sprawdzając status usługi:

{% image "/assets/images/blog/pl/konfiguracja-netdata-vps/H2.png", "Sprawdzanie statusu usługi Netdata za pomocą systemctl aby zweryfikować że demon monitorujący działa aktywnie w tle na serwerze Linux VPS", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo systemctl status netdata
```

Szukaj tekstu `active (running)`.

## Krok 2: Skonfiguruj zaporę

Netdata tworzy lekki serwer WWW wyłącznie do serwowania swojego panelu, domyślnie nasłuchując na **Porcie 19999**.

Ponieważ prawdopodobnie używasz zaporę (którą powinieneś robić), musisz jawnie zezwolić na ten port:

Jeśli używasz [UFW](/pl/blog/konfiguracja-ufw-ubuntu-debian/) (Ubuntu/Debian):
```bash
sudo ufw allow 19999/tcp
```

Jeśli używasz [firewalld](/pl/blog/konfiguracja-firewalld-centos-rhel/) (AlmaLinux/CentOS/Fedora):
```bash
sudo firewall-cmd --permanent --add-port=19999/tcp
sudo firewall-cmd --reload
```

## Krok 3: Dostęp do panelu

Jesteś całkowicie skonfigurowany!

Otwórz swoją ulubioną przeglądarkę internetową i nawiguj do adresu IP swojego serwera, dodając `:19999` na końcu adresu:

`http://your_server_ip:19999`

{% image "/assets/images/blog/pl/konfiguracja-netdata-vps/H3.png", "Pięknie wygenerowany panel monitorowania Netdata wczytany w przeglądarce internetowej pokazujący wykresy metryk serwera VPS w czasie rzeczywistym", "(max-width: 768px) 100vw, 800px" %}

Zostaniesz natychmiast załadowany bezpośrednio do **Lokalnego Panelu Netdata**. Bez haseł, bez konfiguracji, bez oczekiwania. Wszystkie wykresy są generowane na żywo.

Przewiń w dół prawej strony ekranu. Zobaczysz:
- **Użycie CPU przez aktywne rdzenie**: Wykresy pokazują które rdzenie CPU są aktywne i jak bardzo są obciążone.
- **Twarda I/O (odczyt/zapis)**: Wykresy szybkości dysku, pokazują operacje odczytu/zapisu w czasie rzeczywistym.
- **Całkowita i dostępna pamięć (RAM)**: Wykresy pokazują ile pamięci jest używane, buforowane i dostępne dla nowych aplikacji.
- **Przepustowość interfejsów sieciowych**: Wykresy pasma przychodzące i wychodzące dane przez poszczególne interfejsy sieciowe.
- **Procesy i kontenery**: Statystyki działających kontenerów Docker, jeśli używasz konteneryzację.
- **Przerwywania**: Wykresy pokazują systemowe przerwywania i zdarzenia.

### Uwaga bezpieczeństwa

Domyślnie panel Netdata jest dostępny dla każdego kto zna adres IP twojego serwera i numer portu. Podczas gdy mogą zobaczyć jakie oprogramowanie uruchamiasz na podstawie identyfikacji wykresów (np., widząc że używasz MySQL do atakujących), nie mogą zobaczyć twoich haseł ani prywatnego kodu.

Jeśli uruchamiasz serwer produkcyjny, jest wysoce zalecane aby ostatecznie powiązać Netdata z `localhost` i uzyskać dostęp przez odwrotne proxy (używając bloku serwera Nginx) z monitem o hasle.

Jednakże dla środowiska testowego lub deweloperskiego, pozostawienie portu otwartego jest w porządku do szybkiego monitorowania.

Jeśli chcesz zanurzyć w złożone metryki wydajności, wdróż intensywne aplikacje, zainstaluj Netdata na niezwykle wydajnym [Premium VPS](/pl/premium-vps/), uruchom swoje kontenery Docker i obserwuj jak wykresy tańczą w idealnej harmonii.