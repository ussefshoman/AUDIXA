# Voice-Over Platform — M0 to M3

Next.js (App Router) + Supabase + Cloudflare R2 + Wavesurfer.js. This drop contains:

- **M0:** project setup, design tokens (monochrome, RTL/LTR, light/dark), Arabic + English routes (`/ar`, `/en`), Supabase schema (`supabase/migrations/0001_init.sql`).
- **M1:** portfolio player (Wavesurfer waveform, category tabs, one sample at a time) and the price calculator (flat $5/min, $5 minimum, flat rush fees, "Other / Custom" category).

- **M2:** email + Google login, order form (typed script or PDF/TXT/DOCX upload to private R2), "Other / Custom" carried into the order, server-side pricing.
- **M3:** Paymob (Egypt, EGP) and Stripe (international, USD) checkout, signed webhooks, idempotent payment confirmation.

- **M4:** client portal (`/ar/account`): status timeline, studio notes, audio preview, "Satisfied" gate, revision requests, rating, master download, cancel/refund requests.
- **M5:** admin dashboard (`/admin`, English): order queues, edit orders/scripts/quotes, deliver audio, approve/reject revisions and refunds, live notification bell, portfolio tracks, pricing, logo/favicon/titles/WhatsApp/social links.
- **SEO:** metadata + OpenGraph + hreflang, `sitemap.xml`, `robots.txt`, structured data.

- **M6:** Resend emails (owner alerts + client updates in Arabic/English), contact form with spam protection, legal pages, WhatsApp widget.

Remaining: running everything against real services and the final test pass (M7). See `LAUNCH.md`.

## Run it (demo mode, no accounts needed)

```bash
npm install
npm run dev        # http://localhost:3000  ->  redirects to /ar
npm test           # pricing unit tests
npm run typecheck
```

With no env vars the site runs on built-in demo data and synthesized placeholder audio.

## Connect your real services

1. **Supabase:** create a project, run `supabase/migrations/0001_init.sql` in the SQL editor.
   Create a **public** Storage bucket named `branding` (logo + favicon).
2. **Make yourself admin:** sign up once, then in the SQL editor run
   `update profiles set role = 'admin' where id = '<your auth user id>';`
3. **Cloudflare R2:** create a *public* bucket (or custom domain) for portfolio samples.
   Client scripts and master audio will use a separate *private* bucket in M2/M4.
4. Copy `.env.example` to `.env.local` and fill in the three values.
5. Add tracks (until the M5 admin form exists): upload an MP3 to R2, then insert a row into `audio_tracks`
   with its `storage_key`, `duration_sec`, and `category_id`. Optional `peaks` (array of 0–1 numbers) makes the waveform appear instantly.

## Notes

- `src/lib/pricing.ts` is the single price formula; M3 reuses it on the server so the browser preview can never decide what is charged.
- Rounding for partial minutes is proportional (200 words = $6.67). Switch to whole-minute rounding by passing `rounding: 'ceil'`.
- SF Pro / SF Arabic only render on Apple devices (Apple does not license them for web hosting); other devices use their system font.


## M2/M3 setup (in order)

1. Run `supabase/migrations/0002_payments.sql` after `0001`. **Set `pricing_settings.usd_to_egp` to your real rate** (the default 50 is a placeholder).
2. **Supabase Auth:** enable Email, and Google (Authentication → Providers). Add `http://localhost:3000/auth/callback` and your production URL to the allowed redirect URLs.
3. **R2 private bucket** (`R2_PRIVATE_BUCKET`): create an API token with read/write on that bucket, and add a CORS rule allowing `PUT` from your site origin.
4. **Stripe:** add a webhook endpoint `https://<your-domain>/api/webhooks/stripe` for `checkout.session.completed` and `checkout.session.async_payment_succeeded`; put its signing secret in `STRIPE_WEBHOOK_SECRET`. Apple Pay works automatically on Stripe's hosted checkout page (no domain verification needed).
5. **Paymob:** in the merchant dashboard create one payment integration per method you want (card, wallets, Fawry, InstaPay if available to your account) and list their IDs in `PAYMOB_INTEGRATION_IDS`. Set the *Transaction processed callback* to `https://<your-domain>/api/webhooks/paymob`. Copy the secret key, public key and HMAC secret into `.env.local`.
6. Fill every value in `.env.example` and add the same variables in Vercel. `SUPABASE_SERVICE_ROLE_KEY` and the payment secrets must never be exposed to the browser.

Money flow: the browser only *requests* an order; the server recomputes the price, stores the expected amount before the customer pays, and a webhook marks the order paid only if the signature is valid and the amount matches exactly. See `TESTING.md` for what is and is not covered by tests.


## M4/M5 setup
1. Run `supabase/migrations/0003_portal_admin.sql` (hides master file keys from clients; creates the public `branding` bucket with admin-only write).
2. Make yourself admin (once): `update profiles set role = 'admin' where id = '<your auth user id>';`
3. R2: the **public** bucket (samples) needs CORS for `PUT` from your site origin too; set `R2_PUBLIC_BUCKET`.
4. Enable Realtime on `admin_notifications` (Database → Replication) if the migration's `alter publication` line was skipped.
5. Delivery workflow: export a compressed **preview** (MP3/M4A) and the **master** (WAV/FLAC/MP3); upload both from the order page. The client only ever hears the preview until they press "Satisfied".

Security model: every admin page and `/api/admin/*` route re-checks `profiles.role = 'admin'` on each request (non-admins get 404/403). Settings, tracks, social links and pricing are written straight from the browser, so they are protected by database row-level security, not just by the UI.


**Turning a payment provider on/off:** leave its keys empty in the environment variables and its option disappears from the order form (and its API route refuses requests).
