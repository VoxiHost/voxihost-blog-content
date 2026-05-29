---
image: /assets/images/blog/pl/przesylanie-plikow-vps-sftp-filezilla/og-image.png
title: Jak transferować pliki na VPS używając SFTP i FileZilla
description: Kompletny przewodnik dla początkujących do bezpiecznego transferowania plików z komputera na serwer Linux VPS używając SFTP i FileZilla.
date: '2026-03-25'
translationKey: transfer-files-vps-sftp-filezilla
category: Poradniki
tags:
  - sftp
  - filezilla
  - ftp
  - ssh
  - linux
  - vps
  - file transfer
  - ubuntu
  - debian
howto:
  name: Jak połączyć się z VPS używając SFTP i FileZilla
  totalTime: PT5M
  yield: Bezpieczne, wizualne połączenie transferowe zdolne do przeciągania i upuszczania plików między komputerem a serwerem Linux
  tool:
    - Serwer Linux VPS lub dedykowany
    - FileZilla FTP Client zainstalowany na Twoim pulpicie
    - Dane logowania do serwera (IP, użytkownik, hasło lub klucz SSH)
  steps:
    - name: Pobierz FileZilla
      text: Pobierz i zainstaluj bezpłatnego klienta FileZilla z oficjalnej strony internetowej.
      url: step-1-download-filezilla
    - name: Skonfiguruj Menedżera Witryn
      text: Otwórz Menedżera Witryn FileZilla i utwórz stałe, zapisane połączenie z serwerem.
      url: step-2-configure-the-site-manager
    - name: Wybierz protokół SFTP
      text: Zmień protokół z FTP na SFTP - SSH File Transfer Protocol w ustawieniach FileZilla.
      url: step-3-select-the-sftp-protocol
    - name: Dodaj dane logowania
      text: Wprowadź adres IP serwera, nazwę użytkownika i hasło lub wybierz plik klucza SSH.
      url: step-4-add-your-credentials-or-ssh-key
    - name: Połącz i transferuj
      text: Kliknij Połącz, zaakceptuj ostrzeżenie o kluczu hosta i przeciągnij pliki przeciągając i upuszczając je.
      url: step-5-connect-and-transfer
status: published
locale: pl
author:
  name: VoxiHost Team
  link: https://voxihost.pl/
contributors:
  - danielmarszalkowski
---

Gdy wynajmujesz nowy serwer Linux VPS, często jesteś witany przerażającym czarnym ekranem terminala. Chociaż technicznie możesz przesyłać pliki używając poleceń jak `scp` czy `rsync`, jest to znacznie trudniejsze i bardziej podatne na błędy niż użycie wizualnego interfejsu.

Zdecydowanie najlepszą metodą jest użycie **SFTP** (Secure File Transfer Protocol) z darmowym klientem **FileZilla**.

## Wymagania wstępne

Zanim zaczniesz, musisz mieć dwie rzeczy na miejscu:
1. **Zainstalowany serwer WWW** (Nginx lub Apache) z skonfigurowanym blokiem serwera/Wirtualnym Hostem dla Twojej domeny.
2. **Prawidłowe ustawienia DNS** - Twoja domena (np., `twoja_domena.com` i `www.twoja_domena.com`) musi mieć aktywne rekordy `A` wskazujące na publiczny adres IP Twojego serwera.

## Krok 1: Pobierz FileZilla

FileZilla to darmowy, wieloplatformowy klient FTP który jest standardem branży.

Pobierz i zainstaluj FileZilla na swoim komputerze:

{% image "/assets/images/blog/pl/przesylanie-plikow-vps-sftp-filezilla/H1.png", "Pobieranie FileZilla Client z oficjalnej strony internetowej do instalacji SFTP na komputerze", "(max-width: 768px) 100vw, 800px" %}

```bash
# Jeśli używasz system Windows:
wget https://filezilla-project.org/download.php?type=client&platform=win64
# Jeśli używasz system macOS:
wget https://filezilla-project.org/download.php?type=client&platform=osx
# Jeśli używasz system Linux:
wget https://filezilla-project.org/download.php?type=client&platform=linux_x86_64
```

Po pobraniu uruchom instalator i postępuj zgodnie z instrukcjami.

## Krok 2: Skonfiguruj Menedżera Witryn

Po zainstalowaniu otwórz FileZilla. Zamiast używać paska szybkiego połączenia, użyjemy Menedżera Witryn do stworzenia stałego, zapisanego połączenia.

Otwórz Menedżera Witryn:

{% image "/assets/images/blog/pl/przesylanie-plikow-vps-sftp-filezilla/H2.png", "Otwieranie Menedżera Witryn FileZilla w celu utworzenia nowego, stałego połączenia SFTP z serwerem Linux VPS", "(max-width: 768px) 100vw, 800px" %}

Możesz go znaleźć w menu Start lub wyszukując "FileZilla" w swoich aplikacjach.

1. Kliknij **Nowa Strona** w lewym górnym rogu.

2. Wprowadź rozpoznawalną nazwę dla połączenia, np. "Mój Serwer VPS".

3. Kliknij kartę **Protokół** w prawym górnym rogu.

4. Zmień protokół z FTP na SFTP - SSH File Transfer Protocol w rozwijanym menu.

## Krok 3: Wybierz protokół SFTP

Standardowo FileZilla używa zwykłego FTP, który wysyła hasła i dane w postaci jawnym przez internet. To bardzo niebezpieczne!

Zmień na SFTP:

{% image "/assets/images/blog/pl/przesylanie-plikow-vps-sftp-filezilla/H3.png", "Zmienianie protokołu z FTP na SFTP - SSH File Transfer Protocol w ustawieniach FileZilla", "(max-width: 768px) 100vw, 800px" %}

W menu rozwijanym obok pola "Protokół" wybierz:
**`SFTP - SSH File Transfer Protocol`**

## Krok 4: Dodaj dane logowania

Teraz FileZilla potrzebuje informacji aby połączyć się z Twoim serwerem.

1. **Host**: Wprowadź publiczny adres IP Twojego serwera (np., `192.168.1.100`).

2. **Port**: Zostaw to pole puste, chyba że zmieniłeś port SSH (standardowo 22).

3. **Logon Type**: To jest najbezpieczniej, wybierz **`Plik klucza`** jeśli używasz kluczy SSH. Jeśli używasz hasła, wybierz **`Normal`**.

4. **Użytkownik**: Wprowadź nazwę użytkownika (zwykle `root` dla świeżego wdrożenia serwera).

5. **Plik klucza**: Jeśli wybrałeś "Plik klucza", kliknij **Przeglądaj** i znajdź swój prywatny klucz SSH (plik z rozszerzeniem `.pub`).

6. Kliknij **Połącz**, aby połączyć się z serwerem.

> **Ważna uwaga dotycząca haseł:** Jeśli używasz uwierzytelniania hasłowym, FileZilla poprosi o hasło przy każdym połączeniu. Upewnij się że wpisujesz silne, unikalne hasło.

## Krok 5: Połącz i transferuj

Gdy wszystko jest skonfigurowane, czas na transfer!

Kliknij przycisk **Połącz** w dolnym oknie.

Zaakceptuj ostrzeżenie o kluczu hosta (pojawia się tylko przy pierwszym połączeniu z tym hostem). To jest standardowe środowisko bezpieczeństwa chroniące przed "man in the middle" atakami.

{% image "/assets/images/blog/pl/przesylanie-plikow-vps-sftp-filezilla/H5.png", "Okno weryfikacji klucza hosta FileZilla podczas pierwszego połączenia z VPS przez SFTP", "(max-width: 768px) 100vw, 800px" %}

Po nawiązaniu połączenia zobaczysz interfejs podzielony na dwie części:

{% image "/assets/images/blog/pl/przesylanie-plikow-vps-sftp-filezilla/H6.png", "Interfejs FileZilla podzielony na dwie części pokazujący lokalne pliki po lewej i zdalne pliki serwera Linux VPS po prawej dla transferu przeciągania i upuszczania", "(max-width: 768px) 100vw, 800px" %}

- **Lewa strona**: Twój lokalny komputer z plikami i folderami.
- **Prawa strona**: Twój zdalny serwer Linux VPS.

Aby transferować pliki, po prostu **przeciągnij i upuść** je z lewego okna do prawego okna.

Typowe miejsce do przesyłania plików WWW to:
`/var/www/html/`

Przejdź tam w oknie po prawej stronie, przeciągnij swój `index.html` z pulpitu (lewa strona) do tego folderu, a Twoja strona jest aktywna!

Możesz teraz bezpiecznie zarządzać plikami na swoim serwerze VPS z wygody i intuicyjnym interfejsem graficznym!

Jeśli nie masz jeszcze serwera do ćwiczeń, nasze plany [Budget VPS](/pl/budget-vps/) to idealne, tanie środowiska do nauki zarządzania Linuksem bez ryzyka.