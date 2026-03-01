# RoyallAzz Project Details

## Architecture
- Single Next.js 15 app with route groups: (auth), (merchant), (customer)
- Supabase: Auth + PostgreSQL + RLS
- 8 DB tables: merchants, loyalty_programs, rewards, customers, customer_cards, transactions, merchant_qr_codes, customer_qr_codes
- Stripe for subscription billing (lazy-init via `src/lib/stripe.ts` to avoid build errors)

## Implementation Status (2026-02-27)
- **Phase 1-4**: Complete (scaffolding, UI components, layout, Supabase+Auth)
- **Phase 5**: Complete - Dashboard with real KPIs, programs CRUD, QR generator connected to DB
- **Phase 6**: Complete - Customer portal with cards, card detail page, QR code page, settings
- **Phase 7**: Complete - PWA manifest, not-found/error pages, all placeholder pages replaced
- **Phase 8**: Complete - API routes, Stripe integration, enhanced scanner

## API Routes
- `POST /api/scan/validate` - Validate customer QR code, return customer info + cards
- `POST /api/scan/earn` - Add points/stamps to a customer card via QR or ID
- `POST /api/rewards/redeem` - Redeem a reward for a customer (deducts balance)
- `POST /api/stripe/checkout` - Create Stripe Checkout session for plan upgrade
- `POST /api/stripe/webhook` - Handle Stripe subscription events

## Key Components
- `src/components/merchant/PlanSelector.tsx` - Stripe Checkout integration
- `src/components/merchant/ManualScanner.tsx` - Dual mode (email + QR), reward redemption
- `src/components/customer/LoyaltyCard.tsx` - Clickable cards with merchant name
- `src/app/(customer)/customer/qr/qr-display.tsx` - Personal QR with 30s rotation
- `src/app/(customer)/customer/cards/[id]/page.tsx` - Card detail with history + rewards
- `src/lib/stripe.ts` - Lazy Stripe initialization helper

## Build Notes
- All server component pages need `export const dynamic = 'force-dynamic'`
- Pages Router compat files in `src/pages/` (_document, _app, _error)
- Stripe SDK must be lazy-initialized (not top-level) to avoid build errors
- `output: 'standalone'` removed (EPERM symlink on Windows)

## Remaining TODO (post-MVP)
- Camera-based QR scanning (browser camera API in merchant scanner)
- Email notifications
- Supabase Edge Functions (send-push)
- Tests (unit, integration, e2e)
- Stripe customer portal for subscription management
