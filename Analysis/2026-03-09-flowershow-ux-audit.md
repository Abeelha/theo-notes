---
title: Flowershow UX & Code Audit
date: 2026-03-09
type: analysis
status: complete
project: flowershow
tags: [ux, bugs, onboarding, blog]
description: Full UX and code audit of the Flowershow blog listing pipeline, landing pages, dashboard onboarding, and docs discoverability. 3 confirmed bugs, 4 UX gaps, 14-item action plan.
---

# Flowershow UX & Code Analysis Report

**Date**: 2026-03-09
**Branch**: `main`
**Scope**: Blog listing pipeline, landing pages, dashboard onboarding, documentation discoverability

---

## 1. Repository Map (Relevant Areas)

| Path | Description | Lines |
|---|---|---|
| `apps/flowershow/components/public/mdx/list.tsx` | `<List>` blog listing component — subject of bugs A, B, C | 272 |
| `apps/flowershow/components/public/mdx/list-pagination.tsx` | Pagination component — subject of bug B | 94 |
| `apps/flowershow/components/public/layouts/blog.tsx` | BlogLayout with correct `formatDate()` — model for the fix | 121 |
| `apps/flowershow/app/(cloud)/dashboard/page.tsx` | Post-login dashboard shell | 31 |
| `apps/flowershow/components/dashboard/site-quickstarts.tsx` | The three source-oriented onboarding tiles | 54 |
| `apps/flowershow/app/(cloud)/dashboard/hello-world/page.tsx` | Hello World tutorial with commented-out YouTube video | 87 |
| `apps/flowershow/app/(cloud)/dashboard/obsidian-quickstart/page.tsx` | Obsidian tutorial — also has commented-out video block | 80 |
| `apps/flowershow/app/(cloud)/dashboard/new/page.tsx` | GitHub site creation form | 386 |
| `apps/flowershow/server/api/routers/site.ts` | `getListComponentItems` API handler; image URL resolution | 1037–1139 |
| `apps/flowershow/server/api/types.ts` | `PageMetadata` interface | 82–115 |
| `apps/flowershow/app/(cloud)/providers.tsx` | `ConfettiProvider` already wired — infra for celebrate moment | 24 |
| `content/flowershow-app/uses/blogs.md` | Blog landing/marketing page | 418 |
| `content/flowershow-app/README.md` | Homepage | — |
| `content/flowershow-app/docs/list-component.md` | List component reference docs | — |
| `content/flowershow-app/blog/how-to-publish-blog.md` | 8-step blog tutorial with YouTube link `youtu.be/ZbQRlNm2dww` | 309 |
| `content/flowershow-app/config.json` | Navigation config | — |

---

## 2. Blog Listing: Current State & Bugs

### Bug A — Date Formatting: Hardcoded YYYY-MM-DD ✅ CONFIRMED

**File**: `apps/flowershow/components/public/mdx/list.tsx`, lines 248–254

The `fmt()` function at line 237 formats detected dates as YYYY-MM-DD unconditionally. The `applyFormat()` function at lines 191–235 has proper locale-aware formatting but the `slotsFormat` prop invoking it is commented out.

**Fix**: Replace lines 248–253:
```ts
if (isDateInput(v)) {
  return new Date(v).toLocaleDateString('en-US', {
    year: 'numeric',
    month: 'long',
    day: 'numeric',
  });
}
```

---

### Bug B — Pagination: Duplicate Ellipsis Spans ✅ CONFIRMED

**File**: `apps/flowershow/components/public/mdx/list-pagination.tsx`, lines 49–76

For every hidden page, a separate `<span>...</span>` is rendered. On a 50-page blog on page 10 = 43 ellipsis spans in the DOM.

**Fix**: Track `prevWasEllipsis` outside the map to emit only one `...` per gap.

---

### Bug C — Image Empty String Bypasses Placeholder ✅ CONFIRMED

**File**: `apps/flowershow/components/public/mdx/list.tsx`, line 145

`??` doesn't catch `""`. Change to `||`.

---

## 3. Landing Pages: Current State & Gaps

### `/uses/blogs`

- "See demo" → dead end (live blog, no setup context)
- "Learn more →" to `/docs/list-component` buried below fold
- No link to `/blog/how-to-publish-blog` tutorial
- Only Flowershow's own blog in "See it in the wild" showcase

---

## 4. Dashboard / Onboarding: Current State & Gaps

Three tiles are **source-oriented** (Hello World / Obsidian / GitHub), not **goal-oriented** (blog / docs / wiki). Users who want a blog never discover the `<List />` component.

`ConfettiProvider` exists in `providers.tsx` but is unused.

Both tutorial pages (`hello-world/page.tsx:6–16`, `obsidian-quickstart/page.tsx:4–14`) have a YouTube iframe commented out with mismatched title ("Obsidian Tutorial" on the Hello World page).

---

## 5. Docs Discoverability

Zero links from `cloud.flowershow.app` to documentation. Path to find list-component docs from dashboard = 3+ steps out of the app.

---

## 6. Action Plan

| ID | Task | File(s) | Effort | Priority |
|---|---|---|---|---|
| P0-A | Fix date formatting | `list.tsx:248–253` | S | **P0** |
| P0-B | Fix image empty string fallback | `list.tsx:145` | S | **P0** |
| P0-C | Fix pagination ellipsis | `list-pagination.tsx:49–76` | S | **P0** |
| P1-A | Add "Start a Blog" tile + blog-quickstart route | `site-quickstarts.tsx` + new route | M | **P1** |
| P1-B | Create blog-specific GitHub starter template | New repo | M | **P1** |
| P1-C | Link blog tutorial from `/uses/blogs` | `uses/blogs.md` | S | **P1** |
| P1-D | Add Docs link to dashboard nav | Dashboard nav | S | **P1** |
| P2-A | Fix "See demo" dead-end | Demo site content | S | P2 |
| P2-B | Fix/uncomment tutorial videos | `hello-world/page.tsx`, `obsidian-quickstart/page.tsx` | S | P2 |
| P2-C | Post-publish next-steps checklist + confetti | Site detail / `sites.tsx` | M | P2 |
| P2-D | Add user blog showcase examples | `uses/blogs.md` | S | P2 |
| P3-A | Enable `slotsFormat` / `applyFormat()` | `list.tsx` | M | P3 |
| P3-B | Goal-oriented onboarding flow | `site-quickstarts.tsx` + new routes | L | P3 |
| P3-C | In-app `syntaxMode: mdx` hint | Rendering pipeline | L | P3 |
