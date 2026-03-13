---
title: Flowershow UI/UX Testing Plan
date: 2026-03-13
type: plan
status: in-progress
project: flowershow
tags: [ux, testing, onboarding, blog, manual-testing]
description: Manual + AI-assisted UX testing of the new Flowershow onboarding flow. Goal is friction points and stuck moments — not improvement suggestions yet.
---

# Flowershow UI/UX Testing Plan

**Goal:** Find where new users get stuck, what's not intuitive, and what features are missing — across all publishing paths and the new onboarding flow.

**Rule:** You are a new user. Don't fix anything. Don't suggest improvements yet. Just hit walls and write them down.

**Output:** A flat list of friction points posted to Discord/Ola when done.

---

## Before You Start

**Step 1: Send the video you already recorded**
You promised Ola the new-user-experience video in both meetings. Send it now before doing anything else.
> Drop it in Discord to Ola directly.

**Step 2: Set up your doc**
Open a new note: `Analysis/2026-03-13-flowershow-ux-findings.md`
Paste this template at the top:
```
## Friction Log
- [ ] [path tested] — [what you tried] — [what happened / what was missing]
```
Every time something feels off, add a line. Don't stop to analyse it — just log and keep going.

**Step 3: Set up screen recording**
Use OBS, Loom, or Windows Game Bar (`Win + G`).
Record audio + screen. Narrate out loud as you go: *"I'm trying to X, I expected Y, I see Z."*
One recording per session below. Keep each under 10 minutes.

---

## Session 1 — Google Sign-Up → New Onboarding (Priority: highest)

**Why:** Ola said this path is hardest to test automatically. She specifically asked for this.

**Step 1:** Open a private/incognito browser window.

**Step 2:** Go to `cloud.flowershow.app` and sign up **with Google** (not GitHub).

**Step 3:** Walk through every screen of the new onboarding flow without any prior knowledge. Don't skip steps. Don't read ahead.

**Step 4:** Try to publish something using the **drag-and-drop** option (now built into the dashboard). Drop any markdown file.

**Step 5:** Log every moment you hesitated, got confused, or didn't know what to do next.

**Commit output:** Add findings to `Analysis/2026-03-13-flowershow-ux-findings.md` under `## Session 1 — Google Onboarding`

---

## Session 2 — GitHub Publishing Path

**Step 1:** From the dashboard (or new account), choose the GitHub publishing path.

**Step 2:** Connect a repo and publish it. Use the new onboarding UI.

**Step 3:** Try to create a **blog** after publishing:
- Navigate to the docs to find how to set up a blog listing
- Try to add the `<List />` component to your site
- Note how long it takes and whether you found the docs easily

**Step 4:** Try to set up a custom domain. Follow what the UI tells you.

**Log:** Anything that required you to leave the app to find info.

---

## Session 3 — CLI Publishing Path

**Step 1:** From the new onboarding, choose the CLI option.

**Step 2:** Follow the instructions shown in the modal exactly as a new user would.

**Step 3:** Publish a folder of markdown files via CLI.

**Step 4:** Note anything that assumed knowledge you wouldn't have as a new user (terminal experience, file paths, etc.)

---

## Session 4 — Obsidian Publishing Path

**Step 1:** From the new onboarding, choose the Obsidian option.

**Step 2:** Follow the steps shown. Install the plugin if needed, generate a token, configure.

**Step 3:** Publish at least 3 notes from Obsidian.

**Step 4:** Try to publish a note with an image. Check if the image appears on the site.

**Step 5:** Try to hide a specific page from the sidebar. Can you figure out how?

---

## Session 5 — Blog End-to-End (the core use case)

**Why:** This is the #1 use case. We already know from audit that the blog setup path is broken.

**Step 1:** Start from the `/uses/blogs` marketing page as if you discovered Flowershow there.

**Step 2:** Click "Get started for free" and go through signup.

**Step 3:** Try to set up a working blog — posts listing, dates showing, images showing — without any prior knowledge of the `<List />` component.

**Step 4:** Time yourself. Log every wall.

**Step 5:** Try the "See demo" button on `/uses/blogs`. Note what happens after you look at the demo. Can you find your way back to setting one up?

---

## Session 6 — Customisation & Config

**Step 1:** Try to change your site's theme. Find where to do it.

**Step 2:** Try to add a custom CSS tweak.

**Step 3:** Try to hide a page from the sidebar using frontmatter.

**Step 4:** Try to set up a newsletter signup form.

**Step 5:** Try to enable comments on a post.

**Log:** Anything you couldn't find in the docs, or docs you couldn't find at all.

---

## Session 7 — AI-Assisted Session (Claude Code + repo as context)

**Why:** Tests the "power user" path Theo described in the meeting. Also useful as a use-case demo.

**Step 1:** Open Claude Code in `C:\Users\Abeelha\Documents\github\flowershow`

**Step 2:** Ask: *"I want to set up a Flowershow blog with Obsidian. What do I need to do?"*

**Step 3:** Follow Claude's instructions and note where the repo knowledge helped vs. where you still got stuck.

**Step 4:** Try: *"How do I hide a page from the sidebar?"* and *"How do I set up a custom domain?"*

**Log:** Gaps between what AI thinks is possible and what the UI actually supports.

---

## Deliverable

When all sessions are done, consolidate your `Friction Log` into a clean list:

```
## Flowershow UX Findings — [date]

### Blockers (couldn't complete the task)
-

### Confusing (got there eventually but it was unclear)
-

### Missing (feature I expected that doesn't exist)
-

### Bugs (something broke)
-
```

Post this list to Ola on Discord. Keep it flat — no proposed fixes unless they're obvious one-liners. Ola said she wants the raw list of stuck moments, not a redesign.

---

## Video Notes

| Session | Record? | Send to |
|---|---|---|
| Video you already made | Already done — **send to Ola NOW** | Ola DM |
| Session 1 (Google onboarding) | Yes — highest value | Save, share with Ola |
| Session 5 (Blog end-to-end) | Yes — shows the core problem | Save, share with Ola |
| Others | Optional — notes are enough | Keep locally |

Short recordings (5–10 min). Narrate out loud. Don't edit them.
