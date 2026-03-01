# "The AI Citadel" — Animated Visualization for CitedBy.ai Landing Page

## Context

The user wants a futuristic, inventive animated visualization embedded in the CitedBy.ai landing page that visually explains how AI engines decide which brands to cite. This replaces a traditional product video with something interactive and unprecedented — a live canvas simulation of the AEO process.

## What It Does

A full-width (560px tall) HTML5 Canvas animation inserted between the hero and AEO explanation sections. It tells the AEO story visually:

1. **Left**: Search queries type themselves ("Meilleur outil CRM?") then dissolve into particles
2. **Center**: A pulsating neural constellation (25-30 nodes connected by energy lines) — particles flow into it, activating nodes and triggering energy pulses along connections
3. **Right**: Brand names materialize from the network — cited brands glow purple with halos, ghost brands appear dim and fade away
4. **Bottom overlay**: Glassmorphism bar with "Voici comment l'IA decide qui elle recommande" + CTA button
5. **Interactive**: Subtle parallax on mouse movement across the whole constellation

## Files

| File | Action |
|------|--------|
| `components/HeroAnimation.tsx` | **CREATE** (~500 lines) |
| `pages/Home.tsx` | **MODIFY** (+1 import, +1 JSX line at ~line 189) |

No new dependencies. No CSS changes. Pure Canvas 2D API + React.

## Implementation

### 1. Create `components/HeroAnimation.tsx`

**Architecture**: Single self-contained React component using:
- `useRef` for canvas + container + animation frame ID
- `useEffect` for setup/teardown (canvas sizing, DPR, event listeners, animation loop)
- `IntersectionObserver` to pause when off-screen (0 CPU)
- `prefers-reduced-motion` media query → renders static frame instead

**Animation system** (all delta-time based, capped at 50ms):

- **Network nodes** (25-30): Placed in concentric elliptical rings with noise. 3 depth layers for parallax. Each has brightness that lerps toward target then decays.
- **Particles** (max 200): Spawned when query text dissolves. Seek nearest network node. On collision, activate that node. Pool-capped.
- **Energy pulses** (max 20): Spawned when a node activates above 0.7 brightness. Travel along connections to connected nodes, activating them on arrival. Creates cascade effect.
- **Query texts** (max 2): Cycle through 8 French queries. Phases: typing → visible → dissolving into particles → done. Typing cursor blinks.
- **Brand results** (max 3): 35% cited (glow purple, halo, checkmark, drift up), 65% ghost (dim, strikethrough, drift down, fade).

**Render layers** (back to front):
1. Radial background glow (purple center)
2. Network connections (thin lines, opacity scales with node brightness)
3. Energy pulses (additive blending, bright cores with outer glow)
4. Network nodes (3-circle glow per node, additive blending)
5. Particles (small dots, additive blending, fade with life)
6. Query text (left side, purple glow shadow, blinking cursor)
7. Brand results (right side, cited = purple glow + halo + checkmark, ghost = dim + strikethrough)

**JSX structure**:
```tsx
<div className="relative w-full mb-20" style={{ height: '560px' }}>
  <canvas aria-hidden="true" />
  {/* Glassmorphism overlay bar at bottom */}
  <div style={{ background: 'rgba(0,0,0,0.6)', backdropFilter: 'blur(12px)' }}>
    <p>"Voici comment l'IA decide qui elle recommande. Et vous ?"</p>
    <button>Analysez votre site →</button>
  </div>
</div>
```

### 2. Modify `pages/Home.tsx`

- Add import: `import HeroAnimation from '../components/HeroAnimation';`
- Insert `<HeroAnimation />` between hero section (line ~189) and AEO explanation section (line ~191)

## Performance

| Metric | Budget |
|--------|--------|
| Frame time | < 8ms |
| Particles | Max 200 |
| Pulses | Max 20 |
| Off-screen | 0 CPU (IntersectionObserver) |
| Reduced motion | 1 static frame, no loop |
| Memory | ~50KB state |

## Verification

1. `npm run build` — must pass with 0 errors
2. `npm run dev` → navigate to home page → animation should appear between hero and AEO section
3. Scroll away and back — animation should pause/resume
4. Move mouse over animation — subtle parallax shift on constellation
5. Watch for 10+ seconds — queries type, dissolve, particles flow, nodes activate, brands appear/fade
6. Check browser DevTools Performance tab — should stay under 8ms per frame
