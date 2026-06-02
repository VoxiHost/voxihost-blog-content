---
image: /assets/images/blog/pl/serwer-minecraft-1-19-centos-rhel/og-image.png
title: Jak postawić serwer Minecraft Vanilla 1.19.2 (Java 17) na AlmaLinux, CentOS, Rocky Linux i Fedorze
description: Szczegółowy poradnik konfiguracji serwera Minecraft 1.19.2 Vanilla na AlmaLinux, CentOS lub Rocky Linux z użyciem środowiska Java 17.
date: '2026-04-23'
translationKey: minecraft-vanilla-java-1-19-server-setup-almalinux-centos-rocky-fedora
locale: pl
category: Poradniki
tags:
  - minecraft
  - vanilla
  - java 17
  - konfiguracja serwera
  - almalinux
  - rocky linux
  - centos
howto:
  name: Jak postawić serwer Minecraft Vanilla 1.19 na AlmaLinux, CentOS, Rocky Linux i Fedorze
  totalTime: PT10M
  yield: W pełni działający serwer z ery Minecraft 1.19 w środowisku Java 17
  tool:
    - AlmaLinux/Rocky VPS
    - Java 17 JRE
    - Klient SSH
  steps:
    - name: Instalacja Java 17
      text: Zainstaluj wymagany pakiet LTS Java 17 za pomocą 'sudo dnf install java-17-openjdk-headless'.
      url: krok-1-instalacja-java-17
    - name: Tworzenie użytkownika gry
      text: Skonfiguruj ograniczonego użytkownika 'minecraft' ze względów bezpieczeństwa za pomocą 'useradd'.
      url: krok-2-tworzenie-dedykowanego-uzytkownika
    - name: Pobranie pliku JAR 1.19.2
      text: Pobierz oficjalny plik Vanilla 1.19.2 server.jar od Mojang.
      url: krok-3-pobranie-minecraft-1-19-2
    - name: Akceptacja EULA
      text: Zainicjuj serwer i zaakceptuj EULA, edytując plik 'eula.txt'.
      url: krok-4-akceptacja-eula
    - name: Tworzenie skryptu startowego
      text: Utwórz skrypt 'start.sh' z optymalizowanymi flagami RAM od Aikara.
      url: krok-5-tworzenie-skryptu-startowego
    - name: Pierwsze uruchomienie i OP
      text: Uruchom serwer ręcznie, aby nadać sobie uprawnienia administratora.
      url: krok-6-pierwsze-uruchomienie-i-konfiguracja-administratora
    - name: Profesjonalne uruchamianie (Systemd)
      text: Skonfiguruj usługę systemd, aby serwer uruchamiał się automatycznie po restarcie.
      url: krok-7-konfiguracja-uslugi-systemd
status: published
author:
  name: VoxiHost Team
  link: https://voxihost.pl/
contributors:
  - danielmarszalkowski
---

Era Minecrafta od 1.18 do 1.20.4 opiera się na **Java 17**. Ten poradnik obejmuje cały cykl życia Java 17. Informacje o innych wersjach znajdziesz w naszym [Poradniku kompatybilności serwerów Minecraft Java](/pl/blog/jak-zalozyc-serwer-minecraft-centos-rhel/).

> **Nie uruchamiaj jako root:** Zawsze hostuj serwer Minecraft z poziomu ograniczonego konta użytkownika, aby chronić pliki VPS przed potencjalnymi exploitami.

### Wymagania wstępne

* VPS z systemem **AlmaLinux, Rocky Linux lub CentOS** (dostępny w ramach [Premium VPS](/pl/premium-vps/)).
* Dostęp sudo do wstępnej instalacji Javy.
* Szybki dysk SSD/NVMe (wszystkie węzły <span class="text-white">Voxi</span><span class="text-amber-300">Host</span> używają NVMe).

### Obsługiwane wersje
Ten poradnik dla Java 17 jest w pełni kompatybilny z:
* **Era 1.20:** 1.20.4, 1.20.3, 1.20.2, 1.20.1, 1.20
* **Era 1.19:** 1.19.4, 1.19.3, 1.19.2, 1.19.1, 1.19
* **Era 1.18:** 1.18.2, 1.18.1, 1.18

Potrzebujesz innej wersji? Bezpośredni link znajdziesz w naszym [Archiwum linków do serwerów Minecraft Vanilla](/pl/blog/serwer-minecraft-linki-do-pobrania/).

## Krok 1: Instalacja Java 17

Zadbaj o bezpieczeństwo systemu, wykonując najpierw pełną [aktualizację systemu](/pl/blog/jak-zaktualizowac-centos-rhel/). Następnie zainstaluj wymagany pakiet LTS Java 17:

{% image "/assets/images/blog/pl/serwer-minecraft-1-19-centos-rhel/H1.png", "Terminal przedstawiający instalację OpenJDK 17 na systemie Linux", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo dnf check-update
sudo dnf install java-17-openjdk-headless wget -y
```

## Krok 2: Tworzenie dedykowanego użytkownika

Dla bezpieczeństwa nigdy nie uruchamiaj serwera jako root. Jeśli dopiero zaczynasz z uprawnieniami w Linuksie, zapoznaj się z naszym poradnikiem [Tworzenia i zarządzania użytkownikami na AlmaLinux/Rocky](/pl/blog/jak-dodac-uzytkownika-sudo-centos/).

{% image "/assets/images/blog/pl/serwer-minecraft-1-19-centos-rhel/H2.png", "Tworzenie dedykowanego użytkownika 'minecraft' do bezpiecznego hostowania serwera 1.19", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo useradd -m -r -s /bin/bash minecraft
sudo su - minecraft
mkdir server && cd server
```

## Krok 3: Pobranie Minecraft 1.19.2

Szukasz innej wersji? Bezpośrednie linki do pobrania od Mojang dla wszystkich wydań znajdziesz w naszym [Archiwum linków do serwerów Minecraft](/pl/blog/serwer-minecraft-linki-do-pobrania/).

{% image "/assets/images/blog/pl/serwer-minecraft-1-19-centos-rhel/H3.png", "Pobieranie pliku Minecraft 1.19.2 server.jar za pomocą wget", "(max-width: 768px) 100vw, 800px" %}

```bash
wget https://piston-data.mojang.com/v1/objects/f69c284232d7c7580bd89a5a4931c3581eae1378/server.jar
```

## Krok 4: Akceptacja EULA

{% image "/assets/images/blog/pl/serwer-minecraft-1-19-centos-rhel/H4.png", "Pierwsze uruchomienie pliku JAR 1.19.2 w celu wygenerowania plików konfiguracyjnych i akceptacji EULA", "(max-width: 768px) 100vw, 800px" %}

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

{% image "/assets/images/blog/pl/serwer-minecraft-1-19-centos-rhel/H5.png", "Używanie edytora nano do tworzenia i konfigurowania skryptu startowego start.sh", "(max-width: 768px) 100vw, 800px" %}
```bash
nano start.sh
```

W edytorze wklej:
```bash
#!/bin/bash
java -Xmx6G -Xms6G -XX:+UseG1GC -XX:+ParallelRefProcEnabled -XX:MaxGCPauseMillis=200 -XX:+UnlockExperimentalVMOptions -XX:+DisableExplicitGC -XX:+AlwaysPreTouch -XX:G1NewSizePercent=30 -XX:G1MaxNewSizePercent=40 -XX:G1HeapRegionSize=8M -XX:G1ReservePercent=20 -XX:G1HeapWastePercent=5 -XX:G1MixedGCCountTarget=4 -XX:InitiatingHeapOccupancyPercent=15 -XX:G1MixedGCLiveThresholdPercent=90 -XX:G1RSetUpdatingPauseTimePercent=5 -XX:SurvivorRatio=32 -XX:+PerfDisableSharedMem -XX:MaxTenuringThreshold=1 -Dusing.aikars.flags=https://mcflags.emc.gs -Daikar.for.v1.20=false -jar server.jar nogui
```

Nadaj uprawnienia do wykonywania:

{% image "/assets/images/blog/pl/serwer-minecraft-1-19-centos-rhel/H6.png", "Nadawanie uprawnień do wykonywania skryptowi start.sh", "(max-width: 768px) 100vw, 800px" %}
```bash
chmod +x start.sh
```

## Krok 6: Pierwsze uruchomienie i konfiguracja administratora

Przed skonfigurowaniem usługi działającej w tle powinieneś uruchomić serwer ręcznie przynajmniej raz, aby nadać sobie uprawnienia administratora (**OP**).

**1. Ręczne uruchomienie serwera**

{% image "/assets/images/blog/pl/serwer-minecraft-1-19-centos-rhel/H7.png", "Ręczne uruchamianie serwera Minecraft 1.19.2 w celu uzyskania dostępu do konsoli", "(max-width: 768px) 100vw, 800px" %}
Uruchom właśnie utworzony skrypt startowy:
```bash
./start.sh
```

**2. Nadanie uprawnień administratora (OP)**

{% image "/assets/images/blog/pl/serwer-minecraft-1-19-centos-rhel/H8.png", "Nadawanie uprawnień OP przez konsolę serwera", "(max-width: 768px) 100vw, 800px" %}
Gdy serwer zakończy ładowanie (zobaczysz komunikat „Done!"), wpisz bezpośrednio w konsoli:
```text
op twoja_nazwa_gracza_minecraft
```

**3. Zatrzymanie serwera**

{% image "/assets/images/blog/pl/serwer-minecraft-1-19-centos-rhel/H9.png", "Bezpieczne wyłączanie serwera Minecraft 1.19.2", "(max-width: 768px) 100vw, 800px" %}
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

Utwórz plik usługi:

{% image "/assets/images/blog/pl/serwer-minecraft-1-19-centos-rhel/H10.png", "Tworzenie pliku usługi minecraft.service dla systemd", "(max-width: 768px) 100vw, 800px" %}
```bash
sudo nano /etc/systemd/system/minecraft.service
```

Wklej poniższą konfigurację:
```ini
[Unit]
Description=VoxiHost Minecraft 1.19 Server
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

Włącz i uruchom serwer:

{% image "/assets/images/blog/pl/serwer-minecraft-1-19-centos-rhel/H11.png", "Włączanie i uruchamianie usługi minecraft w systemd", "(max-width: 768px) 100vw, 800px" %}
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

Twój serwer jest gotowy! Dla hostingu z niskim pingiem sprawdź nasze opcje [<span class="text-white">Voxi</span><span class="text-amber-300">Host</span> Budget VPS](/pl/budget-vps/).