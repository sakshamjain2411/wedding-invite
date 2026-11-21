# Query-Param Event Visibility Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add query-param–driven event visibility to `invite/index.html` so each guest link shows only the events they are invited to.

**Architecture:** All 6 event cards are hidden by default via a CSS rule added to the existing inline `<style>` tag. Each `<article class="event">` gets a `data-event` slug attribute. On page load, ~15 lines of JS read every `?k=` param, resolve each key against a hardcoded `KEY_MAP`, and un-hide matching cards.

**Tech Stack:** Vanilla HTML/CSS/JS — no build step, no dependencies.

---

### Task 1: Add `data-event` attributes to all 6 event articles

**Files:**
- Modify: `invite/index.html:185,193,201,209,217,225`

The 6 `<article class="event">` elements are at lines 185, 193, 201, 209, 217, 225. Each needs a `data-event` attribute with the event slug.

Slugs: `haldi`, `mayra`, `sangeet`, `barat`, `varmala`, `pheras`

- [ ] **Step 1: Add `data-event` to each article**

Change each opening tag as follows (slugs in order of appearance):

```html
<!-- line 185 -->
<article class="event" data-event="haldi">

<!-- line 193 -->
<article class="event" data-event="mayra">

<!-- line 201 -->
<article class="event" data-event="sangeet">

<!-- line 209 -->
<article class="event" data-event="barat">

<!-- line 217 -->
<article class="event" data-event="varmala">

<!-- line 225 -->
<article class="event" data-event="pheras">
```

- [ ] **Step 2: Verify attributes are present**

```bash
grep -n 'data-event' invite/index.html
```

Expected output — 6 lines, one per slug:
```
185:        <article class="event" data-event="haldi">
193:        <article class="event" data-event="mayra">
201:        <article class="event" data-event="sangeet">
209:        <article class="event" data-event="barat">
217:        <article class="event" data-event="varmala">
225:        <article class="event" data-event="pheras">
```

- [ ] **Step 3: Commit**

```bash
git add invite/index.html
git commit -m "feat: add data-event slugs to event articles"
```

---

### Task 2: Hide all event cards by default via CSS

**Files:**
- Modify: `invite/index.html:11` (inline `<style>` tag)

The inline `<style>` on line 11 is a single-line block:
```
<style>:root{...}body{...}img{...}[hidden]{...}</style>
```

Append one rule inside it to hide all `.event` articles by default. The rule must come **after** the existing declarations in the same tag.

- [ ] **Step 1: Append the hide rule to the inline style**

The line currently ends with `[hidden]{display:none!important}</style>`. Change it so it reads:

```
[hidden]{display:none!important}.event[data-event]{display:none}</style>
```

Full updated line 11:
```html
<style>:root{color-scheme:light;padding-top:env(safe-area-inset-top,0px);padding-bottom:env(safe-area-inset-bottom,0px)}body{margin:0;padding:0}img{max-width:100%}[hidden]{display:none!important}.event[data-event]{display:none}</style>
```

Note: scoping the rule to `.event[data-event]` means it only hides cards that have the attribute, so any card missing the attribute (e.g. a future card you haven't slug-ged yet) stays visible.

- [ ] **Step 2: Open the page with no query params and confirm all 6 event cards are invisible**

Open `invite/index.html` in a browser (or via local server). The "Schedule of Events" heading should be visible but all 6 event cards should not appear.

- [ ] **Step 3: Commit**

```bash
git add invite/index.html
git commit -m "feat: hide event cards by default via CSS"
```

---

### Task 3: Add KEY_MAP and param-reading JS

**Files:**
- Modify: `invite/index.html:594` (just before the closing `</script>` at line 595)

Add the key-resolution logic as the last block inside the existing `<script>` tag. Line 595 is `</script>` — insert the new block immediately before it.

- [ ] **Step 1: Insert the JS block before `</script>`**

```js
/* ================= Event visibility ================= */
(function(){
  const KEY_MAP = {
    "haldi-k1":   "haldi",
    "mayra-k2":   "mayra",
    "sangeet-k3": "sangeet",
    "barat-k4":   "barat",
    "varmala-k5": "varmala",
    "pheras-k6":  "pheras",
  };
  const params = new URLSearchParams(location.search);
  const keys   = params.getAll("k").flatMap(v => v.split(",").map(s => s.trim()));
  const slugs  = new Set(keys.map(k => KEY_MAP[k]).filter(Boolean));
  document.querySelectorAll(".event[data-event]").forEach(el => {
    if (slugs.has(el.dataset.event)) el.style.display = "";
  });
})();
```

Replace the placeholder keys (`haldi-k1`, `mayra-k2`, etc.) with whatever real cryptic strings you want to use before sharing links. The key values in `KEY_MAP` are the only thing you need to change.

- [ ] **Step 2: Verify the block was inserted**

```bash
grep -n "KEY_MAP" invite/index.html
```

Expected: one line near line 594.

- [ ] **Step 3: Test — single event via `?k=`**

Open the page with one key, e.g.:
```
invite/index.html?k=haldi-k1
```
Only the Haldi card should appear.

- [ ] **Step 4: Test — multiple events via repeated params**

```
invite/index.html?k=haldi-k1&k=sangeet-k3
```
Only Haldi and Sangeet cards should appear.

- [ ] **Step 5: Test — multiple events via comma-separated**

```
invite/index.html?k=haldi-k1,sangeet-k3
```
Only Haldi and Sangeet cards should appear (same result as Step 4).

- [ ] **Step 6: Test — unknown key**

```
invite/index.html?k=boguskey
```
No event cards should appear.

- [ ] **Step 7: Test — no key at all**

```
invite/index.html
```
No event cards should appear.

- [ ] **Step 8: Commit**

```bash
git add invite/index.html
git commit -m "feat: add query-param event visibility with KEY_MAP"
```

---

### Task 4: Replace placeholder keys with real cryptic keys

**Files:**
- Modify: `invite/index.html` — the `KEY_MAP` block added in Task 3

The placeholder keys (`haldi-k1`, etc.) are human-readable and guessable. Replace them with short random strings before distributing links.

- [ ] **Step 1: Generate 6 random keys**

```bash
for i in 1 2 3 4 5 6; do openssl rand -hex 4; done
```

This outputs 6 eight-character hex strings, e.g.:
```
a3f9c812
7b2e0d45
...
```

- [ ] **Step 2: Update KEY_MAP with the real keys**

Replace each placeholder key in `KEY_MAP` with the generated strings, keeping the same value (slug):

```js
const KEY_MAP = {
  "a3f9c812": "haldi",
  "7b2e0d45": "mayra",
  // … etc
};
```

- [ ] **Step 3: Re-test one link to confirm**

```
invite/index.html?k=<your-new-haldi-key>
```

Only the Haldi card should appear.

- [ ] **Step 4: Commit**

```bash
git add invite/index.html
git commit -m "feat: set real cryptic keys in KEY_MAP"
```

---

## Final Checklist

- [ ] All 6 articles have `data-event` attributes
- [ ] No events show on a bare URL (no `?k=`)
- [ ] Single-event links show exactly one card
- [ ] Multi-key links (`?k=x&k=y` and `?k=x,y`) both work
- [ ] Scroll-fade animations still trigger normally on the visible cards
- [ ] All other page sections (hero, families, venue, story) are unaffected
