# Plan : personal-site — Portfolio Brutalist + Bold avec Next.js

## Context
Créer un site portfolio/landing page personnel avec Next.js, en style **Brutalist + Bold** (typographie massive, couleurs brutes, design anticonformiste). Le projet utilisera les skills installées (Superpowers, GSD, design, frontend) pour un résultat professionnel.

## Stack technique
- **Next.js 15** (App Router)
- **TypeScript**
- **Tailwind CSS 4**
- **Framer Motion** (animations)
- **Geist / Space Grotesk** (typographies bold)

## Structure du projet

```
C:\Users\User\personal-site\
├── src/
│   ├── app/
│   │   ├── layout.tsx          # Layout racine, fonts, metadata
│   │   ├── page.tsx            # Page d'accueil
│   │   └── globals.css         # Styles globaux + Tailwind
│   ├── components/
│   │   ├── hero.tsx            # Section hero brutalist (titre massif)
│   │   ├── about.tsx           # Section à propos
│   │   ├── projects.tsx        # Grille de projets
│   │   ├── skills.tsx          # Compétences en style brut
│   │   ├── contact.tsx         # Formulaire/liens de contact
│   │   └── navbar.tsx          # Navigation minimaliste
│   └── lib/
│       └── constants.ts        # Données du portfolio (projets, skills, liens)
├── public/
│   └── favicon.ico
├── package.json
├── tsconfig.json
├── tailwind.config.ts
├── next.config.ts
└── postcss.config.mjs
```

## Design Brutalist + Bold

### Principes
- **Typographie massive** : titres en 6xl-9xl, font-weight 900
- **Couleurs brutes** : noir #000, blanc #fff, accent jaune vif #FFE500 ou rouge #FF3333
- **Pas de border-radius** : coins carrés partout
- **Bordures épaisses** : border-4 ou border-8 noir
- **Hover agressifs** : inversions de couleurs, scale, rotations légères
- **Texte uppercase** : titres en majuscules
- **Layout asymétrique** : grilles décalées, overlaps

### Palette
- Background : `#000000` (noir)
- Text : `#FFFFFF` (blanc)
- Accent 1 : `#FFE500` (jaune brut)
- Accent 2 : `#FF3333` (rouge vif)
- Borders : `#FFFFFF` 4px solid

### Sections de la page
1. **Hero** — Titre géant "PORTFOLIO" ou nom, animation de texte, phrase d'accroche
2. **About** — Bloc de texte brut avec bordure épaisse
3. **Projects** — Grille 2-3 colonnes, cartes avec hover inversé
4. **Skills** — Tags/badges en style brutaliste
5. **Contact** — Liens sociaux + email, style terminal/brut

## Étapes d'implémentation

### 1. Scaffolding Next.js
```bash
npx create-next-app@latest personal-site --typescript --tailwind --eslint --app --src-dir --no-import-alias
```

### 2. Installer dépendances
```bash
npm install framer-motion
```

### 3. Configurer les fonts et globals.css
- Importer Space Grotesk depuis Google Fonts via `next/font`
- Configurer les styles brutalist globaux

### 4. Créer les composants
- `navbar.tsx` — Navigation fixe, liens en uppercase
- `hero.tsx` — Titre 9xl animé, accent jaune
- `about.tsx` — Bloc texte avec bordure 4px
- `projects.tsx` — Grille de cartes avec hover inversé noir/jaune
- `skills.tsx` — Tags brutalist
- `contact.tsx` — Liens + style terminal

### 5. Assembler page.tsx
- Importer et empiler tous les composants

### 6. Données dans constants.ts
- Liste de projets (titre, description, lien, tags)
- Liste de compétences
- Liens sociaux

## Vérification
1. `npm run dev` — vérifier le rendu sur http://localhost:3000
2. `npm run build` — vérifier que le build passe sans erreur
3. `npm run lint` — vérifier le linting
4. Tester le responsive (mobile/tablette/desktop)
