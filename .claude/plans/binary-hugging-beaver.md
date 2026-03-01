# Plan : Fusion Account + Plan & Améliorations Design

## Context
L'utilisateur veut fusionner les pages "Plan & Facturation" (`Plan.tsx`) et "Compte" (`Account.tsx`) en une seule page unifiée, ajouter un compteur de projets consommés avec offre d'upgrade, et améliorer le design pour plus de professionnalisme.

## Approche

### 1. Fusionner Account.tsx et Plan.tsx → Nouveau Account.tsx unifié

**Structure par onglets :**
- **Profil** : Infos personnelles + avatar + notifications (from Account)
- **Abonnement** : Plan actuel + compteur projets + upgrade CTA + quotas (from Plan "my-plan" + new)
- **Facturation** : Historique factures + infos paiement + PDF download (from Plan "billing")
- **API & Sécurité** : Clé API + zone danger + suppression (from Account)

**Nouveau header unifié :**
- Grand avatar utilisateur + nom + email + badge plan
- Solde GTO Gas avec barre de progression
- Bouton recharger

**Compteur de projets :**
- Barre de progression visuelle `monthlyProjectCount / maxProjects`
- Texte : "X / Y projets ce mois"
- Si proche de la limite → bandeau d'upgrade avec CTA

**Offre upgrade :**
- Section en gradient-border avec comparaison plan actuel vs plan supérieur
- CTA "Passer au Pro" ou "Passer à l'Agency"
- Features highlights du plan supérieur

### 2. Modifier App.tsx

- Supprimer la route `plan` et le composant `<Plan />` du rendu
- Supprimer `<NavItem>` "Plan & Facturation" de la sidebar
- Passer les nouvelles props à Account : `onUpgrade`, `monthlyProjectCount`, `maxProjects`, `history`
- Conserver `initialPlanTab` → convertir en `initialAccountTab` pour deep linking

### 3. Améliorer le design

- Refonte visuelle de la page Account avec le design system premium
- Utiliser les classes `section-badge`, `icon-box`, `glow-orb`, `dot-grid`
- Onglets pill-shaped avec indicator animé
- Cards avec effets de hover premium (ninja-card amélioré)
- Gradient accents sur les métriques importantes

## Fichiers à modifier

1. **`pages/Account.tsx`** — Réécriture complète (fusion Account + Plan)
2. **`App.tsx`** — Supprimer route Plan, modifier sidebar, mettre à jour props Account
3. **`pages/Plan.tsx`** — Supprimé du rendu mais fichier conservé (CLAUDE.md : NE PAS supprimer)

## Props du nouveau Account.tsx

```typescript
interface AccountProps {
  user: User | null;
  onLogout: () => void;
  onNavigate: (view: any) => void;
  onUpgrade: (plan: 'free' | 'pro' | 'enterprise') => void;
  monthlyProjectCount: number;
  maxProjects: number;
  initialTab?: 'profile' | 'subscription' | 'billing' | 'api';
}
```

## Vérification
- Build compile sans erreur (`npm run build`)
- Screenshot Puppeteer pour vérifier le rendu
- Vérifier que l'upgrade fonctionne (bouton change le plan)
- Vérifier que la navigation sidebar ne montre plus "Plan & Facturation"
- Vérifier que les deep links depuis d'autres pages fonctionnent encore
