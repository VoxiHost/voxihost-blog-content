---
image: /assets/images/blog/pl/monitorowanie-vps-htop-df-free/og-image.png
title: 'Jak monitorować swój VPS: Przewodnik początkującego do htop, df i free'
description: Kompletny przewodnik początkującego do monitorowania zdrowia swojego serwera Linux. Naucz się sprawdzać użycie CPU, RAM i przestrzeni dyskowej używając htop, free i df.
date: '2026-03-25'
translationKey: monitor-vps-htop-df-free
category: Poradniki
tags:
  - monitoring
  - linux
  - vps
  - server administration
  - htop
  - df
  - free
  - cpu
  - ram
howto:
  name: Jak monitorować zasoby VPS Linux używając narzędzi CLI
  totalTime: PT5M
  yield: Jasne zrozumienie obecnego użycia zasobów serwera (CPU, RAM, Dysk)
  tool:
    - VPS lub dedykowany serwer działający na dowolnej dystrybucji Linux
    - Klient SSH (np. terminal, PuTTY)
  steps:
    - name: Zainstaluj i uruchom htop
      text: Uruchom sudo apt install htop (lub dnf install htop) i wpisz htop aby zobaczyć na żywo użycie CPU i pamięci.
      url: krok-1-monitoruj-cpu-i-procesy-za-pomoca-htop
    - name: Sprawdź systemową RAM
      text: Uruchom free -h aby zobaczyć dokładnie ile RAM jest użyte, wolne i buforowane.
      url: krok-2-zrozum-uzycie-pamieci-za-pomoca-free
    - name: Sprawdź całkowitą przestrzeń dyskową
      text: Uruchom df -h aby zobaczyć ile miejsca przechowywania jest dostępne na dyskach twojego serwera.
      url: krok-3-sprawdz-przestrzen-dyskowa-za-pomoca-df
    - name: Znajdź duże foldery
      text: Uruchom du -sh * aby zmierzyć rozmiar konkretnych katalogów aby znaleźć co zajmuje miejsce.
      url: krok-4-znajdz-co-zjada-miejsce-za-pomoca-du
status: published
locale: pl
author:
  name: VoxiHost Team
  link: https://voxihost.pl/
contributors:
  - danielmarszalkowski
---

Gdy twoja strona internetowa zwalnia lub twoja baza danych nagle awariuje, pierwsze pytanie które musisz zadać to: *"Czy mój serwer ma zasoby?"*

Nie potrzebujesz skomplikowanego oprogramowania do tworzenia wykresów aby odpowiedzieć na to. Linux dostarcza garść niezwykle potężnych, wbudowanych narzędzi wiersza poleceń które mogą natychmiast powiedzieć ci dokładnie co dzieje się pod maską twojego serwera Linux.

Oto absolutny przewodnik początkującego do diagnozowania CPU, RAM i przechowywania dyskowego twojego serwera w sekundach.

## Krok 1: Monitoruj CPU i procesy za pomocą `htop`

Stary standard do sprawdzania uruchomionych programów w Linux to polecenie `top`. Jednakże, `top` jest brzydki i trudny do odczytania. Zamiast tego, powinieneś użyć **`htop`**. Jest kolorowy, interaktywny i znacznie łatwiejszy do zrozumienia dla początkujących.

Jeśli nie jest zainstalowany w twoim systemie, zainstaluj go:
```bash
# Na Ubuntu/Debian:
sudo apt install htop -y

# Na AlmaLinux/CentOS/Fedora:
sudo dnf install htop -y
```

Uruchom go po prostu wpisując:

{% image "/assets/images/blog/pl/monitorowanie-vps-htop-df-free/H1.png", "Kolorowy interfejs terminala htop pokazujący na żywo użycie CPU i RAM na VPS Linux", "(max-width: 768px) 100vw, 800px" %}

```bash
htop
```

### Jak czytać htop:
Spójrz na samą górę ekranu.
- **Numerowane paski (1, 2, 3...)**: Te reprezentują rdzenie CPU. Jeśli paski są całkowicie wypełnione (100%) i mocno kolorowane na czerwono, twój serwer walczy aby przetworzyć wszystko co jest na niego rzucane.
- **Mem (Pamięć)**: To pokazuje użycie RAM. Zielona sekcja to aktywnie używana RAM. Żółte/niebieskie sekcje to pamięć podręczna (co jest dobre, Linux używa wolnej RAM aby przyspieszyć dostęp do plików).
- **Lista procesów**: Pod paskami jest ciągle aktualizująca się lista programów. Kolumny **%CPU** i **%MEM** pokazują dokładnie który program zjada twoje zasoby.

*Aby wyjść z `htop`, po prostu naciśnij `q` na klawiaturze, lub `F10`.*

## Krok 2: Zrozum użycie pamięci za pomocą `free`

Jeśli chcesz tylko szybki, czysty obraz swojej RAM, nie potrzebujesz `htop`. Możesz użyć polecenia `free`.

Zawsze uruchamiaj je z flagą `-h`. To oznacza **"ludzko czytelne"**, które tłumaczy wynik na łatwo czytelne Megabajty (M) lub Gigabajty (G) zamiast milionów mylących bajtów.

{% image "/assets/images/blog/pl/monitorowanie-vps-htop-df-free/H2.png", "Uruchamianie free -h aby sprawdzić dostępną RAM i pamięć podręczną na serwerze Linux", "(max-width: 768px) 100vw, 800px" %}

```bash
free -h
```

Przykładowy wynik:
```text
               total        used        free      shared  buff/cache   available
Mem:           2.0G        412M        1.2G         15M        400M        1.4G
Swap:            0B          0B          0B
```

### Najczęstsza panika początkującego:
Często, ludzie patrzą na `free` i panikują ponieważ kolumna "free" jest bardzo niska. **Nie panikuj.**
Spójrz na kolumnę **`available`** zamiast tego.

Linux celowo wypełnia pustą RAM plikami podręcznymi (`buff/cache`) aby twój serwer działał szybciej. Jeśli program nagle potrzebuje RAM, Linux natychmiast porzuca pamięć podręczną i oddaje RAM. Kolumna `available` mówi ci ile RAM jest *rzeczywiście* dostępne dla nowych aplikacji.

## Krok 3: Sprawdź przestrzeń dyskową za pomocą `df`

Brak miejsca na dysku całkowicie zepsuje twój serwer. Bazy danych ulegają uszkodzeniu, logi nie mogą być zapisywane, a usługi odmawiają uruchamiania.

Aby sprawdzić ile miejsca zostało, użyj `df` (Disk Free). Ponownie, użyj flagi `-h` dla rozmiarów ludzko czytelnych:

{% image "/assets/images/blog/pl/monitorowanie-vps-htop-df-free/H3.png", "Uruchamianie df -h aby monitorować całkowitą przestrzeń dyskową i użycie przechowywania na VPS Linux", "(max-width: 768px) 100vw, 800px" %}

```bash
df -h
```

Przykładowy wynik:
```text
Filesystem      Size  Used Avail Use% Mounted on
tmpfs           198M  1.1M  197M   1% /run
/dev/vda1        25G  8.5G   16G  36% /
tmpfs           988M     0  988M   0% /dev/shm
```

Spójrz na linię gdzie **Mounted on** to `/` (główny katalog). W tym przykładzie, całkowity rozmiar dysku to **25G**, użyliśmy **8.5G**, a dysk jest w **36%** pełny. Jesteś całkowicie bezpieczny.

## Krok 4: Znajdź co zjada miejsce za pomocą `du`

Jeśli `df` mówi ci że dysk jest w 99% pełny, twoim następnym pytaniem jest: *"Co zajmuje całe miejsce?"*

Użyj `du` (Disk Usage) aby znaleźć winowajcę. Przejdź do katalogu który podejrzewasz (zwykle `/var/log` lub folder strony internetowej) i uruchom:

{% image "/assets/images/blog/pl/monitorowanie-vps-htop-df-free/H4.png", "Uruchamianie du -sh /* aby znaleźć duże katalogi aplikacji zajmujące miejsce na dysku", "(max-width: 768px) 100vw, 800px" %}

```bash
du -sh /*
```

To wyświetli rozmiar (`-s` dla podsumowania) w formacie ludzko czytelnym (`-h`) każdego folderu i pliku w obecnej lokalizacji. Jeśli zobaczysz maszynowy 15GB plik `error.log`, znalazłeś swój problem!

Możesz też połączyć to z sortowaniem aby łatwo znaleźć największe foldery. Na przykład, aby znaleźć największe foldery w `/var/log` na całym serwerze:

{% image "/assets/images/blog/pl/monitorowanie-vps-htop-df-free/H5.png", "Uruchamianie sudo du -h /var/log/* | sort -h aby znaleźć największe indywidualne pliki logów na Linux", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo du -h /var/log/* | sort -h
```

## Potrzebujesz więcej mocy?

Te polecenia są ratownikami życia do szybkiego debugowania wolnego serwera przez SSH. Jeśli zamiast narzędzi terminalowych chcesz pięknego, historycznego panelu WWW, sprawdź nasz przewodnik [Jak skonfigurować Netdata](/pl/blog/konfiguracja-netdata-vps/). Jeśli odkryjesz że twoje CPU ciągle uderza w 100% lub twoja pamięć `available` jest konsekwentnie na 0, twój projekt oficjalnie urósł poza obecny sprzęt.

Gdy nadejdzie czas na skalowanie, sprawdź nasze wysoko skalowalne opcje [Premium VPS](/pl/premium-vps/) lub niezawodne [Budget VPS](/pl/budget-vps/) aby natychmiast podwoić swoje zasoby i utrzymać swoje aplikacje działające płynnie.