---
image: /assets/images/blog/pl/jak-zainstalować-nodejs-na-ubuntu-i-debian/og-image.png
title: "Jak zainstalować Node.js na Ubuntu i Debian"
description: "Dowiedz się, jak zainstalować stabilną wersję Node.js na systemach Ubuntu i Debian za pomocą oficjalnego repozytorium NodeSource dla optymalnej wydajności."
status: draft
category: Poradniki
tags:
  - nodejs
  - ubuntu
  - debian
  - linux
  - server
date: '2026-07-27'
locale: pl
translationKey: install-nodejs-ubuntu-debian
author:
  name: VoxiHost Team
  link: https://voxihost.pl/
contributors: []
howto:
  name: "Jak zainstalować Node.js na Ubuntu i Debian"
  steps:
    - name: "Krok 1: Aktualizacja pakietów systemowych i instalacja zależności"
      url: "krok-1-aktualizacja-pakietow-systemowych-i-instalacja-zaleznosci"
    - name: "Krok 2: Dodanie repozytorium NodeSource"
      url: "krok-2-dodanie-repozytorium-nodesource"
    - name: "Krok 3: Instalacja Node.js oraz NPM"
      url: "krok-3-instalacja-nodejs-oraz-npm"
    - name: "Krok 4: Weryfikacja instalacji"
      url: "krok-4-weryfikacja-instalacji"
faq:
  - question: "Dlaczego powinienem unikać domyślnego repozytorium Ubuntu dla Node.js?"
    answer: "Domyślne repozytoria dystrybucji często zawierają <strong>przestarzałe wersje</strong> Node.js. Korzystanie z oficjalnego repozytorium NodeSource zapewnia dostęp do najnowszych poprawek bezpieczeństwa i funkcji."
  - question: "Jaka jest różnica między wersją LTS a Current środowiska Node.js?"
    answer: "Wersja <strong>LTS (Long Term Support)</strong> jest przeznaczona do stabilnych środowisk produkcyjnych, podczas gdy wersja <strong>Current</strong> oferuje najnowsze funkcje, ale posiada krótszy okres wsparcia."
  - question: "Jak mogę zaktualizować Node.js do nowszej wersji w przyszłości?"
    answer: "Aktualizację można przeprowadzić uruchamiając skrypt instalacyjny NodeSource dla wybranej wersji, a następnie wykonując <code>sudo apt update && sudo apt install -y nodejs</code>, co nadpisze istniejącą instalację."
  - question: "Czy muszę instalować npm oddzielnie po zainstalowaniu Node.js?"
    answer: "Nie, pakiet Node.js dostarczany przez NodeSource zawiera <strong>npm</strong> domyślnie, więc jest on instalowany automatycznie podczas całego procesu."
  - question: "Jak odinstalować Node.js z mojego serwera?"
    answer: "Możesz usunąć Node.js wykonując polecenie <code>sudo apt purge nodejs</code>, a następnie <code>sudo apt autoremove</code>, aby wyczyścić nieużywane zależności."
---

## Wprowadzenie

Uruchamianie języka JavaScript po stronie serwera wymaga niezawodnego i aktualnego środowiska uruchomieniowego. Chociaż wiele dystrybucji systemu Linux zawiera Node.js w swoich domyślnych menedżerach pakietów, wersje te są często przestarzałe, co może prowadzić do problemów z kompatybilnością nowoczesnych frameworków lub luk w zabezpieczeniach. Dla programistów wdrażających aplikacje na serwerach <span class="text-white">Voxi</span><span class="text-amber-300">Host</span> [Premium VPS](/pl/vps/premium/), utrzymanie aktualnego środowiska jest kluczowe dla wydajności i stabilności.

Najskuteczniejszym sposobem zarządzania środowiskiem uruchomieniowym jest korzystanie z oficjalnych repozytoriów binarnych NodeSource. Takie podejście gwarantuje, że nie ogranicza Cię starsze oprogramowanie dostępne w standardowych repozytoriach systemowych. Dzięki konfiguracji oficjalnego repozytorium zyskujesz płynny dostęp do najnowszych wydań o długoterminowym wsparciu (LTS), co zapewnia stabilność serwera produkcyjnego przy jednoczesnym wsparciu dla nowoczesnych funkcji standardu ECMAScript.

Ten poradnik koncentruje się na czystej, profesjonalnej konfiguracji w systemach Ubuntu oraz Debian. Wykraczamy poza domyślne listy pakietów, aby zainstalować środowisko Node.js gotowe do pracy produkcyjnej. Niezależnie od tego, czy hostujesz lekkie API, czy złożoną aplikację czasu rzeczywistego, przedstawione kroki zapewnią poprawną konfigurację serwera pod kątem długoterminowej niezawodności. Zakładamy, że dysponujesz świeżą instancją z co najmniej 1 GB pamięci RAM, aby proces budowania zakończył się bez problemów z wydajnością pamięci.

{% image "/assets/images/blog/pl/jak-zainstalowac-nodejs-na-ubuntu-i-debian/H1.png", "Dane wyjściowe terminala pokazujące weryfikację poprawnej instalacji Node.js na serwerze Ubuntu", "(max-width: 768px) 100vw, 800px" %}

## Wymagania wstępne

Przed rozpoczęciem instalacji upewnij się, że Twój serwer spełnia podstawowe wymagania, co pozwoli na bezproblemowe wdrożenie. Zalecamy posiadanie co najmniej 1 GB pamięci RAM. Jeśli korzystasz z <span class="text-white">Voxi</span><span class="text-amber-300">Host</span> [Budget VPS](/pl/vps/budget/), zweryfikuj, czy Twoja instancja posiada wystarczającą ilość pamięci, ponieważ proces budowania niektórych modułów Node.js może być wymagający pod kątem zasobów.

Musisz posiadać dostęp do konta root lub użytkownika z uprawnieniami sudo. Ponieważ będziemy konfigurować zewnętrzne repozytoria, system musi być aktualny, aby uniknąć konfliktów z istniejącymi wersjami bibliotek. Jeśli nie przeprowadzałeś ostatnio aktualizacji systemu, rozważ wykonanie polecenia `sudo apt update && sudo apt upgrade -y` przed przejściem dalej.

Dodatkowo, ten przewodnik zakłada, że pracujesz na czystej instalacji Ubuntu 20.04 lub nowszego, bądź Debian 10 lub nowszego. Jeśli zarządzasz firewallem, upewnij się, że został on już skonfigurowany w taki sposób, aby zezwalał na ruch na portach, których docelowo będzie używać Twoja aplikacja.

Na koniec upewnij się, że masz dostęp do terminala lub klienta SSH. Będziemy używać narzędzia `curl` do pobrania skryptów instalacyjnych, więc zweryfikuj, czy Twoje środowisko jest gotowe na obsługę tych zapytań sieciowych. Jeśli planujesz zarządzać wieloma instancjami serwerów lub potrzebujesz rozszerzonej ochrony dla swoich aplikacji, możesz również rozważyć nasze rozwiązanie [Shield](/pl/shield/), aby zabezpieczyć swój ruch na brzegu sieci.

{% image "/assets/images/blog/pl/jak-zainstalowac-nodejs-na-ubuntu-i-debian/H2.png", "Ekran terminala pokazujący świeżo zaktualizowaną instancję Ubuntu gotową do instalacji oprogramowania", "(max-width: 768px) 100vw, 800px" %}

## Krok 1: Aktualizacja pakietów systemowych i instalacja zależności

Zanim pobierzemy pliki binarne Node.js, musimy przygotować środowisko. Standardowe repozytoria dystrybucji często zawierają przestarzałe wersje Node.js, co może powodować problemy z kompatybilnością nowoczesnych frameworków. Aby tego uniknąć, wykorzystamy oficjalne repozytorium NodeSource.

Skrypt instalacyjny wymaga narzędzia `curl` do pobrania niezbędnych kluczy GPG oraz plików konfiguracyjnych repozytorium. Jeśli Twój serwer jest instalacją minimalną, to narzędzie może nie być zainstalowane. Uruchom poniższe polecenie, aby odświeżyć listę pakietów i zainstalować wymaganą zależność:

```bash
## Aktualizacja lokalnego indeksu pakietów i instalacja curl
sudo apt update && sudo apt install -y curl
```

Po zakończeniu instalacji system będzie gotowy do komunikacji z serwerami NodeSource. Utrzymywanie aktualnej listy pakietów gwarantuje, że pobierasz najnowsze poprawki bezpieczeństwa dla bibliotek systemu operacyjnego wraz z nowym środowiskiem Node.js. Zweryfikowaliśmy, że takie podejście minimalizuje konflikty zależności zarówno w systemach Ubuntu, jak i Debian.

{% image "/assets/images/blog/pl/jak-zainstalowac-nodejs-na-ubuntu-i-debian/H3.png", "Wyjście terminala pokazujące pomyślną instalację pakietu curl po aktualizacji apt", "(max-width: 768px) 100vw, 800px" %}

## Krok 2: Dodanie repozytorium NodeSource

Skoro system posiada już niezbędne narzędzia do obsługi zapytań zdalnych, skonfigurujemy repozytorium NodeSource. Repozytorium to działa jako pomost, umożliwiając menedżerowi pakietów śledzenie i instalowanie oficjalnych wydań o przedłużonym wsparciu (LTS) bezpośrednio ze źródła.

Użycie zautomatyzowanego skryptu instalacyjnego jest najbardziej niezawodnym sposobem na obsługę importu kluczy GPG oraz generowanie plików repozytorium. Dzięki zastosowaniu flagi `-E` wraz z `sudo` zapewniamy, że bieżące zmienne środowiskowe zostaną zachowane podczas wykonywania skryptu, co ma kluczowe znaczenie dla poprawnego wykrycia architektury systemu oraz wersji dystrybucji.

Wykonaj poniższe polecenia, aby pobrać skrypt instalacyjny i zastosować konfigurację na serwerze:

```bash
## Pobranie skryptu instalacyjnego NodeSource LTS
curl -fsSL https://deb.nodesource.com/setup_lts.x -o nodesource_setup.sh

## Wykonanie skryptu w celu konfiguracji repozytorium
sudo -E bash nodesource_setup.sh
```

Skrypt automatycznie zaktualizuje lokalne listy pakietów po dodaniu repozytorium. W terminalu pojawi się seria logów wskazujących, że źródła pakietów zostały odświeżone, a klucze GPG zaimportowane pomyślnie. Jeśli pojawią się ostrzeżenia dotyczące brakujących kluczy, należy zweryfikować stabilność połączenia internetowego oraz upewnić się, że zapora sieciowa nie blokuje ruchu wychodzącego do `deb.nodesource.com`. Po zakończeniu tego procesu serwer będzie gotowy do pobrania najnowszych stabilnych plików binarnych Node.js.

{% image "/assets/images/blog/pl/jak-zainstalowac-nodejs-na-ubuntu-i-debian/H4.png", "Wyjście terminala pokazujące skrypt konfigurujący repozytorium NodeSource i aktualizujący listę pakietów", "(max-width: 768px) 100vw, 800px" %}

## Krok 3: Instalacja Node.js oraz NPM

Gdy repozytorium NodeSource jest już aktywne w systemie, końcowy etap instalacji jest prosty. Ponieważ poprzedni krok wywołał automatyczną aktualizację list pakietów, możesz przejść bezpośrednio do instalacji środowiska uruchomieniowego Node.js oraz menedżera pakietów NPM.

Korzystanie z oficjalnego repozytorium gwarantuje otrzymanie nowoczesnej, wspieranej wersji Node.js, która jest znacznie bardziej stabilna i bogata w funkcje niż starsze wersje często spotykane w domyślnych archiwach dystrybucji. Wykonaj poniższe polecenie, aby zainstalować pakiet:

```bash
## Instalacja Node.js oraz NPM z repozytorium NodeSource
sudo apt install -y nodejs
```

Pakiet `nodejs` zawiera zarówno plik binarny do uruchamiania aplikacji, jak i narzędzie `npm` służące do zarządzania zależnościami projektu. Proces instalacji przebiega zazwyczaj szybko, ponieważ wymaga jedynie pobrania plików binarnych z repozytorium, które właśnie skonfigurowano.

Po zakończeniu działania polecenia podstawowe środowisko jest już gotowe. Możesz teraz zweryfikować, czy wszystko zostało poprawnie powiązane i sprawdzić konkretną wersję środowiska uruchomieniowego działającą na Twoim serwerze <span class="text-white">Voxi</span><span class="text-amber-300">Host</span>.

{% image "/assets/images/blog/pl/jak-zainstalowac-nodejs-na-ubuntu-i-debian/H5.png", "Wyjście terminala pokazujące pomyślną instalację pakietu nodejs za pomocą apt", "(max-width: 768px) 100vw, 800px" %}

## Krok 4: Weryfikacja instalacji

Po zainstalowaniu plików binarnych należy potwierdzić, że system poprawnie wskazuje na wersje pochodzące z NodeSource. Jest to kluczowy krok, aby upewnić się, że Twój serwer <span class="text-white">Voxi</span><span class="text-amber-300">Host</span> nie korzysta przypadkowo ze starszej, zbuforowanej lub domyślnej systemowej wersji środowiska uruchomieniowego.

Wykonaj poniższe polecenia, aby wyświetlić aktualnie aktywne wersje:

```bash
## Sprawdź zainstalowaną wersję Node.js
node -v

## Sprawdź zainstalowaną wersję NPM
npm -v
```

Polecenie `node -v` powinno zwrócić ciąg znaków wersji zaczynający się od `v20` lub wyższej, w zależności od bieżącego wydania LTS. Jeśli zobaczysz wynik w formacie `v22.x.x`, Twój serwer jest poprawnie skonfigurowany do używania najnowszego wydania o długoterminowym wsparciu. Polecenie `npm -v` zwróci wersję menedżera pakietów, potwierdzając, że jest on również gotowy do użycia.

> **Uwaga:** Jeśli napotkasz błąd "command not found", zazwyczaj oznacza to, że ścieżka powłoki nie została zaktualizowana. Wystarczy wylogować się i zalogować ponownie do sesji SSH, aby odświeżyć zmienne środowiskowe.

Wszystko jest teraz skonfigurowane dla Twojego środowiska programistycznego. Z powodzeniem ominąłeś przestarzałe repozytoria systemowe i zabezpieczyłeś stabilny fundament dla swoich aplikacji JavaScript na swoim [Premium VPS](/pl/vps/premium/) lub [Budget VPS](/pl/vps/budget/).

{% image "/assets/images/blog/pl/jak-zainstalowac-nodejs-na-ubuntu-i-debian/H6.png", "Terminal pokazujący wynik poleceń node -v oraz npm -v potwierdzający poprawną instalację", "(max-width: 768px) 100vw, 800px" %}

## Podsumowanie

Pomyślnie skonfigurowałeś stabilne środowisko Node.js na swoim serwerze z systemem Linux. Korzystając z oficjalnego repozytorium NodeSource zamiast domyślnych pakietów dystrybucyjnych, zapewniasz swojemu serwerowi <span class="text-white">Voxi</span><span class="text-amber-300">Host</span> dostęp do terminowych poprawek bezpieczeństwa oraz nowoczesnych funkcji języka JavaScript. Taka konfiguracja gwarantuje stabilność wymaganą w środowiskach produkcyjnych, niezależnie od tego, czy uruchamiasz lekkie API na [Budget VPS](/pl/vps/budget/), czy aplikację o dużym natężeniu ruchu na [Premium VPS](/pl/vps/premium/).

Pamiętaj, że aktualizacja środowiska wykonawczego jest kluczowa zarówno dla wydajności, jak i bezpieczeństwa. Gdy pojawi się nowa wersja LTS, możesz zaktualizować swoje środowisko, odświeżając listy pakietów i aktualizując plik binarny:

```bash
## Aktualizacja lokalnych list pakietów
sudo apt update

## Aktualizacja pakietu Node.js do najnowszej wersji
sudo apt upgrade -y nodejs
```

Jeśli planujesz wdrożenie złożonych aplikacji, rozważ użycie menedżera procesów, takiego jak PM2, aby zapewnić działanie swoich usług po ponownym uruchomieniu serwera. Możesz również zapoznać się z naszym przewodnikiem [Instalacja Nginx na Ubuntu i Debian: kompletny przewodnik po serwerze](/pl/blog/instalacja-nginx-ubuntu-debian/), jeśli zamierzasz skonfigurować odwrotne proxy dla swojej aplikacji Node.js. Twój serwer jest teraz gotowy do pracy, testów lub wdrożenia produkcyjnego. Powodzenia w kodowaniu.

{% image "/assets/images/blog/pl/jak-zainstalowac-nodejs-na-ubuntu-i-debian/H7.png", "Ekran terminala pokazujący pomyślną aktualizację pakietów node", "(max-width: 768px) 100vw, 800px" %}
