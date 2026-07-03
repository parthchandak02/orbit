# orbit — N-Body Gravity Product Simulation

## Concept
An N-body gravity simulation where product pricing data from DummyJSON creates an orbital system. Each product is a gravitational body — its price determines mass and orbit size, rating determines glow intensity, stock determines orbiting particle count. Products with higher prices dominate the gravitational field. Tap a product to focus on its orbit, showing its name, price, rating, and stock count.

**Narrative Pattern:** Drill-down (overview of orbital field → detail on tap)

**Tier 2 Patterns (pick 2):** Quiet Zone Detail Panel (A) + Hero Number (C)

## Data Source
- **API endpoint:** `https://dummyjson.com/products?limit=30`
- **Response format:**
  ```json
  {
    "products": [
      {
        "id": 1,
        "title": "iPhone 9",
        "price": 549,
        "rating": 4.69,
        "stock": 94,
        "category": "smartphones",
        "thumbnail": "https://cdn.dummyjson.com/product-images/1/thumbnail.jpg"
      }
    ]
  }
  ```
- **Fallback:** Pre-baked array of 15 realistic products with varied prices (10-999), ratings (3.0-5.0), stocks (5-100), and 5 categories. Use when fetch fails.

## Visual Style
- **Preset:** Dark — Abyss (deep ocean, smooth curves, gentle cyan glow)
- **Palette (locked JS const):**
  ```js
  const PALETTE = {
    bg: '#0a0e17',
    secondary: '#0f1b2d',
    accent1: '#00bcd4',
    accent2: '#0277bd',
    glow: '#4dd0e1',
    text: 'rgba(255,255,255,0.6)',
  };
  ```
- **Coherence tokens:**
  - strokeWeightScale: { hair: 0.5, fine: 1, medium: 2, heavy: 4 }
  - alphaHierarchy: { bg: 0.08, field: 0.25, active: 0.7, label: 0.45 }
  - shapeLanguage: geometric (circles for bodies, arcs for orbits, straight glow lines for gravity)
  - glowBudget: 1 glow color (accent1 cyan), blur <= 30px, <= 20 entities/frame
  - accentColor: ONE cyan (#00bcd4), semantic red (#ff1744) for out-of-stock, green (#00e676) for high stock
- **Motion tokens:** CSS custom properties in style block: --motion-fast: 160ms; --motion-base: 240ms; --motion-slow: 360ms; --ease-out-expo: cubic-bezier(0.22, 1, 0.36, 1)

## Encoding Contract

| Data Field | Visual Channel | Range | Inverse Function | "Bigger means..." | Legend? |
|---|---|---|---|---|---|
| product.price | gravitational mass / orbit radius | mass: 10-100; radius: 40-120px | invertScale(pixel, 40, 120, 10, 999) | more expensive product | yes (mass legend) |
| product.rating | glow intensity / body brightness | shadowBlur: 5-30; globalAlpha: 0.3-1.0 | invertScale(pixel, 5, 30, 0, 5) | higher customer rating | yes (glow legend) |
| product.stock | orbiting particle count per body | 3-15 particles | invertScale(count, 3, 15, 0, 200) | more items in stock | no (physical count) |
| product.category | orbit line color hue | 5 category hues derived from name | n/a (categorical) | different product category | yes (category color key) |

### Inverse Scale Function
```js
const scale = (val, inMin, inMax, outMin, outMax) =>
  outMin + ((val - inMin) / (inMax - inMin)) * (outMax - outMin);
const invert = (pixel, outMin, outMax, inMin, inMax) =>
  inMin + ((pixel - outMin) / (outMax - outMin)) * (inMax - inMin);
```

## Interaction
- **Tap on body:** focus that product's orbit — zoom/pan to center it, highlight its orbit path, show Quiet Zone panel with product title, price, rating, stock, category. Other bodies dim to 0.15 opacity.
- **Tap background or close button:** return to overview mode — all bodies return to full opacity, Quiet Zone panel fades out.
- **Auto-animation:** Bodies continuously orbit in elliptical paths. Slower for high-mass (expensive) products, faster for low-mass (cheap) products. Particles orbit their parent body at higher frequency.
- **Entry animation:** Bodies fade in with scale(0) to scale(1) over 800ms, staggered by product index. Orbits draw in arc-by-arc over 1s.

## Communication Requirements

### Tier 1 (mandatory)
1. **Persistent Data Stamp** — bottom-left, 10px system-ui, opacity 0.4. Shows: "DummyJSON · N products · live/fallback · Xs ago"
2. **Data Process Honesty** — if using fallback data, stamp says "fallback" not "live"
3. **One-to-One Sensory Mapping** — bigger orbit = more expensive, brighter glow = higher rating, more particles = more stock. All physical-world metaphors.

### Tier 2 (2 of 2)
4. **Quiet Zone Detail Panel (A)** — DOM overlay, positioned bottom-center, styled with same Abyss palette. Appears on tap with: product title, price, rating, stock count, category. Fades after 3s idle. Uses `pointer-events: none` for non-interactive elements.
5. **Hero Number (C)** — top-right, 48px, cyan glow. Shows total product count. Accompanying 11px label: "products orbiting"

### Coherence
6. One strokeWeightScale vocabulary (hair/fine/medium/heavy), alphaHierarchy (4 tiers), shapeLanguage (geometric), glowBudget (1 color, <=20 entities, blur <= 30px)
7. Motion tokens as CSS custom properties with prefers-reduced-motion: reduce (collapse to 0.01ms)
8. AI Slop Score < 30 — no Inter/Roboto/Open Sans, no purple gradients, layered background (navy + subtle radial glow), at least fade-in and orbit animation

## Canvas Code Requirements (MANDATORY)

1. Single HTML file `index.html`, no external dependencies, no CDN libraries, no p5.js or Three.js
2. Config object `C` at top with ALL tunable values (gravity strength, damping, min/max mass, orbit speeds, particle counts, animation durations)
3. DPR-aware canvas setup: `const dpr = Math.min(window.devicePixelRatio || 1, 2); canvas.width = W * dpr; canvas.height = H * dpr; ctx.setTransform(dpr, 0, 0, dpr, 0, 0);`
4. Canvas CSS: `position: fixed; top: 0; left: 0; width: 100%; height: 100%; display: block;`
5. Touch guard pattern: `let tapping = false;` set true on touchstart, reset on touchend, block if already tapping
6. First frame NEVER blank — pre-run 60 ticks of gravity simulation with fallback data before first render, or seed initial orbital positions
7. Realistic fallback data (15+ products embedded as fallback array)
8. Short description text (`#desc` div, bottom center below legend, 11px, opacity 0.45, `system-ui, sans-serif`): "Tap a product to see its price, rating, and stock — bigger orbits mean more expensive products"
9. Full-viewport, mobile-responsive. Test for viewports as small as 375x667
10. Works offline after first load (no CDN deps, all code inline)
11. `.nojekyll` file at project root
12. No em-dashes in ANY user-visible strings (title, desc, data stamp, hero label, quiet zone text). Use hyphens exclusively.
13. Font for all text elements: `system-ui, sans-serif`. No serif fonts, no Inter/Roboto/Open Sans.

## Legend (always visible, bottom center)
Mini key showing: mass=size -> price, glow -> rating, 3 sample category colors with labels. Each legend item is a 10px text label with a small colored dot or line.

## Chrome Positions (never overlap)
- Data stamp: `position: fixed; bottom: 20px; left: 16px; font: 10px system-ui, sans-serif; opacity: 0.4;`
- Legend: `position: fixed; bottom: 44px; left: 50%; transform: translateX(-50%); font: 10px system-ui, sans-serif; opacity: 0.35;`
- Description (desc): `position: fixed; bottom: 68px; left: 50%; transform: translateX(-50%); font: 11px system-ui, sans-serif; opacity: 0.45; max-width: 65ch; text-align: center;`
- Hero Number: `position: fixed; top: 20px; right: 20px;` with 48px font and 11px label below
- Quiet Zone panel: `position: fixed; bottom: 100px; left: 50%; transform: translateX(-50%);` pointer-events: none, fade transition

## N-Body Gravity Algorithm

```
For each body:
  For each other body:
    dx = other.x - body.x
    dy = other.y - body.y
    dist = sqrt(dx*dx + dy*dy)
    dist = max(dist, minDist) // prevent singularity
    force = G * body.mass * other.mass / (dist * dist)
    fx = force * dx / dist
    fy = force * dy / dist
    body.vx += fx / body.mass * dt
    body.vy += fy / body.mass * dt
  body.x += body.vx * dt
  body.y += body.vy * dt
  // damping
  body.vx *= damping
  body.vy *= damping
```

Parameters should be in C.gravity:
- C.G = 0.5 (gravitational constant, tunable)
- C.damping = 0.98 (velocity damping per frame)
- C.minDist = 50 (minimum distance to prevent infinite force)
- C.dt = 0.016 (time step, ~60fps)

## Verification Requirements
- Canvas non-blank at t=0 (warmup with fallback data)
- Zero JS console errors (graceful API fallback OK)
- 3-second decode test: source (DummyJSON), freshness (live/fallback + seconds), interaction (tap for details), mapping (bigger orbit = pricier product)
- Tap a body -> Quiet Zone shows correct product data
- prefers-reduced-motion: static frame, interaction still works
- Chrome positions don't overlap (stamp 20px, legend 44px, desc 68px — all distinct)
- Always-visible labels: body name + price visible on hover or as small floating label at rest
