---
image: /assets/images/blog/konfiguracja-firewalld-centos-rhel/og-image.png
title: 'Jak skonfigurować firewalld na AlmaLinux, CentOS, Rocky Linux i Fedora: Kompletny przewodnik serwera'
description: Kompletny przewodnik krok po kroku do konfiguracji firewalld na serwerach AlmaLinux, CentOS Stream, Rocky Linux i Fedora. Naucz się zarządzać strefami, otwierać porty, listować usługi i zabezpieczać swój VPS.
date: '2026-03-25'
translationKey: configure-firewalld-rhel-fedora
category: Poradniki
tags:
  - almalinux
  - centos
  - rocky linux
  - fedora
  - firewalld
  - firewall
  - security
  - linux
  - vps
  - server administration
howto:
  name: Jak skonfigurować firewalld na AlmaLinux, CentOS Stream, Rocky Linux i Fedora
  totalTime: PT10M
  yield: Zabezpieczony serwer rodziny RHEL lub Fedora z poprawnie skonfigurowaną konfiguracją firewalld
  tool:
    - VPS lub dedykowany serwer z AlmaLinux, CentOS Stream, Rocky Linux lub Fedora
    - Klient SSH (np. terminal, PuTTY)
    - Konto użytkownika z uprawnieniami sudo
  steps:
    - name: Uruchom usługę firewalld
      text: Uruchom sudo systemctl enable --now firewalld aby aktywować zaporę sieciową w swoim systemie.
      url: step-1-start-and-enable-firewalld
    - name: Sprawdź aktywne strefy
      text: Uruchom sudo firewall-cmd --get-active-zones aby zobaczyć które interfejsy sieciowe należą do strefy publicznej.
      url: step-2-understanding-zones
    - name: Wyświetl aktualne reguły zapory
      text: Uruchom sudo firewall-cmd --list-all aby przejrzeć aktualnie otwarte porty i usługi.
      url: step-3-check-your-current-rules
    - name: Otwórz porty i usługi
      text: Użyj sudo firewall-cmd --permanent --add-service=http aby trwale zezwolić na ruch.
      url: step-4-allow-services-and-ports
    - name: Przeładuj zaporę
      text: Uruchom sudo firewall-cmd --reload aby zastosować wszystkie trwałe zmiany reguł.
      url: step-5-reload-and-verify
status: published
locale: pl
author:
  name: VoxiHost Team
  link: https://voxihost.pl/
contributors:
  - danielmarszalkowski
---

Jeśli śledziłeś nasz przewodnik dotyczący zabezpieczania Ubuntu, prawdopodobnie nauczyłeś się o UFW. Ale po stronie Red Hat świata Linuksa, AlmaLinux, CentOS Stream, Rocky Linux i Fedora, domyślnym, oficjalnie wspieranym menedżerem zapory sieciowej jest **firewalld**.

Podczas gdy UFW zakłada, że twój serwer ma tylko jedno proste połączenie z internetem, `firewalld` używa koncepcji zwanej "Strefami". Pozwala to na posiadanie całkowicie różnych reguł dla twojego publicznego połączenia internetowego w porównaniu z twoją prywatną siecią lokalną łączącą serwery bazy danych.

Dla standardowego serwera WWW lub VPS jest jednak równie łatwy do skonfigurowania. Oto wszystko co musisz wiedzieć aby zabezpieczyć swój system.

## Krok 1: Uruchom i włącz firewalld

Na większości dystrybucji rodziny RHEL, `firewalld` jest domyślnie zainstalowany ale może nie działać.

{% image "/assets/images/blog/konfiguracja-firewalld-centos-rhel/H1.png", "Uruchamianie sudo systemctl enable --now firewalld na AlmaLinux lub Rocky Linux aby uruchomić i włączyć usługę firewalld przy starcie systemu", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo systemctl enable --now firewalld
sudo systemctl status firewalld
```

Jeśli status to "active (running)", wszystko w porządku. Jeśli nie jest zainstalowany, możesz go pobrać przez menedżer pakietów `dnf`:

```bash
sudo dnf install firewalld -y
sudo systemctl enable --now firewalld
```

## Krok 2: Zrozumienie "Stref"

`firewalld` organizuje reguły w "strefy" na podstawie poziomu zaufania interfejsu sieciowego. Dla VPS podłączonego bezpośrednio do publicznego internetu, cały ruch zazwyczaj przechodzi przez strefę `public`.

Sprawdź w której strefie znajduje się twój główny interfejs sieciowy:

{% image "/assets/images/blog/konfiguracja-firewalld-centos-rhel/H2.png", "Uruchamianie sudo firewall-cmd --get-active-zones na AlmaLinux aby wyświetlić które strefy firewalld są aktywne i które interfejsy są do nich przypisane", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo firewall-cmd --get-active-zones
```

Zobaczysz wynik podobny do tego:
```
public
  interfaces: eth0
```

Potwierdza to, że twoje aktywne połączenie internetowe (`eth0`) używa strefy `public`. Jeśli nie określisz inaczej, wszystkie uruchamiane polecenia będą dotyczyć tej domyślnej strefy.

## Krok 3: Sprawdź swoje aktualne reguły

Zanim zaczniesz dodawać reguły, zobacz co jest już otwarte:

{% image "/assets/images/blog/konfiguracja-firewalld-centos-rhel/H3.png", "Uruchamianie sudo firewall-cmd --list-all na AlmaLinux aby wyświetlić wszystkie aktualnie aktywne reguły zapory i otwarte usługi w strefie publicznej", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo firewall-cmd --list-all
```

Dystrybucje RHEL zazwyczaj mają domyślnie włączone `ssh` i `dhcpv6-client` abyś nie przypadkowo nie zablokował sobie dostępu. Reguła "odrzuć cały ruch przychodzący" jest niejawnie stosowana do wszystkiego *co nie* znajduje się na tej liście.

## Krok 4: Zezwól na usługi i porty

Najczystszym sposobem użycia firewalld jest zezwalanie na predefiniowane **usługi**. "Usługa" w firewalld to po prostu oznaczona reguła dla jednego lub więcej portów.

Na przykład, aby otworzyć serwer na ruch WWW HTTP i HTTPS:

{% image "/assets/images/blog/konfiguracja-firewalld-centos-rhel/H4.png", "Uruchamianie sudo firewall-cmd --permanent --add-service=http aby trwale zezwolić na ruch HTTP przez firewalld na AlmaLinux lub Rocky Linux", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
```

Flaga `--permanent` jest niezwykle ważna. Jeśli ją pominiesz, reguła jest stosowana natychmiast ale znika przy następnym restarcie serwera. Dodając `--permanent`, reguła jest zapisywana na dysku, zapewniając przetrwanie restartu systemu.

### Otwieranie konkretnych portów

Jeśli uruchamiasz niestandardową aplikację, powiedzmy, aplikację Node.js na porcie 3000, użyj formatu portu zamiast tego:

```bash
sudo firewall-cmd --permanent --add-port=3000/tcp
```

### Usuwanie reguł

Dodałeś coś przez pomyłkę? Polecenie odwrotne jest identyczne, po prostu zamień "add" na "remove":

```bash
sudo firewall-cmd --permanent --remove-port=3000/tcp
```

## Krok 5: Przeładuj i zweryfikuj

Ponieważ użyłeś flagi `--permanent` w Kroku 4, żadne z twoich nowych reguł jeszcze nie działa aktywnie. Istnieją tylko na dysku. Aby przesłać zapisane reguły do aktywnego stanu zapory, musisz przeładować:

{% image "/assets/images/blog/konfiguracja-firewalld-centos-rhel/H5.png", "Uruchamianie sudo firewall-cmd --reload na AlmaLinux aby zastosować trwałe zmiany reguł zapory bez restartowania serwera", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo firewall-cmd --reload
```

Na koniec uruchom `--list-all` ponownie aby zweryfikować, że twoje nowe polityki są na miejscu:

{% image "/assets/images/blog/konfiguracja-firewalld-centos-rhel/H6.png", "Uruchamianie sudo firewall-cmd --list-all po przeładowaniu na AlmaLinux aby potwierdzić że nowe reguły HTTP i HTTPS są teraz aktywne", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo firewall-cmd --list-all
```

Powinieneś zobaczyć nowo dodane usługi i porty pod liniami `services:` i `ports:`.

## Zaawansowane: Zezwalanie na konkretne IP

Co jeśli masz bazę danych MySQL na porcie `3306` i chcesz aby tylko twój serwer aplikacyjny (`203.0.113.55`) miał do niej dostęp, nie cały internet?

Możesz użyć "Rich Rule" aby trwale powiązać port z konkretnym źródłowym IP:

```bash
sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="203.0.113.55" port protocol="tcp" port="3306" accept'
sudo firewall-cmd --reload
```

## Podsumowanie

Po skonfigurowaniu, `firewalld` działa cicho w tle zabezpieczając system. Ponieważ natywnie integruje się z rdzeniem Fedory, Rocky, CentOS i AlmaLinux, elegancko obsługuje złożone operacje takie jak mostkowanie kontenerów czy routowanie sieci wewnętrznych. Połączone z [fail2ban](/pl/blog/konfiguracja-fail2ban-centos-rhel/), twój serwer jest wysoce odporny na ogólne próby siłowe botów internetowych.

Aby przetestować swoje umiejętności bezpieczeństwa w aktywnym internecie bez ryzykowania jakichkolwiek produkcyjnych danych, sprawdzenie tymczasowego, niskopoziomowego [Budget VPS](/pl/budget-vps/) jest taniym, niezawodnym sposobem na praktykowanie konfiguracji nieprzeniknionego systemu RHEL.