# CGM w Polsce: refundacja + możliwości integracji danych z szugers

Stan na **wrzesień 2026**. Dokument roboczy — research pod decyzję "z których systemów CGM
szugers ma zaciągać dane i jaką drogą".

> **Uwaga o wiarygodności.** Część źródeł to portale branżowe i sklepy medyczne, nie akty prawne.
> Przed komunikacją marketingową ("wspieramy X, Y, Z") każdą pozycję z sekcji 1 należy potwierdzić
> w aktualnym **rozporządzeniu MZ w sprawie wykazu wyrobów medycznych wydawanych na zlecenie**
> (tekst jednolity: obwieszczenie MZ z 16.06.2025, Dz.U. 2025 poz. 1038) oraz w projekcie zmian
> MZ 1904 z 15.06.2026. Pozycje niepewne oznaczono ⚠️.

---

## 0. TL;DR

* Refundacja w Polsce **nie działa na marki, tylko na grupy wyrobów** (sensory CGM-RT, transmitery,
  sensory FGM). Każdy producent, który spełni kryteria grupy i ma dystrybutora z umową NFZ,
  wchodzi do refundacji. Praktycznie oznacza to **8 rodzin systemów** na rynku PL (sekcja 1).
* **Żaden** z refundowanych producentów nie daje publicznego, samoobsługowego API real-time.
  Dexcom ma najbardziej dojrzały program deweloperski, ale standardowe API jest **retrospektywne
  (opóźnienie ok. 3 h poza USA)**. Real-time API Dexcoma jest tylko dla zaproszonych partnerów.
* Realny stack dla szugers to **kombinacja 4 ścieżek**, nie jedna (sekcja 3). Rekomendacja MVP:
  **Nightscout/xDrip (prawdziwy real-time, power userzy) + Health Connect/HealthKit (zasięg) +
  import CSV (fallback)**, a równolegle start procesu partnerskiego z Dexcomem i Abbottem, bo
  trwa on tygodnie–miesiące.
* Jeśli szugers ma **wyświetlać wartości glikemii lub cokolwiek na ich podstawie sugerować** —
  to niemal na pewno **wyrób medyczny wg MDR** (reguła 11, klasa IIa/IIb). To jest decyzja
  produktowa, nie techniczna, i determinuje całą architekturę (sekcja 4).

---

## 1. Co jest refundowane w Polsce

### 1.1 Jak działa refundacja

Wykaz wyrobów medycznych wydawanych na zlecenie finansuje **grupy**, nie konkretne nazwy handlowe.
Grupy istotne dla glikemii:

| Kod grupy | Czego dotyczy |
|---|---|
| R.01.01 | Zestawy infuzyjne do pomp insulinowych |
| R.02.01 | Zbiorniki na insulinę |
| R.03.01 – R.03.03 | **Sensory do CGM-RT** (monitorowanie ciągłe w czasie rzeczywistym) |
| R.04.01 – R.04.02 | **Transmitery do CGM-RT** |
| R.05.01 – R.05.02 | **Sensory FGM** (flash — skanowanie) |

Kluczowe parametry (stan po zmianach z 1.09.2024, bez istotnych zmian w 2025 i 2026):

* transmitery CGM-RT: do **3 szt. rocznie**, limit finansowania **970 zł**;
* sensory FGM: limit **970 zł**, spełniające kryterium **MARD ≤ 10%**;
* dopłata pacjenta: **20%** dla dzieci i młodzieży do 18 r.ż. (NFZ 80%), **30%** dla dorosłych (NFZ 70%);
* realizacja przez **e-zlecenie** wystawione przez lekarza, u sprzedawcy z umową z NFZ.

Kryteria kwalifikacji (uproszczone, różnią się między grupami): cukrzyca typu 1 lub 3 na
intensywnej insulinoterapii z **nieświadomością hipoglikemii**, glikogenoza, wrodzony
hiperinsulinizm, kobiety w ciąży z cukrzycą, osoby niewidome z cukrzycą. Dla pacjentów
**26+** refundacja CGM-RT jest niezależna od metody podaży insuliny (pen albo pompa);
**poniżej 26 r.ż.** wciąż bywa powiązana z pompą — do weryfikacji w aktualnym wykazie. ⚠️

Kontekst budżetowy: wydatki NFZ na wyroby dla diabetyków wzrosły z ~46,3 mln zł (2016) do
**ponad 637,7 mln zł (2025)**, z czego same systemy monitorowania glikemii to ~542,2 mln zł.
MZ konsekwentnie odpowiada, że ujednolicenie kryteriów FGM/CGM-RT jest "znane, ale musi być
racjonalne kosztowo" — czyli **nie należy zakładać szybkiego rozszerzenia refundacji**,
np. na cukrzycę typu 2.

### 1.2 Systemy obecne w polskiej refundacji

| System | Producent | Typ | Refundacja PL | Uwagi |
|---|---|---|---|---|
| FreeStyle Libre 2 / 2 Plus | Abbott | FGM | ✅ tak | Najszersza baza pacjentów w PL; dorośli od 1.01.2024 |
| FreeStyle Libre 3 / 3 Plus | Abbott | CGM-RT | ⚠️ niepotwierdzone | Źródła PL opisują refundację głównie dla Libre 2 — sprawdzić w wykazie |
| Dexcom G6 | Dexcom | CGM-RT | ✅ tak | |
| Dexcom G7 | Dexcom | CGM-RT | ✅ tak | |
| Dexcom ONE / ONE+ | Dexcom | CGM-RT | ✅ tak | ONE+ wszedł do refundacji jako jeden z nowszych |
| Medtronic Guardian 4 | Medtronic | CGM-RT | ✅ tak | Ekosystem pomp Medtronic |
| Medtronic Simplera / Simplera Sync | Medtronic | CGM-RT | ✅ tak | ~290 zł/szt., dopłata ~188,50 zł, do 5 szt./mies. (maks. 26 / 6 mies.) |
| Eversense E3 / Eversense 365 | Senseonics / Ascensia | CGM-RT implantowany | ✅ tak | Sensor ~2858 zł / 6 mies. |
| Sibionics GS1 | Sibionics | CGM-RT | ✅ tak | Od rozporządzenia MZ z 4.07.2024; MARD 8,83%; odczyt co 5 min przez BLE |
| Medtrum TouchCare S9 | Medtrum | CGM-RT | ✅ tak | Od rozporządzenia MZ z 13.10.2023 |
| Accu-Chek SmartGuide | Roche | CGM-RT | ✅ tak | W sprzedaży w PL od 1.05.2025; cena pełnopłatna ~250 zł |

**Wniosek dla produktu:** żeby "pokryć rynek refundowany" w Polsce, szugers musi obsłużyć co najmniej
**Abbott + Dexcom** (razem zdecydowana większość pacjentów), a docelowo Medtronic i tanie systemy
chińskie (Sibionics, Medtrum), które szybko zyskują udział przez niską cenę.

---

## 2. Możliwości transferu danych — per producent

| Producent | Oficjalne API | Real-time? | Opóźnienie | Jak zdobyć dostęp | Ocena dla szugers |
|---|---|---|---|---|---|
| **Dexcom** | ✅ Dexcom API v3 (REST, OAuth 2.0) | ⚠️ tylko dla zaproszonych partnerów | **1 h w USA, 3 h poza USA** | Self-service sandbox → Limited (do 5 użytkowników) → Full Commercial; weryfikacja **tygodnie–miesiące** | Najlepsza oficjalna droga, ale nie do live'a |
| **Abbott** | ⚠️ LibreView API tylko przez umowę partnerską | pośrednio tak (LibreLinkUp) | ~1 min (LLU) | Bezpośrednia relacja z Abbott, pre-approval, licencja | Największy zasięg w PL, najtrudniejszy formalnie |
| **Medtronic** | ❌ brak publicznego API | ❌ | — | Tylko partnerstwo korporacyjne; CareLink zamknięty, Simplera czyta się właściwie tylko przez CareLink Connect | Realistycznie: poza zasięgiem startupu |
| **Roche (Accu-Chek SmartGuide)** | ❌ brak publicznego API | ❌ | — | Ekosystem mySugr + integracje Apple Health w starszych produktach | Możliwe wejście przez HealthKit/mySugr |
| **Sibionics** | ❌ brak publicznego API | ❌ | — | Eksport raportu AGP do PDF; rozwiązania szpitalne osobno | Tylko import plików albo community |
| **Medtrum** | ❌ brak publicznego API | ❌ | — | — | jw. |
| **Senseonics / Ascensia (Eversense)** | ❌ brak publicznego API | ❌ | — | Dane trafiają z chmury Senseonics do **Glooko** | Ewentualnie przez Glooko |

### 2.1 Dexcom — szczegóły

* REST, OAuth 2.0, zasoby m.in. `/v3/users/self/egvs` (wartości co 5 min z trendem i rate-of-change),
  `/devices`, `/events` (posiłki, insulina, aktywność), `/calibrations` (G6), `/dataRange`.
* **Brak webhooków** w standardowym programie partnerskim — trzeba pollować.
* Dane z aplikacji mobilnych (G6, G7, ONE, ONE+) są dostępne z opóźnieniem **1 h (USA) / 3 h (reszta świata)**;
  dane wgrane z odbiornika przez USB — natychmiast. Dla Polski liczy się więc **3 h**.
* **Dexcom Partner Web APIs** (real-time) dostały clearance FDA i są udostępniane **zaproszonym**
  partnerom (m.in. Garmin, Livongo). To jest droga do live'a, ale przez relację biznesową.
* Uwaga architektoniczna: Dexcom prowadzi osobne hosty regionalne (US / EU / JP) — dla użytkowników
  z Polski trzeba kierować ruch na host europejski. ⚠️ do potwierdzenia w dokumentacji partnerskiej.

### 2.2 Abbott — szczegóły

Trzy różne rzeczy, często mylone:

1. **LibreView** — chmura raportowa dla lekarzy/klinik. Integracja partnerska polega zwykle na tym,
   że partner staje się "member of the LibreView practice" i czyta dane pacjentów, którzy wyrazili zgodę.
   Wymaga umowy z Abbottem i pre-approval aplikacji. Tak działają m.in. Junction, Thryve, Validic, CCN Health.
2. **LibreLinkUp (LLU)** — mechanizm "obserwatora": pacjent zaprasza kogoś do śledzenia odczytów.
   Istnieje **nieoficjalna, zreverse'owana dokumentacja API** (`api.libreview.io`, endpointy
   `llu/auth/login`, `llu/connections`, `llu/connections/{id}/graph`); `glucoseMeasurement`
   aktualizuje się **co ~1 minutę**, `graphData` daje 15-minutowe średnie za ~12 h.
   To jest **najszybsza droga do danych Libre**, ale: brak gwarancji stabilności, ryzyko naruszenia ToS,
   ryzyko blokady kont, i bardzo słaba pozycja regulacyjna dla wyrobu medycznego.
3. **Bezpośrednie BLE** — europejskie Libre 2 i Libre 3/3+ nadają przez Bluetooth i mogą być czytane
   bezpośrednio przez aplikację na telefonie (patrz: xDrip+, Juggluco). Technicznie najszybsze,
   prawnie najbardziej ryzykowne (obchodzenie aplikacji producenta będącej wyrobem medycznym).

---

## 3. Ścieżki integracji dla szugers — ranking

### Ścieżka A — Nightscout / xDrip+ / Juggluco  ⭐ najlepszy stosunek efektu do kosztu na start

* Użytkownik podaje szugersowi **URL swojego Nightscouta + token**; szugers czyta REST
  (`/api/v1/entries`, `/api/v1/treatments`) albo subskrybuje socket.
* **Prawdziwy real-time** (odczyt co 5 min, bez sztucznych opóźnień), zero umów z producentami,
  zero kosztu per user, dane w formacie znormalizowanym.
* Pokrycie sprzętowe jest zaskakująco szerokie: **Juggluco** odbiera przez BLE Libre 2 / 2+ / 3 / 3+,
  **Sibionics GS1**, **Dexcom G7 / ONE+**, **Accu-Chek SmartGuide**, CareSens Air;
  **xDrip+** obsługuje G6, ONE, ONE+, G7, Stelo i Libre 2 (wersja EU) — i oba wysyłają do Nightscouta.
* Ograniczenie: to rozwiązanie dla **power userów** (społeczność #WeAreNotWaiting), nie dla
  przeciętnego pacjenta. Ale to właśnie ci użytkownicy są early adopterami takiego produktu jak szugers.
* Ważne: szugers **nie namawia** do obchodzenia aplikacji producenta — po prostu przyjmuje dane
  z Nightscouta, który użytkownik już ma. To istotna różnica prawna.

### Ścieżka B — Apple HealthKit + Android Health Connect  ⭐ największy zasięg przy zerowych umowach

* Wymaga aplikacji mobilnej szugers (nie zadziała dla web-only).
* Dexcom pisze do Apple Health **z 3-godzinnym opóźnieniem** — czyli dobre do analityki i trendów,
  bezużyteczne do alertów.
* Aplikacja Abbotta historycznie **nie** pisze bezpośrednio do HealthKit; robią to aplikacje trzecie
  (np. LibreSync przez LLU). ⚠️ do sprawdzenia na aktualnej wersji "Libre by Abbott".
* Roche: integracja z Apple Health istniała dla Accu-Chek Connect; dla SmartGuide idzie raczej przez mySugr. ⚠️
* Wniosek: HealthKit/Health Connect to dobra **warstwa uniwersalna dla danych retrospektywnych**,
  nie dla live'a.

### Ścieżka C — agregatory danych (jedna umowa, wiele źródeł)

| Dostawca | Zasięg | Compliance | Uwagi |
|---|---|---|---|
| **Thryve** (DE) | Libre + reszta | GDPR, HIPAA, ISO 27001 | Najmocniejszy footprint europejski, 50 mln+ użytkowników przez ubezpieczycieli |
| **Junction** (dawniej Vital) | 300+ urządzeń, Dexcom + Libre | SOC 2 Type 2, ISO 27001, GDPR, HIPAA | Czyta dane jako member of LibreView practice; obecność US + EU |
| **Terra** | Dexcom i in. | HIPAA, GDPR, SOC 2 Type II | **Ma webhooki** — pushuje odczyty Dexcoma, gdy się pojawią |
| **Rook** | szeroki | HIPAA, GDPR | Tańsza alternatywa |

Plus: jeden kontrakt zamiast pięciu, jeden znormalizowany model danych, gotowy DPA.
Minus: koszt per aktywny użytkownik, uzależnienie od pośrednika, i **agregator nie obchodzi
fizyki opóźnień Dexcoma** — dostaje te same 3 h, tylko podane wygodniej.

### Ścieżka D — bezpośrednie API producentów

* **Dexcom**: warto zacząć proces **od razu** (sandbox jest self-service, commercial trwa miesiącami).
  Nawet z opóźnieniem 3 h to jedyne w pełni legalne, stabilne, umowne źródło danych Dexcoma.
* **Abbott/LibreView**: uruchomić rozmowę partnerską równolegle. Bez tego szugers nie obsłuży
  legalnie największej grupy pacjentów w Polsce.
* **Medtronic / Roche / Sibionics / Medtrum / Eversense**: na tym etapie odpuścić API,
  obsłużyć przez import plików i/lub Nightscouta.

### Ścieżka E — import plików (zawsze jako fallback)

* Dexcom Clarity → **eksport CSV**; LibreView → CSV/PDF; Sibionics → raport AGP w PDF.
* Nie jest sexy, działa dla **każdego** systemu i **każdego** pacjenta, nie wymaga niczyjej zgody
  poza zgodą użytkownika. Dobre na onboarding ("wrzuć swój eksport, zobacz co szugers z tym zrobi").

### Rekomendowana kolejność wdrożenia

1. **Import CSV/PDF** — działa dla wszystkich 8 rodzin systemów, tydzień pracy.
2. **Nightscout REST** — real-time, darmowy, trafia w early adopterów.
3. **Dexcom API v3** — start sandboxa dziś, bo ścieżka commercial jest długa.
4. **Health Connect / HealthKit** — gdy powstanie apka mobilna.
5. **Agregator (Thryve albo Junction)** — gdy trzeba szybko dodać Libre bez własnej umowy z Abbottem.
6. **Abbott bezpośrednio / Dexcom real-time partner** — gdy szugers ma już trakcję i status regulacyjny.

---

## 4. Ograniczenia prawne — do rozstrzygnięcia przed architekturą

* **MDR (2017/745), reguła 11.** Oprogramowanie dostarczające informacji używanych do decyzji
  diagnostycznych/terapeutycznych jest wyrobem medycznym — typowo **klasa IIa**, a przy ryzyku
  poważnego pogorszenia stanu zdrowia nawet **IIb**. Wyświetlanie bieżącej glikemii, alerty
  o hipoglikemii, sugestie dawkowania → wyrób medyczny. Czysta retrospektywna statystyka
  "wellness" bez rekomendacji → możliwe, że nie. **To jest decyzja, którą trzeba podjąć świadomie.**
* **RODO art. 9** — dane o zdrowiu to szczególna kategoria: wyraźna zgoda, DPIA, minimalizacja,
  szyfrowanie, retencja, DPA z każdym procesorem (agregator, hosting, chmura).
* **Nieoficjalne API (LLU, patchowane aplikacje)** — ryzyko naruszenia ToS i blokad; nie da się
  na tym zbudować certyfikowanego wyrobu. Nadaje się wyłącznie na prototyp/PoC, i nawet wtedy
  warto to zapisać jako świadomy dług.
* **Odpowiedzialność za dane** — jeśli szugers pokazuje wartość, którą użytkownik traktuje jako
  podstawę decyzji o insulinie, opóźnienie 3 h musi być **jawnie i widocznie** komunikowane w UI.

---

## 5. Macierz pokrycia: system refundowany × ścieżka integracji

| System (refundowany w PL) | Oficjalne API | Agregator | Health Connect / HealthKit | Nightscout / xDrip / Juggluco | Import pliku |
|---|---|---|---|---|---|
| FreeStyle Libre 2 / 2+ | przez umowę z Abbott | ✅ | ⚠️ | ✅ (xDrip, Juggluco) | ✅ |
| FreeStyle Libre 3 / 3+ | przez umowę z Abbott | ✅ | ⚠️ | ✅ (Juggluco) | ✅ |
| Dexcom G6 | ✅ v3 (3 h) | ✅ | ✅ (3 h) | ✅ (xDrip) | ✅ CSV |
| Dexcom G7 | ✅ v3 (3 h) | ✅ | ✅ (3 h) | ✅ | ✅ CSV |
| Dexcom ONE / ONE+ | ✅ v3 (3 h) | ✅ | ✅ (3 h) | ✅ | ✅ CSV |
| Medtronic Guardian 4 | ❌ | ❌ | ❌ | ⚠️ ograniczone | ✅ |
| Medtronic Simplera / Sync | ❌ | ❌ | ❌ | ❌ (tylko CareLink Connect) | ✅ |
| Eversense E3 / 365 | ❌ (Glooko) | ⚠️ przez Glooko | ❌ | ❌ | ✅ |
| Sibionics GS1 | ❌ | ❌ | ❌ | ✅ (Juggluco) | ✅ PDF/AGP |
| Medtrum TouchCare S9 | ❌ | ❌ | ❌ | ⚠️ | ✅ |
| Accu-Chek SmartGuide | ❌ | ❌ | ⚠️ przez mySugr | ✅ (Juggluco) | ✅ |

Czyta się z tego jedna rzecz: **Nightscout/Juggluco pokrywa więcej refundowanych systemów niż
wszystkie oficjalne API razem wzięte** — i dlatego powinien być w MVP, mimo że adresuje węższą
grupę użytkowników.

---

## 6. Otwarte pytania do weryfikacji

1. Czy FreeStyle Libre 3 / 3 Plus jest objęte refundacją w PL i w której grupie (FGM czy CGM-RT)? ⚠️
2. Aktualna treść kryteriów dla osób < 26 r.ż. — czy nadal wymagana pompa insulinowa? ⚠️
3. Czy projekt MZ 1904 (15.06.2026) zmienia limity albo ujednolica FGM/CGM-RT?
4. Czy Dexcom udostępnia europejski host API i na jakich warunkach dla podmiotu z UE?
5. Czy aktualna aplikacja "Libre by Abbott" zapisuje do HealthKit / Health Connect?
6. Docelowa klasyfikacja MDR szugers — to blokuje decyzję o real-time i o alertach.

---

## Źródła

Refundacja:
- [Obwieszczenie MZ z 16.06.2025 — tekst jednolity rozporządzenia ws. wykazu wyrobów medycznych wydawanych na zlecenie (Dz.U. 2025 poz. 1038)](https://isap.sejm.gov.pl/isap.nsf/DocDetails.xsp?id=WDU20250001038)
- [Rzecznik Praw Dziecka — MZ o refundacji monitorowania glikemii (14.05.2026)](https://brpd.gov.pl/2026/05/14/refundacja-monitorowania-glikemii-mz-odpowiada-rpd/)
- [opieka.farm — MZ o ujednoliceniu refundacji sensorów CGM-RT i FGM](https://opieka.farm/mz-interpelacja-17968-refundacja-cgm-fgm/)
- [opieka.farm — wykaz finansuje grupy wyrobów, nie marki](https://opieka.farm/mz-refundacja-cgm-cukrzyca-typu-2/)
- [Medycyna Praktyczna — nowe systemy CGM refundowane w Polsce: Simplera i Dexcom ONE+](https://www.mp.pl/insulinoterapia/refundacja/344046,nowe-systemy-cgm-refundowane-w-polsce-simplera-i-dexcom-one)
- [Polskie Stowarzyszenie Diabetyków — wykaz wyrobów medycznych wydawanych na zlecenie](https://diabetyk.org.pl/wykaz-wyrobow-medycznych-wydawanych-na-zlecenie/)
- [Dexcom Polska — refundacja](https://dexcom.pl/refundacja/)
- [Abbott — refundacja FreeStyle Libre](https://www.freestyle.abbott/pl-pl/refundacja.html)
- [Sibionics GS1 — refundacja](https://sibionicscgm.com.pl/refundacja/)
- [Medtrum TouchCare — refundacja](https://touchcarecgm.pl/refundacja/)
- [Accu-Chek SmartGuide CGM (Roche Polska)](https://www.accu-chek.pl/produkty/smartguide-cgm)
- [Diabetyk24 — refundacja NFZ Eversense E3](https://diabetyk24.pl/refundacja-nfz-sensor-eversense-e3)
- [PTD — stanowisko ws. refundacji systemów CGM](https://ptdiab.pl/images/aktualnosci/StanowiskoPTDiKOnsultantaiMZ.pdf)

API i integracje:
- [Dexcom API — przegląd endpointów v3](https://developer.dexcom.com/docs/dexcomv3/endpoint-overview/)
- [Dexcom API — getting started](https://developer.dexcom.com/docs/dexcom/getting-started/)
- [Dexcom API — scopes & access](https://developer.dexcom.com/docs/dexcom/scopes-access/)
- [Momentum — Dexcom API integration: a developer's guide](https://www.themomentum.ai/blog/dexcom-api-integration-developer-guide)
- [Dexcom — FDA clears real-time APIs for third-party apps](https://investors.dexcom.com/news/news-details/2021/FDA-Clears-Dexcom-Real-Time-APIs-for-Third-Party-Apps-and-Devices/default.aspx)
- [Abbott Diabetes Care — partner integrations](https://www.diabetescare.abbott/partnerships/integrations/en.html)
- [Junction — Abbott LibreView integration](https://docs.junction.com/wearables/guides/abbott-libreview)
- [Thryve — Abbott FreeStyle Libre integration](https://www.thryve.health/features/connections/abbott-freestyle-libre-integration)
- [Validic — Abbott API integration for developers](https://help.validic.com/space/VCS/4287823892/Abbott+API+Integration+for+Developers)
- [LibreView Unofficial API — dokumentacja](https://libreview-unofficial.stoplight.io/)
- [GitHub — FokkeZB/libreview-unofficial](https://github.com/FokkeZB/libreview-unofficial)
- [Terra — Dexcom integration (webhooki)](https://tryterra.co/integrations/dexcom)
- [ROOK — porównanie z Terra, Spike, Thryve](https://www.tryrook.io/competitors)
- [Nightscout — supported uploaders](https://nightscout.github.io/uploader/uploaders/)
- [Juggluco (Google Play) — obsługiwane sensory](https://play.google.com/store/apps/details?id=tk.glucodata)
- [Senseonics — integracja Eversense z Glooko](https://www.senseonics.com/investor-relations/news-releases/2019/03-05-2019-130312706)
- [xDrip — dyskusja o Simplera / CareLink](https://github.com/NightscoutFoundation/xDrip/discussions/3267)
- [punktum — wearable health data integration: APIs, architecture and MDR](https://punktum.net/insights/wearable-health-data-integration-apis-architecture-regulatory/)
