---
name: tomevault-design
description: AUTO-INVOKE on any task that touches frontend, UI, design, components, layout, page, styling, brand, logo, color, palette, font, typography, motion, animation, copy, button, modal, navbar, sidebar, card, form, table, chart, hero, landing, or anything that ships pixels to tomevault-web. Triggers on file edits to tomevault-web/src/**/*.tsx, tomevault-web/src/app/**, .css, tailwind config, globals.css, or any UI component file. Loads the internal tomevault-design Tome at /home/tomevault/tomevault-design/, which composes four upstream design skills (design-an-interface, impeccable, taste-skill, ui-ux-pro-max) with TomeVault-specific design context (palette, typography, brand, motion, light-dark, deploy discipline, voice, analytics). REPLACES the archived Anthropic frontend-design skill — do not re-introduce that skill. Skip ONLY for pure backend, pure pipeline, pure infra work with no UI surface.
---

# TomeVault Design (Tome Adapter)

This is a thin adapter that points at the internal **tomevault-design Tome** at `/home/tomevault/tomevault-design/`. The Tome is the source of truth — this file just routes you to it and sets the working directory expectation.

## What to do

1. **Read the Tome's orchestrator skill first:**

   ```
   /home/tomevault/tomevault-design/skills/tomevault-design/SKILL.md
   ```

   Follow its instructions. It will tell you which TomeVault context files to load, which of the four bundled skills to apply, and in what order.

2. **Resolve all paths in the orchestrator relative to the Tome root:**

   ```
   /home/tomevault/tomevault-design/
   ```

   When the orchestrator says "load `components/.impeccable.md`", read `/home/tomevault/tomevault-design/components/.impeccable.md`. When it says "run `skills/ui-ux-pro-max/scripts/search.py`", run `/home/tomevault/tomevault-design/skills/ui-ux-pro-max/scripts/search.py`.

3. **The Tome's four bundled skills are at:**

   - `/home/tomevault/tomevault-design/skills/design-an-interface/SKILL.md`
   - `/home/tomevault/tomevault-design/skills/impeccable/SKILL.md` (plus `reference/` subfiles)
   - `/home/tomevault/tomevault-design/skills/taste-skill/SKILL.md`
   - `/home/tomevault/tomevault-design/skills/ui-ux-pro-max/SKILL.md` (plus `data/`, `scripts/`, `templates/`)

4. **TomeVault-specific design decisions live at:**

   `/home/tomevault/tomevault-design/components/`

   These override the bundled skills when they conflict. Palette, typography, brand, motion, light/dark, deploy discipline, analytics, voice.

## When this skill fires

Trigger on any of:

- Frontend / UI / design work on `tomevault-web/`
- Page layout, component design, form / button / modal / nav / card / table / chart
- Brand, logo, favicon, color, font, motion
- Visual review or polish requests

Skip when the task is pure backend, pure infra, or pure pipeline scripting with no UI surface.

## Why this adapter exists

The internal Tome is a separate git repo at `/home/tomevault/tomevault-design/`. Symlinking it into `.claude/skills/` directly would break the orchestrator's relative paths. This adapter is the cheap, removable indirection: parent harness discovers `tomevault-design.md`, which redirects to the real Tome.

Once we publish the Tome and decide to vendor it back into the parent (or install via a future Tome runtime), this adapter goes away.

## Status

The Tome is currently **internal and private** (no public remote, no GitHub repo). We're dogfooding it on the next handful of design surfaces. After 3 uses, we'll evaluate whether the four bundled skill choices, the orchestration routing, and the `components/` coverage are right.

Don't propose changes to the Tome's `components/` files without an actual surface in front of you that needs the change — over-designing the design system pre-dogfood is exactly the failure mode we're trying to avoid.
