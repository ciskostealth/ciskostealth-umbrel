# Umbrel Community App Store

This repository is a personal Umbrel community app store. The store ID is
`ciskostealth-umbrel` — use it wherever `{store-id}` appears below.

## Structure

- `umbrel-app-store.yml`: Defines the app store metadata (`id`, `name`).
- App directories: Named `{store-id}-{app-id}` (all lowercase), e.g.
  `{store-id}-example-app`.
- Each app directory contains:
  - `umbrel-app.yml`: App metadata (id, name, description, version, etc.).
  - `docker-compose.yml`: Docker configuration for the app.
  - Optionally an icon (e.g. `icon.svg`) and gallery screenshots (e.g. `1.jpg`,
    `2.jpg`), referenced from `umbrel-app.yml` via URL.

## Adding Apps

Use the `define-new-app` skill (`.github/skills/define-new-app/SKILL.md`) for the
process, and `.github/instructions/umbrel-apps.instructions.md` for the field reference
and conventions. Do not duplicate those details here.
