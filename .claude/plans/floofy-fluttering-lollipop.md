# Plan de reconstruction de CitedBy.ai sur Google AI Studio

## Contexte

CitedBy.ai est un SaaS d'Answer Engine Optimization (AEO) & GEO construit avec Next.js 15.1, React 19, TypeScript, Tailwind CSS, Supabase et Google Gemini. L'objectif est de fournir **un prompt complet et détaillé par section**, dans l'ordre logique de dépendance, pour reconstruire l'intégralité du SaaS via Google AI Studio (Gemini).

Le SaaS est entièrement en français et comprend ~20 sections couvrant : landing page, authentification, dashboard, analyse AEO 50+ critères, 9 outils génératifs (AEO Studio), gestion de projets, facturation/crédits, et paramètres.

---

## Ordre de reconstruction (20 sections)

| # | Section | Dépend de |
|---|---------|-----------|
| 1 | **Landing Page** (+ tailwind.config, globals.css, root layout) | — |
| 2 | **Types TypeScript** (database, analysis, billing, index) | — |
| 3 | **Config & Utilitaires** (constants, navigation, criteria, cn, utils) | Section 2 |
| 4 | **Système Auth Supabase** (client, server, middleware, context, hook) | Section 2 |
| 5 | **Pages Auth** (connexion, inscription, callback, auth layout) | Section 4 |
| 6 | **Shell Applicatif** (AppShell, Sidebar, TopBar, MobileNav, app layout) | Sections 3-4 |
| 7 | **Dashboard** (tableau-de-bord, dashboard-client, ScoreRing) | Section 6 |
| 8 | **Nouvelle Analyse** (formulaire URL, validation, soumission) | Section 6 |
| 9 | **Moteur d'Analyse** (analyzer.ts, 50 critères, scoring) | Sections 2-3 |
| 10 | **Résultats d'Analyse** (resultats/[id], results-client, composants) | Sections 7-9 |
| 11 | **Hub AEO Studio** (page hub, actions.ts Gemini, tool-wrapper) | Section 6 |
| 12 | **Outils Studio** (8 outils : Source of Truth → Benchmark) | Section 11 |
| 13 | **Projets** (CRUD projets, liste, cards) | Section 6 |
| 14 | **Facturation & Crédits** (pricing, packs crédits) | Section 6 |
| 15 | **Paramètres** (profil utilisateur) | Section 6 |
| 16 | **Prompts IA** (expert, blog, pdf, llm-templates) | — |
| 17 | **Services** (schema-service, robots-service) | Section 2 |
| 18 | **Pages d'erreur & loading** (error, not-found, loading, spinner) | Section 1 |
| 19 | **Config projet** (package.json, tsconfig, next.config, .env.example) | — |
| 20 | **API Routes** (Stripe webhooks) | Sections 2-4 |

---

## SECTION 1 — Prompt complet : Landing Page

Copier-coller ce prompt tel quel dans Google AI Studio :

---

```
Tu es un développeur senior Next.js / React / TypeScript / Tailwind CSS. Je reconstruis le SaaS "CitedBy.ai" étape par étape. Voici la Section 1 : la Landing Page (page d'accueil publique) avec sa fondation technique (Tailwind config, styles globaux, root layout).

=== CONTEXTE TECHNIQUE ===

Stack technique :
- Next.js 15.1 (App Router, dossier src/app/)
- React 19
- TypeScript strict mode
- Tailwind CSS 3.4 avec mode dark activé
- Font : Inter (Google Fonts, variable CSS --font-inter)
- Icônes : lucide-react uniquement
- Pas de Framer Motion sur cette page (page statique)
- Pas de Supabase sur cette page (page publique, pas d'auth)

=== FICHIERS À GÉNÉRER (4 fichiers complets) ===

---

### FICHIER 1 : tailwind.config.ts

Configuration Tailwind personnalisée pour un thème dark premium.

Structure :
- import type { Config } from 'tailwindcss'
- content: ['./src/**/*.{ts,tsx}']
- darkMode: 'class'

theme.extend :

colors :
  background: { DEFAULT: '#0F172A', secondary: '#1E293B', tertiary: '#334155' }
  surface: { DEFAULT: '#1E293B', hover: '#334155', border: 'rgba(51, 65, 85, 0.5)' }
  accent: { DEFAULT: '#3B82F6', hover: '#2563EB', light: '#60A5FA', dark: '#1D4ED8' }
  score: { excellent: '#10B981', good: '#3B82F6', average: '#F59E0B', poor: '#EF4444' }

fontFamily :
  sans: ['var(--font-inter)', 'Inter', 'system-ui', 'sans-serif']

backgroundImage :
  'gradient-hero': 'linear-gradient(135deg, #1E40AF, #7C3AED)'
  'gradient-card': 'linear-gradient(135deg, rgba(59, 130, 246, 0.1), rgba(124, 58, 237, 0.1))'

borderRadius :
  xl: '12px'
  '2xl': '16px'

boxShadow :
  glass: '0 4px 30px rgba(0, 0, 0, 0.3)'
  card: '0 10px 40px rgba(0, 0, 0, 0.2)'

animation :
  'score-fill': 'scoreFill 1.5s ease-out forwards'
  'fade-in': 'fadeIn 0.3s ease-out forwards'
  'slide-up': 'slideUp 0.4s ease-out forwards'

keyframes :
  scoreFill: { '0%': { strokeDashoffset: '283' }, '100%': { strokeDashoffset: '0' } }
  fadeIn: { '0%': { opacity: '0' }, '100%': { opacity: '1' } }
  slideUp: { '0%': { opacity: '0', transform: 'translateY(10px)' }, '100%': { opacity: '1', transform: 'translateY(0)' } }

plugins: []

---

### FICHIER 2 : src/app/globals.css

@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base :
- html : color-scheme: dark; -webkit-font-smoothing: antialiased; -moz-osx-font-smoothing: grayscale
- body : @apply bg-background text-slate-50;
  background-image:
    radial-gradient(ellipse at 20% 50%, rgba(59, 130, 246, 0.08) 0%, transparent 50%),
    radial-gradient(ellipse at 80% 50%, rgba(124, 58, 237, 0.06) 0%, transparent 50%);
  background-attachment: fixed;

@layer components :
- .glass-panel : background: rgba(30, 41, 59, 0.5); backdrop-filter: blur(12px); -webkit-backdrop-filter: blur(12px); border: 1px solid rgba(51, 65, 85, 0.5); border-radius: 12px;
- .card-hover : @apply glass-panel transition-all duration-200; &:hover { background: rgba(30, 41, 59, 0.7); border-color: rgba(59, 130, 246, 0.3); box-shadow: 0 10px 40px rgba(0, 0, 0, 0.2); }
- .grade-badge : @apply inline-flex items-center justify-center font-bold rounded-lg px-3 py-1 text-sm;
- .btn-primary : @apply bg-accent hover:bg-accent-hover text-white font-medium px-6 py-2.5 rounded-xl transition-all duration-200 shadow-lg shadow-accent/20 hover:shadow-xl hover:shadow-accent/30;
- .btn-secondary : @apply bg-slate-800 hover:bg-slate-700 text-slate-200 font-medium px-6 py-2.5 rounded-xl border border-slate-700/50 transition-all duration-200;
- .input-field : @apply w-full bg-slate-800/50 border border-slate-700/50 rounded-xl px-4 py-3 text-slate-50 placeholder:text-slate-500 focus:outline-none focus:border-accent/50 focus:ring-2 focus:ring-accent/20 transition-all duration-200;

@layer utilities :
- .custom-scrollbar : &::-webkit-scrollbar { width: 6px; } &::-webkit-scrollbar-track { background: transparent; } &::-webkit-scrollbar-thumb { background: rgba(148, 163, 184, 0.2); border-radius: 3px; } &::-webkit-scrollbar-thumb:hover { background: rgba(148, 163, 184, 0.4); }
- .no-scrollbar : &::-webkit-scrollbar { display: none; } -ms-overflow-style: none; scrollbar-width: none;

---

### FICHIER 3 : src/app/layout.tsx

Root layout Next.js :
- import type { Metadata } from 'next'
- import { Inter } from 'next/font/google'
- import './globals.css'

const inter = Inter({ subsets: ['latin'], variable: '--font-inter', display: 'swap' })

export const metadata: Metadata = {
  title: { default: 'CitedBy.ai — Optimisation AEO & GEO', template: '%s | CitedBy.ai' },
  description: "La première plateforme tout-en-un d'Answer Engine Optimization. Soyez cité par ChatGPT, Claude, Perplexity et Gemini.",
  keywords: ['AEO', 'GEO', 'Answer Engine Optimization', 'IA', 'citations', 'SEO'],
}

Retourne :
<html lang="fr" className={`dark ${inter.variable}`}>
  <body className="font-sans antialiased custom-scrollbar">
    {children}
  </body>
</html>

---

### FICHIER 4 : src/app/page.tsx — LA LANDING PAGE COMPLÈTE

C'est un Server Component (PAS de 'use client'). Pas de useState, useEffect.

IMPORTS :
import Link from 'next/link'
import { Zap, Search, BarChart3, Shield, Brain, Globe, Sparkles } from 'lucide-react'

DONNÉES — Tableau `features` avec exactement 6 objets :

1. { icon: Search, title: "Audit AEO 50+ critères", description: "Analysez n'importe quelle URL sur 50+ critères d'optimisation pour les moteurs de réponse IA." }
2. { icon: Brain, title: "Solutions IA personnalisées", description: "Générez du code et du contenu prêt à copier-coller, adapté à votre site." }
3. { icon: BarChart3, title: "Citation Tracker", description: "Suivez en temps réel les mentions de votre marque par ChatGPT, Claude, Perplexity et Gemini." }
4. { icon: Shield, title: "Indexation IA", description: "Configurez robots.txt, llms.txt et Schema.org pour maximiser votre visibilité IA." }
5. { icon: Globe, title: "Marketplace Publishers", description: "Accédez à un réseau de publishers pour obtenir des citations sur des sites à forte autorité." }
6. { icon: Sparkles, title: "Studio AEO All-in-One", description: "9 outils génératifs : articles, FAQ, fiches produits, Schema.org, et plus encore." }

STRUCTURE JSX EXACTE :

<div className="min-h-screen">

  {/* === NAVBAR === */}
  <header className="fixed top-0 w-full z-50 bg-background/80 backdrop-blur-md border-b border-slate-700/50">
    <div className="max-w-6xl mx-auto flex items-center justify-between px-6 py-4">
      {/* Logo */}
      <Link href="/" className="flex items-center gap-3">
        <div className="w-8 h-8 rounded-lg bg-gradient-hero flex items-center justify-center">
          <Zap className="w-4 h-4 text-white" />
        </div>
        <span className="text-lg font-bold text-slate-50">
          CitedBy<span className="text-accent">.ai</span>
        </span>
      </Link>
      {/* Boutons */}
      <div className="flex items-center gap-3">
        <Link href="/connexion" className="btn-secondary text-sm">Connexion</Link>
        <Link href="/inscription" className="btn-primary text-sm">Essai gratuit</Link>
      </div>
    </div>
  </header>

  {/* === HERO === */}
  <section className="pt-32 pb-20 px-6">
    <div className="max-w-4xl mx-auto text-center">
      {/* Badge */}
      <div className="inline-flex items-center gap-2 px-4 py-1.5 rounded-full bg-accent/10 border border-accent/20 text-accent text-sm font-medium mb-6">
        <Sparkles className="w-4 h-4" />
        Le Semrush de l&apos;ère IA
      </div>
      {/* Titre H1 */}
      <h1 className="text-4xl md:text-6xl font-bold text-slate-50 mb-6 leading-tight">
        Soyez cité par les IA.{' '}
        <span className="bg-gradient-to-r from-accent to-purple-400 bg-clip-text text-transparent">
          Pas ignoré.
        </span>
      </h1>
      {/* Sous-titre */}
      <p className="text-lg text-slate-400 mb-10 max-w-2xl mx-auto">
        CitedBy.ai analyse, optimise et monitore votre présence dans les réponses
        de ChatGPT, Claude, Perplexity et Gemini. La première plateforme AEO &amp; GEO
        tout-en-un.
      </p>
      {/* CTA */}
      <div className="flex flex-col sm:flex-row gap-4 justify-center">
        <Link href="/inscription" className="btn-primary text-base px-8 py-3">
          Commencer gratuitement
        </Link>
        <Link href="/analyse/nouvelle" className="btn-secondary text-base px-8 py-3">
          Scanner une URL gratuit
        </Link>
      </div>
    </div>
  </section>

  {/* === FEATURES === */}
  <section className="py-20 px-6">
    <div className="max-w-6xl mx-auto">
      <h2 className="text-3xl font-bold text-slate-50 text-center mb-4">
        Tout ce dont vous avez besoin
      </h2>
      <p className="text-slate-400 text-center mb-12 max-w-xl mx-auto">
        Une plateforme complète pour dominer les moteurs de réponse IA.
      </p>
      <div className="grid md:grid-cols-2 lg:grid-cols-3 gap-6">
        {features.map((feature) => {
          const Icon = feature.icon
          return (
            <div key={feature.title} className="card-hover p-6">
              <div className="w-10 h-10 rounded-xl bg-accent/10 flex items-center justify-center mb-4">
                <Icon className="w-5 h-5 text-accent" />
              </div>
              <h3 className="text-lg font-bold text-slate-50 mb-2">{feature.title}</h3>
              <p className="text-sm text-slate-400 leading-relaxed">{feature.description}</p>
            </div>
          )
        })}
      </div>
    </div>
  </section>

  {/* === CTA FINAL === */}
  <section className="py-20 px-6">
    <div className="max-w-3xl mx-auto text-center glass-panel p-12">
      <h2 className="text-3xl font-bold text-slate-50 mb-4">
        Prêt à être cité par les IA ?
      </h2>
      <p className="text-slate-400 mb-8">
        50 crédits offerts. Aucune carte bancaire requise.
      </p>
      <Link href="/inscription" className="btn-primary text-base px-8 py-3">
        Créer mon compte gratuit
      </Link>
    </div>
  </section>

  {/* === FOOTER === */}
  <footer className="border-t border-slate-700/50 py-8 px-6">
    <div className="max-w-6xl mx-auto flex items-center justify-between text-sm text-slate-500">
      <span>CitedBy.ai — Plateforme AEO &amp; GEO</span>
      <span>© {new Date().getFullYear()}</span>
    </div>
  </footer>

</div>

=== RÈGLES IMPÉRATIVES ===

1. Génère UNIQUEMENT du TypeScript strict. Aucun "any".
2. Utilise EXCLUSIVEMENT les classes CSS custom définies dans globals.css (btn-primary, btn-secondary, glass-panel, card-hover, input-field) et les classes Tailwind.
3. Les couleurs custom (background, accent, score) viennent de tailwind.config.ts. Ne mets PAS de couleurs hexadécimales en dur dans le JSX.
4. Le composant page.tsx est un Server Component (PAS de 'use client'). Pas de hooks React.
5. Pas d'images, pas de SVG custom. Uniquement des icônes lucide-react.
6. Le HTML lang est "fr".
7. Tous les textes visibles sont en français.
8. Utilise &apos; pour les apostrophes et &amp; pour les esperluettes dans le JSX.
9. Utilise le composant Link de Next.js (import Link from 'next/link') pour tous les liens internes.
10. Génère chaque fichier EN ENTIER, complet, prêt à copier-coller sans aucune modification.
11. Ajoute des commentaires en français dans le code pour expliquer chaque section majeure.
12. N'ajoute RIEN qui n'est pas décrit ci-dessus. Pas de sections supplémentaires, pas de composants extra.
```

---

## Sections suivantes (résumé)

Après validation de la Section 1, je fournirai les prompts détaillés suivants dans cet ordre :

| Section | Contenu | Fichiers clés |
|---------|---------|---------------|
| **2** | Types TypeScript | `src/types/database.ts`, `analysis.ts`, `billing.ts`, `marketplace.ts`, `index.ts` |
| **3** | Config & Utilitaires | `src/config/constants.ts`, `navigation.ts`, `criteria.ts`, `src/lib/cn.ts`, `utils.ts` |
| **4** | Auth Supabase | `src/lib/supabase/client.ts`, `server.ts`, `middleware.ts`, `src/contexts/auth-context.tsx`, `src/hooks/use-user.ts`, `src/middleware.ts` |
| **5** | Pages Auth | `src/app/(auth)/layout.tsx`, `connexion/page.tsx`, `inscription/page.tsx`, `auth/callback/route.ts` |
| **6** | Shell Applicatif | `src/app/(app)/layout.tsx`, `src/components/layout/app-shell.tsx`, `sidebar.tsx`, `top-bar.tsx`, `mobile-nav.tsx` |
| **7** | Dashboard | `src/app/(app)/tableau-de-bord/page.tsx`, `src/components/dashboard/dashboard-client.tsx` |
| **8** | Nouvelle Analyse | `src/app/(app)/analyse/nouvelle/page.tsx` |
| **9** | Moteur d'Analyse | `src/services/analyzer.ts`, `src/config/criteria.ts` |
| **10** | Résultats d'Analyse | `src/app/(app)/analyse/resultats/[id]/page.tsx`, composants analysis |
| **11** | Hub AEO Studio | `src/app/(app)/aeo-studio/page.tsx`, `actions.ts`, `tool-wrapper.tsx` |
| **12** | 8 Outils Studio | Source of Truth, Schema.org, Robots.txt, Article, FAQ, Fiche, Knowledge Graph, Benchmark |
| **13** | Projets | `src/app/(app)/projets/page.tsx` |
| **14** | Facturation & Crédits | `src/app/(app)/facturation/page.tsx`, `recharge/page.tsx` |
| **15** | Paramètres | `src/app/(app)/parametres/page.tsx` |
| **16** | Prompts IA | `src/config/prompts/expert-prompts.ts`, `blog-prompts.ts`, `pdf-prompts.ts`, `llm-templates.ts` |
| **17** | Services | `src/services/schema-service.ts`, `robots-service.ts` |
| **18** | Pages d'erreur & loading | `error.tsx`, `not-found.tsx`, `loading.tsx`, `loading-spinner.tsx` |
| **19** | Config projet | `package.json`, `tsconfig.json`, `next.config.ts`, `.env.example` |
| **20** | API Routes | `src/app/api/webhooks/stripe/route.ts` |

---

## Vérification

Après chaque section :
1. Coller le code généré dans les fichiers correspondants du projet Next.js
2. Lancer `npm run dev` pour vérifier qu'il compile sans erreur
3. Vérifier visuellement dans le navigateur que le rendu correspond au design dark premium
4. S'assurer que les liens de navigation fonctionnent
5. Après la Section 1 : la landing page doit s'afficher avec navbar, hero, 6 features, CTA et footer
