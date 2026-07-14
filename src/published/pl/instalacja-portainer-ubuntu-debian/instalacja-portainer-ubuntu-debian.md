---
image: /assets/images/blog/pl/instalacja-portainer-ubuntu-debian/og-image.png
title: "Jak zainstalować Portainer CE na Ubuntu i Debian"
description: "Zainstaluj Portainer CE na VPS z Ubuntu lub Debian i zarządzaj kontenerami Docker z poziomu przeglądarki bez użycia terminala."
date: '2026-07-14'
translationKey: setup-portainer-ubuntu-debian
locale: pl
category: Poradniki
tags:
  - docker
  - portainer
  - vps
  - linux
  - ubuntu
  - debian
  - kontenery
status: published
author:
  name: VoxiHost Team
  link: https://voxihost.pl/
contributors:
  - sl0ikkk
howto:
  name: Jak zainstalować Portainer CE na Ubuntu i Debianie
  totalTime: PT10M
  yield: Działający panel Portainer CE gotowy do zarządzania kontenerami Docker przez przeglądarkę
  tool:
    - Serwer VPS z systemem Ubuntu lub Debian
    - Klient SSH (np. terminal, PuTTY)
    - Konto użytkownika z uprawnieniami sudo
  steps:
    - name: Połączenie z serwerem i instalacja Dockera
      text: Połącz się przez SSH i uruchom oficjalny skrypt instalacyjny Dockera.
      url: krok-1-polaczenie-z-serwerem-i-instalacja-dockera
    - name: Przygotowanie miejsca dla Portainera
      text: Utwórz trwały wolumen Docker do przechowywania konfiguracji Portainera.
      url: krok-2-przygotowanie-miejsca-dla-portainera
    - name: Uruchomienie Portainera
      text: Pobierz i uruchom kontener Portainer CE z wymaganymi portami i wolumenem.
      url: krok-3-uruchomienie-portainera
    - name: Pierwsze logowanie w przeglądarce
      text: Otwórz interfejs webowy Portainera i utwórz konto administratora.
      url: krok-4-pierwsze-logowanie-w-przegladarce
    - name: Zarządzanie kontenerami
      text: Zarządzaj kontenerami Docker bez użycia terminala przez panel Portainera.
      url: krok-5-zarzadzanie-kontenerami
faq:
  - question: "Czym różni się Portainer CE od Portainer Business Edition?"
    answer: "Portainer CE (Community Edition) jest darmowy i open-source — obejmuje wszystkie podstawowe funkcje zarządzania Dockerem. Portainer Business Edition (BE) dodaje funkcje korporacyjne, takie jak kontrola dostępu oparta na rolach (RBAC), integracja OAuth/LDAP i rozszerzony support."
  - question: "Czy bezpiecznie jest wystawiać port 9443 na internet?"
    answer: "Portainer używa HTTPS na porcie 9443, ale publiczne wystawianie go bez dodatkowej ochrony nie jest zalecane. Rozważ umieszczenie go za odwrotnym proxy (np. <code>Nginx Proxy Manager</code>) z uwierzytelnianiem lub ograniczenie dostępu do zaufanych adresów IP przez zaporę sieciową."
  - question: "Jak zaktualizować Portainer do nowszej wersji?"
    answer: "Zatrzymaj i usuń istniejący kontener, a następnie pobierz najnowszy obraz i uruchom ponownie tę samą komendę <code>docker run</code>. Twoje dane są zachowane w wolumenie <code>portainer_data</code>: <code>docker stop portainer && docker rm portainer</code>, a następnie ponowne uruchomienie oryginalnej komendy instalacyjnej."
  - question: "Czy Portainer obsługuje Docker Compose?"
    answer: "Tak. Portainer nazywa je <strong>Stosami</strong> (Stacks). Możesz wdrażać pliki Docker Compose i zarządzać nimi bezpośrednio z interfejsu webowego Portainera bez użycia terminala."
  - question: "Co zrobić, jeśli zapomnę hasła administratora Portainera?"
    answer: "Musisz zresetować hasło z poziomu wiersza poleceń. Zatrzymaj kontener, uruchom <code>docker run --rm -v portainer_data:/data portainer/helper-reset-password</code>, skopiuj wygenerowane hasło, a następnie uruchom ponownie Portainer komendą <code>docker start portainer</code>."
---

## Wstęp

Docker to wspaniałe narzędzie, ale wpisywanie komend w czarnej konsoli może być przytłaczające dla początkujących. Z pomocą przychodzi **Portainer** — panel z graficznym interfejsem (GUI), który pozwala zarządzać wszystkim z poziomu przeglądarki internetowej.

W tym poradniku pokażemy Ci, krok po kroku, jak zainstalować Portainera na Twoim serwerze w zaledwie kilka minut. Nie musisz być ekspertem od Linuxa — wystarczy, że będziesz kopiować i wklejać poniższe komendy!

> **Wymagania wstępne:** Będziesz potrzebować serwera [Premium VPS](/pl/premium-vps/) z systemem Ubuntu lub Debian oraz klienta SSH (np. PuTTY lub systemowy Terminal). Jeśli jeszcze nie masz serwera, <span class="text-white">Voxi</span><span class="text-amber-300">Host</span> oferuje [Premium VPS](/pl/premium-vps/) i [Budget VPS](/pl/budget-vps/) z preinstalowanym Ubuntu i Debianem.

---

## Krok 1: Połączenie z serwerem i instalacja Dockera

Najpierw połącz się ze swoim serwerem przez SSH. Następnie upewnijmy się, że na serwerze zainstalowany jest Docker. Jeśli jeszcze go nie masz, wystarczy wkleić poniższą komendę, która zrobi wszystko za Ciebie:

{% image "/assets/images/blog/pl/instalacja-portainer-ubuntu-debian/H1.png", "Terminal podczas instalacji środowiska Docker na Ubuntu", "(max-width: 768px) 100vw, 800px" %}

```bash
curl -fsSL https://get.docker.com -o get-docker.sh && sudo sh get-docker.sh
```

Zaczekaj chwilę, aż instalacja się zakończy.

---

## Krok 2: Przygotowanie miejsca dla Portainera

Portainer potrzebuje bezpiecznego miejsca na dysku, aby zapisywać Twoje hasła i konfiguracje. Wklej poniższą komendę w konsoli i wciśnij ENTER, aby utworzyć tzw. "wolumen":

{% image "/assets/images/blog/pl/instalacja-portainer-ubuntu-debian/H2.png", "Tworzenie wolumenu Docker dla Portainera w terminalu", "(max-width: 768px) 100vw, 800px" %}

```bash
docker volume create portainer_data
```

---

## Krok 3: Uruchomienie Portainera

Teraz uruchomimy samego Portainera. Poniższa komenda pobierze najnowszą wersję programu i uruchomi ją w tle. Wklej ją w całości do konsoli i wciśnij ENTER:

{% image "/assets/images/blog/pl/instalacja-portainer-ubuntu-debian/H3.png", "Uruchamianie kontenera Portainer CE z wymaganymi portami i wolumenem", "(max-width: 768px) 100vw, 800px" %}

```bash
docker run -d -p 8000:8000 -p 9443:9443 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:latest
```

I to tyle! Instalacja po stronie serwera jest zakończona.

---

## Krok 4: Pierwsze logowanie w przeglądarce

Możesz już zamknąć konsolę. Teraz otwórz swoją ulubioną przeglądarkę internetową (np. Chrome, Firefox) i wpisz w pasku adresu:

`https://ADRES_IP_TWOJEGO_SERWERA:9443`
*(Zastąp "ADRES_IP_TWOJEGO_SERWERA" rzeczywistym adresem IP swojego VPS, np. https://192.168.1.50:9443)*

> **Wskazówka:** Jeśli po wejściu na stronę widzisz błąd `Client sent an HTTP request to an HTTPS server.`, oznacza to, że Twoja przeglądarka spróbowała połączyć się przez zwykłe HTTP. Upewnij się, że adres na samym początku ma wpisane ręcznie **https://**.

> **Ważne:** Przeglądarka może wyświetlić ostrzeżenie o "Niezaufanym certyfikacie" (ang. *Your connection is not private*). To całkowicie normalne, ponieważ serwer używa domyślnego certyfikatu własnego podpisu. Kliknij **"Zaawansowane"**, a następnie **"Kontynuuj / Przejdź na stronę"**.

Na ekranie zobaczysz kreator początkowy. Wymyśl dla siebie bezpieczne hasło, wpisz je dwukrotnie i kliknij przycisk tworzenia użytkownika (Create user).

> **Ważne (Limit 5 minut!):** Portainer dba o bezpieczeństwo. Jeśli od momentu uruchomienia (Krok 3) do ustalenia hasła minie ponad 5 minut, instalacja zostanie zablokowana (komunikat *Your Portainer instance timed out*). Aby ją odblokować, wpisz w konsoli `docker restart portainer` i odśwież stronę. Zostaniesz wtedy poproszony o **Setup Token**. Aby go uzyskać, wpisz w konsoli `docker logs portainer`. W logach znajdziesz napis *Use the following token to setup the administrator user:* a pod nim swój token. Skopiuj go i wklej na stronie.

{% image "/assets/images/blog/pl/instalacja-portainer-ubuntu-debian/H4.png", "Ekran tworzenia konta administratora w Portainer w przeglądarce", "(max-width: 768px) 100vw, 800px" %}

---

## Krok 5: Zarządzanie kontenerami

Po zalogowaniu kliknij przycisk **"Get Started"**, a na kolejnym ekranie wybierz środowisko z ikonką wieloryba podpisane jako **"Local"** (czyli serwer, na którym właśnie jesteśmy).

Gotowe! Teraz widzisz pełny panel kontrolny. Od teraz nie musisz używać komend w czarnej konsoli — możesz zarządzać swoimi aplikacjami klikając myszką.

{% image "/assets/images/blog/pl/instalacja-portainer-ubuntu-debian/H5.png", "Panel główny Portainer pokazujący listę aktywnych kontenerów Docker", "(max-width: 768px) 100vw, 800px" %}

---

## Podsumowanie

Portainer to niesamowite narzędzie, które sprawia, że technologia Docker jest dostępna nawet dla osób bez dużego doświadczenia z Linuxem. Jeśli szukasz serwera do własnych projektów lub nauki, <span class="text-white">Voxi</span><span class="text-amber-300">Host</span> ma w ofercie [Premium VPS](/pl/premium-vps/) oraz [Budget VPS](/pl/budget-vps/) z systemem Ubuntu i Debian gotowym do działania.