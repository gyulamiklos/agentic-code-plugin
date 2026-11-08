# Orkesztrált megvalósítás: egy vezénylő, munkadarabonként egy al-ügynök (0.7.0)

## Cél
Ambiciózus, több darabból álló vagy több repót érintő munkánál a fő munkamenet (pl. Opus, medium) vezényeljen: a spec alapján nagyobb, független darabokra bont, darabonként egy olcsóbb modellű (Sonnet) al-ügynök dolgozik a saját worktree-jében, a vezénylő átnéz, összefésül, ellenőriz. Nem feladatonként egy ügynök, hanem darabonként vagy repónként. Az egy-ügynökös és az orkesztrált menet A/B módon összemérhető. Gyulának és a plugin bármely felhasználójának.

Gyula döntései (2026-11-08): új skill (`orchestrate`); a skill csak javasol, a felhasználó dönt; az A/B az `improve-workflow` mérésének bővítése; két ügynök-definíció a pluginban.

## Viselkedés
- [x] **Mikor éri meg** (a skill elején, szabályként): van spec, és legalább két **munkadarab**, amely (a) önállóan commitolható, saját teszttel, (b) más fájlokat érint, mint a többi, (c) a többihez csak a specben előre rögzített felületen kapcsolódik; **vagy** a munka több repót érint (repónként egy darab). Nem éri meg: egy darab; egymásba fonódó darabok; közös fájlok; spec nélküli, felderítő munka; egy-két fájlos változás. Egy darab nem egy spec-feladat, hanem több feladat egy összefüggő csoportja.
- [x] **Javaslat, nem indítás:** a skill egy táblázatot mutat (darab, repó, fájlok, spec-feladatok, modell), mellette a költség-figyelmeztetést (több token, mint egyedül; a nyereség a falióra-idő és a tiszta fő kontextus), és csak a felhasználó igenje után indít. A `spec` 4. lépése és a `git-workflow` §4 felajánlja, ha a feltételek teljesülnek; magától soha nem indul.
- [x] **Felület előbb:** a darabok közötti közös felület (függvény-aláírás, típus, API-szerződés, fájlformátum) a specben rögzített, vagy a vezénylő maga commitolja az alapágon, mielőtt bármelyik darab indul. Ami ezen kívül két darabot is érint, az nem indul párhuzamosan.
- [x] **Darab-brief** (önálló, a beszélgetés nélkül érthető): a `docs/agent-brief.md` blokkja, ha van; a spec útvonala és a darab feladatai; a fájlok, amelyekhez nyúlhat, és amelyekhez nem; a rögzített felület; az elfogadási kritériumok; a check parancs; mit adjon vissza.
- [x] **Futtatás:** darabonként egy `agentic-code:implementer` (Sonnet). A worktree-t mindig a vezénylő hozza létre (az átnézés után módosítva: az `isolation: "worktree"` ágát a brief nem tudja megnevezni, és a kiinduló pontja sem a munkaág). Ugyanabban a repóban a munkaágról, másik repóban az `origin/<default>`-ról; ott a darab ága a repó saját PR-ága, nem olvad be. Soha nem a felhasználó munkapéldánya. Egyszerre legfeljebb 3 darab. A munkás a `quality-gate` szerint dolgozik (teszt előbb, check zöld), a saját ágán helyben commitol; push, PR, merge, rebase, más ág és a darabon kívüli fájl tilos.
- [x] **Munkás jelentése:** ág, commitok, check eredménye (a kimenet utolsó sorai), eltérés a brieftől indokkal, amit nem ellenőrzött.
- [x] **Összefésülés:** a vezénylő darabonként `git diff <alap>...<ág>`-on nézi, hogy a darab a saját fájljain belül maradt és a kritériumokat teljesíti; a darabon kívüli változást visszaküldi. Egyenként olvasztja a munkaágba, minden merge után check (`git-workflow` §7). Ütközésnél mindkét oldalt elolvassa; ha nem egyértelmű, visszaküldi vagy kérdez.
- [x] **Átnézés:** a végén egy friss `agentic-code:reviewer` (fő modell, csak olvas) a teljes diffet a spechez méri (`spec` §5, `quality-gate` 7.). Javítási kör darabonként legfeljebb kettő; utána a vezénylő maga javít vagy a felhasználót kérdezi (`quality-gate` megállási szabály).
- [x] **Hibák:** egy munkás kétszer elbukik → nem indul harmadszor; a vezénylő átveszi vagy kérdez. Egy munkás nem válaszol vagy félbehagyja → a worktree megmarad, a vezénylő jelenti.
- [x] **Zárás:** a spec feladatai kipipálva; jelentés táblázatban (darab, modell, tokenek és eszközhívások a befejezési értesítésből, check, átnézési találatok); a beolvasztott worktree-k és ágak törlése; a be nem olvasztotté csak a felhasználó igenjével. Push és PR a szokott módon, külön engedéllyel.
- [x] **Ügynökök:** `plugins/agentic-code/agents/implementer.md` (`model: sonnet`; ír a saját worktree-jében; a fenti tiltások) és `reviewer.md` (`model: inherit`; `tools: Read, Grep, Glob, Bash`; csak olvas; csak a helyességet vagy a követelményeket érintő hiányokat jelenti, `file:line` bizonyítékkal). A projekt vagy a felhasználó azonos célú saját ügynöke nyer; a skill ezt kimondja.
- [x] **`working-mode` §5:** a „csak olvasó al-ügynök” szabály kivétele az `orchestrate` munkása, a saját worktree-jében.
- [x] **A/B (`improve-workflow`, „Measure a change”):** új változat „single vs orchestrated”. Ugyanaz a spec és kiinduló commit, két homokozó-klón; mindkét oldal egy-egy fej nélküli `claude -p` futás a klónban (`--plugin-dir`, azonos fő modell és erőfeszítés, `--output-format json`), mert egy al-ügynök nem indíthat további al-ügynököt. Mért: összes token és költség a JSON-ból, falióra-idő, körök száma, a check eredménye, a spec kritériumai teljesültek-e, az átnéző találatai. A meglévő szabályok érvényesek: előbb a költség és a felhasználó igenje, 10% alatti különbségnél n=2, százalék csak a saját táblázatból, pontosság-romlásnál a megtakarítás nem számít.
- [x] README: új skill a táblázatban, „What's new” 0.7.0, rövid bekezdés „Orchestrated work” (mikor, költség, A/B). `plugin.json`: 0.7.0, a leírásban `orchestrate`.

## Nincs benne
Feladatonként egy ügynök; automatikus indítás; push, PR vagy merge az al-ügynökök részéről; al-ügynök által indított al-ügynök; ütemezett vagy felhős futtatás; szkript az A/B-hez (kérésre később); a `project-setup` változtatása (az ügynökök a pluginnal jönnek, nem kell őket a projektbe másolni).

## Terv
- Új: `plugins/agentic-code/skills/orchestrate/SKILL.md` (angol, a többi skill felépítését követi: frontmatter `description` a kiváltó helyzetekkel, `argument-hint: "<spec path>"`, számozott szakaszok, „Never” lista).
- Új: `plugins/agentic-code/agents/implementer.md`, `reviewer.md` (a formátum: `name`, `description`, `model`, `tools`; minta egy telepített hivatalos plugin `agents/` mappája).
- Módosul: `spec` §4 (felajánlás), `git-workflow` §4 (több repó → `orchestrate`), `working-mode` §5 (kivétel), `improve-workflow` (új változat), `quality-gate` 7. (a reviewer ügynök neve), README, `plugin.json`.
- Kiadás: 0.7.0, új ág a `feat/model-tiers-and-intake` után (az 0.6.1 előbb olvadjon be, vagy erre az ágra épül, ha Gyula úgy dönt; nem egymásra épülő PR).

## Feladatok
1. Az ügynökök (`agents/`), `claude plugin validate` zöld. 2. Az `orchestrate` skill. 3. A hivatkozó skillek (`spec`, `git-workflow`, `working-mode`, `quality-gate`). 4. Az `improve-workflow` A/B-változata. Ellenőrizve (2026-11-08, egy haiku al-ügynökös próbával): a `total_cost_usd` és a `modelUsage` tartalmazza az al-ügynököt, a felső szintű `usage` csak a fő szálat; a skill ezért az előbbieket méri. 5. README, `plugin.json`, `docs/decisions.md`. 6. `scripts/check.sh` → OK.

## Ellenőrzés
- `scripts/check.sh` → OK (benne `claude plugin validate`).
- Kézi próba `claude --plugin-dir ./plugins/agentic-code` alatt egy mintaprojekten: egy kétdarabos spec (két független modul) → az `orchestrate` javasol, igen után két implementer fut worktree-ben, a vezénylő összefésül, a reviewer jelent, a check zöld, a worktree-k eltűnnek.
- Egy A/B mérés ugyanezen a specen, a táblázat a `docs/decisions.md` bejegyzésébe (a küszöbök és a „megéri” szabály ennek alapján finomítandó).

## Átnézés (2026-11-08)
Friss, csak olvasó al-ügynök, 8 találat, mind javítva: a helyi merge és ágtörlés kivétele a `working-mode` §7-ben és a `git-workflow` §6-ban (a javaslatra adott igen fedi); a worktree-t a vezénylő hozza létre; a munkaág definíciója és a több repós ág kezelése; `--effort` az A/B parancsban; a válasz nélküli munkás esete; az A/B bekezdés elsőbbsége; `git -C` és abszolút útvonal a munkásnak.
