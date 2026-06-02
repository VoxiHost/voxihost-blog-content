---
image: /assets/images/blog/pl/jak-dodac-uzytkownika-sudo-ubuntu/og-image.png
title: 'Jak utworzyć użytkownika sudo na Ubuntu i Debian: Kompletny przewodnik serwera'
description: Kompletny przewodnik dla początkujących do tworzenia nowego użytkownika z uprawnieniami sudo na serwerach Ubuntu i Debian. Przestań logować się jako root i zabezpiecz swój VPS.
date: '2026-03-25'
translationKey: create-sudo-user-ubuntu-debian
category: Poradniki
tags:
  - ubuntu
  - debian
  - sudo
  - user management
  - linux
  - vps
  - security
  - usermod
  - adduser
howto:
  name: Jak utworzyć użytkownika sudo na Ubuntu i Debian
  totalTime: PT5M
  yield: Nowe konto użytkownika niestandardowego z pełnymi uprawnieniami administracyjnymi (sudo)
  tool:
    - VPS lub dedykowany serwer z Ubuntu lub Debian
    - Klient SSH (np. terminal, PuTTY)
    - Dostęp do konta root
  steps:
    - name: Zaloguj się jako root
      text: Połącz się z serwerem przez SSH używając konta root.
      url: krok-1-dodaj-nowego-uzytkownika
    - name: Dodaj nowego użytkownika
      text: Uruchom adduser username i postępuj zgodnie z instrukcjami aby ustawić silne hasło.
      url: krok-2-nadaj-uprawnienia-sudo
    - name: Nadaj uprawnienia sudo
      text: Uruchom usermod -aG sudo username aby dodać nowego użytkownika do grupy sudo.
      url: krok-3-przetestuj-nowego-uzytkownika-sudo
    - name: Przetestuj nowe konto
      text: Przełącz się na nowego użytkownika za pomocą su - username i przetestuj dostęp sudo uruchamiając sudo whoami.
      url: step-3-test-the-new-sudo-user
status: published
locale: pl
author:
  name: VoxiHost Team
  link: https://voxihost.pl/
contributors:
  - danielmarszalkowski
---

Gdy wdrażasz nowy serwer, czy to chmurowy VPS czy maszynę fizyczną, dostawca hostingu zazwyczaj przekazuje ci dane logowania `root`. Konto root ma absolutną władzę nad systemem, bez ograniczeń, bez promptów potwierdzenia, bez siatki bezpieczeństwa. Jedno złe polecenie i możesz wyczyścić cały dysk.

Logowanie bezpośrednio jako root jest powszechnie uważane za złą praktykę bezpieczeństwa. Zamiast tego, powinieneś stworzyć zwykłe konto użytkownika, nadać mu uprawnienia administracyjne (przez `sudo`) i używać tego konta do codziennej pracy.

Na Ubuntu i Debian, tworzenie nowego użytkownika sudo zajmuje około dwóch minut.

## Krok 1: Dodaj nowego użytkownika

Najpierw połącz się z serwerem przez SSH jako użytkownik `root`:

```bash
ssh root@your-server-ip
```

Użyj polecenia `adduser` aby utworzyć nowe konto. Zastąp `yourusername` dowolną nazwą którą chcesz użyć (nie używaj spacji ani wielkich liter):

{% image "/assets/images/blog/pl/jak-dodac-uzytkownika-sudo-ubuntu/H1.png", "Uruchamianie polecenia adduser w terminalu Ubuntu aby utworzyć nowe konto użytkownika niestandardowego", "(max-width: 768px) 100vw, 800px" %}

```bash
adduser yourusername
```

Zostaniesz poproszony o ustawienie i potwierdzenie nowego hasła. Uczyń je silnym, jeśli twój serwer ma włączone uwierzytelnianie hasłem SSH, boty będą w końcu próbować je odgadnąć.

```text
Adding user `yourusername' ...
Adding new group `yourusername' (1001) ...
Adding new user `yourusername' (1001) with group `yourusername' ...
Creating home directory `/home/yourusername' ...
Copying files from `/etc/skel' ...
New password:
Retype new password:
passwd: password updated successfully
```

Po haśle zostaniesz poproszony o informacje o użytkowniku (Pełna nazwa, Numer pokoju, itp.). To relikt z wczesnych dni Uniksa. Nie musisz niczego wypełniać, po prostu naciśnij `ENTER` aby pominąć wszystkie, i naciśnij `Y` gdy zapytane czy informacje są poprawne.

## Krok 2: Nadaj uprawnienia sudo

Domyślnie nowi użytkownicy na Ubuntu i Debian to standardowe, nieuprzywilejowane konta. Nie mogą instalować oprogramowania ani edytować konfiguracji systemu.

Aby nadać uprawnienia administracyjne, musisz dodać użytkownika do grupy `sudo`. Członkowie tej grupy są automatycznie zezwalani na uruchamianie dowolnego polecenia z uprawnieniami root poprzez wpisanie `sudo` przed nim.

Uruchom to polecenie (ponownie, zastąp `yourusername`):

{% image "/assets/images/blog/pl/jak-dodac-uzytkownika-sudo-ubuntu/H2.png", "Uruchamianie polecenia usermod -aG sudo na Ubuntu aby nadać uprawnienia sudo użytkownikowi niestandardowemu", "(max-width: 768px) 100vw, 800px" %}

```bash
usermod -aG sudo yourusername
```

Flagi `-aG` są ważne. `-a` oznacza "dołącz" i `-G` oznacza "grupy". Jeśli zapomnisz `-a`, użytkownik zostanie usunięty ze wszystkich swoich innych grup i dodany *tylko* do grupy sudo, co psuje rzeczy.

## Krok 3: Przetestuj nowego użytkownika sudo

Zanim wylogujesz się z sesji root, upewnij się że nowy użytkownik działa. Użyj polecenia `su` (switch user) aby natychmiast przełączyć się na nowe konto:

{% image "/assets/images/blog/pl/jak-dodac-uzytkownika-sudo-ubuntu/H3.png", "Używanie polecenia su w Ubuntu aby przełączyć konta użytkowników i przetestować dostęp nowego użytkownika", "(max-width: 768px) 100vw, 800px" %}

```bash
su - yourusername
```

Twój prompt polecenia zmieni się z `root@server` na `yourusername@server`. Teraz zweryfikuj że twoje uprawnienia sudo działają uruchamiając polecenie które wymaga dostępu root, jak sprawdzenie katalogu root lub przetestowanie polecenia `whoami`:

{% image "/assets/images/blog/pl/jak-dodac-uzytkownika-sudo-ubuntu/H4.png", "Uruchamianie sudo whoami na Ubuntu aby zweryfikować że uprawnienia sudo zostały pomyślnie nadane nowemu użytkownikowi", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo whoami
```

Pierwszy raz gdy użyjesz `sudo` na nowym koncie, zobaczysz standardowy wykład o szanowaniu prywatności innych i myśleniu przed pisaniem. Potem poprosi o twoje hasło (to które ustawiłeś w Kroku 1, nie hasło root).

Jeśli wynik mówi `root`, sukces! Twój użytkownik ma uprawnienia administracyjne.

## Następne kroki

Teraz gdy masz bezpieczniejszy sposób administracji serwerem, możesz zamknąć sesję SSH i zalogować się ponownie bezpośrednio jako nowy użytkownik:

```bash
ssh yourusername@your-server-ip
```

Dla naprawdę bezpiecznego serwera, twoim następnym ruchem powinno być skonfigurowanie uwierzytelniania kluczem SSH dla tego nowego użytkownika i [całkowite wyłączenie logowania root](/pl/blog/jak-zabezpieczyc-ssh-ubuntu-debian/).

Jeśli szukasz czystego, szybkiego środowiska do hostowania swoich projektów, wdróż [Budget VPS](/pl/budget-vps/), stwórz swojego użytkownika i zacznij budować.