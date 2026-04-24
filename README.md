# Luca's Hub

Private co-parenting dashboard for Luca — parenting-time schedule, change requests, daily routines, approved media, documents, expenses, and communication log. Single-page web app; all data lives in the browser (localStorage).

Access is restricted to:

- `patrick.dilalla@gmail.com` (Tyler)
- `88.anali@gmail.com` (Anali)

## Files

| File | Purpose |
|------|---------|
| `index.html` | The entire app in one file (HTML + CSS + JS). Netlify Identity gate is wired in. |
| `netlify.toml` | Netlify build config + security headers (CSP, X-Frame-Options, noindex). |
| `robots.txt` | Tells search engines to stay out. |
| `.gitignore` | Keeps OS/editor/secret files out of git. |
| `luca-hub.html` | Old copy of the app — safe to delete (`git rm luca-hub.html`). |

## One-time setup

### 1. Push this folder to GitHub

From this folder (`Luca Calendar/`) in a terminal:

```bash
git init
git remote add origin https://github.com/patrickdilalla-lab/luca-calendar.git
# If you already cloned the repo, skip the two lines above.

git rm -f luca-hub.html    # remove the duplicate copy, if present
git add .
git commit -m "Add Netlify Identity gate + security headers"
git branch -M main
git push -u origin main
```

If `git push` complains about non-fast-forward, pull first: `git pull --rebase origin main` then push again.

### 2. Connect the repo to Netlify (free)

1. Go to <https://app.netlify.com/> → **Add new site** → **Import from Git** → **GitHub** → pick `patrickdilalla-lab/luca-calendar`.
2. Build settings: **leave blank** (no build command, publish directory `.`). The `netlify.toml` in the repo already configures this.
3. Click **Deploy**. You'll get a URL like `https://luca-calendar-xyz.netlify.app`.

### 3. Turn on Identity and lock it down

1. In the Netlify site dashboard → **Identity** → **Enable Identity**.
2. **Registration preferences** → set to **Invite only**. Critical — this means nobody can sign up without an invite.
3. (Optional but recommended) **External providers** → enable **Google** so you can sign in with your Gmail account with one click.
4. **Emails** tab → customize the invite email subject/body if you want (not required).
5. Go back to **Identity** → **Invite users** → send invites to:
   - `patrick.dilalla@gmail.com`
   - `88.anali@gmail.com`

Each person clicks the invite link, sets a password (or links their Google account), and from then on they log in at your Netlify URL.

### 4. Defense in depth

The app also checks the email address client-side in `index.html`:

```js
var ALLOW = ['patrick.dilalla@gmail.com', '88.anali@gmail.com'];
```

If someone's email is somehow accepted by Netlify but isn't in that list, they're immediately logged out. To add or remove someone later, edit that array in `index.html`, commit, and push — Netlify auto-deploys on push.

## How access actually works

1. Visitor hits the site → sees a lock screen with a **Sign in** button, nothing else.
2. They click Sign in → Netlify Identity modal asks for email + password (or Google).
3. Netlify checks the email against its invited-users list. Non-invited emails are rejected.
4. The app's own script checks the email against the allowlist. Any mismatch → immediate logout.
5. Only after both checks pass does the dashboard become visible.

### Honest limits

- This is a **client-side gate**. Someone technical who views the raw HTML can see the page structure before logging in. That's OK here because **the actual data (schedule, notes, expenses) lives in each browser's localStorage** — there is no server-side database to read. No Luca-specific content leaks through the gate.
- localStorage per browser means the two parents' data doesn't auto-sync. The app has an **Export / Backup** option in Settings — use that if you want to share state, or move to a small backend later (Supabase, Firebase) if you want automatic sync.
- If you ever lose access to both Netlify Identity accounts, you can re-invite from the Netlify dashboard as the site owner.

## Local development

Open `index.html` directly in a browser. The Identity widget will try to reach Netlify — it won't fully work locally without the site being deployed, so just test the gate after you deploy the first time.

Alternatively, use the Netlify CLI:

```bash
npm install -g netlify-cli
netlify dev
```

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

## Changing the allowlist later

1. Edit `index.html`, find `var ALLOW = [...]`, add or remove emails.
2. In Netlify → Identity → Users, invite the new person or delete the old one.
3. `git commit -am "Update allowlist"` → `git push`. Netlify redeploys automatically.
