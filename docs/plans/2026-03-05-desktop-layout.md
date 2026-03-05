# Desktop Layout + Mobile Hamburger Menu Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Add a TikTok-style 3-column desktop layout (left sidebar, 390px center feed, right actions), a mobile hamburger menu, and a desktop right-side comments panel — all within `index.html` using a `@media (min-width: 900px)` CSS block.

**Architecture:** A new `<aside id="desktop-sidebar">` is added to the HTML body. A `.card-frame` wrapper is added to the JS card template to enable per-card content clipping (required for rounded corners + actions outside card). Desktop layout uses body flexbox; mobile sidebar uses a slide-in panel triggered by a hamburger button.

**Tech Stack:** Vanilla HTML/CSS/JS, single file (`index.html`). No build step. No new dependencies.

---

## Key Technical Notes (read before touching code)

- **`.card-frame` wrapper** is the core structural change. Currently `.card` clips its content via `overflow: hidden`. On desktop the `.actions` must sit *outside* the clip boundary. Solution: move clipping to a new `.card-frame` child div; `.actions` stays as a sibling of `.card-frame` inside `.card`.
- **Carousel slide width**: Currently `.carousel-slide` uses `flex: 0 0 100vw`. On desktop (card is 390px) this breaks layout. Override to `flex: 0 0 390px` at the breakpoint.
- **Gradient/grain pseudo-elements** currently on `.card::before/after` — must move to `.card-frame::before/after` so they don't spill outside the rounded card on desktop.
- **Mobile**: all existing behaviour unchanged. `.card-frame { position: absolute; inset: 0 }` on mobile makes it invisible as a structural change.
- **Hamburger menu**: fixed `top: 16px; left: 16px` on mobile. Same sidebar content as desktop. Slides in from left on tap.
- **Email placeholder**: replace `YOUR_EMAIL` in the sidebar HTML with the real address.

---

## Task 1: Add sidebar HTML and hamburger button to `<body>`

**File:** `index.html` — HTML section (after `<body>`, before `<div id="feed">`)

**Step 1: Add the aside + hamburger button**

Locate the line `<div id="feed"></div>` in the HTML body and replace it with:

```html
<aside id="desktop-sidebar">
  <div class="sidebar-profile">
    <img src="media/cperiod256.png" alt="charlanne herold" class="sidebar-logo">
    <div class="sidebar-name">charlanne herold</div>
    <div class="sidebar-tagline">available for contract work</div>
  </div>
  <div class="sidebar-divider"></div>
  <nav class="sidebar-links">
    <a href="https://www.charlanne.work" target="_blank" rel="noopener" class="sidebar-link">
      <svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="12" cy="12" r="10"/><line x1="2" y1="12" x2="22" y2="12"/><path d="M12 2a15.3 15.3 0 0 1 4 10 15.3 15.3 0 0 1-4 10 15.3 15.3 0 0 1-4-10 15.3 15.3 0 0 1 4-10z"/></svg>
      charlanne.work
    </a>
    <a href="https://www.linkedin.com/in/charlanneherold" target="_blank" rel="noopener" class="sidebar-link">
      <svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M16 8a6 6 0 0 1 6 6v7h-4v-7a2 2 0 0 0-2-2 2 2 0 0 0-2 2v7h-4v-7a6 6 0 0 1 6-6z"/><rect x="2" y="9" width="4" height="12"/><circle cx="4" cy="4" r="2"/></svg>
      linkedin
    </a>
    <a href="https://instagram.com/charlanne.design" target="_blank" rel="noopener" class="sidebar-link">
      <svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><rect x="2" y="2" width="20" height="20" rx="5" ry="5"/><path d="M16 11.37A4 4 0 1 1 12.63 8 4 4 0 0 1 16 11.37z"/><line x1="17.5" y1="6.5" x2="17.51" y2="6.5"/></svg>
      instagram
    </a>
    <a href="mailto:YOUR_EMAIL" class="sidebar-link">
      <svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M4 4h16c1.1 0 2 .9 2 2v12c0 1.1-.9 2-2 2H4c-1.1 0-2-.9-2-2V6c0-1.1.9-2 2-2z"/><polyline points="22,6 12,13 2,6"/></svg>
      email
    </a>
  </nav>
</aside>

<!-- Mobile hamburger -->
<button id="hamburger-btn" aria-label="Open menu">
  <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
    <line x1="3" y1="6" x2="21" y2="6"/><line x1="3" y1="12" x2="21" y2="12"/><line x1="3" y1="18" x2="21" y2="18"/>
  </svg>
</button>

<!-- Mobile sidebar backdrop -->
<div id="mobile-sidebar-backdrop"></div>

<div id="feed"></div>
```

**Step 2: Verify HTML is valid**

Open `index.html` in a browser (or use the dev server). The page should load without errors. The aside won't be visible yet (no CSS). The hamburger button will appear unstyled.

**Step 3: Commit**
```bash
cd "/Users/charlannemorse/Desktop/claude_test/tiktok portfolio"
git add index.html
git commit -m "feat: add desktop sidebar and mobile hamburger HTML"
```

---

## Task 2: Add `.card-frame` wrapper to JS card template

**File:** `index.html` — JS section, the `card.innerHTML = \`...\`` template (~line 748)

**Step 1: Locate the card template**

Find the block starting with `card.innerHTML = \``. It currently looks like:
```js
card.innerHTML = `
  ${carouselHTML}
  ${hasVid ? `<button class="mute-btn"...>...</button><div class="play-pause-overlay">...</div>` : ''}
  <div class="actions">...</div>
  <div class="card-bottom">...</div>
`;
```

**Step 2: Wrap carousel + mute + play-pause + card-bottom in `.card-frame`**

Change the template so `.card-frame` wraps everything EXCEPT `.actions`:

```js
card.innerHTML = `
  <div class="card-frame">
    ${carouselHTML}

    ${hasVid ? `
    <button class="mute-btn" data-id="${p.id}">
      <svg class="icon-muted" width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="#C8FF00" stroke-width="2">
        <polygon points="11 5 6 9 2 9 2 15 6 15 11 19 11 5"/>
        <line x1="23" y1="9" x2="17" y2="15"/><line x1="17" y1="9" x2="23" y2="15"/>
      </svg>
      <svg class="icon-unmuted" width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="#f0f0f0" stroke-width="2" style="display:none">
        <polygon points="11 5 6 9 2 9 2 15 6 15 11 19 11 5"/>
        <path d="M19.07 4.93a10 10 0 0 1 0 14.14M15.54 8.46a5 5 0 0 1 0 7.07"/>
      </svg>
    </button>
    <div class="play-pause-overlay">
      <button class="play-pause-btn" data-id="${p.id}">
        <svg class="icon-pause" width="22" height="22" viewBox="0 0 24 24" fill="var(--lime)" stroke="none">
          <rect x="6" y="4" width="4" height="16" rx="1"/><rect x="14" y="4" width="4" height="16" rx="1"/>
        </svg>
        <svg class="icon-play" width="22" height="22" viewBox="0 0 24 24" fill="var(--lime)" stroke="none" style="display:none;margin-left:3px">
          <polygon points="5 3 19 12 5 21 5 3"/>
        </svg>
      </button>
    </div>` : ''}

    <div class="card-bottom">
      <div class="caption">
        <div class="caption-handle">@charlanne.work</div>
        <div class="caption-title">${p.title}</div>
        ${tagsHTML}
        <div class="caption-hook">${p.caption}</div>
      </div>
    </div>
  </div>

  <div class="actions">
    ${p.url ? `
    <button class="action-btn visit-btn" data-url="${p.url}">
      <div class="action-icon">
        <svg width="22" height="22" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
          <path d="M18 13v6a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V8a2 2 0 0 1 2-2h6"/>
          <polyline points="15 3 21 3 21 9"/><line x1="10" y1="14" x2="21" y2="3"/>
        </svg>
      </div>
      <span class="action-count">view</span>
    </button>` : ''}
    <a class="profile-btn" href="https://www.charlanne.work" target="_blank" rel="noopener">
      <div class="avatar"><img src="media/cperiod256.png" alt="charlanne.work"></div>
    </a>
    <button class="action-btn like-btn" data-id="${p.id}">
      <div class="action-icon">
        <svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
          <path d="M20.84 4.61a5.5 5.5 0 0 0-7.78 0L12 5.67l-1.06-1.06a5.5 5.5 0 0 0-7.78 7.78l1.06 1.06L12 21.23l7.78-7.78 1.06-1.06a5.5 5.5 0 0 0 0-7.78z"/>
        </svg>
      </div>
      <span class="action-count like-count" data-id="${p.id}">${state.likeCounts[p.id]}</span>
    </button>
    <button class="action-btn comment-btn" data-id="${p.id}">
      <div class="action-icon">
        <svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
          <path d="M21 15a2 2 0 0 1-2 2H7l-4 4V5a2 2 0 0 1 2-2h14a2 2 0 0 1 2 2z"/>
        </svg>
      </div>
      <span class="action-count comment-count" data-id="${p.id}">0</span>
    </button>
  </div>
`;
```

**Step 3: Add `.card-frame` mobile CSS** (add inside `<style>`, after `.card` block)

```css
/* ── CARD FRAME (clips content, holds gradient/grain) ── */
.card-frame {
  position: absolute;
  inset: 0;
  overflow: hidden;
  display: flex;
  align-items: flex-end;
  background: var(--dark);
}
```

**Step 4: Move gradient + grain from `.card::after/before` to `.card-frame::after/before`**

Find `.card::after` and `.card::before` in the CSS. Change both selectors:
- `.card::after` → `.card-frame::after`
- `.card::before` → `.card-frame::before`

**Step 5: Update `.card` CSS — remove properties that moved to `.card-frame`**

The `.card` rule currently has `overflow: hidden`, `align-items: flex-end`, and `background: var(--dark)`. Remove those three since they now live on `.card-frame`:

```css
.card {
  position: relative;
  height: 100vh;
  width: 100%;
  scroll-snap-align: start;
  scroll-snap-stop: always;
  display: flex;
  /* removed: align-items, overflow, background */
}
```

**Step 6: Verify mobile layout is identical to before**

Open in browser at mobile size (375px). The feed should look exactly the same as before this task. Check:
- Videos play
- Images display
- Carousel swipe works
- Like/comment buttons work
- Card title/caption visible at bottom

**Step 7: Commit**
```bash
git add index.html
git commit -m "refactor: add card-frame wrapper for desktop clip isolation"
```

---

## Task 3: Add desktop CSS — body layout, feed, sidebar

**File:** `index.html` — CSS section, add a new `@media (min-width: 900px)` block at the end of `<style>`, before `</style>`

**Step 1: Add the desktop media query block**

```css
/* ════════════════════════════════════════════
   DESKTOP LAYOUT  (≥ 900px)
   ════════════════════════════════════════════ */
@media (min-width: 900px) {

  /* ── PAGE LAYOUT ── */
  body {
    display: flex;
    flex-direction: row;
    overflow: hidden;
  }

  /* ── LEFT SIDEBAR ── */
  #desktop-sidebar {
    display: flex;
    flex-direction: column;
    width: 240px;
    flex-shrink: 0;
    height: 100vh;
    padding: 32px 24px;
    border-right: 1px solid rgba(255,255,255,0.07);
    position: fixed;
    left: 0;
    top: 0;
    z-index: 100;
    background: var(--black);
    overflow-y: auto;
  }

  .sidebar-logo {
    width: 48px;
    height: 48px;
    border-radius: 50%;
    object-fit: cover;
    border: 2px solid var(--lime);
    margin-bottom: 12px;
  }

  .sidebar-name {
    font-family: 'Syne', sans-serif;
    font-size: 14px;
    font-weight: 700;
    color: var(--white);
    letter-spacing: 0.03em;
    text-transform: lowercase;
  }

  .sidebar-tagline {
    font-family: 'DM Mono', monospace;
    font-size: 10px;
    color: var(--lime);
    letter-spacing: 0.08em;
    text-transform: uppercase;
    margin-top: 4px;
    opacity: 0.85;
  }

  .sidebar-divider {
    width: 100%;
    height: 1px;
    background: rgba(255,255,255,0.1);
    margin: 20px 0;
  }

  .sidebar-links {
    display: flex;
    flex-direction: column;
    gap: 6px;
  }

  .sidebar-link {
    display: flex;
    align-items: center;
    gap: 10px;
    font-family: 'DM Mono', monospace;
    font-size: 11px;
    color: rgba(240,240,240,0.6);
    text-decoration: none;
    padding: 8px 10px;
    border-radius: 8px;
    letter-spacing: 0.05em;
    transition: color 0.2s, background 0.2s;
  }
  .sidebar-link:hover {
    color: var(--lime);
    background: rgba(200,255,0,0.07);
  }
  .sidebar-link svg {
    flex-shrink: 0;
    opacity: 0.7;
  }
  .sidebar-link:hover svg {
    opacity: 1;
  }

  /* ── FEED (center column) ── */
  #feed {
    margin-left: 240px;
    flex: 1;
    display: flex;
    flex-direction: column;
    align-items: center;
    overflow-y: scroll;
    scroll-snap-type: y mandatory;
  }

  /* ── CARD (desktop: flex row, transparent shell) ── */
  .card {
    width: auto;
    height: 100vh;
    display: flex;
    flex-direction: row;
    align-items: center;
    justify-content: center;
    gap: 16px;
    background: transparent;
    overflow: visible;
    flex-shrink: 0;
  }

  /* ── CARD FRAME (the visible rounded card) ── */
  .card-frame {
    position: relative;
    inset: auto;
    width: 390px;
    height: calc(100vh - 48px);
    border-radius: 12px;
    overflow: hidden;
    flex-shrink: 0;
  }

  /* ── CAROUSEL SLIDES: fixed width on desktop ── */
  .carousel-slide {
    flex: 0 0 390px;
    width: 390px;
  }

  /* Fill card frame with media */
  .carousel-slide-image .card-media,
  .carousel-slide-video .card-media {
    width: 100%;
    height: 100%;
    object-fit: cover;
  }

  /* ── ACTIONS (outside card, to the right) ── */
  .actions {
    position: relative;
    right: auto;
    bottom: auto;
    z-index: auto;
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 20px;
    width: 72px;
  }

  /* Hide mobile-only elements on desktop */
  #hamburger-btn,
  #mobile-sidebar-backdrop { display: none; }

  /* Show desktop sidebar */
  #desktop-sidebar { display: flex; }

  /* Progress dots: reposition for desktop */
  #dots {
    left: calc(240px + 50%);
    transform: translateX(-50%);
  }

  /* Card-bottom text padding adjustment */
  .card-bottom {
    padding: 0 16px 36px 16px;
  }
}
```

**Step 2: Hide desktop sidebar on mobile (default state)**

Find the existing CSS for `#desktop-sidebar` (if any). If not present, add this in the base (non-media-query) CSS:
```css
#desktop-sidebar { display: none; }
#hamburger-btn { display: flex; }
#mobile-sidebar-backdrop { display: none; }
```

**Step 3: Verify desktop layout at 1280px**

Toggle browser to desktop view. Check:
- Left sidebar visible with logo, name, tagline, links
- Feed centered, cards 390px wide with rounded corners
- Actions sit to the right of each card
- Mobile layout unchanged at 375px

**Step 4: Commit**
```bash
git add index.html
git commit -m "feat: add desktop 3-column CSS layout"
```

---

## Task 4: Add mobile hamburger menu CSS + JS

**File:** `index.html` — CSS (base styles) + JS section

**Step 1: Add hamburger button + mobile sidebar CSS** (in base `<style>`, before the desktop media query)

```css
/* ── HAMBURGER BUTTON (mobile only) ── */
#hamburger-btn {
  position: fixed;
  top: 14px;
  left: 14px;
  z-index: 50;
  width: 38px;
  height: 38px;
  background: rgba(8,8,8,0.65);
  border: 1px solid rgba(255,255,255,0.15);
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  color: var(--white);
  backdrop-filter: blur(10px);
  transition: border-color 0.2s, background 0.2s;
}
#hamburger-btn:hover { background: rgba(200,255,0,0.1); border-color: var(--lime); }

/* ── MOBILE SIDEBAR BACKDROP ── */
#mobile-sidebar-backdrop {
  position: fixed;
  inset: 0;
  background: rgba(0,0,0,0.6);
  z-index: 200;
  opacity: 0;
  pointer-events: none;
  transition: opacity 0.3s;
}
#mobile-sidebar-backdrop.open {
  opacity: 1;
  pointer-events: all;
}

/* ── DESKTOP SIDEBAR (also used as mobile slide-in) ── */
#desktop-sidebar {
  position: fixed;
  top: 0;
  left: 0;
  height: 100vh;
  width: 260px;
  background: var(--black);
  border-right: 1px solid rgba(255,255,255,0.07);
  z-index: 300;
  display: flex;
  flex-direction: column;
  padding: 32px 24px;
  overflow-y: auto;
  /* Mobile: hidden off-screen by default */
  transform: translateX(-100%);
  transition: transform 0.3s cubic-bezier(0.32, 0.72, 0, 1);
}
#desktop-sidebar.open {
  transform: translateX(0);
}

.sidebar-logo {
  width: 48px;
  height: 48px;
  border-radius: 50%;
  object-fit: cover;
  border: 2px solid var(--lime);
  margin-bottom: 12px;
}
.sidebar-name {
  font-family: 'Syne', sans-serif;
  font-size: 14px;
  font-weight: 700;
  color: var(--white);
  letter-spacing: 0.03em;
  text-transform: lowercase;
}
.sidebar-tagline {
  font-family: 'DM Mono', monospace;
  font-size: 10px;
  color: var(--lime);
  letter-spacing: 0.08em;
  text-transform: uppercase;
  margin-top: 4px;
  opacity: 0.85;
}
.sidebar-divider {
  width: 100%;
  height: 1px;
  background: rgba(255,255,255,0.1);
  margin: 20px 0;
}
.sidebar-links {
  display: flex;
  flex-direction: column;
  gap: 6px;
}
.sidebar-link {
  display: flex;
  align-items: center;
  gap: 10px;
  font-family: 'DM Mono', monospace;
  font-size: 11px;
  color: rgba(240,240,240,0.6);
  text-decoration: none;
  padding: 8px 10px;
  border-radius: 8px;
  letter-spacing: 0.05em;
  transition: color 0.2s, background 0.2s;
}
.sidebar-link:hover {
  color: var(--lime);
  background: rgba(200,255,0,0.07);
}
.sidebar-link svg { flex-shrink: 0; opacity: 0.7; }
.sidebar-link:hover svg { opacity: 1; }
```

**Step 2: Remove the duplicate sidebar CSS from the desktop media query**

In the `@media (min-width: 900px)` block added in Task 3, remove the `.sidebar-logo`, `.sidebar-name`, `.sidebar-tagline`, `.sidebar-divider`, `.sidebar-links`, `.sidebar-link` rules — they are now in the base CSS. Keep only the layout-specific overrides in the media query:

```css
@media (min-width: 900px) {
  /* Override mobile slide-in to always-visible fixed sidebar */
  #desktop-sidebar {
    transform: translateX(0) !important;
    width: 240px;
  }
  #hamburger-btn,
  #mobile-sidebar-backdrop { display: none !important; }
  /* ... rest of desktop layout rules ... */
}
```

**Step 3: Add hamburger JS** — add this near the bottom of the `<script>` block, after the existing event listeners:

```js
// ── HAMBURGER MENU ──
const hamburgerBtn = document.getElementById('hamburger-btn');
const mobileSidebarBackdrop = document.getElementById('mobile-sidebar-backdrop');
const desktopSidebar = document.getElementById('desktop-sidebar');

function openSidebar() {
  desktopSidebar.classList.add('open');
  mobileSidebarBackdrop.classList.add('open');
  document.body.style.overflow = 'hidden';
}

function closeSidebar() {
  desktopSidebar.classList.remove('open');
  mobileSidebarBackdrop.classList.remove('open');
  document.body.style.overflow = '';
}

hamburgerBtn.addEventListener('click', openSidebar);
mobileSidebarBackdrop.addEventListener('click', closeSidebar);
```

**Step 4: Verify mobile hamburger**

At 375px:
- Hamburger button visible top-left
- Tapping opens sidebar from left
- Tapping backdrop closes it
- Feed still scrolls normally when closed

**Step 5: Verify desktop sidebar**

At 1280px:
- Hamburger button hidden
- Sidebar always visible on left
- Backdrop not visible

**Step 6: Commit**
```bash
git add index.html
git commit -m "feat: add mobile hamburger menu and slide-in sidebar"
```

---

## Task 5: Polish + edge cases

**File:** `index.html`

**Step 1: Fix carousel dots position on desktop**

The carousel dots use `top: min(calc(50% + min(62.5vw, 50vh) + 10px), ...)` which is calculated for full-width mobile. Override for desktop (card is 390px wide):

```css
@media (min-width: 900px) {
  .carousel-dots {
    top: min(calc(50% + min(195px, 50vh) + 10px), calc(100% - 80px));
  }
}
```

**Step 2: Fix mute button z-index on desktop**

The mute button is `position: absolute; top: 16px; right: 16px` inside `.card-frame`. Verify it's still visible (it should be since it's inside `.card-frame` which is the clipping container).

**Step 3: Fix progress dots on desktop**

The `#dots` bar is `position: fixed; right: ...`. On desktop, it overlaps the right action area. Either hide it on desktop or reposition:

```css
@media (min-width: 900px) {
  #dots { display: none; }
}
```

(The scroll position is clear from the feed itself on desktop; the dots are redundant.)

**Step 4: Scroll-hint on desktop**

The scroll hint should say "scroll" instead of "swipe" on desktop. This is optional — simplest fix is to hide it:

```css
@media (min-width: 900px) {
  #scroll-hint { display: none; }
}
```

**Step 5: Commit**
```bash
git add index.html
git commit -m "feat: desktop polish — dots, scroll hint, carousel dots fix"
```

---

## Task 6: Desktop right-side comments panel

**File:** `index.html` — CSS + JS

On desktop, clicking the comment button opens a panel that slides in from the **right** (like TikTok desktop). Mobile bottom-sheet behaviour is unchanged.

**Step 1: Add desktop comment drawer CSS** inside the `@media (min-width: 900px)` block:

```css
@media (min-width: 900px) {
  /* Comment drawer becomes a right-side panel */
  #comment-drawer {
    top: 0;
    left: auto;
    right: 0;
    bottom: 0;
    width: 380px;
    height: 100vh;
    border-top: none;
    border-left: 1px solid rgba(200,255,0,0.15);
    border-radius: 0;
    transform: translateX(100%);
  }
  #comment-drawer.open {
    transform: translateX(0);
  }

  /* Hide the drag handle on desktop */
  .drawer-handle { display: none; }

  /* Lighter backdrop on desktop (doesn't cover the video heavily) */
  #backdrop {
    background: rgba(0,0,0,0.3);
  }
}
```

**Step 2: Add comment count to the drawer header**

Currently `drawer-header` shows just `COMMENTS`. Update it to show the count for the active card.

Find the `openComments(id)` function (or wherever `comment-drawer` gets the `open` class). After setting `state.activeCommentCard = id`, update the header:

```js
// Inside the function that opens the comment drawer:
const drawerHeader = document.querySelector('.drawer-header span');
const count = state.commentsByCard[id] ? state.commentsByCard[id].length : 0;
drawerHeader.textContent = `COMMENTS  ${count}`;
```

Also update `renderComments(id)` to refresh the header count after a comment is added/deleted:
```js
const drawerHeader = document.querySelector('.drawer-header span');
if (drawerHeader) {
  drawerHeader.textContent = `COMMENTS  ${state.commentsByCard[id].length}`;
}
```

**Step 3: Verify on desktop**

- Click comment button → panel slides in from right
- Panel sits to the right of the action buttons
- Video is still visible on the left
- Closing (✕ button or backdrop) slides panel back out
- Count in header updates when comments are added/deleted

**Step 4: Verify on mobile**

- Comment drawer still slides up from bottom
- Behaviour unchanged

**Step 5: Commit**
```bash
git add index.html
git commit -m "feat: desktop right-side comments panel"
```

---

## Task 7: Push to GitHub + deploy to Vercel

**Step 1: Push all commits**
```bash
cd "/Users/charlannemorse/Desktop/claude_test/tiktok portfolio"
git push origin main
```
(Use PAT token as password if prompted — same as last time, but generate a new one first.)

**Step 2: Deploy on Vercel**
1. Go to [vercel.com](https://vercel.com), sign in with GitHub
2. Click **Add New Project**
3. Import `CharHer1221/tiktokstyle-portfolio`
4. Framework: **Other** (static site, no build)
5. Click **Deploy**

Vercel auto-deploys on every push to `main` going forward.
