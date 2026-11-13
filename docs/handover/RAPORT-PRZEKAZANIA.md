# Raport przekazania — Szugers (aplikacja, marka, strona) + badania CGM

**Data sporządzenia:** 2026-11-13
**Sporządzono na podstawie:** całej rozmowy z Claude Code (sesja chmurowa od 2026-09-07 do 2026-11-13),
repozytorium `bpietrzkowski/szugers-site` (gałęzie `main` i `claude/cgm-refundacja-api-poland-0v36kx`),
trzech dokumentów badawczych powstałych w tej sesji oraz opublikowanych polityk prywatności Szugers 1.2
i Harmo 2.0, które są najdokładniejszym dostępnym opisem rzeczywistego stanu aplikacji.
**Odbiorca:** wspólny projekt obejmujący Szugers, Harmo, edytor ISO i spółkę sprzedającą te produkty.

> **Granice tego raportu — przeczytaj najpierw.**
> 1. Kod źródłowy aplikacji Szugers i Harmo **nie był dostępny** w tej sesji. Sesja chmurowa działa
>    na klonie z GitHuba, a jedynym repozytorium było `szugers-site` (strona + polityki). Wszystko, co
>    poniżej dotyczy wnętrza aplikacji, pochodzi z polityk prywatności, z opisu użytkownika w rozmowie
>    albo jest oznaczone jako wniosek.
> 2. O **edytorze ISO** i o **spółce** w tej rozmowie nie było żadnych informacji poza samą nazwą
>    w ostatnim poleceniu. Raport ich nie opisuje i nie zgaduje.
> 3. W tej sesji **nie powstał żaden kod**. Powstały cztery dokumenty (sekcja 4). Prośba o „pełny kod,
>    np. edytor" nie ma tu czego dotyczyć — mówię to wprost, żeby nikt nie szukał plików, których nie ma.
> 4. Oznaczenia pewności: `[FAKT]` — z repozytorium lub opublikowanego dokumentu; `[ROZMOWA]` — z wypowiedzi
>    użytkownika w tej sesji; `[RESEARCH]` — z badań webowych w tej sesji, z poziomem pewności jak w handoucie;
>    `[WNIOSEK]` — moja interpretacja.

---

## Spis treści

1. Cel i stan produktu
2. Technologia
3. Decyzje podjęte, uzasadnienia, opcje odrzucone
4. Artefakty powstałe w tej sesji
5. Źródła, dokumenty, normy, konta i usługi
6. Otwarte problemy, ryzyka, zadania i następne kroki
7. Sprawy firmowe i prawne
8. Załączniki (lista plików)

---

## 1. Cel i stan produktu

### 1.1 Marka i strona szugers.com

`[FAKT]` Domena `szugers.com` kieruje na GitHub Pages z repozytorium `bpietrzkowski/szugers-site`
(plik `CNAME`). Strona główna to landing „coming soon" z logo (SVG + PNG) i animowaną poświatą; bez
treści produktowej. Pod tą samą domeną publikowane są polityki prywatności obu aplikacji oraz filmy
do deklaracji usług na pierwszym planie w Google Play.

`[FAKT]` Harmo jest udostępniana „pod marką Szugers" — polityka Harmo 2.0, pkt 1. Szugers to zatem
jednocześnie nazwa aplikacji i marka parasolowa dla całego portfolio.

### 1.2 Szugers — dziennik cukrzycowy

**Cel.** `[FAKT, polityka 1.2 §5, §14]` „Przeglądarka Twoich własnych danych": zestawia dane historyczne
użytkownika z bieżącą glikemią, agreguje, uśrednia, porównuje i wizualizuje. Deklaratywnie **nie jest
wyrobem medycznym, nie ocenia, nie zaleca, nie prognozuje i nie ostrzega**; nie proponuje dawek insuliny.

**Platformy.** `[FAKT]` Android (aplikacja mobilna), Wear OS (aplikacja towarzysząca), tarcza zegarka,
widżet ekranu głównego.

**Faza.** `[FAKT]` Beta — polityka 1.2 §3 poz. 4: „na czas bety wysyłka z aplikacji jest wyłączona".
`[FAKT]` 2026-09-20 do repo trafił film `media/plakietka-fgs.mp4` do deklaracji
`FOREGROUND_SERVICE_SPECIAL_USE` w Google Play — czyli aplikacja jest w procesie publikacji / przeglądu
w Play Console. `[WNIOSEK]` Status „opublikowana w Play" nie jest potwierdzony w dostępnych źródłach.

**Co działa** (wg polityki 1.2 z 21.09.2026 — lista funkcji, które polityka opisuje jako istniejące):

| Obszar | Funkcje | Źródło |
|---|---|---|
| Źródła glukozy — 7 poziomów | 1) wpisy ręczne; 2) NovoPen przez NFC, glukometr BT (Glucose Service 0x1808), zegarek jako źródło tętna/kroków; 3) Health Connect (odczyt wybranych typów + zapis nazwy aplikacji źródłowej); 4) import CSV z LibreView / Dexcom Clarity + **kreator wniosków** do producenta (tekst do schowka); 5) odczyt powiadomień wskazanych aplikacji (tylko wartość, jednostka, trend, minuta); 6) LibreLinkUp na koncie użytkownika, po osobnym oświadczeniu z zapisaną datą; 7) xDrip+/Nightscout wyłącznie przez Health Connect | §3 |
| Każdy odczyt | pokazuje źródło i wiek w minutach | §3 |
| Insulina | dawki ręczne i z pena NFC, oznaczenia strzałów kontrolnych i wymian wkładu; **wykresy działania insuliny** (uproszczony model wg danych producenta, poglądowe) | §4, §14 |
| Posiłki | węglowodany, białko, tłuszcz, kalorie, cukry, IG, nazwa, czas; **OCR etykiety na urządzeniu** (zdjęcie nie jest zapisywane); karta posiłku do udostępnienia jako tekst | §4, §7 |
| Health Connect | odczyt: sen, kroki, tętno, ćwiczenia, dystans, kalorie, piętra, HRV, SpO2, oddech, temperatura skóry, masa, ciśnienie, nawodnienie; zapis: glukoza, posiłki, insulina (opcjonalnie) | §4, §7 |
| Zegarek | dane z zegarka (tętno, kroki, piętra, start/stop ćwiczenia); **plakietka na zegarku** z danymi; kanał Wear OS | §4, §9, §10 |
| Asystent AI | zapytanie o kontekst posiłku na **własnym kluczu API** użytkownika — Anthropic albo Google AI Studio; bez danych identyfikujących; zgoda per zapytanie | §6, §7 |
| Funkcje towarzyszące | plan dnia, lista zadań, notatki, odczyt kalendarza urządzenia | §4 |
| Dane i porządek | eksport CSV i kopii bazy; „Usuń wszystkie dane" (baza, ustawienia, poświadczenia, plakietka na zegarku); narzędzia „Duplikaty dawek" i „Kontrola danych" z **plikami cofania** w prywatnym katalogu | §10 |
| UI | HUB (edycja odczytów), karty posiłków i iniekcji, progi glikemii, kolory, układ kart; **zakładka Stat z plakietkami „Średnie" i „Stack"** | §11; `[ROZMOWA]` |
| Bezpieczeństwo | dane wyłącznie lokalnie; brak serwera; token LLU i klucze AI szyfrowane AES-GCM kluczem z Android Keystore; hasło LLU nie jest zapisywane; kopia Google i transfer na nowe urządzenie **wyłączone**; HTTPS | §9 |
| Prywatność | brak reklam, analityki, bibliotek śledzących; wszystkie źródła poza wpisami ręcznymi domyślnie wyłączone | §7, §12 |

**Co jest w toku** `[ROZMOWA]`:

- **Zakładka Stat → sekcja „Historia"**: pozioma, przewijana lista plakietek, jedna plakietka = jeden
  dzień, 60 dni wstecz, ten sam komponent/metryki/kolory co „Średnie". Doprecyzowana w rozmowie
  (sekcja 3, D-03..D-05), **nie zaimplementowana** — brak dostępu do kodu. Gotowy prompt: załącznik Z-04.
- **Przygotowanie repozytorium aplikacji do pracy z chmury / z telefonu**: higiena sekretów, CI,
  SessionStart hook, testy obliczeń statystyk. Zaakceptowane co do kierunku (D-06), nie wykonane.

**Co jest zaplanowane / rekomendowane** `[RESEARCH → handout §6]` (to są rekomendacje z badań, nie
zobowiązania produktowe; kolejność uzasadniona w handoucie):

1. Kontrakt danych + import plikowy (częściowo istnieje: CSV LibreView/Clarity).
2. Adapter Nightscout REST (dziś Nightscout tylko pośrednio przez Health Connect).
3. Rejestracja w Dexcom Developer sandbox — zadanie procesowe, długi czas oczekiwania.
4. Formalna decyzja o pozycji wobec MDR (dziś: świadome „nie wyrób medyczny", bez alertów).
5. Rozszerzenie Health Connect / HealthKit przy ewentualnej wersji iOS.
6. Agregator (Thryve/Junction) — tylko gdy potrzebny szybki dostęp do Libre bez umowy z Abbott.
7. Abbott bezpośrednio / Dexcom real-time partner — przy trakcji i statusie regulacyjnym.
8. Funkcje refundacyjne specyficzne dla Polski (kalkulator uprawnień po grupach R.03–R.05, licznik
   sztuk w miesiącu, przypomnienie o zleceniu) — pomysł z oceny Gluroo §4.3, nieuzgodniony.

### 1.3 Harmo — planer dnia

`[FAKT, polityka 2.0 z 7.11.2026]`

- **Czym jest:** zadania, projekty, zadania cykliczne, szablony, notatki i karteczki, czas planowany
  i rzeczywisty, postęp. Android + Wear OS (plansze i widżety) + **rozszerzenie dla Chrome**.
- **Architektura:** brak serwera, brak konta, **aplikacja nie ma uprawnienia dostępu do internetu**;
  rozszerzenie Chrome ma CSP blokujące wszystkie połączenia sieciowe. Telefon↔zegarek kanałem Wear OS;
  telefon↔komputer przez Web Bluetooth (parowanie) i Web Serial (dalsza praca).
- **Model biznesowy:** 5 tygodni pełnej wersji, potem zakup w Google Play (Play Billing). Dane zamówienia
  w Konsoli Play przetwarzane do rozliczeń podatkowych.
- **Kopia zapasowa:** przełącznik „Kopia zadań w Google" (domyślnie wyłączony); ustawienia i stan zakupu
  trafiają do kopii Androida, by okres próbny nie startował od nowa.
- **Stan:** polityka 1.0 (22.10.2026) → 2.0 (07.11.2026) w 7 językach (PL, EN, PT, ES, CS, SK, SL);
  2026-11-08 film `media/harmo-fgs.mp4` do deklaracji usługi na pierwszym planie. `[WNIOSEK]` Harmo jest
  w publikacji lub świeżo opublikowana, z aktywnym rynkiem poza Polską (języki).
- Polityka 2.0 pkt 13 wspomina **„przyszłą synchronizację przez konto Szugers"** jako możliwą, dobrowolną
  funkcję — to jedyny ślad planów serwerowych w całym portfolio.

### 1.4 Edytor ISO i spółka

Brak jakichkolwiek informacji w tej sesji. Patrz granice raportu.

---

## 2. Technologia

### 2.1 Repozytorium `bpietrzkowski/szugers-site`

`[FAKT]`

| Element | Wartość |
|---|---|
| URL | https://github.com/bpietrzkowski/szugers-site |
| Widoczność | publiczna |
| Hosting | GitHub Pages, domena `szugers.com` przez `CNAME` |
| Build | brak — statyczny HTML |
| Deploy | push do `main` → GitHub Pages publikuje automatycznie |
| Gałąź domyślna | `main` |

**Zawartość `main` (stan 2026-11-13):**

```
CNAME                                   szugers.com
index.html                              landing „coming soon" (PL, og:image = logo.png)
logo.svg, logo.png                      logo marki
polityka-prywatnosci.html               Szugers, wersja 1.2 z 21.09.2026
harmo/polityka-prywatnosci.html         Harmo, wersja 2.0 z 07.11.2026, 7 języków (97 kB)
harmo/polityka-prywatnosci-1.0.html     Harmo, wersja 1.0 z 22.10.2026 (zachowana)
media/plakietka-fgs.mp4                 film do deklaracji FGS — Szugers (6,6 MB)
media/harmo-fgs.mp4                     film do deklaracji FGS — Harmo (1,0 MB)
```

**Historia `main`:**

```
2026-09-07  61a4ab2  Create CNAME
2026-09-07  625decc  Add files via upload
2026-09-18  3bff7be  Add files via upload
2026-09-20  c4d57a1  Film do deklaracji FOREGROUND_SERVICE_SPECIAL_USE w Google Play
2026-09-21  1170846  Polityka prywatnosci 1.2 - pliki cofania i kreator wnioskow
2026-10-23  1999213  Harmo: osobna strona polityki prywatności (PL, EN, PT, ES) (#1)
2026-11-07  8be3fdb  Harmo: polityka prywatności 2.0 (Chrome, kopia zadań w Google, zakup, korespondencja) (#2)
2026-11-07  0aa44f8  Harmo: polityka prywatności 2.0 także po czesku, słowacku i słoweńsku (#3)
2026-11-08  b02c0f7  Harmo: film do deklaracji usługi na pierwszym planie (#6)
2026-11-08  99d3b42  Harmo: nowy film do deklaracji usługi na pierwszym planie (#7)
```

**Gałąź robocza tej sesji:** `claude/cgm-refundacja-api-poland-0v36kx` — **niezmergowana** do `main`,
bez konfliktów (dodaje wyłącznie katalog `docs/`). Zawiera:

```
2026-09-07  fffee0a  docs: research CGM w Polsce — refundacja i ścieżki integracji danych
2026-09-17  3ab36a7  docs: handout CGM dla Claude Code — ustalenia, ograniczenia, kontrakt danych
2026-09-17  c479055  docs: ocena ryzyk funkcjonowania Gluroo
2026-11-13  (ten commit) docs/handover/ — raport przekazania + prompt
```

Ścieżki:
- `docs/cgm-refundacja-i-integracje-pl.md` (20 kB)
- `docs/handout-cgm-integracje.md` (23 kB)
- `docs/ocena-ryzyk-gluroo.md` (23 kB)
- `docs/handover/RAPORT-PRZEKAZANIA.md` (ten plik)
- `docs/handover/prompt-desktop-cloud-ready-and-history.md`

Nie utworzono pull requesta (użytkownik o to nie prosił). Decyzja o mergu albo przeniesieniu `docs/`
do wspólnego projektu należy do odbiorcy — patrz zadanie Z-07 w sekcji 6.

### 2.2 Aplikacja Szugers — stos i architektura

`[FAKT z polityki 1.2 + WNIOSEK]` Kod aplikacji **nie jest** w żadnym repozytorium dostępnym w tej sesji
(`list_repos` zwrócił tylko `szugers-site`, `gra-w-biznes`, `BASUdriver`). `[ROZMOWA]` Użytkownik pracuje
nad nią z Claude Code **Desktop** na lokalnym katalogu. Lokalizacja kodu: komputer użytkownika.

Co o stosie wynika z polityki (bez dostępu do kodu nie potwierdzę języka ani frameworka — zakładam
natywny Android, co jest spójne z Keystore, Health Connect, NFC, BLE GATT i Wear OS):

| Warstwa | Rozwiązanie wynikające z polityki |
|---|---|
| Przechowywanie | lokalna baza w prywatnym katalogu aplikacji; brak serwera, brak konta |
| Sekrety | token sesji LLU + klucze API AI: AES-GCM, klucz z **Android Keystore** (sprzętowy, nieeksportowalny); hasło LLU nigdy nie zapisywane |
| Kopia Android | `allowBackup`/transfer **wyłączone** — świadoma decyzja (konsekwencja: dane nie wracają same po zmianie telefonu) |
| Źródła danych | NFC (NovoPen), BLE GATT **Glucose Service 0x1808** (parowanie w systemie, bez lokalizacji), **Health Connect** (odczyt/zapis per typ), **NotificationListenerService** (filtr do wskazanych pakietów, odrzucenie na wejściu), HTTPS do LibreLinkUp, import CSV, kalendarz urządzenia |
| AI | BYOK — klucz użytkownika do Anthropic albo Google AI Studio; wysyłany kontekst posiłku bez danych identyfikujących |
| OCR | na urządzeniu (zdjęcie nie jest zapisywane) |
| Zegarek | Wear OS: aplikacja towarzysząca, tarcza, plakietka; kanał Wear OS w obrębie sparowanych urządzeń |
| Tło | usługa na pierwszym planie typu **special use** (deklaracja w Play z filmem) — `[WNIOSEK]` prawdopodobnie do utrzymania plakietki / odczytu powiadomień |
| Eksport | CSV (nadający się do odczytu maszynowego, art. 20 RODO) i kopia bazy |

**Build i wdrożenie** `[WNIOSEK + ROZMOWA]`: build lokalnie (Desktop), publikacja przez **Google Play
Console** (deklaracje FGS i — z racji Health Connect — deklaracja uprawnień zdrowotnych). Podpisywanie
wyłącznie lokalnie — zasada przyjęta w D-06.

### 2.3 Aplikacja Harmo — stos

`[FAKT z polityki 2.0]` Android + Wear OS + rozszerzenie Chrome (Manifest V3 z CSP bez połączeń
sieciowych; uprawnienia: storage, offscreen document). Komunikacja telefon↔Chrome: Web Bluetooth
(parowanie) → Web Serial (RFCOMM). Usługa na pierwszym planie + uruchamianie po starcie dla utrzymania
połączenia. Play Billing. Kopia zapasowa Androida z selektywnym wykluczaniem danych planu.

Lokalizacja kodu: jak Szugers — poza zasięgiem tej sesji.

### 2.4 Środowisko pracy z Claude Code

`[ROZMOWA, FAKT]`

- **Desktop (lokalnie):** główna ścieżka rozwoju aplikacji. Ma kod, emulator, keystore.
- **Chmura (claude.ai/code, telefon):** działa na klonie z GitHuba; **nie widzi dysku użytkownika**;
  w tej sesji część domen była blokowana przez politykę sieciową (np. mp.pl, diabetyk24.pl,
  developer.dexcom.com — research oparł się o wyszukiwarkę). Przydatna do dokumentów, testów,
  refaktorów logiki, poprawek po review. Warunek: kod aplikacji w repozytorium GitHub z zainstalowaną
  aplikacją Claude GitHub (https://claude.ai/connect-github), sesja uruchomiona z tym repo zaznaczonym.
- Zasady pracy z chmury przyjęte w D-06 (sekcja 3).

---

## 3. Decyzje podjęte, uzasadnienia, opcje odrzucone

### 3.1 Decyzje podjęte w tej sesji

**D-01 · Wyniki badań trzymać jako pliki Markdown w repozytorium, nie jako artefakt/HTML/dokument chmurowy.**
Uzasadnienie: odbiorcą jest Claude Code w repozytorium aplikacji — plik w `docs/` jest kontekstem,
który agent czyta bezpośrednio. Odrzucone: artefakt HTML (ładniejszy, ale nieczytelny dla agenta jako
kontekst), dokument Claude Docs (poza repo). `[ROZMOWA]`

**D-02 · Handout w formacie „ustalenia ponumerowane + poziom pewności + przypis + konsekwencja".**
Uzasadnienie: pozwala w rozmowie z agentem kwestionować konkretne `U-xx` zamiast całego dokumentu;
linijka „→ Konsekwencja" zamienia research w kontekst wykonawczy. `[ROZMOWA]`

**D-03 · „Historia" w zakładce Stat = osobna plakietka per dzień, w przewijanej liście.**
Odrzucone: jedna plakietka z siatką 60 miniatur (moja pierwsza propozycja) — użytkownik wybrał listę
osobnych plakietek. `[ROZMOWA]`

**D-04 · Lista pozioma (karuzela), nie pionowa.**
Uzasadnienie: obok „Średnie" i „Stack" zajmuje tę samą wysokość; przewijanie w bok czyta się jako
oś czasu. Użytkownik potwierdził („Dokładnie tak"). `[ROZMOWA]`

**D-05 · 60 dni wstecz, oś ciągła — dni bez danych jako plakietka „brak danych", nie pomijane; obliczenia
wyciągnięte z „Średnie" do wspólnej funkcji; lista leniwa; sygnalizacja niekompletnego dnia (<70% odczytów).**
Uzasadnienie: ciągłość osi czasu jest informacją kliniczną; zero duplikacji logiki; wydajność; O-1. `[ROZMOWA]`

**D-06 · Praca nad aplikacją z telefonu (sesje chmurowe) — zaakceptowana z zabezpieczeniami.**
Użytkownik: „akceptuję Twoją wizję pracy z telefonu". Zabezpieczenia:
- repo prywatne; keystore, `google-services.json`, `local.properties`, `.env*` w `.gitignore`;
  dane testowe wyłącznie syntetyczne;
- ochrona `main`: brak bezpośrednich pushy, wymagany PR i zielone CI; sesje chmurowe zawsze na gałęzi;
- CI: kompilacja + testy jednostkowe, bez podpisywania i publikacji;
- testy jednostkowe dla obliczeń statystyk **przed** refaktorem;
- SessionStart hook instalujący SDK w kontenerze;
- podpisywanie i publikacja wyłącznie z Desktopu;
- podział ról: telefon = dokumenty, testy, refaktory, drobne zmiany ze specyfikacją; biurko = UI i merge.
Odrzucone: praca z telefonu bez CI (zmiany „w ciemno" w aplikacji liczącej glikemię — ryzyko R-15 z oceny
Gluroo); publikacja z chmury. `[ROZMOWA]`

**D-07 · Prompt dla Desktopu w etapach: kontekst → sekrety → pętla zwrotna → funkcja, z jawnym „stop", gdy
sekret jest już w historii gita.**
Uzasadnienie: `.gitignore` nie usuwa pliku z historii; przepisanie historii to decyzja użytkownika. `[ROZMOWA]`

**D-08 · Ograniczenia implementacyjne O-1..O-8 (handout §3) jako wiążące dla każdej zmiany.**
Najważniejsze: O-1 każda wartość glikemii z widocznym czasem pomiaru i wiekiem; O-2 alerty tylko ze
źródeł < 10 min; O-3 nieoficjalne API tylko jako PoC; O-7 lista urządzeń w konfiguracji; O-8 aplikacja
nie instruuje, jak obejść aplikację producenta. `[RESEARCH]`
**Uwaga o napięciu:** patrz 3.3.

**D-09 · Kolejność ścieżek ingestu:** import plików → Nightscout REST → Dexcom sandbox → Health Connect/
HealthKit → agregator → Abbott bezpośrednio. Uzasadnienie: handout §5 — pokrycie sprzętowe vs koszt vs
bariera wejścia; Nightscout/Juggluco pokrywa więcej refundowanych systemów niż wszystkie oficjalne API razem.
Odrzucone na MVP: agregatory (koszt per user, nie skracają opóźnienia Dexcoma), API Medtronic/Roche/
Sibionics/Medtrum/Eversense (nie istnieją publicznie), Dexcom real-time (tylko zaproszeni). `[RESEARCH]`

**D-10 · Wnioski z Gluroo przyjęte jako wytyczne:** kopiować — strona statusu per źródło, backend zgodny
z Nightscout, model wieloopiekunowy, QR z opakowań → przypomnienia, portal lekarza; nie kopiować — zbieranie
haseł do Dexcom/LLU, scraping powiadomień jako ścieżka produkcyjna, disclaimer jako listek figowy nad
alertami/AI, hosting pod USA rozciągnięty na UE, darmowość bez modelu. `[RESEARCH]`

### 3.2 Decyzje architektoniczne autora aplikacji, odczytane z polityki 1.2 (podjęte przed tą sesją)

Te decyzje **nie zostały podjęte w rozmowie**, ale są fundamentem produktu i muszą być znane odbiorcy:

| # | Decyzja | Uzasadnienie podane w polityce |
|---|---|---|
| A-1 | **Zero zaplecza serwerowego** — brak konta, logowania, synchronizacji, kopii po stronie twórcy | „Nie powstało miejsce, w którym dane mogłyby zostać zgromadzone, a więc z którego mogłyby wyciec. Minimalizacja jako konsekwencja architektury, nie polityki" (§12) |
| A-2 | Twórca **nie jest administratorem** danych; przetwarzanie użytkownika poza RODO (art. 2 ust. 2 lit. c) | brak jakiegokolwiek przekazania danych twórcy (§1) |
| A-3 | **Privacy by default**: wszystkie źródła poza wpisami ręcznymi domyślnie wyłączone; zgoda per typ Health Connect | §12 |
| A-4 | **LibreLinkUp dopuszczone** jako poziom 6 — po osobnym oświadczeniu z datą; hasło niezapisywane; token w Keystore; prawnie oparte na umowie użytkownika z Abbott | §3, §6, §9 |
| A-5 | **Odczyt powiadomień** dopuszczony jako poziom 5 — tylko wskazane aplikacje, tylko liczby, odrzucenie na wejściu | §3, §4 |
| A-6 | **Pozycja „nie wyrób medyczny"** zrealizowana konstrukcją: nie ocenia, nie zaleca, nie prognozuje, **nie ostrzega** | §5, §14 |
| A-7 | AI wyłącznie **BYOK** (klucz użytkownika), zgoda per zapytanie, bez danych identyfikujących | §6, §7 |
| A-8 | Kopia Google i transfer urządzenia **wyłączone** mimo kosztu UX | §9 |
| A-9 | Brak reklam, analityki, crash-reportingu | §7, §12 |
| A-10 | Aplikacja dla **pełnoletnich**; brak zbierania danych < 16 lat | §13 |
| A-11 | Źródła uporządkowane „od najpewniejszego prawnie do najbardziej spornego" i tak komunikowane użytkownikowi | §3 |

### 3.3 Napięcia między decyzjami — do świadomego rozstrzygnięcia

1. **O-3 (handout) vs A-4 (aplikacja).** Handout mówi: nieoficjalne API tylko jako PoC. Aplikacja
   **ma** LibreLinkUp w produkcji (poziom 6). Autor złagodził ryzyko lepiej niż Gluroo (oświadczenie,
   brak hasła, token w Keystore, prawna podstawa w umowie użytkownika z Abbott, O-8 zachowane — aplikacja
   „nie obchodzi żadnych zabezpieczeń"). Ryzyka techniczne R-02 z oceny Gluroo (przerwy po zmianie T&C LLU)
   i R-01 (odcięcie przez producenta) **pozostają**. Rekomendacja: utrzymać poziom 6, ale (a) zaktualizować
   handout O-3 na „nieoficjalne API dopuszczone wyłącznie na koncie i z inicjatywy użytkownika, nigdy
   jako jedyne źródło, z jawnym stanem 'nie płynie od X min'", (b) wdrożyć sygnał wczesny z oceny Gluroo §5.
2. **A-5 (powiadomienia) vs R-03 (kruchość scrapingu).** Ścieżka produkcyjna, ale ograniczona do tego, co
   użytkownik sam wskaże, i bez alertów na jej podstawie — spójne z O-2. Rekomendacja: zostawić, oznaczyć
   w UI jako „zależne od aplikacji źródłowej".
3. **A-6 („nie ostrzega") vs oczekiwania rynku (GotCGM).** Ocena Gluroo pokazuje, że najczęściej używaną
   wartością jest „widzę odczyt na nadgarstku". Szugers ma plakietkę na zegarku, ale **bez alertów** —
   to jest cena pozycji „nie wyrób medyczny". Decyzja Q-1 z handoutu (klasyfikacja MDR) jest realnie
   decyzją: „czy kiedykolwiek chcemy ostrzegać". Dziś odpowiedź brzmi „nie" i jest konsekwentna.

### 3.4 Opcje odrzucone — zbiorczo

| Opcja | Dlaczego odrzucona | Gdzie |
|---|---|---|
| Jedna plakietka z siatką 60 dni | użytkownik wybrał listę osobnych plakietek | D-03 |
| Lista pionowa | gorzej czyta się jako oś czasu, zabiera wysokość | D-04 |
| Artefakt HTML / dokument chmurowy jako nośnik researchu | nieczytelny dla agenta jako kontekst repo | D-01 |
| Agregatory na MVP | koszt per user; nie skracają 3 h Dexcoma | D-09 |
| Próby API Medtronic, Roche, Sibionics, Medtrum, Eversense | nie istnieją publicznie | handout U-16..U-19 |
| Dexcom real-time jako plan | tylko zaproszeni partnerzy | U-13 |
| Praca z telefonu bez CI i ochrony `main` | zmiany w ciemno w aplikacji liczącej glikemię | D-06 |
| Publikacja/podpisywanie z chmury | keystore nie może opuścić desktopu | D-06 |
| Automatyczny wpis estymaty AI do logu bez potwierdzenia | ryzyko R-06 (AI jako wsparcie dawkowania) | ocena Gluroo |
| Tworzenie PR z gałęzi docs | użytkownik nie prosił | 2.1 |

---

## 4. Artefakty powstałe w tej sesji

W tej sesji **nie powstał kod**. Powstały cztery dokumenty i jeden prompt. Wszystkie są w repozytorium
na gałęzi `claude/cgm-refundacja-api-poland-0v36kx` i są dołączone jako osobne pliki do tego przekazania.

| ID | Plik | Zawartość | Rozmiar |
|---|---|---|---|
| Z-01 | `docs/cgm-refundacja-i-integracje-pl.md` | Pierwszy raport badawczy: refundacja CGM w PL (grupy R.03–R.05, limity, dopłaty, 8 rodzin systemów), możliwości API per producent, ranking ścieżek integracji, aspekty MDR/RODO, macierz pokrycia, bibliografia | 20 kB |
| Z-02 | `docs/handout-cgm-integracje.md` | Handout dla Claude Code: instrukcja użycia, 25 ustaleń U-01..U-25 z poziomami pewności i przypisami S1..S39, 8 ograniczeń O-1..O-8, kontrakt danych `GlucoseReading`, specyfikacja ścieżek ingestu, kolejność prac, pytania otwarte Q-1..Q-7 | 23 kB |
| Z-03 | `docs/ocena-ryzyk-gluroo.md` | Ocena ryzyk Gluroo: profil, architektura pozyskiwania danych, 17 ryzyk R-01..R-17 z P/S i dowodami, mapa ciepła, wnioski dla Szugers (kopiować / nie kopiować / różnicować), sygnały wczesne, 28 źródeł | 23 kB |
| Z-04 | `docs/handover/prompt-desktop-cloud-ready-and-history.md` | Gotowy prompt dla Claude Code Desktop realizujący D-05..D-07 w czterech etapach (kontekst, sekrety, pętla zwrotna, funkcja „Historia") | 4 kB |
| Z-05 | `docs/handover/RAPORT-PRZEKAZANIA.md` | ten raport | — |

Pliki istniejące wcześniej, których **nie tworzyłem** i nie modyfikowałem, a które są częścią projektu:
`index.html`, `logo.svg`, `logo.png`, `CNAME`, `polityka-prywatnosci.html`, `harmo/*.html`, `media/*.mp4`.

---

## 5. Źródła, dokumenty, normy, konta i usługi

### 5.1 Dokumenty własne projektu

| Dokument | Adres | Wersja / data |
|---|---|---|
| Polityka prywatności Szugers | https://szugers.com/polityka-prywatnosci.html | 1.2 z 21.09.2026 (ostatnia zmiana treści 18.09.2026); wbudowana także w aplikację |
| Polityka prywatności Harmo | https://szugers.com/harmo/polityka-prywatnosci.html | 2.0 z 07.11.2026; PL/EN/PT/ES/CS/SK/SL |
| Polityka prywatności Harmo (archiwalna) | https://szugers.com/harmo/polityka-prywatnosci-1.0.html | 1.0 z 22.10.2026 |
| Film FGS Szugers | https://szugers.com/media/plakietka-fgs.mp4 | 20.09.2026 |
| Film FGS Harmo | https://szugers.com/media/harmo-fgs.mp4 | 08.11.2026 |

### 5.2 Konta i usługi (nazwy, bez poświadczeń)

| Usługa | Identyfikator / rola | Uwagi |
|---|---|---|
| GitHub | konto `bpietrzkowski`; repo `szugers-site` (publiczne), `gra-w-biznes`, `BASUdriver` (prywatne) | aplikacja Claude GitHub zainstalowana co najmniej dla `szugers-site` |
| GitHub Pages | hosting `szugers.com` | przez `CNAME` |
| Domena `szugers.com` | rejestrator **nieznany** w tej sesji | do uzupełnienia przez odbiorcę |
| Google Play Console | konto deweloperskie — twórca wskazany w politykach: Bartosz Pietrzkowski | deklaracje FGS special use (oba produkty), Play Billing (Harmo), Health Connect (Szugers) |
| Claude Code | claude.ai/code — sesje chmurowe; Claude Code Desktop lokalnie | ta sesja: `session_01EXnhvzWnLJjb6P5ZwjsgLG` |
| Anthropic API / Google AI Studio | **klucze użytkowników końcowych (BYOK)**, nie projektu | aplikacja nie ma własnego klucza |
| Abbott LibreLinkUp | **konta użytkowników końcowych**, nie projektu | aplikacja łączy się na koncie użytkownika |
| Health Connect (Google) | magazyn systemowy na urządzeniu | deklaracja uprawnień w Play |
| Kontakt publiczny | bartosz@pietrzkowski.com — adres opublikowany w obu politykach jako kontakt twórcy | |

### 5.3 Normy, akty prawne, standardy techniczne

| Norma / akt | Zastosowanie w projekcie |
|---|---|
| **RODO** (UE 2016/679): art. 2 ust. 2 lit. c (wyłączenie domowe), art. 6 ust. 1 lit. a/b/c/f, art. 7 ust. 3, art. 9 ust. 1–2 (dane o zdrowiu), art. 15–22, art. 32, art. 37, rozdz. V | podstawa obu polityk prywatności; pozycja „twórca nie jest administratorem" |
| **MDR** (UE 2017/745), reguła 11 klasyfikacji oprogramowania | pozycja Szugers „nie wyrób medyczny" i jej konsekwencje (brak alertów, brak dawek) |
| **AI Act** (UE 2024/1689) | wzmianka w ocenie Gluroo R-06; przy BYOK i braku porad medycznych ekspozycja minimalna — do potwierdzenia prawnie |
| Rozporządzenie MZ ws. wykazu wyrobów medycznych wydawanych na zlecenie — tekst jednolity Dz.U. 2025 poz. 1038; projekt MZ 1904 z 15.06.2026 | refundacja CGM w PL (grupy R.03–R.05) |
| Bluetooth SIG **Glucose Service 0x1808** | odczyt glukometrów BT (poziom 2) |
| Android: Health Connect, NotificationListenerService, Keystore (AES-GCM), FGS `SPECIAL_USE`, polityki Google Play dla aplikacji zdrowotnych i usług na pierwszym planie | architektura Szugers; deklaracje w Play |
| Chrome Extensions Manifest V3, Web Bluetooth, Web Serial | Harmo dla Chrome |
| Nightscout REST API (`/api/v1/entries`, `/api/v1/treatments`) | planowany adapter (D-09) |
| Dexcom API v3 (OAuth 2.0; `/v3/users/self/egvs`, `/devices`, `/events`, `/calibrations`, `/dataRange`) | planowany sandbox (D-09) |

### 5.4 Bibliografia badań (skonsolidowana z Z-01, Z-02, Z-03)

**Refundacja w Polsce**
- Obwieszczenie MZ z 16.06.2025 — tekst jednolity (Dz.U. 2025 poz. 1038): https://isap.sejm.gov.pl/isap.nsf/DocDetails.xsp?id=WDU20250001038
- RPD — MZ o refundacji monitorowania glikemii (14.05.2026): https://brpd.gov.pl/2026/05/14/refundacja-monitorowania-glikemii-mz-odpowiada-rpd/
- opieka.farm — ujednolicenie refundacji CGM-RT/FGM (interpelacja 17968): https://opieka.farm/mz-interpelacja-17968-refundacja-cgm-fgm/
- opieka.farm — wykaz finansuje grupy, nie marki: https://opieka.farm/mz-refundacja-cgm-cukrzyca-typu-2/
- Medycyna Praktyczna — Simplera i Dexcom ONE+ w refundacji: https://www.mp.pl/insulinoterapia/refundacja/344046,nowe-systemy-cgm-refundowane-w-polsce-simplera-i-dexcom-one
- Medycyna Praktyczna — jak wystawić zlecenie na CGM-RT: https://www.mp.pl/insulinoterapia/refundacja/315582,jak-wystawic-zlecenie-na-refundowany-system-ciaglego-monitorowania-glikemii-w-czasie-rzeczywistym
- PSD — wykaz wyrobów medycznych na zlecenie: https://diabetyk.org.pl/wykaz-wyrobow-medycznych-wydawanych-na-zlecenie/
- PTD — stanowisko ws. refundacji CGM (PDF): https://ptdiab.pl/images/aktualnosci/StanowiskoPTDiKOnsultantaiMZ.pdf
- Dexcom PL — refundacja: https://dexcom.pl/refundacja/
- Abbott — refundacja FreeStyle Libre: https://www.freestyle.abbott/pl-pl/refundacja.html
- Sibionics GS1 — refundacja: https://sibionicscgm.com.pl/refundacja/
- Medtrum TouchCare — refundacja: https://touchcarecgm.pl/refundacja/
- Accu-Chek SmartGuide: https://www.accu-chek.pl/produkty/smartguide-cgm
- Diabetyk24 — Eversense E3: https://diabetyk24.pl/refundacja-nfz-sensor-eversense-e3

**API producentów i integracje**
- Dexcom API v3 — endpointy: https://developer.dexcom.com/docs/dexcomv3/endpoint-overview/
- Dexcom API — getting started: https://developer.dexcom.com/docs/dexcom/getting-started/
- Dexcom API — scopes & access: https://developer.dexcom.com/docs/dexcom/scopes-access/
- Momentum — Dexcom API developer's guide: https://www.themomentum.ai/blog/dexcom-api-integration-developer-guide
- Dexcom — FDA clears real-time APIs: https://investors.dexcom.com/news/news-details/2021/FDA-Clears-Dexcom-Real-Time-APIs-for-Third-Party-Apps-and-Devices/default.aspx
- MedTech Dive — Dexcom real-time APIs: https://www.medtechdive.com/news/dexcom-wins-fda-nod-for-real-time-apis-allowing-third-party-developers-acc/603470/
- Abbott — partner integrations: https://www.diabetescare.abbott/partnerships/integrations/en.html
- Junction — LibreView integration: https://docs.junction.com/wearables/guides/abbott-libreview
- Thryve — FreeStyle Libre integration: https://www.thryve.health/features/connections/abbott-freestyle-libre-integration
- Validic — Abbott API: https://help.validic.com/space/VCS/4287823892/Abbott+API+Integration+for+Developers
- LibreView Unofficial (Stoplight): https://libreview-unofficial.stoplight.io/
- GitHub — FokkeZB/libreview-unofficial: https://github.com/FokkeZB/libreview-unofficial
- GitHub — aud/dexcom-share-api: https://github.com/aud/dexcom-share-api
- xDrip — dyskusja Simplera/CareLink: https://github.com/NightscoutFoundation/xDrip/discussions/3267
- DiaTribe — Roche Accu-Chek Connect + Apple Health: https://diatribe.org/roche-integrates-accu-chek-connect-glucose-meter-apple-health
- App Store — Accu-Chek SmartGuide app: https://apps.apple.com/si/app/accu-chek-smartguide-app/id6503455257
- Sibionics — FAQ: https://www.sibionicscgm.com/pages/faq
- Senseonics — Eversense + Glooko: https://www.senseonics.com/investor-relations/news-releases/2019/03-05-2019-130312706
- Nightscout — supported uploaders: https://nightscout.github.io/uploader/uploaders/
- Nightscout Pro — supported devices: https://nightscout.pro/knowledge-base/supported-devices-and-compatibility/
- Juggluco (Google Play): https://play.google.com/store/apps/details?id=tk.glucodata
- Terra — Dexcom integration: https://tryterra.co/integrations/dexcom
- ROOK — competitors: https://www.tryrook.io/competitors
- Dexcom — apps: https://www.dexcom.com/en-us/apps
- LibreSync: https://libresync.com/
- Dexcom — Clarity export: https://www.dexcom.com/en-us/faqs/clarity/using-clarity
- Diabetotech — CGM to smartwatch: https://www.diabetotech.com/cgm-to-smartwatch
- Fortune — Dexcom outage 2019: https://www.fortune.com/2019/12/02/dexcom-outage-blackout-diabetes-patients-blood-sugar-monitor
- FUDiabetes — Sugarmate losing Dexcom access: https://forum.fudiabetes.org/t/sugarmate-losing-dexcom-g6-data-access/11962?page=4
- punktum — wearable data integration, MDR: https://punktum.net/insights/wearable-health-data-integration-apis-architecture-regulatory/

**Gluroo**
- FAQ: https://gluroo.com/support/faqs/ · User Manual: https://gluroo.com/user-manual/ · Privacy: https://gluroo.com/privacy-policy/
- Medical Warnings: https://gluroo.com/medical-warnings/ · AI Copilot: https://gluroo.com/ai/ · For Providers: https://gluroo.com/for-providers/
- System Status: https://gluroo.com/system-status/ · LLU T&C issue: https://gluroo.com/support/libre-cgm-issues-with-gluroo-accepting-new-terms-and-conditions-in-the-librelinkup-app/
- GLI troubleshooting: https://gluroo.com/support/gluroo-local-integration-troubleshooting/
- Smartwatch readings: https://gluroo.com/blog/glucrew/blood-sugar-readings-smartwatch-gluroo/
- Release 2.1.64: https://gluroo.com/blog/releases/gluroo-2-1-64-new-android-always-on-cgm-display-better-iphone-live-activity-cgm-display-more/
- Release Dec 2024: https://gluroo.com/blog/releases/gluroo-dec-2024-feature-release-better-charts-heads-up-display-daily-view-and-health-care-provider-support/
- GotCGM: https://gluroo.com/blog/releases/announcing-gotcgm-powered-by-gluroo/ · https://gotcgm.com/for-gluroo-users/
- Gluroo vs mySugr: https://gluroo.com/blog/life-with-diabetes/gluroo-vs-mysugr-diabetes-apps/
- Parents: https://gluroo.com/blog/glucrew/diabetes-app-parents-are-raving-about/
- T1D Exchange: https://t1dexchange.org/a-new-app-on-the-market-gluroo/ · CB Insights: https://www.cbinsights.com/company/gluroo/people
- Google Play: https://play.google.com/store/apps/details?id=com.gluroo.app&hl=en_US
- Healthline: https://www.healthline.com/healthy/gluroo-the-simplest-yet-most-comprehensive-diabetes-tool-you-may-ever-need
- Savvy Diabetic (2022): https://thesavvydiabetic.com/savvy-apps-10-6-22-explore-gluroo-with-greg-badros-ceo-developer/
- CamDiab — notifications/OS: https://camdiab.com/notifications

---

## 6. Otwarte problemy, ryzyka, zadania i następne kroki

### 6.1 Zadania według priorytetu

| Pri | ID | Zadanie | Dlaczego teraz | Zależy od |
|---|---|---|---|---|
| **P0** | Z-01 | **Decyzja: gdzie żyje kod aplikacji** — GitHub (prywatne repo, aplikacja Claude GitHub) czy wyłącznie lokalnie | warunek pracy z chmury/telefonu (D-06) i jakiejkolwiek pracy agenta nad kodem poza Desktopem | — |
| **P0** | Z-02 | Higiena sekretów przed pierwszym pushem: skan katalogu **i historii**, `.gitignore`, syntetyczne fixtures, `docs/SECRETS.md` | nieodwracalne, jeśli zrobione po fakcie | Z-01 |
| **P0** | Z-03 | **Decyzja Q-1 (MDR):** potwierdzić na piśmie pozycję „przeglądarka danych, bez alertów" jako świadomą, z listą funkcji zakazanych (alerty, dawki, prognozy) | blokuje każdą rozmowę o powiadomieniach/alarmach; dziś spójna, ale nieudokumentowana jako decyzja | — |
| **P1** | Z-04 | CI (build + testy) + SessionStart hook + ochrona `main` | pętla zwrotna dla sesji bez ekranu | Z-01 |
| **P1** | Z-05 | **Testy jednostkowe obliczeń statystyk** (średnie, TIR, zakresy dat, północ, DST, mieszane jednostki) | przed refaktorem „Średnie" | kod |
| **P1** | Z-06 | **Stat → „Historia"** wg D-03..D-05 — prompt Z-04 gotowy | zaakceptowana funkcja | Z-05 |
| **P1** | Z-07 | Merge gałęzi `claude/cgm-refundacja-api-poland-0v36kx` do `main` **albo** przeniesienie `docs/` do repozytorium wspólnego projektu | dokumenty wiszą na niezmergowanej gałęzi | decyzja odbiorcy |
| **P1** | Z-08 | Zaktualizować handout O-3 zgodnie z 3.3 pkt 1 (LLU w produkcji na koncie użytkownika) | handout jest kontekstem dla agenta i dziś przeczy stanowi aplikacji | — |
| **P2** | Z-09 | Adapter **Nightscout REST** (dziś tylko przez Health Connect) | największe pokrycie refundowanych systemów za zero umów | Z-05 |
| **P2** | Z-10 | Rejestracja **Dexcom Developer sandbox** | proces trwa tygodnie–miesiące; start niezależny od kodu | — |
| **P2** | Z-11 | Weryfikacja w źródle pierwotnym: refundacja **Libre 3/3+** w PL i grupa; kryterium pompy < 26 r.ż.; skutki projektu MZ 1904 | komunikacja i ewentualny moduł uprawnień | — |
| **P2** | Z-12 | Stan źródła w UI: „nie płynie od X min, powód" dla LLU i powiadomień | R-02/R-15 z oceny Gluroo | kod |
| **P3** | Z-13 | Publiczna **strona statusu** per źródło (wzór Gluroo) — nawet statyczna na szugers.com | tanie zaufanie | — |
| **P3** | Z-14 | Funkcje refundacyjne PL (kalkulator grup R.03–R.05, licznik sztuk, przypomnienie o zleceniu) | przewaga lokalna niedostępna dla Gluroo | Z-11 |
| **P3** | Z-15 | Landing szugers.com: zastąpić „coming soon" treścią produktową, gdy Play opublikuje | marketing | publikacja |
| **P3** | Z-16 | Przegląd kwartalny dokumentów Z-01..Z-03 (wykaz refundacyjny zmienia się obwieszczeniami) | aktualność kontekstu agenta | — |

### 6.2 Ryzyka (skonsolidowane; szczegóły w Z-03 i Z-02)

| Ryzyko | P | S | Mitygacja istniejąca | Do zrobienia |
|---|---|---|---|---|
| Abbott zmienia T&C/API LibreLinkUp → dane nie płyną | W | Ś | źródło opcjonalne, oświadczenie, inne poziomy źródeł | Z-12; komunikat w aplikacji |
| Zmiana formatu powiadomień aplikacji źródłowej → poziom 5 przestaje działać | W | Ś | filtr tylko wskazanych aplikacji; brak alertów | Z-12; oznaczenie „zależne od aplikacji źródłowej" |
| Sekret trafi do publicznego/prywatnego repo przy przenosinach kodu | Ś | W | — | Z-02 (przed Z-01!) |
| Dryf pozycji regulacyjnej (ktoś doda „alert", bo rynek tego chce) | Ś | W | polityka §14, konstrukcja bez alertów | Z-03 — decyzja na piśmie; O-2 w CLAUDE.md |
| Zmiany akceptowane z telefonu bez testów → błąd w obliczeniach glikemii | Ś | W | — | Z-04, Z-05 |
| Utrata danych użytkownika przy zmianie telefonu (kopia wyłączona, A-8) | W | Ś | eksport CSV/baza | komunikat przy pierwszym uruchomieniu; rozważyć eksport przypominany |
| Google Play odrzuci deklarację FGS special use / Health Connect | Ś | W | filmy FGS przygotowane | brak informacji o wyniku przeglądu — do sprawdzenia |
| Ryzyko kluczowej osoby (jeden twórca, dwa produkty, 7 języków Harmo) | Ś | W | — | przekazanie do spółki (ten raport), dokumentacja |
| Research oparty o wyszukiwarkę, nie o akty prawne | Ś | Ś | oznaczenia pewności | Z-11 |

### 6.3 Pytania otwarte (bez odpowiedzi w tej sesji)

1. Czy Szugers jest już opublikowany w Google Play (produkcja / zamknięte testy / beta otwarta)? Jaki
   wynik miały deklaracje FGS i Health Connect?
2. Stos technologiczny aplikacji (język, framework, baza, min SDK) — do uzupełnienia z kodu.
3. Gdzie jest kod Szugers i Harmo; czy jest w jakimkolwiek systemie kontroli wersji poza lokalnym.
4. Rejestrator domeny `szugers.com` i konto, na którym jest utrzymywana.
5. Edytor ISO — czym jest, gdzie jest kod, jaki ma status.
6. Spółka — forma prawna, nazwa, status rejestracji; czy ma już przejąć rolę wydawcy w Play i stronę
   w politykach prywatności (dziś: osoba fizyczna).
7. Wszystkie pytania Q-1..Q-7 z handoutu i Q-1..Q-5 z oceny Gluroo.

---

## 7. Sprawy firmowe i prawne

### 7.1 Podmiot i role (stan dzisiejszy)

`[FAKT]` Obie polityki wskazują jako twórcę/udostępniającego **osobę fizyczną — Bartosza Pietrzkowskiego**,
Harmo „pod marką Szugers". `[ROZMOWA]` Projekt przechodzi do wspólnego projektu ze **spółką, która je
sprzedaje**. Konsekwencje przeniesienia na spółkę:

- **Polityki prywatności wymagają nowej wersji** w momencie, gdy spółka stanie się wydawcą: zmienia się
  podmiot w pkt 1 obu polityk, administrator danych zamówień i korespondencji (Harmo pkt 6–7), ewentualnie
  adres korespondencyjny. Harmo pkt 13 i Szugers §16 zobowiązują do publikacji zmiany **przed** jej
  wejściem w życie i do zachowania poprzednich wersji pod tym samym adresem.
- **Google Play**: przeniesienie aplikacji między kontami deweloperskimi (osoba fizyczna → spółka) to
  osobna procedura w Play Console; wymaga konta spółki z weryfikacją organizacji.
- **Znaki towarowe**: „Szugers" i „Harmo" — brak informacji o zgłoszeniu do UPRP/EUIPO. Przy wejściu
  spółki warto to rozstrzygnąć (Harmo ma rynek w 7 krajach).

### 7.2 RODO — przyjęta pozycja i jej granice

`[FAKT]` Pozycja obu polityk: brak przetwarzania danych przez twórcę → twórca **nie jest administratorem**
danych w aplikacji; przetwarzanie użytkownika na własnym urządzeniu objęte wyłączeniem domowym
(art. 2 ust. 2 lit. c). Brak IOD (art. 37 nie powstaje). Rozdział V (transfery) nie dotyczy twórcy.

`[WNIOSEK]` Pozycja jest konsekwentna i dobrze uzasadniona architekturą A-1. Jej granice:
- **Harmo**: spółka/twórca **jest** administratorem danych zamówień (art. 6 ust. 1 lit. c i f) i korespondencji
  (lit. f) — polityka 2.0 to uczciwie opisuje; rejestr czynności i retencja podatkowa po stronie wydawcy.
- **Szugers**: analogicznie przy korespondencji e-mail (polityka 1.2 tego wprost nie opisuje — Harmo 2.0
  już tak; **rekomendacja: dopisać w Szugers 1.3**).
- Każda przyszła funkcja z „kontem Szugers" (Harmo pkt 13) przewraca całą pozycję: twórca staje się
  administratorem danych o zdrowiu (art. 9). To jest decyzja strategiczna, nie techniczna.
- Organ nadzorczy: **Prezes UODO**, ul. Stawki 2, 00-193 Warszawa — wskazany w obu politykach.

### 7.3 MDR — pozycja „nie wyrób medyczny"

`[FAKT]` Szugers §14: nie ocenia, nie zaleca, nie prognozuje, nie ostrzega; wykresy insuliny poglądowe;
decyzje na podstawie urządzeń medycznych i lekarza. `[RESEARCH]` Reguła 11 MDR obejmuje oprogramowanie
dostarczające informacji do decyzji diagnostycznych/terapeutycznych (kl. IIa, IIb przy ryzyku poważnego
pogorszenia zdrowia).

`[WNIOSEK]` Pozycja Szugers jest **obronna przez konstrukcję**, nie przez disclaimer — to zasadnicza
różnica wobec Gluroo (R-05). Lista funkcji, które ją przekreślą: alerty hipo/hiper, sugestie dawek,
prognozy glikemii, automatyczny wpis estymaty AI do logu. **Z-03** ma to zapisać jako decyzję.
Rekomendacja: krótka **opinia prawna** (kwalifikacja MDR + AI Act) przed wejściem spółki jako wydawcy —
koszt niewielki, a zamyka największe pytanie strategiczne.

### 7.4 Google Play — wymogi, które projekt już realizuje lub musi

| Wymóg | Szugers | Harmo |
|---|---|---|
| Deklaracja FGS `SPECIAL_USE` z filmem | film 20.09.2026 | film 08.11.2026 |
| Deklaracja uprawnień Health Connect (polityka Health apps) | wymagana — aplikacja czyta/pisze HC | nie dotyczy |
| Formularz Data safety | lokalne przetwarzanie; LLU, AI BYOK jako transfery inicjowane przez użytkownika | brak sieci; Play Billing |
| Polityka prywatności pod publicznym URL | szugers.com/polityka-prywatnosci.html | szugers.com/harmo/polityka-prywatnosci.html |
| NotificationListener — uzasadnienie w Play | wymagane (poziom 5) | nie dotyczy |
| Play Billing / podatki / zwroty | nie dotyczy | tak; prawo konsumenckie UE — zwroty przez Google Play |

### 7.5 Regulacje refundacyjne (otoczenie rynkowe)

`[RESEARCH]` Rozporządzenie MZ ws. wykazu (Dz.U. 2025 poz. 1038), grupy R.03.01–R.05.02, dopłaty 20%/30%,
limit 970 zł; projekt MZ 1904 z 15.06.2026 w toku. Wydatki NFZ na systemy monitorowania ~542 mln zł (2025).
Dla produktu: argument za funkcjami refundacyjnymi (Z-14) i za obsługą systemów refundowanych w PL
(8 rodzin — handout U-05). Termin przeglądu: po wejściu MZ 1904 w życie, potem kwartalnie (Z-16).

### 7.6 Znaki towarowe osób trzecich

`[FAKT, Szugers §15]` FreeStyle, Libre, LibreLinkUp, LibreView (Abbott); Dexcom (Dexcom, Inc.); NovoPen
(Novo Nordisk A/S) — użyte wyłącznie dla wskazania kompatybilności; brak powiązania. Przy rozszerzaniu
źródeł (Nightscout, Sibionics, Medtrum, Roche) dopisać analogicznie.

### 7.7 Terminy

| Co | Kiedy | Źródło |
|---|---|---|
| Szugers polityka 1.2 — obowiązuje od | 21.09.2026 | polityka |
| Harmo polityka 2.0 — obowiązuje od | 07.11.2026 | polityka |
| Harmo — okres próbny użytkownika | 5 tygodni od pierwszego uruchomienia | polityka pkt 6 |
| Projekt MZ 1904 — udostępniony | 15.06.2026; wejście w życie nieznane | research |
| Przegląd dokumentów badawczych | kwartalnie; najbliższy: grudzień 2026 | Z-16 |
| Terminy Play (przeglądy deklaracji) | **nieznane** w tej sesji | — |
| Terminy spółki (rejestracja, przeniesienie Play) | **nieznane** w tej sesji | — |

### 7.8 Kontakty

| Rola | Kontakt | Uwagi |
|---|---|---|
| Twórca / wydawca obu aplikacji | Bartosz Pietrzkowski, bartosz@pietrzkowski.com | adres opublikowany w politykach |
| Organ nadzorczy RODO | Prezes UODO, ul. Stawki 2, 00-193 Warszawa, uodo.gov.pl | wskazany w politykach |
| Abbott (LibreLinkUp), Anthropic, Google — jako dostawcy usług użytkownika końcowego | wg ich polityk | projekt nie ma z nimi umów |
| Partnerstwa do rozpoczęcia | Dexcom Developer Program (sandbox self-service); Abbott partner integrations | Z-10; handout U-12, U-14 |

---

## 8. Załączniki

Dołączone jako osobne pliki do tego przekazania (i obecne w repo na gałęzi `claude/cgm-refundacja-api-poland-0v36kx`):

1. `RAPORT-PRZEKAZANIA.md` — ten dokument
2. `cgm-refundacja-i-integracje-pl.md` — Z-01
3. `handout-cgm-integracje.md` — Z-02
4. `ocena-ryzyk-gluroo.md` — Z-03
5. `prompt-desktop-cloud-ready-and-history.md` — Z-04

Pliki projektu nieutworzone w tej sesji, do pobrania z `main` repozytorium: `index.html`, `logo.svg`,
`logo.png`, `CNAME`, `polityka-prywatnosci.html`, `harmo/polityka-prywatnosci.html`,
`harmo/polityka-prywatnosci-1.0.html`, `media/plakietka-fgs.mp4`, `media/harmo-fgs.mp4`.

---

*Koniec raportu. Sporządzono 2026-11-13 w sesji Claude Code; wszystkie oznaczenia pewności i granice
opisane w nagłówku obowiązują dla całego dokumentu.*
