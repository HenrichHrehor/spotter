# 04 – CHECK

Kontroly sa spúšťajú (1) pri štarte nad dátami, (2) po každom prepočte nad výstupom, (3) ako Node testy bez DOM.

## 1. Validácia dát (blokujúce)
- `id` unikátne a podľa vzoru `^[A-Z]{2}-\d{2}$`.
- `price > 0`, `sqm > 0`, `ownership ∈ {Osobné, Nájom}`, `date` formát `dd.mm.yyyy`.
- `rl ∈ {1..5, null}`, `ft ∈ {1..5, null}`, `ind ∈ {building, own, complex, null}`, `dw/st/tn ∈ {true,false,"adj",null}`.
- `rent` je `null` alebo `[min,max]` s `0 < min ≤ max`.
- `phone` neprázdny; `risk`, `yield`, `usage` neprázdne reťazce; `content` je voliteľný (aktuálne ho má 6 z 16 ponúk).
- Upozornenia (neblokujúce): zástupný telefón (`…0000000`), `rl == null`, `content` chýba, rozpor výnosu medzi textom a výpočtom.

## 2. Kontroly výstupu (po každom `render`)
1. Úplnosť: `počet(Aktívne) + počet(Archív) == počet(ponúk)`; žiadne ID v oboch tabsoch.
2. `0 ≤ ms ≤ 100` pre každú ponuku; Σ `max` v rozpise == Σ váh (120).
3. Súčet bodov v rozpise zodpovedá `raw`; `total == round(raw/mx*100)`.
4. Počet riadkov v tabuľke ≤ 10; počet strán == `ceil(cur/10)`; prázdny stav má text („Žiadne výsledky.“ / „Archív je prázdny.“).
5. Graf: počet bodov == `base.length`; archívne body majú prerušovaný obrys; medián v grafe == medián v KPI.
6. KPI „Ponúk“ == `cur.length`; „z toho predaných“ sa zobrazí len v archíve.
7. „Najlepšia zhoda“ nikdy nie je predaná ponuka.
8. Modál: tlačidlo Video je skryté bez `video`; text tlačidla = „Zobraziť inzerát“ po otvorení; checkbox „Predané“ zodpovedá stavu.
9. Tlač A4: header, panel, tabs, stránkovanie a stĺpce Predané/Detail nie sú v tlačovom výstupe.

## 3. Scenáre (manuálne alebo automatizované)
| # | Akcia | Očakávanie |
|---|---|---|
| S1 | Načítanie | tab Aktívne, 15 aktívnych + 1 archivovaná (DK-06) |
| S2 | Klik Archív | tabuľka = DK-06; graf aj medián nezmenené |
| S3 | V Archíve odškrtnúť DK-06 | zmizne z Archívu, objaví sa v Aktívne, počty 16/0 |
| S4 | Zaškrtnúť „Predané“ v modále | ponuka opustí aktuálny tab, počty sa aktualizujú |
| S5 | Min. zhoda 70 | zostanú len ponuky s `ms ≥ 70` |
| S6 | Max. riziko = 1 | ostanú ponuky s `rl = 1` a prípadne `rl = null`; ponuky s `rl ≥ 2` (BA-09, KE-14, KE-15, DK-06, ZV-01) zmiznú |
| S7 | Vypnúť „Vlastnícka nezávislosť“ | kritérium dá všetkým plné body, `mx` ostáva 120 |
| S8 | Reload | označenia „Predané“ sa zachovajú (localStorage) |

## 4. Zlaté hodnoty (default benchmark z `rules.default.json`)
| ID | ms | raw/120 | €/m² | hrubý výnos |
|---|---|---|---|---|
| BA-01 | 62 | 74 | 3 030 | 6.60 % |
| BA-09 | 58 | 70 | 1 533 | – |
| PD-08 | 64 | 77 | 1 975 | 6.00 % |
| DK-06 | 75 | 90 | 2 323 | – |
| ND-02 | 59 | 71 | 1 554 | – |
| RK-07 | 59 | 70 | 1 254 | – |
| PO-03 | 62 | 75 | 2 449 | – |
| NM-04 | 48 | 58 | 1 116 | – |
| LC-05 | 60 | 72 | 1 234 | – |
| KE-10 | 73 | 88 | 2 256 | – |
| ZA-11 | 73 | 88 | 2 333 | 6.20 % |
| KE-12 | 76 | 91 | 5 960 | 6.24 % |
| KE-13 | 64 | 77 | 4 449 | – |
| KE-14 | 73 | 87 | 2 941 | – |
| KE-15 | 73 | 88 | 2 833 | – |
| ZV-01 | 60 | 73 | 666 | – |

Medián €/m²: **2 289.5** (všetky, vrátane archívu) · **2 256** (len aktívne). Výsledok enginu sa musí zhodovať s tabuľkou na celé body.

## 5. Známe dátové upozornenia
- BA-01: výnos z nájomného 6.6 % vs. text 5.5–6.5 %.
- KE-14: kontakt a telefón sú zástupné; riziko 2/5 pochádza zo zjednodušeného zoznamu.
- KE-15: `rl`, `ft`, `ind` sú odhady; `st = "adj"` (sklad po úprave), výklad potvrdený používateľom.
- Sklad (`st`) je neznámy pri väčšine ponúk – skóre je preto dočasne „opatrné“.
- ZV-01: 593 m² je výmera pozemku, nie podlahová plocha; cena je v inzeráte bez DPH; `rl` je odhad; nájomné nie je uvedené.
