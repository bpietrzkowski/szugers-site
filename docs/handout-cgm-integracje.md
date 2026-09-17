# HANDOUT: integracja danych CGM — kontekst dla Claude Code

**Wersja:** 1.0 · **Data researchu:** 2026-09-17 · **Zakres:** rynek polski
**Przeznaczenie:** dokument kontekstowy do wklejenia / wskazania agentowi przy rozbudowie
funkcjonalności importu danych glikemii w istniejącej aplikacji.

---

## 0. Jak używać tego dokumentu

Dla agenta (Claude Code) pracującego nad tą funkcjonalnością:

1. **Ustalenia w sekcji 2 są wiążące** — każde ma numer `U-xx`, poziom pewności i przypis do
   źródła `[Sxx]`. Nie projektuj wbrew nim bez jawnego zakwestionowania konkretnego `U-xx`.
2. **Poziomy pewności:** `[POTWIERDZONE]` — wiele niezależnych źródeł; `[PRAWDOPODOBNE]` —
   jedno wiarygodne źródło; `[DO WERYFIKACJI]` — sprzeczne albo brak źródła pierwotnego.
   Nie buduj krytycznej ścieżki na `[DO WERYFIKACJI]` bez potwierdzenia.
3. **Sekcja 3 (ograniczenia) ma priorytet nad wygodą implementacji.** To są głównie
   ograniczenia prawne i ToS, nie preferencje architektoniczne.
4. **Sekcja 4 to kontrakt danych** — każdy nowy adapter musi go spełnić, bez wyjątków
   „bo to źródło ma inaczej".
5. Research wykonany przez wyszukiwanie webowe, **nie przez lekturę aktów prawnych**.
   Przed komunikacją zewnętrzną („wspieramy X") potwierdź w źródle pierwotnym `[S1]`.

---

## 1. Kontekst domenowy w jednym akapicie

W Polsce refundacją objęte są **grupy wyrobów** (sensory CGM-RT, transmitery, sensory FGM),
a nie konkretne marki — więc lista wspieranych systemów jest szeroka i zmienna. Praktycznie
oznacza to ~8 rodzin systemów u pacjentów. **Żaden producent nie udostępnia publicznego,
samoobsługowego API czasu rzeczywistego.** Najbardziej otwarty jest Dexcom, ale jego
standardowe API zwraca dane z **wielogodzinnym opóźnieniem**. Wniosek architektoniczny:
aplikacja musi obsłużyć **wiele heterogenicznych źródeł o różnej świeżości**, a nie jedno API.

---

## 2. Ustalenia

### 2.1 Refundacja w Polsce

**U-01 [POTWIERDZONE]** Refundacja finansuje grupy wyrobów, nie marki. MZ potwierdza to wprost
w odpowiedzi dotyczącej CGM w cukrzycy typu 2. `[S3]`
→ *Konsekwencja:* lista wspieranych urządzeń w aplikacji będzie rosła; trzymaj ją w konfiguracji
(dane), nie w kodzie. Nie hardkoduj enumów marek w logice biznesowej.

**U-02 [POTWIERDZONE]** Grupy wyrobów istotne dla glikemii: `R.03.01–R.03.03` (sensory CGM-RT),
`R.04.01–R.04.02` (transmitery CGM-RT), `R.05.01–R.05.02` (sensory FGM). Poza tym `R.01.01`
(zestawy infuzyjne) i `R.02.01` (zbiorniki na insulinę). `[S6]`
→ *Konsekwencja:* jeśli aplikacja ma moduł „czy mi się należy refundacja", modeluj po kodach grup.

**U-03 [POTWIERDZONE]** Parametry po zmianach z 1.09.2024, niezmienione w 2025 i 2026:
transmitery CGM-RT do 3 szt./rok przy limicie 970 zł; sensory FGM przy limicie 970 zł
i kryterium **MARD ≤ 10%**. `[S2][S6]`

**U-04 [POTWIERDZONE]** Dopłata pacjenta: **20%** do 18 r.ż. (NFZ 80%), **30%** dla dorosłych
(NFZ 70%). Realizacja przez e-zlecenie u świadczeniodawcy z umową z NFZ. `[S2][S9]`

**U-05 [POTWIERDZONE]** Systemy obecne w refundacji PL (stan 09.2026):
FreeStyle Libre 2 / 2+ `[S8]`; Dexcom G6, G7, ONE, ONE+ `[S5][S7]`; Medtronic Guardian 4 oraz
Simplera / Simplera Sync `[S5]`; Eversense E3 / 365 `[S12]`; Sibionics GS1 `[S10]`;
Medtrum TouchCare S9 `[S11]`; Roche Accu-Chek SmartGuide `[S13]`.

**U-06 [DO WERYFIKACJI]** Status refundacyjny **FreeStyle Libre 3 / 3 Plus** w Polsce.
Źródła polskie opisują refundację niemal wyłącznie dla Libre 2; brak potwierdzenia dla Libre 3. `[S8]`
→ *Konsekwencja:* nie deklaruj „Libre 3 refundowane" w UI ani w marketingu do czasu weryfikacji.
Technicznie i tak obsługuj Libre 3 (patrz U-14), bo pacjenci używają go pełnopłatnie.

**U-07 [PRAWDOPODOBNE]** Dla pacjentów **26+** refundacja CGM-RT jest niezależna od metody podaży
insuliny (pen albo pompa); **poniżej 26 r.ż.** bywa powiązana z pompą insulinową. `[S5]`
→ *Konsekwencja:* jeśli robisz kalkulator uprawnień — wiek jest parametrem progowym, nie ozdobnym.

**U-08 [POTWIERDZONE]** Presja kosztowa: wydatki na refundowane wyroby dla diabetyków wzrosły
z ~46,3 mln zł (2016) do **ponad 637,7 mln zł (2025)**, z czego systemy monitorowania glikemii
to ~542,2 mln zł (wobec ~55 mln w 2021). MZ odpowiada na postulat ujednolicenia kryteriów
FGM/CGM-RT, że zmiany „muszą być racjonalne wobec kosztów". `[S2][S4]`
→ *Konsekwencja:* nie planuj funkcji, których model biznesowy zakłada szybkie rozszerzenie
refundacji (np. na cukrzycę typu 2).

**U-09 [PRAWDOPODOBNE]** Trwa projekt zmian wykazu (MZ 1904, udostępniony 15.06.2026),
dotykający m.in. ujednolicenia kryteriów FGM / CGM-RT. `[S2]`
→ *Konsekwencja:* zaplanuj przegląd tego dokumentu po wejściu zmian w życie.

### 2.2 Dostęp do danych — producenci

**U-10 [POTWIERDZONE]** **Dexcom** ma publiczny program deweloperski: REST + OAuth 2.0,
zasoby m.in. `/v3/users/self/egvs` (wartości co 5 min wraz z trendem i rate-of-change),
`/devices`, `/events` (posiłki, insulina, aktywność), `/calibrations` (tylko G6), `/dataRange`. `[S14][S15][S16]`

**U-11 [POTWIERDZONE]** Dane Dexcoma z aplikacji mobilnych (G6, G7, ONE, ONE+) są dostępne
w API z opóźnieniem **1 godziny w USA** i **3 godzin poza USA** (w tym Polska). Dane wgrane
z odbiornika przez USB — natychmiast. `[S14][S16]`
→ *Konsekwencja:* **API Dexcoma nie nadaje się do alertów ani do widoku „teraz".**
Jeśli UI pokazuje wartość z tego źródła, musi jawnie pokazywać jej wiek.

**U-12 [POTWIERDZONE]** Dexcom **nie udostępnia webhooków** w standardowym programie partnerskim —
jedyny model to polling. Model dostępu jest trójstopniowy: **Sandbox → Limited (do 5 użytkowników)
→ Full Commercial Partnership**, a weryfikacja do poziomu komercyjnego trwa **tygodnie do miesięcy**. `[S16]`
→ *Konsekwencja:* start procesu partnerskiego jest zadaniem **na teraz**, niezależnym od kodu.

**U-13 [POTWIERDZONE]** Dexcom **Partner Web APIs** (dane czasu rzeczywistego) mają clearance FDA,
ale są udostępniane wyłącznie **zaproszonym** partnerom (m.in. Garmin, Livongo/Teladoc). `[S17][S18]`
→ *Konsekwencja:* real-time z Dexcoma to ścieżka biznesowa, nie techniczna. Nie planuj na MVP.

**U-14 [POTWIERDZONE]** **Abbott** nie ma publicznego API. Dostęp do danych FreeStyle Libre idzie
przez **LibreView** i wymaga bezpośredniej relacji z Abbottem, pre-approval aplikacji i licencji;
partnerzy czytają dane jako „member of the LibreView practice". Tak działają m.in. Junction,
Thryve, Validic. `[S19][S20][S21][S22]`

**U-15 [POTWIERDZONE]** Istnieje **nieoficjalne, zreverse'owane API LibreLinkUp** (`api.libreview.io`;
`llu/auth/login`, `llu/connections`, `llu/connections/{id}/graph`). Pole `glucoseMeasurement`
aktualizuje się **co ~1 minutę**, `graphData` zwraca 15-minutowe średnie za ostatnie ~12 h.
Dokumentacja nie jest afiliowana z Abbottem. `[S23][S24]`
→ *Konsekwencja:* to **najszybsza** droga do danych Libre i jednocześnie **niedopuszczalna
w produkcie regulowanym** — ryzyko naruszenia ToS i blokad kont. Dozwolone wyłącznie jako PoC,
z jawnym zapisem długu technicznego i bez ekspozycji na użytkowników końcowych.

**U-16 [PRAWDOPODOBNE]** **Medtronic** nie ma publicznego API; CareLink jest zamknięty, a dane
z Simplera są odczytywalne zdalnie praktycznie tylko przez aplikację CareLink Connect. `[S25]`
→ *Konsekwencja:* Medtronic obsługuj wyłącznie importem plików. Nie wydawaj budżetu na próby API.

**U-17 [PRAWDOPODOBNE]** **Roche (Accu-Chek SmartGuide)** — brak publicznego API; ekosystem
danych idzie przez mySugr, a integracja z Apple Health istniała dla starszego Accu-Chek Connect. `[S26][S27]`

**U-18 [PRAWDOPODOBNE]** **Sibionics** nie udostępnia publicznego API dla deweloperów; eksport
ogranicza się do raportu AGP w PDF. Firma ma osobne rozwiązania szpitalne. `[S28]`

**U-19 [PRAWDOPODOBNE]** **Senseonics/Eversense** — brak publicznego API; dane z chmury
Senseonics trafiają do platformy **Glooko**. `[S29]`

### 2.3 Drogi alternatywne

**U-20 [POTWIERDZONE]** **Juggluco** odbiera przez BLE dane z: FreeStyle Libre 2 / 2+ / 3 / 3+,
**Sibionics GS1**, **Dexcom G7 / ONE+**, **Accu-Chek SmartGuide**, CareSens Air — i potrafi
wysyłać je do Nightscouta. **xDrip+** obsługuje G6, ONE, ONE+, G7, Stelo oraz Libre 2 (wersja EU,
bez dodatkowego transmitera). `[S30][S31][S32]`
→ *Konsekwencja:* **ścieżka Nightscout pokrywa więcej refundowanych systemów niż wszystkie
oficjalne API razem wzięte.** To uzasadnia jej priorytet w MVP mimo węższej grupy użytkowników.

**U-21 [POTWIERDZONE]** Agregatory danych wearables z compliance UE: **Thryve**
(GDPR, HIPAA, ISO 27001, silny footprint europejski), **Junction** (dawniej Vital; 300+ urządzeń,
SOC 2 Type 2, ISO 27001, obecność US i EU), **Terra** (HIPAA, GDPR, SOC 2 Type II, **oferuje
webhooki** dla danych Dexcoma), **Rook** (HIPAA, GDPR). `[S20][S21][S33][S34]`
→ *Konsekwencja:* agregator skraca time-to-market i daje gotowy DPA, ale **nie skraca opóźnienia
Dexcoma z U-11** — dostaje te same dane, tylko wygodniej podane. Kosztuje per aktywny użytkownik.

**U-22 [PRAWDOPODOBNE]** Dexcom zapisuje dane do **Apple Health z 3-godzinnym opóźnieniem**.
Aplikacja Abbotta historycznie nie zapisuje bezpośrednio do HealthKit — robią to aplikacje trzecie
(np. LibreSync, korzystający z LibreLinkUp). `[S35][S36]`
→ *Konsekwencja:* HealthKit / Health Connect to warstwa **danych retrospektywnych o szerokim
zasięgu**, nie źródło dla alertów. Wymaga aplikacji natywnej — nie zadziała dla web-only.

**U-23 [POTWIERDZONE]** Eksport plikowy działa dla każdego systemu i każdego pacjenta:
Dexcom Clarity → CSV `[S37]`; LibreView → CSV/PDF; Sibionics → PDF/AGP `[S28]`.
→ *Konsekwencja:* najtańszy sposób na 100% pokrycia sprzętowego; dobry onboarding.

### 2.4 Regulacje

**U-24 [POTWIERDZONE]** Oprogramowanie dostarczające informacji wykorzystywanych do decyzji
diagnostycznych lub terapeutycznych podlega MDR (2017/745), reguła 11 — typowo **klasa IIa**,
przy ryzyku poważnego pogorszenia stanu zdrowia **IIb**. `[S38]`
→ *Konsekwencja:* **to jest decyzja blokująca architekturę.** Wyświetlanie bieżącej glikemii,
alerty hipoglikemii i sugestie dawkowania → wyrób medyczny. Retrospektywna statystyka bez
rekomendacji → prawdopodobnie nie. Ustal docelową klasyfikację **zanim** zbudujesz warstwę alertów.

**U-25 [POTWIERDZONE]** Dane glikemiczne to dane o zdrowiu — szczególna kategoria wg **art. 9 RODO**:
wyraźna zgoda, DPIA, minimalizacja, szyfrowanie, polityka retencji, DPA z każdym procesorem
(agregator, hosting, chmura). `[S38]`

---

## 3. Twarde ograniczenia implementacyjne

Poniższe **nie podlegają optymalizacji** przy projektowaniu kodu:

| # | Ograniczenie | Źródło ustalenia |
|---|---|---|
| O-1 | Żadna wartość glikemii nie jest prezentowana bez znacznika czasu pomiaru **i** widocznego wieku danych. | U-11, U-22 |
| O-2 | Warstwa alertów (hipo/hiper) **nie może** opierać się o źródła o opóźnieniu > 10 min. Praktycznie: tylko Nightscout/BLE. | U-11, U-13, U-22 |
| O-3 | Nieoficjalne API (LibreLinkUp, patchowane aplikacje) — wyłącznie PoC, nigdy w ścieżce produkcyjnej użytkownika. | U-15 |
| O-4 | Każde źródło danych ma jawnie zapisaną w kodzie **prowenienciję i klasę świeżości** (patrz sekcja 4). | U-11, U-20 |
| O-5 | Zanim powstanie warstwa rekomendacji/alertów — decyzja o klasyfikacji MDR. | U-24 |
| O-6 | Każdy procesor danych (agregator, hosting) — podpisany DPA przed pierwszym bajtem danych produkcyjnych. | U-25 |
| O-7 | Lista wspieranych urządzeń i ich mapowanie na grupy refundacyjne żyje w konfiguracji, nie w kodzie. | U-01 |
| O-8 | Aplikacja nie instruuje użytkownika, jak obejść aplikację producenta. Przyjmuje dane z Nightscouta, który użytkownik już prowadzi. | U-15, U-20 |

---

## 4. Kontrakt danych dla adapterów

Każdy adapter źródła normalizuje do wspólnego modelu. Propozycja minimalna:

```
GlucoseReading {
  value_mgdl:      int            // kanoniczna jednostka: mg/dL (standard PL)
  measured_at:     timestamp UTC  // czas POMIARU, nie czas pobrania
  ingested_at:     timestamp UTC  // czas pobrania przez nas — do liczenia opóźnienia
  trend:           enum | null    // rising_fast..falling_fast; nie każde źródło podaje
  rate_of_change:  float | null   // mg/dL/min; Dexcom podaje, większość nie
  source:          enum           // dexcom_api | nightscout | health_connect | healthkit
                                  // | aggregator_<x> | file_import
  device_model:    string | null  // g7 | libre3 | sibionics_gs1 | ...
  freshness_class: enum           // realtime (<10 min) | delayed (<6 h) | historical
  is_calibrated:   bool | null
  raw_source_id:   string         // idempotencja — deduplikacja przy re-imporcie
}
```

Zasady:

1. **`measured_at` jest jedynym czasem, po którym wolno sortować i agregować.** `ingested_at`
   służy wyłącznie do diagnostyki opóźnień i do UI („dane sprzed 3 h").
2. **Idempotencja obowiązkowa.** Każde źródło będzie dostarczać nakładające się okna
   (polling Dexcoma, re-import pliku, backfill z Nightscouta). Klucz: `(source, raw_source_id)`
   albo `(source, device_model, measured_at)`.
3. **Jednostki konwertuj na wejściu, nie w widoku.** mmol/L → mg/dL: `× 18.0182`.
4. **`freshness_class` wyliczany przez adapter, nie zgadywany przez UI.**
5. **Priorytet przy konflikcie** (ten sam pomiar z dwóch źródeł): niższe opóźnienie wygrywa,
   przy równym — źródło oficjalne nad community.
6. Adapter **nie interpoluje i nie wygładza** danych. Braki są informacją kliniczną.

---

## 5. Specyfikacja ścieżek ingestu

| Ścieżka | Świeżość | Pokrycie systemów | Koszt | Bariera wejścia | Priorytet |
|---|---|---|---|---|---|
| **Import pliku** (CSV/PDF) | historical | wszystkie | 0 | brak | **1** |
| **Nightscout REST** | realtime (5 min) | Libre 2/3, Dexcom G6/G7/ONE+, Sibionics, Accu-Chek SmartGuide | 0 | user musi mieć Nightscouta | **2** |
| **Dexcom API v3** | delayed (3 h) | Dexcom | 0 na starcie | proces partnerski, tygodnie–miesiące | **3** |
| **Health Connect / HealthKit** | delayed (3 h) | zależne od aplikacji producenta | 0 | wymaga apki natywnej | **4** |
| **Agregator** (Thryve/Junction) | jak źródło | Libre + Dexcom + reszta | per user | umowa + DPA | **5** |
| **Abbott bezpośrednio** | ~1 min | Libre | negocjowany | relacja korporacyjna | **6** |

### 5.1 Nightscout (priorytet 2) — najkonkretniejsze do zbudowania

- Użytkownik podaje **URL własnej instancji + token/API secret**. Traktuj oba jak sekret.
- Odczyt historyczny: `GET /api/v1/entries.json?count=N`; zdarzenia: `/api/v1/treatments`.
  Dostępny też strumień przez socket dla podglądu na żywo. `[S30]` — **potwierdź szczegóły
  parametrów i nagłówków autoryzacji w bieżącej dokumentacji Nightscout przed implementacją.**
- Zakładaj instancje w różnych wersjach i o różnej dostępności — timeouty, retry z backoffem,
  degradacja do ostatnich znanych danych.

### 5.2 Dexcom API v3 (priorytet 3)

- OAuth 2.0; scope’y i zakres dostępu wg `[S15]`.
- Polling, nie webhooki (U-12). Zaplanuj harmonogram uwzględniający 3-godzinne opóźnienie —
  częstszy polling **nie da świeższych danych**, tylko zużyje limit.
- Sandbox jest self-service — zacznij od niego równolegle z resztą prac.
- ⚠️ **[DO WERYFIKACJI]** Dexcom prowadzi osobne hosty regionalne (US/EU/JP); dla użytkowników
  z Polski ruch powinien iść na host europejski. Potwierdź w dokumentacji partnerskiej.

---

## 6. Rekomendowana kolejność prac

1. **Kontrakt danych + import plikowy** — natychmiastowe pokrycie 100% urządzeń, zero zależności zewnętrznych.
2. **Adapter Nightscout** — pierwsze realne dane czasu rzeczywistego, trafia w early adopterów.
3. **Rejestracja w Dexcom sandbox** — zadanie procesowe, uruchom równolegle do 1–2 (U-12).
4. **Decyzja o klasyfikacji MDR** — blokuje warstwę alertów i rekomendacji (U-24, O-5).
5. **Health Connect / HealthKit** — gdy istnieje aplikacja natywna.
6. **Agregator** — gdy potrzebny szybki dostęp do Libre bez własnej umowy z Abbottem.
7. **Abbott / Dexcom real-time partner** — przy trakcji i uporządkowanym statusie regulacyjnym.

---

## 7. Pytania otwarte (blokujące oznaczone 🔴)

| # | Pytanie | Blokuje | Ustalenie |
|---|---|---|---|
| Q-1 🔴 | Docelowa klasyfikacja MDR aplikacji | całą warstwę alertów i rekomendacji | U-24 |
| Q-2 | Czy Libre 3 / 3+ jest refundowane w PL i w której grupie? | komunikację i moduł uprawnień | U-06 |
| Q-3 | Czy < 26 r.ż. nadal wymaga pompy insulinowej? | kalkulator uprawnień | U-07 |
| Q-4 | Czy projekt MZ 1904 (15.06.2026) zmienia limity / ujednolica FGM-CGM? | prognozy rynkowe | U-09 |
| Q-5 | Europejski host API Dexcoma i warunki dla podmiotu z UE | adapter Dexcom | U-10 |
| Q-6 | Czy aktualna aplikacja „Libre by Abbott" pisze do HealthKit / Health Connect? | wycenę ścieżki 4 | U-22 |
| Q-7 | Aktualne parametry API Nightscout (auth, paginacja) | adapter Nightscout | §5.1 |

---

## 8. Bibliografia

### Refundacja
- `[S1]` [Obwieszczenie MZ z 16.06.2025 — tekst jednolity rozporządzenia ws. wykazu wyrobów medycznych wydawanych na zlecenie (Dz.U. 2025 poz. 1038)](https://isap.sejm.gov.pl/isap.nsf/DocDetails.xsp?id=WDU20250001038) — **źródło pierwotne**
- `[S2]` [opieka.farm — MZ o ujednoliceniu refundacji sensorów CGM-RT i FGM (interpelacja 17968)](https://opieka.farm/mz-interpelacja-17968-refundacja-cgm-fgm/)
- `[S3]` [opieka.farm — MZ o refundacji CGM w cukrzycy typu 2: wykaz finansuje grupy wyrobów, nie marki](https://opieka.farm/mz-refundacja-cgm-cukrzyca-typu-2/)
- `[S4]` [Rzecznik Praw Dziecka — refundacja monitorowania glikemii, odpowiedź MZ (14.05.2026)](https://brpd.gov.pl/2026/05/14/refundacja-monitorowania-glikemii-mz-odpowiada-rpd/)
- `[S5]` [Medycyna Praktyczna — nowe systemy CGM refundowane w Polsce: Simplera i Dexcom ONE+](https://www.mp.pl/insulinoterapia/refundacja/344046,nowe-systemy-cgm-refundowane-w-polsce-simplera-i-dexcom-one)
- `[S6]` [Polskie Stowarzyszenie Diabetyków — wykaz wyrobów medycznych wydawanych na zlecenie](https://diabetyk.org.pl/wykaz-wyrobow-medycznych-wydawanych-na-zlecenie/)
- `[S7]` [Dexcom Polska — refundacja](https://dexcom.pl/refundacja/)
- `[S8]` [Abbott — refundacja FreeStyle Libre (PL)](https://www.freestyle.abbott/pl-pl/refundacja.html)
- `[S9]` [Medycyna Praktyczna — jak wystawić zlecenie na refundowany system CGM-RT](https://www.mp.pl/insulinoterapia/refundacja/315582,jak-wystawic-zlecenie-na-refundowany-system-ciaglego-monitorowania-glikemii-w-czasie-rzeczywistym)
- `[S10]` [Sibionics GS1 — refundacja (dystrybutor PL)](https://sibionicscgm.com.pl/refundacja/)
- `[S11]` [Medtrum TouchCare — refundacja (dystrybutor PL)](https://touchcarecgm.pl/refundacja/)
- `[S12]` [Diabetyk24 — refundacja NFZ, sensor Eversense E3](https://diabetyk24.pl/refundacja-nfz-sensor-eversense-e3)
- `[S13]` [Accu-Chek Polska — SmartGuide CGM](https://www.accu-chek.pl/produkty/smartguide-cgm)
- `[S39]` [PTD — stanowisko ws. refundacji systemów ciągłego monitorowania glikemii (PDF)](https://ptdiab.pl/images/aktualnosci/StanowiskoPTDiKOnsultantaiMZ.pdf)

### API producentów
- `[S14]` [Dexcom API — przegląd endpointów v3](https://developer.dexcom.com/docs/dexcomv3/endpoint-overview/)
- `[S15]` [Dexcom API — scopes & access](https://developer.dexcom.com/docs/dexcom/scopes-access/)
- `[S16]` [Momentum — Dexcom API integration: a developer's guide](https://www.themomentum.ai/blog/dexcom-api-integration-developer-guide)
- `[S17]` [Dexcom Investors — FDA clears Dexcom real-time APIs for third-party apps and devices](https://investors.dexcom.com/news/news-details/2021/FDA-Clears-Dexcom-Real-Time-APIs-for-Third-Party-Apps-and-Devices/default.aspx)
- `[S18]` [MedTech Dive — Dexcom wins FDA nod for real-time APIs](https://www.medtechdive.com/news/dexcom-wins-fda-nod-for-real-time-apis-allowing-third-party-developers-acc/603470/)
- `[S19]` [Abbott Diabetes Care — partner integrations](https://www.diabetescare.abbott/partnerships/integrations/en.html)
- `[S20]` [Junction — Abbott LibreView integration guide](https://docs.junction.com/wearables/guides/abbott-libreview)
- `[S21]` [Thryve — Abbott FreeStyle Libre integration](https://www.thryve.health/features/connections/abbott-freestyle-libre-integration)
- `[S22]` [Validic — Abbott API integration for developers](https://help.validic.com/space/VCS/4287823892/Abbott+API+Integration+for+Developers)
- `[S23]` [LibreView Unofficial API — dokumentacja (Stoplight)](https://libreview-unofficial.stoplight.io/)
- `[S24]` [GitHub — FokkeZB/libreview-unofficial](https://github.com/FokkeZB/libreview-unofficial)
- `[S25]` [xDrip — dyskusja #3267: Simplera / CareLink](https://github.com/NightscoutFoundation/xDrip/discussions/3267)
- `[S26]` [DiaTribe — Roche integrates Accu-Chek Connect with Apple Health](https://diatribe.org/roche-integrates-accu-chek-connect-glucose-meter-apple-health)
- `[S27]` [App Store — ACCU-CHEK SmartGuide app](https://apps.apple.com/si/app/accu-chek-smartguide-app/id6503455257)
- `[S28]` [Sibionics CGM — FAQ](https://www.sibionicscgm.com/pages/faq)
- `[S29]` [Senseonics — Eversense CGM launches integration with Glooko](https://www.senseonics.com/investor-relations/news-releases/2019/03-05-2019-130312706)

### Ścieżki alternatywne i agregatory
- `[S30]` [Nightscout — supported uploaders](https://nightscout.github.io/uploader/uploaders/)
- `[S31]` [Juggluco (Google Play) — lista obsługiwanych sensorów](https://play.google.com/store/apps/details?id=tk.glucodata)
- `[S32]` [Nightscout Pro — supported devices and compatibility](https://nightscout.pro/knowledge-base/supported-devices-and-compatibility/)
- `[S33]` [Terra — Dexcom integration (webhooki)](https://tryterra.co/integrations/dexcom)
- `[S34]` [ROOK — porównanie z Terra, Spike, Thryve](https://www.tryrook.io/competitors)
- `[S35]` [Dexcom — aplikacje mobilne i integracje](https://www.dexcom.com/en-us/apps)
- `[S36]` [LibreSync — live glucose dla FreeStyle Libre 3 (przez LibreLinkUp)](https://libresync.com/)
- `[S37]` [Dexcom — using Clarity (eksport CSV)](https://www.dexcom.com/en-us/faqs/clarity/using-clarity)

### Regulacje
- `[S38]` [punktum — wearable health data integration: APIs, architecture and regulatory (MDR, RODO)](https://punktum.net/insights/wearable-health-data-integration-apis-architecture-regulatory/)

---

## 9. Metryka dokumentu

- Research przeprowadzony: **2026-09-17**, metodą wyszukiwania webowego.
- **Nie weryfikowano w źródłach pierwotnych:** treści rozporządzenia MZ `[S1]`, dokumentacji
  partnerskiej Dexcoma, dokumentacji partnerskiej Abbotta, bieżącej dokumentacji Nightscout.
- **Sugerowany przegląd:** po wejściu w życie projektu MZ 1904 oraz przy każdej zmianie
  obwieszczenia refundacyjnego (kwartalnie).
