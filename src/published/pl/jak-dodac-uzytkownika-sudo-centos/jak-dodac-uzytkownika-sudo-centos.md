---
image: /assets/images/blog/pl/jak-dodac-uzytkownika-sudo-centos/og-image.png
title: 'Jak utworzyć użytkownika sudo na AlmaLinux, CentOS, Rocky Linux i Fedora: Kompletny przewodnik serwera'
description: Kompletny przewodnik dla początkujących do tworzenia nowego użytkownika z uprawnieniami sudo na serwerach AlmaLinux, CentOS Stream, Rocky Linux i Fedora.
date: '2026-03-25'
translationKey: create-sudo-user-rhel-fedora
category: Poradniki
tags:
  - almalinux
  - centos
  - rocky linux
  - fedora
  - sudo
  - user management
  - linux
  - vps
  - security
  - usermod
  - wheel
howto:
  name: Jak utworzyć użytkownika sudo na AlmaLinux, CentOS Stream, Rocky Linux i Fedora
  totalTime: PT5M
  yield: Nowe konto użytkownika niestandardowego z pełnymi uprawnieniami administracyjnymi (sudo)
  tool:
    - VPS lub dedykowany serwer z AlmaLinux, CentOS Stream, Rocky Linux lub Fedora
    - Klient SSH (np. terminal, PuTTY)
    - Dostęp do konta root
  steps:
    - name: Zaloguj się jako root
      text: Połącz się z serwerem przez SSH używając konta root.
      url: krok-1-dodaj-nowego-uzytkownika
    - name: Dodaj nowego użytkownika
      text: Uruchom useradd username aby utworzyć strukturę konta.
      url: krok-2-dodaj-uzytkownika-do-grupy-wheel
    - name: Ustaw hasło
      text: Uruchom passwd username aby przypisać hasło do nowego konta.
      url: krok-3-przetestuj-nowego-uzytkownika-sudo
    - name: Nadaj uprawnienia sudo
      text: Uruchom usermod -aG wheel username aby dodać nowego użytkownika do grupy wheel.
      url: step-2-add-the-user-to-the-wheel-group
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

Gdy wdrażasz nowy serwer, czy to chmurowy VPS czy maszynę fizyczną, zazwyczaj otrzymujesz dane logowania `root`. Konto root ma absolutną, nieograniczoną władzę. Jeden źle wpisany polecenie (`rm -rf /` przychodzi do głowy) i serwer znika bez ostrzeżeń ani potwierdzeń.

Z tego powodu logowanie bezpośrednio jako root to straszny nawyk. Standardowa praktyka bezpieczeństwa we wszystkich dystrybucjach Linuksa to tworzenie zwykłego konta użytkownika, nadanie mu uprawnień administracyjnych (przez `sudo`) i używanie tego konta do codziennej administracji.

Na dystrybucjach rodziny RHEL (AlmaLinux, CentOS Stream, Rocky Linux) i Fedorze, proces wymaga tylko trzech poleceń.

## Krok 1: Dodaj nowego użytkownika

Połącz się z serwerem przez SSH jako użytkownik `root`:

```bash
ssh root@your-server-ip
```

W przeciwieństwie do Ubuntu, które używa interaktywnego skryptu `adduser`, dystrybucje oparte na RHEL typowo używają standardowego polecenia `useradd`. Wykonuje swoją pracę cicho bez pytania o dodatkowe informacje.

Zastąp `yourusername` nazwą którą chcesz użyć (małe litery, bez spacji):

{% image "/assets/images/blog/pl/jak-dodac-uzytkownika-sudo-centos/H1.png", "Uruchamianie polecenia useradd yourusername na AlmaLinux, CentOS, Rocky Linux i Fedora aby utworzyć nowe konto użytkownika", "(max-width: 768px) 100vw, 800px" %}

```bash
useradd yourusername
```

Użytkownik teraz istnieje, ale nie ma hasła, co oznacza że nie może się zalogować. Przypisz hasło używając polecenia `passwd`:

{% image "/assets/images/blog/pl/jak-dodac-uzytkownika-sudo-centos/H2.png", "Uruchamianie polecenia passwd yourusername na AlmaLinux, CentOS, Rocky Linux i Fedora aby ustawić hasło dla nowego konta użytkownika", "(max-width: 768px) 100vw, 800px" %}

```bash
passwd yourusername
```

Zostaniesz poproszony o wpisanie i potwierdzenie nowego hasła. Uczyń je silnym i bezpiecznym.

```text
Changing password for user yourusername.
New password:
Retype new password:
passwd: all authentication tokens updated successfully.
```

## Krok 2: Dodaj użytkownika do grupy wheel

Domyślnie nowy użytkownik nie może uruchamiać poleceń administracyjnych. Aby to naprawić, musisz dodać go do konkretnej grupy.

Oto główna różnica między systemami rodziny Debian a RHEL: na Ubuntu/Debian grupa administracyjna nazywa się `sudo`. Na AlmaLinux, CentOS, Rocky Linux i Fedorze grupa administracyjna nazywa się **`wheel`**.

Dodaj swojego nowego użytkownika do grupy `wheel` używając polecenia `usermod`:

{% image "/assets/images/blog/pl/jak-dodac-uzytkownika-sudo-centos/H3.png", "Uruchamianie polecenia usermod -aG wheel yourusername na AlmaLinux, CentOS, Rocky Linux i Fedora aby dodać nowego użytkownika do grupy wheel", "(max-width: 768px) 100vw, 800px" %}

```bash
usermod -aG wheel yourusername
```

**Ważne:** Nigdy nie zapominaj o fladze `-a`. Flagi `-aG` oznaczają "dołącz do grup". Jeśli użyjesz tylko `-G`, użytkownik zostanie usunięty ze wszystkich swoich innych domyślnych grup i dodany *tylko* do `wheel`.

Członkowie grupy `wheel` są automatycznie nadawani pełne uprawnienia `sudo` przez domyślną politykę systemu.

## Krok 3: Przetestuj nowego użytkownika sudo

Zweryfikuj że nowa konfiguracja działa przed wylogowaniem się z konta root. Przełącz się na nowego użytkownika bezproblemowo używając polecenia `su`:

{% image "/assets/images/blog/pl/jak-dodac-uzytkownika-sudo-centos/H4.png", "Uruchamianie polecenia su - yourusername na AlmaLinux, CentOS, Rocky Linux i Fedora aby przełączyć się na nowe konto użytkownika", "(max-width: 768px) 100vw, 800px" %}

```bash
su - yourusername
```

Twój prompt zmieni się z `root@server` na `yourusername@server`. Teraz przetestuj dostęp administracyjny uruchamiając polecenie które wymaga uprawnień root:

{% image "/assets/images/blog/pl/jak-dodac-uzytkownika-sudo-centos/H5.png", "Uruchamianie polecenia sudo whoami na AlmaLinux, CentOS, Rocky Linux i Fedora aby przetestować dostęp sudo", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo whoami
```

Zobaczysz standardowy komunikat ostrzegawczy o szanowaniu prywatności innych. Wpisz hasło które utworzyłeś w Kroku 1.

Jeśli polecenie wyświetla `root`, twoja konfiguracja jest idealnie skonfigurowana.

## Następne kroki

Możesz teraz rozłączyć sesję root i zalogować się ponownie bezpośrednio jako nowo utworzony użytkownik:

```bash
ssh yourusername@your-server-ip
```

Teraz gdy masz użytkownika sudo, twoim następnym priorytetem bezpieczeństwa powinno być skonfigurowanie uwierzytelniania kluczem SSH i [całkowite wyłączenie bezpośredniego logowania root](/pl/blog/jak-zabezpieczyc-ssh-centos-rhel/).

Jeśli jesteś gotowy wdrożyć swoją następną aplikację w środowisku wysokiej wydajności, uruchom [Budget VPS](/pl/budget-vps/), zabezpiecz swoje konta użytkowników i rozpocznij hosting.