# Testing

## Automated (run: `npm test`, `npm run typecheck`)
59 unit tests cover the logic where a bug costs money or breaks trust:

| Area | What is verified |
|---|---|
| Pricing | $5/min flat, $5 minimum, flat rush fees (+$0/+$2/+$3), words↔minutes, rounding modes, bad input |
| Order builder (server) | price decided only from server-side word count; client-sent amounts ignored for typed scripts; "Other" needs a name and is flagged for review; script required (text XOR file); length limits |
| Files & input | PDF/TXT/DOCX only, 10 MB cap, Arabic file names, path tricks, per-user storage keys, open-redirect protection |
| Order status | no skipping payment / work / the satisfaction gate; terminal states; master audio only when `completed` |
| Payments | Paymob HMAC (20 fields, tamper / wrong secret / missing values), outcome mapping, exact-amount matching, USD→EGP |
| Portal & admin rules | only owners act on their orders; "Satisfied" / revision / review / cancel gates; admin cannot skip payment, delivery or the client's approval; refund plan per provider; admin edit whitelist; audio/image file rules; order timeline |
| Email & forms | bilingual templates with correct direction, injected HTML escaped, contact validation, honeypot, reCAPTCHA verdicts, webhook secret comparison |
| SEO | canonical + hreflang URLs; private paths excluded |
| Player | only one sample plays at a time; destroyed players released; time display safe for NaN/negative |

## NOT yet covered by automated tests (needs a machine with internet)
- Page rendering, RTL/mobile layout, waveform player in a real browser
- Supabase RLS policies and `mark_order_paid` against a real database
- Paymob / Stripe sandbox payments and webhooks end to end

## Manual checklist before launch
**Devices/viewports:** iPhone SE (375), iPhone 15 (393), iPad (768/1024), laptop (1440), desktop (1920); Safari iOS, Chrome Android, Chrome/Edge/Firefox/Safari desktop.
For each, in both `/ar` (RTL) and `/en`, light and dark:
- [ ] No horizontal scroll; no text clipping; tap targets ≥ 44px
- [ ] Waveform scrubs by touch; only one sample plays; switching tabs stops playback cleanly
- [ ] Calculator: every category × minutes/words × three speeds shows the expected total
- [ ] Order flow: guest → login → returns to the order with choices kept
- [ ] Typed script, TXT, PDF, DOCX; 11 MB file rejected; `.exe` rejected
- [ ] "Other" without a name is blocked; with a name it appears in the order record

**Payments (sandbox first):**
- [ ] Stripe test card success → order becomes `new`, bell + email fire once
- [ ] Same webhook delivered twice → still one transaction, one notification
- [ ] Paymob test card and a wallet/Fawry test → order becomes `new`
- [ ] Tampered Paymob callback (wrong hmac) → rejected 401
- [ ] Payment amount different from the order → rejected, order stays `pending_payment`
- [ ] Abandon checkout → order stays `pending_payment`, no notification
