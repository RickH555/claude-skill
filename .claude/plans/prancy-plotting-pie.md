# Plan : CitedBy.io — Refonte UI style Frase.io

## Contexte

L'utilisateur veut creer un nouveau projet **citedby.io** dans `C:\Users\User\citedby-io` qui :
- Reprend **100% des fonctionnalites** de citedby.ai (`C:\Users\User\citedby-ai`)
- Adopte l'**interface Frase.io** : sidebar gauche unique, layout epure, panneau contextuel droit
- Support **double mode** (light + dark) avec toggle
- Reorganise la navigation de maniere logique et coherente

### Decisions validees
- **Theme** : Double mode (light + dark) avec toggle
- **Layout** : Sidebar gauche fixe + panneau droit contextuel sur TOUTES les pages
- **Navigation** : Sidebar unique style Frase (pas de TopNav), header minimal (logo, recherche, credits, profil)

---

## Phase 1 — Scaffolding du projet

### 1.1 Copier le projet source
- Copier `C:\Users\User\citedby-ai` → `C:\Users\User\citedby-io`
- Renommer dans `package.json` : `"name": "citedby-io"`
- Mettre a jour les references de nom dans `index.html`, `vite.config.ts`

### 1.2 Installer les dependances supplementaires
- `npm install @headlessui/react` — composants accessibles (toggle, menus)
- `npm install class-variance-authority` — variants de composants pour les 2 themes

---

## Phase 2 — Systeme de theme (Light + Dark)

### 2.1 Creer le systeme de theming
**Fichier** : `globals.css` (refonte)

```
:root (light mode par defaut, style Frase.io) :
  --bg-base:        #FAFAF8    (creme chaud)
  --bg-surface:     #FFFFFF    (cards blanches)
  --bg-elevated:    #F5F5F3    (inputs, hover)
  --bg-sidebar:     #FFFFFF    (sidebar blanche)
  --text-primary:   #1A1A1A    (noir encre)
  --text-secondary: #6B7280    (gris moyen)
  --accent:         #059469    (forest green Frase)
  --border-default: #E5E7EB    (gris clair)

.dark :
  --bg-base:        #09090B    (zinc-950, actuel)
  --bg-surface:     #18181B    (zinc-900)
  --bg-elevated:    #27272A    (zinc-800)
  --bg-sidebar:     #0F0F11
  --text-primary:   #FAFAFA    (blanc)
  --text-secondary: #A1A1AA
  --accent:         #10B981    (emerald)
  --border-default: #27272A
```

### 2.2 Theme Context
**Fichier** : `lib/theme-context.tsx` (NOUVEAU)
- `ThemeProvider` avec `useContext`
- Persistance dans `localStorage('citedby-theme')`
- Detection `prefers-color-scheme` au premier chargement
- Toggle button dans le header

### 2.3 Mettre a jour `tailwind.config.ts`
- `darkMode: 'class'` (deja configure)
- Ajouter les couleurs du light mode
- Toutes les classes Tailwind utiliseront `dark:` prefix

---

## Phase 3 — Nouveau layout (style Frase.io)

### 3.1 Nouveau AppShell
**Fichier** : `components/layout/AppShell.tsx` (refonte)

```
┌─────────────────────────────────────────────────┐
│  Header minimal (h-12)                          │
│  [Logo citedby.io] [Search ⌘K] [Credits] [👤]  │
├────────┬────────────────────────┬───────────────┤
│        │                        │               │
│ Side-  │   Main Content Area    │  Right Panel  │
│ bar    │   (scrollable)         │  (contextuel) │
│ (w-56) │                        │  (w-80)       │
│        │                        │               │
│ Logo   │                        │               │
│ Nav    │                        │               │
│ groups │                        │               │
│        │                        │               │
│ Plan   │                        │               │
│ footer │                        │               │
│        │                        │               │
├────────┴────────────────────────┴───────────────┤
```

### 3.2 Nouveau Header minimal
**Fichier** : `components/layout/Header.tsx` (NOUVEAU, remplace TopNav.tsx)
- Logo "CitedBy.io" a gauche
- Barre de recherche ⌘K au centre
- Credits + toggle theme + avatar a droite
- Hauteur : 48px, border-bottom subtil
- Pas de tabs de section (tout dans la sidebar)

### 3.3 Nouvelle Sidebar unique
**Fichier** : `components/layout/Sidebar.tsx` (refonte complete)

Structure de navigation reorganisee :

```
SIDEBAR CITEDBY.IO
─────────────────────
[Logo CitedBy.io]

📊 OVERVIEW
  └ Dashboard
  └ Projets

🔍 ANALYSE
  └ Nouvel audit
  └ Resultats
  └ Historique
  └ Plan d'action

📡 CITATIONS IA
  └ Monitoring IA
  └ Simulateur
  └ AI Response Layer
  └ Tracking Bots
  └ Alertes AEO
  └ Citations multilingues

✏️ OUTILS
  └ AEO Studio (SMC)
  └ Blog Optimizer
  └ Redacteur IA
  └ Answer First Audit
  └ E-E-A-T Builder
  └ Reputation

🔧 TECHNIQUE
  └ Schema.org
  └ Robots.txt
  └ Source of Truth
  └ LLMs.txt Generator

🏪 MARKETPLACE
  └ Publishers
  └ Annuaire
  └ Clients
  └ Messages
  └ Commandes

─────────────────────
[Credits: 142/200]
[Plan Pro]
[⚙ Parametres]
```

- Sections collapsibles (clic sur le titre pour ouvrir/fermer)
- Icones Lucide pour chaque item
- Indicateur actif : barre laterale bleue/verte + fond surligne
- Footer : credits avec barre de progression + lien vers settings

### 3.4 Panneau contextuel droit
**Fichier** : `components/layout/RightPanel.tsx` (NOUVEAU)
- Se montre/cache selon la page active
- Utilise sur les pages d'audit (resultats, details), outils (preview), etc.
- Slide-in animation depuis la droite
- Bouton de fermeture
- Largeur : 320px (w-80)

---

## Phase 4 — Refactoring du routing (App.tsx)

### 4.1 Simplifier la navigation
- Supprimer le systeme 2 niveaux (FIRST_PAGE_PER_SECTION, PAGE_TO_SECTION)
- Passer a un routing plat : `currentView` pointe directement vers la page
- La sidebar gere elle-meme les groupes et l'etat actif
- Conserver le lazy loading de toutes les pages

### 4.2 Adapter les callbacks
- `onSectionChange` → supprime (plus de sections TopNav)
- `onPageChange(pageId)` → reste le seul mecanisme de navigation
- `activeSection` → derive automatiquement du `currentView` courant dans la sidebar

---

## Phase 5 — Adaptation des pages existantes

### 5.1 Pages a adapter au nouveau layout
Chaque page doit :
- Utiliser les CSS variables du theme (pas de couleurs hardcodees)
- Remplacer `bg-[#1a1d21]` → `bg-[var(--bg-surface)]`
- Remplacer `text-white` → `text-[var(--text-primary)]`
- Remplacer `border-white/5` → `border-[var(--border-default)]`
- Ajouter `dark:` variants pour les cas specifiques

### 5.2 Pages prioritaires (adapter en premier)
1. `Dashboard.tsx` + ses 4 composants
2. `Analyze.tsx` (saisie URL)
3. `Results.tsx` (resultats audit — panneau droit pour details)
4. `AiMonitoring.tsx` (monitoring — panneau droit pour graphiques)
5. `GenerativeTools.tsx` → fusionner dans la sidebar (plus besoin de hub)

### 5.3 Pages secondaires (meme pattern)
- Toutes les autres pages suivent le meme refactoring CSS

---

## Phase 6 — Composants UI

### 6.1 Composants a creer/adapter
- `components/ui/ThemeToggle.tsx` (NOUVEAU) — Switch light/dark
- `components/ui/SearchBar.tsx` (NOUVEAU) — Barre de recherche integree au header
- `components/ui/CollapsibleSection.tsx` (NOUVEAU) — Pour les groupes sidebar
- `components/ui/Card.tsx` — Adapter pour les 2 themes
- `components/ui/Button.tsx` — Adapter pour les 2 themes
- `components/ui/Badge.tsx` — Adapter pour les 2 themes

### 6.2 Branding
- Renommer "CitedBy.ai" → "CitedBy.io" dans tout le projet
- Logo dans TopNav/Sidebar : "Cited" (bold) + "By" (accent) + ".io"

---

## Phase 7 — Renommage et nettoyage

- Renommer toutes les occurrences "citedby.ai" → "citedby.io"
- Mettre a jour `index.html` : `<title>CitedBy.io</title>`
- Mettre a jour les meta tags, favicon references
- Supprimer `components/layout/TopNav.tsx` (remplace par Header.tsx)
- Nettoyer les CSS inutilisees dans globals.css

---

## Fichiers critiques a modifier

| Fichier | Action |
|---------|--------|
| `package.json` | Renommer, ajouter deps |
| `index.html` | Titre, meta |
| `globals.css` | Refonte complete du design system |
| `tailwind.config.ts` | Couleurs light mode |
| `App.tsx` | Simplifier routing, supprimer 2-level nav |
| `components/layout/AppShell.tsx` | Refonte layout |
| `components/layout/TopNav.tsx` | SUPPRIMER → Header.tsx |
| `components/layout/Sidebar.tsx` | Refonte complete |
| `components/layout/Header.tsx` | NOUVEAU |
| `components/layout/RightPanel.tsx` | NOUVEAU |
| `lib/theme-context.tsx` | NOUVEAU |
| `components/ui/ThemeToggle.tsx` | NOUVEAU |
| `components/ui/CollapsibleSection.tsx` | NOUVEAU |
| Toutes les pages `pages/*.tsx` | CSS variables au lieu de couleurs hardcodees |

---

## Ordre d'execution

1. **Copier le projet** (Phase 1)
2. **Theme system** (Phase 2) — globals.css + ThemeProvider + tailwind
3. **Layout** (Phase 3) — Header + Sidebar + AppShell + RightPanel
4. **Routing** (Phase 4) — Simplifier App.tsx
5. **Pages prioritaires** (Phase 5.2) — Dashboard, Analyze, Results, Monitoring
6. **Composants UI** (Phase 6) — Cards, Buttons, Badges adaptes
7. **Pages secondaires** (Phase 5.3) — Toutes les autres pages
8. **Renommage** (Phase 7) — citedby.ai → citedby.io partout

---

## Verification

1. `npm run build` — Build sans erreur
2. `npm run dev` — Serveur de dev fonctionnel
3. Verification visuelle :
   - Toggle light/dark fonctionne
   - Sidebar navigation complete (toutes les 36 pages accessibles)
   - Layout correct sur desktop (sidebar + main + panel droit)
   - Layout correct sur mobile (sidebar overlay)
   - Toutes les fonctionnalites preservees (audit, credits, generation, etc.)
4. `npm test` — Tests existants passent toujours
