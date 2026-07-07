---
image: /assets/images/blog/pl/jak-zainstalowac-uptime-kuma-docker/og-image.png
title: "Jak zainstalować Uptime Kuma w Dockerze: Własny monitoring"
description: "Zbuduj własny system monitorowania serwerów i stron internetowych za pomocą darmowego narzędzia Uptime Kuma. Otrzymuj powiadomienia o awariach na Discordzie."
date: '2026-07-07'
translationKey: "setup-uptime-kuma-docker"
locale: pl
category: "Poradniki"
tags: ["linux", "vps", "docker", "monitoring", "uptime-kuma"]
status: published
author:
  name: VoxiHost Team
  link: https://voxihost.pl/
contributors:
  - danielmarszalkowski
howto:
  name: "Instalacja Uptime Kuma"
  totalTime: "PT15M"
  yield: "Własny serwer monitorowania z powiadomieniami na żywo"
  tool:
    - "Serwer VPS z systemem Linux"
    - "Uprawnienia administratora (sudo)"
  steps:
    - name: "Krok 1: Przygotowanie Docker Compose"
      text: "Instalacja niezbędnych narzędzi."
      url: "krok-1-przygotowanie-docker-compose"
    - name: "Krok 2: Konfiguracja i uruchomienie Uptime Kuma"
      text: "Utworzenie pliku docker-compose.yml i start kontenera."
      url: "krok-2-konfiguracja-i-uruchomienie-uptime-kuma"
    - name: "Krok 3: Pierwsze logowanie i dodanie monitoringu"
      text: "Dodanie serwera do monitorowania."
      url: "krok-3-pierwsze-logowanie-i-dodanie-monitoringu"
    - name: "Krok 4: Skonfigurowanie powiadomień"
      text: "Podpięcie Webhooka Discord do powiadomień."
      url: "krok-4-skonfigurowanie-powiadomien"
faq:
  - question: "Czy Uptime Kuma jest całkowicie darmowa?"
    answer: "Tak, Uptime Kuma to w 100% darmowe i otwartoźródłowe narzędzie do monitorowania na licencji MIT."
  - question: "Jakich zasobów wymaga Uptime Kuma?"
    answer: "Uptime Kuma jest niezwykle lekka i zużywa minimalną ilość zasobów, dzięki czemu idealnie nadaje się na tani serwer <code>Budget VPS</code>."
  - question: "Czy Uptime Kuma pozwala monitorować serwery Minecraft?"
    answer: "Tak, Uptime Kuma posiada wbudowaną funkcję odpytującą serwery Minecraft, która bezpośrednio sprawdza status portu gry."
---

## Wprowadzenie

Czy kiedykolwiek obudziłeś się rano, tylko po to, by odkryć, że Twój serwer Minecraft lub strona internetowa nie działa od kilku godzin? Zamiast polegać na płatnych rozwiązaniach typu Uptime Robot, możesz hostować własny, piękny i w 100% darmowy system monitorowania.

**Uptime Kuma** to potężne narzędzie o otwartym kodzie źródłowym, które potrafi monitorować strony WWW, serwery gier, porty TCP i usługi DNS. W tym szybkim poradniku pokażemy, jak w 15 minut zainstalować je na własnym serwerze za pomocą Dockera!

> **Wskazówka:** Uptime Kuma nie zajmuje wiele zasobów. Do tego zadania spokojnie wystarczy najtańszy plan z naszej oferty [Budget VPS](/pl/budget-vps/) (np. oparty na Ubuntu 22.04).

---

## Krok 1: Przygotowanie Docker Compose

Cała aplikacja Uptime Kuma mieści się w jednym kontenerze Dockera. Jeżeli nie posiadasz jeszcze Dockera, zainstaluj go korzystając ze skryptu instalacyjnego:

{% image "/assets/images/blog/pl/jak-zainstalowac-uptime-kuma-docker/H1.png", "Terminal podczas instalacji środowiska Docker", "(max-width: 768px) 100vw, 800px" %}

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo systemctl enable --now docker
```

Zalecamy używanie wtyczki Docker Compose (w najnowszych wersjach jest już wbudowana, ale możemy się upewnić):

```bash
sudo apt-get install docker-compose-plugin
```

Sprawdź wersję, aby upewnić się, że wszystko działa:

```bash
docker compose version
```

---

## Krok 2: Konfiguracja i uruchomienie Uptime Kuma

Teraz utworzymy dedykowany folder dla naszej aplikacji i skonfigurujemy w nim plik YAML z ustawieniami kontenera.

{% image "/assets/images/blog/pl/jak-zainstalowac-uptime-kuma-docker/H2.png", "Tworzenie pliku docker-compose.yml dla Uptime Kuma", "(max-width: 768px) 100vw, 800px" %}

```bash
mkdir -p ~/uptime-kuma
cd ~/uptime-kuma
```

Utwórz plik `docker-compose.yml` przy pomocy ulubionego edytora tekstowego, na przykład `nano`:

```bash
nano docker-compose.yml
```

Wklej do niego poniższą zawartość:

```yaml
services:
  uptime-kuma:
    image: louislam/uptime-kuma:1
    container_name: uptime-kuma
    volumes:
      - ./uptime-kuma-data:/app/data
    ports:
      - "3001:3001"  # Możesz zmienić lewą wartość na inny port, jeśli 3001 jest zajęty
    restart: always
```

Zapisz plik (`Ctrl+O`, `Enter`) i wyjdź z edytora (`Ctrl+X`).

Następnie uruchom aplikację w tle korzystając z polecenia:

```bash
sudo docker compose up -d
```

To wszystko! Aplikacja pobierze obraz i uruchomi się. Możesz to zweryfikować wpisując `sudo docker ps`.

---

## Krok 3: Pierwsze logowanie i dodanie monitoringu

Aby zalogować się do panelu, otwórz w przeglądarce adres IP swojego serwera, dopisując port `3001`, np.:
`http://TWÓJ_ADRES_IP:3001`

> **Zabezpieczenia:** Pamiętaj, że o ile Uptime Kuma wymusza stworzenie hasła, to łączenie się po HTTP nie jest najbezpieczniejsze. Koniecznie zapoznaj się z naszym poradnikiem [Jak skonfigurować Nginx Proxy Manager](/pl/blog/konfiguracja-nginx-proxy-manager-vps/), by podpiąć swoją domenę pod Uptime Kumę i wygenerować darmowy certyfikat SSL!

Po wejściu na stronę utworzysz swoje konto administratora (nazwa użytkownika i hasło). 

Następnie w lewym górnym rogu kliknij **"Add New Monitor"**.
Masz tu do wyboru mnóstwo opcji, np.:
- **HTTP(s)**: do sprawdzania, czy strona (np. Twój blog) działa.
- **TCP Port**: do sprawdzania, czy proces SSH (port 22) lub aplikacja sieciowa działa.
- **Steam Game Server**: idealne do serwerów gier (np. FiveM, Rust).
- **Minecraft**: wbudowana funkcja odpytująca bezpośrednio serwery MC (podajesz adres IP i port gier).

{% image "/assets/images/blog/pl/jak-zainstalowac-uptime-kuma-docker/H3.png", "Ekran dodawania nowego monitoringu dla serwera Minecraft", "(max-width: 768px) 100vw, 800px" %}

Uzupełnij formularz i kliknij "Save". Twój pierwszy monitoring został uruchomiony!

---

## Krok 4: Skonfigurowanie powiadomień

Monitoring jest bezużyteczny, jeśli nikt nie wie o awarii. Uptime Kuma obsługuje ponad 90(!) usług powiadomień, w tym Telegram, Discord, Slack, Email (SMTP), a nawet Microsoft Teams.

{% image "/assets/images/blog/pl/jak-zainstalowac-uptime-kuma-docker/H4.png", "Konfiguracja powiadomień Webhook z Discorda w Uptime Kuma", "(max-width: 768px) 100vw, 800px" %}

Pokażemy, jak skonfigurować proste powiadomienia na **Discordzie**:
1. Na serwerze Discord wejdź w Ustawienia kanału tekstowego -> **Integracje** -> **Utwórz Webhooka**. Skopiuj jego URL.
2. W Uptime Kuma przejdź do górnego menu i wejdź w **Settings -> Notifications**.
3. Kliknij "Setup Notification".
4. Wybierz typ: **Discord**.
5. Wklej skopiowany *Webhook URL*.
6. Kliknij *Test*, aby sprawdzić czy bot wysłał próbną wiadomość na Twoim kanale. Jeśli tak, zapisz klikając **Save**.

Teraz przy tworzeniu lub edycji jakiegokolwiek monitora (Krok 3) po prawej stronie upewnij się, że zaznaczyłeś swój nowo utworzony kanał powiadomień!

---

## Podsumowanie

Uruchomienie własnego systemu monitorowania to zaledwie kilkanaście minut pracy. Dzięki Uptime Kumie zachowujesz pełną prywatność logów i nie ponosisz opłat abonamentowych za usługi premium (jak krótkie, 1-minutowe interwały sprawdzania, które w rozwiązaniach SaaS bywają bardzo drogie).

Uptime Kuma zainstalowana na stabilnym serwerze [Budget VPS](/pl/budget-vps/) od <span class="text-white">Voxi</span><span class="text-amber-300">Host</span> to gwarancja, że będziesz wiedział o awarii swoich głównych projektów zanim zauważą ją Twoi klienci lub gracze!