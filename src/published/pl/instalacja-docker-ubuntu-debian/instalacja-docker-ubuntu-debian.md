---
image: /assets/images/blog/pl/instalacja-docker-ubuntu-debian/og-image.png
title: 'Jak zainstalować Docker na Ubuntu i Debian: Kompletny przewodnik serwera'
description: Kompletny przewodnik krok po kroku do instalacji najnowszego Docker Engine i Docker Compose na serwerach Ubuntu i Debian poprawnie używając oficjalnego repozytorium Docker.
date: '2026-03-25'
translationKey: install-docker-ubuntu-debian
category: Poradniki
tags:
  - docker
  - ubuntu
  - debian
  - docker engine
  - containers
  - linux
  - vps
  - server administration
  - docker compose
howto:
  name: Jak zainstalować Docker Engine na Ubuntu i Debian
  totalTime: PT10M
  yield: Serwer działający na najnowszej oficjalnej wersji Docker Engine i Docker Compose
  tool:
    - VPS lub dedykowany serwer z Ubuntu lub Debian
    - Klient SSH (np. terminal, PuTTY)
    - Konto użytkownika z uprawnieniami sudo
  steps:
    - name: Zaktualizuj serwer i zainstaluj pakiety wymagane
      text: Uruchom sudo apt update i zainstaluj curl, ca-certificates, i gnupg.
      url: step-1-update-system-and-install-dependencies
    - name: Dodaj oficjalny klucz GPG Docker
      text: Pobierz klucz szyfrowania Docker i zapisz go do /etc/apt/keyrings aby weryfikować pakiety.
      url: step-2-add-dockers-official-gpg-key
    - name: Skonfiguruj repozytorium Docker
      text: Dodaj oficjalne repozytorium do swojej listy źródeł apt i uruchom apt update.
      url: step-3-set-up-the-docker-repository
    - name: Zainstaluj Docker Engine
      text: Uruchom sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin.
      url: step-4-install-docker-engine-and-compose
    - name: Zweryfikuj instalację
      text: Uruchom sudo docker run hello-world aby zweryfikować że silnik pomyślnie uruchamia kontenery.
      url: step-5-verify-the-installation
status: published
locale: pl
author:
  name: VoxiHost Team
  link: https://voxihost.pl/
contributors:
  - danielmarszalkowski
---

Docker fundamentalnie zmienił sposób w jaki aplikacje są rozwijane i wdrażane. Zamiast walczyć z zależnościami ("działa na mojej maszynie ale nie na serwerze!"), Docker pakuje twoją aplikację i wszystko czego potrzebuje w odizolowany, przenośny **kontener** który działa idealnie wszędzie gdzie Docker jest zainstalowany.

Podczas gdy Ubuntu i Debian zawierają wersję Dockera w swoich domyślnych repozytoriach `apt`, jest ona prawie zawsze nieaktualna. Aby uzyskać najnowsze funkcje, poprawki bezpieczeństwa i zintegrowaną `docker-compose-plugin`, musisz zainstalować Docker Engine bezpośrednio z oficjalnego repozytorium Docker.

## Krok 1: Zaktualizuj system i zainstaluj zależności

Przed dodaniem nowego zewnętrznego repozytorium, zaktualizuj swój indeks pakietów i upewnij się że masz niezbędne pakiety podstawowe wymagane przez HTTPS.

{% image "/assets/images/blog/pl/instalacja-docker-ubuntu-debian/H1.png", "Uruchamianie sudo apt update i instalowanie ca-certificates curl gnupg wymagań dla Dockera na Ubuntu", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo apt update
sudo apt install ca-certificates curl gnupg -y
```

## Krok 2: Dodaj oficjalny klucz GPG Docker

Aby zapewnić że oprogramowanie które pobierasz nie zostało naruszone, `apt` używa kluczy GPG. Musisz pobrać oficjalny klucz GPG Docker i przechować go poprawnie w nowym ustandaryzowanym katalogu `/etc/apt/keyrings`.

Utwórz katalog (może już istnieć):

{% image "/assets/images/blog/pl/instalacja-docker-ubuntu-debian/H2.png", "Uruchamianie sudo install -m 0755 -d /etc/apt/keyrings aby utworzyć katalog pierścieni kluczy GPG dla Dockera na Ubuntu", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo install -m 0755 -d /etc/apt/keyrings
```

Pobierz i zapisz klucz:

{% image "/assets/images/blog/pl/instalacja-docker-ubuntu-debian/H3.png", "Pobieranie i dodawanie oficjalnego klucza GPG Docker do /etc/apt/keyrings na Ubuntu używając curl i gpg", "(max-width: 768px) 100vw, 800px" %}

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```
*(Jeśli jesteś na Debian, zmień `ubuntu` w URL na `debian`)*

Na koniec, dostosuj uprawnienia tak aby `apt` mogło czytać klucz podczas aktualizacji:

{% image "/assets/images/blog/pl/instalacja-docker-ubuntu-debian/H4.png", "Uruchamianie sudo chmod a+r aby ustawić uprawnienia odczytu na pliku pierścienia kluczy GPG Docker na Ubuntu", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

## Krok 3: Skonfiguruj repozytorium Docker

Teraz gdy system ufa pakietom przychodzącym z Docker, musisz powiedzieć `apt` gdzie ich znaleźć dodając repozytorium do swojej listy źródeł.

Skopiuj i wklej cały ten blok kodu do swojego terminala i naciśnij Enter. Automatycznie wykrywa architekturę twojego systemu (jak `amd64` lub `arm64`) i nazwę kodową i tworzy plik dla ciebie:

{% image "/assets/images/blog/pl/instalacja-docker-ubuntu-debian/H5.png", "Dodawanie oficjalnego repozytorium apt Docker CE do /etc/apt/sources.list.d na Ubuntu lub Debian", "(max-width: 768px) 100vw, 800px" %}

*(Dla Ubuntu)*

```bash
echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

*(Dla Debian)*
```bash
echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

Z nowym repozytorium dodanym, zaktualizuj swój indeks pakietów jeszcze raz tak aby `apt` widziało nowe pakiety Docker:

```bash
sudo apt update
```

## Krok 4: Zainstaluj Docker Engine i Compose

Na koniec możesz zainstalować najnowszą stabilną wersję Dockera czysto.

To polecenie instaluje podstawowy silnik Docker (`docker-ce`), klient wiersza poleceń (`docker-ce-cli`), runtime (`containerd.io`) i kluczowe wtyczki jak **Docker Compose V2** (`docker-compose-plugin`).

{% image "/assets/images/blog/pl/instalacja-docker-ubuntu-debian/H6.png", "Uruchamianie sudo apt install docker-ce docker-ce-cli containerd.io docker-compose-plugin na Ubuntu", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```

Demon Docker automatycznie zaczyna działać natychmiast po zakończeniu instalacji.

## Krok 5: Zweryfikuj instalację

Aby potwierdzić że cały silnik działa, możesz pobrać i uruchomić oficjalny obraz testowy Docker. Uruchamia on mały kontener, drukuje wiadomość potwierdzenia do twojego terminala i potem wyłącza się.

{% image "/assets/images/blog/pl/instalacja-docker-ubuntu-debian/H7.png", "Uruchamianie sudo docker run hello-world na Ubuntu aby zweryfikować że instalacja Docker Engine działa", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo docker run hello-world
```

Jeśli się powiedzie, zobaczysz długą wiadomość walidacji zaczynającą się od:
> *"Hello from Docker! This message shows that your installation appears to be working correctly."*

> **Docker i twoja zapora (UFW):** Domyślnie Docker modyfikuje reguły `iptables` twojego systemu bezpośrednio. Oznacza to że **Docker całkowicie omija reguły UFW**. Jeśli uruchamiasz kontener i mapujesz port do publicznego internetu (np., `-p 8080:80`), ten port będzie otwarty globalnie nawet jeśli UFW jest ustawione na "odmów". Zawsze bądź ostrożny przy eksponowaniu portów w produkcji.

## Krok 6 (Opcjonalny): Uruchom Docker bez sudo

Domyślnie polecenie `docker` może być uruchamiane tylko przez użytkownika root lub kogoś używającego `sudo`. Może to być irytujące jeśli zarządzasz wieloma kontenerami.

Jeśli stworzyłeś swojego własnego użytkownika niestandardowego z sudo (jak zalecane w naszym [przewodniku użytkownika](/pl/blog/jak-dodac-uzytkownika-sudo-ubuntu/)), możesz dodać tego użytkownika do grupy `docker` aby ominąć wymóg `sudo`.

```bash
sudo usermod -aG docker $USER
```

**Ostrzeżenie:** Bycie w grupie `docker` efektywnie nadaje uprawnienia poziomu root do systemu. Dodawaj tylko użytkowników których całkowicie ufasz.

Aby ta zmiana grupy weszła w życie, musisz całkowicie wylogować się z sesji SSH i zalogować się ponownie, lub uruchomić następujące polecenie aby aktywować logikę grupy w obecnej powłoce:
```bash
su - $USER
```

Teraz spróbuj uruchomić test bez `sudo`:
```bash
docker run hello-world
```

Jeśli to działa, gratulacje! Twój **VPS** jest w pełni wyposażony do wdrażania nieskończonej liczby wstępnie spakowanych aplikacji kontenerowych dostępnych na Docker Hub natywnie i bezpiecznie.

Dla niezawodnego środowiska wspierającego szybkie prototypowanie projektów z Docker, sprawdź naszą wysokowydajną linię przystępnych środowisk [Budget VPS](/pl/budget-vps/) już dziś.