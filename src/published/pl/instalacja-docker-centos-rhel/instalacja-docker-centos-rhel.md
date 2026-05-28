---
image: /assets/images/blog/instalacja-docker-centos-rhel/og-image.png
title: 'Jak zainstalować Docker na AlmaLinux, CentOS, Rocky Linux i Fedora: Kompletny przewodnik serwera'
description: Kompletny przewodnik krok po kroku do instalacji najnowszego oficjalnego Docker Engine i Docker Compose na serwerach AlmaLinux, CentOS Stream, Rocky Linux i Fedora.
date: '2026-03-25'
translationKey: install-docker-almalinux-centos-rocky
category: Poradniki
tags:
  - docker
  - almalinux
  - centos
  - rocky linux
  - fedora
  - docker engine
  - containers
  - linux
  - vps
  - server administration
  - docker compose
howto:
  name: Jak zainstalować Docker Engine na AlmaLinux, CentOS Stream, Rocky Linux i Fedora
  totalTime: PT10M
  yield: W pełni skonfigurowany serwer rodziny RHEL działający na najnowszym Docker Engine i wtyczce Docker Compose
  tool:
    - VPS lub dedykowany serwer z AlmaLinux, CentOS Stream, Rocky Linux lub Fedora
    - Klient SSH (np. terminal, PuTTY)
    - Konto użytkownika z uprawnieniami sudo
  steps:
    - name: Usuń stare konfliktujące pakiety
      text: Uruchom sudo dnf remove docker docker-client docker-client-latest docker-common docker-latest docker-latest-logrotate docker-logrotate docker-engine aby wyczyścić konflikty.
      url: step-1-remove-old-versions
    - name: Skonfiguruj repozytorium Docker
      text: Zainstaluj yum-utils i użyj yum-config-manager aby dodać oficjalne repozytorium Docker.
      url: step-2-set-up-the-docker-repository
    - name: Zainstaluj Docker Engine
      text: Uruchom sudo dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y.
      url: step-3-install-docker-engine
    - name: Uruchom i włącz usługę
      text: Uruchom sudo systemctl enable --now docker aby uruchomić demona.
      url: step-4-start-and-enable-docker
    - name: Zweryfikuj instalację
      text: Uruchom sudo docker run hello-world aby potwierdzić że wszystko działa.
      url: step-5-verify-the-installation
status: published
locale: pl
author:
  name: VoxiHost Team
  link: https://voxihost.pl/
contributors:
  - danielmarszalkowski
---

Docker zrewolucjonizował wdrożenia serwerowe umożliwiając izolowanie aplikacji w lekkie, przenośne, samowystarczalne jednostki zwane kontenerami. Niezależnie od tego jaki jest twój podstawowy system operacyjny, kontener Docker uruchamia się dokładnie tak samo.

Jednakże, wiele domyślnych repozytoriów (jak te wbudowane w RHEL, AlmaLinux, CentOS i Rocky Linux) często wskazuje na Podman zamiast Dockera, lub hostują poważnie nieaktualne wersje Docker Engine. Aby zagwarantować dostęp do najnowszych funkcji bezpieczeństwa i zintegrowanej `docker-compose-plugin`, musisz pobrać go bezpośrednio z oficjalnego źródła Docker.

## Krok 1: Usuń stare wersje

Przed zainstalowaniem oficjalnego silnika, musisz zweryfikować że żadne starsze, konfliktujące pakiety nie pozostają w systemie (nawet jeśli nigdy ich nie instalowałeś sam). Te pakiety zwykle krążą wokół nazw `docker` lub `docker-engine`.

Uruchom to polecenie aby wyczyścić planszę gładko:

{% image "/assets/images/blog/instalacja-docker-centos-rhel/H1.png", "Uruchamianie sudo dnf remove docker aby wyczyścić stare konfliktujące pakiety Docker na AlmaLinux lub Rocky Linux przed świeżą instalacją", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo dnf remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

Jest w porządku jeśli `dnf` zgłasza że żaden z tych pakietów nie jest zainstalowany.

## Krok 2: Skonfiguruj repozytorium Docker

Musisz powiedzieć swojemu menedżerowi pakietów (`dnf`) dokładnie gdzie znaleźć oficjalne wydania Docker. Docker dostarcza wygodne narzędzie konfiguracyjne natywnie wspierane przez systemy RHEL poprzez pakiet `yum-utils`.

Zainstaluj narzędzia:

{% image "/assets/images/blog/instalacja-docker-centos-rhel/H2.png", "Uruchamianie sudo dnf install -y yum-utils na AlmaLinux aby zainstalować narzędzie yum-config-manager potrzebne do dodania repozytorium Docker CE", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo dnf install -y yum-utils
```

Teraz użyj `yum-config-manager` aby bezpiecznie dodać oficjalne repozytorium Docker do źródeł systemowych:

{% image "/assets/images/blog/instalacja-docker-centos-rhel/H3.png", "Uruchamianie sudo yum-config-manager --add-repo aby dodać oficjalne repozytorium Docker CE do AlmaLinux lub Rocky Linux", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```
*(Nawet jeśli jesteś na AlmaLinux, Rocky Linux lub Fedorze, przekazanie ścieżki `/centos/` jest poprawne, ponieważ dzielą one absolutnie dokładnie tę samą architekturę binarną dla Enterprise Linux).*

## Krok 3: Zainstaluj Docker Engine

Z repozytorium bezpiecznie dodanym, twój system wie gdzie szukać. Możesz teraz zainstalować cały pakiet Docker.

To polecenie instaluje podstawowy silnik (`docker-ce`), interfejs wiersza poleceń (`docker-ce-cli`), runtime kontenera (`containerd.io`) i nowoczesne wtyczki jak **Docker Compose V2** (`docker-compose-plugin`).

{% image "/assets/images/blog/instalacja-docker-centos-rhel/H4.png", "Uruchamianie sudo dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin na AlmaLinux aby zainstalować Docker Engine i Compose", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```

## Krok 4: Uruchom i włącz Docker

W przeciwieństwie do Ubuntu, które automatycznie uruchamia usługi natychmiast po pobraniu ich, dystrybucje rodziny RHEL wolą abyś je celowo uruchamiał.

Musisz uruchomić demona Docker i włączyć go tak aby bezpiecznie budził się za każdym razem gdy serwer restartuje. Możesz zrobić oba w jednym poleceniu systemctl:

{% image "/assets/images/blog/instalacja-docker-centos-rhel/H5.png", "Uruchamianie sudo systemctl enable --now docker na AlmaLinux aby uruchomić usługę Docker i włączyć ją do automatycznego uruchamiania przy starcie", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo systemctl enable --now docker
```

Aby potwierdzić że usługa żyje, sprawdź status:

{% image "/assets/images/blog/instalacja-docker-centos-rhel/H6.png", "Uruchamianie sudo systemctl status docker na AlmaLinux aby potwierdzić że demon Docker jest aktywny i działa poprawnie", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo systemctl status docker
```
Szukaj jasnozielonego tekstu `"active (running)"`.

## Krok 5: Zweryfikuj instalację

Aby bezsprzecznie udowodnić że Docker może pomyślnie pobierać obrazy z internetu i uruchamiać je w działające kontenery, użyj standardowego ładunku testowego:

{% image "/assets/images/blog/instalacja-docker-centos-rhel/H7.png", "Uruchamianie sudo docker run hello-world na AlmaLinux lub Rocky Linux aby zweryfikować że Docker Engine jest zainstalowany i działa poprawnie", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo docker run hello-world
```

Jeśli twoja konfiguracja jest poprawna, kontener zostanie pobrany, uruchomi swój kod i wyświetli duży blok potwierdzenia zaczynający się od:
> *"Hello from Docker! This message shows that your installation appears to be working correctly."*

> **Docker i twoja zapora:** Docker zarządza swoimi własnymi regułami sieciowymi bezpośrednio przez `iptables`. Podczas gdy systemy RedHat używają `firewalld`, który jest generalnie bardziej zintegrowany z Docker niż UFW, wciąż najlepszą praktyką jest ostrożność przy eksponowaniu portów poprzez flagę `-p` lub `--publish`. Zawsze weryfikuj otwarte porty za pomocą `sudo firewall-cmd --list-all`.

## Krok 6 (Opcjonalny): Uruchom Docker bez sudo

Prawdopodobnie zauważyłeś że musiałeś wpisać `sudo` aby uruchomić skrypt testowy. Domyślnie demon Docker wiąże się z gniazdem Unix zamiast portem TCP, a to gniazdo jest własnością użytkownika `root`.

Jeśli stworzyłeś swoje własne konto użytkownika (jak opisano w naszym [Przewodniku Zarządzania Użytkownikami](/pl/blog/jak-dodac-uzytkownika-sudo-centos/)), wpisywanie `sudo` setki razy dziennie może być męczące. Możesz nadać swojemu użytkownikowi natywne prawa Docker dodając ich do grupy `docker`.

```bash
sudo usermod -aG docker $USER
```

**Ostrzeżenie:** Grupa `docker` nadaje uprawnienia które są funkcjonalnie równoważne dostępowi root. Dodawaj tylko bardzo zaufanych użytkowników do tej grupy.

Aby zmusić system do uznania twojego nowego statusu grupy bez potrzeby całkowitego wylogowania:

```bash
su - $USER
```

Teraz ponownie uruchom test bez prefiksu sudo:
```bash
docker run hello-world
```

Jeśli to działa, gratulacje! Twój **VPS** jest w pełni wyposażony do wdrażania nieskończonej liczby wstępnie spakowanych aplikacji kontenerowych dostępnych na Docker Hub natywnie i bezpiecznie.

Dla niezawodnego środowiska wspierającego szybkie prototypowanie projektów z Docker, sprawdź naszą wysokowydajną linię przystępnych środowisk [Budget VPS](/pl/budget-vps/) już dziś.