---
image: /assets/images/blog/jak-zaktualizowac-fedore/og-image.png
title: 'Jak aktualizować Fedora 43 i nowsze: Kompletny przewodnik serwera'
description: Kompletny przewodnik krok po kroku do aktualizacji serwerów Fedora 43 i nowszych używając dnf5. Obejmuje dnf5 upgrade, autoremove, wykrywanie restartu i automatyczne aktualizacje dla środowisk produkcyjnych VPS.
date: '2026-03-25'
translationKey: update-fedora
category: Poradniki
tags:
  - fedora
  - dnf5
  - dnf upgrade
  - linux
  - vps
  - server administration
  - dnf-automatic
  - fedora 43
howto:
  name: Jak aktualizować serwer Fedora 43
  totalTime: PT5M
  yield: W pełni zaktualizowany serwer Fedora 43 z najnowszymi poprawkami bezpieczeństwa
  tool:
    - Serwer Linux VPS lub dedykowany z Fedora 43
    - Klient SSH (np. terminal, PuTTY)
    - Dostęp sudo lub root
  steps:
    - name: Sprawdź dostępne aktualizacje
      text: Uruchom sudo dnf5 check-upgrade aby zobaczyć jakie aktualizacje są dostępne bez instalowania czegokolwiek.
      url: the-basics-dnf5-upgrade
    - name: Zainstaluj wszystkie dostępne aktualizacje
      text: Uruchom sudo dnf5 upgrade -y aby pobrać i zainstalować wszystkie oczekujące aktualizacje pakietów.
      url: the-basics-dnf5-upgrade
    - name: Usuń osierocone pakiety
      text: Uruchom sudo dnf5 autoremove -y aby wyczyścić stare biblioteki i zależności które nie są już potrzebne.
      url: cleaning-up-dnf5-autoremove
    - name: Sprawdź czy wymagany jest restart
      text: Uruchom sudo needs-restarting -r aby zweryfikować czy aktualizacja jądra wymaga restartu systemu.
      url: do-you-need-a-reboot-needs-restarting
    - name: Włącz automatyczne aktualizacje bezpieczeństwa
      text: Zainstaluj i skonfiguruj dnf-automatic aby automatycznie stosować poprawki bezpieczeństwa.
      url: automating-patches-with-dnf-automatic
status: published
locale: pl
author:
  name: VoxiHost Team
  link: https://voxihost.pl/
contributors:
  - danielmarszalkowski
---

Fedora porusza się szybko. To dystrybucja która dostarcza to co RHEL będzie uruchamiać za dwa lata, co oznacza że otrzymujesz najnowocześniejsze jądra, nowocześniejsze narzędzia i pakiety które są aktualnie dostępne.

Zaczynając od **Fedory 41**, domyślny menedżer pakietów zmienił się na **dnf5**, pełne przepisanie `dnf` które jest szybsze, używa mniej pamięci i ma czystsze API. Jeśli używasz Fedory 43 lub nowszą, używasz `dnf5`, chociaż `dnf` wciąż działa jako alias wskazujący na ten sam plik binarny.

## Podstawy: dnf5 upgrade

Aby zaktualizować serwer Fedora 43, uruchom:

{% image "/assets/images/blog/jak-zaktualizowac-fedore/H1.png", "Uruchamianie sudo dnf5 upgrade na Fedora 43 - wynik terminala pokazujący aktualizowane pakiety", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo dnf5 upgrade -y
```

To polecenie sprawdza dostępne aktualizacje, pobiera je i instaluje w jednym przejściu. Flaga `-y` pomija potwierdzenia.

Jeśli chcesz zobaczyć co by się zmieniło przed zatwierdzeniem:

{% image "/assets/images/blog/jak-zaktualizowac-fedore/H2.png", "Uruchamianie dnf5 check-upgrade na Fedorze aby podglądnąć dostępne aktualizacje pakietów bez instalowania", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo dnf5 check-upgrade
```

To polecenie wyświetla listę dostępnych aktualizacji bez instalowania czegokolwiek.

**Dobry nawyk** przed uruchamianiem aktualizacji na maszynie która faktycznie obsługuje ruch.

Jedna różnica w stosunku do starszego `dnf`: `dnf5` wyraźniej oddziela koncepcje `update` i `upgrade`. W praktyce, `dnf5 upgrade` to polecenie którego chcesz, obsługuje zarówno aktualizacje pakietów jak i rozwiązywanie zależności. `dnf5 update` jest aliasem i działa tak samo.

## Czyszczenie (dnf5 autoremove)

Po aktualizacjach, pakiety osierocone się gromadzą. Stare jądra, stare biblioteki które nowsze wersje zastąpiły. Wyczyść je:

```bash
sudo dnf5 autoremove -y
```

Fedora domyślnie również zachowuje ostatnie dwie wersje jądra zainstalowane, co jest rozsądne, daje ci zapasowe rozwiązanie jeśli coś pójdzie nie tak. Jeśli chcesz konkretnie usuwać stare jądra:

```bash
sudo dnf5 repoquery --installonly --latest-limit=-2 -q
```

To pokazuje co zostałoby usunięte. Dodaj `--remove` aby faktycznie to zrobić. Bądź ostrożny na systemach gdzie nie masz dostępu do konsoli poza pasmem.

## Czy potrzebujesz restartu? (needs-restarting)

Niektóre aktualizacje jądra lub usług wymagają restartu systemu. Użyj `needs-restarting` aby sprawdzić:

{% image "/assets/images/blog/jak-zaktualizowac-fedore/H3.png", "Uruchamianie sudo needs-restarting -r na Fedorze aby sprawdzić czy restart jest wymagany po aktualizacji jądra", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo needs-restarting -r
```

Jeśli to polecenie zwraca kod wyjścia `1`, restart jest wymagany. Jeśli zwraca `0`, wszystko w porządku.

**Kod wyjścia 1** oznacza że restart jest potrzebny (zwykle aktualizacja jądra). **Kod wyjścia 0** oznacza że wszystko w porządku. Jeśli narzędzie nie jest zainstalowane:

{% image "/assets/images/blog/jak-zaktualizowac-fedore/H4.png", "Instalowanie dnf-utils na Fedorze za pomocą sudo dnf5 install dnf-utils aby uzyskać needs-restarting", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo dnf5 install dnf-utils -y
```

Na serwerze, możesz też sprawdzić które usługi działają na nieaktualnych bibliotekach i zrestartować tylko te, unikając pełnego restartu systemu:

```bash
sudo needs-restarting -s
```

To jest szczególnie przydatne na Fedorze, gdzie aktualizacje jądra są częste. Restartowanie kilku usług jest znacznie mniej disruptywne niż wyłączanie maszyny.

## Automatyzacja poprawek z dnf-automatic

Dla serwerów które działają w tle bez regularnej uwagi ręcznej:

{% image "/assets/images/blog/jak-zaktualizowac-fedore/H5.png", "Instalowanie pakietu dnf-automatic na Fedorze dla automatycznych niezauważalnych aktualizacji bezpieczeństwa", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo dnf5 install dnf-automatic -y
```

Jeśli brakuje pakietu `nano`, zainstaluj go najpierw:

```bash
sudo dnf5 install nano -y
```

Plik konfiguracyjny kontroluje co jest aktualizowane automatycznie:

{% image "/assets/images/blog/jak-zaktualizowac-fedore/H6.png", "Edycja /etc/dnf/automatic.conf na Fedorze aby skonfigurować niezauważalne aktualizacje za pomocą nano", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo nano /etc/dnf/automatic.conf
```

Kluczowe ustawienia:

```ini
[commands]
# security = tylko poprawki bezpieczeństwa (zalecane dla serwerów)
upgrade_type = security

# Faktycznie instaluj aktualizacje, nie tylko pobieraj je
apply_updates = yes

# Wyślij wynik do dziennika
emit_via = stdio
```

Włącz timer systemd aby uruchamiał go codziennie:

```bash
sudo systemctl enable --now dnf-automatic.timer
```

Zweryfikuj że jest zaplanowany:

```bash
sudo systemctl status dnf-automatic.timer
```

Na Fedorze, poprawki bezpieczeństwa pojawiają się regularnie biorąc pod uwagę jak aktywny jest projekt, więc automatyczne aktualizacje bezpieczeństwa robią realną różnicę.

## Szybka jednolinijkowa komenda

Zaloguj się przez SSH, zrób swoje rzeczy, zostaw to czyste:

```bash
sudo dnf5 upgrade -y && sudo dnf5 autoremove -y
```

Następnie sprawdź `needs-restarting -r`. Dwie minuty pracy, system pozostaje aktualny.

## Aktualizacja do następnej wersji Fedory

Fedora przechodzi do nowej wersji co sześć miesięcy, a każda wersja jest wspierana przez około 13 miesięcy. Gdy nadejdzie czas na przejście z 43 do 44:

{% image "/assets/images/blog/jak-zaktualizowac-fedore/H7.png", "Uruchamianie sudo dnf5 system-upgrade download --releasever=43 aby rozpocząć aktualizację wersji Fedory", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo dnf5 system-upgrade download --releasever=44
sudo dnf5 system-upgrade reboot
```

Aktualizacja odbywa się przy następnym uruchomieniu w kontrolowanym środowisku. To jest **obsługiwana ścieżka**, nie próbuj ręcznie zamieniać repozytoriów i uruchamiać `dnf5 distro-sync`, skończysz w złym stanie.

Przed aktualizacją: **zrób migawkę VM**, przeczytaj notki wydania Fedory 44 pod kątem wszystkiego co może zepsuć twoje obciążenie, i jeśli to możliwe przetestuj na klonie najpierw.

## Pułapki SELinux i plików konfiguracyjnych

Dwie rzeczy na które należy zwrócić uwagę po aktualizacjach na Fedorze:

**Odmowy SELinux.** Jeśli usługa przestaje działać po aktualizacji, sprawdź log audytu:

```bash
sudo ausearch -m avc -ts recent
```

Jeśli brakuje polecenia `ausearch`, zainstaluj je najpierw:

```bash
sudo dnf5 install ausearch -y
```

Aktualizacja polityki mogła zmienić co usługa może robić, lub zaktualizowany plik binarny może potrzebować nowej etykiety. Większość czasu to samo się rozwiązuje, ale to pierwsza rzecz do sprawdzenia.

**Konflikty plików konfiguracyjnych RPM.** Gdy pakiet dostarcza nową domyślną konfigurację, tworzy plik **`.rpmnew`** zamiast nadpisywać twoją:

```bash
sudo find /etc -name "*.rpmnew" -o -name "*.rpmsave" 2>/dev/null
```

Sprawdź te po każdej znaczącej aktualizacji. Scal co ważne, usuń co niepotrzebne.

Fedora jest solidnym systemem operacyjnym serwera jeśli utrzymujesz go aktualnym. Sześciomiesięczny cykl wydawania brzmi szybko, ale ścieżka aktualizacji system-upgrade jest gładka i rzadko powoduje niespodzianki. Jeśli chcesz czystego VPS do przetestowania aktualizacji Fedory bez ryzykowania swojej produkcyjnej maszyny, nasze plany [Budget VPS](/pl/budget-vps/) to tani sposób na przejście przez cały proces.

## Podsumowanie

Regularne aktualizacje serwera Fedora to kluczowy element utrzymania bezpiecznego i stabilnego środowiska. Dzięki `dnf5` możesz pewnie zarządzać tym procesem i mieć pewność że system jest zawsze chroniony najnowszymi poprawkami bezpieczeństwa.