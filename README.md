# Xaraash

Suuqa xayeysiisyada Bariga Afrika oo dhan. Luqado: Soomaali, English, Kiswahili, አማርኛ, Français.
East Africa classifieds app. Somalia prices are in US dollars only; other countries show local currency with dollars as the small secondary amount.

> Xayeysiisyada waa tusaalayaal. Sarifka waa sawir-qaad 2026-10-20, ma aha toos. / Listings are samples. Exchange rates are a 2026-10-20 snapshot, not live.

## Qaab-dhismeedka / Layout

| Path | Waxa uu yahay / What |
|---|---|
| `www/` | App-ka laftiisa (HTML/CSS/JS, PWA) / the app itself |
| `www/config.js` | Halkan geli URL-ka chatbot-ka / put your chat server URL here |
| `apps-script/Code.gs` | Keydinta Google Sheets + Drive (xayeysiisyo iyo sawiro) / shared storage |
| `api/chat.js` | Server-ka chatbot-ka (furaha API halkan ayuu joogaa) / chat server, API key lives here |
| `capacitor.config.json` | Beddelka web-ka ilaa Android/iOS / wraps the web app for Android and iOS |
| `.github/workflows/pages.yml` | Daabac web-ka GitHub Pages / publish the web app |
| `.github/workflows/android.yml` | Samee APK Android (gacan ahaan) / build an Android APK (manual) |

## 1. Tijaabi gudaha / Run locally

```bash
npm run serve        # http://localhost:8080
```

## 2. GitHub

```bash
git init -b main
git add .
git commit -m "Xaraash app"
gh repo create xaraash-app --private --source=. --push   # ama ku samee repo GitHub.com oo `git push`
```

Daabac web-ka: Repo → Settings → Pages → Source: **GitHub Actions**. Markaas push kasta wuxuu cusboonaysiinayaa bogga.

## 3. Chatbot-ka

Chatbot-ku wuxuu u baahan yahay server yar, maxaa yeelay furaha API ma gali karo app-ka.

1. Ka samee furaha API console.anthropic.com.
2. Geli mashruuca Vercel (vercel.com → Add New → Project → import repo-gan). Hal-xidhka: `vercel.json` waa diyaar.
3. Vercel → Settings → Environment Variables: `ANTHROPIC_API_KEY` = furahaaga. Ikhtiyaari: `CLAUDE_MODEL` (default `claude-haiku-5-5`), `ALLOWED_ORIGIN` (tusaale `https://USERNAME.github.io`).
4. Nuqul ka qaad URL-ka (`https://MAGACA.vercel.app/api/chat`) oo geli `www/config.js`:
   ```js
   window.XARAASH_CHAT_URL = 'https://MAGACA.vercel.app/api/chat';
   ```
5. Commit + push.

Ilaalin: server-ku wuxuu xaddidaa 12 fariin/daqiiqad IP kasta (best-effort), 500 xaraf fariin kasta, iyo 400 token jawaab kasta. Hubi kharashka API-ga ee akoonkaaga. Ha geli furaha API `www/` gudahiisa.

## 3b. Keydin Google Drive (xayeysiisyo iyo sawiro oo aaladaha oo dhan ka muuqda)

Bilaash, adigoo isticmaalaya akoonkaaga Google. Xayeysiisyada waxay galaan Google Sheet, sawirradana Drive.

1. Tag sheets.new, magac u bixi "Xaraash posts".
2. **Extensions → Apps Script**. Tirtir koodhka jira, ku dheji waxa ku jira `apps-script/Code.gs`, dhagsii Save.
3. Kor ka dooro function-ka **setup** → **Run**. Oggolow marka Google weydiiyo (haddii uu leeyahay "Google hasn’t verified this app": **Advanced → Go to … (unsafe)**; waa script-kaaga).
4. **Deploy → New deployment → Web app**. Execute as: **Me**. Who has access: **Anyone**. **Deploy**. Nuqul ka qaad **Web app URL** (ku dhammaata `/exec`).
5. Geli URL-kaas `www/config.js`:
   ```js
   window.XARAASH_API_URL = 'https://script.google.com/macros/s/.../exec';
   ```
6. Upload `www/config.js` iyo `www/index.html` GitHub, sug Pages 1 ilaa 2 daqiiqo.

Halka lagu keydiyo: Sheet-ka "Xaraash posts" iyo galka "Xaraash photos" waxay ku jiraan **My Drive** (drive.google.com/drive/my-drive). Safka kasta ee Sheet-ka waa xayeysiis; si toos ah ayaad u beddeli kartaa ama u tirtiri kartaa. Si aad u qarisid, status ku qor `hidden`.

Edit/tirtir app-ka dhexdiisa: xayeysiiska aad soo dirtay, bogga faahfaahintiisa waxaa ka muuqda **Edit** iyo **Delete** (aaladdaas oo keliya, furaha waxaa lagu keydiyaa browser-ka). Sawirrada marka la soo diro lama beddeli karo.

Xayeysiis kasta wuxuu leeyahay magac iyo lambar taleefan (ikhtiyaari), oo ka muuqda bogga faahfaahinta iyadoo la leh badhamada Wac iyo WhatsApp. Lambarka waa public, qof walba wuu arki karaa.

Xusuusin:
- Haddii aad Code.gs beddesho, **Deploy → Manage deployments → Edit → New version**.
- Link-ga `/exec` waa public (ku jira config.js), qof walba wuu soo dirri karaa xayeysiis. Server-ku wuxuu xaddidaa 20 xayeysiis daqiiqadii, 5 sawir, 80 xaraf cinwaan. Si aad xayeysiis u qarisid, Sheet-ka **posts** ku qor `hidden` safka ugu dambeeya (status). Tijaabo ahaan ayuu u fiican yahay; mashruuc dhab ah waxaa lagula talinayaa database sida Supabase.
- Sawirradu waxay noqdaan "anyone with the link" Drive-kaaga, waxayna isticmaalaan booskaaga Drive-ka.
- Kamarada/sawirka, Apps Script iyo Drive thumbnails lama tijaabin halkan; tijaabi taleefankaaga ka hor intaadan dadka u dirin.

## 4. Mobile app

### Habka degdegga ah: PWA
Fur URL-ka GitHub Pages ee Chrome (Android) ama Safari (iPhone) → **Add to Home Screen**. Wuxuu u shaqeeyaa sida app, oo offline ayuu furmaa.

### Android APK (Capacitor)
Waxay u baahan tahay Node 20+ iyo Android Studio (ama GitHub Action).
```bash
npm install
npx cap add android
npx cap sync android
npx cap open android      # Android Studio → Build → Build APK
```
Ama GitHub → Actions → **Build Android APK** → Run workflow → soo deji `xaraash-debug-apk`. (Waa debug APK; Play Store waxay u baahan tahay release signing.)

### iPhone (Capacitor)
Waxay u baahan tahay Mac + Xcode + Apple Developer account.
```bash
npm install
npx cap add ios
npx cap sync ios
npx cap open ios
```

## 5. Waxaan weli la dhisin / Not built yet
- Gelitaan (login) iyo diiwaangelin, badhamada La xiriir / Soo sheeg, keydin xayeysiis dhab ah (database), sawir-gelin, iyo sarif toos ah. Xayeysiisyada hadda waa tusaalayaal ku jira `www/index.html`.
- Boggagga badbaadada, shuruudaha iyo hantida maskaxda waa qabyo tusaale ah; turjumaadaha Amharik iyo Faransiis waa kuwo la sameeyay si toos ah, ha la hubiyo qof ku hadla ka hor daabicidda.
- Workflow-ga Android iyo tallaabooyinka iOS lama tijaabin halkan.

## English quick start
1. `npm run serve` to preview.
2. Push to GitHub, enable Pages (Source: GitHub Actions).
3. Deploy the repo to Vercel, set `ANTHROPIC_API_KEY`, paste the function URL into `www/config.js`, push.
4. Install as a PWA, or build the Android APK with Capacitor / the Android workflow.


## Update (2026-10-23)
So: Dhammaan badhamada hadda way shaqeeyaan (profile, soo sheeg, qiimeyn, rakib app, bogagga macluumaadka). Ku soo celi Code.gs Apps Script oo dooro Deploy > Manage deployments > New version.
En: All buttons now work (profile, report, rating, install, info pages). Paste the new Code.gs into Apps Script and deploy a New version so report/rating are saved to your Sheet (tabs "reports" and "ratings").
