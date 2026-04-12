# Plan: Cross-document View Transitions (no JavaScript)

This document describes how to add **fade-style transitions** between the gallery grid and full-size images using the **View Transitions API** in **cross-document** mode—**without** any JavaScript.

## Goal

- User clicks a thumbnail on `gallery.html` → browser navigates to a **real HTML page** that shows that image → transition animates (e.g. fade out old page, fade in new page).
- User clicks back (or a link) to the gallery → same transition in reverse.

## Why the current links must change

Links like `href="assets/1.png"` open the **browser’s bare image view**. That is not an HTML document you control, so you **cannot** opt it into View Transitions or style a root transition.  
**Requirement:** each destination must be an **HTML file** (same origin) that embeds the PNG and participates in the API.

## Target architecture (suggested)


| Piece                                       | Purpose                                                                                                                        |
| ------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| `gallery.html`                              | Grid of thumbnails; each thumbnail links to a **viewer** HTML page, not to `*.png` directly.                                   |
| `view/1.html` … `view/12.html` (or similar) | Minimal pages: same chrome (title, back link), one `<img src="../assets/N.png">`. Same-origin path keeps transitions eligible. |
| `style.css` (or a small add-on sheet)       | Shared styles + **View Transition** opt-in and **fade** customization.                                                         |


You can name the folder `view/`—keep paths short and consistent.

## Implementation steps

### 1. Add one HTML viewer per asset

For each image `assets/1.png` … `assets/12.png`:

- Create `view/N.html`.
- Structure suggestion:
  - `<!DOCTYPE html>`, `<html lang="en">`, charset, viewport, `<title>` (e.g. “Gallery — 1”).
  - Link to the same stylesheet as the gallery: e.g. `<link rel="stylesheet" href="../style.css" />` (fix relative path from `view/`).
  - `<body class="background">` (or a dedicated class) so the look matches the rest of the site.
  - A **back** link to `../gallery.html`.
  - A single **full-size** image: `<img src="../assets/N.png" alt="…" />` inside a wrapper you can style (e.g. `.viewer`, `.viewer__img`).

Keep markup **consistent** across all `N.html` files so one block of CSS applies everywhere.

### 2. Point the gallery to viewer pages

In `gallery.html`, replace:

- `href="assets/N.png"` → `href="view/N.html"` (or your chosen path).

Keep `<img src="assets/N.png">` for the thumbnail; only `href` must target the viewer HTML.

### 3. Opt in to cross-document View Transitions (both sides)

Apply the following on **both** `gallery.html` and every `view/N.html` (via the shared CSS they already load).

1. **Meta tag (where supported)** — in `<head>` of both page types:
  ```html
   <meta name="view-transition" content="same-origin" />
  ```
   This signals that same-origin navigations may use view transitions. (Verify [browser docs](https://developer.chrome.com/docs/web-platform/view-transitions/cross-document) for the exact value your target browsers expect.)
2. **CSS at-rule** — enable automatic navigation transitions:
  ```css
   @view-transition {
     navigation: auto;
   }
  ```
   Place this in `style.css` once (shared), so every page that links the sheet participates.

Reference: [MDN — View Transition API](https://developer.mozilla.org/en-US/docs/Web/API/View_Transition_API), including cross-document behavior and support notes.

### 4. Customize the fade (CSS only)

Without JavaScript, you style the **default root transition** using the UA-provided pseudo-elements, for example:

- `::view-transition-old(root)` — snapshot of the **outgoing** page.
- `::view-transition-new(root)` — snapshot of the **incoming** page.

Typical pattern for a **cross-fade**:

- Set `animation-duration` / `animation-timing-function` on both

Tune durations (e.g. 200–400ms) so it feels close to your existing `.gallery .images img { transition: … }` hover timing if you want visual consistency.

Add these rules to `style.css` after the `@view-transition` block so they apply globally.

### 5. (Optional) Shared element transition

To try a **stronger** match between thumbnail and hero image:

- On the gallery: assign a **unique** `view-transition-name` per thumbnail via CSS (e.g. `.gallery .images a:nth-child(1) img { view-transition-name: gallery-1; }` … up to 12).
- On `view/N.html`: set the same `view-transition-name: gallery-N` on the large image.

**Caveat:** named cross-document transitions depend on **browser support** and matching names; if something looks wrong, remove names and rely on the **root** fade only.

### 6. Test correctly

- **Browser:** Cross-document View Transitions are **most complete in Chromium** (Chrome/Edge). Safari/Firefox may differ or omit support—treat as **progressive enhancement**: without support, users still get normal instant navigation.
- **Protocol:** Test `http://localhost` or `https://…`; avoid `file://` if transitions behave oddly.
- **Navigation:** Use **normal `<a href>`** clicks (same as today). No JS.

### 7. Deploy (e.g. GitHub Pages)

- Ensure **relative URLs** still work from `https://user.github.io/repo/` (paths like `view/1.html` and `../assets/1.png` from inside `view/` should resolve if the repo layout matches local).
- After deploy, verify one full round-trip: gallery → viewer → back to gallery.

### 8. Documentation (optional)

- In `README.md`, add one line: live demo uses View Transitions on supporting browsers; 

## Checklist summary

- [x] Create `view/1.html` … `view/12.html` (or equivalent) with embedded images and back link to `gallery.html`.
- [x] Update `gallery.html` thumbnail `href`s to those HTML files.
- [x] Add `<meta name="view-transition" content="same-origin" />` to gallery and viewer templates.
- [x] Add `@view-transition { navigation: auto; }` plus fade rules for `::view-transition-old(new)(root)` in shared CSS.
- [x] Add matching `view-transition-name` for thumbnails and viewer images (`gallery-1` … `gallery-12`).
- [ ] Test in Chrome (and optionally Edge); confirm fallback in other browsers.

## References

- [MDN: View Transition API](https://developer.mozilla.org/en-US/docs/Web/API/View_Transition_API)
- [Chrome: Cross-document view transitions](https://developer.chrome.com/docs/web-platform/view-transitions/cross-document)

