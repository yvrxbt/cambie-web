# Cambie — cambie.xyz

Static marketing site for [Cambie](https://cambie.xyz), hosted on Firebase Hosting.
Single self-contained HTML file — no build step, no dependencies, no node_modules.

---

## Structure

```
cambie-web/
├── public/
│   └── index.html          ← entire site (HTML + CSS + JS, self-contained)
├── .github/
│   └── workflows/
│       └── deploy.yml      ← auto-deploy on push to main
├── firebase.json           ← Firebase Hosting config
├── .firebaserc             ← Firebase project alias (project: cambie-web)
└── .gitignore
```

---

## Local Preview

No server required — just open the file directly:

```bash
open public/index.html
```

Or use the Firebase local emulator for a closer-to-production preview:

```bash
npm install -g firebase-tools   # if not already installed
firebase serve --only hosting
# → http://localhost:5000
```

---

## Before You Deploy

### 1. Wire up the contact form

The form currently simulates a successful submission. To make it live, sign up at
[formspree.io](https://formspree.io), create a form, and replace the commented-out
block in `index.html`:

```js
// Find this block near the bottom of the <script> tag:
// await fetch('https://formspree.io/f/YOUR_ID', {
//   method: 'POST', headers: { 'Accept': 'application/json' },
//   body: new FormData(form)
// });

// Replace with (uncomment and fill in your ID):
const res = await fetch('https://formspree.io/f/YOUR_ID', {
  method: 'POST',
  headers: { 'Accept': 'application/json' },
  body: new FormData(form)
});
if (!res.ok) throw new Error('Form submission failed');
```

Free Formspree tier: 50 submissions/month. Paid tiers start at $10/mo.

### 2. OG social image (optional but recommended)

Currently there is no `og:image` meta tag. For proper link previews on LinkedIn,
Slack, iMessage etc., create a `1200×630px` image, upload it to Firebase Storage
or a CDN, and add to `<head>`:

```html
<meta property="og:image" content="https://cambie.xyz/og-image.jpg">
<meta name="twitter:image" content="https://cambie.xyz/og-image.jpg">
<meta name="twitter:card" content="summary_large_image">
```

### 3. Analytics (optional)

Firebase Analytics is already included in firebase.json rewrites. To enable,
add the Firebase SDK snippet to `index.html` or use Firebase's built-in
Hosting analytics dashboard at console.firebase.google.com.

---

## Minification

The site is a single HTML file — minify everything in one pass:

```bash
# Install once
npm install -g html-minifier-terser

# Minify (output to a separate file, then swap)
html-minifier-terser \
  --collapse-whitespace \
  --remove-comments \
  --remove-redundant-attributes \
  --minify-css true \
  --minify-js true \
  public/index.html -o public/index.html
```

> Tip: Run minification right before committing the deploy commit. The inline
> theme script, noscript tag, and SVG favicon are all written to survive
> minification without changes.

---

## Manual Deploy

```bash
# 1. Make sure you're logged in
firebase login

# 2. Deploy (Hosting only — skip Functions, Firestore, etc.)
firebase deploy --only hosting
```

That's it. Firebase will print the live URL when done.

---

## Automated Deploy (GitHub Actions)

Every push to `main` triggers `.github/workflows/deploy.yml`, which deploys
directly to Firebase Hosting. **This is free** — GitHub Actions gives 2,000
free minutes/month on private repos (unlimited on public repos), and Firebase
Hosting's Spark plan has no cost for static hosting.

### One-time setup

The easiest path is to let the Firebase CLI handle everything automatically:

```bash
# From the cambie-web directory
firebase init hosting:github
```

This command will:
1. Ask which GitHub repo to connect
2. Create a Firebase service account automatically
3. Store the secret in your GitHub repo as `FIREBASE_SERVICE_ACCOUNT_CAMBIE_WEB`
4. Generate (or confirm) the workflow file

If you prefer to do it manually:

1. Go to [Google Cloud Console](https://console.cloud.google.com) → IAM & Admin → Service Accounts
2. Create a service account with the **Firebase Hosting Admin** role
3. Generate a JSON key
4. In your GitHub repo: Settings → Secrets → Actions → New repository secret
   - Name: `FIREBASE_SERVICE_ACCOUNT_CAMBIE_WEB`
   - Value: paste the entire contents of the JSON key file
5. Push to `main` — the action will run automatically

### What the action does

```
push to main
  → checkout code
  → authenticate with Firebase using the service account secret
  → run firebase deploy --only hosting
  → post deploy status as a GitHub commit check
```

Preview channels (optional): the action also supports deploying PRs to temporary
preview URLs. See [FirebaseExtended/action-hosting-deploy](https://github.com/FirebaseExtended/action-hosting-deploy)
for docs.

---

## Firebase Notes

- **Project ID**: `cambie-web` (set in `.firebaserc`)
- **Hosting config**: `public/` directory is served; `firebase.json`, dotfiles,
  and `node_modules` are excluded from deploys
- **SPA rewrite**: all routes rewrite to `index.html` — safe to keep even for
  a single-page site
- **Free tier (Spark plan)** includes: 10 GB storage, 360 MB/day transfer,
  custom domain, SSL — more than enough for this site
- **Custom domain** (`cambie.xyz`): add it in Firebase Console → Hosting →
  Add custom domain, then update your DNS with the provided records (usually
  takes < 1 hour to propagate)

---

## Remaining TODOs

- [x] Wire up Formspree (`https://formspree.io/f/xpqkgzqb`)
- [ ] Add `og:image` for social link previews (see above)
- [ ] Connect custom domain `cambie.xyz` in Firebase Console
- [ ] Run `firebase init hosting:github` once to link GitHub Actions
- [x] Calendly — "Schedule a conversation" → `https://calendly.com/cambie-info/30min`
