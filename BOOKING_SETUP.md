# Rayan Studios booking integration — staging build

This is an implementation build, **not live**. Do not deploy until the checklist is complete and Stripe test payments and Google Calendar events have been verified. The original site pages, images and styling remain intact; the BOOK A SESSION links now open `booking.html`.

## Services and schedule
Vocal Tracking 3h AED 1800; Studio Rental 4h AED 2000 or 8h AED 3500; Voice Over 3h AED 1500; Instrument Tracking 3h AED 1800. Monday–Thursday 12:00–23:00; Friday 14:00–23:00; Saturday 13:00–17:00; Sunday closed (Asia/Dubai), starts every 30 minutes, booking at least 2 hours in advance, up to 90 days ahead. 100% payment in Stripe Checkout.

## Vercel Environment Variables (server-only)
- DATABASE_URL — Neon connection string (already provisioned if connected; verify)
- STRIPE_SECRET_KEY — start with Stripe **test** secret key
- STRIPE_WEBHOOK_SECRET — from Stripe webhook endpoint (test mode first)
- GOOGLE_CLIENT_ID
- GOOGLE_CLIENT_SECRET — rotate the secret previously shown in a screenshot before production
- GOOGLE_REFRESH_TOKEN — must be generated securely by authorizing contact@rayanproducer.com; never place in GitHub
- GOOGLE_CALENDAR_ID — primary calendar address or specific calendar ID
- SITE_URL — https://www.rayanstudios.ae (use a staging URL for testing)

Google OAuth scopes needed: `https://www.googleapis.com/auth/calendar.events` and `https://www.googleapis.com/auth/calendar.freebusy`. The backend refreshes tokens; this ZIP intentionally does not include a token-generation endpoint or secrets.

## Stripe
Create webhook endpoint `https://YOUR-STAGING-DOMAIN/api/stripe-webhook` subscribing to `checkout.session.completed` and `checkout.session.async_payment_succeeded`. Use Stripe test mode. For live launch, create corresponding live webhook and replace test credentials. Never trust the browser's success redirect as payment confirmation.

## Neon database
Uses the existing `studio_bookings` table and exclusion constraint created earlier. The `service` column stores internal service IDs (`vocal`, `rental4`, `rental8`, `voiceover`, `instrument`). Do not drop the table. Checkout holds last up to 35 minutes before being marked expired by a subsequent availability/checkout request. The database exclusion constraint prevents simultaneous overlapping holds.

## Important prelaunch work
1. Securely generate Google refresh token; add env vars.
2. Test availability and calendar blocking, including different service lengths and weekday edges.
3. Test Stripe checkout, webhook delivery, and Google event creation. The webhook returns a retriable error if calendar creation fails; **manually reconcile** paid bookings if Google is unavailable.
4. Configure booking confirmation emails and studio notifications (not implemented in this build). The Stripe receipt is not a substitute for a full session confirmation email.
5. Add a background cleanup job for abandoned checkouts and a cancellation/refund policy.
6. Add reconciliation/idempotency hardening for Stripe webhook and Google Calendar creation, and prevent late successful payments for expired slots from causing conflicts.
7. Add rate limiting / abuse prevention to public checkout and availability endpoints before production.

**Do not advertise this as a fully operational live booking system until these steps are completed.**
