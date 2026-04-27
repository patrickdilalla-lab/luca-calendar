# Luca's Hub

Private co-parenting dashboard for Luca — parenting-time schedule, change requests, daily routines, approved media, documents, expenses, and communication log. Single-page web app; all data lives in the browser (localStorage).

Access is locked behind a shared password. The current password is set; share it with Anali via a secure channel (text/Signal/in-person, not email). To rotate it later, see *Rotating the password* below.

Intended users:
- Tyler (`patrick.dilalla@gmail.com`)
- Anali (`88.anali@gmail.com`)

## Files

| File | Purpose |
|------|---------|
| `index.html` | The entire app in one file (HTML + CSS + JS). Password gate is wired in. |
| `netlify.toml` | Netlify build config + security headers (CSP, X-Frame-Options, noindex). |
| `robots.txt` | Tells search engines to stay out. |
| `.gitignore` | Keeps OS/editor/secret files out of git. |

## How the lock works

Open the site → password prompt → enter the right password → the app unlocks. The browser remembers you (in `localStorage`) so you don't have to retype on every visit. Click **Log out** in the top-right to clear it.

The password itself is **not** stored in the source code — only its SHA-256 hash. So if someone "view-source"s the page, they see a hash like `73a3199c…`, not the password.

### Honest limits of a shared-password gate

- **It's a shared secret, not per-user accounts.** If the password leaks, both you and Anali need to rotate it.
- **It runs in the browser.** A determined technical visitor can read the page's HTML markup before signing in. That's still safe here because the actual Luca data (schedule, notes, expenses) lives in each browser's localStorage — there is no server database to read. No Luca-specific content leaks through the gate.
- **Don't use a weak password.** SHA-256 is fast, so trivial passwords ("password", "luca123") could be cracked offline. Use at least 12 characters with a number or symbol mixed in.

## One-time setup

### 1. Push to GitHub

```bash
cd "/Users/dilallpa/Documents/Claude/Projects/Luca Calendar"
git add -A
git commit -m "Switch to shared-password gate"
git push origin main
```

### 2. Deploy to Netlify (free)

If your site is already connected to Netlify (from before), it will auto-redeploy on push — done.

If not yet connected:

1. Go to <https://app.netlify.com/> → **Add new site** → **Import from Git** → **GitHub** → pick `patrickdilalla-lab/luca-calendar`.
2. Build settings: leave blank. The `netlify.toml` in the repo configures everything.
3. Click **Deploy**. You'll get a URL like `https://luca-calendar-xyz.netlify.app`.
4. (Optional) Site configuration → **Change site name** → pick something memorable.

That's it. No Identity setup needed.

### 3. Test it

Open the Netlify URL → password prompt appears → type the password → app unlocks.

## Rotating the password

You'll want to change the password from the default before sharing the URL. Here's how:

1. Pick a new password. Aim for 12+ characters.

2. In a Mac Terminal, generate the SHA-256 hash:

   ```bash
   echo -n "YourNewPassword" | shasum -a 256
   ```

   You'll get something like `a1b2c3d4...  -`. Copy just the long hex string (drop the trailing `  -`).

3. Open `index.html`, find this line near the bottom:

   ```js
   var PASSWORD_HASH = '73a3199c459258a364166139a6658332c180d8e8a9461b3caf8cdbad19b8471f';
   ```

4. Replace the hex string between the quotes with the new hash.

5. Commit and push:

   ```bash
   git commit -am "Rotate password"
   git push origin main
   ```

6. Netlify auto-redeploys in ~30 seconds. Anyone who was signed in on the old password will be signed out automatically (because the saved token no longer matches the new hash). Share the new password with Anali via a secure channel — text, signal, in-person — not email.

## Local development

Open `index.html` directly in a browser — the password gate works offline since it doesn't need any third-party service.

## Feature list (all already implemented in `index.html`)

- **Dashboard** — this-week view, today's timeline, quick actions.
- **Schedule** — weekly template editor, per-day overrides, Jiu Jitsu / activities.
- **Change requests** — propose, accept, modify, reject with reply notes.
- **Travel / unavailable** — mark dates one parent isn't available.
- **Daily routines** — morning / after-school / bedtime hygiene + meal plan.
- **Approved media** — movies/TV allowlist + blocklist + screen-time rules.
- **Info bank** — emergency contacts, medical, school, packing list.
- **Documents** — links to Google Drive files; cyber-security notes.
- **Journal / communication log** — tone-checked entries between co-parents.
- **Expenses** — paid-by, split %, receipt link.
- **Activity log** — audit trail of everything.
- **Settings** — parents, Google Calendar ID, export deviations as `.ics`.

## Sync caveat

Because all data lives in each browser's localStorage, your edits and Anali's don't auto-merge. Use **Settings → Backup / Sync** to export and import state. If you outgrow that, switching to a tiny backend (Supabase free tier) gives real per-account sync — happy to wire that up later.
