# Plan Phase 1 — Migration CitedBy.ai vers Next.js 15 + Supabase

## Contexte

CitedBy.ai est un MVP fonctionnel (Vite + React 19 SPA) avec ~9 pages, 10 services, 3 composants, et un "backend" entièrement en localStorage. L'objectif de la Phase 1 est de migrer vers Next.js 15 (App Router) avec Supabase comme backend réel, tout en conservant l'ensemble des fonctionnalités existantes.

**Changements majeurs :**
- Vite → Next.js 15 App Router
- localStorage → Supabase PostgreSQL + RLS
- Auth mock → Supabase Auth réel
- Tailwind CDN → Tailwind npm
- Palette noir/émeraude → slate/bleu/Inter
- API keys dans le browser → Server Actions sécurisées
- App.tsx monolithique (routing par string) → routes fichiers + layouts

---

## Étape 1 — Initialiser le projet Next.js 15

**Fichiers créés :**
- `package.json` — Next.js 15, React 19, TypeScript, Tailwind, @supabase/supabase-js, @supabase/ssr, lucide-react, recharts, framer-motion, d3, @google/genai, clsx, tailwind-merge
- `tsconfig.json` — strict: true, paths `@/*` → `./src/*`
- `next.config.ts` — config minimale
- `postcss.config.mjs` — tailwindcss + autoprefixer
- `.env.local` — NEXT_PUBLIC_SUPABASE_URL, NEXT_PUBLIC_SUPABASE_ANON_KEY, SUPABASE_SERVICE_ROLE_KEY, GEMINI_API_KEY (server-only)
- `.env.example` — template des variables
- `.gitignore` — node_modules, .next, .env.local

**Action :** `npm install` pour installer toutes les dépendances.

---

## Étape 2 — Configurer Tailwind CSS (npm) + thème

**Fichiers créés :**
- `tailwind.config.ts` — Palette slate/bleu, police Inter, extensions custom
- `src/app/globals.css` — Directives Tailwind, classes `.glass-panel`, `.custom-scrollbar`, fond sombre, scrollbar custom

**Palette confirmée :**
- Background : `#0F172A` (slate-900)
- Surface : `#1E293B` (slate-800)
- Accent : `#3B82F6` (blue-500)
- Texte : `#F8FAFC` (slate-50)
- Police : Inter (Google Fonts via next/font)

---

## Étape 3 — Supabase clients (browser + serveur + middleware)

**Fichiers créés :**
- `src/lib/supabase/client.ts` — `createBrowserClient` pour composants client
- `src/lib/supabase/server.ts` — `createServerClient` avec cookies() pour composants serveur
- `src/lib/supabase/middleware.ts` — helper qui rafraîchit la session auth

---

## Étape 4 — Schéma de base de données complet + RLS

**Fichier créé :** `supabase/migrations/001_initial_schema.sql`

Tables créées (17) — exactement celles du prompt original :
- `profiles`, `subscriptions`, `credit_purchases`
- `projects`, `analyses`, `score_history`
- `tracked_queries`, `citations`, `citation_alerts`
- `publishers`, `publisher_offers`, `orders`, `order_messages`, `reviews`
- `generated_solutions`, `indexation_configs`
- `affiliates`, `referrals`, `notifications`

Chaque table a : RLS activé, policies de sécurité, indexes appropriés.
Trigger `handle_new_user()` pour créer un profil automatiquement à l'inscription.

---

## Étape 5 — Types TypeScript (strict, zéro `any`)

**Fichiers créés :**
- `src/types/database.ts` — type `Database` Supabase (Row/Insert/Update par table)
- `src/types/index.ts` — exports centralisés + types applicatifs
- `src/types/analysis.ts` — AnalysisResult, CategoryScore, Recommendation, etc.
- `src/types/marketplace.ts` — Publisher, Offer, Order, Review
- `src/types/billing.ts` — Plan, CreditPurchase

---

## Étape 6 — Utilitaires et configuration

**Fichiers créés :**
- `src/lib/utils.ts` — formatage dates FR, validation URL, etc.
- `src/lib/cn.ts` — helper clsx + tailwind-merge
- `src/config/constants.ts` — AEO_CATEGORIES (50+ critères), PLANS, CREDIT_COSTS
- `src/config/navigation.ts` — items de navigation sidebar
- `src/config/prompts/expert-prompts.ts` — prompts manquants (corrige l'import cassé de SMCStudio)
- `src/config/prompts/blog-prompts.ts` — porté depuis `blog_prompt_db.ts`
- `src/config/prompts/llm-templates.ts` — porté depuis `llm_templates.ts`

---

## Étape 7 — Layout principal (Sidebar + TopBar + Mobile)

**Fichiers créés :**
- `src/app/layout.tsx` — Root layout : metadata, Inter via next/font, providers, `<html lang="fr" class="dark">`
- `src/app/(app)/layout.tsx` — Layout authentifié : AppShell (sidebar + topbar + contenu)
- `src/components/layout/app-shell.tsx` — Wrapper structurel (client component)
- `src/components/layout/sidebar.tsx` — Navigation latérale avec `<Link>` + `usePathname()` pour état actif
- `src/components/layout/top-bar.tsx` — Logo, crédits, avatar utilisateur
- `src/components/layout/mobile-nav.tsx` — Barre de navigation mobile en bas

**Mapping des routes :**
| Ancienne vue | Nouvelle route |
|---|---|
| `analyze` | `/analyse/nouvelle` |
| `dashboard` | `/tableau-de-bord` |
| `projects` | `/projets` |
| `explorer` | `/aeo-studio/knowledge-graph` |
| `benchmark` | `/aeo-studio/benchmark` |
| `smc-studio` | `/aeo-studio` |
| `plans` | `/facturation` |
| `recharge` | `/facturation/recharge` |
| `account` | `/parametres` |

---

## Étape 8 — Middleware Next.js (garde d'authentification)

**Fichier créé :** `src/middleware.ts`
- Rafraîchit la session Supabase sur chaque requête
- Routes `/(app)/*` : redirige vers `/connexion` si pas de session
- Routes `/(auth)/*` : redirige vers `/tableau-de-bord` si déjà connecté
- Landing page `/` : accessible à tous

---

## Étape 9 — Authentification Supabase

**Fichiers créés :**
- `src/app/(auth)/layout.tsx` — Layout centré sans sidebar
- `src/app/(auth)/connexion/page.tsx` — Formulaire login (email/password + magic link)
- `src/app/(auth)/inscription/page.tsx` — Formulaire inscription
- `src/app/(auth)/auth/callback/route.ts` — Callback OAuth/magic link
- `src/contexts/auth-context.tsx` — Provider React Context pour l'état auth
- `src/hooks/use-user.ts` — Hook `useUser()` retournant { user, profile, isLoading }

Remplace entièrement le mock login actuel (Login.tsx hardcode `id: 'usr_123'`).

---

## Étape 10 — Error boundaries et loading states

**Fichiers créés :**
- `src/app/error.tsx` — Error boundary racine
- `src/app/not-found.tsx` — Page 404
- `src/app/loading.tsx` — Loading racine
- `src/app/(app)/loading.tsx` — Loading dans le layout app (skeleton)
- `src/app/(app)/error.tsx` — Error boundary dans le layout app
- `src/components/ui/loading-spinner.tsx`

---

## Étape 11 — Migration des composants UI

**Fichiers créés :**
- `src/components/ui/score-ring.tsx` — porté depuis ScoreRing.tsx (`'use client'`)
- `src/components/ui/score-bar.tsx` — nouveau, barre de progression horizontale
- `src/components/analysis/recommendation-card.tsx` — porté, types stricts
- `src/components/analysis/generation-modal.tsx` — porté, `'use client'`

---

## Étape 12 — Migration des services

| Service actuel | Destination | Raison |
|---|---|---|
| `analyzer.ts` | `src/services/analyzer.ts` + server action | Logique d'analyse (pas d'API externe pour l'instant) |
| `generator.ts` | `src/app/(app)/analyse/actions.ts` | Appels Gemini → server action sécurisé |
| `criteria.ts` | `src/config/criteria.ts` | Données statiques, partagé |
| `database.ts` | **SUPPRIMÉ** | Remplacé par Supabase |
| `smc.ts` | `src/app/(app)/aeo-studio/actions.ts` | Appels Gemini → server action |
| `schema_service.ts` | `src/services/schema-service.ts` | Fonction pure, pas de clé API |
| `robots_service.ts` | `src/services/robots-service.ts` | Fonction pure |

**Règle de sécurité :** `GEMINI_API_KEY` n'est JAMAIS accessible côté client. Tous les appels GenAI passent par des server actions.

---

## Étape 13 — Migration des pages

### 13.1 — Landing Page : `src/app/page.tsx`
- Source : `Home.tsx` (117 lignes)
- Type : Server Component
- `onNavigate` → `<Link href="...">`

### 13.2 — Dashboard : `src/app/(app)/tableau-de-bord/page.tsx`
- Source : `Dashboard.tsx` (278 lignes)
- Décomposé en :
  - `page.tsx` (server — fetch analyses depuis Supabase)
  - `src/components/dashboard/dashboard-client.tsx` (client — charts + interactivité)
  - `src/components/dashboard/kpi-grid.tsx`
  - `src/components/dashboard/trajectory-chart.tsx`
  - `src/components/dashboard/activity-feed.tsx`

### 13.3 — Nouvelle analyse : `src/app/(app)/analyse/nouvelle/page.tsx`
- Source : `Analyze.tsx` (51 lignes)
- Type : Client Component (formulaire)
- Server action pour lancer l'analyse → redirige vers `/analyse/resultats/[id]`

### 13.4 — Résultats : `src/app/(app)/analyse/resultats/[id]/page.tsx`
- Source : `Projects.tsx` (375 lignes — DOIT être décomposé)
- Décomposé en :
  - `page.tsx` (server — fetch par ID)
  - `src/components/analysis/results-client.tsx` (client — onglets)
  - `src/components/analysis/overview-tab.tsx`
  - `src/components/analysis/action-plan-tab.tsx`
  - `src/components/analysis/archives-tab.tsx`
  - `src/components/analysis/category-accordion.tsx`

### 13.5 — Projets : `src/app/(app)/projets/page.tsx`
- Nouvelle page (projets = table Supabase)

### 13.6 — AEO Studio : `src/app/(app)/aeo-studio/...`
- Source : `SMCStudio.tsx` (683 lignes — DOIT être décomposé en 8+ fichiers)
- Routes :
  - `/aeo-studio` — Hub principal
  - `/aeo-studio/source-of-truth` — LLM Source of Truth
  - `/aeo-studio/schema-org` — Schema.org Studio
  - `/aeo-studio/robots-txt` — Robots.txt
  - `/aeo-studio/smc/fiche` — Fiches produits
  - `/aeo-studio/smc/faq` — FAQ
  - `/aeo-studio/smc/article` — Articles
  - `/aeo-studio/knowledge-graph` — Graphe D3.js (`'use client'`)
  - `/aeo-studio/benchmark` — Recharts (`'use client'`)
- Composants partagés : `tool-wrapper.tsx`, `hub-card.tsx`

### 13.7 — Facturation : `src/app/(app)/facturation/page.tsx`
- Source : `Plans.tsx` (176 lignes)

### 13.8 — Recharge : `src/app/(app)/facturation/recharge/page.tsx`
- Source : `Recharge.tsx` (68 lignes)

### 13.9 — Paramètres : `src/app/(app)/parametres/page.tsx`
- Source : `Account.tsx` (75 lignes)

---

## Étape 14 — Vérification build

```bash
npx next build     # 0 erreurs, 0 warnings
npx tsc --noEmit   # TypeScript strict pass
```

Résoudre tous les problèmes :
- Directives `'use client'` manquantes
- Types `any` restants
- Imports cassés
- Variables d'env manquantes

---

## Corrections de bugs existants à traiter

1. **Import cassé** : `SMCStudio.tsx` ligne 5 importe `../services/expert-prompts` qui n'existe pas → créer `src/config/prompts/expert-prompts.ts`
2. **Type manquant** : `database.ts` importe `AgencyOrder` de `../types` qui n'est pas défini → supprimer (database.ts est supprimé)
3. **Sécurité API key** : `vite.config.ts` expose `GEMINI_API_KEY` au browser → éliminé avec les server actions

---

## Fichiers critiques à modifier/comprendre

| Fichier source | Rôle | Impact migration |
|---|---|---|
| `App.tsx` | Routing + state + layout monolithique | Décomposé en routes, layouts, contexts |
| `types.ts` | Tous les types | Évolué vers types alignés DB |
| `pages/SMCStudio.tsx` | 683 lignes, 6+ outils | Décomposé en 8+ fichiers |
| `pages/Projects.tsx` | 375 lignes, résultats d'analyse | Décomposé en 5+ fichiers |
| `services/generator.ts` | Intégration GenAI | Migré vers server action |
| `services/database.ts` | localStorage CRUD | Supprimé, remplacé par Supabase |

---

## Vérification finale

Après la Phase 1 :
- [ ] `npx next build` → 0 erreurs
- [ ] `npx tsc --noEmit` → 0 erreurs
- [ ] `grep -r "API_KEY\|SECRET\|sk_" src/` → 0 résultats
- [ ] RLS activé sur les 17 tables
- [ ] Auth fonctionne (inscription, connexion, déconnexion)
- [ ] Toutes les pages accessibles et fonctionnelles
- [ ] Pas de `any` dans le code TypeScript
- [ ] Chaque fichier < 300 lignes

---

## Ordre d'exécution

```
1. Init Next.js
2. Tailwind
3. Supabase clients
4. DB schema + RLS
5. Types TypeScript
6. Utils + Config
7. Layout (Sidebar, TopBar, Mobile)
8. Middleware auth
9. Auth (pages + context + hook)
10. Error boundaries
11. Composants UI
12. Services
13. Pages (landing → dashboard → analyse → projets → studio → billing → settings)
14. Build verification
```

~55-60 fichiers à créer. Le projet existant reste comme référence — on construit la nouvelle structure à côté dans `src/`.
