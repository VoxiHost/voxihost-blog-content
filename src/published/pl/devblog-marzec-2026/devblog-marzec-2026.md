---
image: /assets/images/blog/pl/devblog-marzec-2026/og-image.png
title: 'Marzec 2026: VoxiHost DevBlog'
description: To był intensywny miesiąc w VoxiHost! W tym DevBlogu opisujemy wdrożenie konsoli VNC, wsparcie dla 7 nowych dystrybucji Linux oraz logowanie przez Google.
date: '2026-04-01'
updated: '2026-06-02'
translationKey: march-2026-devblog
locale: pl
category: Nowości
tags:
  - nowości
  - vnc
  - almalinux
  - rocky linux
  - google auth
  - bezpieczeństwo
  - panel
status: published
author:
  name: VoxiHost Team
  link: https://voxihost.pl/
contributors:
  - danielmarszalkowski
---

Marzec był niesamowicie produktywnym miesiącem w **<span class="text-white">Voxi</span><span class="text-amber-300">Host</span>**. Słuchaliśmy Waszych opinii i ciężko pracowaliśmy nad funkcjami, które sprawią, że zarządzanie Twoją infrastrukturą w chmurze będzie szybsze, bezpieczniejsze i bardziej elastyczne.

Od niskopoziomowego dostępu do serwera po ogromne rozszerzenie listy wspieranych systemów operacyjnych - oto wszystko, co wdrożyliśmy w ciągu ostatnich kilku tygodni.

## 1. Bezpośredni dostęp przez konsolę VNC

Jedna z najbardziej wyczekiwanych funkcji jest już dostępna: **wsparcie dla konsoli VNC** bezpośrednio w Twoim panelu zarządzania.

{% image "/assets/images/blog/pl/devblog-marzec-2026/vnc-support.png", "Panel VoxiHost pokazujący nową integrację konsoli VNC do zdalnego zarządzania serwerami VPS", "(max-width: 768px) 100vw, 800px" %}

Wiemy, jak frustrująca może być utrata dostępu do serwera z powodu błędu w konfiguracji firewallu lub uszkodzenia pliku konfiguracyjnego SSH. Dzięki nowemu wsparciu VNC, możesz uzyskać dostęp do swojego VPS na „poziomie sprzętowym” przez przeglądarkę. Oznacza to, że możesz rozwiązywać problemy z bootowaniem, poprawiać ustawienia sieciowe lub zarządzać serwerem, nawet jeśli SSH jest całkowicie niedostępne.

## 2. 7 nowych dystrybucji OS (Alma, Rocky, Fedora, CentOS)

Wierzymy w wolność wyboru środowiska, które najlepiej pasuje do Twojego przepływu pracy. W tym miesiącu znacząco rozszerzyliśmy naszą bibliotekę systemów, dodając najbardziej stabilne i nowoczesne dystrybucje do zastosowań korporacyjnych i programistycznych.

**Nowe systemy to:**
*   **AlmaLinux 9 & 10** (Idealny następca CentOS)
*   **Rocky Linux 9 & 10** (Wspierany przez społeczność, profesjonalny Linux)
*   **CentOS Stream 9 & 10**
*   **Fedora 43** (Dla tych, którzy potrzebują absolutnie najnowszych pakietów)

{% image "/assets/images/blog/pl/devblog-marzec-2026/new-distributions.png", "Wybór nowych dystrybucji Linux dostępnych do instalacji jednym kliknięciem na VoxiHost VPS", "(max-width: 768px) 100vw, 800px" %}

Wszystkie te dystrybucje są już dostępne jako **instalacja jednym kliknięciem** we wszystkich naszych planach [Premium](/pl/premium-vps/) oraz [Budget](/pl/budget-vps/) VPS.

## 3. Szybsze logowanie dzięki Google Auth

Bezpieczeństwo i wygoda nie muszą się wykluczać. Aby ułatwić Ci życie, wdrożyliśmy **wsparcie dla Google OAuth**.

{% image "/assets/images/blog/pl/devblog-marzec-2026/google-login.png", "Strona logowania Google przekierowująca do panelu VoxiHost", "(max-width: 768px) 100vw, 800px" %}

{% image "/assets/images/blog/pl/devblog-marzec-2026/google-select.png", "Panel logowania VoxiHost prezentujący nową funkcję 'Zaloguj z Google'", "(max-width: 768px) 100vw, 800px" %}

Możesz teraz powiązać swoje konto Google z <span class="text-white">Voxi</span><span class="text-amber-300">Host</span> i logować się jednym kliknięciem. To nie tylko przyspiesza dostęp do panelu, ale pozwala również wykorzystać zaawansowane uwierzytelnianie wieloskładnikowe (MFA) od Google, aby jeszcze lepiej chronić Twoje usługi hostingowe.

## 4. Zweryfikowane opinie przez Trustpilot

Transparentność to jedna z naszych kluczowych wartości. Odświeżyliśmy sekcję recenzji i zintegrowaliśmy ją bezpośrednio z platformą **Trustpilot**. Możesz teraz zobaczyć autentyczne opinie naszych użytkowników wraz z bezpośrednimi linkami do oryginałów.

{% image "/assets/images/blog/pl/devblog-marzec-2026/trustpilot-reviews.png", "Sekcja opinii VoxiHost prezentująca integrację z serwisem Trustpilot", "(max-width: 768px) 100vw, 800px" %}

Chcemy, abyś przed wydaniem choćby złotówki dokładnie wiedział, czego spodziewać się po naszym sprzęcie i wsparciu technicznym.

## 5. Ulepszanie platformy

Poza dużymi nowościami, wprowadziliśmy dziesiątki mniejszych usprawnień:
*   **Nowa platforma blogowa:** Właśnie na nią patrzysz! Uruchomiliśmy nasz blog, aby dostarczać Wam wysokiej jakości tutoriale Linux i świeże informacje z życia firmy.
*   **System promocji:** Nowy, inteligentny banner na stronie głównej będzie teraz zawsze pokazywał najlepsze aktualnie dostępne zniżki.
*   **Poprawki błędów:** Wyeliminowaliśmy błąd na stronie Kontakt, gdzie znaki specjalne były błędnie interpretowane.

## Co dalej?
Marzec był wielkim krokiem naprzód, ale nie zwalniamy tempa. Pracujemy już nad kolejnymi ulepszeniami sieciowymi oraz nowymi zautomatyzowanymi narzędziami w Twoim panelu.

Bądźcie czujni, i jak zawsze - dziękujemy za wybranie **<span class="text-white">Voxi</span><span class="text-amber-300">Host</span>**!


**Gotowy przetestować nową konsolę VNC?** Zaloguj się do swojego **[Panelu](https://dashboard.voxihost.pl)** i sprawdź swoje aktywne usługi już dzisiaj!