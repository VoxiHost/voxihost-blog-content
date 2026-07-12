---
image: /assets/images/blog/pl/instalacja-portainer-ubuntu-debian/og-image.png
title: "Zarządzanie Dockerem przez przeglądarkę – instalacja Portainer na VPS (Ubuntu/Debian)"
description: "Dowiedz się, jak zainstalować Portainer CE na serwerze Ubuntu lub Debian, aby łatwo zarządzać kontenerami Dockera z poziomu graficznego interfejsu (GUI) w przeglądarce."
date: 2026-07-12
translationKey: "setup-portainer-ubuntu-debian"
locale: pl
category: "Poradniki"
tags: ["docker", "portainer", "vps", "linux", "ubuntu", "debian", "kontenery"]
author:
  name: VoxiHost Team
  link: https://voxihost.pl/
contributors:
  - sl0ikkk
howto:
  name: "Jak zainstalować Portainer na VPS"
  totalTime: "PT10M"
  yield: "Działający panel Portainer gotowy do zarządzania kontenerami"
  tool:
    - "Serwer VPS z systemem Ubuntu lub Debian"
    - "Klient SSH"
  steps:
    - name: "Krok 1"
      text: "Połączenie z serwerem i instalacja Dockera"
      url: "krok-1-polaczenie-z-serwerem-i-instalacja-dockera"
    - name: "Krok 2"
      text: "Przygotowanie wolumenu"
      url: "krok-2-przygotowanie-miejsca-dla-portainera"
    - name: "Krok 3"
      text: "Uruchomienie Portainera"
      url: "krok-3-uruchomienie-portainera"
    - name: "Krok 4"
      text: "Pierwsze logowanie"
      url: "krok-4-pierwsze-logowanie-w-przegladarce"
    - name: "Krok 5"
      text: "Zarządzanie"
      url: "krok-5-zarzadzanie-kontenerami"
---

## Wstęp

Docker to wspaniałe narzędzie, ale wpisywanie komend w czarnej konsoli może być przytłaczające dla początkujących. Z pomocą przychodzi **Portainer** – panel z graficznym interfejsem (GUI), który pozwala "wyklikać" wszystko z poziomu przeglądarki internetowej.

W tym poradniku pokażemy Ci, krok po kroku, jak zainstalować Portainera na Twoim serwerze w zaledwie kilka minut. Nie musisz być ekspertem od Linuxa – wystarczy, że będziesz kopiować i wklejać poniższe komendy!

> **Wymagania wstępne:** Będziesz potrzebować czystego serwera [VPS](/premium-vps/) z systemem Ubuntu lub Debian (np. od <span class="text-white">Voxi</span><span class="text-amber-300">Host</span>) oraz programu do połączenia się z serwerem (np. PuTTY lub systemowy Terminal).

---

## Krok 1: Połączenie z serwerem i instalacja Dockera

Najpierw połącz się ze swoim serwerem przez SSH. Następnie upewnijmy się, że na serwerze zainstalowany jest Docker. Jeśli jeszcze go nie masz, wystarczy wkleić poniższą komendę, która zrobi wszystko za Ciebie:

{% image "/assets/images/blog/setup-portainer-ubuntu-debian/H1.png", "Terminal podczas instalacji środowiska Docker", "(max-width: 768px) 100vw, 800px" %}

```bash
curl -fsSL https://get.docker.com -o get-docker.sh && sudo sh get-docker.sh
```

Zaczekaj chwilę, aż instalacja się zakończy. 

---

## Krok 2: Przygotowanie miejsca dla Portainera

Portainer potrzebuje bezpiecznego miejsca na dysku, aby zapisywać Twoje hasła i konfiguracje. Wklej poniższą komendę w konsoli i wciśnij ENTER, aby utworzyć tzw. "wolumen":

{% image "/assets/images/blog/setup-portainer-ubuntu-debian/H2.png", "Tworzenie wolumenu w Dockerze", "(max-width: 768px) 100vw, 800px" %}

```bash
docker volume create portainer_data
```

---

## Krok 3: Uruchomienie Portainera

Teraz uruchomimy samego Portainera. Poniższa komenda pobierze najnowszą wersję programu i uruchomi ją w tle. Wklej ją w całości do konsoli i wciśnij ENTER:

{% image "/assets/images/blog/setup-portainer-ubuntu-debian/H3.png", "Uruchamianie kontenera Portainer z niezbędnymi parametrami", "(max-width: 768px) 100vw, 800px" %}

```bash
docker run -d -p 8000:8000 -p 9443:9443 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:latest
```

I to tyle! Instalacja zakończona.

---

## Krok 4: Pierwsze logowanie w przeglądarce

Możesz już zamknąć konsolę. Teraz otwórz swoją ulubioną przeglądarkę internetową (np. Chrome, Firefox) i wpisz w pasku adresu:

`https://ADRES_IP_TWOJEGO_SERWERA:9443`
*(Zastąp "ADRES_IP_TWOJEGO_SERWERA" rzeczywistym adresem IP swojego VPS, np. https://192.168.1.50:9443)*

> **Wskazówka:** Jeśli po wejściu na stronę widzisz błąd `Client sent an HTTP request to an HTTPS server.`, oznacza to, że Twoja przeglądarka spróbowała połączyć się przez zwykłe HTTP (np. po wpisaniu samego IP). Upewnij się, że adres na samym początku ma wpisane ręcznie **https://**.

> **Ważne:** Przeglądarka może wyświetlić ostrzeżenie o "Niezaufanym certyfikacie" (ang. *Your connection is not private*). To całkowicie normalne, ponieważ serwer używa domyślnego certyfikatu własnego podpisu. Kliknij **"Zaawansowane"**, a następnie **"Kontynuuj / Przejdź na stronę"**.

Na ekranie zobaczysz kreator początkowy. Wymyśl dla siebie bezpieczne hasło, wpisz je dwukrotnie i kliknij przycisk tworzenia użytkownika (Create user).

> **Ważne (Limit 5 minut!):** Portainer dba o bezpieczeństwo. Jeśli od momentu uruchomienia (Krok 3) do ustalenia hasła minie ponad 5 minut, instalacja zostanie zablokowana (komunikat *Your Portainer instance timed out*). Aby ją odblokować, wpisz w konsoli `docker restart portainer` i odśwież stronę. Zostaniesz wtedy poproszony o **Setup Token**. Aby go uzyskać, wpisz w konsoli `docker logs portainer`. W logach znajdziesz napis *Use the following token to setup the administrator user:* a pod nim swój token. Skopiuj go i wklej na stronie.

{% image "/assets/images/blog/setup-portainer-ubuntu-debian/H4.png", "Ekran tworzenia konta administratora w Portainer", "(max-width: 768px) 100vw, 800px" %}

---

## Krok 5: Zarządzanie kontenerami

Po zalogowaniu kliknij przycisk **"Get Started"**, a na kolejnym ekranie wybierz środowisko z ikonką wieloryba podpisane jako **"Local"** (czyli serwer, na którym właśnie jesteśmy). 

Gotowe! Teraz widzisz pełny panel kontrolny. Od teraz nie musisz używać komend w czarnej konsoli – możesz zarządzać swoimi aplikacjami klikając myszką.

{% image "/assets/images/blog/setup-portainer-ubuntu-debian/H5.png", "Panel główny Portainer pokazujący listę kontenerów", "(max-width: 768px) 100vw, 800px" %}

---

## Podsumowanie

Portainer to niesamowite narzędzie, które "uczłowiecza" technologię Docker i sprawia, że jest ona dostępna nawet dla osób bez dużego doświadczenia z systemem Linux. Jeśli potrzebujesz serwera do nauki, sprawdź naszą ofertę [Premium VPS](/premium-vps/), które idealnie nadają się do takich eksperymentów!