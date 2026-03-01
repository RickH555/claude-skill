# Landing Page Improvement Plan — CitedBy.ai

## Context

The user requested "Build a landing page for a SaaS product." CitedBy.ai already has a landing page (`pages/Home.tsx`, 756 lines) with hero, free audit, stats, features, SEO vs AEO comparison, 3 pillars, process timeline, testimonials, and pricing. The UI/UX Pro Max skill analysis identified several improvements to bring it in line with professional SaaS landing page standards.

**Design system generated:** Dark Mode OLED style, Hero + Testimonials + CTA pattern, Plus Jakarta Sans / Inter fonts, purple-violet palette with amber accents.

---

## Changes

### 1. Replace emoji trust badges with Lucide SVG icons
**File:** `pages/Home.tsx` lines 656-670

The trust badges section uses emojis (`🛡️🔒⚡💬🇫🇷`) which violates the no-emoji-icons rule. Replace with Lucide icons.

| Current emoji | Replacement Lucide icon |
|---------------|------------------------|
| 🛡️ RGPD | `<ShieldCheck />` |
| 🔒 SSL | `<Lock />` |
| ⚡ Uptime | `<Zap />` (already imported) |
| 💬 Support | `<Headphones />` |
| 🇫🇷 France | `<Flag />` |

### 2. Add `cursor-pointer` to all interactive cards
**File:** `pages/Home.tsx`

Several cards use `cursor-default` or have no cursor class despite being visually interactive (hover effects). Fix:
- Feature cards (line 429): `cursor-default` → `cursor-pointer`
- Stat cards (line 392): add `cursor-pointer`
- Pillar cards (line 533): add `cursor-pointer`
- Pricing cards (line 695): add `cursor-pointer`
- Testimonial cards (line 636): add `cursor-pointer`

### 3. Add FAQ section between testimonials and pricing
**File:** `pages/Home.tsx` — Insert new section after line 671 (end of testimonials)

Accordion-style FAQ with 5 questions:
1. "Qu'est-ce que l'AEO ?" — Explains Answer Engine Optimization
2. "Comment les IA choisissent-elles les sources ?" — Entity authority, structured data
3. "Combien de temps pour voir des résultats ?" — 2-4 weeks typical
4. "Qu'est-ce que le GTO Gas ?" — Virtual credits system
5. "Est-ce compatible avec le SEO classique ?" — AEO complements SEO

Implementation: Simple `useState<number | null>` toggle for open/closed accordion items. Use `ChevronDown` icon with rotation animation.

### 4. Add final CTA section before page end
**File:** `pages/Home.tsx` — Insert after pricing section (line 749)

Full-width gradient-bordered card with:
- Headline: "Prêt à dominer les réponses IA ?"
- Subtext: "Rejoignez les marques qui se font citer par ChatGPT, Claude et Gemini."
- Primary CTA button: "Commencer gratuitement" → `handleStartAnalysis`
- Secondary link: "Explorer l'annuaire" → `onNavigate('directory')`

### 5. Improve hover transitions consistency
**File:** `pages/Home.tsx`

Ensure all hover effects use `transition-all duration-200` or `transition-colors duration-200` (150-300ms range per UX guidelines). Currently some cards use `duration-300` and `duration-500` — standardize to `duration-300` max for micro-interactions.

### 6. Add new Lucide imports
**File:** `pages/Home.tsx` line 2

Add: `ShieldCheck`, `Lock`, `Headphones`, `Flag`, `ChevronDown` to the Lucide import.

---

## Files Modified

| File | Change |
|------|--------|
| `pages/Home.tsx` | All changes (single file modification) |

## Verification

1. `npm run dev` — Start Vite dev server
2. Open landing page in browser at localhost
3. Verify: No emojis visible in trust badges section
4. Verify: FAQ accordion opens/closes smoothly
5. Verify: Final CTA section visible at bottom
6. Verify: All cards show `cursor: pointer` on hover
7. Verify: No console errors
8. `npm run build` — Ensure production build passes
