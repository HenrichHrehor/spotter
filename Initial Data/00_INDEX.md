# RealBench – podklady pre aplikáciu

**Názov aplikácie: RealBench** (Reality + Benchmark). Aplikácia neporovnáva ponuky len cenou, ale mierou zhody s *tvojím* benchmarkom nehnuteľnosti (plocha, cena, mesto, výklad, sklad, nájomca, vlastnícka nezávislosť, pešia frekvencia) a drží archív predaných ponúk ako trhový benchmark.
Alternatívy: *Kompas Nehnuteľností*, *ProstorScore*, *Lokus*.

Zdroj: `reality_overview_vV.html` (jednosúborová verzia, ~200 riadkov JS). Tento balík z nej robí modulárnu aplikáciu bez zmeny správania.

## Obsah balíka
| Súbor | Účel |
|---|---|
| `00_INDEX.md` | tento index, architektúra, tok dát, výstupy |
| `01_LAYOUT.md` | rozloženie obrazovky, komponenty, farby, responzivita, A4 tlač |
| `02_ENGINE.md` | dátový model, výpočty, filtrovanie, triedenie, stav (archív) |
| `03_RULES.md` | pravidlá bodovania, pravidlá zadávania dát, jazykové a formátovacie pravidlá |
| `04_CHECK.md` | kontroly pred vykreslením, testy, zlaté hodnoty (snapshot) |
| `rules.default.json` | váhy a benchmark ako konfigurácia (engine ich číta, nie sú v kóde) |
| `listings.json` | aktuálnych 15 ponúk extrahovaných z vV (vrátane textov inzerátov) |

## Architektúra (odporúčaná štruktúra)
```
realbench/
  index.html            # shell: načíta moduly, žiadna logika
  src/
    data/listings.json  # ponuky (jediný zdroj pravdy)
    rules/rules.json    # benchmark + váhy (kópia rules.default.json)
    engine/
      model.js          # normalizácia ponuky, odvodené polia (ppm, yield)
      score.js          # matchScore + rozpis
      query.js          # filtre, tab (aktívne/archív), triedenie, stránkovanie
      stats.js          # KPI, medián, zistenia (insights)
      store.js          # stav "predané" (localStorage)
    ui/
      layout.css        # tokeny, grid, tlač
      sidebar.js  kpis.js  chart.js  table.js  modal.js  tabs.js
    check/check.js      # validácia dát + runtime kontroly (pozri 04)
```
Pravidlo: `engine/*` je čistý JS bez DOM (testovateľný v Node), `ui/*` len vykresľuje výsledok enginu.

## Tok dát
`listings.json` + `rules.json` → `model.normalize()` → `score()` → `store.applySold()` → `query()` → `{base, cur}` → `stats` → `ui.render()` → `check.run()`.
`base` = ponuky po filtroch (bez tabu), `cur` = `base` zúžené na zvolený tab. Graf a medián €/m² používajú `base` (aktívne + archív), tabuľka a ostatné KPI `cur`.

## Požadované výstupy na fronte
1. **Obrazovka** – ľavý panel (filtre + benchmark), tabs Aktívne/Archív, 5 KPI kariet, bodový graf, automatické zistenia, tabuľka (10/strana), modálne okno detailu.
2. **Detail ponuky** – metadáta, rozpis zhody s benchmarkom, 3-stĺpcová analýza (Riziko / Výnos / Využitie), prepínač „Zobraziť/Zatvoriť inzerát“, „Predané“, video, kontakt.
3. **Tlač A4 (portrait)** – celkový prehľad (KPI + tabuľka bez ovládacích prvkov) a samostatná tlač jednej ponuky.
4. Úplný výstup = všetky ponuky z dát sú buď v Aktívne, alebo v Archíve (nikdy v oboch, nikdy v žiadnom) – kontroluje `04_CHECK.md`.
