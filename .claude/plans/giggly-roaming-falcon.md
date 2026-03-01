# Plan — Structure des pages et composants de CitedBy.ai

## Contexte

CitedBy.ai est un SaaS de suivi de visibilité de marque dans les réponses IA (ChatGPT, Claude, Perplexity, etc.). Le projet est initialisé avec Vite + React 19 + TypeScript strict + Tailwind v4 + Supabase + Framer Motion + Recharts. Actuellement, tout le code vit dans `App.tsx` avec deux pages inline. Il faut créer une architecture de fichiers propre et scalable.

## Structure de fichiers à créer

```
src/
├── components/
│   ├── ui/                          # Composants UI réutilisables
│   │   ├── Button.tsx
│   │   ├── Card.tsx
│   │   └── Input.tsx
│   └── layout/                      # Mise en page
│       ├── AuthLayout.tsx           # Layout centré pour login/register/forgot
│       ├── DashboardLayout.tsx      # Layout sidebar + header + <Outlet />
│       ├── Sidebar.tsx              # Navigation latérale
│       └── Header.tsx               # Barre supérieure
├── pages/
│   ├── auth/
│   │   ├── LoginPage.tsx            # Connexion
│   │   ├── RegisterPage.tsx         # Inscription
│   │   └── ForgotPasswordPage.tsx   # Mot de passe oublié
│   └── app/
│       ├── DashboardPage.tsx        # Tableau de bord (métriques clés)
│       ├── MonitoringPage.tsx       # Suivi des citations IA
│       ├── AnalyticsPage.tsx        # Graphiques détaillés (Recharts)
│       └── SettingsPage.tsx         # Paramètres du compte
├── config/
│   └── supabase.ts                  # (existe déjà, inchangé)
├── App.tsx                          # Refactorisé : routes imbriquées
├── main.tsx                         # Inchangé
└── index.css                        # Inchangé
```

## Détail de chaque fichier

### 1. Composants UI (`src/components/ui/`)

**Button.tsx** — Bouton réutilisable avec variantes `primary` | `secondary` | `ghost`, taille `sm` | `md` | `lg`. Props typées, pas de `any`. Tailwind pour le styling.

**Card.tsx** — Conteneur avec titre optionnel, padding, ombre légère. Utilisé dans le dashboard et l'analytics.

**Input.tsx** — Champ de formulaire avec label, placeholder, type, message d'erreur optionnel. Utilisé dans les pages auth.

### 2. Layouts (`src/components/layout/`)

**AuthLayout.tsx** — Fond gris clair, contenu centré verticalement/horizontalement, logo CitedBy.ai en haut. Utilise `<Outlet />` de react-router-dom.

**DashboardLayout.tsx** — Grille avec `<Sidebar />` à gauche (fixe, w-64) et zone de contenu à droite (`<Header />` + `<Outlet />`).

**Sidebar.tsx** — Logo CitedBy.ai, liens de navigation (Tableau de bord, Monitoring, Analytiques, Paramètres) avec icônes texte. Lien actif mis en surbrillance via `useLocation()`.

**Header.tsx** — Barre supérieure avec titre de la page courante et bouton déconnexion.

### 3. Pages Auth (`src/pages/auth/`)

**LoginPage.tsx** — Formulaire email + mot de passe + bouton "Se connecter" + liens vers Register et ForgotPassword. Pas de logique Supabase pour l'instant (placeholder `console.log`).

**RegisterPage.tsx** — Formulaire nom + email + mot de passe + confirmation + bouton "Créer un compte" + lien vers Login.

**ForgotPasswordPage.tsx** — Formulaire email + bouton "Réinitialiser" + lien retour Login.

### 4. Pages App (`src/pages/app/`)

**DashboardPage.tsx** — 4 cartes métriques (Citations totales, Modèles IA, Score de visibilité, Tendance) avec données statiques placeholder. Utilise `<Card />`.

**MonitoringPage.tsx** — Titre + tableau placeholder des citations récentes (Modèle IA, Requête, Cité, Date). Données statiques.

**AnalyticsPage.tsx** — Titre + graphique Recharts placeholder (LineChart simple avec données statiques montrant l'évolution des citations sur 6 mois). Utilise `<Card />`.

**SettingsPage.tsx** — Sections Profil (nom, email) et Mot de passe avec champs `<Input />` et boutons `<Button />`. Placeholder sans logique.

### 5. App.tsx — Routes refactorisées

```
/login          → AuthLayout > LoginPage
/register       → AuthLayout > RegisterPage
/forgot-password → AuthLayout > ForgotPasswordPage
/app            → DashboardLayout > DashboardPage (index)
/app/monitoring → DashboardLayout > MonitoringPage
/app/analytics  → DashboardLayout > AnalyticsPage
/app/settings   → DashboardLayout > SettingsPage
/*              → Redirect vers /login
```

Utilise les routes imbriquées de react-router-dom v7 avec `<Outlet />`.

## Contraintes techniques

- TypeScript strict : aucun `any`, tous les props typés avec `interface`
- Tailwind v4 uniquement (classes utilitaires, pas de CSS custom)
- Tout en français (labels, textes, commentaires)
- Composants fonctionnels React
- Les pages sont des placeholders visuels (pas de logique backend pour l'instant)

## Vérification

- `npm run build` doit passer sans erreur (tsc + vite build)
- Toutes les routes doivent être accessibles sans crash
