---
image: /assets/images/blog/pl/instalacja-sterownikow-nvidia-ubuntu/og-image.png
title: 'Kompletny przewodnik: Instalacja sterowników NVIDIA na Ubuntu i AnduinOS'
description: Instrukcja instalacji własnościowych sterowników NVIDIA na Ubuntu, w tym podpisywanie modułu dla Secure Boot oraz instalacja ręczna.
date: '2026-06-30'
translationKey: install-nvidia-drivers-ubuntu
locale: pl
category: Poradniki
tags:
  - nvidia
  - drivers
  - ubuntu
  - linux
  - gpu
status: draft
author:
  name: Anduin
  link: https://github.com/Anduin2017
contributors:
  - Anduin2017
  - danielmarszalkowski
howto:
  name: Instalacja sterowników NVIDIA na Ubuntu
  totalTime: PT25M
  yield: System z działającymi własnościowymi sterownikami NVIDIA i włączoną akceleracją GPU.
  tool:
    - Komputer lub VPS z kartą graficzną NVIDIA
    - Ubuntu lub dystrybucja oparta na Ubuntu (np. AnduinOS)
    - Połączenie z internetem
  steps:
    - name: "Krok 1: Automatyczna instalacja"
      text: Użyj wbudowanego narzędzia ubuntu-drivers, aby zainstalować zalecaną wersję.
      url: krok-1-automatyczna-instalacja
    - name: "Krok 2: (Opcjonalnie) Instalacja z repozytorium PPA"
      text: Dodaj repozytorium PPA graphics-drivers, aby uzyskać najnowsze stabilne wersje.
      url: krok-2-opcjonalnie-instalacja-z-repozytorium-ppa
    - name: "Krok 3: Ręczna instalacja z włączonym Secure Boot"
      text: Podpisz moduł jądra, jeśli na Twoim komputerze jest włączona funkcja Secure Boot.
      url: krok-3-reczna-instalacja-z-wlaczonym-secure-boot
faq:
  - question: "Dlaczego komenda nvidia-smi zgłasza błąd po instalacji?"
    answer: "Najczęściej jest to spowodowane brakiem restartu systemu w celu załadowania modułów jądra lub zablokowaniem niepodpisanego modułu sterownika przez aktywną funkcję UEFI Secure Boot."
  - question: "Jak mogę wrócić do domyślnych, otwartoźródłowych sterowników Nouveau?"
    answer: "Aby przywrócić sterowniki Nouveau, usuń pakiety własnościowe NVIDIA za pomocą komendy <code>sudo apt purge nvidia-*</code>, a następnie zainstaluj Nouveau wpisując <code>sudo apt install xserver-xorg-video-nouveau</code>."
  - question: "Czy mogę uruchomić nvidia-smi na dowolnym serwerze VPS?"
    answer: "Nie, obsługa narzędzi NVIDIA wymaga fizycznego układu graficznego. Narzędzie zadziała tylko w przypadku planów VPS z dedykowanym passthrough GPU lub na serwerach dedykowanych z kartą graficzną."
---

## Wprowadzenie

Wydajna praca karty graficznej NVIDIA pod systemem Linux wymaga instalacji własnościowych sterowników producenta. Choć otwartoźródłowy sterownik Nouveau wystarcza do podstawowych zadań biurowych, nie oferuje on optymalizacji niezbędnych do uruchamiania gier, renderowania wideo oraz obliczeń AI na serwerze lub stacji roboczej w <span class="text-white">Voxi</span><span class="text-amber-300">Host</span>.

W tym przewodniku przedstawiamy trzy główne metody instalacji sterowników NVIDIA na systemach opartych na Ubuntu, od zautomatyzowanego wdrożenia po zaawansowaną instalację ręczną.

> **Wymagania wstępne:** Upewnij się, że Twój system jest zaktualizowany, a komputer posiada kompatybilną kartę NVIDIA. Zawsze [wykonaj kopię zapasową systemu](/pl/premium-vps/) przed modyfikacją sterowników graficznych.

---

## Krok 1: Automatyczna instalacja

Zalecanym i najprostszym podejściem jest automatyczne wykrycie sprzętu przez system i wybór właściwego pakietu:

```bash
sudo apt update
sudo ubuntu-drivers install
```

Po zakończeniu procesu uruchom ponownie system. Metoda ta jest rekomendowana w większości przypadków, ponieważ automatycznie konfiguruje zależności oraz mechanizm DKMS (Dynamic Kernel Module Support).

{% image "/assets/images/blog/pl/instalacja-sterownikow-nvidia-ubuntu/H1.png", "Terminal pokazujący wykrywanie sprzętu i zalecane własnościowe sterowniki NVIDIA", "(max-width: 768px) 100vw, 800px" %}

---

## Krok 2: (Opcjonalnie) Instalacja z repozytorium PPA

Gdy potrzebujesz nowszego sterownika niż wersje oferowane w standardowych repozytoriach (na przykład przy konfiguracji najnowszych architektur GPU), skorzystaj z oficjalnego repozytorium Graphics Drivers PPA:

```bash
sudo add-apt-repository ppa:graphics-drivers/ppa
sudo apt update
```

Następnie wyświetl listę dostępnych wersji i zainstaluj wybraną z nich:

```bash
ubuntu-drivers list
sudo apt install nvidia-driver-550 # Zastąp '550' wybranym numerem wersji
```

---

## Krok 3: Ręczna instalacja z włączonym Secure Boot

W przypadku ręcznej instalacji z oficjalnego pliku `.run` przy aktywnym zabezpieczeniu UEFI Secure Boot, konieczne jest ręczne podpisanie modułu jądra przed jego załadowaniem.

### Generowanie klucza podpisującego

```bash
mkdir ~/mok-keys && cd ~/mok-keys
openssl req -new -newkey rsa:2048 -days 36500 -nodes -keyout MOK.key -out MOK.csr
openssl x509 -req -in MOK.csr -signkey MOK.key -out MOK.crt
openssl x509 -in MOK.crt -outform DER -out MOK.der
```

### Rejestracja klucza (MOK)

```bash
sudo mokutil --import MOK.der
```

Wpisz jednorazowe hasło i uruchom ponownie komputer. Podczas rozruchu wyświetli się niebieski ekran narzędzia MokManager: wybierz opcję „Enroll MOK”, potwierdź import i podaj utworzone hasło.

### Uruchomienie instalatora

Po uruchomieniu systemu zatrzymaj menedżer wyświetlania (jeśli jest aktywny) i uruchom instalator, wskazując ścieżki do wygenerowanych kluczy jako parametry:

```bash
sudo ./NVIDIA-Linux-x86_64-xxx.xx.run --module-signing-secret-key=/home/nazwa_uzytkownika/mok-keys/MOK.key --module-signing-public-key=/home/nazwa_uzytkownika/mok-keys/MOK.crt
```

---

## Podsumowanie

Po zakończeniu instalacji możesz zweryfikować status karty graficznej, wpisując:

```bash
nvidia-smi
```

{% image "/assets/images/blog/pl/instalacja-sterownikow-nvidia-ubuntu/H2.png", "Ekran terminala z uruchomionym poleceniem nvidia-smi prezentującym szczegóły karty graficznej", "(max-width: 768px) 100vw, 800px" %}

Jeśli na ekranie zobaczysz tabelę ze szczegółami o swojej karcie GPU, wszystko działa prawidłowo! Jeśli uruchamiasz kontenery korzystające z akceleracji graficznej, pamiętaj o zapoznaniu się z dokumentacją [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html).

Szukasz wydajnego środowiska do realizacji swoich projektów sztucznej inteligencji? Sprawdź plany [Premium VPS](/pl/premium-vps/) od <span class="text-white">Voxi</span><span class="text-amber-300">Host</span> z dedykowanymi zasobami sprzętowymi.
