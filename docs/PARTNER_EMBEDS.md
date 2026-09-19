# Partner embeds — use Club Madeira widgets on your own site

This guide is for partners who want Club Madeira widgets **on their own website**, not only those who host the full [partner kit](https://github.com/SimonBarnett/Madeira-Partners) at a root domain.

**API truth:** [SimonBarnett/AWS](https://github.com/SimonBarnett/AWS). Widget JS truth: live files on
`https://madeira-widget-bucket.s3.eu-west-2.amazonaws.com/`. This kit does not invent endpoints or voice names.

Invite vs `affiliateCode` vs account handoff vs Awin keys:
[REFERRALS_AND_INVITES.md](REFERRALS_AND_INVITES.md).

---

## What most club / merchant sites actually need

The catalogue members see on a **club or merchant site** is **not** the partner-kit header. It is `madeira-widget.js`. The signed-in **Embed Code** page (`catalog.html`) builds this snippet for you:

```html
<div id="madeira-container"></div>
<script
  data-affiliate="YOUR_8_CHAR_USER_ID"
  data-css="madeira-widget.css"
  src="https://madeira-widget-bucket.s3.eu-west-2.amazonaws.com/madeira-widget.js?v=1.0"></script>
```

- `data-affiliate` is required (live widget errors if missing). On the Embed Code page it is the signed-in `user_id` from `localStorage`, **not** `index.json` `affiliateCode`.
- `data-css` is optional; default `madeira-widget.css`. Other styles the preview offers: `madeira-dark.css`, `madeira-transp.css` (same S3 prefix).
- The widget calls the public catalogue route `GET …/prod/query` ([AWS API README](https://github.com/SimonBarnett/AWS/blob/main/API/README.md) — `/rds` and `/query`).

`index.json` `affiliateCode` is **signup-widget attribution** on a partner login/signup page. It is not this `data-affiliate` catalogue tag, not `POST /ui/invite`, and not `/ui/delegate`.

---

## CDN and hosting rules

| Rule | Why |
|------|-----|
| Serve your pages over **HTTPS** | Widgets call HTTPS API Gateway; mixed content is blocked |
| Host `index.json` at **your site root** (`https://your-domain/index.json`) | Login, signup, header, set-token and audiotour fetch `/index.json` same-origin |
| Host `menu-config.json` at the root **if** you use `header-widget.js` | Header fetches `/menu-config.json`; missing file → built-in default (Account → `/delegate.html`) |
| Do not copy widget JS into your CMS | Always load from the S3 CDN so you stay on the live widget |
| `localStorage` JWT (`authToken`) | Same browser origin only. A widget on `club.example` cannot see a token set on `partner.clubmadeira.io` |

CORS: browser `fetch('/index.json')` is same-origin. Cross-origin **script** tags to the S3 CDN are normal. API CORS is handled in [SimonBarnett/AWS](https://github.com/SimonBarnett/AWS) `API/index.js` (orchestrator). You do not set CORS on the widget bucket for `<script src>`.

There is **no** documented public iframe URL for the catalogue. `catalog-preview-widget.js` uses an iframe only for the **preview** on `catalog.html`. Do not invent an iframe embed.

---

## 1. Widget inventory

CDN prefix for every script below:

`https://madeira-widget-bucket.s3.eu-west-2.amazonaws.com/`

`data-requireToken` is read **only by `header-widget.js`**. Other widgets check `localStorage.authToken` themselves or assume a gated page.

### Partner / club site (typical)

#### `madeira-widget.js` — Smart Catalogue on *your* site

| | |
|--|--|
| **Purpose** | Public catalogue / parts browser for a club or merchant site |
| **Auth** | Public (`GET /prod/query`). No header token gate |
| **Required** | `data-affiliate` on the `<script>` (club/merchant `user_id`) |
| **Optional** | `data-css` (`madeira-widget.css` default; also `madeira-dark.css`, `madeira-transp.css`) |
| **Mount** | `<div id="madeira-container"></div>` plus the script (see snippet above) |
| **Their site?** | **Yes — this is the main embed** |
| **Kit only?** | Generated on `catalog.html`; also shown in `clubs-widget` copy-snippet UI |

#### `network-widget.js` — public galleries

| | |
|--|--|
| **Purpose** | Public clubs / merchants / partners / vouchers grids |
| **Auth** | Public |
| **Required** | Mount `data-network-widget`; `data-type` = `clubs` \| `merchants` \| `partners` \| `discount` (default `clubs`) |
| **Optional** | `data-affiliate` on the **script** tag (voucher click-ref; live vouchers page uses a public example) |
| **Kit pages** | `madeira-clubs.html`, `madeira-merchants.html`, `madeira-partners.html`, `madeira-vouchers.html` |
| **Their site?** | Yes, if you want the same public galleries |
| **Kit only?** | No |

```html
<div data-network-widget data-type="merchants"></div>
<script src="https://madeira-widget-bucket.s3.eu-west-2.amazonaws.com/network-widget.js"></script>
```

#### `madeira-extension.js` — store badge

| | |
|--|--|
| **Purpose** | Chrome Web Store or App Store badge from user-agent |
| **Auth** | None |
| **Optional** | `data-width` (default `180px`; kit uses `200px`) |
| **Kit page** | `extension.html` |
| **Their site?** | Yes, as a download badge |
| **Kit only?** | No |

```html
<script src="https://madeira-widget-bucket.s3.eu-west-2.amazonaws.com/madeira-extension.js" data-width="200px"></script>
```

### Partner-kit chrome (header, login, home)

#### `header-widget.js`

| | |
|--|--|
| **Purpose** | Fixed header, role-gated menu, PWA install, mute button, optional login redirect |
| **Auth** | `data-requireToken="true"` redirects to `loginUrl` from `/index.json` if no JWT |
| **Required** | `data-header-widget` on `<header>` |
| **Optional** | `data-page-name` (logo text; default `Dashboard`), `data-icon` (Font Awesome class; default `fas fa-home`), `data-requireToken` (`true` / `false`) |
| **Also fetches** | `/index.json` (`affiliateCode`, URLs), `/menu-config.json` |
| **Their site?** | Only if you want the Club Madeira header/menu on that origin |
| **Kit only?** | Typical on the hosted kit |

```html
<header data-header-widget data-page-name="Welcome" data-requireToken="false"></header>
<script src="https://madeira-widget-bucket.s3.eu-west-2.amazonaws.com/header-widget.js"></script>
```

#### `role-widget.js`

| | |
|--|--|
| **Purpose** | Cycles Community / Admin / Merchant / Partner explainer cards |
| **Auth** | Public |
| **Required** | Container `#role-widget` (no `data-*`) |
| **Kit page** | `index.html` |
| **Their site?** | Optional marketing |
| **Kit only?** | Mostly kit home |

```html
<div id="role-widget"></div>
<script src="https://madeira-widget-bucket.s3.eu-west-2.amazonaws.com/role-widget.js"></script>
```

#### `login-widget.js`

| | |
|--|--|
| **Purpose** | Login, forgot password, OTP |
| **Auth** | Public page; stores `authToken` in `localStorage` |
| **Required** | `data-login-widget` and `data-container-id` on the **script**; matching container id; non-empty `affiliateCode` in `/index.json` |
| **Kit page** | `login.html` |
| **Their site?** | Yes, if you host login on your domain (same origin as later gated pages) |
| **Kit only?** | No |

```html
<div id="login-widget"></div>
<script data-login-widget data-container-id="login-widget"
  src="https://madeira-widget-bucket.s3.eu-west-2.amazonaws.com/login-widget.js"></script>
```

#### `signup-widget.js`

| | |
|--|--|
| **Purpose** | Merchant / community self-serve signup |
| **Auth** | Public; uses `affiliateCode` from `/index.json` |
| **Required** | `data-signup-widget`, `data-container-id` on the script; `affiliateCode` in `/index.json` |
| **Kit page** | `signup.html` |
| **Their site?** | Yes, for **affiliateCode attribution** (not `/ui/invite`) |
| **Kit only?** | No |

```html
<div id="signup-widget"></div>
<script data-signup-widget data-container-id="signup-widget"
  src="https://madeira-widget-bucket.s3.eu-west-2.amazonaws.com/signup-widget.js"></script>
```

#### `set-token.js`

| | |
|--|--|
| **Purpose** | After onboarding/login redirect: write JWT query params into `localStorage`, then go to `signupLinkUrl` |
| **Auth** | Consumes `authToken`, `user_id`, `contact_name` (and optional `sandbox`, `lastlogin`, `signup_url`) from the query string |
| **Required** | Page with `#redirectMessage` / `#fallbackLink` as in `set-token.html` |
| **Kit page** | `set-token.html` |
| **Their site?** | Only if AWS onboarding `buildSetTokenUrl` points at **your** origin |
| **Kit only?** | Usually kit |

### Signed-in kit pages (rarely copied onto a marketing CMS)

These expect a JWT in `localStorage` on **this origin**. Put them behind `header` `data-requireToken="true"` or users will see errors / redirects.

| Widget | Purpose | Mount | Auth / API (AWS) | Their site? |
|--------|---------|-------|------------------|-------------|
| `metrics-widget.js` | Dashboard cards | `#metrics-widget` | `GET /prod/ui/metrics` — [metrics.js](https://github.com/SimonBarnett/AWS/blob/main/API/routes/ui/metrics.js) | Kit dashboard |
| `chart-widget.js` | Performance charts | `#madeira-charts` | `GET /prod/ui/chart-data` | Kit dashboard |
| `api-widget.js` | Awin / feed **API keys** (not invites) | `#api-keys-container` | `…/ui/api-keys` — [apiKeys.js](https://github.com/SimonBarnett/AWS/blob/main/API/routes/ui/apiKeys.js) | Kit `apikey.html` |
| `catalog-preview-widget.js` | Embed-code builder + CMS instructions | `#catalog-preview-widget` | `GET /prod/ui/cms-providers` — [cmsProviders.js](https://github.com/SimonBarnett/AWS/blob/main/API/routes/ui/cmsProviders.js) | Kit `catalog.html` (use this, then paste `madeira-widget.js` on the club site) |
| `category-widget.js` | Smart Catalogue / Affiliate AI | `#categories-widget` | Live JS hardcodes `…/prod` and `#categories-widget`. Kit HTML still has `data-categories-widget`, `data-api-endpoint`, `data-container-id` — **live constructor does not read those attributes** (verified S3 2026-09-19). | Kit `category.html` |
| `partner-widget.js` | **Invite** UI (`POST /prod/ui/invite`) | `[data-partner-widget]` | [invite.js](https://github.com/SimonBarnett/AWS/blob/main/API/routes/ui/invite.js) — **not** `affiliateCode` | Kit `partner.html`; also opened from clubs Invite modal |
| `clubs-widget.js` | Partner/admin club list; can mount partner-widget | `#clubs-widget` | Live JS calls `GET …/prod/ui/clubs` — **confirm the handler in SimonBarnett/AWS** (not listed on the UI router file read 2026-09-19) | Kit `clubs.html` |
| `delegate-widget.js` | Account **handoff** (`POST /ui/delegate`) | `[data-delegate-widget]` | [delegate.js](https://github.com/SimonBarnett/AWS/blob/main/API/delegate.js) — **not** invite | Kit `account.html` |
| `user-widget.js` | Older user/handoff shell | `[data-user-widget]` | Header fallback page `delegate.html` | Kit only unless you still use `/delegate.html` |
| `stripe-widget.js` | Reconnect Stripe Express | `[data-stripe-widget]` | `POST /ui/stripe` — [stripe.js](https://github.com/SimonBarnett/AWS/blob/main/API/routes/ui/stripe.js) | Kit `account.html` |
| `reset-catalogue.js` | Reset catalogue (OTP) | `[data-reset-catalogue]` | `POST /ui/reset` `{action: initiate\|confirm}` — [reset.js](https://github.com/SimonBarnett/AWS/blob/main/API/routes/ui/reset.js) | Kit `account.html` |
| `gdpr-delete.js` | GDPR account delete | `[data-gdpr-delete]` | `POST /ui/delete` then `/ui/deleteconfirm` — [delete.js](https://github.com/SimonBarnett/AWS/blob/main/API/routes/ui/delete.js) | Kit `account.html` |
| `merchant-parts.js` | Merchant parts table | Script’s **parent** element (kit uses `#merchant-parts-widget`; live JS does not read `data-container-id`) | `GET /prod/ui/merchant-parts` — [merchantParts.js](https://github.com/SimonBarnett/AWS/blob/main/API/routes/ui/merchantParts.js) | Kit `parts.html` |

Example (account settings — kit only):

```html
<header data-header-widget data-page-name="Account Settings" data-requireToken="true"></header>
<script src="https://madeira-widget-bucket.s3.eu-west-2.amazonaws.com/header-widget.js"></script>
<div data-delegate-widget></div>
<script src="https://madeira-widget-bucket.s3.eu-west-2.amazonaws.com/delegate-widget.js?v=1.1"></script>
```

### Loaded as dependencies (do not embed alone unless you know why)

| Script | Used by | Notes |
|--------|---------|--------|
| `audiotour.js` | metrics, charts, catalog-preview, stripe, reset-catalogue, gdpr-delete | Reads `/index.json` `tts-language` and `tts-voice`. See [Voice / TTS](#5-voice--tts) |
| `tagTracker.js` | `merchant-parts.js` | Loaded by that widget; not a partner mount point |
| Font Awesome (cdnjs) | Most widgets inject it if missing | Do not override `.fa-*` display in a way that hides icons |

`speech.html` is a **local browser TTS demo**, not a CDN widget.

---

## 2. Install embeds

### Developers / static HTML

1. HTTPS host (S3+CloudFront, Netlify, IONOS, Fasthosts, …).
2. For **catalogue on a club site**, paste the `madeira-widget.js` snippet. You do **not** need the full kit.
3. For **login/signup/header on your domain**, copy the matching kit page (or the snippets above) and put `index.json` at the **root of that same host**:

```json
{
  "affiliateCode": "YOUR_AFFILIATE_CODE",
  "sandbox": true,
  "loginUrl": "/login.html",
  "signupLinkUrl": "/signup.html",
  "dashboardLinkUrl": "/dashboard.html",
  "tts-voice": "eve",
  "tts-language": "en"
}
```

Replace `affiliateCode` with the code Club Madeira gave you (live kit example `L7WDZWC8` is already public). That code attributes **self-serve signup**, not `/ui/invite`.

4. If you use `header-widget.js`, also host `menu-config.json` at the root (or accept the built-in default).
5. Confirm in DevTools: `/index.json` is 200, no mixed-content errors, S3 scripts load.

### Common CMS (no / low code)

Signed-in **Embed Code** (`catalog.html`) is the supported place for CMS-specific write-ups. It loads provider names and markdown from `GET /ui/cms-providers` ([cmsProviders.js](https://github.com/SimonBarnett/AWS/blob/main/API/routes/ui/cmsProviders.js) reads `dbo.cmsProvider` / `cmsDocLinks`). This kit does **not** copy those CMS names from the database — open Embed Code while logged in.

Practical limits (product does **not** ship a public iframe embed):

| CMS | How to add the catalogue snippet | `index.json` / `menu-config.json` | Limits |
|-----|----------------------------------|-----------------------------------|--------|
| **Wix** | Editor → Embed / Custom HTML (or Velo). Paste the `madeira-widget.js` snippet from Embed Code. | Wix does not serve arbitrary files at `/index.json` unless you host them on a real static origin. Header/login widgets that `fetch('/index.json')` **will not work** on a typical Wix page URL. | Custom HTML may run in a sandboxed iframe; test the catalogue widget. This repo’s `Modules/WIX/` files are **checkout attribution** to `stripe.clubmadeira.io`, not the catalogue embed. |
| **WordPress** | Custom HTML block, or a theme header script. Administrators can usually save `<script>`; Contributors often cannot. | Put JSON files on the same origin (theme, plugin, or a static subdomain). A page at `/blog/shop/` still makes widgets request `https://yoursite/index.json` (site root), not the page path. | Some security plugins strip scripts. No kit iframe fallback. |
| **Squarespace** | Code Block or Settings → Advanced → Code Injection (plan-dependent). | Same root-JSON rule. Marketing templates often cannot host `/menu-config.json`. Use catalogue-only embed. | Script injection is commonly Business plans and above. |
| **Shopify** | `theme.liquid` or a Custom Liquid section. | Shopify’s storefront origin is not a general static file host for `/index.json` unless you add one (e.g. files app + proxy, or a separate static host). Catalogue snippet does **not** need `index.json`. | Theme edits require a published theme. Online Store 2.0 app blocks may strip scripts — use theme Liquid if the block refuses `<script>`. |

**Rule of thumb:** catalogue (`madeira-widget.js`) belongs on the club/merchant CMS. Header, login, invite, account, and dashboard widgets belong on a static origin you control (this kit), where `/index.json` and `/menu-config.json` are real files.

---

## 3. CSS customisation

Widgets inject their own `<style>` (header uses `#header-widget-styles`). You can override **after** those tags, or via `css/page.css` on the kit.

### `css/page.css` hooks (this repo)

| Selector | What partners usually change |
|----------|------------------------------|
| `header` | Background `#333`, text colour, padding. Header widget also forces `position: fixed` |
| `.logo`, `.logo i`, `.logo-text` | Title colour/size |
| `.menu-link`, `.menu-icon`, `.menu-text` | Link colour; hover in page.css is `#66b3ff` |
| `.menu-item.selected .menu-link` | Selected item |
| `.hamburger`, `.menu-list` | Mobile drawer (page.css breakpoint 768px; **header widget** mobile breakpoint is 1199px — they can disagree) |
| `main` | `padding`, `margin-top: 60px` (clear the fixed header), `padding-bottom` for the fixed footer |
| `.welcome` | Home copy column |
| `footer` | Bar colour; `footer.html` also inlines `#333` |
| `#api-keys-wrapper .provider-icon` | API key provider icons |

Example (kit header colours):

```css
header {
  background-color: #1b3d1a;
  color: #f4f1ea;
}
.menu-link:hover,
.menu-item.selected .menu-link,
footer a:hover {
  color: #c4e52a;
}
```

### Widget classes (do not break layout)

**Header** (from `header-widget.js` injectStyles): `.logo`, `.menu-group`, `.menu-list`, `.menu-item`, `.menu-link`, `.menu-icon`, `.menu-text`, `.hamburger`, `.mute-button`, `.mute-button-wrapper`, `.menu-hint`. Colours are mostly `inherit` so `header { color; background }` flows through.

**Role widget:** `.widget-container`, `.header`, `.header-content`, `.content` — scoped poorly (the script also sets global `body { font-family }`). Prefer wrapping `#role-widget` and avoiding a global `body` reset that fights it.

**Metrics:** `#metrics-widget`, `.metrics-container`, `.metric-card`, `.metrics-arrow`.

**Catalogue on a club site:** classes inside `madeira-widget.js` / the chosen `madeira-*.css`. Prefer `data-css` theme files over rewriting `#catalog-widget` internals.

**Do not**

- Set `i, .fa, .fas { display: none }` or `font-family` on `i` that replaces Font Awesome.
- Remove `main` top margin while the header is `position: fixed`.
- Override widget `position: fixed` overlays (loading / GDPR) without testing.

There are **no** documented CSS variables (`--madeira-*`) on the live header widget (S3 2026-09-19).

---

## 4. Menus (`menu-config.json`)

Fetched by `header-widget.js` from `/menu-config.json` on the **current origin**.

```json
{
  "menuItems": [
    { "name": "Home", "icon": "fas fa-home", "href": "/index.html", "roles": [] },
    { "name": "Account", "icon": "fas fa-user-gear", "href": "/account.html", "roles": ["community"] },
    { "name": "Install App", "icon": "fas fa-mobile-alt", "action": "install", "roles": [] }
  ]
}
```

| Field | Meaning |
|-------|---------|
| `name` | Label (and `data-name` on the link; Logout is detected by name) |
| `icon` | Font Awesome class |
| `href` | Page path **or** omit when using `action` |
| `action` | `install` — PWA install prompt (no `href`) |
| `roles` | Who sees the item (see below) |

Evaluation (live `header-widget.js`):

| `roles` | Shown when |
|---------|------------|
| `[]` | Always (Home, Install App) |
| `["notoken"]` | Signed **out** only |
| `["self"]` | Any signed-in user |
| `community` / `merchant` / `partner` / `admin` | Signed in **and** JWT `permissions` includes that role |

If the file is missing, empty, or not a `menuItems` array, the widget uses its **built-in default**. That default matches live galleries but points **Account → `/delegate.html`**. Live `menu-config.json` overrides Account to **`/account.html`**. Keep both HTML files if you host the kit.

**Add / remove / reorder:** edit the `menuItems` array order; hide an item by deleting it; add an item with a real `href` that exists on **that** host. Public `madeira-*.html` use `notoken`. Partner clubs UI is `/clubs.html` with `partner`+`admin`. Do not confuse that with public `/madeira-clubs.html`.

---

## 5. Voice / TTS

Two different systems:

### Audiotour (help button on several kit widgets)

`audiotour.js` (S3) reads `/index.json`:

- `tts-language` — live and code default: `"en"`
- `tts-voice` — live and code default: `"eve"`

It then loads:

`https://madeira-widget-audiotour.s3.eu-west-2.amazonaws.com/{language}/{voice}/{widgetName}-audiotour.json`

Verified 2026-09-19: `en/eve/metrics-widget-audiotour.json` exists. Header mute uses `localStorage.audioMuted`.

**No voice/language enumeration exists in [SimonBarnett/AWS](https://github.com/SimonBarnett/AWS)** (repo tree search 2026-09-19). Do not invent names such as extra ElevenLabs voices. To use another pair, confirm with [support@clubmadeira.io](mailto:support@clubmadeira.io) that the matching prefix exists on `madeira-widget-audiotour`; otherwise the help button stays hidden (missing JSON).

### Category widget spoken replies

`category-widget.js` uses the **browser** `speechSynthesis` API for Affiliate AI replies (plus an iOS “Read it / Quiet” prompt). That path does **not** read `tts-voice` / `tts-language` from `index.json` in the live file reviewed.

`speech.html` in this repo is a standalone demo of browser voices only.

---

## Attribution reminder

| Mechanism | Use on *your* site |
|-----------|-------------------|
| `madeira-widget.js` `data-affiliate` | Catalogue attribution = club/merchant `user_id` |
| `index.json` `affiliateCode` | Signup/login widgets on a partner origin |
| `POST /ui/invite` | Partner Profile / Clubs Invite — invited onboarding tree |
| `POST /ui/delegate` | Account Settings handoff |
| `apikey.html` / `api-widget.js` | Awin keys — not who invited whom |

---

## Support

- [README.md](../README.md) — kit inventory and hosting the full site  
- [REFERRALS_AND_INVITES.md](REFERRALS_AND_INVITES.md)  
- [SimonBarnett/AWS](https://github.com/SimonBarnett/AWS)  
- [support@clubmadeira.io](mailto:support@clubmadeira.io)
