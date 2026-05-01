# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

"Ngoi Nha cua Me" (Mom's House) — a birthday gift web app styled as an illustrated Vietnamese home. The user opens a front door to reveal four rooms, each representing one of Mom's interests.

## Architecture

**Frontend**: A single-file static site (`index.html`) hosted on **GitHub Pages**. No build step, no framework. All HTML, CSS, and JavaScript are inline. Uses vanilla JS + Web Components.

**Backend**: **Supabase** provides four services in one:
- **Postgres database** — stores book chapters, map pins, journal entries, song dedications, memory jar notes, etc.
- **Auth** — email/password or magic link login. Each family member has their own account.
- **Storage** — file hosting for photos, audio, voice memos. Serves all media assets.
- **Row-Level Security (RLS)** — enforced inside Postgres. Controls who can read/write what (e.g., Mom's journal is private unless she shares it).

There is no separate backend server. The frontend calls Supabase directly via its JS client library.

### Structure within index.html

- **CSS** (`<style>` block): Google Fonts (Playfair Display, Quicksand), dark theme with gold accents (#ffeaa7, #d4a843, #b8956a), CSS animations for door reveal/room transitions
- **Door screen** (`#door-screen`): Full-screen overlay, dismissed on click via `openDoor()`
- **Room navigation** (`#house-nav`): Four clickable cards that show/hide room panels
- **Room panels** (`#room-library`, `#room-music`, `#room-map`, `#room-family`): Each toggled via `.active` class
- **JavaScript** (bottom `<script>`): Core functions — `openDoor()`, `showRoom(room)`, `goHome()`, `toggleRecord()`, plus Supabase data fetching

### Room content pattern

Each room panel follows the same structure: back button, h2 title, description paragraph, then room-specific content grid. Content is loaded from Supabase (database for structured data, storage for media files).

## Development

Open `index.html` directly in a browser. No server needed.

To serve locally (useful for testing on mobile):
```
python3 -m http.server 8000
```

## Deployment

Deploy as a static site. Enable GitHub Pages (Settings > Pages > Source: main branch) to publish at `https://<user>.github.io/mom-bday-app/`.

## Key Conventions

- Vietnamese text uses **no diacritics** (ASCII-safe) throughout the UI
- Dark background (#1a1a2e) with gold/warm accent palette
- Body text color is `black` (set intentionally)
- All interactivity is CSS class toggling — no external JS libraries
- Supabase JS client (`@supabase/supabase-js`) is the only external dependency, loaded via CDN
- All media (photos, audio, voice memos) served from Supabase Storage — no Google Drive
