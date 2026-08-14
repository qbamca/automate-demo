# CLAUDE.md

Wskazówki dla Claude Code (i innych asystentów AI) pracujących w tym repozytorium.

## Czym jest ten projekt

Statyczna strona na GitHub Pages z **interaktywnymi demami możliwości** LLM +
automatyzacji dla konkretnych zawodów. Cel jest sprzedażowo-edukacyjny: pokazać
praktykowi z danej branży, że rzeczy, które zjadają mu tydzień, da się
zautomatyzować — nie sprzedać gotowy produkt.

Produkcja: **https://qbamca.github.io/automate-demo/**

Kluczowe konsekwencje techniczne:

- **Zero backendu, zero build stepu, zero zależności.** Każda strona to jeden
  samowystarczalny plik `index.html` z wbudowanym `<style>` i `<script>`.
  Nie ma `package.json`, npm, bundlera, frameworka ani CDN-ów.
- **Nie ma prawdziwych wywołań LLM.** Wszystkie „odpowiedzi AI" to
  zahardkodowane tablice danych odtwarzane w czasie przez `setTimeout`.
  Nigdy nie dodawaj kluczy API ani fetchy do zewnętrznych usług.
- **Wszystkie dane są fikcyjne** — nazwiska, adresy, numery działek, numery KW,
  ceny transakcyjne, sygnatury. Utrzymuj tę zasadę przy każdej edycji i
  pilnuj, żeby dane wyglądały wiarygodnie dla praktyka (poprawne formaty
  numerów KW, TERYT, jednostek ewidencyjnych), ale nie odpowiadały realnym
  osobom ani nieruchomościom.

## Struktura repozytorium

```
index.html               # hub — landing z kartami do dem branżowych
geodezja/index.html      # demo: geodeta przy projektach liniowych (400 kV)
wycena/index.html        # demo: rzeczoznawca majątkowy
.github/workflows/pages.yml  # deploy main -> gh-pages
.nojekyll                # wyłącza przetwarzanie Jekyll na Pages
README.md                # opis projektu po polsku
```

Każdy plik dema ma ~1150 linii i identyczny szkielet. Hub (`index.html`,
~136 linii) jest prostszy: tylko CSS + karty, bez JS.

## Anatomia pliku dema

Kolejność sekcji w `geodezja/index.html` i `wycena/index.html` jest taka sama —
trzymaj się jej przy dodawaniu nowej branży:

1. `<style>` — cały CSS inline. Tokeny w `:root`, potem komponenty.
2. `<nav class="topnav">` — brand + dwie zakładki (`#start`, `#wiecej`).
3. `<div id="view-start">`
   - `<header class="hero">` — nagłówek + `<canvas id="heroCanvas">`
   - `<section id="problemy">` — 6 pain pointów w kartach
   - `<section id="scenariusze">` — 5 `<article class="demo" id="demoN">`
   - `<section id="dlaczego">` — 3 karty „dlaczego szybko i tanio"
   - `<section id="rozmowa">` — CTA na rozmowę
4. `<div id="view-wiecej" hidden>` — katalog **16 scenariuszy** w 4 grupach
   `.cat-group` (A–D), każdy scenariusz jako `<details class="acc">` z
   akapitami oznaczonymi `<b class="k k-problem">` / `<b class="k k-mozliwosc">`.
5. `<script>` — jeden IIFE ze `"use strict"`.

### Konwencje JS

Cały JS mieści się w jednym IIFE na dole pliku. Wzorce, które musisz zachować:

```js
var RM = window.matchMedia("(prefers-reduced-motion: reduce)").matches;
function sleep(ms) { return new Promise(function (r) { setTimeout(r, RM ? 0 : ms); }); }
function $(id) { return document.getElementById(id); }
```

- **`sleep()` + `async function` w handlerach `click`** to podstawowy sposób
  odgrywania „myślenia AI". Każde opóźnienie MUSI iść przez `sleep()`, żeby
  `prefers-reduced-motion` natychmiast pokazywało wynik końcowy.
- **Składnia ES5 dla deklaracji** (`var`, `function (…) {}`) z wyjątkiem
  `async/await`. Nie przepisuj tego na `const`/arrow functions — plik jest
  celowo pisany jednolicie.
- **Identyfikatory `dN…`**: elementy N-tego dema mają prefiks `d1`, `d2`, …
  (`d1run`, `d1reset`, `d1out`, `d1bar`, `d1sum`, `d1-<pole>` dla komórek
  tabeli wynikowej). Trzymaj się tej numeracji.
- **Każde demo ma przycisk `dNrun` i ukryty `dNreset`** („Od nowa"), a reset
  przywraca stan początkowy w całości (czyści `textContent`, zdejmuje klasy
  `hl`, odblokowuje przyciski, ustawia z powrotem `hidden`).
- **Guard `dNbusy`** blokuje ponowne kliknięcie w trakcie animacji.
- **Pojawianie się elementów**: dodaj element z klasą bazową, potem w podwójnym
  `requestAnimationFrame` dorzuć `.show` — inaczej przejście CSS nie zadziała.
- **`aria-live="polite"`** na kontenerach, które zapełniają się w czasie
  (`.chat`, `.findings`, `.checklist`); paski postępu `.scanbar` mają
  `aria-hidden="true"`.
- `wycena/index.html` ma pomocnik `findingsRunner(cfg)` uogólniający dema typu
  „skanuj i wypisz znaleziska" (dema 2 i 4). W `geodezja/` te same dema są
  rozpisane osobno. Przy nowych demach preferuj `findingsRunner`.

### Routing zakładek

Bez routera — funkcja `route()` czyta `location.hash`, przełącza `hidden` na
`#view-start` / `#view-wiecej`, ustawia klasę `.active` na `.tab` i przewija do
kotwicy. Wołana raz na starcie i przy `hashchange`. Kotwice wewnątrz aktywnego
widoku zostawia przeglądarce (`if (!changed) return;`).

### Hero canvas

Każde demo ma własny rysunek na `<canvas id="heroCanvas">` rysowany ręcznie w
`drawHero()`: geodezja — profil linii energetycznej ze słupami i pasem działek,
wycena — wykres rozrzutu cen z linią trendu. Wymagania:

- skalowanie przez `devicePixelRatio`,
- kolory pobierane z tokenów CSS przez `cssVar()` — nigdy na sztywno,
- przerysowanie przy `resize`, przy zmianie `prefers-color-scheme` oraz przez
  `MutationObserver` na atrybucie `data-theme` na `<html>`,
- `aria-label` + `role="img"` na elemencie `<canvas>` (canvas jest wyłącznie
  dekoracyjno-ilustracyjny; treść nigdy nie żyje tylko w nim).

### Motyw i kolory

Trzy warstwy definicji tokenów w `:root`, w tej kolejności — nie zmieniaj jej:

1. `:root { … }` — pełna paleta jasna (wartość domyślna),
2. `@media (prefers-color-scheme: dark) { :root { … } }` — paleta ciemna,
3. `:root[data-theme="light"]` i `:root[data-theme="dark"]` — jawne nadpisania.

W repo **nie ma przełącznika motywu w JS**; `data-theme` obsługiwane jest po to,
by strona zachowywała się poprawnie, gdy atrybut ustawi host. Każdy nowy kolor
definiuj jako token w warstwie 1 i nadpisuj we wszystkich pozostałych.

Palety branżowe (akcent jest jedynym istotnym różnicującym elementem):

| Demo | `--accent` jasny | `--accent` ciemny |
|---|---|---|
| geodezja | `#D9480F` (rdzawy) | `#F0692C` |
| wycena | `#176E4B` (zielony) | `#46B97C` |

Hub (`index.html`) trzyma oba jako `--geo` / `--wyc` i podaje je kartom przez
`--card-accent` / `--card-soft`. Dodając branżę, dodaj tam nową parę tokenów i
klasę `.card-<skrót>`.

Poza akcentem CSS obu dem jest praktycznie identyczny (różnice to tytuł i
paleta). Zmianę w komponencie wspólnym — `.demo`, `.btn`, `.chat`, `.findings`,
`.checklist`, `.acc`, `.cta`, `.scanbar` — **nanoś w obu plikach**, inaczej
dema się rozjadą wizualnie.

## Konwencje treści

Ta część jest równie ważna jak kod — dema są materiałem sprzedażowym.

- **Cały tekst po polsku**, z polską typografią: cudzysłowy „…", półpauza —
  w roli myślnika, `·` jako separator w liniach monospace.
- **Ton**: bezpośredni, konkretny, do praktyka. Bez marketingowego bełkotu,
  bez „rewolucji" i „synergii". Liczby zamiast przymiotników („1 240 punktów
  w 9 s", „tydzień vs godzina").
- **Nie proponuj tego, co jest rynkowym standardem.** Wartość leży *pomiędzy*
  istniejącymi narzędziami — w przeklejaniu, kontroli spójności, raportowaniu.
  Demo, które proponuje „lepszy tachimetr" albo „CRM", jest do wyrzucenia.
- **Struktura scenariusza w katalogu**: dokładnie dwa akapity — `Problem`
  (co boli dzisiaj, w realiach zawodu) i `Możliwość` (co robi automatyzacja,
  konkretnie). Bez trzeciego akapitu, bez cen, bez obietnic wdrożenia.
- **Ilości są stałe**: 6 pain pointów, 5 klikalnych scenariuszy, 16 pozycji
  katalogu w 4 grupach. Trzymaj proporcje przy nowej branży.
- Stopka zawsze przypomina, że dane są fikcyjne.

## Dodanie nowej branży

1. Utwórz `nazwa-branzy/index.html` — najprościej kopiując istniejące demo
   (`wycena/index.html` ma nowszy, bardziej uogólniony JS) i podmieniając:
   `<title>`, paletę akcentową, brand w `.topnav`, hero + `drawHero()`,
   treść wszystkich sekcji i dane w skryptach dem.
2. Dodaj kartę `<a class="demo-card card-<skrót>">` w `index.html` oraz parę
   tokenów `--<skrót>` / `--<skrót>-soft` w **wszystkich czterech** blokach
   `:root` huba.
3. Dopisz wiersz do tabeli branż w `README.md`.
4. Sprawdź lokalnie (patrz niżej) w jasnym i ciemnym motywie, na wąskim
   viewporcie oraz z włączonym `prefers-reduced-motion`.

## Uruchomienie lokalne

Bez build stepu — wystarczy serwer statyczny z katalogu repo:

```bash
python3 -m http.server 8000   # potem http://localhost:8000/
```

Otwarcie pliku przez `file://` też działa, ale linki w hubie (`geodezja/`)
zachowują się wtedy inaczej — preferuj serwer.

Nie ma testów, lintera ani formattera. Weryfikacja jest ręczna: przeklikaj
wszystkie 5 dem wraz z „Od nowa", przełącz obie zakładki, sprawdź oba motywy i
zachowanie przy zawężonym oknie.

## Deploy

`.github/workflows/pages.yml` przy każdym pushu do `main` (lub ręcznie przez
`workflow_dispatch`) usuwa `.github`, dodaje `.nojekyll` i force-pushuje
zawartość drzewa na gałąź `gh-pages`, z której serwuje GitHub Pages. Nie ma
kroku budowania — to, co wylądowało w `main`, jest tym, co widzi użytkownik.
Nie commituj do `gh-pages` ręcznie; gałąź jest nadpisywana przy każdym deployu.

## Praca z gitem

- Rozwijaj na gałęzi wskazanej w zadaniu, nigdy bezpośrednio na `main`.
- Komunikaty commitów po polsku, w konwencji `typ: opis`
  (`feat:`, `fix:` — zgodnie z historią repo).
- Pull requesta twórz tylko wtedy, gdy użytkownik wprost o niego poprosi.
