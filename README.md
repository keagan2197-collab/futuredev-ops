# FutureDev Ops

Property maintenance ticketing system for FutureDev Properties: a WhatsApp chatbot (SendPulse) that
verifies tenants and logs maintenance requests into a Google Sheet, plus a staff dashboard for
managing tickets, tenants, and team access.

## What's in this repo

```
docs/       The dashboard (static HTML/JS). GitHub Pages serves this folder — see Hosting below.
backend/    Code.gs — the Google Apps Script backend (Google Sheets database + SendPulse API proxy).
```

## Architecture

```
WhatsApp tenant  --> SendPulse chatbot flow  --> Apps Script Web App (backend/Code.gs)
                                                        |
                                                        v
                                          Google Sheet (tenant data, tickets, users, audit log)
                                                        ^
                                                        |
                                          Dashboard (docs/index.html)  <-- staff login
```

- **Tenant data** lives in your existing per-property tabs (135 Coleraine, 146 Coleraine, etc.) —
  never re-entered. `Code.gs` reads them directly and also builds a consolidated `AllTenants` tab
  for fast phone-number lookups.
- **Tickets, Users, AuditLog, Media** tabs are created automatically by `setupSheets()`.
- **Authentication** is lightweight (SHA-256 + server-side pepper, session tokens via CacheService)
  — good for a small internal team, not hardened enterprise auth. See the security note at the top
  of `backend/Code.gs`.

## Setup

1. Open your FutureDev Google Sheet → **Extensions → Apps Script**.
2. Paste in `backend/Code.gs`.
3. **Project Settings → Script Properties**, add:
   - `SHARED_SECRET` — long random string (webhook auth token)
   - `SENDPULSE_API_KEY` — SendPulse → Settings → API → API keys → Generate
   - `SENDPULSE_BOT_ID` — your WhatsApp bot ID (from its URL in SendPulse)
   - `PASSWORD_PEPPER` — a second long random string (dashboard password hashing)
4. Run `setupSheets()` once (Run menu).
5. Edit `setupFirstAdmin()` with your real email/password, run it once, then delete the password
   from the source afterward.
6. **Deploy → New deployment → Web app** (Execute as: Me, Access: Anyone). Copy the `/exec` URL.
7. In SendPulse's flow builder, add API Request nodes calling that URL for tenant verification and
   ticket creation (see the flow build guide — ask for it if you don't have it).
8. Open `docs/index.html` (or your GitHub Pages URL), click **Connect Google Sheet**, paste the
   `/exec` URL and your `SHARED_SECRET`.

## Hosting the dashboard on a domain (GitHub Pages)

1. Repo **Settings → Pages** → Source: **Deploy from a branch** → Branch: `main`, folder: `/docs` → Save.
2. GitHub gives you a URL like `https://yourusername.github.io/futuredev-ops/`.
3. To use your own domain: add a `CNAME` record at your DNS provider pointing your subdomain
   (e.g. `dashboard.futuredevproperties.co.za`) to `yourusername.github.io`, then add that hostname
   in the Pages settings.

**Important:** GitHub Pages on the free tier requires a **public repo**. Since `docs/index.html`
does not contain your webhook URL or secret (see below), that's safe — but never commit real
credentials into this file. Each person connects their own copy via the "Connect Google Sheet"
dialog in the dashboard, entering the URL/token client-side only (kept in memory, not saved to disk).

## Known gaps (see comments in `backend/Code.gs` for detail)

- Tenant verification is phone-number-only today (no Email/Active-Inactive column on the source
  tabs yet).
- "Other Managed" properties without a numeric complex number get a placeholder code.
- The SendPulse visual flow still needs the API Request nodes wired into the live message path —
  see the build guide.

## License

Private/internal project for FutureDev Properties. Not licensed for external use.
