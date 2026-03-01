# Plan d'optimisation frontend — CitedBy.ai

## Contexte

Le SPA Vite+React de citedby.ai souffre de plusieurs problemes de performance identifies par audit :
- **Bundle total : ~2.7 MB** (480 KB gzip) dont 960 KB de deps lourdes chargees inutilement
- **Re-renders excessifs** : App.tsx (700 lignes) est un God Component, layout re-render a chaque changement d'etat
- **Fonts CDN bloquantes** : Geist Sans/Mono chargees sans preload
- **Deps mortes** : react-router-dom (43KB) et date-fns (70KB) jamais importees
- **Pas de cache headers** sur Vercel pour les assets statiques
- **Pas de vendor chunking** dans la config Vite

### Contraintes CLAUDE.md
- NE PAS modifier : `services/analyzer.ts`, `services/criteria.ts`, `services/generator.ts`
- NE PAS supprimer de fichiers existants
- NE PAS toucher au backend : `app/`, `lib/`, `supabase/`, `middleware.ts`, `next.config.mjs`

---

## Phase 1 — Quick wins bundle (impact immediat, risque zero)

### 1.1 Supprimer les deps mortes de package.json
- **Fichier** : `package.json`
- Retirer `react-router-dom` (jamais importe, routing custom dans App.tsx)
- Retirer `date-fns` (aucun import trouve dans le codebase)
- **Gain** : ~113 KB retires du bundle

### 1.2 Lazy-loader Login et Analyze
- **Fichier** : `App.tsx` (lignes 6-8)
- Convertir les imports directs en `lazy()` :
  ```
  Home → reste direct (landing page, critique pour LCP)
  Login → lazy(() => import('./pages/Login'))
  Analyze → lazy(() => import('./pages/Analyze'))
  ```
- **Gain** : Reduit le chunk initial de ~30-50 KB

---

## Phase 2 — Dynamic imports des deps lourdes (~960 KB)

### 2.1 jsPDF + html2canvas en import dynamique
- **Fichiers** : `pages/Results.tsx`, `pages/Account.tsx`, `pages/Plan.tsx`, `pages/Recharge.tsx`
- Remplacer `import jsPDF from 'jspdf'` par un `await import('jspdf')` dans la fonction d'export PDF
- Idem pour `html2canvas`
- **Gain** : -586 KB retires du critical path (charges uniquement au clic "Exporter PDF")

### 2.2 Recharts en import dynamique (optionnel)
- **Fichiers** : `pages/EeatBuilder.tsx`, `pages/AiMonitoring.tsx`, `pages/BotTracking.tsx`
- Ces pages sont deja lazy-loaded, donc recharts est deja dans un chunk separe
- **Action** : Verifier que Vite split bien recharts. Si ce n'est pas le cas, le forcer via manualChunks (Phase 3)

---

## Phase 3 — Vite build optimization

### 3.1 Ajouter manualChunks dans vite.config.ts
- **Fichier** : `vite.config.ts`
```ts
build: {
  target: 'es2020',
  chunkSizeWarningLimit: 500,
  rollupOptions: {
    output: {
      manualChunks: {
        'vendor-react': ['react', 'react-dom'],
        'vendor-supabase': ['@supabase/supabase-js', '@supabase/ssr'],
        'vendor-pdf': ['jspdf', 'html2canvas'],
        'vendor-charts': ['recharts'],
        'vendor-ai': ['@google/genai'],
        'vendor-ui': ['lucide-react', 'clsx', 'tailwind-merge', 'sonner'],
      }
    }
  }
}
```
- **Gain** : Meilleur cache long-terme (les vendors changent rarement), chunks paralleles

---

## Phase 4 — Optimisation re-renders React

### 4.1 React.memo sur les composants layout
- **Fichiers** :
  - `components/layout/AppShell.tsx` → `export default React.memo(AppShell)`
  - `components/layout/TopNav.tsx` → `export default React.memo(TopNav)`
  - `components/layout/Sidebar.tsx` → `export default React.memo(Sidebar)`
- **Impact** : Le layout ne re-render plus quand l'etat interne d'App.tsx change (toasts, analysisData, etc.)

### 4.2 Stabiliser les callbacks inline dans App.tsx
- **Fichier** : `App.tsx`
- Convertir les 8+ callbacks inline en useCallback :
  ```ts
  const handleNavigateToAnalysis = useCallback(() => navigateTo('new-analysis'), []);
  const handleNavigateToCredits = useCallback(() => navigateTo('credits'), []);
  const handleNavigateToPlan = useCallback(() => {
    setInitialAccountTab('subscription');
    navigateTo('account');
  }, []);
  ```
- Remplacer `onNavigate={(view) => navigateTo(view)}` par `onNavigate={navigateTo}` (navigateTo est deja stable)

### 4.3 Supprimer le double-key inutile
- **Fichier** : `App.tsx`
- Le pattern `key={refreshKey}` sur chaque page + `key={pageKey}` sur le wrapper = double destruction
- Garder SEULEMENT `key={currentView}` sur le wrapper `<div>` de Suspense (change de page = remount propre)
- Supprimer tous les `key={...refreshKey...}` individuels et les states `refreshKey`/`pageKey`
- **Impact** : Un seul remount par navigation au lieu de deux, plus de remount intempestif lors de `goBack()`

---

## Phase 5 — Optimisation reseau

### 5.1 Fonts : desynchroniser le chargement Geist
- **Fichier** : `index.html`
- Ajouter `preconnect` pour jsdelivr.net
- Charger Geist fonts en async (media="print" + onload trick) :
  ```html
  <link rel="preconnect" href="https://cdn.jsdelivr.net" crossorigin />
  <link rel="preload" href="https://cdn.jsdelivr.net/npm/geist@1.3.1/dist/fonts/geist-sans/style.min.css" as="style" />
  <link href="https://cdn.jsdelivr.net/npm/geist@1.3.1/dist/fonts/geist-sans/style.min.css" rel="stylesheet" media="print" onload="this.media='all'" />
  ```

### 5.2 Cache headers dans vercel.json
- **Fichier** : `vercel.json`
- Ajouter des regles de cache pour les assets hashes :
  ```json
  {
    "source": "/assets/(.*)",
    "headers": [
      { "key": "Cache-Control", "value": "public, max-age=31536000, immutable" }
    ]
  }
  ```
- HTML : `Cache-Control: public, max-age=0, must-revalidate`

### 5.3 CSP : ajouter jsdelivr.net aux sources de style/font
- **Fichier** : `vercel.json`
- Ajouter `https://cdn.jsdelivr.net` a `style-src` et `font-src` dans le header CSP

---

## Phase 6 — Reduire la taille des fonts chargees

### 6.1 Reduire les poids Inter
- **Fichier** : `index.html`
- Inter charge 5 poids (400,500,600,700,800). Verifier si 800 est utilise.
- Si non, retirer `800` du parametre Google Fonts : economise ~15 KB
- Playfair Display est peu utilise (titres serif). Evaluer si on le garde.

---

## Verification

1. `npm run build` → comparer tailles avant/apres (noter les chunks > 200KB)
2. `npm run preview` → verifier que toutes les pages chargent correctement
3. DevTools > Network : verifier le chargement lazy des chunks PDF au clic export
4. DevTools > Performance : profiler un changement de page, verifier que AppShell/TopNav ne re-render pas
5. Lighthouse : viser LCP < 2.5s, TBT < 200ms, CLS < 0.1

## Resultats attendus

| Metrique | Avant | Apres (estime) |
|----------|-------|-----------------|
| Bundle initial (gzip) | ~480 KB | ~250-300 KB |
| Chunks > 500KB | 3 | 0 |
| Re-renders/navigation | 3+ (layout+page+wrapper) | 1 (page uniquement) |
| FCP | ~1.5s | ~1.0s |
| LCP | ~2.5s | ~1.8s |
