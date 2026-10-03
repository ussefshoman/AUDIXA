# Launch checklist

Do these in order. Steps 1–3 can start today and run in parallel because they involve waiting on approvals.

## 1. Accounts and domain (start now: some approvals take days)
- [ ] Buy the domain (needed for Paymob callbacks, Google sign-in, and email verification below).
- [ ] **Paymob:** apply for a merchant account; ask them to enable the methods you want (cards, wallets, Fawry, InstaPay). Note the integration IDs, keys and HMAC secret.
- [ ] **Stripe: optional.** Stripe does not onboard Egyptian businesses directly (it needs a US/UK entity or Stripe Atlas). Skip it for launch: the site hides the Stripe option automatically when `STRIPE_SECRET_KEY` is empty. International cards can pay through Paymob.
- [ ] **Supabase** (new project), **Cloudflare R2** (2 buckets: public samples, private files), **Resend**, **Vercel**, **Google Cloud** (OAuth client), **reCAPTCHA v3** keys.

## 2. DNS
- [ ] Point the domain to Vercel.
- [ ] Add the Resend DNS records (SPF/DKIM) and wait for "verified": emails will not reach inboxes before this.

## 3. Database
- [ ] Run `supabase/migrations/0001` → `0004` in order in the Supabase SQL editor.
- [ ] Authentication: enable Email + Google; add `https://<domain>/auth/callback` to redirect URLs.
- [ ] Sign up once on the site, then run: `update profiles set role = 'admin' where id = '<your user id>';`
- [ ] Database → Webhooks: table `admin_notifications`, event INSERT, POST `https://<domain>/api/webhooks/notify`, header `x-webhook-secret: <NOTIFY_WEBHOOK_SECRET>`.
- [ ] Realtime enabled for `admin_notifications`.
- [ ] Set the real USD→EGP rate in Admin → Portfolio & pricing.

## 4. Deploy
- [ ] Import the project in Vercel; add every variable from `.env.example` (secrets only in Vercel, never in the browser).
- [ ] Paymob dashboard: transaction callback = `https://<domain>/api/webhooks/paymob`.
- [ ] R2 buckets: CORS allowing `PUT` from `https://<domain>`.

## 5. Content (from the admin dashboard)
- [ ] Upload logo + favicon, set titles/taglines, WhatsApp, support email, alert email, social links.
- [ ] Upload portfolio tracks (MP3/M4A) per category.
- [ ] Read and adjust the Terms / Privacy / Refund texts in `src/lib/legal.ts`.
- [ ] Add a 1200×630 image for link previews (currently the logo is used).

## 6. Test before announcing (see TESTING.md)
- [ ] Full order with Paymob **test** credentials: pay → bell + email → start work → deliver → listen → Satisfied → download → review.
- [ ] Revision loop, cancellation/refund request, contact form.
- [ ] Phone + desktop, Arabic + English, light + dark.
- [ ] Switch Paymob to live keys and place one small real order.
- [ ] Submit `https://<domain>/sitemap.xml` in Google Search Console.
