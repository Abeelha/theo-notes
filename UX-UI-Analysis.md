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

The `fmt()` function at line 237 is the display formatter for every slot value in the list component. When it detects a date-like value (via `isDateInput()`), it formats it as YYYY-MM-DD unconditionally:

```ts
if (isDateInput(v)) {
  const d = new Date(v);
  const yyyy = d.getFullYear();
  const mm = String(d.getMonth() + 1).padStart(2, '0');
  const dd = String(d.getDate()).padStart(2, '0');
  return `${yyyy}-${mm}-${dd}`;  // always "2025-06-24", never "June 24, 2025"
}
```

**Root cause — the format system exists but is disabled**: The `applyFormat()` function at lines 191–235 is fully implemented and already handles `'date'` with `d.toLocaleDateString(locale)`. The `slotsFormat` parameter that would route through it is commented out at lines 241–247. This is a half-shipped feature.

**The correct pattern already exists**: `apps/flowershow/components/public/layouts/blog.tsx` lines 107–119 has `formatDate()` producing "June 24, 2025" using `Intl.DateTimeFormatOptions`.

**User impact**: Every Flowershow blog using `eyebrow: "date"` in slots displays raw ISO strings. Immediately visible to all users. Highest-impact single-line fix in the codebase.

**Proposed fix** — replace lines 248–253 in `list.tsx`:
```ts
// BEFORE (manual YYYY-MM-DD construction)
if (isDateInput(v)) {
  const d = new Date(v);
  const yyyy = d.getFullYear();
  const mm = String(d.getMonth() + 1).padStart(2, '0');
  const dd = String(d.getDate()).padStart(2, '0');
  return `${yyyy}-${mm}-${dd}`;
}

// AFTER
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

The pagination component iterates over ALL page numbers and renders a separate `<span>...</span>` for each hidden page:

```ts
Array.from({ length: totalPages }, (_, i) => i + 1).map((page) => {
  if (page === 1 || page === totalPages || (page >= currentPage - 2 && page <= currentPage + 2)) {
    return <button key={page}>...</button>;
  } else {
    return <span key={page} className="...">...</span>;  // ONE span per hidden page
  }
})
```

**Concrete example**: A 50-page blog on page 10:
- Visible: 1, 8, 9, 10, 11, 12, 50
- Hidden: pages 2–7 (6 spans) + pages 13–49 (37 spans) = **43 ellipsis spans** in the DOM

**Proposed fix**: Track `prevWasEllipsis` to only emit one `...` per gap:

```ts
let prevWasEllipsis = false;
Array.from({ length: totalPages }, (_, i) => i + 1).map((page) => {
  const isVisible =
    page === 1 ||
    page === totalPages ||
    (page >= currentPage - 2 && page <= currentPage + 2);

  if (isVisible) {
    prevWasEllipsis = false;
    return <button key={page} ...>{page}</button>;
  }
  if (!prevWasEllipsis) {
    prevWasEllipsis = true;
    return <span key={`ell-before-${page}`} className="list-component-pagination-ellipsis">...</span>;
  }
  return null;
}).filter(Boolean)
```

---

### Bug C — Image Empty String Bypasses Placeholder ✅ CONFIRMED

**File**: `apps/flowershow/components/public/mdx/list.tsx`, line 145

```ts
src={
  (getValue('media', metadata, slotsMap) as string) ??
  'https://r2-assets.flowershow.app/placeholder.png'
}
```

**Root cause**: `??` only falls through on `null` or `undefined`. An empty string `image: ""` in frontmatter (from template stubs or partial drafts) bypasses `??` and passes `""` to Next.js `<Image>`, which fails.

**Note on reported "Upload your own image under /settings" text**: NOT FOUND in any source file. This bug report is either outdated or refers to text in a different context.

**Proposed fix** — one token change:
```ts
// BEFORE
src={(getValue('media', metadata, slotsMap) as string) ?? 'https://r2-assets.flowershow.app/placeholder.png'}

// AFTER
src={(getValue('media', metadata, slotsMap) as string) || 'https://r2-assets.flowershow.app/placeholder.png'}
```

---

### Issue D — Blog Index Requires MDX Knowledge (UX Gap)

**File**: `content/flowershow-app/docs/list-component.md`, lines 6–9

The docs warn:
> Using the `List` component requires MDX rendering, since it uses JSX. Make sure to add `syntaxMode: mdx` in the frontmatter.

**Silent failure mode**:
1. User creates `blog/README.md`
2. Pastes `<List dir="/blog" />` into it
3. Publishes the site
4. The component renders as raw text — no error, just silence
5. User has no idea what went wrong

The 8-step tutorial (`/blog/how-to-publish-blog.md`) does correctly include `syntaxMode: mdx` in its code sample. The gap is that this never surfaces in-app at the failure point.

---

## 3. Landing Pages: Current State & Gaps

### `/uses/blogs` (`content/flowershow-app/uses/blogs.md`)

**What it promises**: "Your personal blog. Live in seconds." Hero + autoplay demo video, 990+ sites stat.

**CTAs**:
- "Get started for free →" → `https://cloud.flowershow.app/login`
- "See demo" → `https://demo-blog.flowershow.app/`
- 4 publishing method cards (GitHub, drag-drop, Obsidian, CLI) with "Get started →"
- "Learn more →" links to individual feature docs (buried below fold)

**What it delivers**: Shows WHAT a finished blog looks like — demo video, screenshots, feature grid. Does NOT show HOW to set up the blog listing page.

**Critical gaps**:

1. **"See demo" leads nowhere actionable.** `demo-blog.flowershow.app` shows a polished live blog with no "how was this made?" context, no link back to setup instructions. Users arrive in a dead end.

2. **The HOW is missing above the fold.** The "Learn more →" link to `/docs/list-component` is buried below the fold inside the "Beautiful blog index" feature card.

3. **No link to the blog setup tutorial** (`/blog/how-to-publish-blog`). The tutorial is the most complete resource but is not linked from this page.

4. **Publishing method cards are source-oriented, not goal-oriented.** After "Get started for free" → dashboard → the same four source tiles appear with no blog-specific guidance. The loop is: marketing page → sign up → dashboard → same paths, no blog setup.

5. **"In the wild" showcase contains only Flowershow's own blog.** No user-submitted examples weaken social proof.

---

### Homepage (`content/flowershow-app/README.md`)

A "Blogs" tile in the use cases section correctly funnels to `/uses/blogs`. The funnel structure is sound.

**Gap**: No "I want to build a blog" → step-by-step path above the fold.

---

## 4. Dashboard / Onboarding: Current State & Gaps

### Dashboard Structure (`apps/flowershow/app/(cloud)/dashboard/page.tsx`, 31 lines)

1. `MigrationBanner` — conditional for OAuth-only users
2. `SiteQuickstarts` — the three tiles
3. `Sites` — user's existing sites (Suspense-wrapped)

### The Three Tiles (`site-quickstarts.tsx`, 54 lines)

| Tile | Title | Destination |
|---|---|---|
| 1 | Publish Hello World Site | `/hello-world` |
| 2 | Publish your Obsidian vault | `/obsidian-quickstart` |
| 3 | Publish your markdown from GitHub | `/new` |

**All three are source-oriented** (where your content lives), not goal-oriented (what you want to build).

**Core UX problem**: A user who arrived via the `/uses/blogs` marketing page — told "Your personal blog. Live in seconds" — sees three tiles about content sources. There is no translation layer between "I want a blog" and "here is how to configure one."

After the user picks a tile and publishes a site, they have a live Flowershow site with their markdown rendered. **They do not have a blog index page.** The `<List />` component is never mentioned, required, or set up during onboarding.

**What's missing**:
- No "What are you building?" step before tile selection
- No goal → setup mapping: "I want a blog → here's what to do next"
- No checklist or progress tracking after site creation
- No link to documentation from anywhere in the dashboard
- `ConfettiProvider` exists in `providers.tsx` but is unused — the infrastructure for a celebration moment is sitting idle

### Tutorial Page: Hello World (`hello-world/page.tsx`)

Lines 6–16 contain a commented-out YouTube iframe:
```tsx
{/* <iframe src="https://www.youtube.com/embed/_2cwU6zwNWQ" title="Flowershow Obsidian Tutorial" ... /> */}
```

The title says "Flowershow Obsidian Tutorial" but the page is "Publish a Demo Site" — title mismatch. The same identical commented-out block also appears in `obsidian-quickstart/page.tsx` lines 4–14. Both pages were scaffolded from the same template and neither video was ever verified or filled in.

---

## 5. Documentation: Discoverability

### What Exists

40+ pages, well-organized, all features covered:
- `/docs/list-component` — component reference, good structure, requires MDX knowledge
- `/blog/how-to-publish-blog` — 8-step tutorial with YouTube link, accurate to current behavior

### Discoverability Failures

**From the cloud app dashboard, there are zero links to documentation.**

Path to find list-component docs from inside the cloud app:
1. Dashboard (no docs link anywhere)
2. User opens new tab, navigates to `flowershow.app`
3. Finds "Docs" in the nav
4. Finds "List component" in the sidebar

3+ navigation steps, requires leaving the app.

**The tutorial `/blog/how-to-publish-blog` is not linked from**:
- The `/uses/blogs` marketing page
- The dashboard
- The hello-world tutorial
- The obsidian-quickstart tutorial
- The `/new` site creation form

It is only accessible via the blog section on flowershow.app.

---

## 6. Video / Demo: Current Implementation

### Content Site Videos

| Page | Implementation | Video |
|---|---|---|
| `uses/obsidian.md` | YouTube iframe embed | `youtube.com/embed/2jOYg0wCg1s` |
| `publish.md` | YouTube iframe embed | `youtube.com/embed/ou1bigOIlPk` |
| `uses/blogs.md` | Local MP4 autoplay (muted) | `/assets/uses/blog/blog-demo.mp4` |
| `blog/how-to-publish-blog.md` | Plain link (not embedded) | `youtu.be/ZbQRlNm2dww` |

### In-App Tutorial Videos

Both tutorial pages have the same commented-out YouTube iframe:
- `hello-world/page.tsx` lines 6–16: `youtube.com/embed/_2cwU6zwNWQ`, title "Flowershow Obsidian Tutorial" (mismatched with page content)
- `obsidian-quickstart/page.tsx` lines 4–14: identical block, same URL

The blog tutorial video (`youtu.be/ZbQRlNm2dww`) is not embedded in-app and is only a plain link in the tutorial article on the marketing site.

---

## 7. Additional Findings

### Finding 1: `applyFormat()` is implemented but disabled

`list.tsx` lines 191–235 contains a full format specification system supporting `date`, `currency`, `number`, `percent`, `join`, `prefix`, `suffix`. The `slotsFormat` prop is defined in `SlotsFormatMap` but commented out at lines 66, 241–247. This is a half-shipped feature.

When shipped, users could write `slotsFormat={{ eyebrow: "date" }}` for locale-aware formatting and `slotsFormat={{ eyebrow: "date:YYYY-MM-DD" }}` to explicitly request ISO format. The P0-A fix (use `toLocaleDateString` directly) should ship first; this is the longer-term configurability story.

### Finding 2: `siteId` is injected by MDX context

`<List>` has `siteId: string` as a required TypeScript prop but it is injected by the MDX provider — users never need to pass it manually. The docs correctly show `<List dir="/blog" />` without `siteId`. However, the TypeScript type can confuse developers reading the source.

### Finding 3: No blog-specific GitHub starter template

The hello-world tutorial links to `github.com/new?template_owner=flowershow&template_name=flowershow-cloud-template` — a generic starter. There is no "blog starter" template that pre-configures:
- `blog/README.md` with `syntaxMode: mdx` and `<List />` already configured
- Sample blog posts with complete frontmatter
- `config.json` with a blog nav link

Such a template would reduce blog setup to zero post-publish steps for GitHub users.

### Finding 4: Image resolution handles wiki-links server-side

`server/api/routers/site.ts` lines 1106–1118 resolves wiki-links in image paths to full URLs before returning data to the client. The `??` vs `||` bug (Bug C) only manifests when the server returns `""` or `null` — not for properly formatted wiki-link images.

---

## 8. Action Plan

### Priority Framework

- **P0**: Confirmed bugs, high visible impact, single-file changes, low risk — fix immediately
- **P1**: UX gaps that directly block the primary user journey (blog setup) — this sprint
- **P2**: Discoverability improvements that reduce support load and improve activation — next sprint
- **P3**: Feature completions and infrastructure — roadmap

---

### P0 — Bug Fixes (high impact, low effort)

#### P0-A: Fix date formatting
- **Problem**: Blog listing dates display as `2025-06-24` instead of `June 24, 2025`
- **File**: `apps/flowershow/components/public/mdx/list.tsx`, lines 248–253
- **Fix**: Replace manual YYYY-MM-DD construction with `new Date(v).toLocaleDateString('en-US', { year: 'numeric', month: 'long', day: 'numeric' })`
- **Reuse**: `apps/flowershow/components/public/layouts/blog.tsx` lines 107–119 has the identical pattern
- **Effort**: S | **Risk**: None

#### P0-B: Fix empty string image fallback
- **Problem**: `image: ""` in frontmatter bypasses the `??` null-coalescing operator, causing Next.js `<Image>` to receive `src=""` and fail
- **File**: `apps/flowershow/components/public/mdx/list.tsx`, line 145
- **Fix**: Change `??` to `||` on the `src` prop
- **Effort**: S | **Risk**: None

#### P0-C: Fix pagination duplicate ellipsis spans
- **Problem**: Each hidden page number renders its own `<span>...</span>` — 43 spans for a 50-page blog
- **File**: `apps/flowershow/components/public/mdx/list-pagination.tsx`, lines 49–76
- **Fix**: Track `prevWasEllipsis` in the map callback; emit only one `...` per gap. Alternatively, pre-compute a `(number | 'ellipsis')[]` array before rendering.
- **Effort**: S | **Risk**: Low (pure visual change)

---

### P1 — Blog Onboarding Gaps (this sprint)

#### P1-A: Add "Start a Blog" path to the dashboard
- **Problem**: The dashboard's three tiles are source-oriented. Users wanting to build a blog have no guided path and will never discover the `<List />` component.
- **File**: `apps/flowershow/components/dashboard/site-quickstarts.tsx` + new route `/dashboard/blog-quickstart`
- **Approach**: Add a fourth tile "Start a blog" that leads to a new page combining: (1) choose your source, (2) explicit "After publishing, add a `<List />` to your blog directory's README" step with a working code snippet pre-configured for copy-paste
- **Effort**: M | **Priority**: P1

#### P1-B: Create a blog-specific GitHub starter template
- **Problem**: Generic `flowershow-cloud-template` requires manual `<List />` setup after publishing
- **Approach**: New GitHub repo `flowershow/flowershow-blog-template` with `blog/README.md` pre-configured with `syntaxMode: mdx`, `<List />`, sample posts with complete frontmatter, and `config.json` with blog nav
- **Link from**: Blog quickstart tile (P1-A), `/uses/blogs`, hello-world tutorial
- **Effort**: M | **Priority**: P1

#### P1-C: Link blog tutorial from `/uses/blogs`
- **Problem**: The complete 8-step tutorial (`/blog/how-to-publish-blog`) exists but is not linked from the marketing page
- **File**: `content/flowershow-app/uses/blogs.md`
- **Fix**: Add a "Step-by-step setup guide →" link near the hero CTAs or immediately after the publishing method cards
- **Effort**: S | **Priority**: P1

#### P1-D: Add Docs link to the cloud dashboard navigation
- **Problem**: Zero links from the cloud app to documentation
- **File**: Dashboard nav component (search for the layout nav in `apps/flowershow/components/dashboard/`)
- **Fix**: Add a "Docs" link to the dashboard navigation bar pointing to `https://flowershow.app/docs`
- **Effort**: S | **Priority**: P1

---

### P2 — Discoverability and Demo (next sprint)

#### P2-A: Fix the "See demo" dead-end
- **Problem**: `demo-blog.flowershow.app` has no "how was this made?" banner or link back to setup
- **Fix**: Add a floating banner to the demo site: "Built with Flowershow — set up your own in minutes →" linking to `/blog/how-to-publish-blog` and `https://cloud.flowershow.app/login`
- **File**: Demo site content (a Flowershow site — edit is a markdown/config change)
- **Effort**: S | **Priority**: P2

#### P2-B: Resolve and uncomment tutorial videos
- **Problem**: `hello-world/page.tsx` and `obsidian-quickstart/page.tsx` both have identical commented-out iframes with a mismatched video URL (`_2cwU6zwNWQ` titled "Obsidian Tutorial" on the Hello World page)
- **Files**: `apps/flowershow/app/(cloud)/dashboard/hello-world/page.tsx` lines 6–16; `apps/flowershow/app/(cloud)/dashboard/obsidian-quickstart/page.tsx` lines 4–14
- **Fix**: Verify `_2cwU6zwNWQ` exists and what it shows. For Hello World: consider the blog tutorial video `ZbQRlNm2dww` instead. For Obsidian: use `2jOYg0wCg1s` (already embedded on `uses/obsidian.md`). Uncomment once correct URLs are confirmed.
- **Effort**: S | **Priority**: P2

#### P2-C: Add post-publish next-steps checklist
- **Problem**: After a site syncs for the first time, there is no next-step guidance. `ConfettiProvider` exists but is unused.
- **File**: Site detail page or `apps/flowershow/components/dashboard/sites.tsx`
- **Fix**: On first sync, show "Your site is live!" banner + 3-item checklist: (1) Visit your site, (2) Add a blog listing page (link to tutorial), (3) Configure settings
- **Effort**: M | **Priority**: P2

#### P2-D: Add user blog examples to `/uses/blogs` showcase
- **Problem**: "See it in the wild" section only shows Flowershow's own blog. Weak social proof.
- **File**: `content/flowershow-app/uses/blogs.md`
- **Fix**: Add 2–4 user-submitted examples (requires community outreach to identify them)
- **Effort**: S | **Priority**: P2

---

### P3 — Roadmap

#### P3-A: Enable `slotsFormat` and ship `applyFormat()` system
- **File**: `apps/flowershow/components/public/mdx/list.tsx`
- The full format spec system is implemented at lines 191–235 but disabled. Uncomment the `slotsFormat` prop path to allow `slotsFormat={{ eyebrow: "date" }}` for user-configurable formatting.
- **Effort**: M | **Priority**: P3

#### P3-B: Goal-oriented onboarding flow
- **Files**: `apps/flowershow/components/dashboard/site-quickstarts.tsx` + new routes
- Add a "What are you building?" step with goal tiles (Blog / Docs / Wiki / Portfolio). Each goal maps to: recommended source, optional blog-specific template, post-setup checklist.
- **Effort**: L | **Priority**: P3

#### P3-C: In-app `syntaxMode: mdx` detection hint
- **File**: Rendering pipeline (`apps/cloudflare-worker/src/processing-utils.js` or client renderer)
- Detect when a file contains JSX-like syntax (`<ComponentName`) but is rendering in MD mode. Inject hint: "Looks like you're using a component. Add `syntaxMode: mdx` to this page's frontmatter."
- **Effort**: L | **Priority**: P3

---

## Summary Table

| ID | Title | File(s) | Effort | Priority |
|---|---|---|---|---|
| P0-A | Fix date formatting (YYYY-MM-DD → human-readable) | `list.tsx` L248–253 | S | **P0** |
| P0-B | Fix empty string image fallback (`??` → `\|\|`) | `list.tsx` L145 | S | **P0** |
| P0-C | Fix duplicate pagination ellipsis spans | `list-pagination.tsx` L49–76 | S | **P0** |
| P1-A | Add "Start a Blog" tile / blog quickstart path | `site-quickstarts.tsx` + new route | M | **P1** |
| P1-B | Create blog-specific GitHub starter template | New GitHub repo | M | **P1** |
| P1-C | Link blog tutorial from `/uses/blogs` | `uses/blogs.md` | S | **P1** |
| P1-D | Add Docs link to cloud dashboard nav | Dashboard nav component | S | **P1** |
| P2-A | Fix "See demo" dead-end (banner on demo site) | Demo site content | S | P2 |
| P2-B | Resolve and uncomment tutorial videos | `hello-world/page.tsx`, `obsidian-quickstart/page.tsx` | S | P2 |
| P2-C | Post-publish next-steps checklist + confetti | Site detail page / `sites.tsx` | M | P2 |
| P2-D | Add user blog examples to `/uses/blogs` showcase | `uses/blogs.md` | S | P2 |
| P3-A | Enable `slotsFormat` / `applyFormat()` system | `list.tsx` | M | P3 |
| P3-B | Goal-oriented onboarding flow | `site-quickstarts.tsx` + new routes | L | P3 |
| P3-C | In-app `syntaxMode: mdx` detection hint | Rendering pipeline | L | P3 |

---

## Verification

For each P0 fix, verify by:
- **P0-A**: Publish a test site with blog posts that have `date` in frontmatter and `eyebrow: "date"` in List slots. Confirm dates display as "March 9, 2026" not "2026-03-09".
- **P0-B**: Create a post with `image: ""` in frontmatter. Confirm the list item shows the placeholder image, not a broken image or error.
- **P0-C**: Create a blog with 15+ posts and `pageSize={3}`. Navigate to page 5. Confirm pagination shows `1 ... 3 4 5 6 7 ... 15` with exactly two ellipsis spans, not 43.

For P1/P2 content changes: verify links resolve correctly, video iframes play, and the tutorial CTA flows from `/uses/blogs` → tutorial → dashboard → site creation.
