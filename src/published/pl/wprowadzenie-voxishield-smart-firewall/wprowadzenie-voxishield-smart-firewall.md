---
image: /assets/images/blog/pl/wprowadzenie-voxishield-smart-firewall/og-image.png
title: 'Panel zarządzania VoxiShield Smart Firewall już dostępny'
description: Zarządzaj bezpieczeństwem VPS dzięki VoxiShield! Konfiguruj reguły wejściowe, limity PPS, filtry egress i blokady GeoIP bezpośrednio z panelu klienta.
date: '2026-07-10'
translationKey: introducing-voxishield-smart-firewall
locale: pl
category: Nowości
tags:
  - voxishield
  - firewall
  - ochrona-ddos
  - bezpieczenstwo-vps
  - nowosci
status: published
author:
  name: VoxiHost Team
  link: https://voxihost.pl/
contributors:
  - danielmarszalkowski
faq:
  - question: "Czy VoxiShield jest wliczony w cenę mojej usługi?"
    answer: "Tak! VoxiShield jest bezpłatnie dołączony do wszystkich usług VoxiHost. Każdy serwer wirtualny posiada włączone pełne filtrowanie brzegowe oraz niestandardowe sterowanie firewallem od pierwszego dnia bez żadnych dodatkowych opłat. Nie płacisz ekstra za ochronę DDoS, czyszczenie ruchu czy konfigurację reguł, ponieważ stanowią one wbudowany standard każdego planu."
  - question: "Jak działa dwuwarstwowa ochrona?"
    answer: "Ruch jest analizowany i filtrowany stopniowo przez dwie odrębne warstwy ochrony:<br><br><b>Warstwa 1: Filtrowanie brzegowe (Edge Scrubbing)</b>: Nasza globalna sieć brzegowa o przepustowości ponad 4+ Tbit/s pochłania masowe ataki wolumetryczne (takie jak flood UDP/SYN), zanim dotrą one do rdzenia naszej sieci.<br><br><b>Warstwa 2: Core Firewall</b>: Oczyszczony ruch docierający do naszych hiperwizorów jest sprawdzany przez inteligentną zaporę Core Firewall. To tam Twoje własne reguły, dostęp do portów, filtry GeoIP i limity pakietów są wdrażane w czasie poniżej sekundy."
  - question: "Przed jakimi typami ataków chroni VoxiShield?"
    answer: "VoxiShield chroni przed szeroką gamą wektorów zagrożeń: wolumetrycznymi floodami UDP/TCP, amplifikacjami NTP/DNS, nieprawidłowymi pakietami protokołów (np. exploitami zapytań dla gier Minecraft, Rust i FiveM), atakami w warstwie aplikacji (L7) HTTP flood, próbami brute-force oraz skanowaniem portów. Nasze szablony automatycznie dopasowują się do ruchu gier i sieci WWW, odrzucając złośliwe pakiety i wpuszczając prawdziwych użytkowników."
  - question: "Czy mogę samodzielnie konfigurować reguły zapory?"
    answer: "Tak! Każdy klient otrzymuje pełny dostęp do zarządzania zaporą z poziomu dashboardu VoxiHost. Możesz tworzyć niestandardowe reguły portów, ograniczać protokoły, ustawiać limity pakietów na sekundę, blokować wybrane kraje filtrem GeoIP, wyłączać dostęp dla sieci VPN/Tor i na bieżąco analizować zablokowane połączenia."
  - question: "Co powinienem zrobić podczas aktywnego ataku?"
    answer: "W większości przypadków nie musisz robić absolutnie nic. VoxiShield neutralizuje ataki automatycznie w ciągu kilku sekund. Jeśli zauważysz spadek wydajności usługi, możesz sprawdzić logi firewalla w swoim dashboardzie lub skontaktować się z działem pomocy. Możemy natychmiast wdrożyć reguły na poziomie Layer 2 lub dostosować filtry brzegowe dla Twojego portu."
  - question: "Czy ochrona VoxiShield zwiększa opóźnienia sieciowe (ping)?"
    answer: "Nie. W przeciwieństwie do tradycyjnych rozwiązań tunelowych, które kierują ruch przez odległe centra filtrowania, nasza sieć brzegowa Layer 1 jest zlokalizowana przy głównych węzłach tranzytowych, a filtrowanie Layer 2 odbywa się na poziomie hiperwizora. Oznacza to, że ochrona działa bezpośrednio na ścieżce pakietu, nie wprowadzając pętli routingu ani nie zwiększając pingu, co pozwala zachować optymalny tickrate gier i szybki czas odpowiedzi API."
---

W **<span class="text-white">Voxi</span><span class="text-amber-300">Host</span>** uważamy, że bezpieczeństwo to nie luksusowa usługa premium, lecz absolutny standard. Dzisiaj z ogromną radością ogłaszamy oficjalne wdrożenie panelu zarządzania **VoxiShield DDoS Protection & Smart Firewall**, który daje Ci pełną kontrolę nad ruchem sieciowym Twojego serwera bezpośrednio z poziomu dashboardu VoxiHost.

Dzięki infrastrukturze filtrującej o przepustowości ponad 4+ Tbit/s, VoxiShield skutecznie neutralizuje nawet najbardziej intensywne ataki sieciowe, dbając o ciągłe działanie Twoich aplikacji, stron WWW i serwerów gier. Co najważniejsze: cały system otrzymujesz **całkowicie za darmo** w pakiecie z każdą naszą usługą.

{% image "/assets/images/blog/pl/wprowadzenie-voxishield-smart-firewall/dashboard-overview.png", "Podgląd panelu zarządzania VoxiShield Firewall w dashboardzie VoxiHost", "(max-width: 768px) 100vw, 800px" %}

---

## Dwuwarstwowy system filtrowania ruchu (Mitigation Pipeline)

Tradycyjne oprogramowanie zapory sieciowej (firewalla) działa bezpośrednio w systemie operacyjnym Twojego serwera, co podczas ataku zużywa cenne zasoby procesora i pamięci RAM. VoxiShield działa zupełnie inaczej, filtrując złośliwy ruch poza Twoim VPS, zanim pakiety w ogóle dotrą do Twojej wirtualnej maszyny.

### Layer 01: PletX Edge (Automatyczne filtrowanie wolumetryczne)
Cały ruch przychodzący trafia najpierw do **globalnych węzłów filtrujących PletX Edge**. Ta warstwa działa w 100% automatycznie i bezobsługowo. Neutralizuje ataki wolumetryczne (np. DNS/NTP amplification lub zalewy UDP) o sile do 4+ Tbit/s, gwarantując, że łącze Twojego serwera nie zostanie przeciążone.

### Layer 02: Core Firewall (Zarządzanie z poziomu dashboardu)
Oczyszczony z ataków wolumetrycznych ruch trafia do naszego przezroczystego **Core Firewalla**. To tutaj masz pełną kontrolę. Z poziomu dashboardu <span class="text-white">Voxi</span><span class="text-amber-300">Host</span> możesz błyskawicznie tworzyć reguły dostępu, zarządzać portami i ograniczać podejrzany ruch z czasem propagacji poniżej jednej sekundy.

---

## Zaawansowany firewall pod Twoją kontrolą

Zaprojektowaliśmy VoxiShield tak, aby skomplikowane reguły `iptables` i `nftables` zastąpić intuicyjnym, graficznym panelem w dashboardzie. Nie musisz być administratorem sieci, aby profesjonalnie zabezpieczyć swój serwer.

### 1. Reguły wejściowe i limity PPS (Inbound Rules & Rate Limiting)
Określaj precyzyjnie, kto i jakimi protokołami może komunikować się z Twoimi aplikacjami:
*   Otwieraj konkretne porty lub zakresy portów.
*   Wybieraj obsługiwane protokoły (TCP, UDP lub oba jednocześnie).
*   Przypisuj akcje (Zezwól / Blokuj) dla poszczególnych reguł.
*   Ustawiaj **dokładne limity pakietów na sekundę (PPS)**, aby zabezpieczyć aplikacje i serwery gier przed atakami typu flood.

{% image "/assets/images/blog/pl/wprowadzenie-voxishield-smart-firewall/inbound-rules.png", "Konfiguracja reguł przychodzących i limitów PPS w dashboardzie VoxiHost", "(max-width: 768px) 100vw, 800px" %}

### 2. Bezpieczeństwo wyjściowe (Egress Security)
Dbaj o czystość i reputację swojego adresu IP. Kontrola ruchu wyjściowego pozwala monitorować pakiety opuszczające Twój serwer, chroniąc przed sytuacją, w której przejęta przez hakerów aplikacja zacznie wysyłać spam lub skanować sieć.
*   **Blokada portu 25 (SMTP)** zapobiegająca masowej wysyłce spamu.
*   Tworzenie własnych reguł zezwalających (whitelist) dla ruchu wychodzącego.

{% image "/assets/images/blog/pl/wprowadzenie-voxishield-smart-firewall/outbound-rules.png", "Konfiguracja reguł ruchu wyjściowego w dashboardzie VoxiHost", "(max-width: 768px) 100vw, 800px" %}

### 3. Analizator blokad (Block Analyzer)
Diagnozowanie problemów z łącznością stało się banalnie proste. Jeśli klient lub zewnętrzna usługa nie może połączyć się z Twoim VPS, możesz natychmiast sprawdzić powód w narzędziu **Block Analyzer**:
*   Sprawdź, czy adres IP nie został zablokowany przez filtry globalne (np. GeoIP/ASN).
*   Dowiedz się, czy dany adres IP nie przekroczył zdefiniowanych limitów pakietów (PPS).

{% image "/assets/images/blog/pl/wprowadzenie-voxishield-smart-firewall/block-analyzer.png", "Weryfikacja blokad adresów IP w narzędziu Block Analyzer w dashboardzie VoxiHost", "(max-width: 768px) 100vw, 800px" %}

### 4. Filtry globalne (Global Inbound)
Odrzucaj niepożądane połączenia na poziomie infrastruktury sieciowej VoxiHost:
*   **Filtr GeoIP**: Blokuj lub zezwalaj na ruch z wybranych państw.
*   **Filtr sieci ASN**: Blokuj całe serwerownie, dostawców VPN lub konkretnych operatorów.
*   **Baza Threat Shield**: Automatyczne, codzienne aktualizowanie list znanych botnetów i złośliwych skanerów sieciowych.

{% image "/assets/images/blog/pl/wprowadzenie-voxishield-smart-firewall/global-filters.png", "Zarządzanie filtrami GeoIP oraz ASN w dashboardzie VoxiHost", "(max-width: 768px) 100vw, 800px" %}

---

## Dedykowane szablony dla gier i protokołów

Różne aplikacje wymagają innej charakterystyki filtrowania ruchu. Strona internetowa oparta o protokół HTTPS potrzebuje innych reguł ochronnych niż serwer Minecraft lub Counter-Strike oparty o protokół UDP.

VoxiShield automatycznie wdraża i dopasowuje szablony filtracji stworzone z myślą o najpopularniejszych grach sieciowych i protokołach sieciowych. Dzięki temu filtry reagują w ułamku sekundy, zachowując minimalne opóźnienia (pingi) podczas gry.

---

## Promocja z okazji premiery: -35% z kodem VOXISHIELD

Chcesz przetestować działanie nowego panelu zarządzania? Przygotowaliśmy dedykowaną zniżkę na uruchomienie nowych usług VPS z wbudowaną ochroną.

Użyj kodu rabatowego **<span class="text-amber-300 font-bold text-xl uppercase tracking-wider">VOXISHIELD</span>** podczas składania zamówienia, aby otrzymać **35% rabatu** na miesięczną opłatę abonamentową.

> **Szczegóły promocji:** Kod promocyjny jest ważny dla wszystkich cykli miesięcznych do **10.07.2026**.

---

## Podsumowanie

W VoxiHost pokazujemy, że zaawansowane bezpieczeństwo sieciowe nie musi wiązać się z dodatkowymi opłatami ani skomplikowaną konfiguracją. Otrzymujesz potężne, bezpłatne narzędzie, które rozwija się wraz z Twoim projektem.

Zabezpiecz swoją infrastrukturę już dziś! Wybierz tani [Budget VPS](/pl/budget-vps/) lub ultraszybki [Premium VPS](/pl/premium-vps/), a konfigurację reguł sieciowych rozpocznij od razu w dedykowanym panelu [VoxiShield Smart Firewall](/pl/shield/) w dashboardzie VoxiHost.
