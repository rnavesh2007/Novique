# NOVIQUE — AI-Powered Fashion E-Commerce Platform

**Style, Personalized by AI**

This is a real, runnable starter codebase for NOVIQUE: a fashion marketplace where the AI builds you a complete outfit — not just a product page. It's organized as two services plus a database:

```
novique/
├── backend/    Express + TypeScript API, Prisma ORM, PostgreSQL
├── frontend/   Next.js 14 (App Router) + TypeScript + Tailwind
├── docker-compose.yml
└── .env.example
```

## Read this first: what's real vs. scaffolded

A production SaaS platform of this scope — three payment gateways, computer-vision visual search, ML trend forecasting, multi-vendor operations — is normally a multi-month effort for a team. This codebase gives you a genuine, working foundation rather than a hollow mockup:

**Fully implemented and working** (once you add your own API keys/DB):
- JWT auth (access + refresh tokens, httpOnly refresh cookie), bcrypt password hashing, email verification & password reset flows, Google OAuth token verification
- Full Prisma schema: users, products, variants, categories, vendors, orders, payments, coupons, reviews, wardrobe, chat
- Product catalog with filtering/search/pagination, cart, wishlist, virtual wardrobe
- Order placement with real stock checks, tax/shipping calc, coupon application, a DB transaction that decrements inventory atomically
- **The AI engine's core logic is real, not a mock**: a deterministic color-theory engine (HSL hue-distance classification: complementary/analogous/monochrome/clashing) combined with a garment-pairing graph, producing the "Complete the Look" compatibility scores and explanations. This is rule-based by design — explainable, not a black box.
- Stripe payment intents + webhook handling (with correct raw-body signature verification)
- OpenAI-backed personal shopper chat, occasion-outfit narration, and review summarization
- Security middleware: helmet, rate limiting (tiered: auth/general/AI), CORS, HPP, Zod validation on every mutating route, parameterized queries via Prisma (no raw SQL), purchase-gated reviews
- Admin and vendor dashboards with real stats queries
- SEO: dynamic sitemap.xml, robots.txt, Open Graph tags, JSON-LD structured data

**Scaffolded with a clear contract, needs your credentials/effort to finish**:
- **Razorpay / PayPal**: stub endpoints documenting exactly what to call (`createRazorpayOrder`, `createPaypalOrder` in `payment.controller.ts`) — Stripe is fully wired as the reference implementation to copy.
- **AI Visual Search**: the endpoint contract is defined (`POST /ai/visual-search`) but needs a vision model call (e.g. GPT-4o with image input) wired in — see the comment in `ai.controller.ts`.
- **AI Trend Prediction**: `trendScore` is a real DB column read by `/ai/trending`, but the job that *computes* it from order/wishlist/view velocity isn't included — see `docs/AI_ARCHITECTURE.md` for the design.
- **Cloudinary image upload**: products store image URLs; the upload endpoint itself isn't wired (straightforward `multer` + Cloudinary SDK addition).
- Google Sign-In button needs the actual Google Identity Services script on the frontend (backend verification is done).

Nothing here is copied from H&M or any other retailer — garment categories, color logic, and UI are original to NOVIQUE.

## Quick start (local development)

```bash
# 1. Copy env file and fill in at least DATABASE_URL, JWT secrets, OPENAI_API_KEY
cp .env.example .env

# 2. Start Postgres + Redis
docker compose up -d postgres redis

# 3. Backend
cd backend
npm install
npx prisma migrate dev --name init
npm run seed        # loads ~15 sample products across garment types/colors
npm run dev          # http://localhost:4000

# 4. Frontend (new terminal)
cd frontend
npm install
echo "NEXT_PUBLIC_API_URL=http://localhost:4000/api" > .env.local
npm run dev          # http://localhost:3000
```

Or run everything with Docker:
```bash
docker compose up --build
```

## Minimum env vars to see the AI features work
- `OPENAI_API_KEY` — powers the stylist chat, occasion-outfit narration, review summaries. Without it, those endpoints return a clear 500/fallback rather than crashing.
- `STRIPE_SECRET_KEY` + `STRIPE_WEBHOOK_SECRET` — for checkout payment.
- Color Intelligence, Complete the Look, and Size Recommendation work with **no external API keys** — they're pure TypeScript logic.

## Architecture docs
- `docs/AI_ARCHITECTURE.md` — how the recommendation engine works, and how to extend trend prediction / visual search
- `docs/API.md` — endpoint reference
- `docs/DEPLOYMENT.md` — Docker/Nginx/Vercel/AWS notes

## Security notes
See inline comments throughout, especially `index.ts`, `auth.controller.ts`, and `recommendation.service.ts`. Highlights: parameterized queries only (Prisma), httpOnly refresh cookie + bearer access token (avoids CSRF on the access-token flow), constant-shape login errors (no email enumeration), purchase-gated reviews, soft-deletes on products, stock re-validated at checkout inside a DB transaction.

This is a starting point for a commercial build, not a finished, audited production system — run a real security review (dependency audit, pen test, rate-limit tuning for your traffic) before launch.
