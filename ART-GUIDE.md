# Art Asset Guide: Creating & Using Isometric Room Art

> **Related issues:** #6-12 (design), #13 (layered asset system)

---

## 1. How the art system works

Each scene is built from **layered images stacked on top of each other**, not a single flat picture. This lets us make individual objects interactive (hoverable, clickable) while keeping the room as a cohesive illustration.

```
┌─────────────────────────────┐
│  Interactive object layers  │  ← Separate PNGs, position: absolute
│  (books, pins, frames...)   │     Click/hover targets
├─────────────────────────────┤
│  Decorative object layers   │  ← Separate PNGs, no interaction
│  (lamp, rug, furniture)     │     Visual depth only
├─────────────────────────────┤
│  Room background            │  ← Single PNG/WebP
│  (walls, floor, fixed       │     Base layer, position: relative
│   furniture)                │
└─────────────────────────────┘
```

---

## 2. What to create as PNG vs SVG

### Use PNG (or WebP) for:
- **Room backgrounds** — complex illustrated scenes with textures, gradients, pixel art detail
- **Interactive objects** — books, record player, photo frames, globe — anything with rich visual detail
- **Decorative props** — lamps, rugs, furniture, plants
- **Character sprites** — cat, family figures

### Use SVG for:
- **UI elements** — back button, navigation arrows, close icons
- **Map pins** — simple colored dots/markers that need to scale cleanly
- **Hover overlays** — glow effects, highlight borders
- **Legend icons** — small colored dots in the map legend
- **Text labels** — room names, tooltips
- **Simple geometric shapes** — borders, dividers, decorative corners

### Rule of thumb
> If it has **texture, shading, or pixel-art detail** → PNG
> If it's **geometric, needs to scale, or is UI chrome** → SVG

---

## 3. PNG asset specifications

### File format
- **PNG-24 with transparency** for all layered objects
- **WebP** acceptable for backgrounds (smaller file size, same quality)
- **No JPEG** — we need transparency for layering

### Canvas sizes

| Asset type | Target size | Notes |
|-----------|-------------|-------|
| Room background | 800×600px | Base canvas for all rooms |
| House exterior | 800×600px | Door screen scene |
| House overview (hub) | 900×600px | Slightly wider for 4-room layout |
| Interactive objects | Varies | Export at actual placement size |
| Books (on shelf) | ~60×120px | Vertical spine visible |
| Map pins | ~20×30px | Small, need to be crisp |
| Photo frames | ~100×120px | With frame border included |
| Record player | ~200×200px | Central music room piece |
| Globe | ~80×80px | Map room decorative |
| Shared sprites | Varies | Keep under 50KB each |

### Resolution
- Design at **2x** (1600×1200 for rooms) for retina displays
- Export at both **1x** and **2x**, or just 2x and let CSS scale down
- If pixel art style: design at **1x** and scale up with `image-rendering: pixelated`

### Color palette
Stay consistent with the app's existing theme:

| Purpose | Color | Hex |
|---------|-------|-----|
| Background | Dark navy | `#1a1a2e` |
| Card background | Deep blue | `#16213e` |
| Primary gold | Light gold | `#ffeaa7` |
| Secondary gold | Medium gold | `#d4a843` |
| Warm accent | Tan | `#b8956a` |
| Muted text | Blue-gray | `#7f8fa6` |
| Pin — lived | Coral red | `#e17055` |
| Pin — visited | Green | `#00b894` |
| Pin — dream | Purple | `#a29bfe` |
| Book spine 1 | Coral | `#e17055` |
| Book spine 2 | Green | `#00b894` |
| Book spine 3 | Blue | `#0984e3` |
| Book spine 4 | Yellow | `#fdcb6e` |

---

## 4. How to create the assets

### Option A: Pixel art (matches reference image)

**Tools:** Aseprite ($20, best for pixel art), Piskel (free, web-based), LibreSprite (free)

**Process:**
1. Set canvas to target size (e.g., 800×600 for a room)
2. Draw the room background on one layer
3. Draw each interactive object on its **own layer**
4. Export background as single PNG
5. Export each object layer as separate PNG (File → Export → By Layer)
6. Keep the `.aseprite` / `.piskel` source file in `images/source/` for future edits

**Pixel art scaling:**
```css
.room-scene img {
  image-rendering: pixelated;  /* crisp pixel edges when scaled up */
  image-rendering: crisp-edges; /* Firefox */
}
```

### Option B: Illustrated / painted style

**Tools:** Procreate (iPad), Photoshop, Clip Studio Paint, Krita (free)

**Process:**
1. Draw full scene in one document with organized layers
2. Group layers: `background`, `furniture`, `interactive-objects`, `decorative`
3. Export each interactive object by hiding other layers and cropping to bounds
4. Export background with all non-interactive elements merged
5. Save source PSD/Procreate file in `images/source/`

### Option C: AI-generated + manual cleanup

**Tools:** Midjourney, DALL-E, Stable Diffusion → then Photoshop/GIMP for separation

**Process:**
1. Generate full room scene with AI
2. Manually cut out interactive objects into separate layers
3. Clean up edges, ensure transparent backgrounds
4. Touch up colors to match palette
5. This works for backgrounds but interactive objects may need manual redrawing for clean edges

---

## 5. SVG specifications

### Creating SVGs

**Tools:** Figma (free), Illustrator, Inkscape (free)

**Inline SVG for UI elements:**
```html
<!-- Back button arrow -->
<svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="#b8956a" stroke-width="2">
  <path d="M19 12H5M12 19l-7-7 7-7"/>
</svg>
```

**Map pin SVG example:**
```html
<svg width="20" height="30" viewBox="0 0 20 30">
  <circle cx="10" cy="10" r="8" fill="#e17055" opacity="0.9"/>
  <circle cx="10" cy="10" r="4" fill="#fff" opacity="0.3"/>
</svg>
```

### SVG best practices
- Use `viewBox` so they scale without fixed dimensions
- Keep fills as `currentColor` where possible (lets CSS control color)
- Minimize path complexity — run through [SVGO](https://jakearchibald.github.io/svgomg/) to optimize
- For hover glows, use SVG `<filter>` with `feGaussianBlur`:

```html
<svg>
  <defs>
    <filter id="glow">
      <feGaussianBlur stdDeviation="3" result="blur"/>
      <feMerge>
        <feMergeNode in="blur"/>
        <feMergeNode in="SourceGraphic"/>
      </feMerge>
    </filter>
  </defs>
  <circle cx="10" cy="10" r="8" fill="#ffeaa7" filter="url(#glow)"/>
</svg>
```

---

## 6. File organization

```
images/
├── source/                    # Editable source files (PSD, Aseprite, Figma links)
│   └── README.md              # Links to Figma/source files, layer naming conventions
│
├── exterior/                  # Door screen (#6)
│   ├── bg-house-exterior.webp
│   ├── door.png
│   ├── door-handle.png
│   └── porch-details.png
│
├── hub/                       # House overview (#7)
│   ├── bg-house-overview.webp
│   ├── room-highlight-library.png
│   ├── room-highlight-music.png
│   ├── room-highlight-map.png
│   └── room-highlight-family.png
│
├── library/                   # Library room (#8)
│   ├── bg-library.webp
│   ├── book-son.png
│   ├── book-daughter.png
│   ├── book-grandchild.png
│   ├── book-family.png
│   ├── lamp.png
│   └── armchair.png
│
├── music/                     # Music room (#9)
│   ├── bg-music.webp
│   ├── record-player.png
│   ├── vinyl-record.png
│   ├── song-1.png
│   ├── song-2.png
│   ├── song-3.png
│   └── speakers.png
│
├── map/                       # Map room (#10)
│   ├── bg-map.webp
│   ├── globe.png
│   ├── wall-map.png
│   └── desk-props.png
│   # (pins are SVG, not PNG)
│
├── family/                    # Family room (#11)
│   ├── bg-family.webp
│   ├── frame-1.png
│   ├── frame-2.png
│   ├── frame-3.png
│   ├── frame-4.png
│   ├── frame-5.png
│   ├── frame-6.png
│   └── couch.png
│
├── ui/                        # Shared elements (#12)
│   ├── btn-back.svg
│   ├── glow-overlay.png
│   ├── sparkle-particle.png
│   └── header-banner.png
│
└── icons/                     # SVG icons
    ├── pin-lived.svg
    ├── pin-visited.svg
    ├── pin-dream.svg
    ├── arrow-back.svg
    └── heart.svg
```

---

## 7. Implementing layers in code

### HTML structure for a room scene

```html
<div class="room-scene" style="position: relative; width: 800px; max-width: 100%;">
  <!-- Base layer -->
  <img src="/images/library/bg-library.webp" alt="" class="room-bg" />

  <!-- Decorative layers (no interaction) -->
  <img src="/images/library/lamp.png" alt="" class="layer" style="top: 15%; left: 70%;" />
  <img src="/images/library/armchair.png" alt="" class="layer" style="top: 60%; left: 65%;" />

  <!-- Interactive layers -->
  <img src="/images/library/book-son.png" alt="Cuốn sách của Con Trai"
       class="layer interactive" style="top: 30%; left: 20%;"
       onclick="openBook('son')" />
  <img src="/images/library/book-daughter.png" alt="Cuốn sách của Con Gái"
       class="layer interactive" style="top: 30%; left: 32%;"
       onclick="openBook('daughter')" />
</div>
```

### CSS for layering

```css
.room-scene {
  position: relative;
  aspect-ratio: 4 / 3;       /* 800×600 ratio */
  max-width: 800px;
  margin: 0 auto;
  overflow: hidden;
}

.room-bg {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.layer {
  position: absolute;
  pointer-events: none;        /* decorative layers don't capture clicks */
}

.layer.interactive {
  pointer-events: auto;        /* interactive layers DO capture clicks */
  cursor: pointer;
  transition: transform 0.3s ease, filter 0.3s ease;
}

.layer.interactive:hover {
  transform: translateY(-4px); /* slight lift */
  filter: drop-shadow(0 0 12px rgba(255, 234, 167, 0.4)); /* gold glow */
}

/* Pixel art crispness */
.room-scene img {
  image-rendering: pixelated;
}
```

### React component (post-migration)

```tsx
// src/components/rooms/RoomScene.tsx
interface LayerProps {
  src: string;
  alt?: string;
  top: string;
  left: string;
  interactive?: boolean;
  onClick?: () => void;
}

function Layer({ src, alt = '', top, left, interactive, onClick }: LayerProps) {
  return (
    <img
      src={src}
      alt={alt}
      className={`layer ${interactive ? 'interactive' : ''}`}
      style={{ top, left }}
      onClick={onClick}
    />
  );
}

interface RoomSceneProps {
  background: string;
  children: React.ReactNode;
}

export function RoomScene({ background, children }: RoomSceneProps) {
  return (
    <div className="room-scene">
      <img src={background} alt="" className="room-bg" />
      {children}
    </div>
  );
}
```

---

## 8. Hover & click effects

### PNG objects — CSS-based effects

| Effect | CSS | When to use |
|--------|-----|-------------|
| Lift | `transform: translateY(-4px)` | Books, frames, general objects |
| Glow | `filter: drop-shadow(0 0 12px rgba(255,234,167,0.4))` | All interactive objects |
| Scale | `transform: scale(1.05)` | Room zones on hub overview |
| Tilt | `transform: rotate(-2deg)` | Photo frames |
| Pull out | `transform: translateX(10px)` | Books pulling off shelf |
| Bounce | `@keyframes bounce` | Map pins |

### SVG objects — inline style changes

```css
.pin-svg:hover circle {
  fill: #ffeaa7;           /* color change */
  filter: url(#glow);       /* SVG glow filter */
}
```

### Combining PNG + SVG

Use SVG overlays on top of PNG objects for precise hit areas:

```html
<div class="interactive-zone" style="position: absolute; top: 30%; left: 20%;">
  <!-- Visual (PNG) -->
  <img src="/images/library/book-son.png" alt="" />
  <!-- Hit area (SVG, invisible) -->
  <svg class="hit-area" viewBox="0 0 60 120">
    <polygon points="0,0 60,0 60,120 0,120" fill="transparent" />
  </svg>
</div>
```

This is useful when the PNG has irregular shapes and you want the click target to match the visible shape rather than the rectangular bounding box.

---

## 9. Performance checklist

- [ ] Compress PNGs with [TinyPNG](https://tinypng.com/) or `pngquant`
- [ ] Convert backgrounds to WebP (`cwebp input.png -o output.webp -q 80`)
- [ ] Lazy-load off-screen room images (`loading="lazy"`)
- [ ] Use `<picture>` with WebP + PNG fallback for Safari compatibility
- [ ] Inline small SVGs (< 2KB) directly in HTML; reference larger ones via `<img>`
- [ ] Sprite sheets for animation frames (if using frame-by-frame animation)
- [ ] Total art budget target: **< 5MB** for initial load, lazy-load the rest
- [ ] Test on throttled 3G in Chrome DevTools

---

## 10. Handoff checklist for illustrator

When commissioning or creating art, ensure each delivery includes:

- [ ] Source file (PSD / Aseprite / Procreate) with named layers
- [ ] Each interactive object exported as **separate transparent PNG**
- [ ] Background exported as **single merged PNG or WebP**
- [ ] Layer map document showing object positions (screenshot with labels or coordinate list)
- [ ] Consistent canvas size across all rooms (800×600 base)
- [ ] Colors matching the hex palette in section 3
- [ ] @2x exports if applicable
