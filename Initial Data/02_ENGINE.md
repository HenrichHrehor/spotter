# 02 – ENGINE

## Dátový model ponuky (`listings.json`)
| Pole | Typ | Poznámka |
|---|---|---|
| `id` | string | `<kód mesta>-<NN>`, unikátne (BA-01, KE-12…) |
| `rank` | int | poradie odporúčania, sekvenčné |
| `title`, `city` | string | `city` obsahuje mesto aj časť (hľadá sa podľa mesta) |
| `price` | int € | > 0 |
| `sqm` | number | > 0 (môže byť desatinné, 18.77) |
| `ownership` | `"Osobné"` \| `"Nájom"` | |
| `date` | `dd.mm.yyyy` | dátum zápisu |
| `contact`, `phone` | string | `phone` bez medzier, použije sa v `tel:` |
| `video`, `url` | string? | voliteľné; tlačidlo/odkaz sa zobrazí len ak existujú |
| `rl` | 1–5 \| null | riziko; `null` = neznáme (UI „n/a“, filter ho nevyraďuje) |
| `rent` | `[min,max]` \| null | mesačné nájomné €; jediný zdroj výnosu |
| `dw`, `st`, `tn` | true \| false \| `"adj"` \| null | výklad, sklad, možnosť nájomcu; `adj` = po úprave |
| `ind` | `"building"` \| `"own"` \| `"complex"` \| null | vlastnícka nezávislosť |
| `ft` | 1–5 \| null | pešia frekvencia |
| `content` | HTML string | pôvodný text inzerátu (zobrazí sa v paneli) |
| `risk`, `yield`, `usage` | HTML string | texty 3-stĺpcovej analýzy |
| `sold` | bool | predvolená hodnota; prepisuje ju `store` |

**Odvodené (nikdy sa neukladajú):** `ppm = round(price/sqm)`; `y = (rent[0]+rent[1])/2 * 12 / price * 100` alebo `null`; `sc` (rozpis), `ms` (0–100).

## score(property, benchmark, rules) → { parts[], raw, mx, total }
Každé kritérium vráti `factor ∈ [0,1]`, body = `weight × factor`.
`total = round(Σ body / Σ váh × 100)`; pri default váhách Σ = 120.

| Kritérium | Faktor |
|---|---|
| Plocha | v rozsahu `[min,max]` = 1; pod min `max(0, 1−(min−a)/min)`; nad max `max(0, 1−(a−max)/max)` |
| Cena | ≤ max = 1; inak `max(0, 1 − 2·(p−max)/max)` |
| Mesto | benchmark mesto prázdne ⇒ 1; `city` (lowercase) obsahuje mesto ⇒ 1; inak 0 |
| Výklad / Sklad / Nájomca | kritérium nevyžadované ⇒ 1; `true` 1; `adj` 0.6; `false` 0; `null` 0.5 |
| Nezávislosť | nevyžadovaná ⇒ 1; `building` 1; `own` 0.85; `complex` 0.4; `null` 0.5 |
| Pešia frekvencia | benchmark 0 ⇒ 1; `ft` null ⇒ 0.5; inak `min(1, ft / minFt)` |

Stav riadka rozpisu: `unk` ❔ (neznámy údaj), `ok` ✅ (faktor ≥ 1), `part` 🟡 (0 < f < 1), `no` ❌ (0). Každý riadok nesie `label`, `note` (hodnota alebo „údaj chýba (pol. body)“, „po úprave (60 %)“, „nevyžadované“), `pts`, `max`.

## query(state) → { base, cur }
1. `base` = všetky ponuky, pre ktoré platí **všetko**: text (`title+contact+id`), lokalita (`city+title`), vlastníctvo, `price ≤ pmax`, `ppm ≤ smax`, `sqm ≥ amin`, `(rl==null || rl ≤ rmax)`, ak `yonly` tak `y != null`, `ms ≥ msmin`.
2. `cur` = `base` zúžené na tab: `active` ⇒ `!sold`, `archive` ⇒ `sold`.
3. Triedenie `cur`: `rank` ↑, `pa` cena ↑, `pd` cena ↓, `sa` €/m² ↑, `yd` výnos ↓ (null posledné), `ra` riziko ↑ (null posledné), `ms` zhoda ↓.
4. Zmena akéhokoľvek filtra/benchmarku/tabu ⇒ strana = 1; zmena `sold` zachová stranu (a oreže ju na posledný možný).

## stats
- KPI počítané nad `cur`, **okrem mediánu €/m² a grafu, ktoré používajú `base`** (archív tvorí benchmark).
- Medián: pri párnom počte priemer dvoch stredných hodnôt.
- Zistenia (nad `cur`): najlacnejšie a najdrahšie €/m² (odchýlka od mediánu v %), najvyšší výnos, najvyšší výnos pri riziku ≤ 2, najlepšia zhoda **len spomedzi nepredaných**; poznámka k vzorcu výnosu (hrubý, pred daňami a nákladmi).

## store
Kľúč `reality_sold` (localStorage) = `{ id: bool }`. Pri štarte prepíše `sold` z dát. Všetky operácie v `try/catch` (storage môže byť nedostupné). `setSold(id, v)`: zmení stav, uloží, prepočíta `base/cur`, tab sa nemení (ponuka z neho zmizne a objaví sa v druhom).
