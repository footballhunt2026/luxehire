# Luxehire Car Rental — Setup Guide

Real backend now. No more localStorage. Plain Node.js — zero npm packages,
zero build step. Just `node server.js` and it runs.

## 1. What changed from the old version

- Fleet, settings, hero images, promo cars — all live on the server now
  (`server/data/*.json`), not in the visitor's browser.
- Admin login is real server-side auth (session cookie + CSRF token), not a
  password hash sitting in your JS.
- New admin tabs: **Customer Hero Images** and **Promo Cars** — both go
  live on the real homepage the second you add them.
- Icon overflow bug fixed, trust badges redesigned, FAQ accordion fixed
  (only one open at a time, height animates correctly for any answer
  length), skeleton loading while the fleet fetches, scroll-in animations
  site-wide, "Admin" link removed from the public footer.

## 2. Run it locally

Requires Node.js 18+ (you have it if `node -v` prints something).

```bash
cd luxehire
cp .env.example .env
```

Open `.env`, set a real admin password:

```
PORT=3000
ADMIN_PASSWORD=YourStrongPasswordHere!
```

Then:

```bash
node server/server.js
```

Visit:
- Site: http://localhost:3000
- Admin: http://localhost:3000/admin/login.html

`ADMIN_PASSWORD` is only read the very first time (it creates the account).
If you skip it, the server generates a random password and prints it once
in the terminal — copy it immediately, you won't see it again.

## 3. Deploying it for real

This needs a host that runs Node.js processes — **not** plain static
hosting (no more GitHub Pages / Netlify-static / S3, since there's now a
real backend). Any of these work fine and have free or cheap tiers:

- Render.com (Web Service, Node)
- Railway.app
- A basic VPS (DigitalOcean, Hetzner, etc.) with `pm2` or `systemd`
- cPanel hosting with "Setup Node.js App" (common on GoDaddy/Namecheap)

General steps:
1. Upload the whole `luxehire/` folder.
2. Set the `ADMIN_PASSWORD` and `PORT` environment variables in your
   host's dashboard (don't upload `.env` itself if the host asks you to
   set env vars in a UI instead).
3. Start command: `node server/server.js` (or `npm start`).
4. Point your domain at it. **Use HTTPS** — cookies are set to `Secure`
   automatically once your host terminates SSL, so login just works once
   the domain has a certificate (most hosts above give you one free).

## 4. Where your data lives

Everything is JSON files under `server/data/`:
- `fleet.json`, `config.json`, `admin.json`, `hero-images.json`, `promo-cars.json`

Back this folder up regularly (or use the "Export Fleet" button in
Settings for a quick fleet-only backup). Uploaded photos live in
`public/uploads/`.

## 5. Admin panel — what's in it now

- **Overview** — quick stats.
- **Manage Fleet / Add New Car** — same as before, but image fields now
  have an **Upload** button (sends a photo straight from your device) as
  well as the old "paste a URL" option.
- **Customer Hero Images** — upload photos of happy customers; they show
  in a "Happy Customers" section on the homepage. Leave it empty and the
  section just doesn't appear — nothing broken either way.
- **Promo Cars** — pick any car already in your fleet, give it a badge
  ("Weekend Deal", "Flash Sale", etc.), and it shows in a highlighted
  "This week's promo cars" section with live pricing. Same — empty list,
  section hides itself.
- **Settings** — WhatsApp number, email, address, and a real password
  change (checks your current password server-side).

## 6. Security notes

- Sessions are server-side, 1 hour idle timeout, `HttpOnly` + `SameSite=Strict`
  cookies — can't be read or stolen by injected JS.
- Every admin write requires a CSRF token issued at login.
- Login attempts are rate-limited (8 tries / 15 min per IP).
- Passwords are hashed with scrypt — never stored in plain text, never
  sent to the browser.
- `/admin/*` pages are marked `noindex` and never cached.

Nothing here needs `npm install` — check `package.json`, dependency list
is empty on purpose so this is easy to run on cheap hosting.
