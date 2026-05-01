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

## Developer Tooling (Claude Code)

This project uses [Claude Code](https://claude.ai/code) as the primary development tool. Below is how to set up the MCP integrations and GitHub connection for new engineers.

### Connecting GitHub to Claude Code

1. Install the GitHub CLI:
   ```bash
   brew install gh
   ```
2. Authenticate with your GitHub account:
   ```bash
   gh auth login
   ```
3. Claude Code automatically picks up `gh` for creating issues, PRs, and managing the repo — no additional configuration needed.

### Setting Up Playwright MCP

Playwright MCP lets Claude Code control a browser to visually test the app, click through rooms, and take screenshots.

1. Add the Playwright MCP server to your Claude Code settings (`~/.claude/settings.json` or project `.claude/settings.json`):
   ```json
   {
     "mcpServers": {
       "playwright": {
         "command": "npx",
         "args": ["@anthropic-ai/mcp-playwright"]
       }
     }
   }
   ```
2. Restart Claude Code. You should see Playwright tools available (e.g. `browser_navigate`, `browser_snapshot`, `browser_click`).
3. Usage example — ask Claude Code:
   - "Open index.html in the browser and take a screenshot"
   - "Click the door to open it, then navigate to the Music Room"
   - "Check if the app is responsive at 375px width"

### Setting Up Context7 MCP

Context7 fetches up-to-date library documentation so Claude Code has accurate API references.

1. Add to your Claude Code settings:
   ```json
   {
     "mcpServers": {
       "context7": {
         "command": "npx",
         "args": ["-y", "@anthropic-ai/context7-mcp@latest"]
       }
     }
   }
   ```
2. Claude Code will automatically use this when you ask about library APIs (e.g. Google Drive API, Web Audio API).

### Setting Up Google Drive (via gcloud CLI)

Google Drive is used for storing app content (images, audio, documents). We use the `gcloud` CLI to sync files locally.

1. Install gcloud CLI:
   ```bash
   curl -sSL https://sdk.cloud.google.com | bash -s -- --disable-prompts --install-dir=$HOME
   source "$HOME/google-cloud-sdk/path.zsh.inc"
   ```
2. Authenticate:
   ```bash
   gcloud auth login
   ```
3. Enable the Drive API in your Google Cloud project:
   ```bash
   gcloud services enable drive.googleapis.com
   ```
4. Use `gsutil` or the Drive API to sync assets to/from the `images/` folder.

### Other Useful MCP Servers

| MCP Server | Purpose |
|------------|---------|
| **Playwright** | Browser automation, visual testing, screenshots |
| **Context7** | Up-to-date library/API documentation |
| **Slack** | Send updates to team channels |
| **Google Calendar** | Schedule collaboration sessions |
| **Notion** | Sync project docs and notes |
| **Google Drive** | Content storage for app assets (community MCP, not official) |

> **Tip**: Run `/help` inside Claude Code to see all available tools and MCP connections.

## Tech Stack

- Pure HTML/CSS/JS (no frameworks)
- Google Fonts: Playfair Display + Quicksand
- CSS animations and transitions
- Responsive design (mobile-friendly)
