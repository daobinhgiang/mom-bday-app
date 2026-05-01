# Mom’s House — implementation plan

**What it is:** A birthday gift web app — illustrated Vietnamese home Mom explores; family adds memories behind the scenes. Today: single-file prototype [`index.html`](index.html). Target: illustrated hub + backend + scheduling.

**Non‑negotiable:** **Illustration drives the schedule.** Ship interactions on real layered art (not gray placeholders). Human illustrator + Vietnamese-home references preferred over AI full scenes.

---

## Principles

| Principle | Means |
|-----------|--------|
| Feeling > features | Art + pacing matter as much as “features.” |
| Mom first | Private bedroom; she controls who reads her journal. |
| Quiet discovery | Lanterns, bookmarks, answering-machine light, cat by newest, optional logbook — **not** notifications-first. |
| Real Vietnamese home | Tile roof, shutters, mats, ceramics, wall yellow — not generic cottage. |
| Works on her phone | Fast loads, readable type; audio after tap if browsers require it. |

---

## Today → target

| Area | Now | Goal |
|------|-----|------|
| Art | Layout placeholders | Layered commissions: exterior, dollhouse hub, 6 rooms, intro figures, props + lighting/Tết/birthday variants |
| Opening | Door tap → app | Black → rain/chime → line **“Mẹ ơi, về nhà thôi.”** → exterior → tap door → ~20s family tableau → hub |
| Hub | 4 cards | **Dollhouse cross‑section**; see all rooms; ambient motion |
| Rooms | Library, Music, Map, Family | **+ Garden, Bedroom**; richer interactions per room |
| Data | Static HTML | Auth, uploads, **`visible_from`** scheduling, permissions |
| Ambience | None | Time-of-day light; optional rain from **her** weather (with consent) |
| Family | — | Login, lantern color/avatar, drag items in, schedule reveals |

---

## Flows (short)

**Mom — first visit:** Card QR/URL → opening sequence → exterior → door → intro animation → explore hub.

**Mom — later:** Optional shorter entry if intro already seen; exit line **“Hẹn gặp Mẹ ngày mai.”** + cat on couch.

**Mom — hints for new stuff:** Window lantern, book ribbon, answering-machine blink, cat placement, front-door logbook — **no** red-dot nagging.

**Family:** Same visuals + edit overlays; drop content; optional **appear-on date**.

**Bedroom:** Hers only — drafts, unsent audio, letters queued for future dates.

---

## Rooms — what to build

One line each: **Art** (illustrator) · **Product** (you).

| Space | Art | Product |
|-------|-----|---------|
| **Hub + exterior** | Cross‑section master + facade; layers for lanterns/doors/cat; morning/evening/night/rain; Tết strips; birthday flag | Hotspots → room focus; cat path + “by newest”; front-door logbook |
| **Library** | Shelves, desk/journal, readable bookmark ribbons; spine system (sprites or tinted strips) | Books open to chapters (text, photos, audio); spine thickens when updated; Mom writes + shares rules; visitor guestbook |
| **Music** | Turntable, crate, piano/sheets, mic; curtain/light beams for “playing”; **vinyl frame** for uploads | Pick record → play + dedication card; piano/karaoke; Mom recordings → new crate items for family |
| **Map** | Big map wall; desk drawer; globe/postcards; 3 pin colors | Pins → postcards (photos, story, voice); gold “dream” → ticket → turns orange when trip happens; Mom-only pins |
| **Family** | Messy warm room: frames, fridge zone, couch/TV, album, machine + jar | Video on couch; curated albums; voicemail UX; jar **one note/day**; “today’s photo” frame |
| **Garden** | Yard, clothesline papers, patch, swing | Animated line drawings; swing borrows music-room audio softly |
| **Bedroom** | Softer room; locked door on dollhouse | Auth-only; journal/drafts; scheduled delivery |

---

## Data (mental model)

**Users:** Mom vs family — avatar, lantern color.

**Contributions:** Typed payloads (chapter, record, pin/postcard, photo, video, voice, jar note, …), **`visible_from`**, room + slot for drag‑drop layout.

**Books:** Chapters + media; “has new” = ribbon from latest chapter date.

**Activity log:** Powers guestbook + logbook.

**Media:** Files in object storage (S3/R2/etc.); metadata in Postgres/Firestore (pick later).

---

## Tech phases

| Phase | Focus |
|-------|--------|
| **A** | Intro + exterior + door on static host; **gate:** layered exterior + hub cross‑section + **one** finished room before calling it done |
| **B** | Split code if needed; auth (magic link + invites); API + DB + storage |
| **C** | Uploads, **`visible_from`**, jobs for scheduled letters / rotations |
| **D** | Weather (city or coarse geo); cheap audio‑reactive motion |

**Rendering:** SVG overlay + raster textures (see below). Canvas/WebGL later only for rain/particles if needed.

---

## SVG overlay architecture

### Concept

Each room is a **full-scene raster PNG** (AI-generated) with an **invisible SVG layer** on top. Clickable items are **separate transparent PNGs** composited as `<image>` elements inside the SVG. This gives us DOM-level hit-testing, CSS effects, and art-swappable structure — without hand-drawing vector art.

### What to generate (AI image generation)

| Asset | Format | Notes |
|-------|--------|-------|
| Room background | PNG/WebP | Full scene — walls, floor, furniture, lighting. One per room. |
| Clickable items | PNG, transparent bg | One per interactable (bookshelf, record player, photo frame, etc.). Must match perspective/scale of the room background it sits on. |

**Tip:** Generate the full room first, then generate each item in the same style/angle — or use AI background removal to isolate items from the full scene.

### How it works

```
+---------------------------+
|  <div class="room">       |
|    <img> room-bg.png      |   <-- full raster background
|    <svg viewBox overlay>   |   <-- same dimensions, sits on top
|      <g id="bookshelf">   |
|        <image> item.png   |   <-- positioned to align with bg
|      </g>                 |
|      <g id="record">      |
|        <image> item.png   |
|      </g>                 |
|    </svg>                 |
+---------------------------+
```

### SVG structure per room

```xml
<div class="room" id="room-library">
  <img src="rooms/library-bg.png" class="room-bg" />
  <svg viewBox="0 0 1024 1024" class="room-overlay">

    <g id="bookshelf" class="interactable"
       data-type="poem" data-content-id="poem-1"
       role="button" aria-label="Bookshelf — click to read">
      <image href="items/bookshelf.png" x="120" y="400" width="200" height="300" />
    </g>

    <g id="vinyl-player" class="interactable"
       data-type="music" data-content-id="song-1"
       role="button" aria-label="Record player — click to listen">
      <image href="items/vinyl-player.png" x="600" y="500" width="180" height="150" />
    </g>

  </svg>
</div>
```

### CSS effects on interactables

```css
.room {
  position: relative;
  display: inline-block;
}
.room-bg {
  width: 100%;
  display: block;
}
.room-overlay {
  position: absolute;
  top: 0; left: 0;
  width: 100%; height: 100%;
}

/* Hover: glow + slight scale */
.interactable {
  cursor: pointer;
  transition: filter 0.3s ease, transform 0.3s ease;
  transform-origin: center;
}
.interactable:hover {
  filter: drop-shadow(0 0 12px rgba(212, 168, 67, 0.7));
  transform: scale(1.05);
}

/* Click: zoom in */
.interactable.active {
  transform: scale(1.15);
  filter: drop-shadow(0 0 20px rgba(255, 234, 167, 0.9));
}
```

### JS handler (generic, data-driven)

```js
document.querySelectorAll('.interactable').forEach(el => {
  el.addEventListener('click', () => {
    const { type, contentId } = el.dataset;
    openContent(type, contentId);  // one function handles all content types
  });
});
```

Adding a new interactable = adding a `<g>` with `data-*` attributes. No new JS needed.

### Effects summary

| Effect | How | Separate item PNG needed? |
|--------|-----|---------------------------|
| Gold glow on hover | `filter: drop-shadow()` on SVG `<g>` | Yes (for glow to trace item shape) |
| Scale / zoom | `transform: scale()` on SVG `<g>` | Yes (item lifts off background) |
| Shadow / lift | `filter: drop-shadow()` | Yes |
| Color tint | `filter: sepia() hue-rotate()` | Yes |
| Outline highlight | SVG `<rect>` stroke behind `<image>` | No (but looks less polished) |

### Workflow

1. **AI generates** room backgrounds + individual item PNGs (pixel art / isometric style)
2. **Engineer builds** SVG overlay per room — positions `<image>` elements to align with background
3. **CSS handles** all hover/click effects — no per-item JS
4. **Content system** maps `data-content-id` to the actual content (poem, song, photo, memory)
5. **Swapping art** = replace PNG files, adjust `x/y/width/height` if needed — structure and interactions unchanged

---

## Illustration — brief, handoff, QA

**Brief:** Mood board (architecture + Saigon-yellow palette); warm Ghibli‑adjacent **not** toy-flat; intro figures readable small; **cat** sheet (walk, sit, couch loaf).

**Exports (agree once):**

| Piece | Give engineering |
|-------|------------------|
| Exterior | Layers: sky, house, lantern glow, porch, door states, wet/rain |
| Hub | Layered cross‑section + flat fallback; lantern/Tết/birthday overlays |
| Rooms | Background + separate FX (curtains, steam, motes) where animated |
| Props | PNG sprites (crate, machine, jar, postcards) |

**Minimum asset set:** Opening handwriting asset · exterior (+ rain, Tết, birthday) · hub × ~4 lighting moods · 6 room heroes · intro tableau · closing dim + cat · edit-mode hotspot guide (ghost overlay optional).

**QA:** Reads Vietnamese-home to locals · taps match painted doors · lighting consistent · assets compressed + lazy-loaded.

**Sound:** Different stems for intro / exterior / hub / rooms; spot SFX (door, page, needle, tape deck). Typography: Unicode for Mom-facing copy vs ASCII prototype — **decide**.

---

## Legal / safety (don’t skip)

Consent for faces/voices · copyrighted songs (prefer uploads / licensing) · kids’ PII minimal · Mom account recovery · eventual **export** of her memories.

---

## Roadmap

| Milestone | Ship |
|-----------|------|
| **P0** | Intro + exterior + door + **real exterior art** (contract + mood board done) |
| **P1** | Dollhouse hub in illustrated cross‑section + **≥2** painted rooms + fake “new” cues |
| **P2** | Auth + one room end‑to‑end with uploads (e.g. Library) |
| **P3** | Rest of rooms + scheduling + logbook |
| **P4** | Bedroom lock, jar rule, voicemail polish, karaoke |
| **P5** | Weather automation + polish |

Swap **P3/P4** if bedroom trust matters sooner.

---

## Open decisions

Hosting scale · Safari autoplay/storage · diacritics in UI · weather = geolocation vs city · video = self-hosted vs unlisted embed · post‑birthday hosting cost · illustrator revisions vs slip plan (hero-only fallback) · record/postcard art = **templates** vs custom each time.

---

## Done when

It feels like **her house**, not an app · surprises appear **on the date set** · she finds updates by **exploring** · art reads **Vietnamese home** to someone who knows one.
