---
image: /assets/images/blog/pl/konfiguracja-fail2ban-centos-rhel/og-image.png
title: 'Jak skonfigurować fail2ban na AlmaLinux, CentOS, Rocky Linux i Fedora: Kompletny przewodnik serwera'
description: Kompletny przewodnik do instalacji i konfiguracji fail2ban na serwerach AlmaLinux, CentOS Stream, Rocky Linux i Fedora. Chroń SSH i usługi WWW przed atakami siłowymi z automatycznym banowaniem IP i integracją z firewalld.
date: '2026-03-25'
translationKey: setup-fail2ban-rhel-fedora
category: Poradniki
tags:
  - fail2ban
  - almalinux
  - centos
  - rocky linux
  - fedora
  - security
  - vps
  - brute force protection
  - ssh
  - firewalld
  - intrusion prevention
howto:
  name: Jak skonfigurować fail2ban na AlmaLinux, CentOS Stream, Rocky Linux i Fedora
  totalTime: PT10M
  yield: Serwer AlmaLinux, CentOS Stream, Rocky Linux lub Fedora chroniony przez fail2ban z aktywnym więzieniem SSH i integracją z firewalld
  tool:
    - VPS lub dedykowany serwer z AlmaLinux, CentOS Stream, Rocky Linux lub Fedora
    - Dostęp SSH z uprawnieniami sudo lub root
  steps:
    - name: Zainstaluj fail2ban
      text: Uruchom sudo dnf install fail2ban -y aby zainstalować fail2ban z repozytorium EPEL.
      url: instalowanie-fail2ban
    - name: Utwórz lokalną konfigurację więzienia
      text: 'Skopiuj jail.conf do jail.local za pomocą: sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local'
      url: utworzenie-lokalnej-konfiguracji-wiezienia
    - name: Skonfiguruj dla backendu firewalld
      text: Ustaw banaction = firewallcmd-ipset w jail.local aby zintegrować fail2ban z firewalld.
      url: integracja-firewalld
    - name: Skonfiguruj więzienie SSH
      text: Włącz i dostroś więzienie sshd w jail.local z pożądanymi ustawieniami banowania.
      url: wiezienie-ssh
    - name: Włącz i uruchom fail2ban
      text: Uruchom sudo systemctl enable --now fail2ban aby uruchomić fail2ban i włączyć go przy starcie.
      url: wlaczenie-i-uruchomienie-fail2ban
    - name: Zweryfikuj że więzienie SSH jest aktywne
      text: Uruchom sudo fail2ban-client status sshd aby potwierdzić aktywne monitorowanie i banowanie.
      url: weryfikacja-dzialania-i-aktywnego-wiezienia
status: published
locale: pl
author:
  name: VoxiHost Team
  link: https://voxihost.pl/
contributors:
  - danielmarszalkowski
---

fail2ban monitoruje logi autoryzacji i automatycznie banuje adresy IP które gromadzą zbyt wiele nieudanych prób logowania. Na serwerze z publicznym IP, cicho blokuje setki zautomatyzowanych skanerów które w przeciwnym razie przetaczałyby przez twoje porty w poszukiwaniu słabych danych uwierzytelniających.

Dla serwerów działających na AlmaLinux, CentOS Stream, Rocky Linux lub Fedorze, konfiguracja jest prawie identyczna z dowolnym innym systemem Linux. Jedna znacząca różnica to backend zapory: te dystrybucje używają `firewalld`, i fail2ban musi o tym wiedzieć tak aby wstawiaj bany przez `firewalld` zamiast próbować surowe polecenia `iptables` które mogą być w konflikcie.

Podczas gdy dostawcy premium jak **<span class="text-white">Voxi</span><span class="text-amber-300">Host</span>** preinstalują fail2ban na swoich szablonach z działającą konfiguracją bazową, większość domyślnych obrazów Linuksa tego nie robi. Ten przewodnik przeprowadzi przez instalację i dostrojenie go do twoich specyficznych potrzeb.

## Instalowanie fail2ban

Na systemach opartych na RHEL, fail2ban jest dostępny z EPEL (Extra Packages for Enterprise Linux). Jeśli nie jest jeszcze zainstalowany:

{% image "/assets/images/blog/pl/konfiguracja-fail2ban-centos-rhel/H1.png", "Uruchamianie sudo dnf install epel-release -y na AlmaLinux, CentOS, Rocky Linux i Fedorze aby zainstalować epel-release", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo dnf install epel-release -y
sudo dnf install fail2ban -y
```

Na Fedorze, jest w głównych repozytoriach:

```bash
sudo dnf install fail2ban -y
```

Włącz i uruchom go:

{% image "/assets/images/blog/pl/konfiguracja-fail2ban-centos-rhel/H2.png", "Uruchamianie sudo systemctl enable --now fail2ban na AlmaLinux, CentOS, Rocky Linux i Fedorze aby włączyć i uruchomić fail2ban", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo systemctl enable --now fail2ban
```

## Utworzenie lokalnej konfiguracji więzienia

Zachowanie fail2ban jest kontrolowane przez "więzienia", każdy z nich obserwuje konkretny log pod kątem wzorców niepowodzeń i banuje obciążające IP.

**Nie edytuj bezpośrednio `/etc/fail2ban/jail.conf`.** Aktualizacje pakietów go nadpisują. Zamiast tego utwórz lokalne nadpisanie:

```bash
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo nano /etc/fail2ban/jail.local
```

### Ustawienia globalne

```ini
[DEFAULT]
# Nigdy nie banuj tych IP
ignoreip = 127.0.0.1/8 ::1 TWOJ.DOMOWY.IP.ADRES
# Czas trwania banu w sekundach (86400 = 24 godziny, -1 = stały)
bantime = 3600
# Okno do zliczania niepowodzeń w
findtime = 600
# Niepowodzenia przed banem
maxretry = 5
# Użyj dziennik systemd (poprawny backend dla tych dystrybucji)
backend = systemd
```

Dodaj swój domowy IP do `ignoreip`, oszczędza cię przed frustrującą samoblokadą.

### Integracja firewalld

To jest krytyczna różnica od systemów opartych na Debian. fail2ban domyślnie używa `iptables`, które są w konflikcie z `firewalld`. Ustaw poprawny backend:

```ini
[DEFAULT]
# Użyj firewalld do banowania (wymagane na RHEL/Fedora)
banaction = firewallcmd-ipset
banaction_allports = firewallcmd-allports
```

Bez tego, fail2ban może wydawać że działa ale tak naprawdę nic nie blokuje, lub tworzy reguły iptables których firewalld ignoruje.

### Więzienie SSH

Znajdź lub dodaj sekcję `[sshd]`:

```ini
[sshd]
enabled = true
port = ssh
# Jeśli zmieniłeś port SSH:
# port = 2222
filter = sshd
logpath = %(sshd_log)s
maxretry = 3
bantime = 86400
```

Trzy uderzenia i jesteś na 24 godzin to rozsądna polityka dla SSH.

## Włączenie i uruchomienie fail2ban

Zastosuj konfigurację:

```bash
sudo systemctl restart fail2ban
```

## Weryfikacja działania i aktywnego więzienia

Sprawdź status więzienia SSH:

{% image "/assets/images/blog/pl/konfiguracja-fail2ban-centos-rhel/H3.png", "Uruchamianie sudo fail2ban-client status sshd na AlmaLinux, CentOS, Rocky Linux i Fedorze aby sprawdzić status więzienia SSH", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo fail2ban-client status sshd
```

Oczekiwany wynik:
```
Status for the jail: sshd
|- Filter
|  |- Currently failed: 1
|  |- Total failed: 34
|  `- Journal matches: _SYSTEMD_UNIT=sshd.service
`- Actions
|  |- Currently banned: 2
|  |- Total banned: 8
|  `- Banned IP list: 203.0.113.45 198.51.100.12
```

Jeśli `Currently banned` jest niezerowe, coś już próbowało i niepowiodło przeciwko twojemu SSH. Dobrze, działa.

Zweryfikuj że firewalld faktycznie egzekwuje bany:

```bash
sudo firewall-cmd --list-rich-rules | grep fail2ban
```

Powinieneś zobaczyć wymienione reguły `fail2ban-sshd`. Jeśli to polecenie nic nie zwraca, `banaction` nie było ustawione poprawnie, wróć i sprawdź `jail.local`.

## Odbanowanie IP

Aby natychmiast usunąć konkretny ban:

```bash
sudo fail2ban-client set sshd unbanip 203.0.113.45
```

Nie jest wymagany restart. Reguła firewalld jest usuwana na miejscu.

## Sprawdzanie logów

Obserwuj co fail2ban robi w czasie rzeczywistym:

```bash
sudo tail -f /var/log/fail2ban.log
```

Na publicznym serwerze zobaczysz że to szybko się zapełnia. Zdarzenia banowania, zdarzenia odbanowania i okazjonalne błędy jeśli coś jest źle skonfigurowane. Jeśli przestałeś widzieć bany ale wiesz że SSH jest ciągle atakowane, sprawdź czy fail2ban wciąż działa:

{% image "/assets/images/blog/pl/konfiguracja-fail2ban-centos-rhel/H4.png", "Uruchamianie sudo systemctl status fail2ban na AlmaLinux, CentOS, Rocky Linux i Fedorze aby sprawdzić status fail2ban", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo systemctl status fail2ban
```

## Ochrona usług WWW

fail2ban dostarcza filtry dla Nginx i Apache. Dodaj więzienia dla nich:

```ini
[nginx-http-auth]
enabled = true
filter = nginx-http-auth
port = http,https
logpath = /var/log/nginx/error.log
maxretry = 5

[nginx-limit-req]
enabled = true
filter = nginx-limit-req
port = http,https
logpath = /var/log/nginx/error.log
maxretry = 10
```

Zrestartuj po dodaniu więzień:

```bash
sudo systemctl restart fail2ban
sudo fail2ban-client status
```

Polecenie `status` pokazuje wszystkie aktywne więzienia. Każde które włączyłeś powinno pojawić się z własnym licznikiem.

## Uwaga o SELinux

Na systemach gdzie SELinux jest egzekwowane (co jest domyślne), fail2ban generalnie działa bez problemów ponieważ wchodzi w interakcję z firewalld na wyższym poziomie. Jeśli zobaczysz odmowy uprawnień w `/var/log/audit/audit.log` związane z fail2ban, sprawdź:

```bash
sudo ausearch -m avc -ts recent | grep fail2ban
```

Najczęstsze problemy są rozwiązywane przez instalowanie pakietu `fail2ban` przez oficjalne repozytoria (które zawierają poprawne konteksty SELinux) zamiast instalacji ręcznej.

Jeśli chcesz czysty serwer oparty na RHEL do przetestowania tej konfiguracji, nasze plany [Budget VPS](/pl/budget-vps/) pozwalają cię uruchomić, skonfigurować i eksperymentować bez dotykania czegokolwiek ważnego.