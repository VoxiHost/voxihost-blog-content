---
image: /assets/images/blog/pl/jak-zalozyc-serwer-teamspeak-3-ubuntu-debian/og-image.png
title: Jak założyć serwer TeamSpeak 3 na Ubuntu i Debian
description: Kompletny poradnik krok po kroku, z którego dowiesz się, jak zainstalować, skonfigurować i zabezpieczyć własny serwer TeamSpeak 3 na swoim VPS.
date: '2026-06-14'
translationKey: setup-teamspeak-3-server-ubuntu-debian
locale: pl
category: Poradniki
tags:
  - teamspeak
  - komunikator
  - ubuntu
  - debian
howto:
  name: Jak założyć serwer TeamSpeak 3 na Ubuntu i Debian
  totalTime: PT15M
  yield: W pełni działający serwer głosowy TeamSpeak 3 gotowy do użycia
  tool:
    - VPS z systemem Ubuntu lub Debian
    - Klient SSH
    - Konto użytkownika z uprawnieniami sudo
  steps:
    - name: Krok 1 — Aktualizacja systemu
      text: Zaktualizuj pakiety i zainstaluj wymagane narzędzia, takie jak wget i bzip2.
      url: '#krok-1--aktualizacja-systemu'
    - name: Krok 2 — Tworzenie dedykowanego użytkownika
      text: Utwórz bezpiecznego, odizolowanego użytkownika do obsługi serwera TeamSpeak.
      url: '#krok-2--tworzenie-dedykowanego-uzytkownika'
    - name: Krok 3 — Pobieranie i rozpakowywanie
      text: Pobierz najnowsze pliki serwera TeamSpeak 3 i rozpakuj archiwum.
      url: '#krok-3--pobieranie-i-rozpakowywanie-teamspeak-3'
    - name: Krok 4 — Akceptacja licencji
      text: Zaakceptuj licencję użytkownika TeamSpeak, co pozwoli na uruchomienie serwera.
      url: '#krok-4--akceptacja-licencji-eula'
    - name: Krok 5 — Pierwsze uruchomienie i Privilege Key
      text: Uruchom serwer ręcznie, aby przechwycić klucz administratora (Privilege Key).
      url: '#krok-5--pierwsze-uruchomienie-i-privilege-key'
    - name: Krok 6 — Usługa Systemd
      text: Utwórz usługę systemd, aby serwer działał automatycznie w tle.
      url: '#krok-6--konfiguracja-uslugi-systemd'
status: draft
author:
  name: VoxiHost Team
  link: https://voxihost.pl/
contributors:
  - sl0ikkk
---

TeamSpeak 3 to od wielu lat jeden z najbardziej niezawodnych i stabilnych komunikatorów głosowych dla graczy oraz społeczności internetowych. Stawiając własny serwer na VPS zyskujesz pełną kontrolę, prywatność oraz niezależność od zewnętrznych dostawców.

Z tego poradnika dowiesz się, jak prawidłowo zainstalować i zabezpieczyć własny serwer TeamSpeak 3 na systemie Ubuntu lub Debian.

> **Wymagania wstępne:** Przed rozpoczęciem upewnij się, że posiadasz [serwer VPS z systemem Ubuntu lub Debian](/pl/premium-vps/), dostęp SSH oraz użytkownika z uprawnieniami `sudo`.

## Krok 1 — Aktualizacja systemu

Zacznijmy od odświeżenia listy pakietów oraz instalacji niezbędnych narzędzi (`wget` do pobierania plików, `bzip2` do rozpakowywania archiwum):

{% image "/assets/images/blog/pl/jak-zalozyc-serwer-teamspeak-3-ubuntu-debian/H1.png", "Aktualizacja systemu i instalacja wget oraz bzip2 na Ubuntu", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install wget bzip2 -y
```

## Krok 2 — Tworzenie dedykowanego użytkownika

Ze względów bezpieczeństwa żadna usługa dostępna publicznie nie powinna być uruchamiana z konta `root`. Stwórzmy nowego użytkownika dedykowanego specjalnie dla serwera TeamSpeak:

{% image "/assets/images/blog/pl/jak-zalozyc-serwer-teamspeak-3-ubuntu-debian/H2.png", "Tworzenie użytkownika systemowego teamspeak", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo adduser --disabled-password --gecos "" teamspeak
```

Zaloguj się teraz na to nowo utworzone konto:

```bash
sudo su - teamspeak
```

## Krok 3 — Pobieranie i rozpakowywanie TeamSpeak 3

Kolejnym krokiem jest pobranie plików serwerowych. Najnowszą wersję znajdziesz zawsze na [oficjalnej stronie pobierania TeamSpeak](https://teamspeak.com/en/downloads/#server).

{% image "/assets/images/blog/pl/jak-zalozyc-serwer-teamspeak-3-ubuntu-debian/H3.png", "Pobieranie plików serwerowych TeamSpeak 3", "(max-width: 768px) 100vw, 800px" %}

```bash
wget https://files.teamspeak-services.com/releases/server/3.13.8/teamspeak3-server_linux_amd64-3.13.8.tar.bz2
```

Rozpakuj pobrane archiwum i przenieś pliki bezpośrednio do katalogu domowego, aby zachować porządek, po czym usuń zbędne puste foldery:

```bash
tar xvf teamspeak3-server_linux_amd64-3.13.8.tar.bz2
mv teamspeak3-server_linux_amd64/* .
rm -rf teamspeak3-server_linux_amd64 teamspeak3-server_linux_amd64-3.13.8.tar.bz2
```

## Krok 4 — Akceptacja licencji (EULA)

Serwer TeamSpeak 3 nie uruchomi się, dopóki nie zaakceptujesz warunków umowy EULA. Robi się to w bardzo prosty sposób, tworząc pusty plik o nazwie `.ts3server_license_accepted`:

{% image "/assets/images/blog/pl/jak-zalozyc-serwer-teamspeak-3-ubuntu-debian/H4.png", "Akceptacja warunków licencji TeamSpeak 3", "(max-width: 768px) 100vw, 800px" %}

```bash
touch .ts3server_license_accepted
```

## Krok 5 — Pierwsze uruchomienie i Privilege Key

To **niezwykle ważny** krok. Musimy uruchomić serwer ręcznie, ponieważ przy pierwszym starcie konsola wygeneruje klucz uprawnień (**Privilege Key**) oraz dane do ServerQuery. Klucz ten będzie Ci potrzebny, aby zdobyć uprawnienia administratora w swoim kliencie TS3.

{% image "/assets/images/blog/pl/jak-zalozyc-serwer-teamspeak-3-ubuntu-debian/H5.png", "Ręczne uruchomienie serwera TS3, aby wygenerować klucz uprawnień", "(max-width: 768px) 100vw, 800px" %}

```bash
./ts3server_startscript.sh start
```

Skopiuj `Privilege Key` oraz `ServerAdmin password` i zapisz je w bezpiecznym miejscu!

Kiedy już to zrobisz, wyłącz serwer, ponieważ za chwilę skonfigurujemy go tak, aby działał jako usługa systemowa w tle:

```bash
./ts3server_startscript.sh stop
```

## Krok 6 — Konfiguracja usługi Systemd

Profesjonalnie postawiony serwer powinien uruchamiać się automatycznie w tle i wstawać samodzielnie po restarcie maszyny. Wyloguj się z konta `teamspeak`, powracając na swoje główne konto (`sudo`):

```bash
exit
```

Teraz stwórz nowy plik usługi dla systemd:

{% image "/assets/images/blog/pl/jak-zalozyc-serwer-teamspeak-3-ubuntu-debian/H6.png", "Tworzenie usługi systemd dla serwera TeamSpeak", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo nano /etc/systemd/system/teamspeak.service
```

Wklej do edytora poniższą konfigurację:

```ini
[Unit]
Description=TeamSpeak 3 Server
After=network.target

[Service]
WorkingDirectory=/home/teamspeak
User=teamspeak
Group=teamspeak
Type=forking
ExecStart=/home/teamspeak/ts3server_startscript.sh start inifile=ts3server.ini
ExecStop=/home/teamspeak/ts3server_startscript.sh stop
PIDFile=/home/teamspeak/ts3server.pid
RestartSec=15
Restart=always

[Install]
WantedBy=multi-user.target
```

Zapisz plik i zamknij edytor (`CTRL + O`, `ENTER`, `CTRL + X`).

Przeładuj konfigurację systemd, włącz uruchamianie przy starcie systemu i wystartuj serwer TeamSpeak:

{% image "/assets/images/blog/pl/jak-zalozyc-serwer-teamspeak-3-ubuntu-debian/H7.png", "Aktywacja i startowanie usługi TeamSpeak w tle", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo systemctl daemon-reload
sudo systemctl enable teamspeak
sudo systemctl start teamspeak
```

Możesz zweryfikować, czy usługa działa poprawnie, używając polecenia:

```bash
sudo systemctl status teamspeak
```

## Podsumowanie

Gratulacje! Twój serwer TeamSpeak 3 właśnie wystartował i działa poprawnie.

Uruchom teraz swój program TeamSpeak 3 na komputerze i połącz się ze swoim serwerem wpisując adres IP VPS-a. Pojawi się okienko z prośbą o podanie klucza — wklej tam **Privilege Key**, który skopiowałeś w Kroku 5, aby błyskawicznie stać się Administratorem Serwera.

### Kolejne kroki
* Jeśli posiadasz aktywny firewall (zaporę sieciową), musisz otworzyć dla TeamSpeaka porty. Sprawdź nasz poradnik, [jak skonfigurować UFW](/pl/blog/konfiguracja-ufw-ubuntu-debian/). TeamSpeak wymaga portów `9987/udp` (Głos), `10011/tcp` (ServerQuery) oraz `30033/tcp` (Transfer plików).
* Szukasz serwera, który zniesie duże zloty graczy i uchroni Cię przed atakami? Sprawdź [tani hosting VPS na godziny](/pl/budget-vps/), a jeśli grasz profesjonalnie – postaw na niezawodność z [VoxiShield](/pl/shield/) i wybierz serwer w ofercie [Premium VPS](/pl/premium-vps/).
