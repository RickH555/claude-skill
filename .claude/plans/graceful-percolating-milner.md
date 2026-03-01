# Plan : Mode sombre / clair (Dark Mode Toggle)

## Contexte

L'app web AstroLab utilise deja des CSS custom properties (`:root`) pour toutes les couleurs structurelles (`--color-bg`, `--color-surface`, `--color-text`, etc.). Les composants les referent via `var(--color-*)`. Il n'y a ni `darkMode` dans Tailwind, ni ThemeContext, ni localStorage. L'ajout du dark mode consiste a : definir un jeu de variables sombres sous `html.dark`, creer un ThemeContext avec persistance localStorage + detection systeme, et placer un toggle dans les deux navs.

## Strategie

**CSS variables pures** (pas `darkMode: 'class'` de Tailwind). Les composants utilisent deja `var(--color-*)` partout — on redefinit ces variables sous `html.dark` et tout bascule automatiquement sans toucher au JSX de chaque composant. On ajoute 4 nouvelles variables structurelles (`--color-border`, `--color-hover`, `--color-active`, `--color-skeleton`) pour remplacer les couleurs Tailwind hardcodees (slate-200, slate-50, indigo-50) dans les composants UI de base.

## Fichiers a creer (2)

### 1. `apps/web/src/contexts/ThemeContext.tsx` (NOUVEAU)
- `ThemeContext` + `ThemeProvider` suivant le pattern de AuthContext
- State `theme: 'light' | 'dark'`
- `getInitialTheme()` : localStorage d'abord, sinon `prefers-color-scheme`, sinon `light`
- `applyTheme()` : ajoute/retire `dark` sur `document.documentElement`
- `toggleTheme()` : bascule + sauvegarde localStorage + classe transition 300ms
- Ecoute `prefers-color-scheme` changes (si pas de choix manuel)

### 2. `apps/web/src/hooks/useTheme.ts` (NOUVEAU)
- Pattern identique a `useAuth.ts` (ligne 1-10)
- `useContext(ThemeContext)` + throw si null

## Fichiers a modifier (10)

### 3. `apps/web/src/styles/globals.css`
- Ajouter dans `:root` : `--color-border: #e2e8f0`, `--color-hover: #f1f5f9`, `--color-active: #eef2ff`, `--color-skeleton: #e2e8f0`
- Ajouter bloc `html.dark { ... }` avec toutes les variables overridees :
  - `--color-primary: #818cf8`, `--color-primary-dark: #6366f1`
  - `--color-bg: #0f172a`, `--color-surface: #1e293b`
  - `--color-text: #f1f5f9`, `--color-text-muted: #94a3b8`
  - `--color-border: #334155`, `--color-hover: #334155`, `--color-active: #312e81`, `--color-skeleton: #334155`
- Ajouter classe `.theme-transition` (transition 300ms sur bg/color/border)

### 4. `apps/web/index.html`
- Ajouter script inline synchrone dans `<head>` pour prevention FOUC (flash) :
  ```html
  <script>
    (function(){var t=localStorage.getItem('astrolab-theme');
    if(t==='dark'||(!t&&matchMedia('(prefers-color-scheme:dark)').matches))
    document.documentElement.classList.add('dark')})();
  </script>
  ```

### 5. `apps/web/src/App.tsx`
- Import `ThemeProvider`
- Wrapper le plus externe : `<ThemeProvider>` > `<AuthProvider>` > ...

### 6. `apps/web/src/components/layout/MerchantNav.tsx`
- Import `useTheme` + `Toggle`
- Ajouter toggle dans le bloc desktop sidebar (bottom, avant businessName)
- Ajouter toggle dans le menu overlay mobile (apres les nav items)
- Remplacer : `border-slate-200` -> `border-[var(--color-border)]` (5 occurrences lignes 45,46,71,90,105,167)
- Remplacer : `bg-indigo-50` -> `bg-[var(--color-active)]` (lignes 62,79,153)
- Remplacer : `hover:bg-slate-50` -> `hover:bg-[var(--color-hover)]` (lignes 63,80)

### 7. `apps/web/src/components/layout/CustomerNav.tsx`
- Import `useTheme` + `Toggle`
- Ajouter toggle desktop : avant l'avatar dans le `div` droite (ligne 55)
- Ajouter toggle mobile : avant l'avatar dans le `div` droite (ligne 76)
- Remplacer : `border-slate-200` -> `border-[var(--color-border)]` (lignes 32,70,87)

### 8. `apps/web/src/components/ui/Toggle.tsx`
- `bg-slate-300` -> `bg-[var(--color-border)]` (track desactive, ligne 16)

### 9. `apps/web/src/components/ui/Button.tsx`
- `hover:bg-indigo-50` -> `hover:bg-[var(--color-active)]` (outline variant)

### 10. `apps/web/src/components/ui/Input.tsx`
- `border-slate-300` -> `border-[var(--color-border)]`

### 11. `apps/web/src/components/ui/Card.tsx`
- Ajouter `border border-[var(--color-border)]` pour definition en dark mode

### 12. `apps/web/src/components/ui/Tabs.tsx`
- `bg-slate-100` -> `bg-[var(--color-hover)]`
- `bg-white` -> `bg-[var(--color-surface)]`

## Hors scope (phase suivante)
- Badge.tsx, Toast.tsx : couleurs semantiques (green/red/amber) — fonctionnent acceptablement en dark
- Graphiques Recharts : theming interne de la lib
- Autres pages avec `border-slate-*` hardcode (MerchantClients, etc.)

## Verification
1. `pnpm build` — zero erreur TypeScript
2. Preview : landing page en mode clair → toggle → mode sombre → rafraichir → persiste
3. Login merchant → sidebar : toggle visible, bascule fonctionne
4. Login customer → navbar : toggle visible, bascule fonctionne
5. FOUC : rafraichir en dark mode → pas de flash blanc
6. System pref : supprimer localStorage, changer pref OS → suit automatiquement
