# Desktop Layout Design
**Date:** 2026-03-05
**Project:** charlanne.work TikTok-style portfolio
**Approach:** CSS media query (Option A) — single file, no new dependencies

---

## Goal

Add a TikTok-style desktop layout at `≥900px` breakpoint. Mobile layout remains completely unchanged.

---

## Layout Structure

At `≥900px`, the page uses a 3-column flexbox layout:

```
┌─────────────────┬──────────────────┬──────────────┐
│   LEFT SIDEBAR  │   CENTER FEED    │  RIGHT PANEL │
│     240px       │     390px        │    90px      │
│   (fixed)       │   (scrollable)   │  (per card)  │
└─────────────────┴──────────────────┴──────────────┘
```

- Left sidebar: `position: fixed`, full height
- Center feed: same scroll-snap behaviour, locked to 390px wide
- Right actions: per-card, sit outside the card to its right (not a fixed sidebar)
- Below `<900px`: collapses back to current mobile layout, zero changes

---

## Left Sidebar

A new `<aside id="desktop-sidebar">` element added to `index.html`, hidden on mobile.

**Contents (top to bottom):**
- Logo: `media/cperiod256.png` (~48px)
- Name: `charlanne herold`
- Tagline: `available for contract work`
- Divider
- Links with icons:
  - 🌐 charlanne.work → `https://www.charlanne.work`
  - 💼 LinkedIn → `https://www.linkedin.com/in/charlanneherold`
  - 📷 Instagram → `https://instagram.com/charlanne.design`
  - ✉️ Email → `mailto:` (use existing contact email)

**Styling:**
- Background: `--bg` (`#0a0a0a`)
- Link hover colour: `--lime` (`#c8f135`)
- Fixed position, left edge of viewport

---

## Center Feed (Desktop)

- Cards: `border-radius: 12px`, `overflow: hidden` — rounded corners like TikTok desktop
- Card width: `390px`, height: `calc(100vh - 32px)`
- Small vertical gap between cards (`16px`)
- Existing mobile action overlay (`.actions`) hidden on desktop via `display: none`

---

## Right Action Buttons (Desktop)

The `.actions` block is repositioned outside the card on desktop, sitting to the card's right, vertically centred.

**Order (top to bottom):**
1. Avatar (links to `charlanne.work`)
2. Like button + count
3. Comment button + count
4. View button + "view" label (only shown if project has a `url`)

**Styling:**
- Same icons and interaction as mobile
- Count displayed below each icon (like TikTok desktop)
- No circles/backgrounds on icons (consistent with mobile)

---

## What Does NOT Change

- Comments drawer
- Sound banner + auto-hide
- Like/comment localStorage persistence
- Carousel logic
- All mobile styles and behaviour
- `projects.js` data

---

## Implementation Notes

- All changes in `index.html` only
- Add `<aside id="desktop-sidebar">` to HTML before `.feed`
- Add `@media (min-width: 900px)` CSS block
- On desktop: `.actions` switches from `position: absolute` (overlaid) to a flex child sitting outside the card
- Bump `projects.js?v=` cache-bust if projects.js changes
