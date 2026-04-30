# Ngoi Nha cua Me (Mom's House)

A birthday gift web app for Mom — an illustrated Vietnamese-style home she walks through. Each room maps to one of her loves: books, music, travel, and family.

## Rooms

| Room | Vietnamese | What's inside |
|------|-----------|---------------|
| Library | Thu Vien | Memory books from each family member |
| Music Room | Phong Nhac | Songs people associate with Mom, with personal notes |
| Map Room | Phong Ban Do | Pins for places she's lived, visited, and dreams of visiting |
| Family Room | Phong Gia Dinh | Photos and grandkid drawings |

## Getting Started

No build tools, no dependencies. Just HTML + CSS + JS in a single file.

```bash
# Clone and open
git clone https://github.com/daobinhgiang/mom-bday-app.git
cd mom-bday-app
open index.html
```

To serve locally (e.g. for mobile testing):

```bash
python3 -m http.server 8000
# Visit http://localhost:8000
```

## How It Works

1. **Door reveal** — A full-screen door overlay greets the visitor. Clicking opens it with a fade transition.
2. **Room navigation** — Four cards represent rooms. Clicking a card shows that room's content panel.
3. **Room panels** — Each room has its own content section (books, songs, map pins, photos). A "Ve nha" (go home) button returns to the room selection.

All state management is done via CSS class toggling (`.hidden`, `.active`, `.spinning`). There is no routing — everything lives on one page.

## Customizing Content

All content is hardcoded in `index.html`. To personalize:

- **Library**: Edit the `.book` divs (lines ~395-411) — change titles and memory quotes
- **Music**: Edit the `.song-list li` items (lines ~422-443) — change song names and dedications
- **Map**: Edit the `.map-pin` spans (lines ~458-469) — add/remove places, use classes `pin-lived`, `pin-visited`, or `pin-dream`
- **Family**: Edit the `.photo-frame` divs (lines ~478-509) — change emoji placeholders, captions, and dates

## Deploying

This is a static site. Any static host works:

- **GitHub Pages**: Settings > Pages > Source: main branch
- **Netlify/Vercel**: Point to the repo, no build command needed
- **Manual**: Upload `index.html` to any web server

## Future Ideas

- Family members secretly drop new items into rooms (memories, songs, photos)
- Mom can rearrange and add her own content
- Replace emoji placeholders with real photos
- Add actual audio playback for the music room
- Add Vietnamese diacritics toggle for proper text display

## Tech Stack

- Pure HTML/CSS/JS (no frameworks)
- Google Fonts: Playfair Display + Quicksand
- CSS animations and transitions
- Responsive design (mobile-friendly)
