---
title: Flowershow UX Bugs — Implementation Plan
date: 2026-03-09
type: plan
status: approved
project: flowershow
tags: [bugs, blog, ux, list-component, pagination]
description: P0 bug fixes for the blog listing component — date formatting, image fallback, pagination ellipsis. Plus P1–P2 onboarding and discoverability improvements.
---

# Flowershow UX Bugs — Implementation Plan

**Project**: `C:\Users\Abeelha\Documents\github\flowershow`
**Branch**: `main`
**Source analysis**: [[2026-03-09-flowershow-ux-audit]]

---

## P0 — Fix Now (all in 2 files, ~1hr total)

### P0-A: Fix date formatting
- **File**: `apps/flowershow/components/public/mdx/list.tsx:248–253`
- **Change**: Replace manual `${yyyy}-${mm}-${dd}` construction in `fmt()` with:
  ```ts
  return new Date(v).toLocaleDateString('en-US', {
    year: 'numeric',
    month: 'long',
    day: 'numeric',
  });
  ```
- **Verify**: Blog listing dates show "March 9, 2026" not "2026-03-09"

### P0-B: Fix image empty string fallback
- **File**: `apps/flowershow/components/public/mdx/list.tsx:145`
- **Change**: `??` → `||` on the `src` prop
- **Verify**: Post with `image: ""` shows placeholder, not broken image

### P0-C: Fix pagination duplicate ellipsis spans
- **File**: `apps/flowershow/components/public/mdx/list-pagination.tsx:49–76`
- **Change**: Track `prevWasEllipsis` flag outside `.map()`, emit only one `<span>...</span>` per gap
- **Verify**: 50-page blog on page 10 shows exactly `1 ... 8 9 10 11 12 ... 50` (2 ellipsis spans, not 43)

---

## P1 — This Sprint

### P1-A: Add "Start a Blog" tile to dashboard
- **File**: `apps/flowershow/components/dashboard/site-quickstarts.tsx`
- Add 4th tile "Start a blog" → new route `/dashboard/blog-quickstart`
- New page: source picker + "add `<List />` to your blog directory README" step with copy-paste snippet

### P1-B: Create blog starter GitHub template
- New repo `flowershow/flowershow-blog-template`
- Pre-configured: `blog/README.md` with `syntaxMode: mdx` + `<List />`, sample posts, `config.json` with blog nav

### P1-C: Link blog tutorial from `/uses/blogs`
- **File**: `content/flowershow-app/uses/blogs.md`
- Add "Step-by-step setup guide →" link to `/blog/how-to-publish-blog` near the hero CTAs

### P1-D: Add Docs link to dashboard nav
- Add "Docs" link pointing to `https://flowershow.app/docs` in the cloud app navigation

---

## P2 — Next Sprint

| Task | What | Where |
|---|---|---|
| P2-A | "How was this made?" banner on demo site | `demo-blog.flowershow.app` content |
| P2-B | Fix/uncomment tutorial videos | `hello-world/page.tsx:6–16`, `obsidian-quickstart/page.tsx:4–14` |
| P2-C | Post-publish checklist + confetti | Site detail page |
| P2-D | Add user blog examples to showcase | `uses/blogs.md` |

---

## P3 — Roadmap

| Task | What |
|---|---|
| P3-A | Enable `slotsFormat` / `applyFormat()` system in `list.tsx` |
| P3-B | Goal-oriented "What are you building?" onboarding |
| P3-C | In-app `syntaxMode: mdx` detection hint |
