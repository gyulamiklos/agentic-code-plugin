# Állapot

Frissítve: 2026-11-08. Egy oldal maradjon: a `/agentic-code:save-work` írja át, a `/agentic-code:resume-work` olvassa először.

## Mi ez
Az `agentic-code` Claude Code plugin forrása (skillek, ügynökök, guard hook, `project-setup` sablonok).

## Most
- **0.6.1 beolvasztva** (#9): modell a feladathoz, képes bemenet a `spec`-ben és a `design`-ban, eldobható szkriptek.
- **0.7.0 beolvasztva** (#10): `orchestrate` skill, `agents/implementer.md` (Sonnet) és `reviewer.md` (fő modell), A/B „single vs orchestrated” az `improve-workflow`-ban. Spec: `docs/specs/2026-11-08-orchestrate-0.7.md`; friss al-ügynökös átnézés megvolt (8 találat, javítva); első A/B mérés a `docs/decisions.md` legfelső bejegyzésében.
- README átrendezve: telepítés és használat elöl, „What's new” csak az utolsó hét, rövid sorokkal (szabály a `CLAUDE.md` Kiadás részében). A repó nyilvános, `gyulamiklos/agentic-code-plugin`.
- Elutasítva vagy elhalasztva (2026-11-08): ASD-STE100 átírás, „zero-terminal” mód, külön `ephemeral-run` skill; az „LLM wiki” a 0.6.0 mérése utánig.

## Blokkol / várunk rá
- Gyula: az eszközökön `/plugin marketplace update agentic-code-plugin`.

## Következő lépés
1. Az `orchestrate` mérése egy valódi, nagyobb specen (darabonként több fájl), interaktív módban. A meglévő projektek `scripts/check.sh`-ja a régi sablonból van: a `project-setup existing` frissítse (a worktree-ből útvonallal hívva különben a vezénylő checkoutját nézi).
2. A `motion-workshop` oldalán: a `dev-recap` olvassa a „What's new” sorokat és a `notes/` döntéseit; a `motion-film` a `DESIGN.md`-t, ha nincs márkacsomag (ott készül, nem itt).
3. A 0.6.0 kipróbálása egy valódi többfős projekten; `notes.py health --save` kiindulás, négy hét múlva összevetés.
4. Régebbi nyitott tételek: a 0.5.0 küszöbeinek mérése; a TDAD és az ImpossibleBench elsődleges forrásai.

## Súrlódás
- A `scripts/check.sh` kb. 35 s, minden commitnál lefut. Ha zavaró lesz: a lassú tesztek külön címkére.
- A hook alatt a `git commit` exportálja a `GIT_*` változókat; az ideiglenes git-repós tesztek importkor törlik őket. Új ilyen teszt ugyanígy tegye.
- Fej nélküli A/B futás: a `bypassPermissions` mód a helyi auto módban tiltott; `acceptEdits` + `--allowedTools` működik. Az al-ügynök shellje minden hívás után visszaáll; a láncolt parancsok elakadnak.
