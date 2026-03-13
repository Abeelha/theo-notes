---
title: Flowershow UX Testing — Full Findings
date: 2026-03-13
type: analysis
status: complete
project: flowershow
tags: [ux, testing, onboarding, blog, cli, obsidian, github]
description: Full manual + AI-assisted UX audit across all publishing paths — auth, onboarding, CLI, Obsidian, GitHub, blog setup, customisation, and config. 14 friction points found across 7 sessions.
---

# Flowershow UX Testing — Full Findings

**Date**: 2026-03-13
**Tester**: Theodoro Bertol (manual) + Claude Code (AI analysis, Sessions 5–7)
**Scope**: New user experience across all publishing paths, blog setup, and customisation
**Artefacts**: Videos embedded inline (`first-onboarding.mp4`, `fixing-cli-path-name.mp4`, `github-publish-test-with-small-issue-on-sign-out.mp4`, `copy-name-delete-site.mp4`)

---

## Summary

| Severity | Count |
|---|---|
| 🔴 Blocker | 2 |
| 🟠 Confusing | 6 |
| 🟡 Missing | 4 |
| ⚪ Polish | 3 |

---

## 🔴 Blockers

### B1 — Auth: Same email on GitHub + Google throws `OAuthAccountNotLinked`

**Session**: 1 — Google sign-up
**Reproduction**: Have a GitHub account with email `x@example.com`. Try to log in with Google using the same email.
**Result**: Hard error — `OAuthAccountNotLinked`. No explanation, no recovery path, no prompt to log in with GitHub instead.
**Expected**: "This email is already linked to a GitHub account — log in with GitHub instead" with a redirect button.
**Artefact**: `![[auth-error-same-email-git-google.png]]` · `![[login-issue-github-google.mp4]]`

> The inverse does NOT have this bug: if you first sign up with Google and then try GitHub with the same email, it works. The failure is one-directional (GitHub-first → Google fails).

---

### B2 — Blog setup is invisible to new users

**Session**: 5 — Blog end-to-end
**Reproduction**: Sign up via the `/uses/blogs` marketing page. Go through onboarding. Try to set up a blog listing.
**Result**: The `<List />` component — the only way to create a blog index — is never mentioned in any onboarding screen, modal, or welcome message. Users get a live site but have no blog index. There is no template, no hint, no link to setup docs.
**Root cause (from code)**: The three onboarding paths (CLI / Obsidian / GitHub) are source-oriented. None of them include a post-publish "now set up your blog" step. The `DashboardEmptyState` just says "Create your first site" — no goal framing.
**Expected**: At minimum, a "Set up your blog listing →" link after first publish, pointing to `/docs/list-component` or a blog starter template.

---

## 🟠 Confusing

### C1 — Obsidian plugin: wrong site name = silent infinite loading

**Session**: 4 — Obsidian
**Reproduction**: Create a site called `test`. In the Obsidian plugin settings, enter a different site name (or leave it blank). Hit "Publish".
**Result**: The publish modal shows "Waiting for content upload..." indefinitely. No error, no timeout, no indication that the site name is wrong.
**Expected**: A validation error: "No site found with this name. Check your site name in Flowershow settings."
**Artefact**: Covered in `first-onboarding.mp4` (~14:20)

---

### C2 — CLI: `publish sync` command confused with "sync" as an ongoing operation

**Session**: 3 — CLI
**Observation**: The command shown in the onboarding modal is `publish sync ./my-content --name test`. A new user reads "sync" and thinks it's starting a background daemon or file watcher. It's actually a one-shot publish. The word "sync" implies ongoing synchronisation.
**Expected**: Rename to `publish push` or add inline copy "This publishes your content once. Run it again to update."

---

### C3 — CLI: `--name` must match site name exactly — silent failure on mismatch

**Session**: 3 — CLI
**Reproduction**: Create site called `test`. Run `publish sync ./my-content --name ideas`.
**Result**: Command exits with error. No hint about which sites exist or what the correct name is.
**Expected**: On name mismatch, the CLI should list available site names and suggest the closest match.
**Artefact**: `![[change-name-cli-sync-site.png]]`

---

### C4 — Delete site: copied name includes trailing space

**Session**: 1 — Onboarding
**Reproduction**: In Site Settings → delete site → double-click the site name to copy it → paste into the confirmation input.
**Result**: The pasted value has a trailing space. The input doesn't trim, so it doesn't match and the delete button stays disabled.
**Expected**: Either trim whitespace on input, or don't include a trailing space in the site name display text.
**Artefact**: `![[copy-name-delete-site.png]]` · `![[copy-name-delete-site.mp4]]`

---

### C5 — Session redirect bug on logout: lands on previous account's site

**Session**: 2 — GitHub
**Reproduction**: Be viewing Site A on Account 1 → log out → log in as Account 2.
**Result**: Redirected to the Site A settings URL from Account 1, which returns 404 for Account 2.
**Expected**: After login, redirect to `/dashboard` unconditionally — never back to the previous session's URL.
**URL observed**: `https://cloud.flowershow.app/site/cmmp965mq0001jl04wq4tsfsn/settings`
**Artefact**: `![[github-publish-test-with-small-issue-on-sign-out.mp4]]`

---

### C6 — Custom domain DNS instructions: A record shown for subdomains, CNAME value shows `www`

**Session**: 6 — Customisation (verified in code + Anuar's bug report, Innovation Team Sync 2026-03-13)
**Reproduction**: Add a subdomain (e.g. `blog.mysite.com`) as a custom domain.
**Result 1**: UI defaults to suggesting an A record. For subdomains, a CNAME is the correct approach.
**Result 2**: When switching to CNAME view, the "Name" field shows `www` instead of the subdomain prefix (e.g. `blog`).
**Root cause**: `domain-configuration.tsx:39` — `activeRecordType` defaults to A record when no subdomain is detected. The detection logic (`getSubdomain()`) may not be correctly identifying subdomain cases.
**Expected**: Detect subdomain vs apex domain and default to CNAME for subdomains. Populate the CNAME name field with the actual subdomain prefix.

---

## 🟡 Missing Features

### M1 — No blog starter template

**Session**: 5 — Blog
**Observation**: The onboarding templates are generic. None pre-configure a blog listing page with `<List />`, sample posts with correct frontmatter (`title`, `date`, `description`, `image`), or `syntaxMode: mdx`. A user who wants a blog must discover all of this independently.
**Expected**: A "Blog starter" template option in the create-site flow that creates a repo pre-wired for blogging.

---

### M2 — `<List />` component fails silently in `.md` files (MDX mode not set)

**Session**: 5 — Blog (confirmed in code review)
**Reproduction**: Add `<List dir="/blog" />` to a `.md` file without `syntaxMode: mdx` in frontmatter.
**Result**: The JSX renders as raw text. No error, no warning.
**Expected**: Either auto-detect JSX and switch to MDX rendering, or display an inline hint: "Add `syntaxMode: mdx` to use components."

---

### M3 — No in-app docs link

**Session**: 6 — Customisation
**Observation**: From `cloud.flowershow.app`, there is no link to documentation anywhere in the UI. To find how to set up a blog listing, newsletter form, or custom theme, a user must independently navigate to `flowershow.app/docs` in a new tab.
**Expected**: A "Docs" link in the dashboard navigation bar.

---

### M4 — Newsletter and comments setup not discoverable from the dashboard

**Session**: 6 — Customisation
**Observation**: Newsletter setup requires embedding an external form (Tally, Mailchimp, etc.) by pasting raw HTML — this is never mentioned in the dashboard or settings. Comments require a GitHub repository to be connected. Neither feature is surfaced as a "feature you can enable" anywhere in the UI — only in docs.
**Expected**: A "Features" or "Add to your site" panel in site settings listing: Comments, Newsletter, Analytics, Custom Domain — with one-click access to setup guides.

---

## ⚪ Polish

### P1 — Drag-and-drop target area is not obvious

**Session**: 1 — Onboarding
**Observation**: Dragging a file directly to the "Import files" button area doesn't work. Clicking the button, then dragging into the modal file zone, does work. The affordance for where to drop is unclear.
**Artefact**: Covered in `first-onboarding.mp4` (~2:10)

---

### P2 — CLI install: EBADENGINE warnings are alarming but non-blocking

**Session**: 3 — CLI
**Observation**: `npm install -g @flowershow/publish@latest` on Node v22.18.0 prints multiple `EBADENGINE` warnings. The install succeeds, but new users seeing "Unsupported engine" warnings will think something is broken.
**Root cause**: The CLI requires `node: ^20.20.0 || >=22.22.0`. Node v22.18.0 is below 22.22.0.
**Note**: `publish` command not found after install is a separate Windows PATH issue — not a Flowershow bug. Fix: `npx @flowershow/publish auth login` works immediately; permanent fix is adding npm global bin dir to PATH.

---

### P3 — GitHub publish worked well — no UX issues

**Session**: 2 — GitHub
**Observation**: The full GitHub publishing flow (connect repo → select branch → create site → publish) worked without friction. The experience is the smoothest of the three publishing paths.

---

## Session 7 — AI Coding Test: Blog Setup via Codebase Context

**Method**: Claude Code reading the Flowershow repo directly (`C:\Users\Abeelha\Documents\github\flowershow`) — no live site access, no manual testing.

**Task**: Set up `theo-notes` as a publishable Flowershow site with auto-indexing blog-style views.

### What I Did

**Step 1 — Vault structure**: Created 4 sections (Analysis, Plans, Ideas, Notes), each with a `README.mdx` index file.

**Step 2 — Bases indexing**: Used `file.inFolder("Analysis")` syntax in each index. Set `syntaxMode: mdx` on each index page individually (global MDX mode is a dashboard setting, not in `config.json`).

**Step 3 — config.json**: Set `superstack` theme with dark mode toggle, sidebar restricted to the 4 sections, nav links to all sections.

**Step 4 — Frontmatter schema**: Defined a consistent schema for all notes:
```yaml
title, date, type, status, project, tags, description
```

### What Was Easy (Docs Worked)
- Bases `file.inFolder()` syntax was clear from the beta announcement
- `contentHide` newly documented — works for hiding internal notes
- Sidebar `paths` config — well documented in `sidebar.md`
- `superstack` theme config — example in `config-file.md`

### What Was Hard (Docs Gaps Found)

**Gap 1 — `siteId` prop confusion**: The `ListProps` TypeScript interface has `siteId: string` as a required field. The docs correctly show `<List dir="/blog" />` without it (it's injected by MDX context), but the type mismatch would confuse any developer reading the component source. Should be typed as optional or moved to an internal interface.

**Gap 2 — No "complete blog setup" example**: Getting from "I want a blog" to a working `<List />` configuration required reading: `obsidian-bases.md` (Bases syntax), `list-component.md` (List component), `syntax-mode.md` (MDX requirement), and `sidebar.md` (config). No single doc ties them together. A "Set up a blog in 3 steps" guide doesn't exist.

**Gap 3 — Bases vs `<List />` — which to use?**: For Obsidian users, Bases is more natural. For GitHub/CLI users, `<List />` is documented first. There's no comparison or guidance on when to use which. I defaulted to Bases since this is an Obsidian vault, but a new user would have no basis for this decision.

**Gap 4 — Global `syntaxMode` not in config.json**: Setting `syntaxMode: mdx` globally requires going to the dashboard (Site Settings → Rendering). It's not a `config.json` field. I had to add it to every index page individually. This is not clearly documented — the syntax-mode docs describe per-page and global options but don't show the global option as a `config.json` key (because it isn't one).

### AI Test Verdict
**The AI path (Claude Code + repo as context) works well** — reading the source is faster than reading the docs because the source is authoritative and complete. The docs path for a non-AI user would require significantly more trial-and-error. A new user without AI assistance would likely hit silent failures on at least 3 of the 4 gaps above.

---

## Prioritised Findings Table

| ID | Finding | Severity | Session | Effort to Fix |
|---|---|---|---|---|
| B1 | Auth: OAuthAccountNotLinked for same email on GitHub+Google | 🔴 Blocker | 1 | M |
| B2 | Blog setup is invisible — `<List />` never mentioned in onboarding | 🔴 Blocker | 5 | M |
| C1 | Obsidian: wrong site name = silent infinite loading | 🟠 Confusing | 4 | S |
| C2 | CLI: "sync" implies ongoing — it's one-shot | 🟠 Confusing | 3 | S |
| C3 | CLI: `--name` mismatch gives no hint about correct names | 🟠 Confusing | 3 | S |
| C4 | Delete site: copied name has trailing space | 🟠 Confusing | 1 | S |
| C5 | Logout → login redirects to previous account's site | 🟠 Confusing | 2 | S |
| C6 | DNS instructions: wrong default for subdomains | 🟠 Confusing | 6 | S |
| M1 | No blog starter template | 🟡 Missing | 5 | M |
| M2 | `<List />` fails silently in `.md` without `syntaxMode: mdx` | 🟡 Missing | 5 | S |
| M3 | No docs link anywhere in the cloud app | 🟡 Missing | 6 | S |
| M4 | Newsletter & comments not discoverable from settings | 🟡 Missing | 6 | M |
| P1 | Drag-drop zone affordance unclear | ⚪ Polish | 1 | S |
| P2 | CLI EBADENGINE warnings confuse new users | ⚪ Polish | 3 | S |
