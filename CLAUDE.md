# opk-app — Oslo Psykologkontor klientportal / ADHD-app

Dette repoet er **appen som selvstendig produkt** (mock-preview), hostet på Netlify med continuous deploy. Norsk i UI og kommentarer.

## Hva dette er
- Én selvstendig HTML/CSS/JS-app. **`index.html` er kanonisk — rediger den.** Ingen byggesteg, ingen rammeverk.
- Voks fra den funksjonelle v2-klientportalen (GAS-login, søvndagbok, verktøy) + high-end-design (verktøyhjul, levende bakgrunn, frostede kort, mørk modus, Ro-modus, enso-logo).
- Kjører i **mock/preview-modus** på alle ikke-prod-verter (`PREVIEW = hostname !== 'oslopsykologkontor.no'`). Da kontaktes ikke ekte backend.

## Deploy
- Push til `main` → **Netlify auto-deployer** (~30 s): https://opk-app-preview-k7q2m9x4.netlify.app
- Site `opk-app-preview-k7q2m9x4`, publish `.`, build command tom, `noindex`.
- I Claude Code: rediger `index.html`, så `git add -A && git commit -m "..." && git push`. (Cowork kan ikke pushe selv — sandkasse-nettverk.)

## Mock-brukere (kun i preview)
- **OPK1** → kun søvn-modul · **OPK2** → adhd-modul. Oppslagstabell `DEMO_CODES` (gamle alias SØVN/ADHD/K001/K002 beholdt).
- Legg til flere i `DEMO_USERS` + `DEMO_CODES`.

## Designsystem
- Salvie `#5C6B4A`, varm bakgrunn `#FAF8F5`, leire/clay `#C8956D`. DM Serif Display + Inter. Enso pensel-ring-logo.
- Signaturer: radialt **verktøyhjul** i `renderToolLauncher` (koblet til `openTool`, med «Vis alle som liste»-fallback), levende bakgrunn (`.app-bg` blobs + røyk + vignett), frostede kort, søvnvindu som klokke, «Flere moduler»/«Ikke bygget» (`FUTURE_MODULES`).

## Harde regler
- **Additive endringer.** Ikke bryt: GAS-login (`apiCall`), `PREVIEW`/`DEMO_USERS`/`DEMO_CODES`, `loadTool/persistTool/clearToolStore`, `openTool` + alle `render*Tool`, `renderSleepClock`, `renderDashboard` + `FUTURE_MODULES`, mørk modus (`toggleDark`), Ro-modus (`body.calm` → `animation:none`), logo-fallback-IIFE.
- **Kun én `index.html`.** Aldri versjonskopier.
- Service worker er **av i preview** med vilje (unngå gammel cache) — ikke skru den på der.
- Verifiser at JS parser etter endringer (hent ut `<script>`, `new Function(...)`), og render-test login → OPK1/OPK2 → hjul.

## Full prosjektkontekst (i foreldermappa `../prosjekt-kontekst/`)
Hvis Claude Code er åpnet på `Nettside-OPK` (anbefalt), les:
- `KONSOLIDERING-OG-STRUKTUR.md` — struktur (Cowork + Code), to spor, beslutninger, fallgruver, neste steg.
- `APP-PRODUKT.md`, `ADHD-app_fra_verktoysamling_til_app.md` — app-produktcharter.
- `BYGGEPLAN.md` — kø av nye verktøy/moduler.
- `BESLUTNINGER.md`, `FALLGRUVER.md`, `ARKITEKTUR.md`.

NB: `opk-app/` → Netlify (preview). Prod-portalen `/min-side/` er et **separat** deploy-mål (nettside-repo → One.com). Ikke bland.
