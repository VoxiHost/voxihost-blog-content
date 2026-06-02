---
image: /assets/images/blog/pl/jak-zabezpieczyc-ssh-ubuntu-debian/og-image.png
title: 'Jak zabezpieczyć SSH na Ubuntu i Debian: Kompletny przewodnik serwera'
description: Kompletny przewodnik do utwardzania SSH na serwerach Ubuntu i Debian. Wyłącz logowanie root, skonfiguruj uwierzytelnianie oparte na kluczach, zmień domyślny port, skonfiguruj ufw i zablokuj swój VPS przed atakami siłowymi.
date: '2026-03-25'
translationKey: secure-ssh-ubuntu-debian
category: Poradniki
tags:
  - ssh
  - ubuntu
  - debian
  - security
  - vps
  - hardening
  - key authentication
  - ufw
  - brute force protection
howto:
  name: Jak zabezpieczyć SSH na Ubuntu i Debian
  totalTime: PT15M
  yield: Utwierdzony serwer Ubuntu lub Debian z uwierzytelnianiem SSH kluczowym, wyłączonym logowaniem root i dostępem chronionym przez zaporę
  tool:
    - VPS lub dedykowany serwer z Ubuntu lub Debian
    - Klient SSH z obsługą generowania kluczy (np. terminal, PuTTY)
    - Niestandardowy użytkownik sudo (zobacz wymaganie poniżej)
  steps:
    - name: Wygeneruj parę kluczy SSH
      text: Uruchom ssh-keygen -t ed25519 na swojej lokalnej maszynie aby wygenerować nowoczesną parę kluczy SSH.
      url: skonfiguruj-uwierzytelnianie-oparte-na-kluczach-ssh
    - name: Skopiuj swój klucz publiczny na serwer
      text: Uruchom ssh-copy-id user@your-server-ip aby zainstalować swój klucz publiczny na serwerze.
      url: wylacz-logowanie-root
    - name: Wyłącz logowanie root
      text: Ustaw PermitRootLogin no w /etc/ssh/sshd_config aby zapobiec bezpośredniemu dostępowi root.
      url: wylacz-uwierzytelnianie-haslem
    - name: Wyłącz uwierzytelnianie hasłem
      text: Ustaw PasswordAuthentication no w /etc/ssh/sshd_config aby wymagać tylko logowania opartego na kluczach.
      url: zaostrz-kilka-dodatkowych-ustawien
    - name: Zmień domyślny port SSH
      text: Ustaw Port 2222 w sshd_config i zaktualizuj swoje reguły zapory UFW odpowiednio.
      url: zmien-domyslny-port
    - name: Zrestartuj SSH i zweryfikuj
      text: Uruchom sudo systemctl restart ssh i zweryfikuj z nowego okna terminala przed zamknięciem sesji.
      url: zrestartuj-ssh-i-zweryfikuj
status: published
locale: pl
author:
  name: VoxiHost Team
  link: https://voxihost.pl/
contributors:
  - danielmarszalkowski
---

Port 22 jest ciągle skanowany. Moment gdy uruchamiasz VPS z publicznym IP, zautomatyzowane boty zaczynają go bombardować w poszukiwaniu słabych haseł i domyślnych danych z obrazów chmurowych które nie były dotknięte. Utwardzanie SSH zajmuje około 15 minut i sprawia że twój serwer jest dramatycznie mniej interesujący dla każdego kto próbuje się włamać.

> **Wymaganie:** Ten przewodnik wyłącza logowanie root. **Musisz** mieć niestandardowego użytkownika z uprawnieniami `sudo` gotowego **przed** uruchomieniem jakichkolwiek z tych kroków. Jeśli jeszcze tego nie zrobiłeś, postępuj zgodnie z naszym przewodnikiem [Jak utworzyć użytkownika sudo na Ubuntu i Debian](/pl/blog/jak-dodac-uzytkownika-sudo-ubuntu/), potem wróć tutaj.

## Skonfiguruj uwierzytelnianie oparte na kluczach SSH

Jedna najskuteczniejsza zmiana jaką możesz zrobić. Logowania hasłowe mogą być siłowo złamane. Uwierzytelnianie oparte na kluczach nie może, nie w żadnym realistycznym przedziale czasowym.

Na swojej **maszynie lokalnej**, wygeneruj parę kluczy:

{% image "/assets/images/blog/pl/jak-zabezpieczyc-ssh-ubuntu-debian/H1.png", "Uruchamianie ssh-keygen -t ed25519 -C \"your-server-label\" aby wygenerować nową parę kluczy SSH", "(max-width: 768px) 100vw, 800px" %}

```bash
ssh-keygen -t ed25519 -C "your-server-label"
```

Użyj `ed25519`, jest szybszy i bezpieczniejszy od starszego algorytmu RSA. Gdy zostaniesz poproszony o hasło, **ustaw je**. Szyfruje klucz prywatny na dysku, więc nawet jeśli ktoś naruszy twój laptop, wciąż nie będzie mógł użyć klucza bez niego.

Skopiuj klucz publiczny na serwer. Zastąp `youruser` swoim rzeczywistym nazwą użytkownika sudo:

{% image "/assets/images/blog/pl/jak-zabezpieczyc-ssh-ubuntu-debian/H2.png", "Uruchamianie ssh-copy-id -i ~/.ssh/id_ed25519.pub youruser@your-server-ip aby skopiować klucz publiczny na serwer", "(max-width: 768px) 100vw, 800px" %}

```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub youruser@your-server-ip
```

**Przetestuj że klucz działa przed przejściem dalej.** Otwórz nowe okno terminala i połącz się. Jeśli logujesz się bez monitu o hasło, klucz jest zainstalowany poprawnie. **Nie zamykaj jeszcze swojej obecnej sesji**, wciąż potrzebujesz jej jako zapasowej jeśli coś pójdzie nie tak w późniejszych krokach.

Opcjonalnie: dodaj wpis do `~/.ssh/config` na swojej maszynie lokalnej dla szybkiego dostępu:

{% image "/assets/images/blog/pl/jak-zabezpieczyc-ssh-ubuntu-debian/H3.png", "Szybkie połączenie z serwerem", "(max-width: 768px) 100vw, 800px" %}

```
Host myserver
    HostName your-server-ip
    User youruser
    IdentityFile ~/.ssh/id_ed25519
```

{% image "/assets/images/blog/pl/jak-zabezpieczyc-ssh-ubuntu-debian/H4.png", "Szybkie połączenie z serwerem", "(max-width: 768px) 100vw, 800px" %}

Po tym, `ssh myserver` to wszystko co musisz wpisać.

## Wyłącz logowanie root

Loguj się przez SSH jako swój użytkownik sudo od tego momentu. Bezpośrednie logowanie root to ryzyko bezpieczeństwa, jeśli twoja sesja zostanie naruszona, atakujący ma pełny nieograniczony dostęp bez dodatkowych kroków.

Otwórz konfigurację demona SSH:

{% image "/assets/images/blog/pl/jak-zabezpieczyc-ssh-ubuntu-debian/H5.png", "Wyłączanie logowania root w sshd_config", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo nano /etc/ssh/sshd_config
```

Znajdź i zaktualizuj tę linię:

```ini
PermitRootLogin no
```

Jeśli kiedykolwiek potrzebujesz dostępu root na serwerze, zaloguj się jako swój użytkownik sudo i uruchom `sudo su` stamtąd.

## Wyłącz uwierzytelnianie hasłem

Twój klucz działa, więc teraz wyeliminuj logowania hasłowe całkowicie:

```bash
sudo nano /etc/ssh/sshd_config
```

Ustaw oba te:

```ini
PasswordAuthentication no
PubkeyAuthentication yes
```

> **Ważne:** Niektóre wersje Ubuntu i Debian ustawiają `PasswordAuthentication` w pliku drop-in pod `/etc/ssh/sshd_config.d/` który **nadpisuje** główną konfigurację. Sprawdź to:
> ```bash
> grep -r "PasswordAuthentication" /etc/ssh/
> ```
> Jeśli widzisz że jest ustawione na `yes` gdziekolwiek w wynikach, edytuj ten konkretny plik, nie główny `sshd_config`.

## Zaostrz kilka dodatkowych ustawień

Podczas gdy masz otwarty `sshd_config`, dodaj te aby dalej zredukować powierzchnię ataku:

```ini
# Rozłącz po 3 nieudanych próbach
MaxAuthTries 3

# Zamykaj nieuwierzytelnione połączenia szybciej
LoginGraceTime 30

# Wyłącz nieużywane funkcje
X11Forwarding no
AllowTcpForwarding no
```

Jeśli tylko konkretne nazwy użytkowników powinny móc logować się zdalnie, dodaj listę dozwolonych:

```ini
AllowUsers youruser
```

Żadne konto niewymienione na liście zostanie odrzucone na poziomie SSH, nawet z ważnym kluczem.

## Zmień domyślny port

Port 22 pojawia się na każdej liście celów domyślnych skanerów. Przeniesienie SSH na niestandardowy port nie powstrzyma zdeterminowanego atakującego od skanowania portów, ale eliminuje praktycznie cały zautomatyzowany szum. Logi autoryzacji spadają z setek nieudanych prób logowania dziennie do efektywnie zera.

W `sshd_config`, zaktualizuj port:

```ini
Port 2222
```

Wybierz dowolny nieużywany port powyżej 1024. **Przed restartem SSH**, zaktualizuj swoją zaporę aby zezwolić na nowy port i zamknąć stary:

```bash
sudo ufw allow 2222/tcp
sudo ufw deny 22/tcp
sudo ufw status
```

Upewnij się że **2222 pokazuje się jako ALLOW** w wynikach przed przejściem dalej.

## Zrestartuj SSH i zweryfikuj

Zastosuj wszystkie swoje zmiany:

```bash
sudo systemctl restart ssh
```

Potem, w **nowym oknie terminala** (zachowaj swoją obecną sesję otwartą), przetestuj połączenie na nowym porcie:

```bash
ssh -p 2222 youruser@your-server-ip
```

Jeśli łączy się czysto, skończone. Jeśli nie, wróć do swojej obecnej sesji i debuguj. Uruchom `sudo sshd -t` aby sprawdzić `sshd_config` pod kątem błędów składni przed ponownym restartowaniem.

Typowe problemy:
- Zapora nie zaktualizowana dla nowego portu
- `PasswordAuthentication no` ustawione w pominiętym pliku który został przeoczony

## Sprawdź co serwer faktycznie widzi

Po zablokowaniu, sprawdź na żywo próby autoryzacji:

```bash
sudo journalctl -u ssh --since "1 hour ago" | grep "Failed"
```

Na poprawnie utwardzonym serwerze powinieneś nic nie widzieć, lub tylko garstkę prób na starym porcie cicho porzucanych przez zaporę.

## Uwaga o fail2ban

Z włączonym uwierzytelnianiem kluczowym SSH i wyłączonym uwierzytelnianiem hasłem, ataki siłowe na SSH są już niemożliwe. fail2ban staje się mniej krytyczny dla samego SSH. Mówiąc to, wciąż jest przydatny do ochrony innych usług jak Nginx i Apache, a uruchamianie go obok tych ustawień dodaje rozsądną warstwę obrony w głąb. Zobacz nasz przewodnik konfiguracji fail2ban [fail2ban setup guide](/pl/blog/konfiguracja-fail2ban-ubuntu-debian/) jeśli chcesz go dodać.

Jeśli chcesz bezpieczne miejsce do poćwiczenia tego procesu utwardzania bez ryzykowania serwera produkcyjnego, nasze plany **[Budget VPS](/pl/budget-vps/)** są przystępną piaskownicą aby zablokować, zepsuć i zacząć od nowa tyle razy ile potrzebujesz.