# Marcus Wise Content Tracker — PWA

A self-contained Progressive Web App for tracking the Marcus Wise content plan: drafts, scheduled, published, formats (Image / Reels), destinations. Works fully offline once installed; cross-device sync via Supabase.

Categories: **lists · howto · 3points · tweet · script · quote · maxims · question**

## Files

- `index.html` — page shell + styles
- `app.js` — full app (Preact + htm)
- `sw.js` — service worker (offline cache)
- `manifest.json` — PWA manifest
- `seed.json` — first-load content library (~1,600 cards from your MW batches)
- `icon.svg` / `icon-192.png` / `icon-512.png` — app icon

## Before deploying — Supabase setup (5 min)

1. Go to **[supabase.com](https://supabase.com)** → sign in → **New Project**.
   - Name: `marcuswise-tracker`
   - Database password: generate + save
   - Region: closest to you
   - Wait ~2 min for provisioning.
2. **Left sidebar → Settings → API**. Copy:
   - **Project URL** (looks like `https://xxxxxxxx.supabase.co`)
   - **anon / public key** (long JWT string starting with `eyJ...`)
3. Open `app.js` and replace the two lines near the top:
   ```js
   const SUPABASE_URL = 'YOUR_SUPABASE_URL_HERE';
   const SUPABASE_ANON_KEY = 'YOUR_SUPABASE_ANON_KEY_HERE';
   ```
   with your actual values.
4. **Left sidebar → SQL Editor → New query** — paste and run:
   ```sql
   create table if not exists sync (
     user_id uuid primary key references auth.users(id) on delete cascade,
     data    jsonb not null,
     updated_at timestamptz default now()
   );
   alter table sync enable row level security;
   create policy "own row read"   on sync for select using  (auth.uid() = user_id);
   create policy "own row write"  on sync for insert with check (auth.uid() = user_id);
   create policy "own row update" on sync for update using  (auth.uid() = user_id);
   ```
5. **Left sidebar → Authentication → Providers → Email** — make sure **Email** is enabled.

## Deploying to GitHub Pages

1. Create the repo **`edgaritis/marcwise-tracker`** on GitHub (public).
2. Push the **contents of this `mw/` folder** to the repo root (so top level has `index.html`, `app.js`, `seed.json`, etc — not the `mw/` folder itself).
   ```bash
   cd mw/
   git init
   git branch -m main
   git remote add origin https://github.com/edgaritis/marcwise-tracker.git
   git add .
   git commit -m "Initial commit"
   git push -u origin main
   ```
3. Repo → **Settings → Pages** → Source: Deploy from a branch → Branch: `main` / `(root)` → Save.
4. Wait ~1 min. Your PWA is live at:
   `https://edgaritis.github.io/marcwise-tracker/`

## Installing the PWA

- **Desktop (Chrome/Edge):** open the URL → address bar → install icon → Install.
- **iPhone (Safari):** open the URL → Share → Add to Home Screen.
- **Android (Chrome):** open the URL → menu → Install app.

## First sign-in

1. Open the app → click **Sign in to sync** → enter email → **Send code**.
2. Open the email from Supabase. **Long-press** the "Sign in" link → **Copy Link**.
3. Back in the app, paste the link into the box → **Sign in**.
4. Your 1,601 seed posts sync up to Supabase; all future changes sync automatically.
