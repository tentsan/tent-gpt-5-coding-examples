# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repo Is

A showcase of demo apps generated entirely from single GPT-5/GPT-5.2 prompts. It contains:
- YAML prompt definitions in `examples/`
- Pre-built demo apps (mostly single `index.html` files) in `apps/`
- A Next.js gallery front-end in `front-end/` that displays and previews all demos

This repo is **not accepting contributions** — it's for reference and inspiration only.

## Commands

```bash
# Development (run from front-end/)
cd front-end
npm install
npm run dev          # copies apps/ -> public/, starts Next.js dev server with Turbopack on :3000

# Build & lint
npm run build        # runs prebuild (copy-docs) then next build (static export)
npm run lint         # next lint
```

## Architecture

### Data Flow

1. **YAML files** (`examples/*.yaml`) define each demo: `id`, `title`, `prompt`, `tags`, optional `camera`/`microphone` flags
2. **`front-end/lib/code-examples.ts`** (`loadApps()`) reads all YAML files at build time from the repo root, parses them, and produces `CodeExample[]` objects with poster URLs (from OpenAI CDN) and iframe URLs
3. **`front-end/lib/load-yaml.ts`** recursively walks the repo root for `.yaml`/`.yml` files (skipping `node_modules`, `.git`, `.next`, `misc`, etc.)
4. **`front-end/scripts/copy-docs.mjs`** (prebuild step) copies `apps/*/` directories into `front-end/public/` so demo apps are served as static files at `/<slug>/index.html`
5. **`front-end/next.config.ts`** detects public subdirectories with `index.html` and sets up rewrites for pretty URLs (`/<slug>` → `/<slug>/index.html`)

### Front-End (Next.js)

- Static export (`output: "export"`) — no server runtime
- `app/page.tsx` is the single route; uses `force-static` and calls `loadApps()` server-side
- UI components: `components/app-grid.tsx` (gallery grid), `components/app-card.tsx` (card), `components/app-modal.tsx` (preview modal with iframe + prompt display)
- Uses shadcn/ui components (`components/ui/`), Tailwind CSS v4, Radix primitives

### Adding a New Example

1. Create `examples/<slug>.yaml` with `title`, `prompt`, and optionally `tags`, `camera`, `microphone`
2. Place the built demo app in `apps/<slug>/index.html`
3. The gallery will pick it up automatically at build time

### Key Conventions

- Demo apps in `apps/` are self-contained static HTML (most are single-file)
- Some demos (e.g., `asteroid-game`, `espresso`) are full Next.js static exports with `_next/` chunks
- Poster images are hosted externally on `cdn.openai.com/devhub/gpt5prompts/<slug>.png`
- The `apps/` directory is duplicated into `front-end/public/` at build time — edit the source in `apps/`, not `front-end/public/`
