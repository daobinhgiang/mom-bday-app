---
name: ngoi-nha-cua-me-design
description: Use this skill to generate well-branded interfaces and assets for Ngôi Nhà Của Mẹ ("Mom's House") — a private, hand-painted Vietnamese-home web app gift. Contains design tokens, painted assets, type rules, motion principles, and a UI kit. Use for production, prototypes, mocks, or related deliverables (cards, postcards, slides for the family).
user-invocable: true
---

Read the `README.md` in this skill first — it contains the emotional north star, content/voice rules, visual foundations (palette, type, illustration, motion, sound, time-of-day), iconography rules, and component patterns.

Then explore:
- `colors_and_type.css` — design tokens (colors, type, spacing, shadows, easings)
- `assets/` — painted SVGs (logo, lantern, house glyph, bookmark, folded note, house cross-section, paper/watercolor textures)
- `preview/` — small specimen cards demonstrating the system
- `ui_kits/app/` — interactive recreation of the app (door, hub, six rooms, farewell)

If creating visual artifacts (HTML mocks, prototypes, postcards, slides), copy assets from `assets/` and use tokens from `colors_and_type.css`. If working on production code, lift the same tokens.

**Critical rules (do not break):**
- Vietnamese with full diacritics. Test every type with `Mẹ ơi, về nhà thôi.`
- No emoji, no icon font, no Lucide. Painted SVGs only.
- No flat material design, glassmorphism, gradients-as-decoration, or modern-SaaS aesthetics.
- No badges, red dots, streaks, notifications, or onboarding.
- Never pure black on pure white. Use `--ink` on `--paper-cream`.
- Easings have settle. Never linear. Default 480ms.
- "Home or app?" — when in doubt, choose home.

If the user invokes this skill without other guidance, ask whether they're making a postcard, a new room, a slide for the birthday, a printable, or production code. Then ask 2–3 questions and act as an expert designer who outputs HTML or production code as needed.
