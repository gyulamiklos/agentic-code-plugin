# Modellválasztás al-ügynöknek, képes bemenet, eldobható szkriptek (0.6.1)

## Cél
Három kis skill-szöveges bővítés, Gyula hat pontos javaslatlistájából az elfogadott három (2026-11-08). A többi (ASD-STE100 átírás, „LLM wiki”, „zero-terminal” mód, külön `ephemeral-run` skill) elutasítva vagy elhalasztva, indok a `docs/decisions.md`-ben.

## Viselkedés
- [x] `working-mode` §5: al-ügynök indításakor a modell a feladathoz igazodik. Gépies, csak olvasó munka (széles keresés, listázás, log- és fájlfeldolgozás, formázás): gyors modell. Tervezés, design, spec elleni átnézés, minőségi és biztonsági átnézés: a fő modell (a `model` elhagyva). Kétség esetén a fő modell. Egy projekt vagy felhasználó saját ügynök-definíciójának `model` mezője nyer.
- [x] `spec` §1 a széles feltáráshoz, §5 az átnézéshez hivatkozza ezt.
- [x] `quality-gate` 7. lépés: a független átnézés a fő modellt kapja.
- [x] `spec` §2: UI- vagy architektúra-változásnál egyszer kér képet (mockup, vázlat, a mostani állapot képernyőképe, ábra), ha van. A spec szövegesen rögzíti, mit dönt el a kép, és az útvonalát hivatkozza: a friss munkamenet ne a kép újraolvasásán múljon.
- [x] `design` §1: az interjú kér meglévő vizuális anyagot; a terv megnevezi, mit vett át belőle.
- [x] `working-mode` §3: egyszeri adatkérdésre eldobható szkript a munkamenet scratchpadjába (vagy a repón kívüli ideiglenes mappába), az eredmény és a parancs a válaszba kerül; a repóba csak kérésre; ha a kérdés visszatér, `scripts/` alá kerül.

## Nincs benne
Külön konfigurációs fájl a modellekhez (a Claude Code nem olvasna ilyet; a mechanizmus az Agent tool `model` paramétere). Automatikus törlés. Írásra jogosult al-ügynökök (az orkesztrált megvalósítás külön spec lesz, ha kell).

## Feladatok
1. A skillek szövege (fent). 2. `plugin.json` 0.6.1, README „What's new”. 3. `scripts/check.sh` → OK.

## Ellenőrzés
`scripts/check.sh` (a `claude plugin validate` is fut benne); kézi próba `claude --plugin-dir ./plugins/agentic-code` alatt egy spec-interjún.
