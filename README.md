# OPK-app — preview/produkt-repo

Dette er **appens eget repo** (klientportal/ADHD-app som selvstendig produkt). Hostes på Netlify med **continuous deploy**: hver `git push` går automatisk live. Ingen manuell opplasting.

- `index.html` — selve appen (kanonisk fil; rediger denne).
- `manifest.webmanifest`, `img/icon-*.png` — PWA / app-ikon (enso-logo).
- `_headers`, `netlify.toml`, `robots.txt` — privat hosting (`noindex`).
- Mock-modus slår seg på automatisk på ikke-prod-host. Innloggingskoder: **OPK1** (søvn-modul), **OPK2** (adhd-modul).

## Engangsoppsett (du gjør dette én gang, i nettleser + Terminal)

1. Opprett et **tomt, privat GitHub-repo**, f.eks. `opk-app` (ingen README/`.gitignore` fra GitHub).
2. I Terminal, stå i denne mappa (`opk-app/`):
   ```bash
   git init
   git add -A
   git commit -m "OPK-app: første versjon"
   git branch -M main
   git remote add origin https://github.com/<din-bruker>/opk-app.git
   git push -u origin main
   ```
3. I **Netlify**: *Add new site → Import an existing project → GitHub → velg `opk-app`*.
   - Build command: **(tomt)**
   - Publish directory: **`.`** (rot)
   - Deploy.
   - (Eller koble det eksisterende `opk-app-preview`-prosjektet til repoet under *Site configuration → Build & deploy → Link repository*.)
4. (Valgfritt) *Site configuration → Change site name* for en mindre gjettbar URL.

## Daglig flyt (etter oppsett)

1. Claude redigerer `index.html` (eller andre filer) her i mappa.
2. Du kjører:
   ```bash
   git add -A && git commit -m "beskrivelse" && git push
   ```
3. Netlify bygger og publiserer automatisk (~30 sek). Hard-refresh på telefonen.

## Personvern

`noindex` + `robots.txt` gjør lenken ulistet/ikke-googlebar. Det er **ingen ekte innloggingslås** på Netlify gratis — hvem som helst med lenken kan åpne. For ekte lås: Netlify Pro (passord) eller Cloudflare Access (gratis, e-post). Greit nå siden appen kun kjører mock-data.
