# Cambie — cambie.xyz

Static marketing site for [Cambie](https://cambie.xyz), hosted on Firebase Hosting.
Single self-contained HTML file. Minification runs in CI — the repo always contains
readable source.

---

## Structure

```
cambie-web/
├── public/
│   ├── index.html          ← entire site (HTML + CSS + JS, readable source)
│   └── og.jpg              ← 1200×630 social preview image
├── .github/
│   └── workflows/
│       └── deploy.yml      ← auto-deploy on push to main (minifies in CI)
├── package.json            ← dev scripts (preview, deploy)
├── firebase.json           ← Firebase Hosting config
├── .firebaserc             ← Firebase project alias (project: cambie-web)
└── .gitignore
```

---

## Editing the site

Always edit `public/index.html` — it is the readable source and is what lives in
the repo. **Do not manually minify it.** Minification happens automatically in CI
before each deploy; the minified version never touches the repo.

**Preview locally:**

```bash
open public/index.html          # instant, no server needed

# or with the Firebase emulator (closer to production):
npm install                     # first time only
npm run preview                 # → http://localhost:5000
```

**Deploy:**

```bash
git add .
git commit -m "your message"
git push                        # GitHub Action minifies and deploys automatically
```

---

## Automated Deploy (GitHub Actions)

Every push to `main` triggers `.github/workflows/deploy.yml`:

```
checkout readable source
  → minify HTML/CSS/JS in CI runner
    → firebase deploy --only hosting (using FIREBASE_TOKEN secret)
```

**This is free.** GitHub Actions gives 2,000 free minutes/month on private repos
(unlimited on public), and Firebase Hosting's Spark plan has no cost for static sites.

### GitHub secret required

The workflow authenticates using a Firebase CI token stored as a GitHub secret.
This was set up using `firebase login:ci`. If the token ever expires or needs to
be rotated:

```bash
firebase login:ci               # prints a new token
```

Then update it in GitHub: repo → **Settings → Secrets and variables → Actions →
`FIREBASE_TOKEN`** → update value.

### Manual deploy (bypassing CI)

```bash
firebase login                  # if credentials have expired: firebase login --reauth
firebase deploy --only hosting
```

---

## Firebase Notes

- **Project ID**: `cambie-web` (set in `.firebaserc`)
- **Hosting config**: only the `public/` directory is served; `firebase.json`,
  dotfiles, and `node_modules` are excluded
- **SPA rewrite**: all routes resolve to `index.html` — correct for a single-page site
- **Free tier (Spark plan)**: 10 GB storage, 360 MB/day transfer, custom domain, SSL
- **Custom domain** (`cambie.xyz`): Firebase Console → Hosting → Add custom domain
  → follow the DNS instructions (usually live within an hour)

---

## Integrations

| Service | Status | Detail |
|---|---|---|
| Formspree (contact form) | ✅ Live | `https://formspree.io/f/xpqkgzqb` — submissions go to Formspree dashboard |
| Calendly | ✅ Live | "Schedule a conversation" → `https://calendly.com/cambie-info/30min` |
| OG / social image | ✅ Live | `public/og.jpg` + meta tags pointing to `https://cambie.xyz/og.jpg` |
| GitHub Actions deploy | ✅ Live | Triggers on push to `main`, uses `FIREBASE_TOKEN` secret |
| Custom domain | ✅ Live | `cambie.xyz` connected in Firebase Console |
| Email (`info@cambie.xyz`) | ✅ Fine as-is | Formspree notifies you of submissions; reply manually from any inbox for now |

---

## Remaining TODOs

- [ ] **OG preview test**: Paste `cambie.xyz` into [opengraph.xyz](https://www.opengraph.xyz) to confirm social previews render correctly on LinkedIn, Slack, iMessage
- [ ] **Email routing** (when needed): Forward `info@cambie.xyz` to your inbox via [Cloudflare Email Routing](https://developers.cloudflare.com/email-routing/) (free) — only worth doing once volume picks up or you want a cleaner reply-from address
- [ ] **Analytics**: Optional — once there is traffic worth measuring (Fathom or Plausible are good privacy-friendly options)
