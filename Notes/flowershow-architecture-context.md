---
title: Flowershow Architecture Context
date: 2026-03-09
type: note
status: complete
project: flowershow
tags: [architecture, reference, flowershow]
description: Key architecture facts, file paths, and component details for the Flowershow monorepo. Use as context when working on Flowershow tasks.
---

# Flowershow Architecture Context

**Repo**: `C:\Users\Abeelha\Documents\github\flowershow` · **Branch**: `main`

---

## What is Flowershow?

SaaS platform that turns markdown files into published websites. Users connect GitHub, drag-and-drop files, or use the Obsidian plugin. Live app: `cloud.flowershow.app`. Marketing site: `flowershow.app`.

**Monorepo (Turborepo)**:
- `apps/flowershow` — Next.js cloud app
- `apps/cloudflare-worker` — file processing (frontmatter parsing, sync)
- `apps/cli` — CLI tool
- `apps/flowershow-mcp` — MCP server
- `content/flowershow-app/` — marketing site content (MDX/MD pages)

---

## Key Components

### `<List />` component
- **Source**: `apps/flowershow/components/public/mdx/list.tsx`
- Fetches via `api.site.getListComponentItems` (tRPC)
- `siteId` is injected by MDX context — users don't pass it
- Props: `dir`, `slots`, `pageSize`
- Default slots: `{ headline: 'title', summary: 'description' }`
- Requires `syntaxMode: mdx` in frontmatter

**Slots**:
- `media` → image
- `eyebrow` → small text above title (dates go here)
- `headline` → title
- `summary` → description
- `footnote` → authors

### `applyFormat()` system
- Fully implemented at `list.tsx:191–235`
- Supports: `date`, `currency`, `number`, `percent`, `join`, `prefix`, `suffix`
- `slotsFormat` prop is commented out — not yet shipped to users

### Pagination
- `apps/flowershow/components/public/mdx/list-pagination.tsx`
- Window logic: first + last + ±2 from current

### BlogLayout
- `apps/flowershow/components/public/layouts/blog.tsx`
- Has correct `formatDate()` using `Intl.DateTimeFormatOptions` → "March 9, 2026"

### Dashboard
- `apps/flowershow/app/(cloud)/dashboard/page.tsx` — shell (31 lines)
- `apps/flowershow/components/dashboard/site-quickstarts.tsx` — 3 tiles
- `apps/flowershow/app/(cloud)/providers.tsx` — `ConfettiProvider` (wired but unused)

---

## Image Resolution

API handler at `server/api/routers/site.ts:1106–1118` resolves wiki-links in image paths server-side. Client receives full URL string, `null`, or `""`.

---

## Frontmatter Parsing

`apps/cloudflare-worker/src/processing-utils.js` — uses `gray-matter`. Fields: `title`, `description`, `authors`, `date`, `image`, `publish`, `syntaxMode`, `hero`, etc.

---

## `PageMetadata` interface

`apps/flowershow/server/api/types.ts:82–115`

```ts
interface PageMetadata {
  title: string;
  description?: string;
  image?: string;
  authors?: string[];
  date?: string;
  publish?: boolean;
  syntaxMode?: 'md' | 'mdx';
  [key: string]: any;
}
```
