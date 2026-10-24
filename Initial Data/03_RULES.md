# 03 – RULES

## A. Pravidlá bodovania
1. Váhy a benchmark sú **len** v `rules.json`; engine ich nesmie mať natvrdo.
2. Neznámy údaj (`null`) dostane 50 % bodov a v rozpise ❔ – ani bonus, ani trest.
3. Výpočet vždy normalizuje na 0–100 podľa súčtu váh (zmena váh/kritérií nerozbije stupnicu).
4. Skóre sa počíta pre všetky ponuky (aj archív), len odporúčanie „najlepšia zhoda“ ignoruje predané.

## B. Pravidlá zadávania dát
1. **Nič nevymýšľať.** Pole, ktoré inzerát ani používateľ nepotvrdil, ostáva `null`. Výnimka: odhad rizika (`rl`) a pešej frekvencie (`ft`) – vtedy text v `risk`/`usage` obsahuje označenie „môj odhad“ / „Doplnené mnou“.
2. Údaj, ktorý nie je z inzerátu, ale od používateľa (napr. „plne presklený priestor“), sa v texte označí „Doplnené mnou“.
3. `content` je text inzerátu doslovne (len `\n` → `<br>`); nepridávať analýzu, tá patrí do `risk`/`yield`/`usage`.
4. Výnos sa vypočíta **iba** z `rent`; nájomné sa odvodí len z explicitnej sumy alebo percenta uvedeného v inzeráte (PD-08: „nad 6 %“ ⇒ použitá spodná hranica 780 €/mes.).
5. Zástupné kontakty (napr. `0940000000`) sa v dátach označia v texte a pred ostrým použitím sa nahradia.
6. Pri rozpore medzi textom a výpočtom (BA-01: text 5,5–6,5 %, výpočet 6,6 %) sa rozpor zapíše do poznámky, nezakrýva sa.
7. ID: dvojpísmenový kód mesta + poradové číslo; `rank` sa neprečísluje pri archivácii.
8. V reťazcoch v dátach nepoužívať neescapované `'` (v JSON používať `"`).

## C. Pravidlá archívu
- „Predané“ = presun do Archívu; odškrtnutie = návrat do Aktívne.
- Predvolený tab je vždy **Aktívne** (pri každom načítaní).
- Archív sa neodstraňuje z výpočtu mediánu €/m² ani z grafu.

## D. Jazyk a formát
- UI po slovensky; čísla `toLocaleString('sk-SK')`; ceny `149 000 €`; plocha `25 m²`; výnos `6.2 %`.
- Typografia: slovenské úvodzovky „ “, apostrof ’ v textoch na obrazovke.
- Jediný súbor v1 nesmie načítavať nič zo siete (tlač a offline musia fungovať).
- Print-first: každý nový prvok dostane triedu `noprint`, ak nemá byť na papieri.
