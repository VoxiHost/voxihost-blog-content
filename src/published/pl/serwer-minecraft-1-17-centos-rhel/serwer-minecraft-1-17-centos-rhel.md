---
image: /assets/images/blog/pl/serwer-minecraft-1-17-centos-rhel/og-image.png
title: Jak postawić serwer Minecraft Vanilla 1.17.1 (Java 16) na AlmaLinux, CentOS, Rocky Linux i Fedorze
description: Instrukcja instalacji serwera Minecraft 1.17.1 na AlmaLinux, CentOS lub Rocky Linux z użyciem środowiska Java 16/17.
date: '2026-04-23'
translationKey: minecraft-vanilla-java-1-17-server-setup-almalinux-centos-rocky-fedora
locale: pl
category: Poradniki
tags:
  - minecraft
  - vanilla
  - java 16
  - java 17
  - konfiguracja serwera
  - almalinux
  - rocky linux
  - centos
howto:
  name: Jak postawić serwer Minecraft Vanilla 1.17 na AlmaLinux, CentOS, Rocky Linux i Fedorze
  totalTime: PT10M
  yield: W pełni działający serwer Minecraft 1.17 z wariantem zapasowym Java 17
  tool:
    - AlmaLinux/Rocky VPS
    - Java 17 JRE
    - Klient SSH
  steps:
    - name: Weryfikacja repozytorium
      text: Upewnij się, że 'sudo dnf check-update' zostało uruchomione, aby znaleźć pakiety OpenJDK.
      url: '#krok-1-instalacja-java-17-wariant-zapasowy'
    - name: Wdrożenie Javy
      text: Zainstaluj środowisko Java 17 wymagane dla stabilności wersji 1.17.
      url: '#krok-1-instalacja-java-17-wariant-zapasowy'
    - name: Tworzenie użytkownika gry
      text: Skonfiguruj ograniczonego użytkownika 'minecraft' ze względów bezpieczeństwa za pomocą 'useradd'.
      url: '#krok-2-tworzenie-dedykowanego-uzytkownika'
    - name: Pobranie server.jar
      text: Pobierz oficjalny plik binarny 1.17.1 od Mojang.
      url: '#krok-3-pobranie-1-17-1'
    - name: Akceptacja EULA
      text: Wygeneruj i zaakceptuj plik eula.txt, aby umożliwić uruchomienie serwera.
      url: '#krok-4-akceptacja-eula'
    - name: Konfiguracja pamięci
      text: Wdróż skrypt 'start.sh' z optymalizowanymi flagami RAM od Aikara.
      url: '#krok-5-tworzenie-skryptu-startowego'
    - name: Pierwsze uruchomienie i OP
      text: Uruchom serwer ręcznie, aby nadać sobie uprawnienia administratora.
      url: '#krok-6-pierwsze-uruchomienie-i-konfiguracja-administratora'
    - name: Profesjonalne uruchamianie (Systemd)
      text: Skonfiguruj usługę systemd, aby serwer uruchamiał się automatycznie po restarcie.
      url: '#krok-7-konfiguracja-uslugi-systemd'
status: published
author:
  name: VoxiHost Team
  link: https://voxihost.pl/
contributors:
  - danielmarszalkowski
---

Minecraft 1.17.x pierwotnie wymagał **Java 16**. Jednak ponieważ Java 16 była krótkotrwałym wydaniem, rzadko jest dostępna w standardowych repozytoriach pochodnych RHEL (AlmaLinux, Rocky Linux, CentOS 9 Stream). Dlatego użyjemy **Java 17**, która jest w pełni wstecznie kompatybilna z wersją 1.17.1. Szerszy przegląd wymagań Javy znajdziesz w naszym [Poradniku kompatybilności serwerów Minecraft Java](/pl/blog/jak-zalozyc-serwer-minecraft-centos-rhel/).

> Bezpieczeństwo przede wszystkim: uruchamianie publicznych serwerów gier jako użytkownik `root` niepotrzebnie naraża cały system. Uważnie wykonaj krok 2, aby skonfigurować bezpieczne środowisko.

### Obsługiwane wersje
Ten poradnik jest w pełni kompatybilny z:
* **Era 1.17:** 1.17.1, 1.17

Dokładny link do pobrania swojej wersji znajdziesz w naszym [Archiwum linków do serwerów Minecraft Vanilla](/pl/blog/serwer-minecraft-linki-do-pobrania/).

## Wymagania wstępne

* VPS z systemem **AlmaLinux, Rocky Linux lub CentOS** (dostępny w ramach [Premium VPS](/pl/premium-vps/)).
* Dostęp root lub sudo przez SSH (do instalacji Javy).
* **Ograniczony użytkownik inny niż root** do bezpiecznego uruchamiania serwera.

## Krok 1: Instalacja Java 17 (wariant zapasowy)

Przed przystąpieniem zalecamy [aktualizację systemu](/pl/blog/jak-zaktualizowac-centos-rhel/), aby zapewnić stabilność.

Jak wspomniano, Java 16 to starsza wersja przejściowa niedostępna w domyślnych repozytoriach `dnf` dla RHEL. Instalacja standardowej LTS **Java 17** jest najbezpieczniejszym i zalecanym podejściem.

```bash
sudo dnf check-update
sudo dnf install java-17-openjdk-headless wget -y
```

{% image "/assets/images/blog/pl/serwer-minecraft-1-17-centos-rhel/H1.png", "Terminal przedstawiający instalację OpenJDK na systemie Linux", "(max-width: 768px) 100vw, 800px" %}

## Krok 2: Tworzenie dedykowanego użytkownika

Dla bezpieczeństwa nigdy nie uruchamiaj serwera jako root. Jeśli dopiero zaczynasz z uprawnieniami w Linuksie, zapoznaj się z naszym poradnikiem [Tworzenia i zarządzania użytkownikami na AlmaLinux/Rocky](/pl/blog/jak-dodac-uzytkownika-sudo-centos/).

{% image "/assets/images/blog/pl/serwer-minecraft-1-17-centos-rhel/H2.png", "Tworzenie dedykowanego użytkownika 'minecraft' do bezpiecznego hostowania serwera", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo useradd -m -r -s /bin/bash minecraft
sudo su - minecraft
mkdir server && cd server
```

## Krok 3: Pobranie 1.17.1

Szukasz innej wersji? Bezpośrednie linki do pobrania od Mojang dla wszystkich wydań znajdziesz w naszym [Archiwum linków do serwerów Minecraft](/pl/blog/serwer-minecraft-linki-do-pobrania/).

{% image "/assets/images/blog/pl/serwer-minecraft-1-17-centos-rhel/H3.png", "Pobieranie pliku Minecraft 1.17.1 server.jar za pomocą wget", "(max-width: 768px) 100vw, 800px" %}

```bash
wget https://piston-data.mojang.com/v1/objects/a16d67e5807f57fc4e550299cf20226194497dc2/server.jar
```

## Krok 4: Akceptacja EULA

{% image "/assets/images/blog/pl/serwer-minecraft-1-17-centos-rhel/H4.png", "Pierwsze uruchomienie pliku JAR 1.17.1 w celu wygenerowania plików konfiguracyjnych i akceptacji EULA", "(max-width: 768px) 100vw, 800px" %}

Uruchom serwer raz, aby wygenerować wymagane pliki konfiguracyjne:

```bash
java -jar server.jar nogui
sed -i 's/eula=false/eula=true/' eula.txt
```

## Krok 5: Tworzenie skryptu startowego

> **Pro Tip: Edytor Nano**
> Nano to przyjazny dla początkujących edytor tekstu w terminalu. Jeśli polecenie `nano` nie zostanie znalezione, zainstaluj go: `sudo dnf install nano -y`.
> * **Aby zapisać:** Wciśnij `CTRL + O`, a następnie `ENTER`.
> * **Aby wyjść:** Wciśnij `CTRL + X`.

Wklej poniższą zawartość (flagi Aikara zoptymalizowane pod G1GC):

{% image "/assets/images/blog/pl/serwer-minecraft-1-17-centos-rhel/H5.png", "Używanie edytora nano do tworzenia i konfigurowania skryptu startowego start.sh", "(max-width: 768px) 100vw, 800px" %}

```bash
nano start.sh
```

W edytorze wklej:
```bash
#!/bin/bash
java -Xmx4G -Xms4G -XX:+UseG1GC -XX:+ParallelRefProcEnabled -XX:MaxGCPauseMillis=200 -XX:+UnlockExperimentalVMOptions -XX:+DisableExplicitGC -XX:+AlwaysPreTouch -XX:G1NewSizePercent=30 -XX:G1MaxNewSizePercent=40 -XX:G1HeapRegionSize=8M -XX:G1ReservePercent=20 -XX:G1HeapWastePercent=5 -XX:G1MixedGCCountTarget=4 -XX:InitiatingHeapOccupancyPercent=15 -XX:G1MixedGCLiveThresholdPercent=90 -XX:G1RSetUpdatingPauseTimePercent=5 -XX:SurvivorRatio=32 -XX:+PerfDisableSharedMem -XX:MaxTenuringThreshold=1 -Dusing.aikars.flags=https://mcflags.emc.gs -Daikar.for.v1.20=false -jar server.jar nogui
```

{% image "/assets/images/blog/pl/serwer-minecraft-1-17-centos-rhel/H6.png", "Nadawanie uprawnień do wykonywania skryptowi start.sh", "(max-width: 768px) 100vw, 800px" %}

Nadaj uprawnienia do wykonywania:
```bash
chmod +x start.sh
```

## Krok 6: Pierwsze uruchomienie i konfiguracja administratora

Przed skonfigurowaniem usługi działającej w tle powinieneś uruchomić serwer ręcznie przynajmniej raz, aby nadać sobie uprawnienia administratora (**OP**).

**1. Ręczne uruchomienie serwera**

{% image "/assets/images/blog/pl/serwer-minecraft-1-17-centos-rhel/H7.png", "Ręczne uruchamianie serwera Minecraft 1.17.1 w celu uzyskania dostępu do konsoli", "(max-width: 768px) 100vw, 800px" %}

Uruchom właśnie utworzony skrypt startowy:
```bash
./start.sh
```

**2. Nadanie uprawnień administratora (OP)**

{% image "/assets/images/blog/pl/serwer-minecraft-1-17-centos-rhel/H8.png", "Używanie polecenia op w konsoli do nadania uprawnień administratora", "(max-width: 768px) 100vw, 800px" %}

Gdy serwer zakończy ładowanie (zobaczysz komunikat „Done!"), wpisz bezpośrednio w konsoli:
```text
op twoja_nazwa_gracza_minecraft
```

**3. Zatrzymanie serwera**

{% image "/assets/images/blog/pl/serwer-minecraft-1-17-centos-rhel/H9.png", "Wykonywanie polecenia stop w celu bezpiecznego wyłączenia serwera", "(max-width: 768px) 100vw, 800px" %}

Aby zapisać dane świata i przygotować serwer do działania w tle, wpisz:
```text
stop
```
Spowoduje to powrót do normalnej linii poleceń Linuksa.

## Krok 7: Konfiguracja usługi Systemd

Dla profesjonalnej konfiguracji używamy `systemd`. Dzięki temu serwer uruchamia się automatycznie po restarcie VPS i obsługuje awarie w sposób kontrolowany.

Wyjdź z konta użytkownika `minecraft` z powrotem na konto root/sudo:
```bash
exit
```

{% image "/assets/images/blog/pl/serwer-minecraft-1-17-centos-rhel/H10.png", "Tworzenie pliku minecraft.service dla profesjonalnego hostowania w tle", "(max-width: 768px) 100vw, 800px" %}

Utwórz plik usługi:
```bash
sudo nano /etc/systemd/system/minecraft.service
```

Wklej poniższą konfigurację:
```ini
[Unit]
Description=VoxiHost Minecraft 1.17 Server
After=network.target

[Service]
User=minecraft
WorkingDirectory=/home/minecraft/server
ExecStart=/home/minecraft/server/start.sh
Restart=on-failure
RestartSec=10

[Install]
WantedBy=multi-user.target
```

{% image "/assets/images/blog/pl/serwer-minecraft-1-17-centos-rhel/H11.png", "Włączanie i uruchamianie usługi minecraft w systemd", "(max-width: 768px) 100vw, 800px" %}

Włącz i uruchom serwer:
```bash
sudo systemctl daemon-reload
sudo systemctl enable minecraft
sudo systemctl start minecraft
```

### Zarządzanie serwerem
* **Sprawdź status:** `sudo systemctl status minecraft`
* **Podgląd logów:** `sudo journalctl -u minecraft -f`
* **Zatrzymaj serwer:** `sudo systemctl stop minecraft`

## Kolejne kroki: bezpieczeństwo i zarządzanie

Teraz, gdy serwer działa, pamiętaj o:

1. **Ochrona DDoS**: Wszystkie serwery <span class="text-white">Voxi</span><span class="text-amber-300">Host</span> są objęte automatyczną ochroną [VoxiShield](/pl/shield/). Twój serwer jest już monitorowany, aby zapobiec przestojom podczas ataków.
2. **Otwarcie zapory**: Zezwól na ruch na porcie `25565` poleceniami: `sudo firewall-cmd --permanent --add-port=25565/tcp` a następnie `sudo firewall-cmd --reload`. Szczegóły znajdziesz w naszym [Poradniku konfiguracji Firewalld](/pl/blog/konfiguracja-firewalld-centos-rhel/).
3. **Transfer plików**: Chcesz wgrać istniejący świat? Skorzystaj z SFTP zgodnie z opisem w naszym [Tutorialu FileZilla](/pl/blog/przesylanie-plikow-vps-sftp-filezilla/).
4. **Utwardzenie i monitoring**: Zwiększ bezpieczeństwo VPS, [zabezpieczając SSH](/pl/blog/jak-zabezpieczyc-ssh-centos-rhel/) i [konfigurując Fail2ban](/pl/blog/konfiguracja-fail2ban-centos-rhel/). Możesz też [monitorować zasoby systemowe](/pl/blog/monitorowanie-vps-htop-df-free/) za pomocą htop.

Szukasz stabilnego domu dla swojego świata? Sprawdź **[Premium VPS](/pl/premium-vps/)**.