# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Quartz v4 is a static site generator for publishing markdown notes as a digital garden website. It converts markdown (with Obsidian-flavored markdown support) into an interconnected, searchable static site with graph visualization, backlinks, and full-text search.

## Common Commands

- `npx quartz build` — Build the static site
- `npx quartz build --serve` — Build and serve locally with hot reload
- `npx quartz build --watch` — Watch mode for incremental rebuilds
- `npm run check` — Type check (tsc) + format verification (prettier)
- `npm run format` — Auto-format with prettier
- `npm test` — Run tests (Node.js test runner via tsx)
- `npx quartz sync` — Sync content to/from GitHub

## Architecture

### Plugin Pipeline

The core architecture is a three-phase plugin pipeline:

```
Markdown → [Transformers] → [Filters] → [Emitters] → Static HTML
```

1. **Transformers** (`quartz/plugins/transformers/`) — Process markdown/HTML ASTs using unified.js (remark + rehype). Each transformer can hook into text pre-processing, markdown plugins, and HTML plugins.
2. **Filters** (`quartz/plugins/filters/`) — Decide which content gets published (e.g., `RemoveDrafts`).
3. **Emitters** (`quartz/plugins/emitters/`) — Generate output files (HTML pages, RSS, sitemaps, OG images). Support `partialEmit` for incremental builds.

Plugins are factory functions returning plugin instances. Plugin interfaces are defined in `quartz/plugins/types.ts`.

### Processing Pipeline

- `quartz/processors/parse.ts` — Reads markdown, applies transformer chains via unified.js
- `quartz/processors/filter.ts` — Applies filter plugins
- `quartz/processors/emit.ts` — Runs emitter plugins to produce output
- `quartz/worker.ts` — Worker thread for parallel markdown parsing
- `quartz/build.ts` — Build orchestrator handling full and incremental builds

### Component System

UI uses **Preact** (not React). Components are in `quartz/components/` and follow the `QuartzComponent` interface, which extends Preact components with optional `css`, `beforeDOMLoaded`, and `afterDOMLoaded` static resources.

### Configuration

- `quartz.config.ts` — Main config: plugin selection, theme, analytics, locale
- `quartz.layout.ts` — Page layout composition (which components go in header, sidebar, footer, etc.)

### Path System

`quartz/util/path.ts` uses branded/nominal types (`FilePath`, `FullSlug`, `SimpleSlug`, `RelativeURL`) to prevent mixing different path representations at the type level.

## Code Style

- TypeScript strict mode with `noUnusedLocals` and `noUnusedParameters`
- Prettier: 100 char width, no semicolons, 2-space indentation, trailing commas
- ESM (`"type": "module"`) throughout
- JSX via Preact (`jsxImportSource: "preact"`)
- Conventional Commits for PR titles (`feat:`, `fix:`, `docs:`, etc.)

## Testing

Tests use the Node.js built-in test runner with `tsx`. Test files use the `*.test.ts` suffix (e.g., `quartz/util/path.test.ts`, `quartz/util/fileTrie.test.ts`). Run a single test file with `npx tsx --test quartz/util/path.test.ts`.

## Requirements

- Node.js >= 22
- npm >= 10.9.2
