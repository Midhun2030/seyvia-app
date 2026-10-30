@AGENTS.md

# Seyvia: context for Claude

Read README.md first for routes, env vars and deploy steps. This file holds the decisions, preferences and history behind them, so work can continue on another machine without the original conversation.

## The business

- Seyvia (spelled S-E-Y-V-I-A, capital S in the wordmark) rents kits of **4 toys (1 big + 3 small)** to families in **Bengaluru**, swapped monthly, for ages 0–3. Kits currently cover 8–36 months; 0–8 months is "coming soon" with a waitlist.
- Audience: **moms**. The buyer is the parent, not the child.
- **Hygiene is the #1 trust message.** The 3-step clean (deep wash with detergent / mild bleach, child-safe sanitiser, checked and sealed in a fresh wrap) must be communicated strongly on every relevant screen.
- Story: toys are costly, kids get bored fast, then toys become clutter and waste. Seyvia means less cost, less clutter, kinder to the planet, and development-focused toys picked with research + AI, then chosen by people.
- Toys are **mixed materials and brands** (plastic, silicone, fabric, wood). Never position them as "wooden" or "Channapatna".
- Pricing: **monthly only, no multi-month plans.** ₹1,000/month for 8–12 months, ₹2,500/month for older groups. Deposit ₹1,000 is a placeholder.
- Original kit data came from the owner's sheet (toykits.pdf). The 24–36 kit uses the ride-on bike as standard, with the trampoline as a paid upgrade (owner's choice).

## Design decisions (owner-approved, don't undo)

- Palette: **marigold + charcoal**, light theme only. Background `#FFFDF8`, text `#26231F`, accent marigold `#F2B01E` (buttons carry charcoal text), wash `#FFF3D1`, pale lilac `#EDE7FA` for small touches. Tokens are in `src/app/globals.css`.
- **Rejected**, don't reintroduce: blue as brand colour, dark/navy section or footer backgrounds, emoji, rainbow palettes, Fraunces/serif headings, multi-month plan tiers, cartoonish toy photos.
- Fonts: Outfit (headings) + Figtree (body) via next/font.
- Shapes: cards 24px radius, image tiles 20px, buttons and chips full pills, inputs 14px.
- UI work follows the `design-taste-frontend` skill: no em-dashes in copy, eyebrows used sparingly, real images, no fake stats, plain copy.
- **Desktop and mobile are deliberately different products**, not one responsive layout. Phones must feel like a native app: bottom tabs (Home, Toys, Clean, Orders, Help), compact top bar, swipe rows, bottom sheets, key screens fitting one viewport, tab bar hidden in checkout. See "Two experiences" in README. New mobile screens go in `src/app/m/*` and must be added to `MOBILE_PATHS` in `src/proxy.ts`.
- The kits flow must be obvious: age → pick a kit → see every toy for the age → Continue (sticky bar). Don't go back to "inputs on top, pile of cards below".
- **Growth-chart ruler** (added 2026-10-27, waiting for owner review): the age picker is a marigold measuring tape marked 0 to 36 months, each band as wide as its months (`components/catalog/GrowthRuler.tsx`, phone version `components/mobile/VerticalRuler.tsx`). The scalloped `components/ui/Seal.tsx` sticker means cleaned and sealed, so use it only for that. Full design board: `design/seyvia-website-design.html`.

## How the owner likes to work

- Asks for an **expert's judgement** (a senior product designer / engineer with 5–10 years' experience): push back, flag missed things, and don't implement literally when it's worse for parents.
- Wants clarifying questions **before** big builds when something is genuinely ambiguous, not after.
- Not a developer: explain in plain language, give exact steps when they must act (logins, accepting terms).
- Confirm before outward-facing actions (deploys, pushing code, creating cloud resources).

## Architecture notes

- Next.js **16** with `cacheComponents: true`. Runtime APIs (cookies, params, searchParams) must sit inside `<Suspense>`; `new Date()` in server components needs `await connection()` first. Client components read browser-only values via `src/hooks/useBrowserValue.ts`. Check `node_modules/next/dist/docs/` before using unfamiliar APIs.
- `src/proxy.ts` (Next 16's name for middleware) does mobile/desktop rewriting and Basic Auth for `/admin`.
- DB: Drizzle over Postgres (Neon in prod) or PGlite locally. Schema plus idempotent bootstrap SQL in `src/db/schema.ts`, with no migration tool yet.
- Auth: phone + 6-digit code (`src/server/auth.ts`), hashed codes, rate limits, httpOnly session cookie. SMS sending is stubbed in `src/server/sms.ts`.
- Orders: `POST /api/orders` requires a session; the phone comes from the session, never the form. Prices are computed server-side by `quote()` in `src/lib/orders.ts`.
- Shared between desktop and mobile: `components/account/AccountView.tsx`, `components/help/HelpContent.tsx`, `components/checkout/CheckoutFlow.tsx`, the data files.
- Tests: `src/app/api/flow.test.ts` (vitest, mocks `next/headers`). Run `npm test`, `npm run lint` and `npm run build` before calling anything done. Visual checks were done with playwright-core and system Chrome screenshots (desktop 1440px and the iPhone 13 profile).

## Status (2026-10-26)

Done: desktop landing + kits flow redesign, mobile app experience, monthly age-based pricing, test catalogue (49 toys), Vercel + Neon deploy, PWA manifest, noindex until launch.

Not yet on the live link: the mobile app experience (built and verified locally; deploy with `npx vercel deploy --prod` once the owner says so).

Next up (owner's stated order): redesign the inside of checkout (phone code, delivery form, review) and the post-order screens (confirmation, Orders tab) for both desktop and mobile. Then SMS provider, Razorpay, the real catalogue and photos. The full pre-launch list is in README.

Open items to confirm with the owner: deposit amount, trampoline upgrade price, damage policy wording, "contact within a working day" promise, deposit refund timing, real WhatsApp number and email.
