# 01 – LAYOUT

## Rozloženie
```
┌ header (modrý pruh, nadpis + podnadpis) ─────────────────────────┐
├ aside 270px (sticky)     │ main                                  │
│  Hľadať, Lokalita        │  [Aktívne (n)] [📦 Archív (m)] hint   │
│  Vlastníctvo             │  KPI ×5                               │
│  Max. cena (range)       │  Karta: bodový graf + legenda         │
│  Max. €/m² (range)       │  Karta: automatické zistenia          │
│  Min. plocha (range)     │  Karta: tabuľka ponúk                 │
│  Max. riziko, Len s výn. │  Stránkovanie                         │
│  🎯 Benchmark (viď nižšie)│                                       │
│  Min. zhoda (range)      │                                       │
│  Zoradiť, Reset, Tlač    │                                       │
└──────────────────────────┴───────────────────────────────────────┘
```
Grid: `grid-template-columns: 270px 1fr; gap:16px; max-width:1400px; padding:0 14px`. `aside` je `position:sticky; top:10px`.

### Sekcia 🎯 Benchmark v ľavom paneli
text *Mesto*; dve číselné polia *min/max plocha*; číselné pole *max cena* (krok 10 000); checkboxy *Výklad, Skladová časť, Možnosť nájomcu, Vlastnícka nezávislosť* (default zapnuté); select *Min. pešia frekvencia* (Nezáleží / 3 / 4 / 5, default 5). Každá zmena okamžite prepočíta skóre.

## Komponenty
- **Tabs**: dve tlačidlá, aktívne má modré pozadie; počty = všetky ponuky podľa `sold` (nie podľa filtrov). V Archíve hint: „Odškrtnutím „Predané“ sa ponuka vráti medzi aktívne. Archív ostáva súčasťou benchmarku (medián, graf).“
- **KPI** (5 kariet): Ponúk (+ „z toho n predaných“), Priem. cena, Medián €/m² (aj archív), Priem. hrubý výnos, Spolu kapitál.
- **Graf**: SVG scatter, viewBox 640×250. X = plocha m², Y = €/m². Polomer bodu = 5 + výnos % (6 ak výnos neznámy). Farba = riziko (1 zelená `#28a745`, 2 `#8bc34a`, 3 `#f0ad4e`, 4–5 `#d9534f`, neznáme `#94a3b8`). Archív = priehľadný (opacity .12) s prerušovaným obrysom. Prerušovaná modrá čiara = medián €/m². Klik na bod = detail. Popis bodu = ID.
- **Tabuľka** stĺpce: # · Nehnuteľnosť (+ štítok PREDANÉ) · Vlastn. · Plocha · €/m² · Cena · Hrubý výnos (+ lišta, 100 % = 8 %) · Zhoda 🎯 · Riziko (5 bodiek) · Predané (checkbox) · Detail. Predaný riadok: opacity .55, sivé pozadie. Klik na riadok = detail.
- **Odznak zhody**: ≥75 zelený, ≥50 oranžový, inak červený.
- **Modál**: nadpis, ID + lokalita; meta mriežka (Cena, Plocha, €/m² vs medián, Hrubý výnos, Riziko, Vlastníctvo, Kontakt, Dátum); box „🎯 Zhoda s benchmarkom“ s rozpisom (✅ 🟡 ❌ ❔); 3 stĺpce analýzy; žltý panel s textom inzerátu; pätka: Predané, Zobraziť/Zatvoriť inzerát, Tlač, Video (len ak existuje), Kontaktovať (`tel:`).

## Tokeny
Primárna `#0056b3` (tmavá `#003d80`), pozadie `#f1f4f8`, text `#1e293b`, okraj `#cbd5e1`, cena `#d9534f`, výnos zelená `#28a745`, upozornenie `#f0ad4e`. Písmo `Segoe UI, Arial`. Základ 14 px.

## Responzivita
`≤900px`: jeden stĺpec, panel nie je sticky, KPI 2 stĺpce, analýza 1 stĺpec. Tabuľka sa posúva vodorovne vo vlastnom kontajneri.

## Tlač A4
`@page { size:A4 portrait; margin:15mm 12mm }`, `print-color-adjust:exact`, písmo 10 pt, tabuľka 9 pt, `tr{break-inside:avoid}`. Skryť: header, aside, tabs, stránkovanie, stĺpce Predané/Detail, všetko s triedou `noprint`. Modál sa pri tlači renderuje ako bežný blok (tlač jednej ponuky).
