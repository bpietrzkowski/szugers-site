# Gluroo — ocena ryzyk funkcjonowania

**Wersja:** 1.0 · **Data researchu:** 2026-09-17 · **Metoda:** wyszukiwanie webowe (strony Gluroo,
sklepy aplikacji, prasa branżowa, dokumentacja społeczności) — bez dostępu do kodu, umów ani
rozmów z zespołem Gluroo.
**Cel:** zrozumieć, na jakich założeniach stoi Gluroo i które z tych założeń mogą się zawalić,
żeby szugers świadomie powtórzył dobre decyzje i nie powtórzył złych.

> Poziomy pewności jak w handoucie: `[POTWIERDZONE]` — wiele źródeł lub źródło własne Gluroo;
> `[PRAWDOPODOBNE]` — jedno wiarygodne źródło; `[DO WERYFIKACJI]` — wnioskowanie.
> Ocena ryzyka: **P** = prawdopodobieństwo (N/Ś/W), **S** = skutek (N/Ś/W).

---

## 0. Streszczenie

Gluroo to bardzo dobrze zaprojektowany produkt, który stoi na **czterech kruchych filarach**:

1. **Dane pobiera z nieoficjalnych API producentów** (Dexcom Share i LibreLinkUp na login/hasło
   użytkownika) — dokładnie tą drogą, którą Sugarmate straciło dostęp do danych Dexcoma. `[S13][S14]`
2. **Deklaruje „nie jesteśmy wyrobem medycznym"**, a jednocześnie pokazuje glikemię w czasie
   rzeczywistym na zegarku i ekranie blokady, wysyła alerty opiekunom i ma asystenta AI liczącego
   węglowodany ze zdjęcia. W UE to jest opis wyrobu medycznego klasy IIa+ z reguły 11 MDR. `[S9][S10][S11]`
3. **Nie ma modelu przychodowego** — aplikacja jest darmowa, firma bootstrappowana, bez VC,
   z obietnicą „zawsze za darmo dla tych, dla których płacenie byłoby trudnością". `[S5][S6]`
4. **Jest firmą amerykańską z hostingiem HIPAA**, obsługującą użytkowników w 100+ krajach,
   w tym w UE, gdzie HIPAA nic nie znaczy, a dane glikemiczne dzieci to szczególna kategoria RODO. `[S7][S8]`

Żaden z tych filarów nie jest jeszcze pęknięty. Ale każdy może pęknąć **decyzją kogoś innego**
(Dexcom, Abbott, regulator, sklep aplikacji), nie decyzją Gluroo. To jest esencja ryzyka
tego produktu — i główna lekcja dla szugers.

---

## 1. Profil Gluroo — fakty

| Cecha | Ustalenie | Pewność | Źródło |
|---|---|---|---|
| Podmiot | Gluroo Imaginations, Inc. (USA) | POTWIERDZONE | [S6] |
| Założyciel / CEO | Greg Badros, PhD CS, ex-Google i ex-Facebook; założył po diagnozie T1D u syna | POTWIERDZONE | [S5][S6] |
| Finansowanie | Bootstrap, bez VC; „nie zdecydowaliśmy jeszcze, jak będziemy się finansować" | POTWIERDZONE | [S5] |
| Cena | Bezpłatna dla pacjentów i lekarzy | POTWIERDZONE | [S12] |
| Zespół | 4 osoby na poziomie executive; ludzie z T1D w rodzinach, doświadczenie w big tech i NGO | PRAWDOPODOBNE | [S5][S6] |
| Użytkownicy | Podawane liczby rosną w czasie: 32 000 → 45 000 w 100+ krajach → „blisko 250 000" | DO WERYFIKACJI (rozbieżne źródła) | [S5][S3] |
| Platformy | iOS, Android, web (app.gluroo.com); zegarki (WearOS, Apple Watch) | POTWIERDZONE | [S1][S3] |
| Lokalizacja | UI w wielu językach; mg/dL i mmol/L; **parsowanie wpisów tylko po angielsku** | POTWIERDZONE | [S3] |
| Status regulacyjny | „NIE jest wyrobem medycznym i NIE MOŻE być używana jako wyrób medyczny"; nie zweryfikowana przez FDA; brak śladu CE | POTWIERDZONE | [S9][S10] |
| Produkt siostrzany | **GotCGM** (2025) — wyłącznie routing odczytów CGM na zegarek/lock screen/Dynamic Island, bez logowania | POTWIERDZONE | [S15][S16] |
| Strona statusu | Publiczny system status z osobnymi torami: backend, Dexcom fetching, Libre fetching, web | POTWIERDZONE | [S17] |

### 1.1 Funkcje istotne dla oceny ryzyka

- **GluCrew** — wielu opiekunów loguje w imieniu jednej osoby z cukrzycą; dane w czasie
  rzeczywistym (bolusy, posiłki, ustawienia pompy) dzielone między urządzeniami. `[S1][S11]`
- **AI ze zdjęcia posiłku** — węglowodany, kalorie, tłuszcz, białko, błonnik. `[S11]`
- **@Roo / GluChat** — asystent AI do porównywania posiłków i „poprawy zarządzania cukrzycą". `[S11]`
- **Insulina** — ręcznie (MDI), ze smart penów, automatycznie z DIY Loop / iAPS. `[S11]`
- **Alerty i wyświetlanie real-time** — zegarki, widgety, Live Activity, Always-On Display. `[S18]`
- **Portal dla lekarzy** (grudzień 2024). `[S19]`

### 1.2 Architektura pozyskiwania danych

Cztery drogi, wszystkie oparte o cudzą infrastrukturę: `[S1][S2]`

| Droga | Mechanizm | Charakter |
|---|---|---|
| **Dexcom Share / Follow** | Użytkownik podaje login i hasło Dexcom; Gluroo odpytuje nieoficjalny endpoint Share | nieoficjalne, credentials użytkownika |
| **LibreLinkUp** | Użytkownik podaje login i hasło LLU; Gluroo odpytuje API LLU | nieoficjalne, credentials użytkownika |
| **Gluroo Global Connect (GGC)** | Backend zgodny z Nightscout — wszystko, co potrafi uploadować do Nightscouta, trafia do Gluroo | otwarte, standard społecznościowy |
| **Gluroo Local Integration (GLI)** | Odczyt wartości z **powiadomień Androida** aplikacji Medtronic Guardian 4, CamAPS, xDrip+ | scraping UI, bardzo kruche |

Łańcuch do zegarka: *CGM → chmura producenta → Gluroo → aplikacja pośrednia → zegarek.* `[S2]`
Każdy element to osobny punkt awarii; Gluroo samo opisuje takie konfiguracje jako „złożone,
kruche, z wieloma punktami awarii". `[S20]`

---

## 2. Rejestr ryzyk

### A. Zależności technologiczne od stron trzecich

**R-01 · Utrata dostępu do danych Dexcom przez nieoficjalny endpoint Share** — P: **Ś**, S: **W**
Gluroo używa prywatnego API Dexcom Share, które jest „niewspierane przez Dexcom i może przestać działać".
Precedens: **Sugarmate** (przejęty przez Tandem) przestał otrzymywać dane z Dexcom Share. Dexcom
równolegle buduje oficjalną, zaproszeniową ścieżkę real-time — co obniża jego tolerancję dla
obejść. `[S13][S14][S21]`
*Sygnał wczesny:* zmiany w aplikacji Dexcom Follow, komunikaty o „nieautoryzowanych aplikacjach".
*Dla szugers:* nie budować podstawowej ścieżki danych na credentials użytkownika do cudzej chmury.

**R-02 · Przerwy w danych Libre po każdej zmianie regulaminu LibreLinkUp** — P: **W** (cyklicznie), S: **Ś**
Gluroo ma **dedykowaną stronę wsparcia**: przy każdym serwisie Abbott aktualizuje T&C i dopóki
użytkownik nie zaakceptuje ich w aplikacji LLU, „Gluroo nie ma dostępu do API LLU i dane nie
płyną". Rozwiązanie to instrukcja dla użytkownika, nie fix po stronie Gluroo. `[S4]`
*Interpretacja:* to jest już **zmaterializowane ryzyko o charakterze powtarzalnym**, a nie hipoteza.
Abbott w każdej chwili może zmienić coś więcej niż T&C.
*Dla szugers:* to samo co R-01. Dodatkowo — każde źródło musi mieć w UI jasny stan „dane nie płyną
od X min, powód: …", bo inaczej opiekun patrzy na starą wartość jak na aktualną.

**R-03 · Kruchość Gluroo Local Integration (scraping powiadomień)** — P: **W**, S: **Ś**
Odczyt wartości z powiadomień Androida łamie się przy każdej zmianie formatu powiadomienia w
aplikacji źródłowej, zmianie wersji Androida i polityk tła. Gluroo ma osobny przewodnik
troubleshootingu GLI. CamAPS jednocześnie podnosi minimalne wersje OS. `[S22][S23]`
*Dla szugers:* dopuszczalne tylko jako opcja „power user" z jawnym ostrzeżeniem; nigdy w ścieżce,
na której opiera się alert.

**R-04 · Cała ścieżka danych przez chmurę — brak odczytu bezpośredniego BLE** — P: **Ś**, S: **W**
Gluroo nie łączy się z sensorem; zależy od dostępności chmur Dexcom/Abbott **i** własnego backendu.
Precedens systemowy: awaria Dexcom Follow z 2019 r. odcięła rodziców od danych dzieci na wiele godzin. `[S24]`
*Dla szugers:* to samo ograniczenie dotyczy każdego, kto nie ma umowy z producentem.
Różnica polega na tym, czy UI **uczciwie pokazuje wiek danych** — patrz handout, O-1.

### B. Regulacyjne i prawne

**R-05 · Rozjazd między disclaimerem „nie wyrób medyczny" a rzeczywistą funkcją** — P: **Ś**, S: **W**
Gluroo deklaruje: nie służy do decyzji dawkowania, nie jest wyrobem medycznym. Jednocześnie
pokazuje glikemię real-time na zegarku, ekranie blokady i Always-On Display, wysyła alerty
opiekunom i ma AI liczące węglowodany. W UE reguła 11 MDR klasyfikuje oprogramowanie dostarczające
informacji do decyzji terapeutycznych jako wyrób klasy IIa (IIb przy ryzyku poważnego
pogorszenia zdrowia). Disclaimer nie zmienia **przeznaczenia wynikającego z funkcji**. `[S9][S10][S25]`
*Sygnał wczesny:* pierwsze pytanie organu nadzoru albo sklepu aplikacji o klasyfikację; pozew
po incydencie klinicznym.
*Dla szugers:* to jest **największa różnica strategiczna do zrobienia**. Jeśli szugers pójdzie
drogą MDR (choćby klasy I lub IIa z jednostką notyfikowaną), będzie jedynym takim produktem w
tej niszy w Polsce.

**R-06 · AI (zdjęcie posiłku, @Roo) jako faktyczne wsparcie decyzji dawkowania** — P: **W**, S: **W**
Liczba węglowodanów ze zdjęcia trafia do logu, a z logu — do decyzji o bolusie. To jest wsparcie
decyzji terapeutycznej niezależnie od disclaimerów. Ryzyko: błąd modelu → błędna dawka → hipo/hiper.
Dodatkowo AI Act (systemy wysokiego ryzyka w wyrobach medycznych) i brak oznaczenia, jak model
został zwalidowany klinicznie. `[S11][S10]`
*Dla szugers:* jeżeli wchodzi AI — walidacja na polskich posiłkach, jawna niepewność estymaty,
brak automatycznego wpisu do logu bez potwierdzenia.

**R-07 · HIPAA ≠ RODO; użytkownicy z UE i dane dzieci** — P: **Ś**, S: **W**
Gluroo chwali się „chmurą zgodną z HIPAA" i szyfrowaniem w spoczynku; polityka prywatności ma
sekcję o EOG i prawach z RODO. `[S7][S8]` Nie znaleziono: informacji o lokalizacji danych w UE,
mechanizmie transferu (DPF / SCC), DPIA, ani o obsłudze zgody rodzicielskiej dla dzieci —
a model „GluCrew" jest wprost adresowany do rodziców dzieci z T1D. `[S1][S26]`
*Dla szugers:* dane w UE, DPA z każdym procesorem, jawna ścieżka zgody opiekuna. To jest
przewaga, nie koszt.

**R-08 · Skuteczność wyłączeń odpowiedzialności w prawie konsumenckim UE** — P: **N**, S: **W**
Wyłączenia typu „MUST NOT BE USED AS A MEDICAL DEVICE" w regulaminie B2C mogą być w UE uznane za
klauzule niedozwolone, zwłaszcza gdy funkcja produktu zaprzecza treści wyłączenia. `[S9]`
*Dla szugers:* nie kopiować tego wzorca; odpowiedzialność ograniczać **konstrukcją produktu**
(wiek danych, brak rekomendacji dawek), nie regulaminem.

### C. Bezpieczeństwo

**R-09 · Przechowywanie haseł użytkowników do Dexcom i LibreLinkUp po stronie serwera** — P: **N**, S: **W**
Żeby cyklicznie odpytywać Share i LLU, Gluroo musi przechowywać (a nie tylko hashować) credentials
użytkownika do cudzych usług. To koncentruje w jednym miejscu klucze do danych medycznych
dziesiątek tysięcy osób w dwóch największych ekosystemach CGM. `[S1][S7]`
*Skutek wycieku:* nie tylko dane Gluroo, ale pełny dostęp do kont Dexcom/Abbott użytkowników.
*Dla szugers:* architektura, w której szugers **nigdy nie zna** hasła użytkownika do cudzej usługi
(OAuth u Dexcoma, token Nightscouta generowany przez użytkownika, agregator z własnym consent flow).

**R-10 · Naruszenie ToS producentów przez użytkowników** — P: **Ś**, S: **Ś**
Podanie hasła do Share/LLU aplikacji trzeciej może naruszać regulaminy Dexcom i Abbott. W razie
zaostrzenia polityki to **użytkownik** traci konto — a razem z nim dostęp do własnej aplikacji
producenta. `[S13][S4]`

### D. Biznesowe i organizacyjne

**R-11 · Brak modelu przychodowego** — P: **W**, S: **W**
Bezpłatna, bez VC, z publiczną obietnicą utrzymania darmowego dostępu dla potrzebujących
i jednoczesnym przyznaniem, że model finansowania nie jest ustalony. `[S5][S12]`
*Scenariusze:* (a) przejęcie przez producenta CGM lub firmę pompową — precedens Sugarmate/Tandem —
i zamknięcie integracji z konkurencją; (b) wygaszenie; (c) nagłe wprowadzenie płatności.
*Dla szugers:* użytkownicy Gluroo w Polsce są **realną grupą do przejęcia**, jeśli szugers da im
ścieżkę importu i model, któremu można zaufać na lata.

**R-12 · Ryzyko kluczowej osoby** — P: **Ś**, S: **W**
Produkt jest silnie związany z założycielem (osobiste motywacje, wywiady, LinkedIn). Zespół
executive to 4 osoby obsługujące 100+ krajów i cztery ścieżki integracji. `[S5][S6]`

**R-13 · Rozproszenie fokusu — GotCGM** — P: **Ś**, S: **N**
Wydzielenie GotCGM (2025) sugeruje, że najczęściej używaną wartością Gluroo jest **wyświetlanie
odczytu na zegarku**, a nie kompleksowe logowanie. To samo w sobie mówi dużo o rynku: użytkownicy
płacą uwagą za „glikemia na nadgarstku", nie za dashboardy. `[S15][S16]`
*Dla szugers:* prosty use case „widzę wartość tam, gdzie patrzę" musi działać perfekcyjnie,
zanim powstanie cokolwiek zaawansowanego.

**R-14 · Konkurencja platformowa** — P: **W**, S: **Ś**
Dexcom Follow, LibreLinkUp, CareLink Connect, Apple Health / Health Connect i natywne komplikacje
zegarków systematycznie zjadają najprostszy use case Gluroo. Producent zawsze będzie miał dane
szybciej i bez opóźnień. `[S13][S21]`

### E. Produktowe i kliniczne

**R-15 · Opiekun ufa nieaktualnej wartości** — P: **Ś**, S: **W**
W modelu GluCrew opiekun (rodzic, szkoła) podejmuje decyzję na podstawie wartości na zegarku. Gdy
łańcuch (R-01..R-04) przestaje płynąć bez wyraźnej sygnalizacji, decyzja jest podejmowana na
starych danych. `[S2][S4]`
*Dla szugers:* patrz handout O-1 i O-2. Alerty wyłącznie ze źródeł < 10 min, jawny wiek danych.

**R-16 · Parsowanie wpisów tylko po angielsku** — P: **W** (dla PL), S: **Ś**
Interfejs jest zlokalizowany, ale logowanie tekstowe rozumie wyłącznie angielskie słowa kluczowe. Dla
polskiego rodzica to bariera, która realnie obniża jakość logów. `[S3]`
*Dla szugers:* polski język w warstwie wejścia danych to nie „lokalizacja", tylko funkcja kliniczna.

**R-17 · Nierówne pokrycie systemów refundowanych w Polsce** — P: **W**, S: **Ś**
Gluroo natywnie wspiera Dexcom i Libre; Medtronic tylko przez GLI (scraping); brak wzmianek o
Sibionics, Medtrum, Accu-Chek SmartGuide, Eversense poza drogą Nightscout. `[S1]` W Polsce
te systemy są refundowane i rosną (patrz handout U-05).

---

## 3. Mapa ciepła

| Ryzyko | P | S | Kategoria | Status |
|---|---|---|---|---|
| R-02 Przerwy Libre po zmianach T&C LLU | W | Ś | technologiczne | **zmaterializowane, powtarzalne** |
| R-06 AI jako wsparcie dawkowania | W | W | regulacyjne | latentne |
| R-11 Brak modelu przychodowego | W | W | biznesowe | latentne |
| R-01 Utrata Dexcom Share | Ś | W | technologiczne | precedens (Sugarmate) |
| R-04 Pełna zależność od chmur | Ś | W | technologiczne | precedens (Dexcom 2019) |
| R-05 Disclaimer vs funkcja (MDR) | Ś | W | regulacyjne | latentne |
| R-07 HIPAA≠RODO, dane dzieci | Ś | W | prawne | latentne |
| R-12 Kluczowa osoba | Ś | W | organizacyjne | strukturalne |
| R-15 Opiekun ufa starej wartości | Ś | W | kliniczne | latentne |
| R-03 Kruchość GLI | W | Ś | technologiczne | chroniczne |
| R-14 Konkurencja platformowa | W | Ś | biznesowe | postępujące |
| R-16 Tylko angielski w parsowaniu | W | Ś | produktowe (PL) | stałe |
| R-17 Pokrycie systemów PL | W | Ś | produktowe (PL) | stałe |
| R-10 Naruszenie ToS przez userów | Ś | Ś | prawne | latentne |
| R-09 Hasła do cudzych usług na serwerze | N | W | bezpieczeństwo | strukturalne |
| R-08 Skuteczność disclaimerów w UE | N | W | prawne | latentne |
| R-13 Rozproszenie fokusu (GotCGM) | Ś | N | organizacyjne | obserwowalne |

---

## 4. Co z tego wynika dla szugers

### 4.1 Skopiować (Gluroo zrobiło to dobrze)

1. **Publiczna strona statusu z osobnymi torami per źródło danych.** `[S17]` Tania, buduje
   zaufanie, wymusza monitoring.
2. **Backend zgodny z Nightscout jako uniwersalne wejście** (Gluroo Global Connect). `[S1]`
   To dokładnie ścieżka priorytetu 2 z handoutu — Gluroo potwierdza, że działa w praktyce.
3. **Model wieloosobowy (GluCrew)** — jedna osoba z cukrzycą, wielu opiekunów z rolami. `[S1]`
   W Polsce, gdzie refundacja dla dzieci 4–18 lat jest najhojniejsza (handout U-04), to trafia
   dokładnie w najlepiej refundowaną grupę.
4. **Skanowanie QR z opakowań sensorów/zestawów → przypomnienia o wymianie.** `[S11]`
   Banalne, a w PL łączy się naturalnie z limitami refundacyjnymi (3 sensory / miesiąc).
5. **Portal dla lekarza.** `[S19]` W PL lekarz wystawia e-zlecenie — to naturalny punkt styku.

### 4.2 Nie kopiować (Gluroo poniesie tego koszt)

1. **Zbieranie loginów i haseł do Dexcom / LibreLinkUp.** (R-01, R-02, R-09, R-10)
2. **Scraping powiadomień jako ścieżka produkcyjna.** (R-03)
3. **„Nie jesteśmy wyrobem medycznym" jako listek figowy nad alertami i AI.** (R-05, R-06, R-08)
4. **Hosting i compliance zaprojektowane pod USA, rozciągnięte na UE.** (R-07)
5. **Darmowość bez modelu.** (R-11) — uczciwy, płatny plan od początku jest bardziej wiarygodny
   niż obietnica „może kiedyś".

### 4.3 Zrobić inaczej — przewagi możliwe tylko lokalnie

1. **Refundacja jako funkcja produktu:** kalkulator uprawnień po kodach grup R.03–R.05,
   licznik wykorzystanych sztuk w miesiącu, przypomnienie o zleceniu. Gluroo tego nie ma i mieć nie będzie.
2. **Polski język w warstwie wejścia** — parsowanie wpisów, nazwy posiłków, produkty z polskich sklepów.
3. **Pokrycie systemów refundowanych w PL** — Sibionics, Medtrum, SmartGuide przez Nightscout/Juggluco
   (handout U-20), czyli więcej niż Gluroo natywnie.
4. **Dane w UE, RODO od pierwszego dnia, ścieżka zgody opiekuna.**
5. **Świadoma decyzja o MDR** — nawet klasa I z deklaracją zgodności to więcej, niż ma Gluroo.

---

## 5. Sygnały do obserwacji (early warnings)

| Sygnał | Co oznacza | Gdzie patrzeć |
|---|---|---|
| Wpis „Dexcom fetching: degraded" utrzymujący się > 48 h | R-01 się materializuje | [S17] |
| Nowa strona wsparcia Gluroo o zmianach LLU | kolejny cykl R-02 | [S4], blog Gluroo |
| Ogłoszenie o płatnych planach albo przejęciu | R-11 → decyzja o kierunku | [S6], prasa |
| GotCGM wypiera Gluroo w rankingach sklepów | R-13, zmiana fokusu firmy | sklepy aplikacji |
| Dexcom publikuje ogólnodostępne real-time API | R-01 znika, ale i przewaga Nightscouta maleje | handout U-13 |
| Abbott oficjalnie blokuje aplikacje trzecie w LLU | R-02 → trwałe | [S4], społeczność Nightscout |

---

## 6. Pytania otwarte

| # | Pytanie | Wpływ na ocenę |
|---|---|---|
| Q-1 | Czy Gluroo ma DPF/SCC i gdzie fizycznie leżą dane użytkowników z UE? | R-07 |
| Q-2 | Ilu użytkowników ma Gluroo w Polsce i jakich systemów CGM używają? | R-17, §4.1 pkt 3 |
| Q-3 | Czy Gluroo prowadzi rozmowy o oficjalnym partnerstwie z Dexcom / Abbott? | R-01, R-02 |
| Q-4 | Czy model AI do węglowodanów był walidowany klinicznie i na jakich kuchniach? | R-06 |
| Q-5 | Jaka jest rzeczywista liczba użytkowników (32k / 45k / 250k)? | wiarygodność wszystkich liczb |

---

## 7. Źródła

- `[S1]` [Gluroo — Frequently Asked Questions](https://gluroo.com/support/faqs/)
- `[S2]` [Gluroo — Using Gluroo to get blood sugar readings on your smartwatch](https://gluroo.com/blog/glucrew/blood-sugar-readings-smartwatch-gluroo/)
- `[S3]` [Gluroo — User Manual](https://gluroo.com/user-manual/)
- `[S4]` [Gluroo — Libre CGM issues: accepting new Terms and Conditions in the LibreLinkUp app](https://gluroo.com/support/libre-cgm-issues-with-gluroo-accepting-new-terms-and-conditions-in-the-librelinkup-app/)
- `[S5]` [T1D Exchange — A new app on the market: Gluroo](https://t1dexchange.org/a-new-app-on-the-market-gluroo/)
- `[S6]` [CB Insights — Gluroo: CEO, founder, key executive team](https://www.cbinsights.com/company/gluroo/people)
- `[S7]` [Gluroo — For Providers (HIPAA, hosting, szyfrowanie)](https://gluroo.com/for-providers/)
- `[S8]` [Gluroo — Privacy Policy](https://gluroo.com/privacy-policy/)
- `[S9]` [Gluroo — Medical Warnings](https://gluroo.com/medical-warnings/)
- `[S10]` [Gluroo — Diabetes Copilot (AI)](https://gluroo.com/ai/)
- `[S11]` [Google Play — Gluroo: Diabetes Log Tracker](https://play.google.com/store/apps/details?id=com.gluroo.app&hl=en_US)
- `[S12]` [Gluroo — Gluroo vs. mySugr](https://gluroo.com/blog/life-with-diabetes/gluroo-vs-mysugr-diabetes-apps/)
- `[S13]` [FUDiabetes forum — Sugarmate losing Dexcom G6 data access](https://forum.fudiabetes.org/t/sugarmate-losing-dexcom-g6-data-access/11962?page=4)
- `[S14]` [GitHub — aud/dexcom-share-api (nieoficjalny wrapper prywatnego API Share)](https://github.com/aud/dexcom-share-api)
- `[S15]` [Gluroo — Announcing GotCGM powered by Gluroo](https://gluroo.com/blog/releases/announcing-gotcgm-powered-by-gluroo/)
- `[S16]` [GotCGM — for Gluroo users](https://gotcgm.com/for-gluroo-users/)
- `[S17]` [Gluroo — System Status](https://gluroo.com/system-status/)
- `[S18]` [Gluroo 2.1.64 — Android Always-On CGM display, iPhone Live Activity](https://gluroo.com/blog/releases/gluroo-2-1-64-new-android-always-on-cgm-display-better-iphone-live-activity-cgm-display-more/)
- `[S19]` [Gluroo Dec 2024 feature release — charts, HUD, daily view, health-care provider support](https://gluroo.com/blog/releases/gluroo-dec-2024-feature-release-better-charts-heads-up-display-daily-view-and-health-care-provider-support/)
- `[S20]` [Diabetotech — CGM to smartwatch (opis kruchości łańcucha)](https://www.diabetotech.com/cgm-to-smartwatch)
- `[S21]` [Dexcom — FDA clears Dexcom real-time APIs for third-party apps and devices](https://www.biospace.com/fda-clears-dexcom-real-time-apis-for-third-party-apps-and-devices)
- `[S22]` [Gluroo — Gluroo Local Integration (GLI) overview and troubleshooting](https://gluroo.com/support/gluroo-local-integration-troubleshooting/)
- `[S23]` [CamDiab — CamAPS FX notifications / wymagania OS](https://camdiab.com/notifications)
- `[S24]` [Fortune — Dexcom outage leaves diabetes patients without blood sugar data (2019)](https://www.fortune.com/2019/12/02/dexcom-outage-blackout-diabetes-patients-blood-sugar-monitor)
- `[S25]` [punktum — Wearable health data integration: APIs, architecture and regulatory (MDR)](https://punktum.net/insights/wearable-health-data-integration-apis-architecture-regulatory/)
- `[S26]` [Gluroo — The diabetes app parents are raving about](https://gluroo.com/blog/glucrew/diabetes-app-parents-are-raving-about/)
- `[S27]` [Healthline — Gluroo: the simplest yet most comprehensive diabetes tool](https://www.healthline.com/healthy/gluroo-the-simplest-yet-most-comprehensive-diabetes-tool-you-may-ever-need)
- `[S28]` [The Savvy Diabetic — Savvy Apps: Gluroo with Greg Badros (2022)](https://thesavvydiabetic.com/savvy-apps-10-6-22-explore-gluroo-with-greg-badros-ceo-developer/)

---

## 8. Metryka dokumentu

- Research: **2026-09-17**, wyłącznie źródła publiczne.
- Nie weryfikowano: treści regulaminu Gluroo (ToS), faktycznej lokalizacji danych, kodu aplikacji,
  bieżących regulaminów Dexcom Share i LibreLinkUp.
- Dokument towarzyszy `handout-cgm-integracje.md`; odwołania „handout U-xx / O-xx" dotyczą tamtego pliku.
- Przegląd zalecany przy: zmianie modelu biznesowego Gluroo, komunikacie Dexcom/Abbott o aplikacjach
  trzecich, wejściu szugers na rynek.
