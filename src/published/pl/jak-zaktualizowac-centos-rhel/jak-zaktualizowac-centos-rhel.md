---
image: /assets/images/blog/pl/jak-zaktualizowac-centos-rhel/og-image.png
title: 'Jak aktualizować AlmaLinux, CentOS Stream i Rocky Linux: Kompletny przewodnik serwera'
description: Kompletny przewodnik krok po kroku do aktualizacji serwerów AlmaLinux 9/10, CentOS Stream 9/10 i Rocky Linux 9/10. Obejmuje dnf update, dnf upgrade, autoremove, reboot detection i dnf-automatic dla środowisk produkcyjnych VPS.
date: '2026-03-25'
translationKey: update-almalinux-centos-rocky
category: Poradniki
tags:
  - almalinux
  - centos
  - rocky linux
  - dnf update
  - dnf upgrade
  - linux
  - vps
  - server administration
  - dnf-automatic
  - rhel
howto:
  name: Jak aktualizować AlmaLinux, CentOS Stream i Rocky Linux
  totalTime: PT5M
  yield: W pełni zaktualizowany serwer AlmaLinux, CentOS Stream lub Rocky Linux z najnowszymi poprawkami bezpieczeństwa
  tool:
    - Serwer Linux VPS lub dedykowany
    - Dostęp SSH lub root
  steps:
    - name: Sprawdź dostępne aktualizacje
      text: Uruchom sudo dnf check-update aby zobaczyć jakie aktualizacje są dostępne bez instalowania czegokolwiek.
      url: sprawdzenie-dostepnych-aktualizacji
    - name: Zainstaluj wszystkie dostępne aktualizacje
      text: Uruchom sudo dnf update -y aby pobrać i zainstalować wszystkie oczekujące aktualizacje pakietów.
      url: instalacja-wszystkich-dostepnych-aktualizacji
    - name: Usuń osierocone pakiety
      text: Uruchom sudo dnf autoremove -y aby wyczyścić stare biblioteki i zależności które nie są już potrzebne.
      url: czyszczenie-starych-pakietow
    - name: Sprawdź czy wymagany jest restart
      text: Uruchom sudo needs-restarting -r aby sprawdzić czy aktualizacja jądra wymaga restartu systemu.
      url: sprawdzanie-czy-wymagany-jest-restart
    - name: Włącz automatyczne aktualizacje bezpieczeństwa
      text: Zainstaluj i skonfiguruj dnf-automatic aby automatycznie stosować poprawki bezpieczeństwa.
      url: wlaczanie-automatycznych-aktualizacji-bezpieczenstwa
faq:
  - question: "Jaka jest różnica między dnf update a dnf upgrade?"
    answer: "W nowoczesnym menedżerze pakietów DNF (używanym w systemach AlmaLinux, Rocky Linux i CentOS Stream) polecenia <code>dnf update</code> i <code>dnf upgrade</code> są aliasami i wykonują dokładnie to samo działanie."
  - question: "Jak zainstalować wyłącznie aktualizacje bezpieczeństwa?"
    answer: "Możesz ograniczyć proces aktualizacji tylko do pakietów zawierających poprawki bezpieczeństwa za pomocą polecenia <code>sudo dnf upgrade --security</code>."
  - question: "Czy bezpieczne jest aktualizowanie serwera produkcyjnego?"
    answer: "Tak, choć zaleca się wykonywanie aktualizacji w godzinach niskiego ruchu, wcześniejsze wykonanie kopii zapasowej (snapshotu) oraz sprawdzenie narzędziem <code>needs-restarting -r</code>, czy po aktualizacji wymagany jest restart."
  - question: "Jak sprawdzić historię aktualizacji DNF lub cofnąć zmianę?"
    answer: "Historię wszystkich operacji DNF sprawdzisz wpisując <code>sudo dnf history</code>. Możesz cofnąć wybraną transakcję za pomocą polecenia <code>sudo dnf history rollback ID</code>, gdzie ID to numer transakcji."
  - question: "Jak skonfigurować automatyczne codzienne aktualizacje bezpieczeństwa?"
    answer: "W tym celu należy zainstalować pakiet dnf-automatic komendą <code>sudo dnf install dnf-automatic -y</code>, a następnie ustawić <code>upgrade_type = security</code> w pliku konfiguracyjnym <code>/etc/dnf/automatic.conf</code>."
status: published
locale: pl
author:
  name: VoxiHost Team
  link: https://voxihost.pl/
contributors:
  - danielmarszalkowski
---

Podczas gdy wynajmujesz nowy serwer Linux VPS, rzadko kiedy myślisz o stabilności. Aktualizacje systemowe są kluczowe dla bezpieczeństwa, wydajności i stabilności.

Wszystkie trzy dystrybucje (AlmaLinux 9, CentOS Stream 9/10 i Rocky Linux 9/10) używają tego samego menedżera pakietów: **`dnf`**.

## Sprawdzenie dostępnych aktualizacji

W przeciwieństwie do `apt` z systemów Debian/Ubuntu, `dnf` łączy operacje "sprawdzania aktualizacje" i "aktualizowanie pakietów" w jedno polecenie.

Sprawdź dostępne aktualizacje:

```bash
sudo dnf check-update
```

To polecenie wyświetli listę dostępnych aktualizacji bez instalowania czegokolwiek.

## Instalacja wszystkich dostępnych aktualizacji

Zainstaluj wszystkie dostępne aktualizacje:

```bash
sudo dnf update -y
```

To polecenie pobierze i instaluje wszystkie oczekujące pakiety, a następnie stosuje je do systemu.

## Czyszczenie starych pakietów

Po pewnym czasie używaniu serwera, stare pakiety mogą się gromadzić. Biblioteki które nie są już potrzebne zajmują miejsce i mogą powodować konflikty.

`dnf autoremove` to bezpieczne narzędzie do usuwania starych pakietów:

```bash
sudo dnf autoremove -y
```

To polecenie wyszuka pakiety które są już oznaczone jako osierocone i usunie je wraz z ich zależnościami.

## Sprawdzanie czy wymagany jest restart

Niektóre aktualizacje jądra lub usługi wymagają restartu systemu. Zamiast zgadywać, możesz to sprawdzić:

```bash
sudo needs-restarting -r
```

Jeśli to polecenie wyświetla kod wyjścia `1` i informuje że restart jest wymagany, musisz zrestartować serwer.

## Włączanie automatycznych aktualizacji bezpieczeństwa

Dla środowisk produkcyjnych VPS, zaleca się włączenie automatycznych aktualizacji bezpieczeństwa.

{% image "/assets/images/blog/pl/jak-zaktualizowac-centos-rhel/H6.png", "Instalowanie dnf-automatic dla niezauważonych aktualizacji na AlmaLinux 9", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo dnf install dnf-automatic -y
```

Następnie edytuj konfigurację aby ustawić zachowanie które chcesz:

```bash
sudo nano /etc/dnf/automatic.conf
```

Jeśli brakuje pakietu `nano`, zainstaluj go najpierw:

{% image "/assets/images/blog/pl/jak-zaktualizowac-centos-rhel/H7.png", "Instalowanie edytora nano za pomocą sudo dnf install nano -y na CentOS Stream", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo dnf install nano -y
```

Dodaj te ustawienia:

```ini
[commands]
# What types of updates to apply automatically
upgrade_type = security
# Nie stosuj aktualizacji wydańcowych w środowisku produkcyjnych
apply_updates = yes
```

Zapisz plik i wyjdź.

Włącz i uruchom usługę:

```bash
sudo systemctl enable --now dnf-automatic.timer
```

Sprawdź czy działa:

```bash
sudo systemctl status dnf-automatic.timer
```

Powinieneś zobaczyć `"Active: active (waiting)"`.

## Szybka aktualizacja do nowej wersji głównej

Gdy nadejdzie czas na aktualizację do nowej wersji głównej (np., z AlmaLinux 9 do AlmaLinux 10), zwykły `dnf update` nie wystarczy. Potrzebujesz użyć specjalnej opcji `--releasever` aby poinstruować menedżerowi aby użył właściwych repozytoriów.

```bash
sudo dnf update --releasever=10
```

To polecenie zaktualizuje system do AlmaLinux 10 i przygotuje go do aktualizacji.

Po aktualizacji zrestartuj serwer:

```bash
sudo reboot
```

## Na co zwracać uwagę

Najczęstszym problemem na systemach rodziny RHEL jest **SELinux**. Jeśli aktualizacja zmienia uprawnienia plików lub ścieżki binarne, polityki SELinux mogą zablokować usługę przed poprawnym uruchomieniem po aktualizacji. Sprawdź log audytu jeśli coś przestanie działać po aktualizacji:

```bash
sudo ausearch -m avc -ts recent
```

Jeśli brakuje polecenia `ausearch`, zainstaluj je najpierw:

{% image "/assets/images/blog/pl/jak-zaktualizowac-centos-rhel/H8.png", "Instalowanie setroubleshoot-server aby diagnozować odmowy dostępu SELinux na AlmaLinux", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo dnf install setroubleshoot-server -y
```

Obsługa plików konfiguracyjnych w `dnf` jest nieco bardziej agresywna niż w `apt`. Gdy pakiet dostarcza nową domyślną konfigurację, `dnf` może nadpisać twoją dostosowaną wersję z sufiksem **`.rpmnew`** na oryginalnej. Zawsze sprawdzaj te po dużej aktualizacji:

```bash
sudo find /etc -name "*.rpmnew" -o -name "*.rpmsave"
```

## Podsumowanie

Aktualizacja serwera Linux to kluczowy element utrzymania bezpiecznego i stabilnego środowiska. Używając `dnf` z odpowiednimi flagami, możesz pewnie zarządzać tym procesem i mieć pewność że system jest zawsze chroniony najnowszymi poprawkami bezpieczeństwa.

Jeśli szukasz niezawodnego środowiska do testowania, nasze plany [Budget VPS](/pl/budget-vps/) to idealne, tanie serwery do eksperymentowania bez ryzyka.