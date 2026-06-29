# DESIGNSYSTEM.md — OPK Klientportal (opk-app) — UTKAST

> **Status: utkast / inventar.** Dette dokumentet beskriver designet slik det
> **faktisk er implementert** i `opk-app/index.html` i dag, og foreslår et
> konsolidert token-sett. **Ingen kode er endret** for å lage dette — det er en
> kartlegging + forslag. Tag-konvensjon i tabellene:
> **DAGENS** = verifisert i koden nå · **FORSLAG** = ny/konsolidert verdi som ikke finnes ennå.
>
> Kilde: `opk-app/index.html` (ren HTML/CSS/JS, ingen rammeverk).
> Tall er hentet med programmatisk uttrekking og adversarisk verifisert mot fila
> (fler-agent-analyse, 2026-06-28). Linjenumre kan flytte seg ved redigering.
>
> **Implementeringsstatus (2026-06-29 — FULL RETROFIT FULLFØRT):**
> - ✅ **Steg 1** — token-fundamentet: primitiver (`--sage-600` osv.), alias re-uttrykt
>   via primitiver (identiske verdier), alle nye skalaer i `:root`. Null visuell endring.
> - ✅ **Steg 2** — tokens tatt i bruk der verdien er identisk (`--focus-ring` ×5,
>   `--shadow-inset` ×6) + umerkelig dedup (skygge-triple, `99/999px`→`--r-pill`).
> - ✅ **Steg 3 — farge-konsolidering** (commit `a6c13dd`): 37 hardkodede hex som
>   duplikerte et token rutet til `var(--token)` (eksakt + 2× Δ2), property-aware i
>   `<style>`. Gradienter/skygger/SVG-attr/JS-canvas urørt. De øvrige ~80 hex er
>   *legitimt distinkte* designfarger (gradient-stopp, mørk-flater, kategori-aksenter,
>   state) — bevart med vilje. Ingen visuell endring.
> - ✅ **Steg 4a — type + radius** (commit `f8826fd`): 194 `font-size` → 9 `--text`-tokens
>   (UI-bånd 10.5–24+30px, alle skift ≤1px; store overskrifter/clamps urørt).
>   `border-radius` enkeltverdier kollapset til `{--r-xs/sm/md/card}`, `50%`→`--r-full`.
> - ✅ **Steg 4b — spacing** (commit `3698475`): `--space`-skalaen redefinert til 8-trinns
>   4px-rutenett (4/8/12/16/20/24/28/32). 470 padding/margin/gap-verdier snappet til
>   tokens (~180 uendret, ~86 ≤1px, ~204 ±2px blandet retning). inset/positsjon, negative
>   marginer, 1–3px hårfine og >32px strukturelle urørt.
>
> Alle steg property-aware (matcher CSS-egenskap, ikke blind find-replace) og verifisert
> i preview på 3 moduler (søvn/adhd), lys + mørk, dashboard + verktøy + modal. Live.
>
> **Gjenstår (valgfritt, ikke nødvendig for målet):** mørk-komplettering av dobbeltrolle-
> tokens (mørk modus fungerer allerede; dette ville bare gjort den mer token-drevet
> internt). Egen, separat-verifisert jobb om ønskelig.

---

## 1. Overordnet bilde

Appen har **et ekte token-system**: 27 CSS custom properties i `:root` + ~602
`var()`-bruk. Merkevarefargene er korrekt tokenisert (`--sage:#5C6B4A`,
`--bg:#FAF8F5`, `--accent:#C8956D`). Lys modus + mørk modus (`body.dark`) +
Ro-modus (`body.calm`, kun animasjon/skygge av).

**Problemet er ikke fravær av system, men lekkasje rundt det:** ~100 unike hex
og ~135 unike rgba er hardkodet ved siden av tokenene, og **typografi + spacing
har ingen tokens overhodet**. Reell intensjon er ~7–8 fargefamilier spredt på
~100 nær-like verdier.

| Dimensjon | Token i dag | Faktisk bruk | Reell intensjon |
|---|---|---|---|
| Farge (hex) | 27 `:root`-tokens | ~101 unike hex / ~174 forekomster | ~7–8 familier |
| Farge (rgba) | — | ~135 unike rgba / ~194 forekomster | 4–5 RGB-tripler, varierende alfa |
| Font-size | 0 tokens | 33 px-verdier (8.5–64px) + 3 clamp | ~8–10 trinn |
| Font-weight | 400/500/600/700 lastet | 600×46, 700×21, 400×9, 500×2 | 3 roller |
| Spacing | 0 tokens | 41 unike px-verdier / ~357 forekomster | ~6–7 trinn |
| Radius | 2 tokens | 22 unike verdier / ~87 forekomster | ~5–6 trinn |
| Shadow | 2 tokens | 34 unike box-shadow / ~47 forekomster | ~4 nivåer |
| Backdrop/glass | 0 tokens | blur 6/7/14/18px | 3 nivåer |

---

## 2. Viktigste inkonsistenser (rangert)

1. **Mørk modus overstyrer kun 13 av 27 tokens (høy).** `--sage, --sage-dark,
   --sage-deep, --sage-light, --blue, --blue-soft, --teal, --teal-soft,
   --accent-soft, --warn, --bad` har ingen dark-verdi → arver lys-verdien →
   manuelle hardkodede rgba-fikser per komponent (f.eks. `#C9D2BD` ×9 som «lys
   salvietekst på mørk» der et token mangler).
2. **Mørk-salvie/dyp grønn: ~10 nær-identiske verdier (høy).** Tokenisert:
   `#3E4A33`, `#2E3726`. Ad-hoc ved siden: `#4A573B, #4A5838, #49573C, #55643F,
   #4E5C3D, #3A4130, #2B3022, #323F29` — umulige å skille på skjerm.
3. **Off-white/krem-hale: 1 token + ~14 engangsvarianter (høy).** `#FFFCF7`,
   `#FFFEFC`, `#F6F2EA`, `#F3F0E7`, `#F1F3EC`, `#F3F5EC`, `#EEF1E7`, `#EDF0E8`,
   `#EDEAE0`, `#ECE9E1`, `#E4E0D5` — praktisk talt identiske med `--bg`/`--bg-warm`.
4. **Kaotisk font-size uten skala (høy).** Småtekst-båndet er overfylt:
   `12.5px`×24, `13px`×26, `13.5px`×25, `14.5px`×16. Halv-piksler gir ingen synlig
   forskjell; samme rolle (knappetekst) er 13px ett sted, 13.5px et annet.
5. **Blå/teal program-aksenter har token, men hardkodede avvik (høy).** `--blue`
   brytes av `#7A8DA8, #7C8BA6, #5C7099, #3D516E, #B3C2D6`; `--teal` av `#3F6354,
   #4F7A66, #A7CDB2, #A9CABC`. Verst: `KB_DOTS` blander token + rå hex + en
   fallback `#4F7A66` som ikke matcher `--teal`.
6. **rgba-eksplosjon (middels).** Samme tripler i ~135 alfa-varianter; to nær-like
   skygge-tripler `rgba(60,74,51,…)` vs `rgba(62,74,51,…)` brukt om hverandre;
   inkonsistent notasjon (`.10` vs `.1`).
7. **Inter 500 lastes men brukes ~2x (middels).** Vektsystemet er reelt binært
   (600 + 700). Enten utnytt 500 til «medium», eller fjern det fra font-URL.
8. **Radius/spacing-avvik på 1px (middels/lav).** `12/13/14/16/18px` om hverandre;
   `99px`/`999px` dupliserer `--r-pill:100px`.

### Brand-avvik
- Rå merkefarger uten token: `#6A7A5A`×3 (salvie-stopp), login-gradient
  `#18200E/#232C18/#2E3A24` (der `--sage-deep` finnes), orb `#92A06F`.
- Kald grå (`#CCC`) i en ellers varm palett; `#2A241C` er en varm dublett av `--ink`.
- E-post-HTML bryter font-stacken (`Inter,Arial,sans-serif`).

---

## 3. Foreslått arkitektur

To lag: **primitiver** (rå fargestopp, f.eks. `--sage-600`) → **semantiske alias**
(`--bg`, `--ink`, `--accent`, `--card` …) som peker på primitivene. Dark/calm
overstyrer **kun semantiske alias**, aldri primitiver. Det fjerner behovet for de
manuelle per-komponent-fiksene i mørk modus.

---

## 4. Fargetokens (forslag)

### Salvie (grønn kjerne)
| Token (FORSLAG) | Hex | DAGENS opphav |
|---|---|---|
| `--sage-900` | `#2E3726` | `--sage-deep` |
| `--sage-800` | `#3E4A33` | `--sage-dark`; samler `#3A4130, #2B3022, #323F29` |
| `--sage-700` | `#4A573B` | samler `#4A5838, #49573C, #4E5C3D, #55643F` |
| `--sage-600` | `#5C6B4A` | `--sage` — **MERKEVARE, uendret** |
| `--sage-500` | `#6A7A5A` | rå `#6A7A5A`×3 → tokeniseres |
| `--sage-400` | `#92A06F` | rå orb-farge |
| `--sage-300` | `#A8B49A` | `--sage-light` |
| `--sage-200` | `#C5CEBA` | `--sage-pale`; samler `#C9D2BD×9, #C9D1BE, #CFD7C4, #D5DCC9, #D9DFCE` |
| `--sage-100` | `#EDF0E8` | `--sage-mist` |

### Krem / off-white
| Token (FORSLAG) | Hex | DAGENS opphav |
|---|---|---|
| `--cream-0` | `#FFFFFF` | `--card`; `#fff`×29 |
| `--cream-50` | `#FFFCF7` | `#FFFCF7`×2; samler `#FFFEFC` |
| `--cream-100` | `#FAF8F5` | `--bg` — **MERKEVARE off-white** |
| `--cream-200` | `#F2EDE4` | `--bg-warm`; samler `#F6F2EA, #F3F0E7, #F1F3EC, #F3F5EC, #EEF1E7, #EDF0E8` |
| `--cream-300` | `#ECE6DC` | `--border`; samler `#EDEAE0, #ECE9E1, #E4E0D5` |

### Terrakotta / leire (accent)
| Token (FORSLAG) | Hex (lys / dark) | DAGENS opphav |
|---|---|---|
| `--clay-300` | `#E0B488` | dark pill |
| `--clay-200` | `#D8A77B` | dark `--accent`; samler `#DAA97C×2, #D4A074` |
| `--clay-500` | `#C8956D` lys / `#D8A77B` dark | `--accent` — **MERKEVARE** |
| `--clay-700` | `#9C6A3C` | `#9C6A3C`×3; samler `#8F6543, #7E5733×2, #C1785A` |
| `--accent-soft` | `#F6EADC` | uendret |

### Program-aksenter (blå/teal) + status
| Token | Hex lys (DAGENS) | Hex dark (FORSLAG — mangler i dag) |
|---|---|---|
| `--blue` (Søvn) | `#5E7290` | `#B3C2D6` (allerede i bruk l.997) |
| `--blue-soft` | `#E9EDF3` | — |
| `--teal` (Autisme) | `#5F8575` | `#A9CABC` (allerede i bruk l.998) |
| `--teal-soft` | `#E4EDE8` | — |
| `--good` | `#4A7C59` | `#82B891` (DAGENS dark) |
| `--warn` | `#A8742F` | FORSLAG: gi dark-verdi |
| `--bad` | `#B05A4A` | FORSLAG: gi dark-verdi |

> FORSLAG: rut hardkodede blå-avvik (`#7A8DA8, #7C8BA6, #5C7099, #3D516E`) →
> `var(--blue)`; teal-avvik (`#3F6354, #4F7A66, #A7CDB2`) → `var(--teal)`; fjern
> fallback `#4F7A66` i `KB_DOTS`.

### Blekk (varmgrå tekst)
| Token | Hex lys (DAGENS) | Hex dark (DAGENS) |
|---|---|---|
| `--ink` | `#2A2A28` | `#ECE9E1` — samler rå `#2A241C` |
| `--ink-soft` | `#5C5C58` | `#BBB6AB` |
| `--ink-faint` | `#746E62` | `#938E82` — samler `#8A857C×3, #8A8279`; fjern kald `#CCC` |

---

## 5. Typografi (FORSLAG — ingen type-tokens finnes i dag)

Fonter (DAGENS, behold): **DM Serif Display 400** (overskrifter/tall),
**Inter 400/500/600/700** (UI). Global stack må også gjelde e-post-HTML.

### Skala
| Token (FORSLAG) | px | Erstatter (DAGENS) |
|---|---|---|
| `--text-2xs` | 11 | 10.5×9, 11×15, 11.5×9 |
| `--text-xs` | 12 | 12×16, 12.5×24 |
| `--text-sm` | 13 | 13×26, 13.5×25 |
| `--text-base` | 14 | 14×15, 14.5×16 |
| `--text-md` | 16 | 15×7, 15.5×4, 16×2 |
| `--text-lg` | 18 | 17, 18, 19 |
| `--text-xl` | 21 | 20×2, 21×3, 22×5, 23×2 |
| `--text-2xl` | 24 | 24×5, 26, 27×2, 28 |
| `--text-3xl` | 30 | 29, 30×5, 34 |
| `--text-display` | `clamp(40px,5vw,60px)` | 38, 46, 54, 64×2 + eksisterende clamps |

> FORSLAG: sett brødtekst + input til **16px** (input er 15px i dag → 16px
> hindrer iOS-zoom ved fokus).

### Vekter
| Rolle (FORSLAG) | Vekt | DAGENS |
|---|---|---|
| Brød / serif | 400 | 400×9 |
| Medium (UI/nav) | 500 | kun 2x — utnytt eller fjern fra font-URL |
| Semibold | 600 | 600×46 (overbrukt — reserver til ett nivå) |
| Strong / tall | 700 | 700×21 |

### Tracking / line-height
| Token (FORSLAG) | Verdi | Erstatter (DAGENS) |
|---|---|---|
| `--track-caps` | `.12em` | `.2/.22/.16/.14/.1/.08/.06/.05/.04em` |
| `--track-serif` | `-.01em` | `-.015/-.01/-.005em` |
| `--lh-tight` | `1.1` | `1/1.06/1.08` |
| `--lh-snug` | `1.3` | `1.15/1.2/1.3` |
| `--lh-body` | `1.55` | `1.5/1.55/1.6/1.65` |

---

## 6. Spacing (FORSLAG — ingen spacing-tokens finnes i dag)

DAGENS: 41 px-verdier / ~357 forekomster. Mest brukt: 16/12/14/8/10.

| Token (FORSLAG) | px | DAGENS nær-dubletter samlet |
|---|---|---|
| `--space-1` | 4 | 3, 4, 5 |
| `--space-2` | 8 | 6, 7, 8, 9 |
| `--space-3` | 12 | 10, 11, 12, 13 |
| `--space-4` | 16 | 14, 15, 16, 17, 18 |
| `--space-5` | 24 | 20, 22, 24 |
| `--space-6` | 32 | 26, 28, 30, 32 |

---

## 7. Radius

| Token | px | DAGENS |
|---|---|---|
| `--r-xs` (FORSLAG) | 4 | 2, 3, 5, 6 |
| `--r-sm` (FORSLAG) | 8 | 7, 8, 9, 10, 11 |
| `--r-md` (FORSLAG) | 14 | 12×7, 13×7, 14×13, 15, 16×6 |
| `--r-card` (DAGENS) | 20 | samler 18×5, 19, 23, 24×3, 26 |
| `--r-pill` (DAGENS) | 100 | samler `99px×3`, `999px` |
| `--r-full` (FORSLAG) | `50%` | 50%×18 (sirkler) |

---

## 8. Elevation / shadow

DAGENS: 2 tokens + 34 hardkodede box-shadow. FORSLAG: 4 nivåer.

| Token | Verdi | DAGENS |
|---|---|---|
| `--shadow-inset` (FORSLAG) | `inset 0 1px 2px rgba(60,74,51,.05)` | brukt ×5 hardkodet |
| `--shadow-sm` (FORSLAG) | `0 4px 12px -6px rgba(60,74,51,.35)` | samler `0 3px 8px…, 0 5px 14px…, 0 6px 16px…` |
| `--shadow` (DAGENS) | `0 10px 40px -18px rgba(60,74,51,.28)` lys / `0 12px 40px -18px rgba(0,0,0,.55)` dark | — |
| `--shadow-lift` (DAGENS) | `0 18px 50px -18px rgba(60,74,51,.35)` lys / `0 20px 52px -18px rgba(0,0,0,.65)` dark | — |

> FORSLAG: normaliser de to nær-like triplene `rgba(60,74,51,…)` /
> `rgba(62,74,51,…)` til **én** (`60,74,51`).

---

## 9. Backdrop / glass (FORSLAG — ingen glass-tokens finnes i dag)

| Token (FORSLAG) | Verdi | DAGENS opphav |
|---|---|---|
| `--blur-sm` | `7px` | `.node` blur(7), `.onbo-overlay` blur(6) |
| `--blur-md` | `14px` | `.bottom-nav` blur(14) |
| `--blur-lg` | `18px` | `.card/.login-card` blur(18) saturate(1.1) |
| `--glass-sheer` | `rgba(255,253,250,.74)` | `.card` (gjennomsiktig) |
| `--glass-solid` | `rgba(255,253,250,.93)` | `.login-card` (opak) |
| `--glass-subtle` | `rgba(255,251,245,.6)` | `.node` |
| `--glass-scrim` | `rgba(30,38,22,.45)` | `.onbo-overlay` (mørk scrim) |

---

## 10. States (hover / active / focus / disabled)

DAGENS: ingen state-tokens; fokusring finnes hardkodet (`box-shadow:0 0 0 4px
rgba(92,107,74,.10)` ×3).

| Token (FORSLAG) | Verdi | DAGENS opphav |
|---|---|---|
| `--focus-ring` | `0 0 0 4px rgba(92,107,74,.10)` | i bruk ×3 |
| `--state-hover` | `rgba(92,107,74,.08)` | innenfor salvie-alfaserien |
| `--state-active` | `rgba(92,107,74,.16)` | salvie .16-alfa i bruk |
| `--state-disabled-opacity` | `0.5` | **FORSLAG (ingen konsistent verdi i dag)** |

---

## 11. Glassmorphism — anbefalte steder

Glass finnes på 4 reelle nivåer i dag; hver flate har egne hardkodede verdier.
Anbefaling: konsolider til 3 blur-tokens + bruk **solid tekstflate bak brødtekst**.

1. **Kort-glass** (`.card,.tool-card,.stat,.stat-strip`) — primær-glasset
   (`rgba(255,253,250,.74)` + blur(18) saturate(1.1)). Legg `saturate` inn i
   token; **dropp `!important`** (det tvinger dark-override til også å bruke det).
2. **Login-kort** (`.login-card`) — `.93`-alfa, mer opakt. Formaliser to nivåer:
   `--glass-solid` (opak, bak tekst) vs `--glass-sheer` (gjennomsiktig).
3. **Bunn-nav** (`.bottom-nav`) — bruker ren hvit (`#FFF`, brand-avvik) + blur(14).
   Juster til off-white-base + felles blur-token.
4. **Verktøyhjul-noder** (`.node`) — lett glass (blur 7), legitimt eget nivå
   (`--glass-subtle`), men rut off-white-basen til kremskalaen.
5. **Onboarding-overlay** (`.onbo-overlay`) — mørk scrim-glass (blur 6 ≈ 7).
   Slå sammen til ett blur-trinn.

**Kontrast/lesbarhet:** glass legges bak *kort-flater*, ikke bak lange
tekstblokker. Brødtekst sitter alltid på en solid/halv-opak flate
(`--glass-solid` ≥ .9 alfa) — aldri rett på den levende bakgrunnen. Alle
tekstfarger (`--ink`/`--ink-soft`) ligger over de frostede flatene, ikke i selve
blur-laget.

---

## 12. Ikke-mål / merknader

- Dette er et **utkast**. Ingenting i `index.html` er endret.
- Migrering bør være additiv og verifiseres på tvers av lys/mørk/Ro-modus +
  alle moduler (søvn/adhd/autisme/utbrenthet) — samme regime som ellers i repoet.
- Anbefalt rekkefølge ved en evt. opprydding: (1) primitiv→alias-arkitektur +
  fullstendig dark-override, (2) farge-konsolidering (krem + mørk-salvie først),
  (3) type- + spacing-skala, (4) shadow/glass-tokens. Hver som egen commit.
