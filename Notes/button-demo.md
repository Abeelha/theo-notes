---
title: Button & Badge Demo
date: 2026-03-17
type: note
status: complete
project: theo-notes
tags: [ui, demo, css]
description: Demo of badge and button styles available across all notes.
---

# Button & Badge Demo

## Status Badges

Use these inline next to headings or plan titles to show status at a glance.

<div class="badge-group">
  <span class="badge">In Progress</span>
  <span class="badge badge-done">Done</span>
  <span class="badge badge-blocked">Blocked</span>
  <span class="badge badge-draft">Draft</span>
  <span class="badge badge-idea">Idea</span>
  <span class="badge badge-outline">Review</span>
</div>

---

## Link Buttons (CTAs)

Solid and outline variants — use for navigation between notes.

<div class="btn-group">
  <a href="/Plans" class="btn">View Plans →</a>
  <a href="/Analysis" class="btn btn-outline">Analysis</a>
  <a href="/Ideas" class="btn btn-sm">Ideas</a>
</div>

---

## In Context: Plan Status Row

How you'd use this at the top of a plan or analysis note:

**Flowershow MCP Server** <span class="badge badge-done">Done</span>

**Custom Domain Flow** <span class="badge">In Progress</span>

**Changelog Page** <span class="badge badge-draft">Draft</span>

---

## Copy-Paste Snippets

```html
<!-- Single badge -->
<span class="badge">In Progress</span>
<span class="badge badge-done">Done</span>
<span class="badge badge-blocked">Blocked</span>
<span class="badge badge-draft">Draft</span>
<span class="badge badge-idea">Idea</span>

<!-- Badge group -->
<div class="badge-group">
  <span class="badge">In Progress</span>
  <span class="badge badge-done">Done</span>
</div>

<!-- Link buttons -->
<a href="/Plans" class="btn">View Plans →</a>
<a href="/Analysis" class="btn btn-outline">Analysis</a>
<a href="/Notes" class="btn btn-sm">Notes</a>

<!-- Button group -->
<div class="btn-group">
  <a href="/Plans" class="btn">Plans →</a>
  <a href="/Analysis" class="btn btn-outline">Analysis</a>
</div>
```
