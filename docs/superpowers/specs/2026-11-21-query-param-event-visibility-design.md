# Query-Param Event Visibility — invite/index.html

**Date:** 2026-11-21  
**Status:** Approved

## Goal

Send personalised invite links where each guest only sees the event cards they are invited to. No server required — all logic lives in the page.

## Behaviour

- All 6 event `<article>` cards are **hidden by default**.
- On page load, JS reads every `?k=` query param (supports both `?k=abc&k=xyz` and `?k=abc,xyz`).
- Each key is looked up in a hardcoded `KEY_MAP` object and resolves to an event slug (`haldi`, `mayra`, `sangeet`, `barat`, `varmala`, `pheras`).
- Matching cards are shown; unrecognised keys are silently ignored.
- No key / no valid match → schedule section renders with heading but zero event cards.

## URL Examples

```
# Single event
https://example.com/invite/?k=abc123

# Two events (multiple params)
https://example.com/invite/?k=abc123&k=def456

# Two events (comma-separated)
https://example.com/invite/?k=abc123,def456
```

## Data Model

```js
const KEY_MAP = {
  "abc123": "haldi",
  "def456": "mayra",
  "ghi789": "sangeet",
  "jkl012": "barat",
  "mno345": "varmala",
  "pqr678": "pheras",
};
```

Keys are arbitrary strings chosen by the sender. Add or change keys freely.

## HTML Changes

Each `<article class="event">` gets a `data-event` attribute matching its slug:

```html
<article class="event" data-event="haldi"> … </article>
<article class="event" data-event="mayra"> … </article>
<!-- etc. -->
```

## CSS Change

One rule added to the existing `<style>` block:

```css
.event { display: none; }
```

The JS un-hides matching articles by setting `style.display = ""`.

## JS Logic (~15 lines)

Added at the bottom of the existing `<script>` block:

```js
const KEY_MAP = { /* … */ };

const params = new URLSearchParams(location.search);
const keys = params.getAll("k").flatMap(v => v.split(","));
const slugs = new Set(keys.map(k => KEY_MAP[k]).filter(Boolean));
document.querySelectorAll(".event[data-event]").forEach(el => {
  if (slugs.has(el.dataset.event)) el.style.display = "";
});
```

## What Doesn't Change

- Page structure, envelope animation, scroll-fade behaviour.
- No server, no build step, no dependencies.
