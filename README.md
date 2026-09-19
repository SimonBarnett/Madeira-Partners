![Club Madeira icon](https://madeira-widget-bucket.s3.eu-west-2.amazonaws.com/icon-192.png)

# Club Madeira Partner Kit

Static HTML scaffolding and partner-facing integration docs for
[partner.clubmadeira.io](https://partner.clubmadeira.io/). Thin page shells load
widgets from the S3 widget CDN. This repository is **not** the AWS API, not
widget source of truth, and not Awin / Xero / mailbox ops.

| This repo **is** | This repo **is not** |
|------------------|----------------------|
| MIT-licensed HTML templates, `index.json`, `menu-config.json`, PWA assets | [SimonBarnett/AWS](https://github.com/SimonBarnett/AWS) (API, Lambda, RDS) |
| Partner-site examples that host on S3 / CloudFront / any static host | Widget JS (ships from S3; verify live files on the bucket) |
| Optional Wix checkout snippets under `Modules/WIX/` | Awin merchant-feed onboarding or Daily Awin Report ops |

Club Madeira is a weekends-mostly **affiliate club**: partners promote merchants;
commission comes from “buy from my club”. It is not a dump of catalogues without
promotion.

**Putting widgets on your own site** (catalogue snippet, CMS, CSS, menus, TTS):
**[docs/PARTNER_EMBEDS.md](docs/PARTNER_EMBEDS.md)**.

**API truth:** cite [SimonBarnett/AWS](https://github.com/SimonBarnett/AWS). Do
not invent endpoints. Invite / metrics / onboarding / delegate are documented in
[docs/REFERRALS_AND_INVITES.md](docs/REFERRALS_AND_INVITES.md).

---

## Live architecture

```
Partner browser
    │
    ▼
partner.clubmadeira.io  (CloudFront → static hosting from this kit)
    │  HTML shells, index.json, menu-config.json, manifest.json, sw.js
    │
    ├─► https://madeira-widget-bucket.s3.eu-west-2.amazonaws.com/
    │     header, login, signup, api, category, chart, metrics,
    │     partner, user, catalog-preview, merchant-parts, set-token,
    │     madeira-extension, role, clubs, delegate, network, stripe, …
    │
    └─► https://ytepcnwske.execute-api.eu-west-2.amazonaws.com
          (referenced in category.html; other widgets call the same API —
          verify paths in SimonBarnett/AWS)
                │
                ▼
          RDS Users.referrer, SystemOTPs, Stripe Express, …
```

| Domain / host | Role |
|---------------|------|
| partner.clubmadeira.io | This kit (CloudFront) |
| madeira-widget-bucket (S3 eu-west-2) | Widget CDN |
| ytepcnwske.execute-api.eu-west-2.amazonaws.com | API Gateway |
| stripe.clubmadeira.io | Wix checkout post (`Modules/WIX/`) |
| clubmadeira.uk / thesmartcatalogue.com | Wix marketing |
| smartcatalogue.uk / clubmadeira.io | IONOS |

There is no `sim.html` / lesson game in this kit today — do not invent one.

---

## Page inventory

Every HTML file in this repository (`git ls-files '*.html'`). Filenames are the
live names: **`apikey.html`**, not `api-keys.html`.

| File | Role | `data-requireToken` | Primary widget / notes |
|------|------|---------------------|------------------------|
| `index.html` | Public home | `false` | `header-widget.js`, `role-widget.js` |
| `login.html` | Login | `false` | `login-widget.js` |
| `signup.html` | Signup (uses `affiliateCode`) | `false` | `signup-widget.js` |
| `dashboard.html` | Metrics + charts | `true` | `metrics-widget.js`, `chart-widget.js` |
| `apikey.html` | API keys (Awin etc.) | `true` | `api-widget.js` — **not** invite tracking |
| `catalog.html` | Embed / catalog preview | `true` | `catalog-preview-widget.js` |
| `category.html` | Smart Catalogue / Affiliate AI | `true` | `category-widget.js` → API Gateway host above |
| `partner.html` | **Partner Profile** (invite UI) | `true` | `partner-widget.js` → `POST /ui/invite` |
| `account.html` | **Account Settings** (live menu) | `true` | `delegate-widget.js`, `stripe-widget.js`, `reset-catalogue.js`, `gdpr-delete.js` |
| `delegate.html` | Older account-handoff shell | `true` | `user-widget.js` — header fallback if `menu-config.json` is missing |
| `clubs.html` | Partner/admin “My Clubs” | `true` | `clubs-widget.js` (can mount partner-widget to invite) |
| `madeira-clubs.html` | Public clubs gallery | `false` | `network-widget.js` `data-type="clubs"` |
| `madeira-merchants.html` | Public merchant network | `false` | `network-widget.js` `data-type="merchants"` |
| `madeira-partners.html` | Public partner network | `false` | `network-widget.js` `data-type="partners"` |
| `madeira-vouchers.html` | Public vouchers / deals | `false` | `network-widget.js` `data-type="discount"` |
| `parts.html` | Merchant parts | `true` | `merchant-parts.js` |
| `extension.html` | Extension / store badges | n/a | `madeira-extension.js` |
| `set-token.html` | JWT handoff redirect | n/a | `set-token.js` |
| `speech.html` | Local TTS demo (not kit core) | n/a | Inline only |
| `footer.html` | Shared footer fragment | — | Fetched by most pages |

Optional ecommerce attribution (not kit core): `Modules/WIX/wix_checkoutData.js`,
`Modules/WIX/wix_onOrderPaymentStatusUpdated.js` — post to `stripe.clubmadeira.io`.

---

## Configuration files (already in the repo)

Do **not** create `index.json`, `menu-config.json`, `footer.html`,
`manifest.json`, `sw.js`, `css/page.css`, or `apikey.html` from scratch. Edit
the copies that ship here.

### `index.json`

Live schema (values on `partner.clubmadeira.io` are public examples; replace
`affiliateCode` with your own):

| Key | Purpose |
|-----|---------|
| `affiliateCode` | Signup-widget attribution for this partner site. **Not** `/ui/invite`. |
| `sandbox` | When `true`, widgets/API may log extra sandbox behaviour. Keep documented; live currently ships `true`. |
| `loginUrl` | Redirect target when `data-requireToken="true"` and there is no JWT |
| `signupLinkUrl` | “Sign up” link used by the login widget |
| `dashboardLinkUrl` | Post-login dashboard path |
| `tts-voice` | Widget TTS voice (live: `eve`) |
| `tts-language` | Widget TTS language (live: `en`) |

### `menu-config.json`

Drives the header widget. Live Account is **`/account.html`**, partner Clubs is
**`/clubs.html`**, public galleries are `madeira-*.html`, API Keys is
**`/apikey.html`**. If this file is missing, the header widget still has a
built-in default that points Account at `/delegate.html` — keep both pages.

`roles` as used on live:

| `roles` value | Typical visibility |
|---------------|--------------------|
| `[]` | Everyone (Home, Install App) |
| `["notoken"]` | Signed-out only (public galleries, Login) |
| `["self"]` | Any signed-in user (Dashboard, API Keys, Logout) |
| `["community"]` / `["merchant"]` / `["partner"]` / `["admin"]` | Role-gated |

Exact role evaluation is in `header-widget.js` on S3, not in this repo.

---

## Attribution: four things that are not the same

| Mechanism | Use |
|-----------|-----|
| `affiliateCode` | Signup widget on **this site** attributes new merchant/community registrations to the site owner |
| `POST /ui/invite` + `Users.referrer` | Admin / partner / owner **invites** someone; completed users get `Users.referrer` = inviter `user_id` |
| `POST /ui/delegate` | **Account handoff** on an existing user (`account.html`) |
| Awin keys on `apikey.html` | Feed credentials for Daily Awin Report — **does not** record who invited whom |

Full citations: [docs/REFERRALS_AND_INVITES.md](docs/REFERRALS_AND_INVITES.md).

---

## Security and tokens

- Login stores a JWT in **`localStorage`** as `authToken`, plus `user_id` and
  `contact_name` (and related keys such as `lastlogin`). Treat the browser as
  the security boundary; XSS on a partner page can steal the token.
- Pages with `data-requireToken="true"` are gated by the header widget. If the
  token is missing, the widget redirects to `loginUrl` from `index.json`.
- `account.html` **requires** a token (live). `delegate.html` is aligned to the
  same rule even though it is only the menu fallback.
- `set-token.html` + `set-token.js` write the JWT after onboarding / login
  redirects, then send the browser on to the dashboard (or other return URL).
- `index.json` `sandbox: true` is a **documented live flag**, not a secret.
  Do not commit private API keys or Stripe secrets into this kit.
- `category.html` points at the current API Gateway host. If AWS rotates it,
  update that `data-api-endpoint` and verify in `SimonBarnett/AWS`.

---

## Hosting a partner site

These files already exist. Typical flow:

1. Clone this repository (or download the static files).
2. Set `affiliateCode` in `index.json` to **your** code. Leave `sandbox`
   documented; ask ops before flipping it on a production fork.
3. Optionally rebrand `index.html` copy, `css/page.css` header colours, and
   `footer.html`.
4. Upload the tree to any static host (S3 + CloudFront, Netlify, Vercel, IONOS,
   Fasthosts). Enable HTTPS.
5. Confirm `index.json`, `menu-config.json`, and `footer.html` are at the site
   root and publicly readable.

### Amazon S3 (short)

1. Create a bucket in `eu-west-2`, enable static website hosting, index document
   `index.html`.
2. Upload this repository’s files.
3. Allow public `s3:GetObject` on the objects you intend to serve (or put
   CloudFront in front and lock the bucket).
4. Point a hostname at CloudFront if you want HTTPS on a custom domain.

UK partners often use IONOS or Fasthosts for local support and GDPR; S3 is
fine for a low-maintenance static site.

### What to test

- Header menu matches `menu-config.json` (Account → `/account.html`, API Keys →
  `/apikey.html`).
- Public: `/`, `/login.html`, `/signup.html`, `/madeira-clubs.html`.
- Authenticated: `/dashboard.html`, `/partner.html`, `/account.html`,
  `/clubs.html`.
- `index.json` and `menu-config.json` fetch without 404.
- Protected pages redirect to `loginUrl` when signed out.
- Footer Source and MIT links still work.
- PWA: `manifest.json` `start_url` is `https://partner.clubmadeira.io/index.html`
  on the official site; forks should change it. `sw.js` caches the pages and
  root-level icons listed in that file (`android-chrome-*.png`, not
  `/images/icon-*.png`).

---

## Widget script map

Partner-facing install, CMS notes, CSS overrides, menus and TTS:
**[docs/PARTNER_EMBEDS.md](docs/PARTNER_EMBEDS.md)**.

All widget URLs use prefix
`https://madeira-widget-bucket.s3.eu-west-2.amazonaws.com/` unless noted.

| Page | Scripts |
|------|---------|
| index | `header-widget.js`, `role-widget.js` |
| login | `header-widget.js`, `login-widget.js` |
| signup | `header-widget.js`, `signup-widget.js` |
| dashboard | `header-widget.js`, `chart-widget.js`, `metrics-widget.js` |
| apikey | `header-widget.js`, `api-widget.js` |
| catalog | `header-widget.js`, `catalog-preview-widget.js` |
| category | `header-widget.js`, `category-widget.js` (`data-api-endpoint` = API Gateway) |
| partner | `header-widget.js`, `partner-widget.js` |
| account | `header-widget.js`, `delegate-widget.js`, `stripe-widget.js`, `reset-catalogue.js`, `gdpr-delete.js` |
| delegate | `header-widget.js`, `user-widget.js` |
| clubs | `header-widget.js`, `clubs-widget.js` |
| madeira-* | `header-widget.js`, `network-widget.js` |
| parts | `header-widget.js`, `merchant-parts.js` |
| set-token | `set-token.js` |
| extension | `madeira-extension.js` |
| speech | inline only |

Chrome extension (live `extension.html`): Web Store id
`ilnlmljfigjdlfppgnkffmlpmpdaiegc`. iOS app: App Store id `6751989113`.

---

## Who to ask

Product: Simon Barnett. Ops contacts: John Smith, Justine Greenfield.
Accountant: Mathew Forshaw. Stuart Coward is retired. This is a “who to ask”
list only.

---

## Support

- [docs/PARTNER_EMBEDS.md](docs/PARTNER_EMBEDS.md) to embed widgets on a partner
  or club site (including `madeira-widget.js`).
- [docs/REFERRALS_AND_INVITES.md](docs/REFERRALS_AND_INVITES.md) for invite vs
  affiliate vs delegate vs Awin.
- [SimonBarnett/AWS](https://github.com/SimonBarnett/AWS) for API / RDS / Lambda.
- Email [support@clubmadeira.io](mailto:support@clubmadeira.io).
- [X / clubmadeira](https://x.com/clubmadeira).

---

**Licence:** MIT — see [LICENSE](LICENSE).  
**Copyright © 2025 Simon Barnett.** Software remains Simon Barnett’s personal
copyright; the company operates the product under licence. MIT applies to the
partner-kit HTML in this repository.
