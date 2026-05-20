# Reunion by Felix — Drop 001 Setup Progress

## What's Been Built

### Landing Page (`index.html`)
- Pre-drop state: email + optional SMS/phone waitlist form
- Live drop state: Shopify Buy Button (headless) + secondary "future drops" email form
- Drop state controlled by a single config flag (`DROP_LIVE`)
- Shopify JS Buy SDK wired in — fetches default product variant automatically
- Phone field added with SMS consent note
- Both forms post to `https://reunion-command-center.vercel.app/api/email-signup`

### Config Block (index.html line ~411)
Fill these in before going live:
```js
const DROP_LIVE = false;                             // flip to true on drop day
const SHOPIFY_DOMAIN = 'YOUR-STORE.myshopify.com';  // e.g. reunionbyfelix.myshopify.com
const SHOPIFY_STOREFRONT_TOKEN = 'YOUR-STOREFRONT-ACCESS-TOKEN';
const PRODUCT_ID = 'gid://shopify/Product/YOUR-PRODUCT-ID';
const VINYL_PRICE = '$28';                           // display only
```

---

## Checklist

### Part 1 — Shopify Setup
- [ ] Log into existing Shopify store (repurposing merch store)
- [ ] Create vinyl product: `Did The Rain Stop? — Vinyl`
  - Set price
  - Set inventory count, enable tracking
  - Leave as **Draft** until drop day
- [ ] Get Product ID from URL: `admin.shopify.com/store/YOUR-STORE/products/XXXXXXX`
  - Wrap as: `gid://shopify/Product/XXXXXXX`
- [ ] Create Storefront API token
  - Settings → Apps and sales channels → Develop apps → Create app
  - Enable scopes: `unauthenticated_read_product_listings`, `unauthenticated_write_checkouts`, `unauthenticated_read_checkouts`
  - Install app → API credentials → copy Storefront API access token
- [ ] Confirm store domain (browser URL bar while in admin: `admin.shopify.com/store/STORE-NAME`)
  - Domain is: `STORE-NAME.myshopify.com`

### Part 2 — Landing Page Config
- [ ] Fill in `SHOPIFY_DOMAIN` in index.html line ~411
- [ ] Fill in `SHOPIFY_STOREFRONT_TOKEN`
- [ ] Fill in `PRODUCT_ID`
- [ ] Update `VINYL_PRICE` if not $28
- [ ] Deploy updated index.html

### Part 3 — Backend Update
- [ ] Supabase: add `phone` column (type: text, nullable) to waitlist table
- [ ] `reunion-command-center` repo: update `/api/email-signup` handler to read and store `phone`
  ```js
  const { email, phone, source } = req.body;
  await db.from('waitlist').insert({ email, phone: phone || null, source });
  ```

### Part 4 — Klaviyo
- [x] Create account at klaviyo.com
- [x] Branding configured (colors, sender: Felix Ames / felix@reunionbyfelix.com)
- [x] Reunion Waitlist created (List ID: S6a3iz)
- [x] Past Buyers list created (List ID: QSQWfE) — imported from old Shopify merch store
- [x] Existing waitlist imported from Supabase CSV
- [x] Auto-sync wired in reunion-command-center /api/email-signup → pushes new signups to Klaviyo automatically
- [ ] Deploy reunion-command-center to Vercel (activates the auto-sync)
- [ ] Add phone column to email_signups table in Supabase
- [ ] Connect to Shopify (Integrations → Shopify) — do when Shopify is active
- [ ] Build 3 flows:
  - Order confirmation (ships in 2–3 weeks messaging)
  - Shipping notification (tracking link)
  - Post-purchase day 30 (tease next drop)
- [ ] Draft drop announcement campaign to "Reunion Waitlist" — send 24hr before drop

### Part 5 — Shipping
- [ ] Get USPS PO Box (~$50–200/yr) — use as return address so home address never appears on labels
- [ ] Create PirateShip account at pirateship.com (free)
  - Enter PO Box as return address
  - Connect to Shopify (Settings → Integrations → Shopify)
- [ ] Order 12" corrugated LP mailers + "Do Not Bend" labels (Uline or Amazon)
- [ ] Set shipping rates in Shopify (Settings → Shipping and delivery)
  - Suggested: $6 domestic / $18 international — or build into price + offer free shipping

### Part 6 — Drop Day
- [ ] Publish vinyl product in Shopify (Draft → Active)
- [ ] Confirm inventory count
- [ ] Set `DROP_LIVE = true` in index.html
- [ ] Deploy landing page
- [ ] Connect Shopify to Spotify for Artists (Spotify for Artists → Merch → Connect store)
- [ ] Send Klaviyo campaign to waitlist
- [ ] Post on socials

---

## Key Decisions Made
- Using **headless Shopify** — Shopify runs as backend only, landing page is the storefront
- Using **Shopify Buy SDK** (not embedded iframe) for full styling control
- Checkout happens on Shopify's hosted checkout (checkout.shopify.com) — this is fine and expected
- Repurposing existing Shopify merch store — no new store needed
- Self-fulfilling with **PirateShip + USPS** for 100–300 unit run
- Return address will be a **USPS PO Box**
- **Klaviyo** for email marketing (free tier covers current list size)
- No variants on product — code fetches default variant automatically via product ID

## Stack
- Frontend: static HTML/CSS/JS — hosted on Vercel (or wherever landing page lives)
- Email collection backend: `reunion-command-center` (Vercel) → Supabase
- E-commerce: Shopify (inventory, checkout, order management)
- Email marketing: Klaviyo
- Shipping labels: PirateShip
- SMS marketing: TBD (Klaviyo SMS or Postscript once list grows)
