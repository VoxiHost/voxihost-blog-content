---
image: /assets/images/blog/serwer-minecraft-1-8-8-centos-rhel/og-image.png
title: Jak postawić klasyczny serwer Minecraft 1.8.8 (Java 8) na AlmaLinux, CentOS, Rocky Linux i Fedorze
description: Poradnik instalacji serwera Minecraft 1.8.8, idealny do nostalgicznego PvP i klasycznej rozgrywki Vanilla z użyciem Java 8 na dystrybucjach RHEL.
date: '2026-04-23'
translationKey: minecraft-vanilla-java-1-8-8-server-setup-almalinux-centos-rocky-fedora
locale: pl
category: Poradniki
tags:
  - minecraft
  - vanilla
  - java 8
  - konfiguracja serwera
  - almalinux
  - rocky linux
  - centos
  - legacy
howto:
  name: Jak postawić klasyczny serwer Minecraft (1.8.8/1.16.5) na AlmaLinux, CentOS, Rocky Linux i Fedorze
  totalTime: PT10M
  yield: W pełni działający klasyczny serwer Minecraft w środowisku Java 8
  tool:
    - AlmaLinux/Rocky VPS
    - Java 8 JRE
    - Klient SSH
  steps:
    - name: Instalacja Java 8
      text: Zainstaluj klasyczne środowisko Java wymagane dla wersji od 1.8.8 do 1.16.5.
      url: '#krok-1-instalacja-java-8'
    - name: Tworzenie użytkownika gry
      text: Skonfiguruj ograniczonego użytkownika 'minecraft' ze względów bezpieczeństwa za pomocą 'useradd'.
      url: '#krok-2-tworzenie-dedykowanego-uzytkownika'
    - name: Pobranie klasycznego pliku JAR
      text: Pobierz oficjalny plik binarny 1.8.8 bezpiecznie do folderu domowego użytkownika.
      url: '#krok-3-pobranie-pliku-jar-1-8-8'
    - name: Akceptacja regulaminu
      text: Edytuj plik 'eula.txt', ustawiając wartość true, aby zaakceptować warunki Mojang.
      url: '#krok-4-akceptacja-eula'
    - name: Wdrożenie skryptu
      text: Utwórz plik 'start.sh' do uruchamiania serwera z minimum 2GB RAM.
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

Klasyczne wersje Minecrafta z ery **1.7.10 do 1.16.5** wymagają **Java 8** dla legendarnej stabilności. Ten poradnik obejmuje cały klasyczny zakres. Wymagania dla nowoczesnych wersji znajdziesz w naszym [Poradniku kompatybilności serwerów Minecraft Java](/pl/blog/jak-zalozyc-serwer-minecraft-centos-rhel/).

> Starsze wersje często zawierają znane luki bezpieczeństwa w bibliotekach zewnętrznych. NIGDY nie uruchamiaj tych wersji jako root; zawsze używaj dedykowanego konta z ograniczonymi uprawnieniami.

### Obsługiwane wersje
Ten poradnik dla Java 8 jest w pełni kompatybilny z:
* **Era 1.16:** 1.16.5, 1.16.4, 1.16.3, 1.16.2, 1.16.1, 1.16
* **1.13 – 1.15:** 1.15.2, 1.15.1, 1.15, 1.14.4, 1.13.2
* **Klasyczne (1.7 – 1.12):** 1.12.2, 1.11.2, 1.10.2, 1.9.4, 1.8.9, 1.8.8, 1.7.10

Dokładny link do pobrania swojej wersji znajdziesz w naszym [Archiwum linków do serwerów Minecraft Vanilla](/pl/blog/serwer-minecraft-linki-do-pobrania/).

## Wymagania wstępne

* VPS z systemem **AlmaLinux, Rocky Linux lub CentOS** (dostępny w ramach [Premium VPS](/pl/premium-vps/)).
* Dostęp root lub sudo przez SSH (do instalacji Javy).
* **Ograniczony użytkownik inny niż root** do bezpiecznego uruchamiania serwera.

## Krok 1: Instalacja Java 8

Najpierw wykonaj pełną [aktualizację systemu](/pl/blog/jak-zaktualizowac-centos-rhel/), aby upewnić się, że listy pakietów są aktualne.

Na standardowych serwerach klasy enterprise (AlmaLinux 8 i 9, Rocky Linux 8 i 9) Java 8 jest dostępna natywnie:

{% image "/assets/images/blog/serwer-minecraft-1-8-8-centos-rhel/H1.png", "Terminal przedstawiający instalację OpenJDK 8 na serwerze Linux", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo dnf check-update
sudo dnf install java-1.8.0-openjdk-headless wget -y
```

> **Rozwiązywanie problemów (EL10 i nowsza Fedora):** Jeśli pojawi się błąd `No match for argument`, oznacza to, że Twoja dystrybucja Linuksa jest zbyt nowa i oficjalnie wycofała Java 8 ze swoich domyślnych repozytoriów. Możesz łatwo zainstalować wysoce zoptymalizowaną dystrybucję **Amazon Corretto 8**, uruchamiając:
> ```bash
> sudo rpm --import https://yum.corretto.aws/corretto.key
> sudo curl -L -o /etc/yum.repos.d/corretto.repo https://yum.corretto.aws/corretto.repo
> sudo dnf install java-1.8.0-amazon-corretto-devel wget -y
> ```

## Krok 2: Tworzenie dedykowanego użytkownika

Dla bezpieczeństwa nigdy nie uruchamiaj serwera jako root. Nawet starsze wersje powinny być izolowane. Jeśli dopiero zaczynasz z uprawnieniami w Linuksie, zapoznaj się z naszym poradnikiem [Tworzenia i zarządzania użytkownikami na AlmaLinux/Rocky](/pl/blog/jak-dodac-uzytkownika-sudo-centos/).

{% image "/assets/images/blog/serwer-minecraft-1-8-8-centos-rhel/H2.png", "Tworzenie dedykowanego użytkownika 'minecraft' do bezpiecznego hostowania starszego serwera", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo useradd -m -r -s /bin/bash minecraft
sudo su - minecraft
mkdir server && cd server
```

## Krok 3: Pobranie pliku JAR 1.8.8

Szukasz innej klasycznej wersji? Bezpośrednie linki do pobrania od Mojang dla wszystkich historycznych wydań znajdziesz w naszym [Archiwum linków do serwerów Minecraft](/pl/blog/serwer-minecraft-linki-do-pobrania/).

{% image "/assets/images/blog/serwer-minecraft-1-8-8-centos-rhel/H3.png", "Pobieranie pliku Minecraft 1.8.8 server.jar za pomocą wget", "(max-width: 768px) 100vw, 800px" %}

```bash
wget https://launcher.mojang.com/v1/objects/5fafba3f58c40dc51b5c3ca72a98f62dfdae1db7/server.jar
```

## Krok 4: Akceptacja EULA

{% image "/assets/images/blog/serwer-minecraft-1-8-8-centos-rhel/H4.png", "Pierwsze uruchomienie pliku JAR 1.8.8 w celu wygenerowania plików konfiguracyjnych i akceptacji EULA", "(max-width: 768px) 100vw, 800px" %}

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

Wersja 1.8.8 jest znacznie lżejsza od nowoczesnych, dla małej grupy graczy często wystarczy 2GB RAM.

{% image "/assets/images/blog/serwer-minecraft-1-8-8-centos-rhel/H5.png", "Używanie edytora nano do tworzenia i konfigurowania skryptu startowego start.sh", "(max-width: 768px) 100vw, 800px" %}

```bash
nano start.sh
```

W edytorze wklej:
```bash
#!/bin/bash
java -Xmx2G -Xms2G -jar server.jar nogui
```

{% image "/assets/images/blog/serwer-minecraft-1-8-8-centos-rhel/H6.png", "Nadawanie uprawnień do wykonywania skryptowi start.sh", "(max-width: 768px) 100vw, 800px" %}

Nadaj uprawnienia do wykonywania:
```bash
chmod +x start.sh
```

## Krok 6: Pierwsze uruchomienie i konfiguracja administratora

Przed skonfigurowaniem usługi działającej w tle powinieneś uruchomić serwer ręcznie przynajmniej raz, aby nadać sobie uprawnienia administratora (**OP**).

{% image "/assets/images/blog/serwer-minecraft-1-8-8-centos-rhel/H7.png", "Ręczne uruchamianie serwera Minecraft 1.8.8 w celu uzyskania dostępu do konsoli", "(max-width: 768px) 100vw, 800px" %}

**1. Ręczne uruchomienie serwera**
Uruchom właśnie utworzony skrypt startowy:
```bash
./start.sh
```

{% image "/assets/images/blog/serwer-minecraft-1-8-8-centos-rhel/H8.png", "Nadawanie sobie uprawnień OP przez konsolę serwera", "(max-width: 768px) 100vw, 800px" %}

**2. Nadanie uprawnień administratora (OP)**
Gdy serwer zakończy ładowanie (zobaczysz komunikat „Done!"), wpisz bezpośrednio w konsoli:
```text
op twoja_nazwa_gracza_minecraft
```

{% image "/assets/images/blog/serwer-minecraft-1-8-8-centos-rhel/H9.png", "Bezpieczne wyłączanie serwera Minecraft poleceniem stop", "(max-width: 768px) 100vw, 800px" %}

**3. Zatrzymanie serwera**
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

{% image "/assets/images/blog/serwer-minecraft-1-8-8-centos-rhel/H10.png", "Tworzenie pliku usługi minecraft.service dla systemd", "(max-width: 768px) 100vw, 800px" %}

Utwórz plik usługi:
```bash
sudo nano /etc/systemd/system/minecraft.service
```

Wklej poniższą konfigurację:
```ini
[Unit]
Description=VoxiHost Minecraft 1.8.8 Server
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

{% image "/assets/images/blog/serwer-minecraft-1-8-8-centos-rhel/H11.png", "Włączanie i uruchamianie usługi minecraft w systemd", "(max-width: 768px) 100vw, 800px" %}

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

Osiągnij wysokie TPS na naszych planach **[Budget VPS](/pl/budget-vps/)**!