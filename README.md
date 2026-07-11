# Automate × AI — dema możliwości dla branż

Interaktywne dema pokazujące, jak modele językowe i automatyzacja (budowane
szybko i tanio dzięki programowaniu agentowemu) mogą wesprzeć konkretne zawody.
Każde demo: landing z pain pointami i 5 klikalnymi symulacjami + zakładka
„Więcej scenariuszy" z katalogiem 16 kolejnych pomysłów.

Strona: **https://qbamca.github.io/automate-demo/**

## Branże

| Katalog | Branża | Zakres |
|---|---|---|
| [`geodezja/`](geodezja/) | Geodeta przy projektach liniowych | EGiB, służebności, raporty z terenu, kontrola danych brygad, operaty pod PODGiK |
| [`wycena/`](wycena/) | Rzeczoznawca majątkowy | akty notarialne, RCN, księgi wieczyste, MPZP, kontrola spójności operatu |

## Zasady wspólne

- Dema pokazują **klasy możliwości**, nie gotowe produkty.
- Wszystkie dane, nazwiska, adresy i numery są fikcyjne.
- Interakcje AI są symulowane — pojedyncze pliki HTML, zero backendu i kluczy API.
- Nie proponujemy tego, co jest już rynkowym standardem — automatyzacja żyje
  *pomiędzy* istniejącymi narzędziami.

## Dodanie nowej branży

Nowy katalog `nazwa-branzy/index.html` (samowystarczalny plik wg wzorca
istniejących dem) + karta na stronie głównej `index.html`. Deploy na GitHub
Pages jest automatyczny po pushu do `main` (`.github/workflows/pages.yml`).
