# CitedBy.ai — Refonte Frontend Complète

## Context
CitedBy.ai est un SaaS AEO (Answer Engine Optimization) construit avec Next.js 14 App Router. Le projet a déjà une base solide (types, services, lib, i18n, composants UI, API routes) mais le frontend doit être **entièrement reconstruit** avec un design dark premium (style Linear/Vercel) et des données mock pour fonctionner sans Supabase/API keys.

**Projet** : `C:\Users\User\Downloads\SaaSV13 - Rick`

## Ce qui est CONSERVÉ tel quel
- `types.ts` — Toutes les interfaces TypeScript
- `components/ui/` — 13 composants UI production-ready
- `components/charts/` — 3 composants Recharts
- `components/ScoreRing.tsx`, `CategoryCard.tsx`, `RecommendationCard.tsx`, `AICitationPreview.tsx`
- `tailwind.config.ts` — Design tokens (primary: #10b981, bg: #0a0a0f)
- `i18n/` — Système multilingue FR/EN/ES
- `lib/` — Utilitaires (utils.ts, theme-context, workspace-context)
- `services/` — 15 services AI
- `app/api/` — Routes API intactes
- `app/layout.tsx` — Root layout avec fonts

## Design Direction
- Dark premium : `bg-[#0a0a0f]`, cards `bg-white/5 backdrop-blur-xl border border-white/10 rounded-2xl`
- Primary emerald `#10b981` pour accents/CTAs/états actifs
- Gradients subtils pour hero et scores
- Animations : `fade-in-up`, `scale-in`, `shimmer` (déjà dans globals.css)
- Fonts : DM Sans (sans) + Instrument Serif (serif)
- Icônes : lucide-react

---

## Phase 0 : Fondation (mock data + auth bypass)

### Créer
- **`lib/mock-data.ts`** — Données mock centrales typées (MOCK_USER, MOCK_WORKSPACE, MOCK_PROJECTS×3, MOCK_ANALYSIS_RESULT, MOCK_CITATIONS×8, MOCK_RECOMMENDATIONS×6, MOCK_TREND_DATA×8 semaines, MOCK_ENGINE_DATA×5, MOCK_COMPETITOR_DATA×5, MOCK_REPUTATION_REPORT, MOCK_SMC_CONTENTS×4, MOCK_DIRECTORY_ENTRIES×8, MOCK_AFFILIATE_STATS, MOCK_ACTIVITY_FEED×10)
- **`lib/mock-auth.tsx`** — MockAuthProvider + useMockAuth() hook

### Modifier
- **`app/(app)/app/layout.tsx`** — Auth bypass conditionnel (si pas de Supabase env vars → mock auth)
- **`lib/workspace-context.tsx`** — Fallback vers MOCK_WORKSPACE si Supabase échoue
- **`components/providers.tsx`** — Ajouter MockAuthProvider + I18nProvider

---

## Phase 1 : Shell & Navigation

### Modifier
- **`app/(app)/app/sidebar.tsx`** — Réécriture complète :
  - Nav principale : Dashboard, Analyze, Projects, Analytics, Executive
  - Section Generative Tools (collapsible) : 6 sous-items
  - Directory, Recharge, Affiliation
  - Section Settings (collapsible) : billing, api, integrations, team
  - Account en bas
  - Credits display, workspace switcher, language selector
  - Mobile responsive avec hamburger + overlay

---

## Phase 2A : Pages Publiques (parallèle avec Phase 1)

### Modifier
- **`app/(public)/page.tsx`** — Landing page redesign (hero + gradient orbs, features grid, pricing 3 tiers, testimonials, FAQ accordion, footer)
- **`app/(auth)/layout.tsx`** — Layout dark centré
- **`app/(auth)/login/page.tsx`** — Design split/centered, mock auth bypass
- **`app/(auth)/register/page.tsx`** — Même approche + champ nom + checkbox CGU

---

## Phase 2B : Dashboard + Onboarding (après Phase 0+1)

### Modifier
- **`app/(app)/app/page.tsx`** — Dashboard : 4 KPI cards, trend chart, engine chart, activity feed, quick actions
- **`app/(app)/app/onboarding/page.tsx`** — Wizard 3 étapes avec design amélioré

---

## Phase 3A : Module Projects (après Phase 2B)

### Modifier
- **`app/(app)/app/projects/page.tsx`** — Liste projets avec search/filter, grid cards, empty state
- **`app/(app)/app/projects/[id]/layout.tsx`** — Breadcrumb + 5 tabs + header projet
- **`app/(app)/app/projects/[id]/overview/page.tsx`** — KPIs, charts, alerts, scan button mock
- **`app/(app)/app/projects/[id]/citations/page.tsx`** — Citations filtrables par engine
- **`app/(app)/app/projects/[id]/ai-visibility/page.tsx`** — Scores par engine + progress bars
- **`app/(app)/app/projects/[id]/recommendations/page.tsx`** — Filtres + RecommendationCards
- **`app/(app)/app/projects/[id]/packs/page.tsx`** — Content packs avec generate modal

---

## Phase 3B : Analytics + Executive (parallèle avec 3A)

### Modifier
- **`app/(app)/app/analytics/page.tsx`** — Period selector, KPIs, Recharts trend chart, top prompts
- **`app/(app)/app/executive/page.tsx`** — KPIs premium, ROI metrics, charts, executive summary

---

## Phase 4A : Page Analyze (après Phase 1)

### Créer
- **`app/(app)/app/analyze/page.tsx`** — URL input glassmorphism, progress animation mock (crawling→analyzing→scoring→generating), résultats avec ScoreRing + CategoryCards

---

## Phase 4B : Generative Tools (après Phase 1)

### Créer
- **`app/(app)/app/generative-tools/page.tsx`** — Hub : grid 6 cards outil
- **`app/(app)/app/generative-tools/layout.tsx`** — Layout wrapper
- **`app/(app)/app/generative-tools/smc-studio/page.tsx`** — Studio/Library tabs, 3 outils (Fiche/FAQ/Article), génération mock
- **`app/(app)/app/generative-tools/content-writer/page.tsx`** — Multi-step (vertical→type→form→result), 9 verticals, 8 content types
- **`app/(app)/app/generative-tools/reputation/page.tsx`** — Form→Results, sentiment score, mentions, competitor comparison
- **`app/(app)/app/generative-tools/source-of-truth/page.tsx`** — Form entreprise, services dynamiques, FAQs, génération llm.txt
- **`app/(app)/app/generative-tools/schema-org/page.tsx`** — Form + type selector, JSON-LD output
- **`app/(app)/app/generative-tools/robots-txt/page.tsx`** — Platform selector, toggles AI bots, output robots.txt

---

## Phase 4C : Directory (après Phase 1)

### Créer
- **`app/(app)/app/directory/page.tsx`** — Search/filter, grid entries, "Join" CTA
- **`app/(app)/app/directory/[id]/page.tsx`** — Profil entité : score ring, catégories, engines

---

## Phase 4D : Recharge + Affiliation + Account (après Phase 1)

### Créer
- **`app/(app)/app/recharge/page.tsx`** — Credits display, 3 packs (Jerrican/Pompe/Citerne), achat mock
- **`app/(app)/app/affiliation/page.tsx`** — 3 tabs (Overview/Referrals/Payouts), lien affilié, stats
- **`app/(app)/app/account/page.tsx`** — Profil éditable, password, notifications, data export

---

## Phase 5 : Settings (après Phase 1)

### Modifier
- **`app/(app)/app/settings/page.tsx`** — Refresh design
- **`app/(app)/app/settings/billing/page.tsx`** — Refresh design
- **`app/(app)/app/settings/api/page.tsx`** — CRUD clés API + docs
- **`app/(app)/app/settings/integrations/page.tsx`** — Cards GSC/Slack/Zapier

---

## Phase 6 : Polish & Composants Partagés

### Créer
- **`components/page-header.tsx`** — Header réutilisable (title, description, breadcrumbs, actions)
- **`components/kpi-card.tsx`** — Card KPI réutilisable (label, value, change, icon)
- **`components/code-block.tsx`** — Affichage code (JSON-LD, robots.txt) avec copy
- **`components/progress-bar.tsx`** — Barre de progression animée

### Modifier
- **`app/(app)/app/loading.tsx`**, **`error.tsx`**, **`not-found.tsx`** — Design refresh
- **`components/ui/index.ts`** — Ajouter exports des nouveaux composants

---

## Graphe de Dépendances

```
Phase 0 (Fondation)
  ├── Phase 1 (Navigation)
  │     ├── Phase 2B (Dashboard) → Phase 3A (Projects) + Phase 3B (Analytics)
  │     ├── Phase 4A (Analyze)          ┐
  │     ├── Phase 4B (Generative Tools) │ Tous en
  │     ├── Phase 4C (Directory)        │ parallèle
  │     ├── Phase 4D (Recharge/Affil)   │
  │     └── Phase 5 (Settings)          ┘
  └── Phase 2A (Pages Publiques) ← parallèle avec Phase 1

Phase 6 (Polish) — après tout le reste
```

## Totaux
- **~20 fichiers à créer**, **~27 fichiers à modifier** = **~47 fichiers**
- **~30 routes/pages** au total

## Vérification
1. `npm run build` — Le build Next.js doit passer sans erreurs
2. `npm run dev` — Toutes les pages accessibles via navigation sidebar
3. Vérifier chaque route manuellement dans le navigateur
4. Vérifier le responsive (mobile sidebar, bottom nav)
5. Vérifier que tout fonctionne SANS variables d'environnement Supabase/Stripe
