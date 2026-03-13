# Flowershow — Context Document for Claude Web

**Repo**: `flowershow` monorepo · **Branch**: `main` · **Date**: 2026-03-09

  

---

  

## What is Flowershow?

SaaS platform that turns markdown files into published websites. Users connect GitHub, drag-and-drop files, or use an Obsidian plugin. Live app: `cloud.flowershow.app`. Marketing site: `flowershow.app`. Monorepo with Turborepo: `apps/flowershow` (Next.js app), `apps/cloudflare-worker` (file processing), `apps/cli`, `apps/flowershow-mcp`. Content for marketing site lives in `content/flowershow-app/`.

  

---

  

## Active Research: UX & Bug Analysis

  

We completed a full UX + code audit. Three confirmed bugs, four UX gaps, two new discoveries. Below is everything needed to implement fixes.

  

---

  

## 3 Confirmed Bugs

  

### Bug 1 — Date Formatting: YYYY-MM-DD shown in blog listings

- **File**: `apps/flowershow/components/public/mdx/list.tsx:248–253`

- **Root cause**: `fmt()` function unconditionally formats detected dates as YYYY-MM-DD by manually constructing `${yyyy}-${mm}-${dd}`

- **Why it's wrong**: Correct approach already exists at `apps/flowershow/components/public/layouts/blog.tsx:107–119` — a `formatDate()` function using `Intl.DateTimeFormatOptions` that produces "March 9, 2026"

- **The infrastructure exists but is disabled**: `applyFormat()` at `list.tsx:191–235` already has proper locale-aware date formatting, but the `slotsFormat` prop that would invoke it is commented out at lines 66 and 241–247

- **Fix**: In `list.tsx:248–253`, replace manual date construction with:

  ```ts

  return new Date(v).toLocaleDateString('en-US', { year: 'numeric', month: 'long', day: 'numeric' });

  ```

- **Effort**: S | **Risk**: None

  

### Bug 2 — Pagination: Renders one ellipsis span per hidden page (not per gap)

- **File**: `apps/flowershow/components/public/mdx/list-pagination.tsx:49–76`

- **Root cause**: The `.map()` renders `<span>...</span>` for EVERY hidden page individually. On a 50-page blog on page 10: pages 2–7 (6 spans) + pages 13–49 (37 spans) = 43 separate ellipsis elements

- **Expected**: Standard pagination `1 ... 8 9 10 11 12 ... 50` — one `...` per gap

- **Fix**: Track `prevWasEllipsis` outside the map callback; only emit one span per gap. Or pre-compute a `(number | 'ellipsis')[]` array and render that.

- **Effort**: S | **Risk**: Low

  

### Bug 3 — Image: Empty string `image: ""` bypasses placeholder fallback

- **File**: `apps/flowershow/components/public/mdx/list.tsx:145`

- **Root cause**: Uses `??` (nullish coalescing) which only catches `null`/`undefined`, not empty string `""`. If frontmatter has `image: ""`, Next.js `<Image>` receives `src=""` and fails.

- **Fix**: Change `??` to `||` on that one line

- **Note**: Previously reported as "shows text 'Upload your own image under /settings'" — that text does NOT exist in the codebase. The actual symptom is a broken image.

- **Effort**: S | **Risk**: None

  

---

  

## 4 UX Gaps

  

### Gap 1 — Dashboard onboarding is source-oriented, not goal-oriented

- **File**: `apps/flowershow/components/dashboard/site-quickstarts.tsx` (54 lines)

- **Current state**: Three tiles — "Publish Hello World Site" (`/hello-world`), "Publish your Obsidian vault" (`/obsidian-quickstart`), "Publish your markdown from GitHub" (`/new`). All tell users WHERE to put their content, not WHAT they're building.

- **Problem**: A user who came via the "blog" marketing page signs up and sees no blog-specific path. The `<List />` component that creates a blog index is never mentioned during onboarding.

- **Fix**: Add a "Start a Blog" tile linking to a new `/dashboard/blog-quickstart` page that combines source selection + explicit "add `<List />` to your blog index" step with a working code snippet.

- **Dashboard page**: `apps/flowershow/app/(cloud)/dashboard/page.tsx` (31 lines — just `SiteQuickstarts` + `Sites` + optional `MigrationBanner`)

  

### Gap 2 — `/uses/blogs` landing page shows WHAT but not HOW

- **File**: `content/flowershow-app/uses/blogs.md` (418 lines of MDX/JSX)

- **Current state**: Beautiful marketing page. Hero + demo video (`/assets/uses/blog/blog-demo.mp4`). Feature grid. FAQs. Two CTAs: "Get started for free" and "See demo".

- **Problems**:

  1. "See demo" → `demo-blog.flowershow.app` — a live blog with no "how was this made?" context or link back to setup docs. Dead end.

  2. No link to the 8-step blog tutorial (`/blog/how-to-publish-blog`)

  3. "Learn more →" link to `/docs/list-component` is buried below the fold under a feature card

- **Fix**: Add "Step-by-step setup guide →" link near the hero CTAs; add "How was this made?" banner to demo site

  

### Gap 3 — Zero docs links from inside the cloud app

- **Problem**: From `cloud.flowershow.app`, there are no links to `flowershow.app/docs` anywhere. Documentation lives on the marketing site.

- **Path to docs from dashboard**: Dashboard → open new tab → navigate to flowershow.app → find Docs in nav → find list-component in sidebar. 3+ steps out of the app.

- **Fix**: Add a "Docs" link to the dashboard navigation. The most impactful single-link change possible.

  

### Gap 4 — Blog index setup requires silent MDX knowledge

- **File**: `content/flowershow-app/docs/list-component.md`

- **Problem**: Users paste `<List dir="/blog" />` into a `.md` file → nothing renders, no error. Must know to add `syntaxMode: mdx` to frontmatter (or rename to `.mdx`). The docs warn about this but it never surfaces in-app.

  

---

  

## Key Architecture Facts

  

### `<List />` component

- **Source**: `apps/flowershow/components/public/mdx/list.tsx`

- Fetches items via `api.site.getListComponentItems` (tRPC)

- `siteId` prop is INJECTED by MDX context — users don't pass it manually

- Props: `dir` (folder to list), `slots` (map of display slots to frontmatter fields), `pageSize` (optional)

- Default slots: `{ headline: 'title', summary: 'description' }` — no date or image by default

- Requires `syntaxMode: mdx` in frontmatter (or `.mdx` file extension)

  

### Blog post card layout (slots)

- `media` → image thumbnail

- `eyebrow` → small text above title (used for dates)

- `headline` → post title

- `summary` → description

- `footnote` → small text below (used for authors)

  

### `applyFormat()` system (half-shipped)

- Fully implemented at `list.tsx:191–235`

- Supports: `date`, `currency`, `number`, `percent`, `join`, `prefix`, `suffix`

- The `slotsFormat` prop is commented out — not exposed to users yet

- Bug 1 fix (direct `toLocaleDateString`) should ship first; `applyFormat` is the longer-term configurability story

  

### Image resolution

- API handler at `server/api/routers/site.ts:1106–1118` resolves wiki-links in image paths server-side before returning to client

- Client receives a full URL string or `null` / `""`

- Bug 3 fires when server returns `""` (from `image: ""` in frontmatter)

  

### ConfettiProvider

- Already wired in `apps/flowershow/app/(cloud)/providers.tsx`

- Unused — infrastructure for a "site is live!" celebration moment is sitting idle

  

### Tutorial videos (commented out)

- `apps/flowershow/app/(cloud)/dashboard/hello-world/page.tsx:6–16` — YouTube iframe commented out, URL: `youtube.com/embed/_2cwU6zwNWQ`, title "Flowershow Obsidian Tutorial" (mismatched — this is the Hello World page)

- `apps/flowershow/app/(cloud)/dashboard/obsidian-quickstart/page.tsx:4–14` — identical commented-out block, same URL

- Correct Obsidian video (`2jOYg0wCg1s`) is already embedded on `content/flowershow-app/uses/obsidian.md`

- Blog tutorial video: `youtu.be/ZbQRlNm2dww` (not embedded anywhere in-app, plain link only)

  

---

  

## Prioritized Action Plan

  

| ID | Task | File(s) | Effort | Priority |

|---|---|---|---|---|

| P0-A | Fix date formatting in `fmt()` | `list.tsx:248–253` | S | **P0** |

| P0-B | Fix image empty string fallback (`??` → `\|\|`) | `list.tsx:145` | S | **P0** |

| P0-C | Fix pagination duplicate ellipsis spans | `list-pagination.tsx:49–76` | S | **P0** |

| P1-A | Add "Start a Blog" tile + blog-quickstart route | `site-quickstarts.tsx` + new route | M | **P1** |

| P1-B | Create blog-specific GitHub starter template | New repo `flowershow/flowershow-blog-template` | M | **P1** |

| P1-C | Link blog tutorial from `/uses/blogs` hero | `uses/blogs.md` | S | **P1** |

| P1-D | Add Docs link to dashboard nav | Dashboard nav component | S | **P1** |

| P2-A | Add "How was this made?" banner to demo site | `demo-blog.flowershow.app` content | S | P2 |

| P2-B | Fix/uncomment tutorial videos | `hello-world/page.tsx:6–16`, `obsidian-quickstart/page.tsx:4–14` | S | P2 |

| P2-C | Post-publish next-steps checklist + confetti | Site detail page | M | P2 |

| P2-D | Add user blog examples to `/uses/blogs` showcase | `uses/blogs.md` | S | P2 |

| P3-A | Enable `slotsFormat` / `applyFormat()` | `list.tsx` | M | P3 |

| P3-B | Goal-oriented "What are you building?" onboarding | `site-quickstarts.tsx` + new routes | L | P3 |

| P3-C | In-app `syntaxMode: mdx` detection + hint | Rendering pipeline | L | P3 |

  

---

  

## Files Ready for P0 Fixes

  

All three P0 bugs are in two files. Implement them together:

  

```

apps/flowershow/components/public/mdx/list.tsx

  line 145: ?? → ||

  lines 248–253: manual YYYY-MM-DD → toLocaleDateString

  

apps/flowershow/components/public/mdx/list-pagination.tsx

  lines 49–76: emit one ellipsis per gap, not per hidden page

```

  

---

  

## Verification Steps for P0

  

- **Date**: Publish test site with blog posts having `date` frontmatter + List with `slots={{ eyebrow: 'date' }}`. Confirm "March 9, 2026" not "2026-03-09".

- **Image**: Create post with `image: ""`. Confirm placeholder image appears, not broken.

- **Pagination**: Blog with 15+ posts, `pageSize={3}`, navigate to page 5. Confirm exactly two `...` spans appear, not 37.