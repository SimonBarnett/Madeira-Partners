# Referrals, invites, account handoff, and Awin keys

This document is the partner-kit view of four **different** mechanisms. Do not
treat them as one “referral” system. API behaviour lives in
[SimonBarnett/AWS](https://github.com/SimonBarnett/AWS) — this kit does not
invent extra endpoints.

| Concern | What it is | What it is not | Source of truth |
|---------|------------|----------------|-----------------|
| `index.json` `affiliateCode` | Signup-widget **attribution** for merchant/community signups that happen on a partner site (also appended to Affiliate AI URLs such as `/category.html?affiliate=…`) | Not the invited-user tree. Not account handoff. Not an Awin publisher key. | This repo `index.json`; consumed by signup/login/header widgets |
| `POST /ui/invite` + `Users.referrer` | Partner/admin/owner **invite** to onboard a new user (Stripe Express + `SystemOTPs` `token_type = onboarding`) | Not `affiliateCode`. Not Daily Awin Report. | [API/routes/ui/invite.js](https://github.com/SimonBarnett/AWS/blob/main/API/routes/ui/invite.js), [API/routes/token/onboarding.js](https://github.com/SimonBarnett/AWS/blob/main/API/routes/token/onboarding.js) |
| `GET /ui/metrics` (partner cards) | Last-7-day merchant/community signups `WHERE referrer = @userId` | Does not count `affiliateCode` signups unless those users also have `Users.referrer` set | [API/routes/ui/metrics.js](https://github.com/SimonBarnett/AWS/blob/main/API/routes/ui/metrics.js) `fetchPartnerMetrics` |
| `POST /ui/delegate` | **Account handoff**: replace contact details on an **existing** user (`SystemOTPs` `token_type = delegation`) | Not an invite. Does not create a new `Users` row. | [API/routes/ui/index.js](https://github.com/SimonBarnett/AWS/blob/main/API/routes/ui/index.js) (`path.endsWith('/delegate')`) → [API/delegate.js](https://github.com/SimonBarnett/AWS/blob/main/API/delegate.js) |
| Awin Add Key (`apikey.html`) | Merchant/community **API keys** (Awin and other feeds) used by Daily Awin Report ops | Does **not** track who invited whom | [API/routes/ui/apiKeys.js](https://github.com/SimonBarnett/AWS/blob/main/API/routes/ui/apiKeys.js); out of scope for this kit |

Daily Awin Report emails track merchant onboard/product stats only. They do **not**
track who invited whom.

---

## 1. `affiliateCode` — partner-site signup attribution

`index.json` on this kit (and live `https://partner.clubmadeira.io/index.json`)
includes `affiliateCode`. The live example value is already public; partners
replace it with their own code.

- Login and signup widgets require a non-empty `affiliateCode` or they error.
- Header / Affiliate AI menu can append `?affiliate=<code>` to `/category.html`.
- This is **related to** but **distinct from** the `/ui/invite` referral tree.

There is no claim in this kit that `affiliateCode` writes `Users.referrer`.
Verify any signup-widget → RDS mapping in AWS before documenting further.

---

## 2. `POST /ui/invite` — invited onboarding tree

**Handler:** `SimonBarnett/AWS` → `API/routes/ui/invite.js`  
**Router:** `API/routes/ui/index.js` — `path.endsWith('/invite') && method === 'POST'`

Verified behaviour (read 2026-09-19):

- Requires a valid JWT (`decoded.user_id`).
- Caller must have `admin`, `partner`, or `owner` in `Users.permissions`.
- Non-admins who are not `owner` may only invite `tokenType === 'merchant'`.
- Body: `mobile`, `email`, `tokenType` (required); optional `url` / `communityId`.
- Rejects if the email already exists on `Users`, or a pending
  `SystemOTPs` row exists with `token_type = 'onboarding'` for that email.
- Creates a Stripe Express account.
- Writes `SystemOTPs` with `token_type = 'onboarding'`, 48-hour expiry, payload
  including `referrerId` = the inviter’s 8-character `user_id`, plus email,
  phone, `tokenType`, Stripe account id, signup URL, PIN, and JWT.
- Enqueues `SEND_EMAIL` / `emailType: 'onboarding'`.

**Partner UI that triggers invite**

- `partner.html` loads `partner-widget.js`, which POSTs
  `{apiEndpoint}/prod/ui/invite` with `email`, `mobile`, `tokenType`, optional
  `url` / `communityId`.
- `clubs.html` loads `clubs-widget.js`; its Invite button mounts
  `partner-widget` in a modal (same invite path).

---

## 3. Onboarding complete — `Users.referrer`

**Handler:** `SimonBarnett/AWS` → `API/routes/token/onboarding.js`

- Pending invites live in `SystemOTPs` where `token_type = 'onboarding'`.
- `handleComplete` loads that OTP, generates a new 8-character `user_id`, and
  `createUser` with `referrer: onboardingData.referrer_by`.
- `getOnboardingData` / `handleComplete` set `referrer_by` from
  `record.user_id` on the OTP row (the inviter who called `/ui/invite`).
- Completed users therefore have `Users.referrer` = inviter `user_id`.
- After create, the flow builds a `set-token` redirect (`buildSetTokenUrl`) so
  the browser lands on `set-token.html` with a JWT.

Exact Stripe field mapping and role-specific columns are in AWS; do not
re-specify them here.

---

## 4. `GET /ui/metrics` — partner signup cards

**Handler:** `SimonBarnett/AWS` → `API/routes/ui/metrics.js`  
**Page:** `dashboard.html` via `metrics-widget.js`

`fetchPartnerMetrics(pool, userId)`:

```sql
SELECT
  SUM(CASE WHEN role = 'merchant' THEN 1 ELSE 0 END) as merchantSignups,
  SUM(CASE WHEN role = 'community' THEN 1 ELSE 0 END) as communitySignups
FROM Users
WHERE referrer = @userId
  AND created_at >= DATEADD(day, -7, GETDATE())
```

Cards: “Merchant Signups (Last 7 Days)” and “Community Signups (Last 7 Days)”.
Admin metrics count the same roles for **all** users in the last 7 days (no
referrer filter).

---

## 5. `POST /ui/delegate` — account handoff, not invite

**Router:** `API/routes/ui/index.js` — `path.endsWith('/delegate') && method === 'POST'`  
**Handler:** `SimonBarnett/AWS` → `API/delegate.js` (required as `../../delegate`
from the UI router; **not** `API/routes/ui/delegate.js`)

Verified behaviour:

- `action: 'initiate'` (live Account page) takes `first_name`, `phone_number`,
  `email_address`. Writes `SystemOTPs` with `token_type = 'delegation'` and
  emails a delegation token/OTP.
- `action: 'accept'` updates the **existing** `Users` row (`first_name`,
  `email_address`, `phone_number`, `password`) for `delegatorId`. It does not
  create a new user and does not set `Users.referrer`.
- Self-delegation (same email) is allowed for testing.

**Pages**

| File | Live role | Token | Widgets |
|------|-----------|-------|---------|
| `account.html` | Menu **Account** (`/account.html`, roles `community`) | `data-requireToken="true"` | `delegate-widget.js` → `POST /ui/delegate`, plus Stripe, reset catalogue, GDPR delete |
| `delegate.html` | Header-widget **fallback** if `menu-config.json` is missing | `data-requireToken="true"` (aligned with live account security intent) | `user-widget.js` (older handoff shell) |

Keep both files. Do not treat `delegate.html` as the live Account page.

---

## 6. What this kit must not claim

- Awin Add Key / Daily Awin Report emails do **not** track inviters.
- `affiliateCode` is **not** `POST /ui/invite`.
- `/ui/delegate` is **not** an invite.
- Extra REST paths beyond invite / metrics / onboarding / delegate were not
  documented here; inspect AWS before adding any.
