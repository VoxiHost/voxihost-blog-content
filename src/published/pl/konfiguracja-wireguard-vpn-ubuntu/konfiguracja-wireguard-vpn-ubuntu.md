---
image: /assets/images/blog/pl/konfiguracja-wireguard-vpn-ubuntu/og-image.png
title: Jak skonfigurować serwer VPN WireGuard na Ubuntu i Debian
description: 'Kompletny poradnik dla początkujących: instalacja, konfiguracja i generowanie kluczy dla błyskawicznego serwera VPN WireGuard na Twoim serwerze Linux VPS.'
date: '2026-03-25'
updated: '2026-06-02'
translationKey: setup-wireguard-vpn-ubuntu-debian
locale: pl
category: Poradniki
tags:
  - wireguard
  - vpn
  - ubuntu
  - debian
  - linux
  - vps
  - security
  - networking
howto:
  name: Jak zainstalować serwer VPN WireGuard za pomocą skryptu auto-instalacyjnego Angristan
  totalTime: PT10M
  yield: Wysoce bezpieczne, prywatne połączenie VPN gotowe do użycia na telefonie lub komputerze
  tool:
    - Serwer VPS lub dedykowany z systemem Ubuntu lub Debian
    - Klient SSH (np. terminal, PuTTY)
    - Konto użytkownika z uprawnieniami sudo
  steps:
    - name: Pobierz skrypt instalacyjny VPN
      text: 'Pobierz zaufany skrypt WireGuard autorstwa Angristan za pomocą polecenia: curl -O https://raw.githubusercontent.com/angristan/wireguard-install/master/wireguard-install.sh'
      url: krok-1-pobierz-zaufany-skrypt-instalacyjny
    - name: Uruchom auto-instalator
      text: Nadaj skryptowi uprawnienia do wykonywania (chmod +x) i uruchom go za pomocą sudo ./wireguard-install.sh.
      url: krok-2-uruchom-auto-instalator
    - name: Odpowiedz na pytania konfiguracyjne
      text: Zaakceptuj domyślne ustawienia sieciowe automatycznie wykryte przez skrypt.
      url: krok-3-pytania-konfiguracyjne
    - name: Wygeneruj pierwszy profil klienta
      text: Podaj nazwę dla swojego pierwszego urządzenia (np. MojaKomorka), aby wygenerować plik .conf i kod QR.
      url: krok-4-wygeneruj-pierwszy-profil-klienta
    - name: Połącz swoje urządzenia
      text: Pobierz aplikację WireGuard na telefon lub PC i zeskanuj kod QR, aby połączyć się bezpiecznie.
      url: krok-5-polacz-swoje-urzadzenia
faq:
  - question: "Dlaczego WireGuard jest lepszy od OpenVPN?"
    answer: "WireGuard to nowoczesny protokół VPN, który jest znacznie szybszy, zużywa mniej danych, ma prostszy kod źródłowy oraz w znacznie mniejszym stopniu obciąża baterię urządzeń mobilnych w porównaniu do OpenVPN."
  - question: "Czy skrypt instalacyjny WireGuard autorstwa Angristana jest bezpieczny?"
    answer: "Tak. Skrypt instalacyjny Angristana to otwartoźródłowe, powszechnie sprawdzane przez społeczność narzędzie. Automatyzuje ono instalację pakietów, konfigurację routingu i reguł zapory sieciowej, co zapobiega błędom ręcznej konfiguracji."
  - question: "Jak dodać nowego użytkownika (klienta) po wstępnej instalacji?"
    answer: "Nie musisz instalować VPN od nowa. Uruchom ponownie skrypt poleceniem <code>sudo ./wireguard-install.sh</code>. Skrypt wykryje instalację i wyświetli menu zarządzania, w którym wybierając opcję 1 wygenerujesz nowego klienta."
  - question: "Jak całkowicie odinstalować WireGuard z serwera?"
    answer: "Aby usunąć WireGuard wraz ze wszystkimi pakietami i konfiguracją, uruchom ponownie skrypt instalacyjny (<code>sudo ./wireguard-install.sh</code>) i wybierz opcję odinstalowania (uninstall)."
  - question: "Czy mogę połączyć kilka urządzeń przy użyciu jednego pliku konfiguracyjnego?"
    answer: "Nie. WireGuard opiera się na relacji peer-to-peer 1:1. Jeśli połączysz dwa urządzenia przy użyciu tego samego pliku konfiguracyjnego, połączenia będą się nawzajem rozłączać. Każde urządzenie musi mieć wygenerowany osobny profil."
status: published
author:
  name: VoxiHost Team
  link: https://voxihost.pl/
contributors:
  - danielmarszalkowski
---

**WireGuard** to nowoczesny, rewolucyjny protokół VPN, który całkowicie zdominował rynek prywatności online. Jest znacznie szybszy i bezpieczniejszy od starszych standardów, takich jak OpenVPN, a połączenie nawiązywane jest niemal błyskawicznie, bez nadmiernego obciążania baterii urządzeń mobilnych.

Chociaż *można* ręcznie konfigurować tablice IP i reguły NAT, aby go zainstalować, dla większości użytkowników jest to proces skomplikowany i podatny na błędy. 

Zamiast tego, globalna społeczność open-source polega na sprawdzonym, powszechnie zaufanym skrypcie bash autorstwa dewelopera o pseudonimie *Angristan*. Pozwala on na bezproblemową i bezpieczną konfigurację WireGuard na dowolnym serwerze VPS w mniej niż dwie minuty.

Oto jak wdrożyć własny, prywatny VPN na dowolnym serwerze Linux VPS, aby omijać ograniczenia i przeglądać sieć bezpiecznie.

## Krok 1: Pobierz zaufany skrypt instalacyjny

Zaloguj się na swój serwer przez SSH. Przed przystąpieniem do instalacji upewnij się, że Twój system jest aktualny:

{% image "/assets/images/blog/pl/konfiguracja-wireguard-vpn-ubuntu/H1.png", "Uruchamianie sudo apt update i apt upgrade -y na Ubuntu w celu aktualizacji systemu przed instalacją VPN WireGuard", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo apt update && sudo apt upgrade -y
```

Następnie pobierz oficjalny skrypt instalacyjny bezpośrednio z repozytorium GitHub Angristan:

{% image "/assets/images/blog/pl/konfiguracja-wireguard-vpn-ubuntu/H2.png", "Pobieranie skryptu auto-instalacyjnego WireGuard z GitHub za pomocą curl na Ubuntu VPS", "(max-width: 768px) 100vw, 800px" %}

```bash
curl -O https://raw.githubusercontent.com/angristan/wireguard-install/master/wireguard-install.sh
```

## Krok 2: Uruchom auto-instalator

Zanim będziesz mógł uruchomić pobrany plik, **musisz** nadać mu uprawnienia do wykonywania:

{% image "/assets/images/blog/pl/konfiguracja-wireguard-vpn-ubuntu/H3.png", "Uruchamianie sudo chmod, a następnie sudo ./wireguard-install.sh, aby włączyć instalator WireGuard na Ubuntu", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo chmod +x wireguard-install.sh
```

Teraz uruchom skrypt z uprawnieniami roota:

```bash
sudo ./wireguard-install.sh
```

## Krok 3: Pytania konfiguracyjne

{% image "/assets/images/blog/pl/konfiguracja-wireguard-vpn-ubuntu/H4.png", "Pytania konfiguracyjne skryptu WireGuard pokazujące adres IP, port i opcje resolvera DNS na Ubuntu", "(max-width: 768px) 100vw, 800px" %}

Największą zaletą tego skryptu jest fakt, że automatycznie wykrywa on interfejsy sieciowe serwera, publiczne adresy IP oraz konfiguracje DNS.

Podczas uruchamiania skryptu zostaniesz poproszony o potwierdzenie kilku ustawień. **W 99% przypadków powinieneś po prostu nacisnąć `Enter`, aby zaakceptować domyślne wartości dla każdego pytania.**

Pytania będą wyglądać mniej więcej tak:
```text
IPv4 or IPv6 public address: (Automatycznie wypełniony Twoim IP)
Public interface: (Automatycznie wypełniony, zazwyczaj eth0 lub enp3s0)
WireGuard port: [51820]
First DNS resolver to use for the clients: [1.1.1.1]
```

Przejdź przez nie, naciskając `Enter`. Skrypt szybko pobierze pakiety `wireguard`, skonfiguruje złożone reguły przekierowania ruchu IP w jądrze Linux, ustawi firewall i wygeneruje główne klucze szyfrujące serwera.

## Krok 4: Wygeneruj pierwszy profil klienta

{% image "/assets/images/blog/pl/konfiguracja-wireguard-vpn-ubuntu/H5.png", "Skrypt WireGuard proszący o nazwę klienta i generujący pierwszy plik .conf oraz kod QR na Ubuntu", "(max-width: 768px) 100vw, 800px" %}

WireGuard wykorzystuje bezpieczną kryptografię peer-to-peer. Aby połączyć telefon lub laptop z VPN, musisz wygenerować plik konfiguracyjny klienta (`.conf`) dla każdego urządzenia.

Zaraz po zakończeniu instalacji pakietów, skrypt automatycznie poprosi o utworzenie pierwszego klienta:

```text
Client name: 
```
Wpisz rozpoznawalną nazwę bez spacji, np. `marcin_iphone` lub `moj_macbook`, i naciśnij Enter.

```text
Client's DNS server: [1]
```
Naciśnij Enter, aby zaakceptować domyślny DNS (zazwyczaj Cloudflare lub Google).

Skrypt wykona teraz coś niezwykle pomocnego: nie tylko utworzy plik `.conf` w Twoim folderze domowym, ale także wyświetli **ogromny kod QR** bezpośrednio w oknie terminala przy użyciu znaków ASCII!

## Krok 5: Połącz swoje urządzenia

### Podłączanie telefonu komórkowego (iOS/Android):
Podłączenie telefonu jest banalnie proste:
1. Przejdź do App Store lub Google Play Store.
2. Pobierz oficjalną, darmową aplikację **WireGuard**.
3. Otwórz aplikację i kliknij ikonę `+`, aby dodać nowy tunel.
4. Wybierz **"Utwórz z kodu QR"** (Create from QR code).
5. Skieruj aparat telefonu na wielki kod QR wyświetlony w Twoim terminalu.

Nazwij połączenie, przesuń przełącznik na "On" i gotowe - od teraz cały ruch Twojego telefonu jest szyfrowany przez Twój własny serwer VPS!

### Podłączanie laptopa lub PC (Windows/Mac/Linux):
Laptopy nie mogą tak łatwo skanować kodów QR z terminala. Zamiast tego musisz pobrać wygenerowany plik `.conf`.

Jeśli nazwałeś swojego klienta `moj_macbook`, skrypt zapisał plik o nazwie `moj_macbook.conf` w katalogu, z którego go uruchomiłeś (zazwyczaj `/home/uzytkownik/` lub `/root/`).

1. Pobierz plik `moj_macbook.conf` na swój komputer osobisty. (Najprościej zrobić to bezpiecznie za pomocą [klienta SFTP, takiego jak FileZilla](/pl/blog/przesylanie-plikow-vps-sftp-filezilla/) lub WinSCP).
2. Pobierz oficjalną aplikację **WireGuard** dla Windows lub Mac ze strony producenta.
3. Kliknij "Import tunnel(s) from file" i wybierz plik `.conf`.
4. Kliknij "Activate". Twoje połączenie jest teraz zabezpieczone!

## Generowanie kolejnych klientów

Jeśli chcesz dodać drugi laptop, smart TV lub udostępnić bezpieczne połączenie członkowi zespołu, nie musisz ponownie instalować WireGuarda.

Po prostu uruchom skrypt ponownie:

{% image "/assets/images/blog/pl/konfiguracja-wireguard-vpn-ubuntu/H6.png", "Ponowne uruchomienie skryptu instalacyjnego WireGuard w celu otwarcia menu zarządzania i dodania kolejnych klientów VPN", "(max-width: 768px) 100vw, 800px" %}

```bash
sudo ./wireguard-install.sh
```

Ponieważ WireGuard jest już zainstalowany, skrypt zmieni się w przejrzyste menu zarządzania. 

Wybierz opcję `1`, aby błyskawicznie wygenerować kolejny plik `.conf` i kod QR.

Jeśli szukasz stabilnego środowiska do testowania konfiguracji WireGuard, plany **[Budget VPS](/pl/budget-vps/)** od **<span class="text-white">Voxi</span><span class="text-amber-300">Host</span>** są idealnym punktem wyjścia. Możesz uruchomić nową instancję w kilka sekund i od razu zacząć budować swoją prywatną sieć.