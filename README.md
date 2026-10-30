# Seyvia

> Portable full-stack source package for the Seyvia storefront and rental-operations dashboard.

For an install, environment setup, Cloudinary image uploads, and Git hand-off, start with [docs/SETUP.md](docs/SETUP.md). Secrets are intentionally excluded from this package; copy `.env.example` to `.env.local` on the new machine.

Monthly development-toy kits for babies and toddlers (8 months to 3 years), deep-cleaned, sealed, delivered and swapped every month across Bengaluru. Next.js 16 (App Router, Cache Components), Tailwind 4, Postgres.

Live (team preview): https://project-3i53q.vercel.app

## Two experiences, same URLs

Phones get an app-style UI (bottom tabs, sheets, swipeable cards). Laptops and tablets get the desktop site. `src/proxy.ts` decides per request from the user agent and **rewrites** phone requests to the `src/app/m/*` tree, so links are identical on every device.

- Force a view on any link: `?view=mobile` or `?view=desktop` (remembered in the `seyvia_view` cookie for 30 days). Handy for reviewing the phone UI on a laptop.
- Visiting `/m/...` directly redirects to the real URL.
- Paths with a mobile version: `/`, `/kits`, `/kits/[age]`, `/hygiene`, `/account`, `/help`, `/checkout`. Everything else (terms, privacy, admin) uses the desktop page on every device.
- Installable as a home-screen app (PWA manifest in `src/app/manifest.ts`, icons in `public/`).

## Pages

| URL | Desktop (`src/app/(site)`) | Phone (`src/app/m`) |
| --- | --- | --- |
| `/` | Landing: hero, shop by age, compare, clean promise, how it works, why grid, toy marquee, pricing, FAQ | Home tab: age grid on the first screen, swipeable how-it-works and starter kits, prices |
| `/kits` | Choose an age (tiles + birth-month finder + newborn waitlist) | Toys tab: age list + newborn waitlist |
| `/kits/[age]` | Age tabs, pick a kit, every toy for the age, toy quick view, sticky order bar | One screen: age chips, Kits / All toys switch, swipe to pick a kit, bottom sheets, pinned Continue |
| `/hygiene` | Full 3-step clean promise | Clean tab |
| `/help` | Contact + FAQ | Help tab (+ "View desktop site") |
| `/checkout?kit=…` | Your kit → phone code → delivery → review | Same flow, full screen, tab bar hidden |
| `/account` | Sign in with phone code, orders and status | Orders tab |
| `/terms`, `/privacy` | Draft policies | (desktop page) |
| `/admin` | Orders, status updates, waitlist (HTTP Basic Auth) | (desktop page) |

Age slugs: `8-12-months`, `12-18-months`, `18-24-months`, `24-36-months`.

## Business rules (as of 2026-10-26)

- **Pricing is month to month only.** Price is set by age group (`monthlyInr` on each group in `src/data/kits.ts`): 8–12 months ₹1,000/month, every older group ₹2,500/month.
- Refundable deposit ₹1,000 (`src/config/site.ts`), a **placeholder**, not decided yet.
- 24–36 month launch kit can upgrade the ride-on to a trampoline for +₹700/month (placeholder price).
- Delivery only to Bengaluru pincodes (560–562); other pincodes join the waitlist.
- Payment is not online yet: the team confirms each order and sends a payment link.

## Catalogue

- `src/data/kits.ts`: age groups, the 4 **launch kits** (from the business's kit sheet), helpers.
- `src/data/sample-kits.ts`: **test data**, 2 extra kits per age (32 real products) to make the catalogue feel full. Marked `sample: true`. Remove before launch.
- Toy photos: `public/toys/*.jpg` (temporary shop-listing images). Lifestyle photos: `public/photos/*.jpg` (Unsplash/Pexels, free licence).
- Cleaning copy: `src/data/hygiene.ts`. FAQ: `src/data/faq.ts`.

## Running locally

```bash
npm install
npm run dev        # http://localhost:3000 (use -p 3100 if 3000 is taken)
npm test           # API tests on in-memory Postgres
npm run lint
npm run build
```

- Without `DATABASE_URL`, an embedded Postgres (PGlite) is used in `.data/`.
- With `.env.local` from `vercel env pull`, local dev uses the **shared Neon database** (test orders show up in the team admin).
- In development the 6-digit sign-in code is shown on screen and `/admin` is open.

## Environment variables

| Name | Where | Purpose |
| --- | --- | --- |
| `DATABASE_URL` | Vercel (from the Neon integration) | Postgres. Tables are created on first use. |
| `AUTH_SECRET` | Vercel | Secret for sign-in codes and sessions. Required in production. |
| `ADMIN_PASSWORD` / `ADMIN_USER` | Vercel | Protects `/admin` (user defaults to `admin`). |
| `UNSAFE_SHOW_SMS_CODES=1` | Vercel (now) | Shows sign-in codes on screen because no SMS provider exists yet. **Remove before real launch.** |
| `SITE_INDEXING=1` | Not set | Lets search engines index the site. Leave unset until launch. |
| `NEXT_PUBLIC_SITE_URL` | Optional | Public origin; falls back to the Vercel URL. |

## Deploying

Vercel project `project-3i53q` (account `prasath30`), Neon database `seyvia-db` in Singapore, deployed from the CLI (no GitHub repo yet).

```bash
npx vercel login
npx vercel link --yes --project project-3i53q
npx vercel env pull .env.local      # optional: use the shared DB locally
npx vercel deploy --prod
```

`vercel.json` pins the framework to Next.js (the project was created as "Other", which served only `public/`).

## Before launch

- Add an SMS provider in `src/server/sms.ts` (MSG91 / Exotel / Twilio Verify with a DLT template), then remove `UNSAFE_SHOW_SMS_CODES`.
- Online payment (Razorpay) in `POST /api/orders`.
- Real catalogue and photos; delete `sample-kits.ts`.
- Confirm the deposit, the upgrade price and the operational promises in the copy (clean steps, damage policy, "within a working day", refund timing). Review `/terms` and `/privacy`.
- Real WhatsApp number and email in `src/config/site.ts`.
- Set `SITE_INDEXING=1`, add a custom domain.
