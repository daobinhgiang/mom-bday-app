# Technical Migration: Static HTML → Next.js + TypeScript + Supabase

> **Related issue:** [#23 — Migrate from static HTML to Next.js with TypeScript and Supabase](https://github.com/daobinhgiang/mom-bday-app/issues/23)

---

## 1. Why migrate

The current app is a single `index.html` file with inline CSS/JS. This works for a prototype but blocks the roadmap:

| Need | Blocked by static HTML |
|------|----------------------|
| Family members add content | No database, no auth |
| Scheduled reveals (`visible_from`) | No server-side logic |
| Image/audio uploads | No storage backend |
| Multiple rooms with rich interactions | Single file becomes unmanageable |
| Mom-only private bedroom | No auth or permissions |

**Next.js + TypeScript + Supabase** gives us: component architecture, server-side rendering, auth, Postgres, object storage, and realtime — all with generous free tiers.

---

## 2. Current architecture

```
index.html (single file, ~540 lines)
├── <style>        — All CSS inline (~345 lines)
│   ├── Google Fonts: Playfair Display, Quicksand
│   ├── Theme: dark #1a1a2e, gold accents #ffeaa7 / #d4a843 / #b8956a
│   ├── Door reveal animation
│   ├── Room card navigation
│   ├── Room panel styles (library, music, map, family)
│   └── Responsive breakpoint at 600px
│
├── <body>         — All HTML inline
│   ├── #door-screen        — Full-screen door overlay
│   ├── #app                — Main app container
│   │   ├── <header>        — Title + subtitle
│   │   ├── #house-nav      — 4 room cards (grid)
│   │   ├── #room-library   — Books grid (4 books)
│   │   ├── #room-music     — Record player + song list (3 songs)
│   │   ├── #room-map       — Map pins (10 locations, 3 categories)
│   │   ├── #room-family    — Photo wall (6 frames)
│   │   └── <footer>
│
└── <script>       — 4 functions
    ├── openDoor()      — Fades door, shows app
    ├── showRoom(room)  — Hides nav, shows room panel
    ├── goHome()        — Hides all panels, shows nav
    └── toggleRecord()  — Toggles spinning class on record player
```

### Key patterns to preserve
- **State is CSS class toggling** — `.hidden`, `.active`, `.spinning`
- **No routing** — everything is one page, rooms toggle via display
- **Animations** — `fadeUp`, `pulse`, `spin`, hover transforms
- **Content is hardcoded** — 4 books, 3 songs, 10 map pins, 6 photo frames

---

## 3. Target architecture

```
mom-bday-app/
├── src/
│   ├── app/
│   │   ├── layout.tsx              # Root layout, fonts, metadata
│   │   ├── page.tsx                # Door screen → house overview → rooms
│   │   └── globals.css             # Global styles, CSS variables
│   │
│   ├── components/
│   │   ├── DoorScreen.tsx          # Full-screen door overlay
│   │   ├── HouseNav.tsx            # Room card navigation grid
│   │   ├── RoomPanel.tsx           # Shared room panel wrapper (back btn, title, desc)
│   │   ├── BackButton.tsx          # "← Về nhà" button
│   │   │
│   │   └── rooms/
│   │       ├── LibraryRoom.tsx     # Bookshelf grid
│   │       ├── MusicRoom.tsx       # Record player + song list
│   │       ├── MapRoom.tsx         # Map container + pins
│   │       └── FamilyRoom.tsx      # Photo wall grid
│   │
│   ├── lib/
│   │   ├── supabase.ts            # Supabase client (browser)
│   │   ├── supabase-server.ts     # Supabase client (server components)
│   │   └── types.ts               # Database types (generated from Supabase)
│   │
│   ├── hooks/
│   │   └── useRoom.ts             # Room navigation state
│   │
│   └── data/
│       └── seed.ts                # Initial content (migrated from hardcoded HTML)
│
├── public/
│   └── images/                    # Art assets (isometric rooms, sprites)
│
├── supabase/
│   ├── migrations/                # SQL migration files
│   │   └── 001_initial_schema.sql
│   └── seed.sql                   # Seed data (from current hardcoded content)
│
├── next.config.ts
├── tsconfig.json
├── package.json
├── .env.local                     # NEXT_PUBLIC_SUPABASE_URL, NEXT_PUBLIC_SUPABASE_ANON_KEY
├── CLAUDE.md                      # Updated for new architecture
└── index.html                     # Keep as static fallback (freeze current version)
```

---

## 4. Component mapping

How each piece of the current HTML maps to Next.js components:

| Current HTML | Next.js component | State |
|-------------|-------------------|-------|
| `#door-screen` | `<DoorScreen />` | `doorOpen: boolean` |
| `#house-nav` + `.room-card` | `<HouseNav />` | `activeRoom: string \| null` |
| `.room-panel` wrapper | `<RoomPanel />` | Shared layout, receives `children` |
| `.back-btn` | `<BackButton />` | Calls `goHome()` via context/prop |
| `#room-library` + `.bookshelf` | `<LibraryRoom />` | Fetches books from Supabase |
| `#room-music` + `.record-player` | `<MusicRoom />` | `isSpinning: boolean`, fetches songs |
| `#room-map` + `.map-container` | `<MapRoom />` | Fetches locations from Supabase |
| `#room-family` + `.photo-wall` | `<FamilyRoom />` | Fetches photos from Supabase |

### State management

No state library needed. Use React `useState` at the page level:

```typescript
// src/app/page.tsx
'use client';

import { useState } from 'react';

type RoomName = 'library' | 'music' | 'map' | 'family';

export default function Home() {
  const [doorOpen, setDoorOpen] = useState(false);
  const [activeRoom, setActiveRoom] = useState<RoomName | null>(null);

  if (!doorOpen) return <DoorScreen onOpen={() => setDoorOpen(true)} />;
  if (activeRoom) return <RoomPanel room={activeRoom} onBack={() => setActiveRoom(null)} />;
  return <HouseNav onSelectRoom={setActiveRoom} />;
}
```

---

## 5. Supabase schema

### Tables

```sql
-- 001_initial_schema.sql

-- Family members who can contribute content
create table users (
  id uuid primary key default gen_random_uuid(),
  email text unique not null,
  display_name text not null,
  role text not null check (role in ('mom', 'family')),
  avatar_url text,
  lantern_color text,  -- for future visual personalization
  created_at timestamptz default now()
);

-- Library room: memory books
create table books (
  id uuid primary key default gen_random_uuid(),
  title text not null,
  message text not null,
  author_id uuid references users(id),
  color text not null default '#e17055',  -- spine color
  sort_order int not null default 0,
  visible_from timestamptz,  -- null = visible immediately
  created_at timestamptz default now(),
  updated_at timestamptz default now()
);

-- Music room: songs with dedications
create table songs (
  id uuid primary key default gen_random_uuid(),
  title text not null,
  dedication text not null,
  attributed_to text not null,  -- "Con Trai", "Gia Đình", etc.
  audio_url text,  -- optional: Supabase Storage URL
  sort_order int not null default 0,
  visible_from timestamptz,
  created_at timestamptz default now()
);

-- Map room: locations
create table locations (
  id uuid primary key default gen_random_uuid(),
  name text not null,
  category text not null check (category in ('lived', 'visited', 'dream')),
  story text,  -- optional postcard message
  image_url text,  -- optional photo
  visible_from timestamptz,
  created_at timestamptz default now()
);

-- Family room: photos and drawings
create table photos (
  id uuid primary key default gen_random_uuid(),
  image_url text,  -- Supabase Storage URL
  caption text not null,
  author text not null,  -- "2024", "Cháu vẽ", etc.
  emoji_placeholder text,  -- fallback if no image uploaded yet
  sort_order int not null default 0,
  visible_from timestamptz,
  created_at timestamptz default now()
);

-- Activity log for guestbook + logbook features (future)
create table activity_log (
  id uuid primary key default gen_random_uuid(),
  user_id uuid references users(id),
  action text not null,  -- 'added_book', 'uploaded_photo', 'visited', etc.
  room text,
  metadata jsonb,
  created_at timestamptz default now()
);
```

### Row Level Security (RLS)

```sql
-- Everyone can read visible content
create policy "Public read" on books for select
  using (visible_from is null or visible_from <= now());

create policy "Public read" on songs for select
  using (visible_from is null or visible_from <= now());

create policy "Public read" on locations for select
  using (visible_from is null or visible_from <= now());

create policy "Public read" on photos for select
  using (visible_from is null or visible_from <= now());

-- Family members can insert their own content
create policy "Family insert" on books for insert
  with check (auth.uid() = author_id);

-- Mom can read ALL content (including future-scheduled)
create policy "Mom reads all" on books for select
  using (
    exists (select 1 from users where id = auth.uid() and role = 'mom')
  );
-- Repeat for songs, locations, photos
```

### Seed data

Migrate current hardcoded content to `supabase/seed.sql`:

```sql
-- Books (from current index.html lines 395-411)
insert into books (title, message, color, sort_order) values
  ('Cuốn sách của Con Trai', 'Mẹ ơi, con nhớ ngày Mẹ đưa con đi học ngày đầu tiên...', '#e17055', 1),
  ('Cuốn sách của Con Gái', 'Mẹ luôn là người đầu tiên con gọi khi con cần lời khuyên...', '#00b894', 2),
  ('Cuốn sách của Cháu', 'Bà ngoại nấu ăn ngon nhất thế giới!', '#0984e3', 3),
  ('Cuốn sách của Gia Đình', 'Những bữa cơm chiều cuối tuần cùng nhau...', '#fdcb6e', 4);

-- Songs (from lines 422-443)
insert into songs (title, dedication, attributed_to, sort_order) values
  ('Bông Hồng Cài Áo', 'Bài hát này làm con nhớ Mẹ mỗi khi nghe', 'Con Trai', 1),
  ('Mẹ Tôi', 'Mẹ hay hát bài này ru con ngủ', 'Con Gái', 2),
  ('Diễm Xưa - Khánh Ly', 'Bài hát Mẹ nghe nhiều nhất', 'Gia Đình', 3);

-- Locations (from lines 458-469)
insert into locations (name, category) values
  ('Sài Gòn', 'lived'), ('Hà Nội', 'lived'), ('Huế', 'lived'),
  ('Đà Lạt', 'visited'), ('Nha Trang', 'visited'), ('Hội An', 'visited'), ('Phú Quốc', 'visited'),
  ('Paris', 'dream'), ('Tokyo', 'dream'), ('Hạ Long', 'dream');

-- Photos (from lines 478-509)
insert into photos (emoji_placeholder, caption, author, sort_order) values
  ('👨‍👩‍👧‍👦', 'Bữa cơm gia đình ngày Tết', '2024', 1),
  ('🎨', 'Hình vẽ Bà Ngoại của bé Minh', 'Cháu vẽ', 2),
  ('🌴', 'Kỷ niệm đi biển cùng gia đình', '2023', 3),
  ('🎂', 'Sinh nhật Mẹ năm ngoái', '2025', 4),
  ('💎', 'Ngày cưới của Ba Mẹ', 'Kỷ niệm', 5),
  ('✍️', '"Con yêu Bà Ngoại" - bé An', 'Cháu vẽ', 6);
```

---

## 6. CSS migration

### Option: CSS Modules (recommended)

Keep the existing CSS but split it into module files per component. This is the lowest-risk migration path — no new CSS framework to learn, just reorganize.

```
src/
  app/globals.css           # CSS variables, reset, fonts, keyframes
  components/
    DoorScreen.module.css
    HouseNav.module.css
    RoomPanel.module.css
    rooms/
      LibraryRoom.module.css
      MusicRoom.module.css
      MapRoom.module.css
      FamilyRoom.module.css
```

### CSS variables (extract from current inline styles)

```css
/* globals.css */
:root {
  /* Colors */
  --color-bg: #1a1a2e;
  --color-bg-card: #16213e;
  --color-bg-card-hover: #0f3460;
  --color-gold: #ffeaa7;
  --color-gold-dark: #d4a843;
  --color-warm: #b8956a;
  --color-text-muted: #7f8fa6;
  --color-border: #1a3a6a;
  --color-border-hover: #ffeaa7;

  /* Pin colors */
  --color-pin-lived: #e17055;
  --color-pin-visited: #00b894;
  --color-pin-dream: #a29bfe;

  /* Book spine colors */
  --color-book-1: #e17055;
  --color-book-2: #00b894;
  --color-book-3: #0984e3;
  --color-book-4: #fdcb6e;

  /* Fonts */
  --font-display: 'Playfair Display', serif;
  --font-body: 'Quicksand', sans-serif;

  /* Spacing */
  --room-max-width: 800px;
  --nav-max-width: 900px;
}
```

### Font loading (Next.js built-in)

```typescript
// src/app/layout.tsx
import { Playfair_Display, Quicksand } from 'next/font/google';

const playfair = Playfair_Display({
  subsets: ['latin', 'vietnamese'],
  weight: ['400', '700'],
  variable: '--font-display',
});

const quicksand = Quicksand({
  subsets: ['latin', 'vietnamese'],
  weight: ['400', '600'],
  variable: '--font-body',
});

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="vi" className={`${playfair.variable} ${quicksand.variable}`}>
      <body>{children}</body>
    </html>
  );
}
```

---

## 7. Migration steps (in order)

### Phase 1: Scaffold (no Supabase yet)

1. `npx create-next-app@latest mom-bday-app --typescript --app --src-dir`
2. Delete boilerplate, copy `public/images/`
3. Set up `globals.css` with CSS variables and keyframes from current `<style>`
4. Set up fonts in `layout.tsx`
5. Create `DoorScreen.tsx` — port door HTML + CSS
6. Create `HouseNav.tsx` — port room cards
7. Create room components — port each room's HTML + CSS
8. Wire up `page.tsx` with `useState` for navigation
9. **Checkpoint: app looks and behaves identically to `index.html`**

### Phase 2: Add Supabase

10. `npm install @supabase/supabase-js @supabase/ssr`
11. Create Supabase project at [supabase.com](https://supabase.com)
12. Add `.env.local` with Supabase URL + anon key
13. Create `lib/supabase.ts` client
14. Run migration SQL (`001_initial_schema.sql`)
15. Run seed SQL
16. Update room components to fetch from Supabase instead of hardcoded data
17. **Checkpoint: content loads from database, still looks identical**

### Phase 3: Auth + permissions

18. Set up Supabase Auth (magic link)
19. Create login/invite flow for family members
20. Enable RLS policies
21. Add family member content insertion UI
22. **Checkpoint: family can add content, Mom sees everything**

### Phase 4: Storage + media

23. Set up Supabase Storage buckets (photos, audio)
24. Add image upload to Family room
25. Add audio upload to Music room
26. Replace emoji placeholders with real uploaded images
27. **Checkpoint: real media in the app**

---

## 8. Environment variables

```bash
# .env.local (DO NOT commit)
NEXT_PUBLIC_SUPABASE_URL=https://xxxxx.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJhbGciOi...
SUPABASE_SERVICE_ROLE_KEY=eyJhbGciOi...  # server-side only, never expose
```

---

## 9. Development workflow (post-migration)

```bash
# Install dependencies
npm install

# Run dev server
npm run dev
# → http://localhost:3000

# Generate Supabase types (after schema changes)
npx supabase gen types typescript --project-id <project-id> > src/lib/types.ts

# Run Supabase locally (optional)
npx supabase start
npx supabase db reset  # apply migrations + seed

# Build for production
npm run build

# Type check
npx tsc --noEmit
```

---

## 10. What NOT to migrate yet

These are in the roadmap but should NOT be part of the initial migration:

| Feature | Why defer |
|---------|----------|
| Isometric art assets (#6-12) | Art not ready; migration should achieve visual parity with current CSS |
| Garden + Bedroom rooms | New rooms, not a port of existing functionality |
| Weather/time-of-day lighting | Requires external API, ambient features are P5 |
| Realtime updates | Nice-to-have, add after core works |
| PWA / offline | Optimize later |
| `visible_from` scheduling | Schema supports it, but UI for setting dates comes in Phase 3+ |

---

## 11. CLAUDE.md update plan

After migration, update `CLAUDE.md` to reflect the new architecture:

- Change "single-file static site" to "Next.js + TypeScript + Supabase"
- Document component structure and file locations
- Update development commands (`npm run dev` instead of `python3 -m http.server`)
- Document Supabase schema and how to add content
- Update deployment section (Vercel/hosting instead of GitHub Pages)
- Keep key conventions (Vietnamese text, color palette, no external JS libraries beyond React)

---

## 12. Risk mitigation

| Risk | Mitigation |
|------|-----------|
| Visual regression during migration | Keep `index.html` as frozen reference; screenshot-compare at each checkpoint |
| Supabase free tier limits | Very low traffic (~10 users); monitor usage dashboard |
| Over-engineering the migration | Strict rule: port existing functionality first, zero new features in Phase 1 |
| Font loading flash (FOUT) | Next.js `next/font` handles this automatically with font-display: swap |
| Breaking mobile layout | Test at 360px width at every checkpoint |
| Losing hardcoded content | Seed SQL preserves all current content; `index.html` stays in repo as backup |

---

## Issue dependency graph

```
#23 Migration (this doc)
 │
 ├── Phase 1: Scaffold
 │   ├── #3  Standardize font sizes
 │   └── #4  Font loading strategy (solved by next/font)
 │
 ├── Phase 2: Supabase
 │   └── #24 Research hosting → informs deployment target
 │
 ├── After migration complete:
 │   ├── #13 Layered asset system ← depends on component architecture existing
 │   ├── #14 Door screen with art ← depends on DoorScreen.tsx
 │   ├── #15 Room navigation ← depends on HouseNav.tsx
 │   ├── #16-19 Interactive rooms ← depends on room components
 │   └── #20 Room transitions ← depends on navigation system
 │
 └── Art (parallel, no code dependency):
     └── #6-12 Design assets (can be worked on independently)
```
