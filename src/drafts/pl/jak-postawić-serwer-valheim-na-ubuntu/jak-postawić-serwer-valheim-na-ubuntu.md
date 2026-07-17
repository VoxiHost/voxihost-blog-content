---
image: "/assets/images/blog/pl/jak-postawić-serwer-valheim-na-ubuntu/og-image.png"
title: "Jak postawić serwer Valheim na Ubuntu"
description: "Dowiedz się, jak uruchomić własny serwer Valheim na Ubuntu przy użyciu SteamCMD. Poradnik obejmuje instalację, konfigurację systemd oraz ustawienia zapory."
status: draft
category: Poradniki
tags:
  - valheim
  - game-server
  - ubuntu
  - linux
  - steam
  - vps
date: '2026-07-20'
locale: pl
translationKey: host-valheim-server-ubuntu
author:
  name: VoxiHost Team
  link: https://voxihost.pl/
contributors: []
howto:
  name: "Jak postawić serwer Valheim na Ubuntu"
  steps:
    - name: "Krok 1: Przygotowanie systemu i zależności"
      url: "krok-1-przygotowanie-systemu-i-zależności"
    - name: "Krok 2: Pobranie i instalacja serwera Valheim"
      url: "krok-2-pobranie-i-instalacja-serwera-valheim"
    - name: "Krok 3: Konfiguracja zmiennych środowiskowych"
      url: "krok-3-konfiguracja-zmiennych-środowiskowych"
    - name: "Krok 4: Konfiguracja usługi Systemd"
      url: "krok-4-konfiguracja-usługi-systemd"
    - name: "Krok 5: Otwarcie portów zapory sieciowej i uruchomienie serwera"
      url: "krok-5-otwarcie-portów-zapory-sieciowej-i-uruchomienie-serwera"
faq:
  - question: "Jakie są minimalne wymagania sprzętowe dla serwera Valheim?"
    answer: "Dla płynnej rozgrywki zalecamy przynajmniej <strong>4GB pamięci RAM</strong> oraz <strong>2 rdzenie procesora</strong>. Nasze plany Premium VPS są zoptymalizowane pod kątem tego obciążenia."
  - question: "W jaki sposób zaktualizować dedykowany serwer Valheim?"
    answer: "Serwer można zaktualizować, uruchamiając ponownie komendę aktualizacji <code>steamcmd</code> z flagą <code>app_update 896660 validate</code>, a następnie restartując usługę systemd."
  - question: "Dlaczego mój serwer Valheim nie chce się uruchomić?"
    answer: "Częstymi przyczynami są zbyt krótkie hasło serwera (musi mieć <strong>co najmniej 5 znaków</strong>) lub konflikty portów w skrypcie startowym."
  - question: "Jak ustawić hasło administratora dla serwera Valheim?"
    answer: "Lista administratorów jest zarządzana przez plik <code>adminlist.txt</code>, który tworzy się w folderze konfiguracyjnym serwera dopiero po pierwszym poprawnym uruchomieniu."
  - question: "Czy muszę włączać cross-play na moim serwerze?"
    answer: "Włączenie funkcji cross-play pozwala graczom z różnych platform dołączyć do serwera. Upewnij się, że masz otwarte porty <strong>2456-2457 UDP</strong> w zaporze sieciowej VoxiHost."
---

## Wprowadzenie

Valheim zdefiniował na nowo gatunek survivalu dzięki unikalnemu połączeniu mitologii nordyckiej i proceduralnego generowania świata. Choć gra na serwerach publicznych bywa zabawna, nic nie zastąpi kontroli nad własną, dedykowaną instancją. Uruchamiając własny serwer, zyskujesz pełną władzę nad ustawieniami świata, liczbą graczy i stałą dostępnością, dzięki czemu Twoja baza pozostaje bezpieczna nawet wtedy, gdy jesteś offline.

W celu zapewnienia płynnej rozgrywki zalecamy użycie [Premium VPS](/pl/premium-vps/) z co najmniej 4 GB pamięci RAM i dwoma dedykowanymi rdzeniami procesora. Ten zapas sprzętowy gwarantuje, że silnik fizyczny i symulacja świata pozostaną stabilne, gdy Twoja baza rozrośnie się z małej chaty do rozległej sali biesiadnej. Uruchomienie serwera na [Budget VPS](/pl/budget-vps/) jest możliwe w przypadku mniejszych grup, jednak należy pamiętać, że Valheim jest wymagający dla procesora podczas intensywnej walki lub modyfikacji terenu.

W tym przewodniku przejdziemy przez cały proces wdrażania na systemie Ubuntu 22.04 LTS. Skupiamy się na użyciu SteamCMD do czystej instalacji oraz systemd do niezawodnego zarządzania procesami. Po zakończeniu tego samouczka Twój świat Valheim będzie aktywny, dostępny dla znajomych i zarządzany jako natywna usługa działająca w tle, która automatycznie uruchamia się ponownie po restarcie systemu.

{% image "/assets/images/blog/pl/jak-postawić-serwer-valheim-na-ubuntu/H1.png", "Menu główne gry Valheim pokazujące połączenie z serwerem dedykowanym", "(max-width: 768px) 100vw, 800px" %}

## Wymagania wstępne

Przed rozpoczęciem instalacji upewnij się, że Twój serwer <span class="text-white">Voxi</span><span class="text-amber-300">Host</span> posiada świeżą instalację systemu Ubuntu 22.04 LTS lub nowszego. Czyste środowisko zapobiega konfliktom z istniejącymi bibliotekami lub innymi usługami gier. Powinieneś posiadać dostęp do terminala z uprawnieniami roota lub sudo, ponieważ będziemy instalować zależności systemowe oraz konfigurować dedykowane konto użytkownika.

Upewnij się, że Twój firewall jest przygotowany na obsługę ruchu gry. Valheim wymaga otwarcia portów UDP w zakresie od 2456 do 2457 zarówno dla standardowej rozgrywki, jak i funkcji cross-play. Jeśli używasz UFW, miej na uwadze te wymagania, choć szczegółowe reguły skonfigurujemy w dalszej części poradnika.

Zweryfikuj, czy Twój serwer dysponuje co najmniej 4 GB pamięci RAM. Choć gra może uruchomić się przy mniejszej ilości, narzut pamięci podczas generowania świata i śledzenia bytów może prowadzić do poważnych spadków wydajności lub nieoczekiwanych awarii. Minimum 2 rdzenie procesora jest niezbędne do obsługi obciążenia symulacji, zwłaszcza podczas starć lub gdy wielu graczy przebywa w tym samym obszarze. Upewnij się, że na swoim lokalnym komputerze masz gotowego klienta SSH, aby połączyć się z instancją serwera.

{% image "/assets/images/blog/pl/jak-postawić-serwer-valheim-na-ubuntu/H2.png", "Widok terminala potwierdzający podstawowe wymagania systemowe i dostęp SSH", "(max-width: 768px) 100vw, 800px" %}

## Krok 1: Przygotowanie systemu i zależności

Aby uruchomić dedykowany serwer gry, musimy najpierw włączyć repozytorium multiverse i upewnić się, że system obsługuje biblioteki 32-bitowe, które są wymagane przez platformę Steam. Rozpocznij od aktualizacji list pakietów i instalacji niezbędnych komponentów SteamCMD.

Uruchom poniższe polecenia, aby skonfigurować repozytorium i pobrać wymagane biblioteki:

```bash
## Włącz repozytorium multiverse dla oprogramowania niewolnego
sudo add-apt-repository multiverse

## Dodaj obsługę architektury 32-bitowej
sudo dpkg --add-architecture i386

## Odśwież listy pakietów i zainstaluj SteamCMD
sudo apt update
sudo apt install -y lib32gcc-s1 steamcmd
```

Po zakończeniu instalacji należy zabezpieczyć środowisko poprzez utworzenie dedykowanego użytkownika systemowego. Uruchamianie serwerów gier jako użytkownik root stanowi poważne zagrożenie bezpieczeństwa. Izolując proces na oddzielnym koncie użytkownika, ograniczamy potencjalny wpływ ewentualnych luk w oprogramowaniu serwerowym.

Wykonaj to polecenie, aby utworzyć użytkownika:

```bash
## Utwórz dedykowanego użytkownika systemowego dla Valheim
sudo useradd -m valheim
```

To polecenie tworzy nowego użytkownika o nazwie `valheim` i automatycznie generuje katalog domowy w lokalizacji `/home/valheim`. Katalog ten będzie służył jako baza dla plików serwera oraz konfiguracji. Po zainstalowaniu zależności i przygotowaniu konta użytkownika, mamy już solidne fundamenty pod instalację serwera.

{% image "/assets/images/blog/pl/jak-postawić-serwer-valheim-na-ubuntu/H3.png", "Terminal pokazujący pomyślną instalację SteamCMD oraz utworzenie użytkownika valheim", "(max-width: 768px) 100vw, 800px" %}

## Krok 2: Pobranie i instalacja serwera Valheim

Teraz wypełnimy katalog serwera plikami gry. Aby uniknąć konfliktów uprawnień i zachować porządek w systemie, proces pobierania przeprowadzimy bezpośrednio jako nowo utworzony użytkownik `valheim`. Dzięki temu wszystkie pliki gry, logi oraz dane konfiguracyjne będą własnością odpowiedniego konta, a nie użytkownika root.

Do pobrania kompilacji dedykowanego serwera użyjemy narzędzia SteamCMD. Należy pamiętać, że określamy AppID `896660`, który jest oficjalnym identyfikatorem dedykowanego serwera Valheim.

Wykonaj poniższe polecenie, aby pobrać pliki serwera:

```bash
## Pobierz i zweryfikuj pliki serwera Valheim w lokalnym katalogu
sudo -u valheim /usr/games/steamcmd +force_install_dir /home/valheim/server +login anonymous +app_update 896660 validate +quit
```

Proces ten może potrwać kilka minut, w zależności od prędkości Twojego łącza. Flaga `validate` gwarantuje, że wszystkie pliki zostały pobrane poprawnie i są kompletne. Po zakończeniu działania polecenia, plik wykonywalny serwera oraz powiązane z nim biblioteki będą znajdować się w katalogu `/home/valheim/server`.

Możesz potwierdzić obecność plików, wyświetlając zawartość katalogu:

```bash
## Zweryfikuj obecność plików serwera w docelowym katalogu
ls -lh /home/valheim/server
```

Powinieneś zobaczyć listę plików, w tym `valheim_server.x86_64`. Po pomyślnym pobraniu plików i przypisaniu ich własności do użytkownika `valheim`, jesteśmy gotowi przejść do fazy konfiguracji usługi.

{% image "/assets/images/blog/pl/jak-postawić-serwer-valheim-na-ubuntu/H4.png", "Terminal pokazujący pomyślne pobranie plików serwera Valheim za pomocą SteamCMD", "(max-width: 768px) 100vw, 800px" %}

## Krok 3: Konfiguracja zmiennych środowiskowych

Valheim wymaga dostępu do określonych bibliotek Steam, aby poprawnie działać w systemie Linux. Jeśli te ścieżki nie zostaną jawnie wskazane, plik binarny nie uruchomi się, zgłaszając błędy o brakujących bibliotekach. Aby rozwiązać ten problem, utworzymy skrypt, który ustawia `LD_LIBRARY_PATH` przed wywołaniem serwera.

Utwórz skrypt pomocniczy w katalogu serwera:

```bash
## Utwórz skrypt pomocniczy
sudo -u valheim nano /home/valheim/server/start_valheim.sh
```

Wklej poniższą zawartość do pliku:

```bash
#!/bin/bash
export LD_LIBRARY_PATH=./linux64:$LD_LIBRARY_PATH
export SteamAppId=896660
./valheim_server.x86_64 -name "TwojaNazwaSerwera" -port 2456 -world "Dedykowany" -password "TwojeHaslo" -public 1
```

Zapisz plik i nadaj mu uprawnienia do wykonywania:

```bash
## Nadaj uprawnienia do wykonywania skryptu
sudo -u valheim chmod +x /home/valheim/server/start_valheim.sh
```

Dzięki temu serwer przy każdym uruchomieniu poprawnie wczyta zależności z folderu `linux64`.

{% image "/assets/images/blog/pl/jak-postawić-serwer-valheim-na-ubuntu/H5.png", "Terminal z widocznym utworzonym skryptem pomocniczym", "(max-width: 768px) 100vw, 800px" %}

## Krok 4: Konfiguracja usługi systemd

Aby zapewnić automatyczne uruchamianie serwera po restarcie systemu oraz jego niezawodne działanie w tle, utworzymy dedykowaną usługę systemd. Takie rozwiązanie jest znacznie lepsze niż ręczne uruchamianie serwera w sesji terminala, ponieważ zapewnia wbudowaną logikę restartu oraz standardowe funkcje logowania zdarzeń.

Otwórz nowy plik usługi przy użyciu wybranego edytora tekstu:

```bash
## Utwórz plik definicji usługi systemd
sudo nano /etc/systemd/system/valheim.service
```

Wklej poniższą konfigurację do pliku. Ta jednostka definiuje środowisko, ustawia katalog roboczy oraz wykonuje skrypt startowy przygotowany w poprzednim kroku.

```ini
[Unit]
Description=Serwer dedykowany Valheim
After=network.target

[Service]
Type=simple
User=valheim
Group=valheim
WorkingDirectory=/home/valheim/server
ExecStart=/home/valheim/server/start_valheim.sh
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

> **Ostrzeżenie:** Hasło serwera musi mieć co najmniej 5 znaków. Jeśli będzie krótsze, binarny plik Valheim nie zainicjuje się poprawnie, a usługa przejdzie w stan pętli awaryjnej.

Po zapisaniu i zamknięciu pliku, przeładuj menedżer systemd, aby zarejestrować nową usługę.

```bash
## Przeładuj systemd i uruchom usługę Valheim
sudo systemctl daemon-reload
sudo systemctl enable --now valheim
```

Twój serwer jest teraz aktywny. Możesz w dowolnym momencie sprawdzić jego status, wykonując polecenie `sudo systemctl status valheim`.

{% image "/assets/images/blog/pl/jak-postawić-serwer-valheim-na-ubuntu/H6.png", "Widok konfiguracji pliku usługi systemd dla serwera Valheim", "(max-width: 768px) 100vw, 800px" %}

## Krok 5: Otwarcie portów zapory sieciowej i uruchomienie serwera

Valheim wymaga określonych portów sieciowych do obsługi przychodzących połączeń graczy oraz matchmakingu Steam. Domyślnie gra wykorzystuje protokół UDP na portach od 2456 do 2457. Jeśli używasz `ufw` do zarządzania bezpieczeństwem serwera, musisz jawnie zezwolić na ten ruch, aby dotarł do procesu gry.

Wykonaj poniższe polecenia, aby skonfigurować reguły zapory sieciowej:

```bash
## Zezwól na niezbędny ruch UDP dla Valheim
sudo ufw allow 2456:2457/udp
```

Jeśli musisz odświeżyć usługę, aby upewnić się, że najnowsze konfiguracje zostały zastosowane, użyj następujących poleceń:

```bash
## Zrestartuj usługę, aby zastosować ostatnie zmiany
sudo systemctl restart valheim
```

Możesz potwierdzić, że serwer działa poprawnie, sprawdzając dzienniki systemowe. Jest to szczególnie przydatne w przypadku napotkania problemów z połączeniem, ponieważ wyświetli postęp inicjalizacji serwera i pokaże, czy pomyślnie zarejestrował się on w głównym serwerze Steam.

```bash
## Wyświetl bieżące logi serwera
sudo journalctl -u valheim -f
```

> **Uwaga:** Serwer generuje kilka plików zarządzania, takich jak `adminlist.txt`, `bannedlist.txt` oraz `permittedlist.txt`, dopiero po pierwszym udanym uruchomieniu. Jeśli nie widzisz tych plików w `/home/valheim/server/`, upewnij się, że usługa działa oraz że Twój użytkownik posiada niezbędne uprawnienia do zapisu w tym katalogu.

Twój serwer jest teraz osiągalny. Gracze mogą łączyć się, używając publicznego adresu IP Twojego serwera, po którym następuje port 2456. Jeśli korzystasz z serwera <span class="text-white">Voxi</span><span class="text-amber-300">Host</span> [Premium VPS](/pl/premium-vps/), upewnij się, że ustawienia zapory sieciowej w panelu również zezwalają na ten ruch UDP.

{% image "/assets/images/blog/pl/jak-postawić-serwer-valheim-na-ubuntu/H7.png", "Terminal pokazujący aktywny status usługi systemd Valheim oraz reguły zapory sieciowej", "(max-width: 768px) 100vw, 800px" %}

## Krok końcowy

Twój dedykowany serwer Valheim działa już w pełni na systemie Ubuntu. Przenosząc symulację gry na <span class="text-white">Voxi</span><span class="text-amber-300">Host</span> [Premium VPS](/pl/premium-vps/), zapewniasz stałą wydajność oraz niskie opóźnienia dla swoich graczy, niezależnie od jakości Twojego lokalnego łącza internetowego.

Utrzymanie serwera wymaga jedynie przestrzegania kilku prostych nawyków. Aby zainstalować aktualizacje gry, należy zatrzymać usługę, uruchomić proces aktualizacji SteamCMD, a następnie ponownie włączyć serwer. Pozwala to uniknąć uszkodzenia plików podczas trwania aktualizacji.

```bash
## Zatrzymanie serwera przed aktualizacją
sudo systemctl stop valheim

## Uruchomienie komendy aktualizującej
sudo -u valheim /usr/games/steamcmd +force_install_dir /home/valheim/server +login anonymous +app_update 896660 validate +quit

## Ponowne uruchomienie serwera
sudo systemctl start valheim
```

W przypadku zaobserwowania nieoczekiwanego zachowania lub spadków wydajności, w pierwszej kolejności należy sprawdzić logi za pomocą komendy `sudo journalctl -u valheim -n 50`. Większość problemów konfiguracyjnych wynika z nieprawidłowych uprawnień do plików lub reguł zapory sieciowej, które kolidują z zewnętrznymi ograniczeniami sieciowymi.

Pamiętaj, że dane Twojego świata są przechowywane w katalogu wskazanym podczas instalacji. Regularne tworzenie kopii zapasowych katalogu `/home/valheim/server/` jest niezbędne, aby chronić postępy przed przypadkowym usunięciem świata lub błędami w konfiguracji. Gdy serwer jest aktywny, a zaplecze zabezpieczone, możesz zaprosić znajomych do odkrywania dziesiątego świata Valheim bez ograniczeń typowych dla hostingu typu peer-to-peer.

{% image "/assets/images/blog/pl/jak-postawić-serwer-valheim-na-ubuntu/H8.png", "Serwer Valheim działający pomyślnie w terminalu Linux", "(max-width: 768px) 100vw, 800px" %}
