# Plan : Refonte complète de la page Compte & Plan

## Contexte
L'utilisateur souhaite une refonte complète de la page "Compte & Plan" (Account.tsx) avec une navigation fluide et compréhensive. Actuellement, la page utilise 3 onglets horizontaux (Profil, Abonnement & Projets, Facturation) qui regroupent trop d'informations par section. L'objectif est de réorganiser le contenu en sections plus granulaires avec une navigation latérale type "Settings GitHub" pour une meilleure UX.

## Layout : Sidebar gauche + Contenu droit

### Navigation sidebar (7 sections)
```
┌──────────────────┐  ┌────────────────────────────────────────┐
│  Général         │  │                                        │
│  Abonnement      │  │   Contenu de la section active         │
│  Crédits GTO Gas │  │                                        │
│  Facturation     │  │                                        │
│  API & Intégr.   │  │                                        │
│  Notifications   │  │                                        │
│  Sécurité        │  │                                        │
│                  │  │                                        │
│  ─────────────── │  │                                        │
│  🔴 Déconnexion  │  │                                        │
└──────────────────┘  └────────────────────────────────────────┘
   ~250px                         flex-1
```

**Mobile** : la sidebar devient un scrollable horizontal de pills (comme les tabs actuels mais avec plus d'items) en haut de page.

## Fichier à modifier

### `pages/Account.tsx` (~730 → ~850 lignes)

#### 1. State : remplacer `activeTab` 3 valeurs → `activeSection` 7 valeurs

```tsx
type Section = 'general' | 'subscription' | 'credits' | 'billing' | 'api' | 'notifications' | 'security';

const [activeSection, setActiveSection] = useState<Section>(
  initialTab === 'profile' ? 'general' :
  initialTab === 'subscription' ? 'subscription' :
  initialTab === 'billing' ? 'billing' : 'general'
);
```

Conserver `initialTab` dans les props (compatibilité avec App.tsx) — mapper vers les nouvelles sections.

Ajouter à l'import Lucide : `Shield, Globe, Settings, ChevronRight` (retirer ceux devenus inutilisés).

#### 2. Sidebar Navigation Component (inline)

```tsx
const sections = [
  { key: 'general', icon: UserIcon, label: 'Général' },
  { key: 'subscription', icon: ShieldCheck, label: 'Abonnement' },
  { key: 'credits', icon: Fuel, label: 'Crédits GTO Gas' },
  { key: 'billing', icon: CreditCard, label: 'Facturation' },
  { key: 'api', icon: Key, label: 'API & Intégrations' },
  { key: 'notifications', icon: Bell, label: 'Notifications' },
  { key: 'security', icon: Shield, label: 'Sécurité' },
];
```

**Desktop** (`hidden lg:block`, `w-64 shrink-0`) :
- Chaque item : hover `bg-white/[0.04]`, active `bg-purple-500/10 text-purple-400 border-l-2 border-purple-500`
- Séparateur avant "Déconnexion" en bas
- Sticky `top-6`

**Mobile** (`lg:hidden`) :
- Barre horizontale scrollable `overflow-x-auto` avec pills
- Active = gradient purple pill (comme les tabs actuels)

#### 3. Layout principal

```tsx
<div className="flex gap-8">
  {/* Sidebar desktop */}
  <aside className="hidden lg:block w-64 shrink-0">
    <div className="sticky top-6 space-y-1">
      {/* Profile mini-card */}
      <div className="p-4 mb-4 text-center">
        <avatar + nom + email + badge plan>
      </div>
      {/* Nav items */}
      {sections.map(...)}
      {/* Separator + Déconnexion */}
    </div>
  </aside>

  {/* Mobile pills */}
  <div className="lg:hidden overflow-x-auto ...">
    {sections.map(...pills)}
  </div>

  {/* Content */}
  <main className="flex-1 min-w-0">
    {activeSection === 'general' && renderGeneral()}
    {activeSection === 'subscription' && renderSubscription()}
    {/* etc */}
  </main>
</div>
```

#### 4. Sections — Répartition du contenu existant

| Section | Contenu | Source actuelle |
|---------|---------|-----------------|
| **Général** | Infos personnelles (nom, email, workspace) + bouton Enregistrer | Tab "Profil" → carte "Informations Personnelles" |
| **Abonnement** | Plan actuel + banner upgrade + pricing cards + projet counter | Tab "Abonnement" (tout le contenu existant) |
| **Crédits GTO Gas** | Solde GTO Gas + bouton Recharger + historique d'utilisation | Tab "Profil" → carte "GTO Gas balance" (enrichi) |
| **Facturation** | Moyen de paiement + adresse facturation + historique factures | Tab "Facturation" (tout le contenu existant) |
| **API & Intégrations** | Clé API + copier + documentation link | Tab "Profil" → carte "Clés API & Intégrations" |
| **Notifications** | Toggles notification (audit terminé, alerte score, newsletters) | Tab "Profil" → carte "Préférences de notification" |
| **Sécurité** | Changer mot de passe + 2FA toggle + sessions actives + zone danger (supprimer compte) | Tab "Profil" → carte "Zone de danger" (enrichi) |

#### 5. Détail des nouvelles sections

**Section Général** — `renderGeneral()`
- Reprend exactement la carte "Informations Personnelles" existante (lignes 309-330)
- Pas de changement de contenu, juste déplacé dans sa propre section

**Section Abonnement** — `renderSubscription()`
- Reprend tout le contenu du tab "subscription" actuel (lignes 374-586) sans modification
- Project counter + GTO gas card + upgrade banner + plan actuel + pricing cards

**Section Crédits GTO Gas** — `renderCredits()`
- Reprend la carte "GTO Gas balance" existante (lignes 262-280)
- Agrandir légèrement : titre "Votre solde GTO Gas", progress bar plus visible
- Bouton "Recharger" → `onNavigate('recharge')`
- Ajouter section "Dernières consommations" (3 lignes mock : "Audit citedby.ai -10 Gas", etc.)

**Section Facturation** — `renderBilling()`
- Reprend exactement le tab "billing" actuel (lignes 591-676) sans modification

**Section API & Intégrations** — `renderApi()`
- Reprend la carte "Clés API" existante (lignes 286-306)
- Ajouter une 2ème carte "Webhooks" avec un toggle "Activer les webhooks" + champ URL endpoint (mock, non fonctionnel)

**Section Notifications** — `renderNotifications()`
- Reprend la carte "Préférences de notification" existante (lignes 333-355)
- Ajouter une catégorie "Crédits & Paiements" avec 2 toggles :
  - "Solde GTO Gas bas" (alerte si < 20 crédits)
  - "Confirmation de paiement" (reçu par email)

**Section Sécurité** — `renderSecurity()`
- Formulaire "Changer le mot de passe" (3 inputs : ancien, nouveau, confirmer) — mock, bouton "Mettre à jour"
- Toggle "Authentification à deux facteurs (2FA)" — désactivé par défaut, mock
- Carte "Sessions actives" — 2 lignes mock (navigateur actuel + mobile)
- Zone de danger existante (lignes 358-366) — déplacée ici

#### 6. Conservation du flux StripeCheckout

Le bloc StripeCheckout fullpage (lignes 172-190) et le modal succès (lignes 682-725) restent **exactement identiques** et positionnés en dehors du layout sidebar. La condition `{!(upgradeModal && paymentStep === 'payment')}` masque tout le contenu (sidebar + content) pendant le paiement, comme actuellement.

## Séquence d'implémentation

| Étape | Action |
|-------|--------|
| 1 | Modifier le state : `activeTab` → `activeSection` avec 7 valeurs, mapper `initialTab` |
| 2 | Supprimer l'ancien layout (header + tabs + 3 tab contents) |
| 3 | Créer le layout sidebar + content (desktop sidebar + mobile pills) |
| 4 | Créer `renderGeneral()` — infos personnelles (déplacées du tab Profil) |
| 5 | Créer `renderSubscription()` — tout le contenu du tab Abonnement (copié tel quel) |
| 6 | Créer `renderCredits()` — solde GTO Gas (déplacé du tab Profil) + consommations mock |
| 7 | Créer `renderBilling()` — tout le contenu du tab Facturation (copié tel quel) |
| 8 | Créer `renderApi()` — clés API (déplacées du tab Profil) + webhooks mock |
| 9 | Créer `renderNotifications()` — notifications (déplacées du tab Profil) + crédits/paiements |
| 10 | Créer `renderSecurity()` — mot de passe + 2FA + sessions + zone danger |
| 11 | Build `npm run build` — vérifier 0 erreurs |

## Vérification
1. `npm run build` — doit passer sans erreur
2. Navigation sidebar : clic sur chaque section → contenu correspondant s'affiche
3. Mobile : pills scrollables horizontalement, section active highlight purple
4. Upgrade flow : clic Upgrade → StripeCheckout fullpage → succès modal → retour section Abonnement
5. Toutes les fonctionnalités existantes préservées : copier API key, télécharger facture PDF, toggles notifications
6. Design cohérent : dark theme, `ninja-card`, accents purple/violet, `rounded-2xl`
