# Plan : CitedBy.ai Public Version

## Contexte

Creer un nouveau SaaS "CitedBy.ai Public" a `C:\Users\User\citedby-public\` — version grand public du SaaS PRO existant (`citedby-ai`). 9 modules au lieu de 36. Meme backend Supabase (auth + DB partagees). Design degrade jaune metallique / noir avec reflets.

## Decisions

- **Emplacement** : `C:\Users\User\citedby-public\` (dossier separe)
- **Backend** : Meme projet Supabase (auth + tables partagees)
- **Marketplace** : Annuaire d'agences + place de marche de services combines
- **Theme** : Gold/amber (#DAA520, #FFD700, #B8860B) sur noir (#0a0a0a, #111111)

## 9 Modules

1. Dashboard — KPIs, chart tendance, audits recents, actions rapides
2. Nouvel Audit — Formulaire URL + animation 4 etapes
3. Analyse URL (Results) — Scores, categories, recommandations
4. Monitoring AEO — Scan 5 moteurs IA + scheduling (sans retargeting)
5. Alertes AEO — Liste alertes avec severite, filtres, reglages
6. Marketplace Agence — **NOUVEAU** : annuaire agences certifiees + achat de services AEO
7. Affiliation — Lien affilie, stats, referrals, payouts, assets marketing
8. GTO Gas (Credits) — 3 packs, Stripe checkout, facture PDF
9. Compte & Plan — Profil, abonnement, facturation (sans whitelabel)

## Arborescence fichiers

```
C:\Users\User\citedby-public\
├── .env.example / .env.local
├── .gitignore
├── index.html
├── index.tsx
├── App.tsx                    (simplifie : 11 views au lieu de 36)
├── types.ts                   (trimme : User, AnalysisResult, Recommendation, CategoryScore)
├── globals.css                (theme gold/noir + .gold-reflection)
├── tailwind.config.ts         (palette gold, animations shimmer/pulse/reflection)
├── postcss.config.cjs
├── vite.config.ts             (port 5174)
├── tsconfig.json
├── package.json
│
├── components/
│   ├── Navbar.tsx             (adapte gold, moins d'items)
│   ├── Sidebar.tsx            (3 sections, 9 items, gold active)
│   ├── MobileBottomNav.tsx    (4 items)
│   ├── AppFooter.tsx          (gold)
│   ├── ToastSystem.tsx        (copie directe)
│   ├── ScoreRing.tsx          (couleurs gold)
│   ├── StripeCheckout.tsx     (gold)
│   ├── OnboardingModal.tsx    (simplifie, gold)
│   ├── CommandPalette.tsx     (moins de commandes)
│   ├── ShortcutsModal.tsx     (gold)
│   ├── GoldCard.tsx           (NOUVEAU : carte reutilisable bordure gold)
│   ├── GoldButton.tsx         (NOUVEAU : bouton gradient gold + shimmer)
│   └── ShimmerEffect.tsx      (NOUVEAU : overlay reflet gold)
│
├── pages/
│   ├── Home.tsx               (landing gold avec widget audit gratuit)
│   ├── Login.tsx              (auth gold)
│   ├── Dashboard.tsx          (4 KPIs gold, chart, audits recents)
│   ├── Analyze.tsx            (formulaire URL, animation 4 etapes)
│   ├── Results.tsx            (scores, categories, recommandations)
│   ├── AiMonitoring.tsx       (5 moteurs IA + scheduling, sans retargeting)
│   ├── AlertsAEO.tsx          (alertes avec severite)
│   ├── MarketplaceAgence.tsx  (NOUVEAU : 3 onglets agences/services/achats)
│   ├── Affiliation.tsx        (3 onglets overview/referrals/payouts)
│   ├── Recharge.tsx           (3 packs GTO Gas)
│   └── Account.tsx            (3 onglets profil/abo/facturation)
│
├── services/
│   ├── analyzer.ts            (copie — NE PAS MODIFIER)
│   ├── criteria.ts            (copie — NE PAS MODIFIER)
│   ├── generator.ts           (copie — NE PAS MODIFIER)
│   ├── database.ts            (copie + clef 'citedby_public_db')
│   ├── ai-monitoring.ts       (copie)
│   ├── scheduling.ts          (copie)
│   ├── answer-first-audit.ts  (copie)
│   └── marketplace.ts         (NOUVEAU : mock agences/services/reviews)
│
├── types/
│   ├── ai-monitoring.ts       (copie)
│   ├── scheduling.ts          (copie)
│   └── marketplace.ts         (NOUVEAU : Agency, MarketplaceService, Review, Purchase)
│
├── i18n/
│   ├── index.ts, context.tsx  (copie + clef 'citedby_public_locale')
│   ├── types.ts               (adapte : -PRO keys, +marketplace keys)
│   ├── LanguageSelector.tsx   (gold)
│   └── locales/ fr.ts, en.ts, es.ts  (adapte)
│
└── lib/
    └── supabase.ts            (copie de citedby-ai/src/lib/supabase.ts)
```

## Design System Gold/Noir

### Palette Tailwind (`tailwind.config.ts`)

| Token | Valeur | Usage |
|-------|--------|-------|
| `gold-300` | #FFD700 | Pure gold, accents forts |
| `gold-400` | #DAA520 | Goldenrod, primary |
| `gold-500` | #B8860B | Dark goldenrod, gradient mid |
| `gold-600` | #8B6914 | Deep gold, gradient end |
| `surface` | #0a0a0a | Background principal |
| `surface-100` | #111111 | Cards |
| `surface-300` | #1c1c1c | Surfaces elevees |
| `primary` | #DAA520 | Semantic primary |

### Animations Tailwind

- `gold-shimmer` : sweep horizontal gold (3s infinite)
- `gold-pulse` : box-shadow gold pulse (2s infinite)
- `reflection` : balayage diagonal gold (4s infinite)

### Classes CSS (`globals.css`)

- `.btn-gradient` : `linear-gradient(135deg, #DAA520, #B8860B, #8B6914)` + pseudo-element shimmer hover
- `.gradient-text` : `linear-gradient(135deg, #FFD700, #DAA520, #B8860B)` clip text
- `.ninja-card` : `bg-[#111111]` + `border-amber-500/[0.08]` + hover gold glow
- `.gold-reflection` : pseudo-element `::before` avec sweep gold diagonal
- `#scroll-progress` : gradient gold

## Matrice Copie vs Adaptation

### Copie directe (12 fichiers)
`postcss.config.cjs`, `services/analyzer.ts`, `services/criteria.ts`, `services/generator.ts`, `services/ai-monitoring.ts`, `services/scheduling.ts`, `services/answer-first-audit.ts`, `types/ai-monitoring.ts`, `types/scheduling.ts`, `components/ToastSystem.tsx`, `i18n/index.ts`

### Copie + edit mineur (5 fichiers)
`services/database.ts` (DB_KEY), `i18n/context.tsx` (LOCALE_KEY), `index.tsx`, `vite.config.ts` (port), `tsconfig.json`

### Adaptation majeure (theme + simplification — 20 fichiers)
`App.tsx`, `globals.css`, `tailwind.config.ts`, `types.ts`, toutes les pages et composants layout

### Fichiers entierement nouveaux (6 fichiers)
`pages/MarketplaceAgence.tsx`, `services/marketplace.ts`, `types/marketplace.ts`, `components/GoldCard.tsx`, `components/GoldButton.tsx`, `components/ShimmerEffect.tsx`

## MarketplaceAgence.tsx — Design detaille

### 3 onglets
1. **Agences Certifiees** — Grille de cartes agences (logo, nom, certification or/argent/bronze, rating etoiles, specialites, projets completes)
2. **Services AEO** — Cartes services (nom, agence, description, delai, prix, categorie, tags, rating)
3. **Mes Achats** — Tableau historique (service, agence, date, statut, montant)

### 2 modals (createPortal)
- **Profil Agence** : logo, certification, about, specialites, services proposes, avis clients, statistiques
- **Detail Service** : description, inclus, options standard/premium, avis, bouton achat

### Types (`types/marketplace.ts`)
```
Agency { id, name, logo, description, website, certification: gold|silver|bronze, specialties[], rating, reviewCount, completedProjects, satisfactionRate, avgDeliveryDays, memberSince, featured }
MarketplaceService { id, agencyId, agencyName, name, description, includes[], category: audit|optimization|content|monitoring|strategy|training, price, premiumPrice?, deliveryDays, tags[], rating, reviewCount, popular }
MarketplaceReview { id, serviceId, agencyId, authorName, rating, comment, date }
Purchase { id, serviceId, serviceName, agencyId, agencyName, date, amount, status: pending|in_progress|delivered|completed|refunded, option: standard|premium }
```

## Sidebar Navigation (Public)

### Principal
- Accueil → `home`
- Nouvel Audit → `analyze`
- Dashboard → `dashboard`
- Monitoring AEO → `monitoring`

### Marketplace
- Agences & Services → `marketplace`

### Mon Compte
- GTO Gas → `recharge`
- Affiliation → `affiliation`
- Alertes AEO → `alerts`
- Compte & Plan → `account`

## Ordre d'implementation

### Phase 1 : Infrastructure (~15 fichiers)
1. Creer dossier + `package.json` + `npm install`
2. Config : `vite.config.ts`, `tsconfig.json`, `postcss.config.cjs`, `tailwind.config.ts`
3. `index.html`, `index.tsx`, `globals.css`
4. `.env.example`, `.gitignore`

### Phase 2 : Types + Services + i18n (~15 fichiers)
5. `types.ts` + `types/marketplace.ts` + `types/ai-monitoring.ts` + `types/scheduling.ts`
6. Copier services (7 fichiers)
7. Creer `services/marketplace.ts` (mock data)
8. Systeme i18n complet (6 fichiers)

### Phase 3 : Composants partages (~13 fichiers)
9. `GoldCard.tsx`, `GoldButton.tsx`, `ShimmerEffect.tsx`
10. `ToastSystem.tsx`, `ScoreRing.tsx`, `StripeCheckout.tsx`
11. `Navbar.tsx`, `Sidebar.tsx`, `MobileBottomNav.tsx`, `AppFooter.tsx`
12. `OnboardingModal.tsx`, `CommandPalette.tsx`, `ShortcutsModal.tsx`

### Phase 4 : App.tsx + Auth (~3 fichiers)
13. `App.tsx` (orchestrateur 11 views)
14. `pages/Login.tsx`
15. `pages/Home.tsx`
→ Verifier : `npm run dev` fonctionne, login mock OK

### Phase 5 : Pages coeur (~5 fichiers)
16. `pages/Analyze.tsx`
17. `pages/Results.tsx`
18. `pages/Dashboard.tsx`
19. `pages/AiMonitoring.tsx`
20. `pages/AlertsAEO.tsx`

### Phase 6 : Pages gestion + Marketplace (~4 fichiers)
21. `pages/Recharge.tsx`
22. `pages/Account.tsx`
23. `pages/Affiliation.tsx`
24. `pages/MarketplaceAgence.tsx` (le plus gros nouveau fichier)

### Phase 7 : Build + Verification
25. `npm run build` → 0 erreurs
26. Tester navigation complete
27. Tester responsive (mobile sidebar, bottom nav)

## Verification

1. `npm run build` → 0 erreurs TypeScript
2. `npm run dev` → toutes les pages accessibles
3. Pour chaque module :
   - Navigation OK depuis sidebar
   - Theme gold coherent (pas de traces purple)
   - Modals centrees, scroll lock, createPortal
   - Responsive mobile (bottom nav, sidebar drawer)
4. Marketplace specifiquement :
   - Recherche + filtres fonctionnels
   - Modal profil agence s'ouvre correctement
   - Modal detail service s'ouvre correctement
   - Flow d'achat simule fonctionne
