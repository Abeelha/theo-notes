---
title: Flowershow UX Testing — Full Findings
date: 2026-03-13
type: analysis
status: complete
project: flowershow
tags: [ux, testing, onboarding, blog, cli, obsidian, github]
description: Manual UX testing across all publishing paths — auth, onboarding, CLI, Obsidian, and GitHub. 7 friction points found across sessions 1–4, plus AI coding test in session 7.
---

# Flowershow UX Testing — Full Findings

**Date**: 2026-03-13
**Tester**: Theodoro Bertol (manual) + Claude Code (AI analysis, Sessions 5–7)
**Scope**: New user experience across all publishing paths, blog setup, and customisation

---

## Summary

| Severity | Count |
|---|---|
| 🔴 Blocker | 2 |
| 🟠 Confusing | 4 |
| ⚪ Polish | 2 |

---

## 🔴 Blockers

### B1 — Auth: Same email on GitHub + Google throws `OAuthAccountNotLinked`

**Session**: 1 — Google sign-up

- Have a GitHub account with email `x@example.com`
- Try to log in with Google using the same email
- Hard error — `OAuthAccountNotLinked` with no explanation, no recovery path, no prompt to use GitHub instead
- The inverse does NOT have this bug: Google-first → GitHub works fine. Failure is one-directional (GitHub-first → Google fails)
- **Expected**: "This email is already linked to a GitHub account — log in with GitHub instead" with a redirect button

![[auth-error-same-email-git-google.png]]

![[login-issue-github-google.mp4]]

---

---

### B2 — Security: Obsidian plugin stores PAT token in plain text, committed to git

**Session**: 4 — Obsidian

- The Flowershow Obsidian plugin saves the PAT token to `.obsidian/plugins/flowershow/data.json`
- This file is tracked by git by default — anyone who publishes their vault to a public GitHub repo exposes their token
- The token is visible in plain text in the diff and in the full git history
- There is no warning in the plugin UI, the onboarding docs, or the setup flow that this file should be excluded from version control
- **Expected**: Either store the token outside the vault directory, or ship a `.gitignore` with `data.json` excluded, or at minimum show a warning: "Do not commit `.obsidian/plugins/flowershow/data.json` — it contains your secret token"

![[pat-token-public-security.png]]

---

## 🟠 Confusing

### C1 — Obsidian plugin: wrong site name = silent infinite loading

**Session**: 4 — Obsidian

- Create a site called `test`. In the Obsidian plugin settings, enter a different site name (or leave it blank). Hit "Publish"
- The publish modal shows "Waiting for content upload..." indefinitely — no error, no timeout, no indication the site name is wrong
- **Expected**: Validation error — "No site found with this name. Check your site name in Flowershow settings"

![[first-onboarding.mp4]]

*(issue visible at ~14:20)*

---

### C2 — CLI: `publish sync` confused with an ongoing operation

**Session**: 3 — CLI

- The command shown in the onboarding modal is `publish sync ./my-content --name test`
- New users read "sync" and think it starts a background daemon or file watcher — it's actually a one-shot publish
- **Expected**: Rename to `publish push`, or add copy: "This publishes your content once. Run it again to update"

---

### C3 — CLI: `--name` mismatch gives no hint about correct names

**Session**: 3 — CLI

- Create site called `test`. Run `publish sync ./my-content --name ideas`
- Command exits with an error but gives no hint about which sites exist or what the correct name is
- **Expected**: On name mismatch, list available site names and suggest the closest match

![[change-name-cli-sync-site.png]]

---

### C4 — Delete site: copied name includes trailing space

**Session**: 1 — Onboarding

- Site Settings → delete site → double-click the site name to copy it → paste into the confirmation input
- The pasted value has a trailing space — the input doesn't trim, so it doesn't match and the delete button stays disabled
- **Expected**: Trim whitespace on input, or don't include a trailing space in the site name display text

![[copy-name-delete-site.png]]

![[copy-name-delete-site.mp4]]

---

### C5 — Session redirect bug on logout: lands on previous account's site

**Session**: 2 — GitHub

- Be viewing Site A on Account 1 → log out → log in as Account 2
- Redirected to the Site A settings URL from Account 1, which returns 404 for Account 2
- URL observed: `https://cloud.flowershow.app/site/cmmp965mq0001jl04wq4tsfsn/settings`
- **Expected**: After login, redirect to `/dashboard` unconditionally — never back to the previous session's URL

![[github-publish-test-with-small-issue-on-sign-out.mp4]]

---

---

## ⚪ Polish

### P1 — Drag-and-drop target area is not obvious

**Session**: 1 — Onboarding

- Dragging a file directly to the "Import files" button area doesn't work
- Clicking the button, then dragging into the modal file zone, does work
- The affordance for where to drop is unclear

![[first-onboarding.mp4]]

*(issue visible at ~2:10)*

---

### P2 — CLI install: EBADENGINE warnings are alarming but non-blocking

**Session**: 3 — CLI

- `npm install -g @flowershow/publish@latest` on Node v22.18.0 prints multiple `EBADENGINE` warnings
- Install succeeds, but new users seeing "Unsupported engine" will think something is broken
- Root cause: CLI requires `node: ^20.20.0 || >=22.22.0` — Node v22.18.0 is below 22.22.0
- Note: `publish` command not found after install is a separate Windows PATH issue, not a Flowershow bug. Workaround: `npx @flowershow/publish auth login`

---

## Session 7 — AI Coding Test: Blog Setup via Codebase Context

**Method**: Claude Code reading the Flowershow repo directly — no live site access, no manual testing.

**Task**: Set up `theo-notes` as a publishable Flowershow site with auto-indexing blog-style views.

### What Was Done

- Created 4 sections (Analysis, Plans, Ideas, Notes), each with a `README.mdx` index file
- Used `file.inFolder("Analysis")` Bases syntax for auto-indexing in each section
- Set `syntaxMode: mdx` on each index page individually (global MDX mode is a dashboard setting, not a `config.json` field)
- Configured `superstack` theme with dark mode toggle, sidebar restricted to the 4 sections, nav links to all sections
- Defined a consistent frontmatter schema: `title`, `date`, `type`, `status`, `project`, `tags`, `description`

### What Was Easy (Docs Worked) - for AI coding {claude-code}

- Bases `file.inFolder()` syntax — clear from the beta announcement
- `contentHide` — newly documented, works for hiding internal notes
- Sidebar `paths` config — well documented in `sidebar.md`
- `superstack` theme config — example in `config-file.md`

### Docs Gaps Found

- **Gap 1 — `siteId` prop confusion**: `ListProps` has `siteId: string` as required, but it's injected by MDX context — users never pass it. The type mismatch would confuse any developer reading the component source
- **Gap 2 — No end-to-end blog setup guide**: Getting from "I want a blog" to a working `<List />` required reading 4 separate docs pages. No single guide ties them together
- **Gap 3 — Bases vs `<List />` — which to use?**: No comparison or guidance on when to use which. Obsidian users would naturally reach for Bases; GitHub/CLI users see `<List />` documented first. No signposting
- **Gap 4 — Global `syntaxMode` not in `config.json`**: Setting it globally requires the dashboard (Site Settings → Rendering) — not a `config.json` field. The docs describe both per-page and global options but don't make clear that the global option is dashboard-only

### AI Test Verdict

**The AI path (Claude Code + repo as context) works well** — reading the source is faster and more reliable than reading the docs. A non-AI user would likely hit silent failures on at least 3 of the 4 gaps above.

![[AI-coding-flowershow-note.mp4]]

---

## Prioritised Findings Table

| ID  | Finding                                                                                                                                                                 | Severity     | Session | Effort |
| --- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | ------- | ------ |
| B1  | Auth: OAuthAccountNotLinked for same email on GitHub+Google                                                                                                             | 🔴 Blocker   | 1       | M      |
| B2  | Security: PAT token stored in plain text in `data.json`, committed to public git repos                                                                                  | 🔴 Blocker   | 4       | S      |
| C1  | Obsidian: wrong site name = silent infinite loading                                                                                                                     | 🟠 Confusing | 4       | S      |
| C2  | CLI: "sync" implies ongoing (understandable since we create the site in the dashboard, but as a old CLI user its a bit confusing instead of using "publish" its "sync") | 🟠 Confusing | 3       | S      |
| C3  | CLI: `--name` mismatch gives no hint about correct names                                                                                                                | 🟠 Confusing | 3       | S      |
| C4  | Delete site: copied name has trailing space                                                                                                                             | 🟠 Confusing | 1       | S      |
| C5  | Logout → login redirects to previous account's site                                                                                                                     | 🟠 Confusing | 2       | S      |
| P1  | Drag-drop zone affordance unclear                                                                                                                                       | ⚪ Polish     | 1       | S      |
| P2  | CLI EBADENGINE warnings confuse new users                                                                                                                               | ⚪ Polish     | 3       | S      |
