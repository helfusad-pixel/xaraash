# Xaraash

Suuqa xayeysiisyada Bariga Afrika oo dhan. Luqado: Soomaali, English, Kiswahili, አማርኛ, Français.
East Africa classifieds app. Somalia prices are in US dollars only; other countries show local currency with dollars as the small secondary amount.

> Xayeysiisyada waa tusaalayaal. Sarifka waa sawir-qaad 2026-10-20, ma aha toos. / Listings are samples. Exchange rates are a 2026-10-20 snapshot, not live.

## Qaab-dhismeedka / Layout

| Path | Waxa uu yahay / What |
|---|---|
| `www/` | App-ka laftiisa (HTML/CSS/JS, PWA) / the app itself |
| `www/config.js` | Halkan geli URL-ka chatbot-ka / put your chat server URL here |
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
