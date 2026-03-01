# Plan : Porter les features de citedby-v3 vers V2 Next.js

## Contexte

citedby-v3 (Vite, `C:\Users\User\Documents\citedby-v3`) contient le frontend le plus a jour avec 36 pages, AI Overviews, et des composants refactores. Mais il n'a PAS de backend (pas de Supabase, Stripe, auth).

Le projet V2 Next.js (`C:\Users\User\Downloads\SaaS X-Final-Rick-Dossier Sauvegarde\V2SaaS X-Final-Rick-Test - Test Copie`) a 33 pages + toute l'infrastructure backend (Supabase, Stripe, API routes, migrations).

**Objectif** : Ecraser le frontend de V2 avec celui de citedby-v3 tout en preservant l'infrastructure backend de V2.

## Strategie

**Overwrite frontend, keep backend.** C'est safe car :
- Le SPA (pages/, services/, components/) et le backend (app/, lib/, supabase/) sont independants
- Ni l'un ni l'autre n'importe depuis l'autre couche
- Aucun des deux App.tsx n'utilise Supabase directement

## Abreviations
- **V3** = `C:\Users\User\Documents\citedby-v3`
- **V2** = `C:\Users\User\Downloads\SaaS X-Final-Rick-Dossier Sauvegarde\V2SaaS X-Final-Rick-Test - Test Copie`

---

## Phase 1 : Copier les NOUVEAUX fichiers (V3 → V2)

22 fichiers qui n'existent PAS dans V2 :

### 3 nouvelles pages
- `pages/AiMonitoring.tsx` (66 KB)
- `pages/Integrations.tsx` (23 KB)
- `pages/ApiDeveloper.tsx` (29 KB)

### 8 nouveaux composants
- `components/AppFooter.tsx`
- `components/CommandPalette.tsx`
- `components/MobileBottomNav.tsx`
- `components/Navbar.tsx`
- `components/OnboardingModal.tsx`
- `components/ShortcutsModal.tsx`
- `components/Sidebar.tsx`
- `components/ToastSystem.tsx`

### 5 nouveaux services
- `services/ai-monitoring.ts`
- `services/api-developer.ts`
- `services/integrations.ts`
- `services/retargeting.ts`
- `services/scheduling.ts`

### 5 nouveaux types
- `types/ai-monitoring.ts`
- `types/api-developer.ts`
- `types/integrations.ts`
- `types/retargeting.ts`
- `types/scheduling.ts`

### 1 nouveau CSS
- `globals.css` (racine)

---

## Phase 2 : Ecraser les fichiers frontend existants (V3 → V2)

~70 fichiers qui existent dans les DEUX projets — on prend la version V3 (plus recente, avec AI Overviews) :

### 33 pages (overwrite V2/pages/ avec V3/pages/)
Toutes les pages existantes : Account, ActionPlan, Affiliation, AiCitationSimulator, AiResponseLayer, AlertsAEO, Analyze, AnswerFirstAudit, BlogOptimizer, BotTracking, Clients, ContentWriter, Dashboard, Directory, DirectoryEntity, EeatBuilder, GenerativeTools, History, Home, LlmsTxtGenerator, Login, Messages, MissionOrders, Plan, Projects, PublisherRequest, Recharge, Reputation, Results, RobotsTxt, SchemaOrg, SMCStudio, SourceOfTruth

### 7 composants existants (overwrite)
AICitationPreview, CategoryCard, GenerationModal, RecommendationCard, ScoreRing, SourceOfTruthModal, StripeCheckout

### 15 services racine + sous-dossiers (overwrite)
- 15 fichiers .ts racine (ai-citation-simulator, analyzer, answer-first-audit, blog_prompt_db, content_writer, criteria, database, generator, llms-txt-generator, llm_templates, pdf_prompt, reputation_service, robots_service, schema_service, smc)
- `services/ai-response-layer/` (6 fichiers)
- `services/eeat/` (8 fichiers)

### 2 types existants (overwrite)
- `types/ai-response-layer.ts`
- `types/eeat.ts`

### i18n complet (overwrite)
- `i18n/context.tsx`, `i18n/index.ts`, `i18n/types.ts`, `i18n/LanguageSelector.tsx`
- `i18n/locales/fr.ts`, `i18n/locales/en.ts`, `i18n/locales/es.ts`

### Fichiers racine frontend (overwrite)
- `App.tsx` (V3: 44KB refactorise vs V2: 87KB monolithe)
- `types.ts` (ajoute `'AI Overviews'` au type AIMention)
- `index.tsx` (ajoute `import './globals.css'`)
- `index.html` (V3: 17 lignes propres vs V2: 105 lignes avec Tailwind CDN)

---

## Phase 3 : Merger tailwind.config.ts

Le SEUL fichier a merger manuellement. Combiner :
- Les `content` paths de V3 (pages/, components/, App.tsx, etc.)
- Les `content` paths de V2 (src/**)
- Les couleurs de V3 (`primary: "#7c3aed"`)
- Les couleurs de V2 (`leadradar: { ... }`)
- Les animations de V2 (pulse-live, slide-in)

---

## Phase 4 : NE PAS TOUCHER (garder V2)

Tous les fichiers backend et config :
- `app/` (API routes Next.js : Stripe webhooks, auth, runs)
- `lib/` (Supabase client, Stripe, utils)
- `supabase/` (10 edge functions, 8 migrations SQL)
- `middleware.ts`, `next.config.mjs`, `vercel.json`
- `package.json`, `package-lock.json`
- `.env.example`, `.env.local`
- `vite.config.ts`, `tsconfig.json`, `postcss.config.cjs`
- `scripts/`, `docs/`, `qa/`, `n8n/`, `tests/`, `src/`
- `.claude/`, `.git/`, `CLAUDE.md`, `README.md`
- `CitedBy_AOE_Criteria_50.json`, `DEPLOYMENT_AUDIT.md`
- `app/globals.css` (CSS Next.js, separe du SPA)

---

## Phase 5 : Validation

1. `npm install` (regenerer lock si besoin)
2. `npm run build` → 0 erreurs TypeScript
3. `npm run dev` → ouvrir http://localhost:5173
4. Verifier visuellement :
   - 36 pages accessibles via sidebar
   - AI Overviews (rose #f43f5e) visible dans Monitoring, Simulator, Integrations, Reputation
   - 3 nouvelles pages : AI Monitoring, Integrations, API Developer
   - Composants refactores : Navbar, Sidebar, CommandPalette (Ctrl+K)

---

## Implementation

Methode : utiliser `cp -r` en bash pour copier par lot les fichiers de V3 vers V2. Le tailwind.config.ts sera merge via Edit.
