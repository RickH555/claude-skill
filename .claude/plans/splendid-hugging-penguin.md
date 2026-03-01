# Plan : Monitoring IA + Opportunites Strategiques CitedBy.ai

## Contexte

CitedBy.ai est positionne comme generateur de contenu AEO (13 outils), mais manque cruellement de **monitoring** — la capacite de mesurer l'impact des optimisations. Sans monitoring, l'utilisateur ne sait pas si ca marche. Ce plan ajoute 6 features strategiques pour combler ce gap et creer un avantage concurrentiel decisif.

---

## Features a implementer

### COURT TERME (impact rapide)

#### Feature 1 : AI Monitoring Dashboard (NOUVELLE PAGE)
> Tracker les citations de marque sur ChatGPT, Claude, Perplexity, Gemini

**Fichiers a creer :**
- `types/ai-monitoring.ts` — Interfaces MonitoringResult, CitationEntry, EngineResult, CompetitorData, SentimentBreakdown
- `services/ai-monitoring.ts` — Service avec Gemini + fallback simulation deterministe
- `pages/AiMonitoring.tsx` — Page complete avec :
  - Header : nom de marque + URL + selecteur periode
  - 4 KPI cards : Total Citations, Share of Voice, Sentiment, Tendance
  - Graphique citations/temps (Recharts AreaChart)
  - Repartition par moteur IA (BarChart horizontal)
  - Tableau citations recentes (Engine, Query, Position, Date, Sentiment)
  - Section comparaison concurrents
  - Cout : 5 Gas par scan

**Fichiers a modifier :**
- `App.tsx` — Ajouter 'ai-monitoring' au union type, lazy import, routing, sidebar (section PRINCIPAL, entre Dashboard et Monitoring AEO), command palette, breadcrumb
- `i18n/types.ts` + locales — Optionnel (textes hardcodes FR acceptes)

**Complexite : Haute** | ~600 lignes page + ~200 lignes service + ~50 lignes types

---

#### Feature 2 : Score Gratuit de Visibilite IA (sur Home, avant login)
> Outil viral de lead generation accessible sans compte

**Fichiers a modifier :**
- `pages/Home.tsx` — Nouvelle section apres le hero :
  - Formulaire : Nom de marque + URL
  - ScoreRing anime (0-100) avec les 4 piliers E-E-A-T en mini-barres
  - Resume textuel + CTA "Creez votre compte pour l'analyse complete"
  - Reutilise le service `services/answer-first-audit.ts` existant (pas de Gas requis, gratuit)
  - Bandeau "GRATUIT" visible

**Complexite : Moyenne** | ~200 lignes ajoutees a Home.tsx

---

#### Feature 3 : White-Label Agences (onglet dans Account)
> Les agences (plan Agency) personnalisent le branding

**Fichiers a modifier :**
- `pages/Account.tsx` — Nouvel onglet "White Label" (visible uniquement pour plan === 'enterprise') :
  - Formulaire : Logo URL, Nom agence, Couleur primaire, Couleur secondaire
  - Preview en temps reel du branding personnalise
  - Export rapport PDF brande (integration jsPDF existant)
  - Badge "Agency" + "Coming Soon" pour domaine custom
  - Stockage localStorage

**Complexite : Moyenne** | ~250 lignes ajoutees a Account.tsx

---

### MOYEN TERME (differentiation)

#### Feature 4 : Before/After Scoring (historique E-E-A-T)
> Suivre les ameliorations E-E-A-T dans le temps

**Fichiers a modifier :**
- `pages/EeatBuilder.tsx` — Nouvel onglet "Historique" (7eme onglet) :
  - Timeline des audits passes (stockes en localStorage)
  - Cartes Before/After avec deltas visuels (+15 pts, -3 pts)
  - Graphique d'evolution des 4 piliers E-E-A-T (Recharts LineChart)
  - Bouton "Comparer avec le dernier audit"
- `services/eeat/eeat-auditor.ts` — Ajouter fonctions de stockage/comparaison localStorage

**Complexite : Moyenne** | ~200 lignes page + ~50 lignes service

---

#### Feature 5 : Workflow Integre (Audit → Fix automatique)
> Un clic pour passer d'un probleme detecte a sa correction

**Fichiers a modifier :**
- `pages/EeatBuilder.tsx` — Dans l'onglet Audit E-E-A-T, apres resultats :
  - Chaque finding 'missing' ou 'incomplete' recoit un bouton "Corriger"
  - Cliquer navigue vers l'onglet generateur correspondant avec donnees pre-remplies
  - Bandeau workflow en haut : "3 problemes detectes → Tout corriger (11 Gas estimes)"
  - Mapping : experience→Citation Block, expertise→Author Bio, authority→Trust Signals, trust→About Page

**Complexite : Moyenne** | ~150 lignes ajoutees

---

#### Feature 6 : Export CMS (WordPress/Webflow)
> Outputs prets a l'emploi pour les CMS populaires

**Fichiers a modifier :**
- `pages/EeatBuilder.tsx` — Ajouter a chaque panneau de resultats :
  - Bouton "Copier HTML" (existe deja partiellement via CodeBlock)
  - Bouton "Telecharger .html"
  - Selecteur format : "HTML brut" | "WordPress Shortcode" | "Webflow Embed"
  - Chaque format wrappe le contenu genere differemment

**Complexite : Faible** | ~100 lignes, composant reutilisable ExportPanel

---

## Ordre d'implementation

| # | Feature | Dependances | Fichiers |
|---|---------|-------------|----------|
| 1 | Types AI Monitoring | Aucune | `types/ai-monitoring.ts` (NEW) |
| 2 | Service AI Monitoring | #1 | `services/ai-monitoring.ts` (NEW) |
| 3 | Page AI Monitoring | #1, #2 | `pages/AiMonitoring.tsx` (NEW) |
| 4 | Integration Monitoring | #3 | `App.tsx`, `GenerativeTools.tsx` |
| 5 | Score Gratuit Home | Aucune | `pages/Home.tsx` |
| 6 | White-Label Agences | Aucune | `pages/Account.tsx` |
| 7 | Export CMS | Aucune | `pages/EeatBuilder.tsx` |
| 8 | Workflow Integre | Aucune | `pages/EeatBuilder.tsx` |
| 9 | Before/After Scoring | Aucune | `pages/EeatBuilder.tsx`, `services/eeat/eeat-auditor.ts` |

**Parallelisation** : Les features 5, 6, 7, 8, 9 sont independantes et peuvent etre lancees en agents paralleles.

---

## Fichiers critiques

| Fichier | Action | Raison |
|---------|--------|--------|
| `types/ai-monitoring.ts` | CREER | Types pour le monitoring |
| `services/ai-monitoring.ts` | CREER | Service Gemini + simulation |
| `pages/AiMonitoring.tsx` | CREER | Page monitoring complete |
| `App.tsx` (ligne 309) | MODIFIER | Union type, routing, sidebar, command palette, breadcrumb |
| `pages/Home.tsx` | MODIFIER | Section score gratuit |
| `pages/Account.tsx` | MODIFIER | Onglet White Label |
| `pages/EeatBuilder.tsx` | MODIFIER | Onglet historique, workflow, export CMS |
| `services/eeat/eeat-auditor.ts` | MODIFIER | Stockage/comparaison localStorage |

## Patterns a reutiliser

- **Service pattern** : `getApiKey()` → Gemini → `fallbackSimulation()` (voir `services/eeat/eeat-auditor.ts`)
- **Recharts** : Deja utilise dans `Dashboard.tsx` (AreaChart, ResponsiveContainer)
- **Tabs** : Pattern useState activeTab (voir `EeatBuilder.tsx`, `Account.tsx`)
- **ScoreRing** : Composant existant `components/ScoreRing.tsx`
- **CodeBlock avec copy** : Existe dans `EeatBuilder.tsx`
- **Credit deduction** : `onDeductCredits(amount)` prop pattern
- **Sidebar NavItem** : Pattern existant dans App.tsx avec `.includes(currentView)`
- **Lazy loading** : `React.lazy(() => import('./pages/X'))` pattern

## Verification

1. `npm run build` — Pas d'erreurs TypeScript
2. Dev server (`npm run dev`) — Navigation vers chaque nouvelle feature
3. Puppeteer :
   - Home page → voir section Score Gratuit → tester le formulaire
   - Login → Sidebar → cliquer "Monitoring IA" → voir dashboard
   - Account → onglet White Label (uniquement plan Agency)
   - Outils Generatifs → E-E-A-T Builder → onglet Historique
   - E-E-A-T Builder → Audit → resultats → boutons "Corriger"
   - E-E-A-T Builder → generer → boutons export CMS
