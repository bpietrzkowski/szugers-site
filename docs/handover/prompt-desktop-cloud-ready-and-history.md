# Prompt dla Claude Code Desktop — przygotowanie repo do pracy z chmury + sekcja „Historia" w Stat

Wklej w Claude Code Desktop w katalogu aplikacji Szugers. Realizuje decyzje D-05..D-07 z raportu przekazania.

---

Pracujemy nad aplikacją Szugers. Wykonaj poniższe etapy PO KOLEI, na nowej gałęzi
`feat/cloud-ready-and-history`. Po każdym etapie zrób osobny commit. Nie łącz etapów.

## Etap 0 — kontekst
Pobierz z repo https://github.com/bpietrzkowski/szugers-site, gałąź
`claude/cgm-refundacja-api-poland-0v36kx`, trzy pliki i skopiuj je do `docs/` w tym repo:
- docs/handout-cgm-integracje.md
- docs/ocena-ryzyk-gluroo.md
- docs/cgm-refundacja-i-integracje-pl.md
Przeczytaj handout w całości. Sekcja 3 (ograniczenia O-1..O-8) i sekcja 4 (kontrakt danych)
obowiązują przy każdej zmianie w tym repo.

Utwórz lub uzupełnij `CLAUDE.md` w katalogu głównym: krótki opis projektu, wskazanie na
docs/handout-cgm-integracje.md jako źródło ograniczeń, oraz zasady:
- każda wartość glikemii w UI ma widoczny czas pomiaru i wiek danych (O-1),
- alerty tylko ze źródeł o opóźnieniu < 10 min (O-2),
- lista wspieranych urządzeń żyje w konfiguracji, nie w kodzie (O-7),
- obliczenia statystyk (średnie, TIR, zakresy dat) muszą mieć testy jednostkowe,
- zmiany z sesji chmurowych trafiają wyłącznie na gałęzie i przez PR, nigdy na main.

## Etap 1 — zabezpieczenie repo przed publikacją na GitHub
Zanim cokolwiek trafi do zdalnego repo:
1. Przeszukaj katalog roboczy I historię gita pod kątem sekretów: keystore (*.jks, *.keystore),
   google-services.json, GoogleService-Info.plist, local.properties, .env*, klucze API,
   tokeny (Nightscout, Dexcom, LibreLinkUp), hasła w kodzie i konfiguracji.
2. Uzupełnij .gitignore o wszystkie powyższe. Jeśli któryś z tych plików JEST już w historii
   gita — ZATRZYMAJ SIĘ i pokaż mi listę, zanim cokolwiek zrobisz; decyzję o przepisaniu
   historii podejmuję ja.
3. Sprawdź dane testowe i fixtures: jeśli zawierają prawdziwe eksporty glikemii (Clarity,
   LibreView, Nightscout), zastąp je danymi syntetycznymi o tej samej strukturze.
4. Dodaj `docs/SECRETS.md` z listą, co jest sekretem, gdzie żyje (lokalnie / sekret CI /
   sekret środowiska chmurowego) i czego nigdy nie commitujemy.

## Etap 2 — pętla zwrotna dla sesji bez ekranu
1. Wykryj system budowania (Gradle / Xcode / Flutter / RN — co jest w repo) i dodaj CI
   (GitHub Actions) uruchamiany na każdy PR: kompilacja + testy jednostkowe. Bez podpisywania,
   bez publikacji — to zostaje na desktopie.
2. Dodaj SessionStart hook dla Claude Code w chmurze (.claude/settings.json + skrypt), który
   instaluje SDK i zależności potrzebne do `build` i `test`. Skrypt ma być idempotentny
   i nie może wymagać sekretów.
3. Zidentyfikuj kod liczący statystyki dla zakładki Stat (średnie, TIR, ewentualnie
   odchylenie, zakresy dat). Jeśli nie ma testów jednostkowych — napisz je TERAZ, przed
   etapem 3, na danych syntetycznych, z przypadkami brzegowymi: dzień bez danych, dzień
   z jednym odczytem, granica północy, zmiana czasu letni/zimowy, mieszane jednostki.

## Etap 3 — zakładka Stat: sekcja „Historia"
1. Wyciągnij obliczenia z plakietki „Średnie" do czystej funkcji przyjmującej zakres dat
   (lub listę odczytów dla zakresu). Plakietka „Średnie" ma używać tej funkcji — zero
   zduplikowanej logiki. Testy z etapu 2 muszą nadal przechodzić.
2. Dodaj sekcję „Historia" obok „Średnie" i „Stack": POZIOMA, przewijana lista plakietek,
   jedna plakietka = jeden dzień, od dziś wstecz do 60 dni. Każda plakietka to ten sam
   komponent, te same metryki i kolory co „Średnie", liczone dla jednego dnia, z etykietą
   daty (dziś / wczoraj / dd.mm).
3. Dni bez danych pokazuj jako plakietkę w stanie „brak danych" — nie pomijaj ich, oś
   czasu ma być ciągła.
4. Lista leniwa/zwirtualizowana — 60 plakietek nie może spowalniać otwarcia zakładki.
   Obliczenia per dzień poza wątkiem UI, z cache'em unieważnianym przy nowych odczytach.
5. Zgodnie z O-1: jeśli dane dla dnia są niekompletne (np. < 70% oczekiwanych odczytów),
   plakietka ma to sygnalizować, a nie udawać pełną średnią.

## Na koniec
Wypisz mi: (a) listę plików, które oznaczyłeś jako sekrety, (b) jak uruchomić build i testy
lokalnie, (c) co muszę kliknąć ręcznie na GitHubie (repo prywatne, ochrona main: wymagany PR
i zielone CI, brak bezpośrednich pushy), (d) czego NIE zrobiłeś i dlaczego.
Nie twórz PR-a ani nie pushuj — pokażę Ci, kiedy.
