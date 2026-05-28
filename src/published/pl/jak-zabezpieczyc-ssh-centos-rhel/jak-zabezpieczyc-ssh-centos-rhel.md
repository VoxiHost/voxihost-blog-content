---
image: /assets/images/blog/jak-zabezpieczyc-ssh-centos-rhel/og-image.png
title: 'Jak zabezpieczyć SSH na AlmaLinux, CentOS, Rocky Linux i Fedora: Kompletny przewodnik serwera'
description: Kompletny przewodnik do utwardzania SSH na serwerach AlmaLinux, CentOS Stream, Rocky Linux i Fedora. Wyłącz logowanie root, skonfiguruj uwierzytelnianie oparte na kluczach, zmień domyślny port, skonfiguruj firewalld i chroń swój VPS przed atakami siłowymi.
date: '2026-03-25'
translationKey: secure-ssh-rhel-fedora
category: Poradniki
tags:
  - ssh
  - almalinux
  - centos
  - rocky linux
  - fedora
  - security
  - vps
  - hardening
  - key authentication
  - firewalld
  - brute force protection
howto:
  name: Jak zabezpieczyć SSH na AlmaLinux, CentOS Stream, Rocky Linux i Fedora
  totalTime: PT15M
  yield: Utwierdzony serwer rodziny RHEL lub Fedora z uwierzytelnianiem SSH kluczowym, wyłączonym logowaniem root i dostępem chronionym przez firewalld
  tool:
    - VPS lub dedykowany serwer z AlmaLinux, CentOS Stream, Rocky Linux lub Fedora
    - Klient SSH z obsługą generowania kluczy (np. terminal, PuTTY)
    - Dostęp sudo lub root
  steps:
    - name: Wygeneruj parę kluczy SSH
      text: Uruchom ssh-keygen -t ed25519 na swojej lokalnej maszynie aby wygenerować nowoczesną parę kluczy SSH.
      url: set-up-ssh-key-authentication
    - name: Skopiuj swój klucz publiczny na serwer
      text: Uruchom ssh-copy-id user@your-server-ip aby zainstalować swój klucz publiczny na serwerze.
      url: set-up-ssh-key-authentication
    - name: Wyłącz logowanie root
      text: Ustaw PermitRootLogin no w /etc/ssh/sshd_config aby zapobiec bezpośredniemu dostępowi root.
      url: disable-root-login
    - name: Wyłącz uwierzytelnianie hasłem
      text: Ustaw PasswordAuthentication no w /etc/ssh/sshd_config aby wymagać tylko logowania opartego na kluczach.
      url: disable-password-authentication
    - name: Zmień domyślny port SSH
      text: Ustaw Port 2222 w sshd_config, zaktualizuj etykiety portów SELinux i skonfiguruj firewalld odpowiednio.
      url: change-the-default-port
    - name: Zrestartuj SSH i zweryfikuj
      text: Uruchom sudo systemctl restart sshd i przetestuj swoje połączenie przed zamknięciem obecnej sesji.
      url: restart-sshd-and-verify
status: published
locale: pl
author:
  name: VoxiHost Team
  link: https://voxihost.pl/
contributors:
  - danielmarszalkowski
---

Moment gdy serwer z publicznym IP idzie na żywo, zautomatyzowane skanery zaczynają sondować port 22. To nie jest osobiste, to po prostu dzieje się w internecie. Większość z nich szuka logowań root ze słabymi hasłami lub domyślnymi danymi z obrazów chmurowych które nie były dotknięte.

Zablokowanie SSH na AlmaLinux, CentOS Stream, Rocky Linux i Fedorze zajmuje te same 15 minut co na dowolnym serwerze Linux, z jednym dodatkowym krokiem który systemy oparte na RHEL wymagają: mówienie SELinux o jakichkolwiek zmianach portów które robisz. Pomiń ten krok i będziesz się dziwić dlaczego SSH przestało działać.

> **Wymaganie:** Ten przewodnik wyłącza logowanie root. **Musisz** mieć niestandardowego użytkownika z uprawnieniami `sudo` gotowego **przed** uruchomieniem jakichkolwiek z tych kroków. Jeśli jeszcze tego nie zrobiłeś, postępuj zgodnie z naszym przewodnikiem [Jak utworzyć użytkownika sudo na AlmaLinux, CentOS, Rocky Linux i Fedora](/pl/blog/jak-dodac-uzytkownika-sudo-centos/), potem wróć tutaj.

## Skonfiguruj uwierzytelnianie oparte na kluczach SSH

Rób klucze przed czymkolwiek innym. Uwierzytelnianie hasłem to główny wektor ataków siłowych SSH, a przełączenie na klucze eliminuje go całkowicie.

Na swojej **maszynie lokalnej**, wygeneruj parę kluczy ed25519:

{% image "/assets/images/blog/jak-zabezpieczyc-ssh-centos-rhel/H1.png", "Uruchamianie ssh-keygen -t ed25519 -C \"your-server-label\" na AlmaLinux, CentOS, Rocky Linux i Fedorze aby wygenerować parę kluczy ed25519", "(max-width: 768px) 100vw, 800px" %}

```bash
ssh-keygen -t ed25519 -C "your-server-label"
```

Ustaw hasło gdy zostaniesz o nie poproszony. Szyfruje klucz prywatny na dysku, jeśli ktoś dostanie się do twojej lokalnej maszyny, wciąż nie będzie mógł użyć klucza bez hasła.

Skopiuj klucz publiczny na serwer:

{% image "/assets/images/blog/jak-zabezpieczyc-ssh-centos-rhel/H2.png", "Uruchamianie ssh-copy-id -i ~/.ssh/id_ed25519.pub user@your-server-ip na AlmaLinux, CentOS, Rocky Linux i Fedorze aby skopiować klucz publiczny na serwer", "(max-width: 768px) 100vw, 800px" %}

```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub user@your-server-ip
```

Otwórz **nowe okno terminala** i zweryfikuj że możesz połączyć się z kluczem przed zmianą czegokolwiek innego. Jeśli logujesz się bez monitu o hasło, klucz działa. Zachowaj swoją obecną sesję otwartą, będziesz jej potrzebował jako zapasowe jeśli coś pójdzie nie tak w późniejszych krokach.

## Wyłącz logowanie root

Bezpośrednie logowanie root to niepotrzebne ryzyko. Jeśli twój klucz zostanie naruszony, atakujący natychmiast ma nieograniczony dostęp. Użyj konta niestandardowego z sudo zamiast tego.

Edytuj konfigurację SSH:

{% image "/assets/images/blog/jak-zabezpieczyc-ssh-centos-rhel/H3.png", "Uruchamianie sudo nano /etc/ssh/sshd_config na AlmaLinux aby otworzyć i edytować plik konfiguracyjny demona SSH aby wyłączyć logowanie root", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo nano /etc/ssh/sshd_config
```

Znajdź i ustaw:

```ini
PermitRootLogin no
```

Jeśli to nie jest ustawione, na systemach opartych na RHEL domyślne mogą się różnić w zależności od obrazu chmurowego. Zawsze ustawiaj to jawnie.

## Wyłącz uwierzytelnianie hasłem

Z Twoim kluczem potwierdzonym działającym, wyłącz hasła:

{% image "/assets/images/blog/jak-zabezpieczyc-ssh-centos-rhel/H4.png", "Edytowanie /etc/ssh/sshd_config na AlmaLinux aby ustawić PasswordAuthentication no i PubkeyAuthentication yes aby wymusić logowanie tylko oparte na kluczach", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo nano /etc/ssh/sshd_config
```

Ustaw:

```ini
PasswordAuthentication no
PubkeyAuthentication yes
```

Na tych dystrybucjach, główny plik konfiguracyjny jest zazwyczaj autorytatywny. Ale sprawdź dwukrotnie czy nie ma nadpisów:

```bash
grep -r "PasswordAuthentication" /etc/ssh/
```

Jeśli cokolwiek w `/etc/ssh/sshd_config.d/` ustawia to na `yes`, napraw ten plik.

## Zaostrz kilka dodatkowych ustawień

Małe zmiany które redukują ekspozycję:

```ini
# Rozłącz po 3 nieudanych próbach autoryzacji
MaxAuthTries 3

# Zmniejsz okno dla niekompletnych połączeń
LoginGraceTime 30

# Wyłącz funkcje których nie używasz
X11Forwarding no
AllowTcpForwarding no
```

Jeśli tylko konkretni użytkownicy powinni mieć dostęp SSH:

```ini
AllowUsers youruser
```

Żaden użytkownik systemowy niewymieniony nie będzie mógł uwierzytelnić zdalnie, nawet z ważnymi danymi. Przydatne do blokowania kont aplikacyjnych.

## Zmień domyślny port

To jest miejsce gdzie systemy oparte na RHEL różnią się od systemów opartych na Debian. SELinux kontroluje które porty usługi mogą nasłuchiwać. Jeśli zmienisz port SSH bez aktualizacji SELinux, usługa nie uda się zrestartować.

Najpierw sprawdź które porty SSH może obecnie używać:

```bash
sudo semanage port -l | grep ssh
```

Dodaj swój nowy port do listy dozwolonych:

```bash
sudo semanage port -a -t ssh_port_t -p tcp 2222
```

Jeśli `semanage` nie jest dostępne:

```bash
sudo dnf install policycoreutils-python-utils -y
```

Potem edytuj `sshd_config`:

```ini
Port 2222
```

Teraz zaktualizuj firewalld aby zezwolić na nowy port i usunąć stary:

```bash
sudo firewall-cmd --permanent --add-port=2222/tcp
sudo firewall-cmd --permanent --remove-service=ssh
sudo firewall-cmd --reload
```

Zweryfikuj:

```bash
sudo firewall-cmd --list-all
```

Powinieneś zobaczyć `2222/tcp` na liście portów i `ssh` usunięte z usług.

## Zrestartuj sshd i zweryfikuj

Na systemach rodziny RHEL usługa to `sshd`, nie `ssh`:

```bash
sudo systemctl restart sshd
```

W **nowym oknie terminala**, połącz się na nowym porcie:

```bash
ssh -p 2222 user@your-server-ip
```

Jeśli działa, skończone. Jeśli nie, użyj swojej obecnej sesji do debugowania. Sprawdź błędy składni konfiguracji najpierw:

```bash
sudo sshd -t
```

To polecenie waliduje konfigurację bez faktycznego restartowania, powie ci czy jest literówka lub nieprawidłowe ustawienie.

## Zweryfikuj przypisanie portu SELinux

Po restarcie, potwierdź że SELinux zaakceptowało port:

```bash
sudo semanage port -l | grep ssh
```

Powinieneś zobaczyć swój nowy port wymieniony. Jeśli restart się powiódł, to powinno już być w porządku.

## Sprawdź logi autoryzacji

Zobacz co uderza w twój serwer:

```bash
sudo journalctl -u sshd --since "1 hour ago" | grep -E "Failed|Invalid"
```

Na poprawnie utwardzonym serwerze z wyłączonym uwierzytelnianiem hasłem i działającym na niestandardowym porcie, ten log powinien być zasadniczo pusty.

## Odmowy SELinux

Jeśli sshd nie uda się uruchomić lub połączyć po zmianie portu, sprawdź odmowy SELinux:

```bash
sudo ausearch -m avc -ts recent | grep sshd
```

To powie ci dokładnie co SELinux zablokowało, co czyni naprawianie tego znacznie prostszym niż zgadywanie.

Jeśli chcesz czysty VPS oparty na RHEL aby poćwiczyć to bez ryzyka, nasze plany [Budget VPS](/pl/budget-vps/) są wystarczająco tanie aby uruchomić skrzynkę testową, utwardzić ją i zacząć od nowa jeśli cokolwiek się zepsuje.