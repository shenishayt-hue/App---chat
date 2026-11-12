# LYNKO CHAT

Run: `npm install && npm start` (copy `.env.example` to `.env` first).

## Admin panel — `/admin`
Set `ADMIN_USER` and `ADMIN_PASS` in the environment (Railway → Variables). If either is missing, `/admin` returns 404.
- **Overview** – accounts, online now, 24h/7d active, live messages, 14-day chart, system health
- **Official** – post / delete messages in the verified *Lynko Official* chat (every account sees it, nobody can reply)
- **Errors** – server, browser and process errors, grouped (kept 14 days)
- **Users** – recent / searchable accounts (username, ID, last seen only — chats are end-to-end encrypted and unreadable)

Wrong-password lockout: 6 tries / 15 min per IP. Session lasts 12 h; changing `ADMIN_PASS` logs everyone out.

## Realtime extras
- **Typing indicator** – sent on the first keystroke, refreshed every 2.5 s, cleared the moment you stop / send.
- **Last seen / online** – heartbeat every 25 s while the app is open. Profile → *Hide my last seen* hides yours (and theirs from you).
- **Notifications** – Web Push (works with the app closed). Needs `VAPID_*` keys (`npm run vapid`). Alerts show only the sender name because chats are end-to-end encrypted. iPhone: must be added to the Home Screen first.
- **Voice calls** (Calls tab, 📞 in a chat) – WebRTC audio only; the server just relays the handshake. For reliable calls on mobile data add a TURN server (`TURN_URL/USER/PASS`).
- **Lynko AI** – Groq-powered assistant chat; `GROQ_API_KEY` in env. Toggle in Profile. AI messages go to Groq (not end-to-end encrypted); history is kept only on the phone for 24 h.

## Storage (Backblaze B2)
Set `S3_BUCKET`, `S3_ENDPOINT` (e.g. `https://s3.us-west-004.backblazeb2.com`), `S3_ACCESS_KEY` (keyID), `S3_SECRET_KEY` (applicationKey).
The region is read from the endpoint. At start-up the log says `✔ media bucket reachable` or the exact reason it is not.
Bucket lifecycle: set "Keep only the last version" so deleted/expired files do not pile up.

## Calls on mobile data
Add a free TURN relay: Cloudflare (`CF_TURN_KEY_ID`, `CF_TURN_API_TOKEN`) or Metered (`METERED_DOMAIN`, `METERED_API_KEY`).

## Call audio relay (no TURN needed)
If the direct phone-to-phone path can't be made within ~5 s, voice automatically goes through the server over a WebSocket (`/api/relay`, `relay.js`).
Audio is still end-to-end encrypted (AES-GCM per frame); the server only forwards bytes. Needs the `ws` package (already in package.json).

## Emoji picker
`emoji.js` adds a 😊 button in the composer (recent + cute categories). Native emoji → Apple style on iPhone.

## Android APK ("Install app" → downloads the APK)
1. GitHub repo → Settings → Secrets and variables → Actions → **Variables** → add `LYNKO_URL` = your live site (e.g. `https://xxxx.up.railway.app`, no trailing slash).
2. Actions tab → **Build APK** → Run workflow. It builds `android/` and commits `lynko.apk`.
3. Railway redeploys. On Android the **Install app** button now downloads `/lynko.apk`; until the file exists it falls back to the normal PWA install. iPhone keeps "Add to Home Screen".

## APK updates ("Update your app" popup)
Every **Build APK** run gets a higher version number and writes `apk-version.json`. Installed apps check `/api/app-version` (on open, when brought back to the front, every 30 min) and show **Update your app** → *Update now* downloads `/lynko.apk`.

**One-time setup — signing key (required, otherwise Android says "App not installed" when updating):**
1. On your computer: `keytool -genkeypair -v -keystore lynko.jks -alias lynko -keyalg RSA -keysize 2048 -validity 10000` (use the same password for the store and the key)
2. `base64 -w0 lynko.jks` (Mac: `base64 -i lynko.jks`) → copy the output
3. GitHub repo → Settings → Secrets and variables → Actions → **Secrets** → add `KEYSTORE_B64` (the base64 text) and `KEYSTORE_PASS` (the password)
4. Keep `lynko.jks` safe — lose it and nobody can update, they must uninstall first.

The first APK signed with this key must be installed fresh (uninstall the old one once). After that every build updates in place. Only needed for native changes — website changes (chat.html, JS) reach the app automatically without a new APK.
