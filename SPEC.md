# Specification: pre-execution decisions (View Transitions gallery)

This document lists **specifications and open decisions** that should be settled **before** implementing [PLAN.md](PLAN.md). The goal is to avoid rework by agreeing on URLs, markup, CSS boundaries, accessibility, and acceptance criteria up front.

---

## 1. Scope and constraints


| ID  | Topic                      | Decision taken                                                                            |
| --- | -------------------------- | ----------------------------------------------------------------------------------------- |
| C1  | **JavaScript**             | Confirm: **no** runtime JS on the gallery or viewer pages (static HTML + CSS only).       |
| C2  | **Pages that participate** | **Only** `gallery.html` + `view/*.html` opt in.                                           |
| C3  | **Carousel → viewer**      | Out of scope for PLAN: `index.html` carousel links today go to `gallery.html`; no change. |


---

## 2. Information architecture and URLs


| ID  | Topic                         | Decision taken                                                                                                                        |
| --- | ----------------------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| IA1 | **Viewer folder name**        | `view/`. Drives all `href`s and relative paths; must stay stable after deploy.                                                        |
| IA2 | **Viewer file naming**        | Pattern: `1.html` … `12.html` . Must match asset names (`assets/1.png` … `assets/12.png`) and gallery links.                          |
| IA3 | **Asset inventory**           | Confirm the canonical list is `**assets/1.png` through `assets/12.png`** with no gaps;                                                |
| IA4 | **Thumbnail `href` vs `src`** | PLAN: thumbnails keep `src="assets/N.png"` and `href` points to viewer HTML. No mixed pattern. No thumbnails linking directly to PNG. |


---

## 3. Viewer page structure (per `view/N.html`)


| ID  | Topic                       | Decision taken                                                                                                                                                                  |
| --- | --------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| V1  | **Document language**       | `lang` on `<html>` (`en` — match `gallery.html`).                                                                                                                               |
| V2  | `**<title>**` pattern**       | Pattern: `Gallery — {N} · Carousel 3D`                                                                                                                                           |
| V3  | **Meta: View Transitions**  | Include `<meta name="view-transition" content="same-origin" />` on **gallery and every viewer** (per PLAN). Confirm no conflicting meta from future head partials.              |
| V4  | **Stylesheet loading**      | Single shared `../style.css`                                                                                                                                                    |
| V5  | **Body / layout classes**   | Add a scoped class (e.g. `body.background.viewer-page`) for viewer-specific layout without affecting `index.html`.                                                              |
| V6  | **Chrome: back navigation** | Text for the back control: “← Gallery” (icon + `aria-label`). Must be a real `<a href="../gallery.html">`                                                                       |
| V7  | **Main landmark**           | Wrap primary content in `<main>` for consistency.                                                                                                                               |
| V8  | **Full-size image**         | Wrapper class names (e.g. `.viewer`, `.viewer__figure`, `.viewer__img`) — BEM or project convention; must be identical in every `N.html`.                                       |
| V9  | **Image sizing**            | Behavior: `max-width` / `max-height`, `object-fit: contain` , padding from viewport edges, safe area on mobile. Prevents layout jump between images of different aspect ratios. |


---

## 4. View Transitions API (CSS behavior)


| ID  | Topic                                          | Decision taken                                                                                                                                                                                                    |
| --- | ---------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| VT1 | **Opt-in**                                     | Use `@view-transition { navigation: auto; }` in the chosen stylesheet(s); confirm it applies to all participating documents that load that CSS.                                                                   |
| VT2 | **Root transition styling**                    | **Default approach :** adjust `animation-duration` and `animation-timing-function` on `::view-transition-old(root)` and `::view-transition-new(root)` only.                                                       |
| VT3 | **Timing**                                     | Target duration (**300 ms**) and easing (`ease-in-out`) — optionally align loosely with `.gallery .images img { transition: … }` in `style.css` for perceived consistency. |
| VT4 | **Named transitions (`view-transition-name`)** | **Yes:** define naming scheme (`gallery-1` … `gallery-12`), matching rules on thumbnails vs viewer `<img>`, and a fallback plan if a browser mis-renders (revert to root-only).                                   |
| VT5 | `**prefers-reduced-motion**`                   | Disable decorative transition animation for users who prefer reduced motion (accessibility), reducing time of animation-duration to 0.01ms for view-transition-... pseudo-elements                                                                                                                      |


---

## 5. Accessibility


| ID  | Topic                  | Decision taken                                                                                                                 |
| --- | ---------------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| A1  | `**alt` text**         | Thumbnails and viewer images: one repeated pattern (“Artwork 1”). Must be consistent with how images are used.                 |
| A2  | **Focus styles**       | Ensure the back link (and any interactive controls) have visible `:focus-visible` styles consistent with the rest of the site. |
| A3  | **Motion sensitivity** | Tie to VT5: reduced-motion handling is part of the spec, not an afterthought.                                                  |


---

## 6. Browser support and progressive enhancement


| ID  | Topic                      | Decision taken                                                                                                                                                |
| --- | -------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| B1  | **Primary target**         | Acknowledge **Chromium** (Chrome/Edge) as the reference for cross-document View Transitions; other engines may show **instant** navigation with no animation. |
| B2  | **User-facing disclaimer** | On **README** states that transitions are an enhancement on supported browsers only.                                                                          |
| B3  | **Testing matrix**         | Minimum browsers to manually verify before considering the work “done” (Chrome desktop + Chrome Android).                                                     |


---

## 7. Deployment (e.g. GitHub Pages)


| ID  | Topic                       | Decision taken                                                                                                                                                                                                                                           |
| --- | --------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| D1  | **Base URL**                | This repo is published as a **project site** (e.g. `https://carloswimmer.github.io/carrousel-3d/`). Confirm navigation uses **relative** paths only (no leading `/` rooted to domain) so `gallery.html`, `view/N.html`, and `assets/` resolve correctly. |
| D2  | **Trailing slash behavior** | If any server redirects folders oddly, confirm links still work with relative links suffice + smoke-test after deploy.                                                                                                                                    |
| D3  | **Post-deploy check**       | Acceptance: open gallery → one viewer → back link → browser **Back** button; all resolve without 404.                                                                                                                                                    |


---

## 8. Maintenance and authoring workflow


| ID  | Topic                                  | Decision taken      |
| --- | -------------------------------------- | ------------------- |
| M1  | **How the 12 HTML files are produced** | **Hand-authored** . |


---

## 9. Documentation updates (after implementation)


| ID   | Topic       | Decision taken                                                   |
| ---- | ----------- | ---------------------------------------------------------------- |
| DOC1 | **README**  | Add a short “View Transitions” note (supported browsers, no JS). |
| DOC2 | **PLAN.md** | Mark steps complete .                                            |


---

## 10. Acceptance criteria (definition of done)

Before closing the implementation task, confirm:

- Every gallery thumbnail `href` targets a viewer HTML page; no thumbnail opens a raw `*.png` as the only navigation target for that flow (unless an explicit exception is documented).
- Every viewer page validates the chosen structure (title pattern, back link, image path, shared CSS).
- `@view-transition` and root pseudo styling match SPEC **VT1–VT3** and **VT5**.
- If named transitions were in scope (**VT4**), they either work as intended in the reference browser or are reverted per fallback plan.
- `prefers-reduced-motion` behavior matches **VT5**.
- Deployed site: round-trip navigation works (**D3**).
- Documentation matches **DOC1** / **DOC2**.

---

## 11. References (shared with PLAN)

- [MDN: View Transition API](https://developer.mozilla.org/en-US/docs/Web/API/View_Transition_API)
- [Chrome: Cross-document view transitions](https://developer.chrome.com/docs/web-platform/view-transitions/cross-document)

---

## Suggested order of resolution

1. **IA1–IA4** (paths and naming) — blocks file creation.
2. **V1–V9** (viewer template) — one template, then duplicate.
3. **VT1–VT5** and **A1–A3** — implement once in CSS/markup.
4. **B1–B3**, **D1–D3** — verify.
5. **M1**, **DOC1–DOC2** — process and docs.

When these rows are filled in, execution of [PLAN.md](PLAN.md) should proceed without ambiguous choices.