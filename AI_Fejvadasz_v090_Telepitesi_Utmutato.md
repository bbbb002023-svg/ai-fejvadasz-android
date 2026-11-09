# AI Fejvadász 0.9.0 – frissítőcsomag (Android + Render)

**A csomag nem APK.** A meglévő 0.8.2 Android projekt ZIP-jére és a 0.8.2.1-es Render szerver ZIP-jére épülő **felülíró frissítés**. A korábbi ZIP-eket ne töröld. A 0.9-es csomagot ne bontsd ki a GitHubra történő feltöltés előtt. Titkos kulcsot soha ne helyezz a ZIP-be.

## Mit tartalmaz?

- Régió **elsőként** vizsgálva, kötelező város/vármegye beállítással; régiómező az Állások oldal tetején.
- Brave Search: legfeljebb 20 keresőindex-találat forrásonként, a közelmúltban indexelt oldalak előnyben részesítésével. Ez **nem** bizonyítja, hogy friss maga az állás.
- Az **eredeti hirdetési URL** megnyitása és a `JobPosting` strukturált adatok ellenőrzése. Csak igazolható közzétételi dátumú, maximum 20 napos, jelenleg elérhető, régión belüli állások jutnak a főlistába. Nincs megfelelő bizonyíték → kimarad.
- Csak a ténylegesen visszakapott munkakör/cég/adatok jelennek meg; AI-elemzés a CV-ből jóváhagyott szakmai profil és a hirdetés tartalma alapján, több kisebb csoportban.
- **80% FELETT** automatikusan az Ajánlott állások részbe kerülnek az AI-val értékelt hirdetések. A százalék **szakmai illeszkedés**, nem felvételi valószínűség.
- A fő kártyán munkakör → munkáltató → régió, megjelenési kor, értékelés. Rákoppintva részletek; a portál / eredeti link **az adatlap végén**.
- Nem küld jelentkezést automatikusan. Nem igényel újabb API-kulcsot.

**Bizonyíthatósági korlát:** A JSON-LD `JobPosting` adat és az elérhető oldal nem garantálja, hogy a munkáltató a jelentkezést az adott percben még elfogadja. Ezt a részleteknél kifejezetten jelezzük. Sok keresőindex-találat emiatt szándékosan nem jelenik meg. Nem tudunk minden portálra kiterjedő 100%-os aktivitásgaranciát vállalni.

## 1. GitHub – APK összeállítása

A meglévő repository: `https://github.com/bbbb002023-svg/ai-fejvadasz-android`

1. Add file → Upload files: töltsd fel a repository **gyökerébe** a `AI_Fejvadasz_v090_Alkalmazas_Szerver_Frissites.zip` fájlt. Commit changes.
2. Nyisd meg: `.github/workflows/build-apk.yml` → Edit (ceruza). A `ZIP kibontasa` részben a meglévő sor **után** írj be még egy sort:

   ```yaml
   - name: ZIP kibontasa
     run: |
       mkdir -p project
       unzip -q AI_Fejvadasz_v082_CV_Keresok_Android_Projekt.zip -d project
       unzip -oq AI_Fejvadasz_v090_Alkalmazas_Szerver_Frissites.zip -d project
   ```

3. Commit changes. Actions → legújabb **AI Fejvadasz APK** futás → `AI-Fejvadasz-APK` artefaktum. Ebből származik a telepítendő APK. A meglévő Android-projekt egyéb részét a patch nem módosítja. **Fontos:** a GitHub Actions debug APK aláírása futásonként eltérhet. Ha a telefon nem engedi a frissítést, a régi alkalmazást NE töröld a helyi CV és beállítások biztonságos mentése előtt; a tartós megoldás állandó aláírókulccsal készített APK.

## 2. Render – szerverfrissítés

Service: `https://dashboard.render.com/` → `ai-fejvadasz-android` → Settings / Build & Deploy.

Build Command teljes tartalma:

```bash
unzip -oq AI_Fejvadasz_AI_Szerver_v0821_Brave_Javitas.zip -d ai-server && unzip -oq AI_Fejvadasz_v090_Alkalmazas_Szerver_Frissites.zip -d ai-server && node ai-server/apply-v09-server.js && cd ai-server && npm install --omit=dev
```

Start Command marad:

```bash
cd ai-server && npm start
```

Mentsd el / Deploy. **A meglévő `OPENAI_API_KEY`, `AI_APP_ACCESS_TOKEN`, `BRAVE_SEARCH_API_KEY` változókhoz ne nyúlj.**

Ezután a `/health` elvárt `version` értéke `0.9.0`.

## 3. Beállítás a telefonon

Állások → **1. lépés: keresési régió** → add meg pontosan a városokat / vármegyéket, pl. `Nagykanizsa, Zala vármegye, Vas vármegye` → Régió mentése → Új keresés indítása.

A keresés az eredeti oldalak ellenőrzése és több AI-kérés miatt több percet is igénybe vehet, és kevesebb találatot adhat, mint a régi 0.8.2.1, amely ellenőrizetlen linkeket is mutatott.

## Tesztek és biztonság

A patch tartalmaz 17 automatikus egységtesztet (v09-verify.test.js és v09-ui.test.js). Ezek helyben sikeresen lefutottak. Az **APK GitHub Actions-fordítása és az éles hirdetéses végpont tesztje még nem történt meg** a csomag átadásakor.

- A szerver titkai csak Render Environment Variables-ként maradnak.
- Az ellenőrzés nem követ külső hosztra átirányító, nem biztonságos hivatkozásokat.
- A portáloldal esetleges blokkolása vagy hiányzó `JobPosting` adata nem jelent igazolt aktív hirdetést.
- Az álláskeresés nem küld pályázatokat a felhasználó jóváhagyása nélkül.
