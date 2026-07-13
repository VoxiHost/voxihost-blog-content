---
image: /assets/images/blog/pl/prywatny-dysk-filebrowser-docker/og-image.png
title: "Jak zainstalować FileBrowser na VPS z Dockerem"
description: "Dowiedz się, jak w 5 minut zainstalować FileBrowser za pomocą Dockera i zamienić swój serwer VPS w prywatny dysk w chmurze z dostępem przez przeglądarkę."
date: 2026-07-13
translationKey: "private-cloud-filebrowser-docker"
locale: pl
category: Poradniki
tags: ["docker", "filebrowser", "chmura", "vps", "linux", "storage"]
status: published
author:
  name: VoxiHost Team
  link: https://voxihost.pl/
contributors:
  - sl0ikkk
howto:
  name: "Instalacja FileBrowser w Dockerze"
  totalTime: "PT5M"
  yield: "W pełni funkcjonalny, prywatny dysk w chmurze w przeglądarce"
  tool:
    - "Dowolny, nawet najsłabszy serwer VPS (np. z oferty VoxiHost)"
    - "Zainstalowany Docker"
  steps:
    - name: "Krok 1"
      text: "Pobieranie i instalacja"
      url: "krok-1-szybkie-uruchomienie-przez-dockera"
    - name: "Krok 2"
      text: "Pierwsze logowanie"
      url: "krok-2-pierwsze-logowanie-do-dysku"
    - name: "Krok 3"
      text: "Konfiguracja konta"
      url: "krok-3-zmiana-domyslnego-hasla-wazne"
    - name: "Krok 4"
      text: "Zarządzanie plikami"
      url: "krok-4-wgrywanie-i-udostepnianie-plikow"
faq:
  - question: "Czy potrzebuję domeny, żeby używać FileBrowsera?"
    answer: "Nie, FileBrowser działa bezpośrednio przez adres IP serwera i numer portu (np. <code>http://ADRES_IP:8080</code>). Domena jest opcjonalna."
  - question: "Czy bezpiecznie jest udostępniać FileBrowser na publicznym IP?"
    answer: "Po wykonaniu kroków z tego poradnika Twoje konto będzie chronione silnym hasłem. Dla większego bezpieczeństwa rozważ umieszczenie FileBrowsera za reverse proxy z HTTPS (np. Nginx + Let's Encrypt) lub ograniczenie dostępu przez firewall."
  - question: "Jak FileBrowser uruchamia się po restarcie serwera?"
    answer: "Flaga <code>--restart always</code> w poleceniu Docker run sprawia, że kontener uruchamia się automatycznie po każdym restarcie serwera."
  - question: "Czy mogę ograniczyć FileBrowser tylko do wybranego folderu?"
    answer: "Tak. Zastąp <code>-v /:/srv</code> konkretną ścieżką, np. <code>-v /home/user/pliki:/srv</code>, aby ograniczyć dostęp wyłącznie do tego katalogu."
---

## Wstęp

Posiadanie serwera VPS to świetna sprawa, ale przesyłanie na niego plików przez terminal (SFTP) bywa uciążliwe. Co powiesz na to, by zamienić Twój tani serwer w wygodny dysk w chmurze (jak Dysk Google czy Dropbox), obsługiwany w 100% z poziomu przeglądarki?

Pomoże w tym **FileBrowser** – niesamowicie lekki menedżer plików. Działa błyskawicznie, nie wymaga domeny (możesz wejść po prostu przez adres IP) i jest tak mało zasobożerny, że idealnie sprawdzi się na każdym, nawet najtańszym [Budget VPS](/pl/budget-vps/) z naszej oferty!

---

## Krok 1: Szybkie uruchomienie przez Dockera

Zaloguj się na swój serwer przez terminal. Uruchomienie FileBrowsera sprowadza się do jednej, prostej komendy. Wklej poniższy kod i wciśnij ENTER:

{% image "/assets/images/blog/pl/prywatny-dysk-filebrowser-docker/H1.png", "Terminal podczas uruchamiania kontenera FileBrowser", "(max-width: 768px) 100vw, 800px" %}

```bash
docker run -d --name filebrowser -v /:/srv -p 8080:80 --restart always filebrowser/filebrowser
```

**Co dokładnie robi ta komenda?**
- `-p 8080:80` otwiera panel na porcie 8080.
- `-v /:/srv` daje aplikacji dostęp do głównego dysku Twojego serwera, dzięki czemu możesz przeglądać i edytować absolutnie każdy plik systemowy przez przeglądarkę!

---

## Krok 2: Pierwsze logowanie do dysku

Instalacja na serwerze to już wszystko! Teraz otwórz przeglądarkę internetową i wpisz adres IP swojego serwera wraz z portem 8080:

`http://ADRES_IP_SERWERA:8080`

Zobaczysz schludny, minimalistyczny ekran logowania.

{% image "/assets/images/blog/pl/prywatny-dysk-filebrowser-docker/H2.png", "Ekran logowania do panelu FileBrowser", "(max-width: 768px) 100vw, 800px" %}

Aby się zalogować, jako login wpisz **`admin`**. W najnowszych wersjach FileBrowsera zrezygnowano ze stałego domyślnego hasła ze względów bezpieczeństwa.
Twoje **jednorazowe hasło startowe** zostało wygenerowane automatycznie. Aby je poznać, wróć do konsoli i wpisz:

```bash
docker logs filebrowser
```

W logach znajdziesz linijkę z nowym, wygenerowanym hasłem. Skopiuj je i użyj do pierwszego logowania.

---

## Krok 3: Zmiana domyślnego hasła (Ważne!)

Zaraz po zalogowaniu musisz zadbać o swoje bezpieczeństwo, aby nikt obcy nie wszedł na Twój dysk. Kliknij w zakładkę **Settings** (Ustawienia) w panelu po lewej stronie, a następnie przejdź do sekcji **Profile Settings** (Ustawienia profilu).

{% image "/assets/images/blog/pl/prywatny-dysk-filebrowser-docker/H3.png", "Zakładka zmiany hasła w ustawieniach profilu", "(max-width: 768px) 100vw, 800px" %}

Wpisz nowe, trudne do odgadnięcia hasło, zatwierdź je i kliknij przycisk "Update". Od teraz Twój prywatny dysk jest w pełni bezpieczny.

---

## Krok 4: Wgrywanie i udostępnianie plików

Teraz możesz robić ze swoim dyskiem co tylko zechcesz. W prawym górnym rogu znajdziesz przyciski do tworzenia nowych folderów, wgrywania plików (Upload), a nawet tworzenia pustych plików tekstowych, które możesz edytować bezpośrednio w przeglądarce!

Zaznaczając dowolny plik, możesz kliknąć ikonkę udostępniania (Share), aby wygenerować specjalny link. Możesz wysłać ten link znajomemu, aby pobrał plik z Twojego serwera bez konieczności logowania.

{% image "/assets/images/blog/pl/prywatny-dysk-filebrowser-docker/H4.png", "Interfejs FileBrowser podczas udostępniania pliku", "(max-width: 768px) 100vw, 800px" %}

---

## Podsumowanie

FileBrowser to "must-have" na każdym prywatnym serwerze. Zużywa ułamek procenta pamięci RAM, a daje ogromną wygodę przy zarządzaniu plikami w Linuxie. Wykorzystaj to rozwiązanie na swoim nowym serwerze z serii [Premium VPS](/pl/premium-vps/) i zapomnij o konieczności używania programów FTP!
