# orbit

Products orbit in a gravitational field driven by DummyJSON data — price determines mass and orbit size, rating determines glow intensity, stock determines orbiting particle count. Tap any product to focus on its orbit and see its details.

**Visual Style:** Dark — Abyss (deep navy, cyan-teal strokes, gentle orbital glow)

**Data Source:** DummyJSON Products API (30 products with fallback)

**How it works:**
- Price → gravitational mass and orbit radius (the more expensive, the bigger the orbit)
- Rating → glow intensity and body brightness (higher rating = brighter glow)
- Stock → orbiting particle count per product (more stock = more particles)
- Category → orbit line color hue (category-based color coding)
- N-body gravity simulation: each body exerts gravitational pull on every other body
- Tap to focus a product: camera zooms in, quiet zone panel shows price, rating, stock, category
- Hero number in top-right shows total product count
- Data stamp shows source, freshness, and live/fallback status

**Live:** https://parthchandak02.github.io/orbit/

**Tech:** Native Canvas 2D, single HTML file, zero dependencies.
