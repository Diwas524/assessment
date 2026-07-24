# Setting up Firebase for the assessment form

## 1. Create/open your Firebase project
1. Go to https://console.firebase.google.com and create a project (or use an existing one).
2. **Upgrade to the Blaze (pay-as-you-go) plan.** Phone Auth SMS messages require this — the free Spark plan won't send codes. Cost is small (a few cents per SMS).

## 2. Turn on the pieces this form needs
In the left sidebar of the Firebase Console:
- **Authentication → Sign-in method →** enable **Phone**.
- **Authentication → Settings → Authorized domains →** add the domain you'll host this file on (e.g. `neighborcarehomes.com`, or `localhost` while testing). If your domain isn't on this list, phone sign-in will silently fail.
- **Firestore Database →** click **Create database** (production mode is fine — the rules file below locks it down).
- **Storage →** click **Get started** (production mode is fine — same reasoning).

## 3. Get your web app config
- **Project settings (gear icon) → General → Your apps → Add app → Web (`</>`)**.
- Firebase gives you a `firebaseConfig` object. Copy it into `assessment.html`, replacing the placeholder near the top:
```js
const firebaseConfig = {
  apiKey: "...",
  authDomain: "...",
  projectId: "...",
  storageBucket: "...",
  messagingSenderId: "...",
  appId: "..."
};
```

## 4. Deploy the security rules (included in this batch)
These lock every assessment record and file to the phone number that verified it — nobody can read or write anyone else's data.

```bash
npm install -g firebase-tools
firebase login
# from the folder containing firebase.json, .firebaserc, firestore.rules, storage.rules:
firebase deploy --only firestore:rules,storage:rules
```
Before running this, edit `.firebaserc` and put your real project ID in place of `YOUR_PROJECT_ID`.

If you'd rather not install the CLI, you can instead paste the contents of `firestore.rules` into **Firestore Database → Rules** in the console (and click Publish), and `storage.rules` into **Storage → Rules** (and click Publish).

## 5. Host it somewhere real — don't just double-click the file
This is almost certainly the cause of the `firebase is not defined` / `auth is not defined` errors you saw. Firebase Phone Auth's reCAPTCHA and script loading depend on the page being served over `http(s)://` from an **authorized domain** — opening the file directly from disk (`file:///...`) or previewing it in a sandboxed preview window can block the SDK from loading or initializing.

To test properly, either:
- **Firebase Hosting** (easiest, free): put `assessment.html` in a `public/` folder next to these config files, then run `firebase deploy --only hosting`. You'll get a real `https://your-project.web.app` URL.
- **Local dev server**: run `npx serve .` (or `python3 -m http.server`) in the folder and open `http://localhost:PORT/assessment.html` — just make sure `localhost` is in your Authorized Domains list (step 2).
- **Your existing website**: upload `assessment.html` to your host as usual, as long as that domain is in Authorized Domains.

## 6. Quick checklist if you still see errors
- Open DevTools → Network tab, reload, and check whether the `firebase-app-compat.js` etc. requests actually succeeded (not blocked by an ad blocker or corporate network filter).
- Open DevTools → Console — Firebase usually prints a clear error (e.g. `auth/invalid-app-credential`, `auth/unauthorized-domain`) once the SDK does load, which is far more actionable than "not defined."
- Confirm you replaced *all* the placeholder values in `firebaseConfig` — a leftover `"YOUR_API_KEY"` will fail silently in some browsers.

## Files in this batch
| File | Purpose |
|---|---|
| `firestore.rules` | Locks each assessment doc to its verified phone number |
| `storage.rules` | Locks attachment uploads to the owner's phone number, PDF-only, 20MB max |
| `firebase.json` | Firebase CLI project config (rules + optional hosting) |
| `firestore.indexes.json` | Empty indexes stub (required by `firebase.json`, safe to leave as-is) |
| `.firebaserc` | Where you put your real project ID |
| `assessment.html` | The form itself |
