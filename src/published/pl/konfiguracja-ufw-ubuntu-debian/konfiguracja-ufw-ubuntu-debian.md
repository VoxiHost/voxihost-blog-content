---
image: /assets/images/blog/konfiguracja-ufw-ubuntu-debian/og-image.png
title: 'Jak skonfigurować zaporę UFW na Ubuntu i Debian: Kompletny przewodnik serwera'
description: Kompletny przewodnik dla początkujących do konfiguracji Uncomplicated Firewall (UFW) na serwerach Ubuntu i Debian. Naucz się zezwalać na SSH, blokować ruch i zabezpieczać swój VPS.
date: '2026-03-25'
translationKey: configure-ufw-ubuntu-debian
category: Poradniki
tags:
  - ubuntu
  - debian
  - ufw
  - firewall
  - security
  - linux
  - vps
  - server administration
  - iptables
howto:
  name: Jak skonfigurować zaporę UFW na Ubuntu i Debian
  totalTime: PT10M
  yield: Zabezpieczony serwer Ubuntu lub Debian z poprawnie skonfigurowaną zaporą UFW
  tool:
    - VPS lub dedykowany serwer z Ubuntu lub Debian
    - Klient SSH (np. terminal, PuTTY)
    - Konto użytkownika z uprawnieniami sudo
  steps:
    - name: Zainstaluj UFW (jeśli potrzebne)
      text: Uruchom sudo apt install ufw aby zainstalować Uncomplicated Firewall.
      url: step-1-install-ufw-if-necessary
    - name: Ustaw domyślne polityki
      text: Uruchom sudo ufw default deny incoming i sudo ufw default allow outgoing.
      url: step-2-set-default-policies
    - name: Zezwól na połączenia SSH
      text: Uruchom sudo ufw allow ssh lub sudo ufw allow 2222/tcp jeśli zmieniłeś port.
      url: step-3-allow-ssh-crucial
    - name: Zezwól na HTTP i HTTPS
      text: Uruchom sudo ufw allow http i sudo ufw allow https aby otworzyć porty WWW.
      url: step-4-allow-other-necessary-services
    - name: Włącz zaporę
      text: Uruchom sudo ufw enable aby aktywować zaporę i sudo ufw status aby zweryfikować.
      url: step-5-enable-ufw
status: published
locale: pl
author:
  name: VoxiHost Team
  link: https://voxihost.pl/
contributors:
  - danielmarszalkowski
---

Domyślnie, gdy uruchamiasz usługę na serwerze Linux, wiąże się z portem i zaczyna nasłuchiwać internetu. Jeśli instalujesz bazę danych i nie konfigurujesz jej jawnie aby wiązać się tylko z `localhost`, nagle jest wystawiona na publiczny internet.

Zapora zmienia to z modelu "domyślnie zezwalaj" na model "domyślnie odmawiaj". Zapora odrzuca cały ruch przychodzący *z wyjątkiem* konkretnych portów które jawnie otwierasz.

Na Ubuntu i Debian, standardowym narzędziem do zarządzania zaporą jest **UFW** (Uncomplicated Firewall). To przyjazny dla użytkownika frontend dla `iptables` który przekształca złożone polecenia sieciowe w prosty angielski.

## Krok 1: Zainstaluj UFW (jeśli konieczne)

Sprawdź czy jest tam:
{% image "/assets/images/blog/konfiguracja-ufw-ubuntu-debian/H1.png", "Uruchamianie sudo ufw status na Ubuntu aby sprawdzić czy zapora UFW jest zainstalowana i aktywna", "(max-width: 768px) 100vw, 800px" %}
```bash
sudo ufw status
```

Jeśli mówi "Status: inactive", jest zainstalowana ale wyłączona. Jeśli mówi "command not found", zainstaluj ją:
```bash
sudo apt update
sudo apt install ufw -y
```

## Krok 2: Ustaw domyślne polityki

Zanim zaczniemy otwierać porty, musimy ustalić reguły bazowe. Najbezpieczniejszą bazą jest zablokowanie wszystkiego przychodzącego i zezwolenie na wszystko wychodzące.

{% image "/assets/images/blog/konfiguracja-ufw-ubuntu-debian/H2.png", "Ustawianie domyślnej polityki UFW na odmowę przychodzącego i zezwolenie na wychodzący na Ubuntu Linux", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

Gdy UFW jest włączone, te polecenia zapewniają że nikt nie może połączyć się z twoim serwerem chyba że jawnie otworzysz dla niego dziurę, podczas gdy twój serwer wciąż może sięgać na zewnątrz aby pobierać aktualizacje i wysyłać emaile.

## Krok 3: Zezwól na SSH (KLUCZOWE)

**Nie włączaj jeszcze zapory.** Ponieważ domyślna polityka przychodząca to "odmów", włączenie UFW teraz natychmiast zerwałoby twoje połączenie SSH i zablokowałoby ci dostęp do serwera.

Musisz najpierw jawnie zezwolić na ruch SSH.

{% image "/assets/images/blog/konfiguracja-ufw-ubuntu-debian/H3.png", "Uruchamianie sudo ufw allow ssh na Ubuntu aby zezwolić na połączenia SSH na porcie 22 przed włączeniem zapory", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo ufw allow ssh
```

Otwiera to port 22. Jeśli śledziłeś nasz przewodnik aby [zabezpieczyć SSH i zmienić domyślny port](/pl/blog/jak-zabezpieczyc-ssh-ubuntu-debian/), musisz określić dokładny port i protokół (np., jeśli zmieniłeś go na 2222):

```bash
sudo ufw allow 2222/tcp
```

## Krok 4: Zezwól na inne niezbędne usługi

Teraz otwórz porty dla wszystkiego innego co twój serwer hostuje. Większość usług ma nazwy które UFW rozpoznaje, ale możesz zawsze używać numerów portów bezpośrednio.

Dla serwera WWW (HTTP / HTTPS):
{% image "/assets/images/blog/konfiguracja-ufw-ubuntu-debian/H4.png", "Zezwalanie na port HTTP 80 i HTTPS 443 przez zaporę UFW na serwerze Debian/Ubuntu", "(max-width: 768px) 100vw, 800px" %}
```bash
sudo ufw allow http
sudo ufw allow https
```
*(Co tłumaczy się na porty 80 i 443).*

Dla niestandardowej aplikacji działającej na porcie 8080:
```bash
sudo ufw allow 8080/tcp
```

### Zezwalanie na konkretne adresy IP

Jeśli masz bazę danych (jak MySQL na porcie 3306) do której chcesz uzyskać dostęp ze swojego domowego IP lub innego serwera, ale *nie* z całego internetu, możesz zezwolić na pojedynczy IP:

```bash
sudo ufw allow from 203.0.113.50 to any port 3306
```

## Krok 5: Włącz UFW

Z portem SSH bezpiecznie dozwolonym, czas włączyć zaporę:

{% image "/assets/images/blog/konfiguracja-ufw-ubuntu-debian/H5.png", "Uruchamianie sudo ufw enable aby aktywować zaporę UFW na Ubuntu z aktywnymi regułami", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo ufw enable
```

Zobaczysz ostrzeżenie: `Command may disrupt existing ssh connections. Proceed with operation (y|n)?`. Wpisz `y` i naciśnij Enter.

Ponieważ otworzyłeś port SSH w Kroku 3, twoje połączenie pozostanie aktywne. Sprawdź status aby zobaczyć aktywne reguły:

{% image "/assets/images/blog/konfiguracja-ufw-ubuntu-debian/H6.png", "Uruchamianie sudo ufw status verbose pokazujące aktywne reguły zapory dla SSH, HTTP i HTTPS na Ubuntu", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo ufw status verbose
```

## Jak usuwać reguły

Jeśli popełnisz błąd lub nie potrzebujesz już otwartego portu, możesz usunąć regułę. Najłatwiejszy sposób to wyświetlenie reguł jako numerowanej listy:

{% image "/assets/images/blog/konfiguracja-ufw-ubuntu-debian/H7.png", "Uruchamianie sudo ufw status numbered na Ubuntu aby wyświetlić reguły zapory z numerami do usunięcia", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo ufw status numbered
```

Znajdź numer obok reguły którą chcesz usunąć i usuń ją:

{% image "/assets/images/blog/konfiguracja-ufw-ubuntu-debian/H8.png", "Uruchamianie sudo ufw delete 3 aby usunąć konkretną regułę zapory UFW według numeru na Ubuntu", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo ufw delete 3
```

UFW poprosi o potwierdzenie, a potem reguła zniknie. Pamiętaj że usunięcie linii 3 przesunie numery poniżej w górę, więc zawsze uruchamiaj `status numbered` na nowo przed usunięciem następnej reguły.

## Dalsze kroki

UFW jest idealne dla 99% typowych konfiguracji serwerów. Czysto obsługuje odrzucanie złośliwego tła szumu podczas wystawiania dokładnie tego co zamierzasz wystawić. Jeśli połączysz to z [fail2ban](/pl/blog/konfiguracja-fail2ban-ubuntu-debian/), twój serwer nie tylko odrzuci niechciany ruch, ale aktywnie zbanuje IP które próbują nadużywać twoich otwartych portów.

Jeśli nie masz serwera do ćwiczeń, nasz [Budget VPS](/pl/budget-vps/) jest idealnym, niskokosztowym środowiskiem aby uruchomić, skonfigurować zaporę, psuć rzeczy i zaczynać całkowicie od nowa bez żadnych problemów.