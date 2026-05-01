# Ngôi Nhà Của Mẹ — Design System

> *"Một ngôi nhà mà Mẹ sống trong đó."*
> A web app gift for a Vietnamese mother turning 60. Every room is a way her family loves her.

---

## What this is

**Ngôi Nhà Của Mẹ** ("Mom's House") is a private, single-purpose web app gift — not a product, not a platform. It is a single illustrated cross-section of a cozy Vietnamese home. Mom explores rooms; family secretly leaves things inside them.

Rooms:
- **Thư Viện** — library of written memories, one book per family member
- **Phòng Nhạc** — music room, songs dedicated to her
- **Phòng Bản Đồ** — map of places she's lived, visited, and dreams of
- **Phòng Gia Đình** — family room with photos and voice notes
- **Phòng Ngủ** — her private bedroom for her own writing
- **Vườn** — garden for the grandkids' drawings

This document is the design brief turned into rules — colors, type, motion, sound, components — so that every screen, slide, and asset built from this system feels like the same warm, slightly imperfect home.

---

## Emotional north star

> Coming home at dusk after a long day.

Warm, hand-made, slightly imperfect, full of small details left by people who love her.

It is **not** a productivity app, social network, or photo-sharing tool. It is closer to a children's picture book, a handwritten journal, and a music box — fused into one quiet world.

**Every design decision answers: would this feel like a *home*, or like an *app*? When in doubt, choose home.**

---

## Sources

- **GitHub:** `daobinhgiang/mom-bday-app` (default branch `main`) — currently a single `index.html` with a placeholder dark-blue/gold version. The aesthetic in this repo does **not** match the design brief; the system below is built from the brief.
- **Design brief:** "Ngôi nhà của Mẹ — Design Brief" (provided by user, included verbatim as moodboard rules).
- **Aesthetic references** (curated, not assets):
  - Studio Ghibli interiors — *Whisper of the Heart*, *When Marnie Was There*
  - Vietnamese illustrators — Tâm Bùi, Khoa Lê, Camelia Pham
  - Old Saigon photography — ochre walls, blue shutters, terracotta roofs
  - Children's books — Beatrix Potter, Jon Klassen, Vietnamese folk tales
  - Skeuomorphic Apple apps (iBooks shelf, Notes leather) — but more handmade

---

## Index

| File / Folder | Purpose |
|---|---|
| `README.md` | This file — brand, content, visual, iconography rules |
| `colors_and_type.css` | All design tokens (CSS variables) — colors, type, spacing, shadows, easings |
| `fonts/` | Webfonts (Vietnamese-supporting handwriting + serif) |
| `assets/` | Painted backgrounds, room illustrations, paper textures, lanterns, logo |
| `preview/` | Cards rendered in the Design System tab |
| `ui_kits/app/` | The full app UI kit — interactive recreation of the house |
| `SKILL.md` | Skill manifest for Claude Code / Agent Skills |

---

## Content fundamentals

### Voice
Warm, intimate, family. **Mẹ** (Mom) — never "user", never her first name. Children speak as **con** (child), grandchildren as **cháu**.

Vietnamese with **full diacritics**, always. Never strip dấu marks (`Mẹ`, never `Me`). Test `Mẹ ơi, về nhà thôi` in every type specimen — if it doesn't render perfectly, the font is wrong.

### Tone
- **Warm, not formal.** Avoid 官方/governmental Vietnamese. Use the way a grown child speaks softly to their mother.
- **Quiet, not cheerful.** No exclamation points unless someone is genuinely calling out. No "Welcome back!" energy.
- **Specific, not generic.** "Cuốn sách của Con Trai" (Son's book), not "User's content". Names, relations, places.
- **Slightly imperfect.** A typo left in a child's drawing caption is fine. Charming.

### Examples (good)
- `Hẹn gặp Mẹ ngày mai.` — closing screen
- `Cuốn sách của Con Trai` — book label, library
- `"Mẹ ơi, con nhớ ngày Mẹ đưa con đi học ngày đầu tiên..."` — book content
- `"Bài hát này làm con nhớ Mẹ mỗi khi nghe" — Con Trai` — song dedication
- `Bà ngoại nấu ăn ngon nhất thế giới!` — grandkid drawing caption
- `Mở cửa vào nhà` — door tap label

### Examples (bad)
- `Welcome to your dashboard` — app voice
- `You have 3 new items!` — notification voice
- `Tap to continue →` — generic CTA
- `Get started` — onboarding voice
- `Nhấn để xem thêm` — too clinical, even in Vietnamese

### UI copy is minimal
The app should have **almost no UI text**. Rooms are recognized by their illustration, not labels. Where copy is needed, it is:
1. Family content (a book title, a song dedication, a postcard) — handwritten
2. Mom's own writing — handwritten
3. Soft ambient labels (`Hẹn gặp Mẹ ngày mai`) — serif

### No emoji.
Not in copy, not in UI. Use painted icons, lantern glyphs, or nothing.

### Casing
Vietnamese sentence case. Proper nouns (place names, family relation labels treated as titles like *Con Trai*) capitalized.

---

## Visual foundations

### Palette philosophy
> **Aged photograph, not screen-bright.**
> If a color looks like it came from a CSS picker, it's wrong.

All colors live in `colors_and_type.css`. The system has four bands:

1. **Surfaces** — ochre walls, warm cream, faded teak. These cover 90% of the canvas.
2. **Pigments** (accents) — lantern yellow, lacquer red, jade green, indigo blue. Used like real pigments: *sparingly*, never as backgrounds.
3. **Time-of-day light** — golden hour, soft lamp yellow, cool moonlight. These shift across the day as a global filter.
4. **Ink** (text) — dark brown-black on cream. **Never** pure black on pure white.

**Banned**: electric blue, hyperlink blue, alert red, generic grays, neon, gradients-as-decoration.

### Type
Two voices:
- **Handwritten** — `Caveat` (Google Fonts; substitutes for a custom Vietnamese handwriting font). Used for personal content: journal entries, postcards, family contributions, book content.
  - **⚠ Substitution flag:** Caveat has limited Vietnamese diacritic optical quality. The brief calls for a custom-commissioned handwriting font that handles `dấu` marks beautifully. Replace when available.
- **Serif body** — `Lora` (Google Fonts). Used for longer reading: book chapters, postcards, the rare ambient UI label. Lora has good Vietnamese coverage.
- **No sans-serif tech fonts.** Inter, SF, Roboto, Quicksand — all break the spell.

Test string for any new font: `Mẹ ơi, về nhà thôi.`

### Illustration & backgrounds
The single most important design element. Every surface in the app is **painted**, not designed.
- Hand-painted watercolor + ink linework feel
- Soft edges, visible brush texture, occasional bleeds
- Vietnamese architectural specifics: tile roof (mái ngói), wooden shutters, woven mats (chiếu), ceramic tea set, low wooden furniture
- Characters drawn as small lantern-carrying figures, not photo-realistic
- Every room must contain ≥5 ambient details that aren't interactive: steaming teapot, open book, slippers, moth at a window

**Production note:** Real watercolor illustrations should be commissioned. AI illustration is a fallback that needs heavy curation — the consistency problem will break the world. In this design system, painted-look placeholders use SVG with grain overlays + hand-drawn paths; treat them as scaffolding.

### Backgrounds
- **Paper-grain cream** is the default canvas (`--paper-cream`).
- **Full-bleed painted scenes** for hero/room views — house cross-section, room interior.
- **Repeating textures**: subtle paper grain (always-on, multiply blend at ~6% opacity), woven mat texture for floors.
- **No flat fills on top-level surfaces.** Everything has a slight tonal variation, like watercolor pooling.

### Layout
- Asymmetric, hand-placed feel. No 12-col grid rigor.
- Generous margins. Whitespace is breath.
- Objects sit on surfaces (a book on a shelf, a postcard on a desk) — they don't float in space.
- No fixed nav, no sticky headers, no toolbars. The illustration *is* the nav.
- Logbook by the front door is the only list-based UI.

### Corners & borders
- **Corner radii**: not uniform. Painted edges are organic. CSS containers use `border-radius: 2px–4px` (subtle paper edge) or no radius. **No 16px pill-corner UI.**
- **Borders**: painted ink lines, slightly imperfect. In CSS, this means 1px solid `--ink-soft` with optional dashed/wavy SVG overlay for emphasis.

### Shadows & elevation
Shadows are *cast*, not glowed.
- `--shadow-paper` — tiny offset, brown-ish, like paper on a desk
- `--shadow-object` — slightly deeper, for books, photos, lanterns lying on surfaces
- `--shadow-room` — soft, large, warm; for a focused element inside a dim room
- **No blue shadows.** No `0 0 60px rgba(0,0,0,0.3)` modal-style shadows.
- Inner shadows sparingly, only for inset surfaces (a drawer, a niche).

### Cards
"Card" is a misleading word here. There are:
- **Postcards** — paper texture, slight tilt (`rotate(-1deg)`), torn or scalloped edge, handwritten content
- **Books** — leather/cloth spine, embossed title, sit on shelves
- **Photo frames** — wooden border, photo inside, sometimes tilted
- **Notes** — folded paper, sometimes pinned with a painted pin

None of these are flat rectangles with `box-shadow: 0 4px 12px rgba(0,0,0,0.1)`. If you find yourself reaching for that, you're designing an app, not a home.

### Transparency & blur
- Almost never. The world is solid and physical.
- Exception: dimming overlay at session-close (`Hẹn gặp Mẹ ngày mai`) — a warm amber-to-deep-blue wash.
- Exception: weather overlays — rain streaks, fog. These are watercolor-like, not gaussian-blurred.

### Imagery vibe
- Warm. Always warm. Even rain is warm.
- Slight grain, like 35mm film or old photo paper.
- Saturation low-to-medium. Never crushed blacks or blown highlights.
- B&W photography is fine inside a frame (a photo of grandparents) — never as a UI motif.

### Animation principles
> **Movement should feel physical, not digital.**

- Default easing: **slow ease-out with a settle** — `cubic-bezier(0.22, 0.9, 0.32, 1)` followed by a small overshoot of <2%.
- Doors swing on hinges, with overshoot
- Books pull off shelves with weight (a downward dip before lifting)
- Records: lift → swing → drop, three discrete motions, not a single tween
- Postcards flip with paper-bend physics (CSS rotateY + slight rotateX during flip)
- Curtains sway continuously, slowly, slightly different rates per curtain
- Lantern flames flicker on a 4–6 second irregular cycle
- The cat moves between rooms once every ~10 minutes, real time

**Banned animations:**
- Linear easing
- Instant transitions
- Snappy/bouncy "delightful" microinteractions
- Spring physics that feel like iOS

Default duration: **400–700ms**. Never <250ms.

### Hover & press states
- **Hover** (desktop): a subtle warming — slightly more saturation, slight scale (1.01–1.02), slight tilt for objects (`rotate(-1deg)`). Never a darker fill or border-color change to a hyperlink color.
- **Press**: the object dips slightly, like you're pressing on paper. `translateY(1px) scale(0.99)`.
- **No focus rings** in the system blue. If a focus ring is needed for accessibility, use a soft ochre outline.

### Time & weather responsiveness
The house lives in real time:
- **Sunrise** — warm gold filter, birds added to ambient layer
- **Noon** — bright cream filter
- **Sunset** — amber filter, lanterns begin to glow
- **Night** — deep blue filter with lantern halos, crickets in ambient layer
- **Rain** — water on windows, rain layer rises in mix
- **Tết** — red banners, peach blossoms, lì xì envelopes
- **Birthday week** — flag on roof, cake on kitchen counter
- **Giỗ / Vu Lan** — incense smoke from bàn thờ niche (only if family confirms tradition)

These are global SVG filter overlays + small environmental sprites. Implemented as a `--time-filter` and `--weather-filter` CSS variable layer.

### New-content indicators
**No badges. No red dots. No numbers.**

New content shows as:
- A glowing lantern in the window of a room
- A bookmark sticking out of a book
- A blinking answering machine light
- The cat sitting next to the newest item
- A folded note in the family room jar

### Sound
Sound is **half the experience**. Treat it as a first-class design element.
- Per-room ambient base layer (rain + page rustles in library; vinyl static in music room; etc.)
- Interaction sounds recorded from real objects, never synthesized (door creak, page turn, vinyl drop)
- Time-of-day adds layers (crickets at night, birds at morning)
- **Master mute**: one tap, top-right, always visible. **Default-on**.

---

## Iconography

> **There is almost no iconography in this app.**

The brief is explicit: rooms are recognized by their illustration, not by icons or labels. The app has no nav bar, no hamburger, no settings gear, no share button.

Where icon-like glyphs *do* appear, they are:
- **Painted objects**, not iconic glyphs. A "music room" is a room with a turntable painted in it, not a `🎵` or a record-icon SVG.
- **A small house** in the top-left corner of any room — to return to the hub. This is painted as a framed photo on the wall, not rendered as an icon font.
- **A lantern** as the new-content indicator — painted, glowing, animated.
- **A bookmark** sticking out of a book — painted, not an icon.

**No emoji.** The placeholder repo currently uses emoji (`📚 🎵 🌐 📷`) for rooms — these are explicitly **wrong** and must be replaced with painted illustrations before the app ships.

**No icon font.** No Lucide, Heroicons, Material Icons, FontAwesome.

**Unicode chars** are not used as icons either, with one acceptable exception: a back-arrow `←` for the rare list-view back button (Logbook). Even this should be replaced with a painted arrow when illustration capacity allows.

In `assets/`, painted SVG glyphs are provided for: house-icon (return-to-hub), lantern (new-content), bookmark, folded-note, paper-pin. These are the *only* glyph-like assets in the system; everything else is a full illustration.

---

## How to use this system

1. **Read the brief.** Then read `colors_and_type.css`. Then look at `preview/` cards.
2. **Open a UI kit** in `ui_kits/app/index.html` to see the system in motion.
3. **When stuck**, ask: *home or app?* Choose home.
4. **When adding a new screen or component**, copy a painted asset from `assets/` instead of drawing one from scratch.
5. **When picking a color**, use a CSS var. Never hand-pick hex.
6. **When adding text**, ask Mẹ first — she's the only audience.
