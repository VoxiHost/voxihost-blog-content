---
image: /assets/images/blog/pl/konfiguracja-fail2ban-ubuntu-debian/og-image.png
title: 'Jak skonfigurować fail2ban na Ubuntu i Debian: Kompletny przewodnik serwera'
description: Kompletny przewodnik do instalacji i konfiguracji fail2ban na serwerach Ubuntu i Debian. Chroń SSH i usługi WWW przed atakami siłowymi z automatycznym banowaniem IP, niestandardowymi więzieniami i integracją z ufw.
date: '2026-03-25'
updated: '2026-06-02'
translationKey: setup-fail2ban-ubuntu-debian
category: Poradniki
tags:
  - fail2ban
  - ubuntu
  - debian
  - security
  - vps
  - brute force protection
  - ssh
  - ufw
  - intrusion prevention
howto:
  name: Jak skonfigurować fail2ban na Ubuntu i Debian
  totalTime: PT10M
  yield: Serwer Ubuntu lub Debian chroniony przez fail2ban z aktywnym więzieniem SSH i automatycznym banowaniem IP
  tool:
    - VPS lub dedykowany serwer z Ubuntu lub Debian
    - Dostęp SSH z uprawnieniami sudo lub root
  steps:
    - name: Zainstaluj fail2ban
      text: Uruchom sudo apt install fail2ban -y aby zainstalować fail2ban z domyślnego repozytorium.
      url: instalowanie-fail2ban
    - name: Utwórz lokalną konfigurację więzienia
      text: 'Skopiuj jail.conf do jail.local za pomocą: sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local'
      url: utworzenie-lokalnej-konfiguracji-wiezienia
    - name: Skonfiguruj więzienie SSH
      text: Edytuj jail.local aby włączyć i dostroić więzienie sshd z pożądanymi ustawieniami banowania.
      url: wiezienie-ssh
    - name: Włącz i uruchom fail2ban
      text: Uruchom sudo systemctl enable --now fail2ban aby uruchomić fail2ban i włączyć go przy starcie.
      url: wlaczenie-i-uruchomienie-fail2ban
    - name: Zweryfikuj że więzienie SSH jest aktywne
      text: Uruchom sudo fail2ban-client status sshd aby potwierdzić że więzienie monitoruje SSH i liczy nieudane próby.
      url: weryfikacja-dzialania-i-aktywnego-wiezienia
faq:
  - question: "Co to jest fail2ban i jak chroni mój serwer?"
    answer: "Fail2ban to oprogramowanie zapobiegające włamaniom, które monitoruje logi systemowe (np. logi uwierzytelniania) pod kątem powtarzających się błędnych prób logowania. Po wykryciu ataku typu brute-force, program automatycznie dodaje tymczasową regułę do zapory sieciowej, blokując złośliwy adres IP."
  - question: "Dlaczego powinno się edytować plik jail.local zamiast jail.conf?"
    answer: "Plik <code>jail.conf</code> jest plikiem domyślnym dostarczanym przez system pakietów i zostanie nadpisany podczas aktualizacji fail2ban. Zapisanie własnych reguł w pliku <code>jail.local</code> gwarantuje, że nie zostaną one utracone podczas aktualizacji."
  - question: "Jak sprawdzić listę adresów IP aktualnie zablokowanych przez fail2ban?"
    answer: "Listę zablokowanych adresów IP dla konkretnego filtra (np. dla SSH) sprawdzisz w terminalu wpisując polecenie: <code>sudo fail2ban-client status sshd</code>."
  - question: "Jak ręcznie odblokować (odbanować) adres IP?"
    answer: "Możesz ręcznie usunąć blokadę dla danego adresu IP za pomocą narzędzia klienckiego fail2ban: <code>sudo fail2ban-client set sshd unbanip TWÓJ_ADRES_IP</code>."
  - question: "Jak fail2ban integruje się z zaporą UFW na systemach Ubuntu/Debian?"
    answer: "Na systemach Ubuntu/Debian fail2ban automatycznie wykrywa obecność zapory UFW. W razie wykrycia naruszeń reguł, fail2ban dynamicznie dodaje tymczasowe reguły blokujące (deny) bezpośrednio do tabel UFW."
status: published
locale: pl
author:
  name: VoxiHost Team
  link: https://voxihost.pl/
contributors:
  - danielmarszalkowski
---

fail2ban monitoruje logi autoryzacji i automatycznie banuje adresy IP które gromadzą zbyt wiele nieudanych prób logowania. To jedno z tych narzędzi które działa cicho w tle i ujawnia się tylko gdy sprawdzasz listę banów i zdajesz sobie sprawę że zablokowało setki adresów które młotały twój port SSH w poszukiwaniu słabych danych uwierzytelniających.

To nie jest srebrna kula, jeśli już skonfigurowałeś uwierzytelnianie oparte na kluczach SSH z wyłączonymi hasłami, ataki siłowe przeciwko SSH są już bezużyteczne. Ale fail2ban obejmuje wszystko inne: usługi WWW, serwery pocztowe, dowolna aplikacja która loguje niepowodzenia autoryzacji. I dla serwerów gdzie uwierzytelnianie hasłowe jest wciąż używane dla niektórych usług, jest to praktyczna pierwsza linia obrony.

fail2ban jest często nie instalowany domyślnie na świeżych obrazach Linuksa, chociaż dostawcy premium jak **<span class="text-white">Voxi</span><span class="text-amber-300">Host</span>** preinstalują go na swoich szablonach z rozsądną konfiguracją bazową. Jeśli musisz go zainstalować od zera lub dostroić do swoich specyficznych potrzeb, ten przewodnik to obejmuje również.

## Instalowanie fail2ban

Jeśli nie jest jeszcze zainstalowany:

{% image "/assets/images/blog/pl/konfiguracja-fail2ban-ubuntu-debian/H1.png", "Uruchamianie sudo apt install fail2ban -y na Ubuntu lub Debian aby zainstalować fail2ban z repozytorium apt", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo apt update
sudo apt install fail2ban -y
```

Po zainstalowaniu, usługa uruchamia się automatycznie. Zweryfikuj:

{% image "/assets/images/blog/pl/konfiguracja-fail2ban-ubuntu-debian/H2.png", "Uruchamianie sudo systemctl status fail2ban na Ubuntu aby zweryfikować że usługa fail2ban jest aktywna i działająca", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo systemctl status fail2ban
```

## Utworzenie lokalnej konfiguracji więzienia

fail2ban działa przez "więzienia", każdy z nich obserwuje konkretny plik logów pod kątem wzorców niepowodzeń i banuje obciążające IP.

**Nigdy nie edytuj bezpośrednio `/etc/fail2ban/jail.conf`.** Ten plik jest nadpisywany przez aktualizacje pakietów. Zamiast tego utwórz lokalne nadpisanie:

```bash
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo nano /etc/fail2ban/jail.local
```

Teraz edytuj `jail.local`:

```bash
sudo nano /etc/fail2ban/jail.local
```

### Ustawienia globalne

Na górze pliku ustaw globalne ustawienia które stosują się do wszystkich więzień:

```ini
[DEFAULT]
# Biała lista IP, nigdy nie banuj tych
ignoreip = 127.0.0.1/8 ::1 TWOJ.DOMOWY.IP.ADRES
# Czas trwania banu w sekundach (3600 = 1 godzina, -1 = stały)
bantime = 3600
# Okno do zliczania niepowodzeń w
findtime = 600
# Niepowodzenia przed banem
maxretry = 5
# Użyj dziennik systemd (lepszy backend dla nowoczesnych Ubuntu/Debian)
backend = systemd
```

Dodaj swój domowy IP do `ignoreip`, oszczędza cię przed frustrującą samoblokadą.

### Integracja z ufw

Domyślnie fail2ban używa `iptables` do banowania IP. Jeśli używasz `ufw`, powiedz fail2ban aby używał go zamiast tego dla spójności:

W `jail.local` pod `[DEFAULT]`:

```ini
banaction = ufw
```

To wstawia reguły `ufw` deny dla zbanowanych IP, co ładnie współgra z twoją istniejącą konfiguracją zapory.

### Więzienie SSH

Przewiń w dół do sekcji `[sshd]` lub dodaj ją:

```ini
[sshd]
enabled = true
port = ssh
# Jeśli zmieniłeś port SSH, umieść go tutaj:
# port = 2222
filter = sshd
logpath = %(sshd_log)s
maxretry = 3
bantime = 86400
```

Krótszy `maxretry` i dłuższy `bantime` niż ustawienia globalne są rozsądne dla SSH, trzy nieudane próby w oknie banują na cały dzień.

## Włączenie i uruchomienie fail2ban

Zastosuj konfigurację:

```bash
sudo systemctl restart fail2ban
```

## Weryfikacja działania i aktywnego więzienia

Sprawdź status więzienia SSH:

{% image "/assets/images/blog/pl/konfiguracja-fail2ban-ubuntu-debian/H3.png", "Uruchamianie sudo fail2ban-client status sshd na Ubuntu aby sprawdzić ile adresów IP jest zbanowanych w więzieniu SSH", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo fail2ban-client status sshd
```

Oczekiwany wynik:
```
Status for the jail: sshd
|- Filter
|  |- Currently failed: 2
|  |- Total failed: 47
|  `- File list: /var/log/auth.log
`- Actions
|  |- Currently banned: 3
|  |- Total banned: 12
|  `- Banned IP list: 203.0.113.1 198.51.100.5 ...
```

To "currently banned" i "total banned" to dowód że działa.

## Odbanowanie IP

Jeśli przypadkowo zbanowałeś się lub legalnego użytkownika:

```bash
sudo fail2ban-client set sshd unbanip 203.0.113.1
```

Zastąp IP tym który musisz odbanować. Zmiany wchodzą w życie natychmiast, nie jest wymagany restart.

## Sprawdzanie logów fail2ban

Obserwuj co fail2ban robi w czasie rzeczywistym:

```bash
sudo tail -f /var/log/fail2ban.log
```

Na serwerze publicznym zobaczysz że to szybko się zapełnia. Zdarzenia banowania, odbanowania i okazjonalne błędy jeśli coś jest źle skonfigurowane. Jeśli przestałeś widzieć bany ale wiesz że SSH jest ciągle atakowane, sprawdź czy fail2ban wciąż działa:

```bash
sudo systemctl status fail2ban
```

## Ochrona Nginx i Apache

fail2ban dostarcza filtry dla popularnych usług WWW. Aby dodać ochronę Nginx:

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

Dla Apache:

```ini
[apache-auth]
enabled = true
filter = apache-auth
port = http,https
logpath = /var/log/apache2/error.log
maxretry = 5
```

Zrestartuj po dodaniu więzień:

```bash
sudo systemctl restart fail2ban
sudo fail2ban-client status
```

Ostatnie polecenie `status` pokazuje wszystkie aktywne więzienia i ich liczniki banów.

Jeśli chcesz czysty serwer VPS do przetestowania tej konfiguracji przed wdrożeniem produkcyjnym, nasze plany [Budget VPS](/pl/budget-vps/) są wystarczająco tanie aby przejść przez całą konfigurację bez ryzyka.