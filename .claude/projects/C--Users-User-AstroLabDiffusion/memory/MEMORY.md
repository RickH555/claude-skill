# AstroLab - Projet Memoire

## Projet
- **Nom**: AstroLab (diffusion) - Carte de fidelite digitale B2B2C
- **Chemin racine monorepo**: `C:\Users\User\AstroLabDiffusion\astrolab\`
- **Stack**: Turborepo + pnpm workspace + TypeScript strict

## Structure Monorepo
- `apps/web/` - React 18 + Vite 5 (package: `@astrolab/web`)
- `apps/mobile/` - Expo SDK 51 (package: `@astrolab/mobile`)
- `packages/shared/` - Types + client Supabase (package: `@astrolab/shared`)
- `packages/tsconfig/` - Config TS partagee (package: `@astrolab/tsconfig`)

## Commandes cles
- `pnpm install` - Installer deps
- `pnpm build` - Build tous les packages
- `pnpm typecheck` - Verifier types
- `pnpm dev:web` ou `pnpm dev --filter @astrolab/web` - Dev server web (port 5173)
- `pnpm dev:mobile` - Dev server mobile

## Architecture technique
- Filtrage Turbo utilise les noms de package (`@astrolab/web`, pas `web`)
- Web tsconfig: pas de `paths` alias (resolution via workspace link pnpm)
- Vite config: alias `@astrolab/shared` -> `../../packages/shared/src` pour hot reload
- Shared: `main`/`types` pointent vers `./src/index.ts` directement (pas dist)
- Shared tsconfig: `composite: true` pour le build avec declarations

## Types partages (packages/shared/src/types/)
- **enums.ts**: constantes as const + types derives (UserRole, LoyaltyType, SubscriptionStatus, etc.)
- **index.ts**: interfaces alignees 1:1 avec le schema SQL
- Interfaces: Profile, Merchant, Customer, LoyaltyCard, Reward, Scan, Campaign, QRCode, CampaignRedemption, OneSignalSubscription
- `createSupabaseClient(url, anonKey)` - factory sans cles hardcodees

## Schema Supabase (supabase/migrations/)
- 10 tables: profiles, merchants, customers, loyalty_cards, rewards, scans, campaigns, qr_codes, campaign_redemptions, onesignal_subscriptions
- RLS active sur les 10 tables (34 policies)
- 3 triggers: on_auth_user_created, updated_at, validate_qr_redemption
- 3 helpers RLS: auth.user_role(), auth.merchant_id(), auth.customer_id()
- UNIQUE: loyalty_cards(customer_id, merchant_id), campaign_redemptions(campaign_id, customer_id)
- Seed data: supabase/seed.sql (1 merchant, 3 customers, 5 scans, 1 campagne)

## Auth Web (apps/web/ - etape 3)
- TailwindCSS v3 + PostCSS + variables CSS custom (:root)
- Client Supabase: `src/lib/supabase.ts` via factory shared + env vars
- AuthContext: user, profile, loading, signUp/signIn/signOut, isMerchant/isCustomer
- Routing: ProtectedRoute, RoleRoute, PublicRoute (React Router v6 layout route pattern)
- Router dans `src/router.tsx`, App.tsx = layout root avec AuthProvider + ToastProvider + Outlet
- main.tsx = RouterProvider direct
- Composants UI: Button (4 variantes), Input, Card, Spinner, Toast (context)
- Layouts: AuthLayout (centre), AppLayout (navbar)
- Pages: Landing, Login, SignUpMerchant, SignUpCustomer, ForgotPassword, ResetPassword, NotFound
- Dashboards placeholder: MerchantDashboard, CustomerDashboard

## Preferences utilisateur
- Terminer chaque etape par "Etape suivante ?"

## Etapes completees
- Etape 1: Initialisation monorepo Turborepo (commit OK)
- Etape 2: Schema Supabase complet + RLS + triggers + types (commit OK)
- Etape 3: Authentification web complete - landing, signup, login, routing protege (commit 1bc7181)
