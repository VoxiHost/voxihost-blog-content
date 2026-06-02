---
image: /assets/images/blog/pl/jak-zaktualizowac-ubuntu-debian/og-image.png
title: 'Jak aktualizować Ubuntu i Debian: Kompletny przewodnik serwera'
description: Kompletny przewodnik krok po kroku do aktualizacji serwerów Ubuntu i Debian Linux. Obejmuje apt update, apt upgrade, aktualizacje jądra, automatyczne aktualizacje i najlepsze praktyki dla środowisk produkcyjnych VPS.
date: '2026-03-24'
translationKey: update-ubuntu-debian
category: Poradniki
tags:
  - ubuntu
  - debian
  - apt update
  - apt upgrade
  - linux
  - vps
  - server administration
  - unattended-upgrades
  - kernel update
howto:
  name: Jak aktualizować serwer Ubuntu i Debian
  totalTime: PT5M
  yield: W pełni zaktualizowany serwer Ubuntu lub Debian z najnowszymi poprawkami bezpieczeństwa
  tool:
    - Serwer Linux VPS lub dedykowany
    - Klient SSH (np. terminal, PuTTY)
    - Dostęp sudo lub root
  steps:
    - name: Odśwież indeks pakietów
      text: Uruchom sudo apt update aby zsynchronizować lokalny indeks pakietów z repozytoriami.
      url: odswiezenie-indeksu-pakietow
    - name: Zainstaluj dostępne aktualizacje
      text: Uruchom sudo apt upgrade -y aby zainstalować wszystkie oczekujące aktualizacje pakietów.
      url: instalacja-dostepnych-aktualizacji
    - name: Obsłuż zatrzymane pakiety
      text: Jeśli apt upgrade zgłasza zatrzymane pakiety, użyj sudo apt full-upgrade -y aby rozwiązać zmiany zależności.
      url: obsluga-zatrzymanych-pakietow
    - name: Usuń stare pakiety
      text: Uruchom sudo apt autoremove -y aby wyczyścić stare biblioteki i zależności które nie są już potrzebne.
      url: czyszczenie-starych-pakietow
    - name: Sprawdź czy wymagany jest restart
      text: Uruchom cat /var/run/reboot-required aby sprawdzić czy aktualizacja jądra wymaga restartu systemu.
      url: sprawdzanie-czy-wymagany-jest-restart
    - name: Włącz automatyczne aktualizacje bezpieczeństwa
      text: Zainstaluj i skonfiguruj unattended-upgrades aby automatycznie stosować poprawki bezpieczeństwa.
      url: automatyczne-aktualizacje-bezpieczenstwa
faq:
  - question: "Jaka jest różnica między apt update a apt upgrade?"
    answer: "Komenda <code>apt update</code> pobiera aktualną listę pakietów i metadanych z repozytoriów (nie instalując niczego), natomiast <code>apt upgrade</code> pobiera i instaluje faktyczne wersje pakietów."
  - question: "Jak naprawić wstrzymane pakiety (packages kept back) w Ubuntu?"
    answer: "Pakiety są wstrzymywane, gdy ich aktualizacja wymaga zainstalowania nowych zależności lub usunięcia starych. Możesz bezpiecznie wymusić ich aktualizację za pomocą polecenia <code>sudo apt full-upgrade</code>."
  - question: "Czy aktualizowanie serwera produkcyjnego poprzez apt upgrade jest bezpieczne?"
    answer: "Tak, standardowe aktualizacje pakietów są bezpieczne. Dobrą praktyką jest jednak wykonanie snapshotu serwera przed aktualizacją, wykonywanie jej przy niskim natężeniu ruchu i sprawdzenie pliku <code>/var/run/reboot-required</code>."
  - question: "Do czego służy polecenie apt autoremove?"
    answer: "Komenda <code>apt autoremove</code> usuwa pakiety (najczęściej starsze jądra systemowe i biblioteki), które zostały zainstalowane automatycznie jako zależności, ale nie są już wymagane przez żadne zainstalowane oprogramowanie."
  - question: "Jak skonfigurować automatyczne aktualizacje bezpieczeństwa?"
    answer: "Aby włączyć automatyczną instalację poprawek bezpieczeństwa w tle, zainstaluj pakiet <code>unattended-upgrades</code> i dostosuj jego konfigurację w pliku <code>/etc/apt/apt.conf.d/50unattended-upgrades</code>."
status: published
locale: pl
author:
  name: VoxiHost Team
  link: https://voxihost.pl/
contributors:
  - danielmarszalkowski
---

Jeśli wdrożyłeś świeży serwer VPS i nie jesteś pewien co robić najpierw, zaktualizuj go. Brzmi oczywiste, ale zaskakująca liczba serwerów działających na publicznym internecie uruchamia pakiety które nie były dotykane od instalacji systemu. To problem.

Ubuntu i Debian dostarczają solidne domyślne ustawienia, ale "solidne" nie znaczy "bezpieczne na zawsze". Pakiety dostają CVE co tydzień. Jądro jest łatkowane. OpenSSH, OpenSSL, curl, wszystkie z nich miały poważne luki w zabezpieczeniach przez lata które były już naprawione w aktualizacji której większość ludzi nie chciała się zastosować. Więc naprawmy to.

Zanim zaczniemy: jeśli wdrażasz świeży serwer z premium dostawcą jak **<span class="text-white">Voxi</span><span class="text-amber-300">Host</span>**, system automatycznie uruchamia pełną aktualizację pakietów natychmiast po wdrożeniu przy pierwszym uruchomieniu. Ale gdy twój serwer działa przez jakiś czas, nadal będziesz musiał wiedzieć jak sam go utrzymywać.

## Odświeżenie indeksu pakietów

Zanim zaktualizujesz cokolwiek, odśwież lokalny indeks pakietów. To nie instaluje nic, tylko sprawdza jakie aktualizacje są faktycznie dostępne:

{% image "/assets/images/blog/pl/jak-zaktualizowac-ubuntu-debian/H1.png", "Uruchamianie sudo apt update na Ubuntu lub Debian w celu odświeżenia indeksu pakietów i sprawdzenia dostępnych aktualizacji", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo apt update
```

## Instalacja dostępnych aktualizacji

Następnie zainstaluj je:

{% image "/assets/images/blog/pl/jak-zaktualizowac-ubuntu-debian/H2.png", "Uruchamianie sudo apt upgrade -y na Ubuntu lub Debian w celu zainstalowania wszystkich dostępnych aktualizacji pakietów z zaktualizowanego indeksu", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo apt upgrade -y
```

To wszystko dla rutynowej konserwacji. Uruchom te dwa, skończone. Flaga `-y` pomija potwierdzenie, co jest przydatne gdy uruchamiasz to w skrypcie lub po prostu nie chcesz pilnować terminala.

Jedna rzecz warta poznania: `apt upgrade` nie usunie pakietów ani nie pobierze nowych zależności. Tylko aktualizuje to co jest już zainstalowane. To celowe. Umożliwia bezpieczne uruchomienie na aktywnym serwerze bez martwienia się że coś się zepsuje ponieważ pakiet został zamieniony.

## Obsługa zatrzymanych pakietów

Czasami `apt upgrade` zgłasza że niektóre pakiety zostały zatrzymane ("kept back"). Zwykle dzieje się tak gdy pakiety mają zaktualizowane zależności które jeszcze nie są zainstalowane.

Aby rozwiązać ten problem:

```bash
sudo apt full-upgrade -y
```

To polecenie może instalować nowe pakiety lub usuwać te w konflikcie, aby rozwiązać zależności.

## Czyszczenie starych pakietów

Po aktualizacjach, stare pakiety mogą zostać na systemie jako osierocone. `apt autoremove` usuwa je bezpiecznie:

{% image "/assets/images/blog/pl/jak-zaktualizowac-ubuntu-debian/H3.png", "Uruchamianie sudo apt autoremove -y na Ubuntu lub Debian w celu usunięcia starych pakietów zależności które nie są już potrzebne po aktualizacjach", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo apt autoremove -y
```

To polecenie usuwa pakiety które nie są już potrzebne przez żaden inny zainstalowany pakiet.

## Sprawdzanie czy wymagany jest restart

Aktualizacje jądra wymagają restartu systemu. Ubuntu i Debian tworzą plik wskaźnika gdy restart jest wymagany:

{% image "/assets/images/blog/pl/jak-zaktualizowac-ubuntu-debian/H4.png", "Sprawdzanie pliku /var/run/reboot-required na Ubuntu w celu określenia czy restart systemu jest wymagany po aktualizacji jądra", "(max-width: 768px) 100vw, 800px" %}

```bash
cat /var/run/reboot-required
```

Jeśli ten plik istnieje, musisz zrestartować serwer.

## Automatyczne aktualizacje bezpieczeństwa

Dla środowisk produkcyjnych VPS, gdzie nie logujesz się codziennie, zaleca się włączenie automatycznych aktualizacji bezpieczeństwa.

Zainstaluj pakiet `unattended-upgrades`:

```bash
sudo apt install unattended-upgrades -y
sudo dpkg-reconfigure --priority=low unattended-upgrades
```

Skonfiguruj automatyczne aktualizacje:

```bash
sudo nano /etc/apt/apt.conf.d/50unattended-upgrades
```

Dodaj te ustawienia:

```ini
Unattended-Upgrade::Automatic-Reboot "false"
Unattended-Upgrade::Remove-Unused-Dependencies "true"
Unattended-Upgrade::Automatic-Reboot-WithUsers "true"
```

Połącz je, pozwól im działać, sprawdź czy restart jest potrzebny, gotowe.

## Aktualizacja do nowej wersji głównej (do-release-upgrade)

Jeśli chcesz przejść z Ubuntu 22.04 na 24.04:

{% image "/assets/images/blog/pl/jak-zaktualizowac-ubuntu-debian/H5.png", "Uruchamianie sudo do-release-upgrade na Ubuntu w celu aktualizacji z jednej głównej wersji do następnej, na przykład 22.04 do 24.04", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo do-release-upgrade
```

Zrób migawkę przed aktualizacją na środowisku produkcyjnym.

## Na co naprawdę zwracać uwagę

Aktualizacje jądra i libc to te które zawsze wymagają restartu. Zmiany plików konfiguracyjnych to te które mogą cię zaskoczyć. Podczas aktualizacji, jeśli pakiet dostarcza nową domyślną konfigurację dla czegoś co dostosowałeś, apt zapyta czy zachować twoją wersję czy zainstalować nową. Czytaj te monitory uważnie.

Regularnie aktualizowane serwery również zawodzą w bardziej przewidywalny sposób. Jeśli coś się zepsuje po aktualizacji, wiesz dokładnie kiedy to się stało i jakie pakiety się zmieniły. Na serwerze który nie był aktualizowany przez 8 miesięcy, debugowanie awarii to znacznie bardziej brudne doświadczenie.

Dla całkowicie utwardzonego serwera, aktualizacje pakietów to tylko krok pierwszy. Rozważ skonfigurowanie [Zapory UFW](/pl/blog/konfiguracja-ufw-ubuntu-debian/) i [fail2ban](/pl/blog/konfiguracja-fail2ban-ubuntu-debian/) aby aktywnie odrzucać złośliwe tło.

Jeśli chcesz czystego VPS do ćwiczenia tego, nasze plany [Budget VPS](/pl/budget-vps/) są wystarczająco tanie że możesz zrobić migawkę, eksperymentować i zniszczyć to bez żadnego stresu.