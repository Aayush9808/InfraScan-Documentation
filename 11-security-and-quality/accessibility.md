# Accessibility

A source-level review of the InfraScan dashboard against **WCAG 2.2 Level AA** success criteria, with the contrast mathematics shown so the findings can be checked rather than trusted.

> **No conformance claim is made.** There is no automated scan, no screen-reader session, no 200% zoom or reflow pass, and no testing with disabled users. This page is a *reviewed* assessment of the shipped source, not a certification. See [What was not tested](#what-was-not-tested).

- [Scope and method](#scope-and-method)
- [Summary of findings](#summary-of-findings)
- [Colour contrast](#colour-contrast)
- [Text size, zoom and reflow](#text-size-zoom-and-reflow)
- [Keyboard operation](#keyboard-operation)
- [Focus management](#focus-management)
- [Names, roles and values](#names-roles-and-values)
- [Status messages](#status-messages)
- [Motion and animation](#motion-and-animation)
- [Forms, labels and help text](#forms-labels-and-help-text)
- [What was not tested](#what-was-not-tested)
- [Prioritised remediation](#prioritised-remediation)
- [How to verify conformance later](#how-to-verify-conformance-later)
- [Related pages](#related-pages)

---

## Scope and method

Reviewed artefacts:

| Artefact | What was inspected |
|---|---|
| `web/index.html` | Landmarks, headings, ARIA roles/states, labels, live regions, dialogs |
| `web/assets/css/app.css` | Colour tokens, font sizes, focus styling, breakpoints, reduced-motion block |
| `web/assets/js/app.js` | Router, sidebar drawer, focus movement, status rendering |
| `web/assets/js/inspect.js` | Tab widget semantics and keyboard handling, lightbox |
| `web/assets/js/palette.js` | Command palette focus trap, `aria-activedescendant`, listbox pattern |
| `web/assets/js/camera.js` | Native dialog usage, stream cleanup |
| `web/assets/js/api.js` / `store.js` | Error/status announcements, persisted settings |

Methods:

- **Static source review** of markup, ARIA and CSS.
- **Relative-luminance calculation** using the WCAG 2.x formula on the declared CSS custom properties, with translucent surfaces composited over the base background by alpha blending.
- **Breakpoint reading** of the responsive rules to determine when content is removed from the accessibility tree.
- **No** axe/Lighthouse run, **no** NVDA/JAWS/VoiceOver session, **no** user testing, **no** 400% zoom or 320 CSS-pixel reflow test, **no** Windows High Contrast Mode or `forced-colors` test.

Contrast values below are computed from the stylesheet as committed. They are reproducible with the formula in [Colour contrast](#colour-contrast).

## Summary of findings

| ID | Finding | WCAG 2.2 SC | Severity | Status |
|---|---|---|---|---|
| ACC-01 | `--muted-dim` text is 3.90–4.22:1, below the 4.5:1 minimum; used 35 times | 1.4.3 Contrast (Minimum) | Medium | Open |
| ACC-02 | Palette trigger has no accessible name once its text is hidden at ≤820 px | 4.1.2 Name, Role, Value | Medium | Open |
| ACC-03 | Command-palette search input has a placeholder but no accessible name | 4.1.2 Name, Role, Value | Medium | Open |
| ACC-04 | Model status chip becomes a colour-only dot at ≤620 px | 1.4.1 Use of Color, 4.1.2 | Medium | Open |
| ACC-05 | Route changes and drawer close do not move or restore focus | 2.4.3 Focus Order | Medium | Open |
| ACC-06 | Tabs lack `aria-controls`/`tabpanel` linkage | 1.3.1 Info and Relationships | Low | Open |
| ACC-07 | Component borders at 1.20–1.52:1 are far below the 3:1 non-text minimum | 1.4.11 Non-text Contrast | Low | Open |
| ACC-08 | Helper and label typography falls to 0.52–0.68 rem (≈8.3–10.9 px) | 1.4.4 Resize Text (risk, not a failure) | Low | Open |
| ACC-09 | No `sr-only` utility; "hidden" text is removed from the tree with `display:none` | 1.3.1 / 4.1.2 (technique) | Informational | Open |
| ACC-10 | Skip link, focus ring, reduced-motion support, live regions and native dialogs are correct | 2.4.1, 2.4.7, 2.3.3, 4.1.3 | — | **Conforms in source** |

The remaining two findings in the chapter index, SEC-14 to SEC-17, correspond to ACC-01, ACC-02/03, ACC-04 and ACC-05. SEC-18 is the overall evidence boundary.

---

## Colour contrast

### Method

For each pair of foreground and background colours:

1. Convert sRGB values to linear RGB (`c/12.92` when `c ≤ 0.04045`, otherwise `((c+0.055)/1.055)^2.4`).
2. Compute relative luminance `L = 0.2126R + 0.7152G + 0.0722B`.
3. Compute the ratio `(L_lighter + 0.05) / (L_darker + 0.05)`.

Backgrounds are taken from the tokens `--bg: #05080f` and `--bg-deep: #03060b`. Card surfaces use `--surface: rgba(13, 23, 38, 0.66)`, which composites over `--bg` to an effective `#0a121e`. Transparent overlays such as `rgba(255,255,255,0.045)` inside `.glass` lighten it slightly, so the figures below are conservative for text on cards.

### Text colours

| Token | Hex | On `--bg` | On card surface | Required (normal text) | Result |
|---|---|---:|---:|---:|---|
| `--text` | `#e9f1fa` | 17.58 | 16.48 | 4.5 | Pass |
| `--text-soft` | `#c2d2e4` | 13.01 | 12.19 | 4.5 | Pass |
| `--muted` | `#8299b3` | 6.83 | 6.40 | 4.5 | Pass |
| **`--muted-dim`** | **`#5f748c`** | **4.16** | **3.90** | **4.5** | **Fail (ACC-01)** |
| `--accent` | `#5eead4` | 13.54 | 12.70 | 4.5 | Pass |
| `--accent-strong` | `#2dd4bf` | 10.76 | 10.09 | 4.5 | Pass |
| `--sky` | `#38bdf8` | 9.35 | 8.77 | 4.5 | Pass |
| `--violet` | `#a78bfa` | 7.36 | 6.90 | 4.5 | Pass |
| `--critical` | `#ff6575` | 7.03 | 6.59 | 4.5 | Pass |
| `--moderate` | `#f7b955` | 11.47 | 10.76 | 4.5 | Pass |
| `--minor` | `#4cc9f0` | 10.42 | 9.77 | 4.5 | Pass |
| `--success` | `#5eead4` | 13.54 | 12.70 | 4.5 | Pass |
| `--danger` | `#ff7a7a` | 7.94 | 7.44 | 4.5 | Pass |

**ACC-01 in detail.** `--muted-dim` is the only failing text token. It is used **35 times**, including `.control-hint` (0.66 rem), `.dropzone small` (0.68 rem), `.legend small` (0.65 rem), `.sidebar__foot` (0.65 rem), `.topbar__eyebrow` (0.62 rem), `.hint` (0.65 rem), `.mini-label` (0.68 rem), `.stat > span` (0.64 rem) and `.brand__text small` (0.62 rem). These are exactly the helper, hint and metadata texts a user relies on to understand the tool, so the failure is functional, not cosmetic.

**Remediation.** Raising `--muted-dim` from `#5f748c` to about `#75899f` (or darker, e.g. `#6d8299`) brings it above 4.5:1 on both backgrounds while keeping the intended hierarchy between `--muted` and `--text-soft`. A single token change fixes all 35 occurrences; the alternative is replacing `--muted-dim` with `--muted` at those call sites, which passes at 6.40:1 on the card surface.

### Non-text contrast

| Element | Effective colour | Ratio vs `--bg` | Required | Result |
|---|---|---:|---:|---|
| `--border` (0.13 alpha) | `#181e29` | 1.20 | 3.0 | Fail (ACC-07) |
| `--border-strong` (0.24 alpha) | `#27313f` | 1.52 | 3.0 | Fail (ACC-07) |
| Range track (unfilled, 0.16 alpha) | ~`#1c2432` | ~1.3 | 3.0 | Fail (ACC-07) |
| Focus ring `--accent` | `#5eead4` | 13.54 | 3.0 | Pass |
| Slider thumb | `#ccfff7` | ~17 | 3.0 | Pass |

**ACC-07 in detail.** SC 1.4.11 requires 3:1 for the visual boundary of interactive components where the boundary is the only way to identify them. Buttons, chips and dropzones use `border: 1px solid var(--border)`, which is 1.20:1 — effectively invisible as a boundary. The compensating factor is the strong focus indicator (13.54:1) and the fact that buttons also carry text and background tints. The honest summary is: **focus visibility is good; boundary visibility is weak.**

---

## Text size, zoom and reflow

The stylesheet uses `rem` for essentially all typography — a single `px` font-size exists in the file (15 px) — so browser zoom scales text and layout together. This is a genuine strength for SC 1.4.4 and SC 1.4.10.

The risk is the opposite of a fixed-size problem (ACC-08): many helper texts are already very small at 100% zoom.

| Selector | Size | ≈ px at 16 px root | Note |
|---|---:|---:|---|
| `.scanner__readout .mono` | 0.52 rem | 8.3 | Decorative HUD text |
| `.scanner__foot small` | 0.57 rem | 9.1 | HUD |
| `.brand__text small` | 0.62 rem | 9.9 | Branding |
| `.topbar__eyebrow` | 0.62 rem | 9.9 | Section label |
| `.control-hint` | 0.66 rem | 10.6 | Instruction text under controls |
| `.dropzone small` | 0.68 rem | 10.9 | Format/size help |
| `.hint` / `.mini-label` | 0.65 / 0.68 rem | 10.4 / 10.9 | Metadata |

There is no WCAG minimum font size, so this is not a criterion failure. It is a **low-vision usability risk**: a user at 200% zoom is fine, but a user with 20/20 vision on a 13-inch laptop may struggle with the 8–11 px helper text — the same text that ACC-01 already renders too faint. Fixing ACC-01 and raising the smallest helper sizes to a 0.75 rem floor would address both findings together.

Reflow was exercised at 390, 820, 1024 and 1440 px with no page-level horizontal overflow, which is consistent with SC 1.4.10 at 320 CSS pixels for these views, although a formal 320 px reflow test was not executed.

---

## Keyboard operation

**Conforms in source (ACC-10) and partially in behaviour.**

| Control | Keyboard support | Evidence |
|---|---|---|
| Skip link | Reachable as the first focusable element, targets `#main` | `.skip-link` is `position: fixed` and revealed on focus; `<main id="main" tabindex="-1">` |
| Global focus ring | Visible on every focusable element | `:focus-visible { outline: 2px solid var(--accent); outline-offset: 3px; }` |
| Image-source tabs | Arrow Left/Right move between tabs, roving `tabindex`, `aria-selected` updated | `inspect.js` binds `ArrowLeft`/`ArrowRight` and sets `button.tabIndex = 0/-1` |
| Command palette | `⌘K`/`Ctrl+K` opens; Up/Down navigate; Enter activates; Escape closes; focus restored to the trigger | `palette.js` stores `previousFocus` and restores it on close |
| Camera, lightbox and palette dialogs | Native `<dialog>` + `showModal()` gives a focus trap and Escape-to-close | `camera.js`, `inspect.js`, `palette.js` |
| Drawer | Escape closes the sidebar; scrim click closes it | `app.js` keydown handler |
| Forms | Native `input`, `select`, `button` elements, so standard activation applies | — |

**Gaps.** The tabs implement the APG keyboard model but not the full ARIA relationship (ACC-06). The drawer's focus is not returned to the menu button on close (ACC-05).

---

## Focus management

**ACC-05.** `route()` in `app.js` toggles `view.hidden` for each view, updates `aria-current` on the nav items and rewrites the document title, but it never moves focus. A keyboard or screen-reader user who activates a navigation link stays focused on that link, which is inside the sidebar and is now visually and semantically secondary to the newly revealed view. The new view is never announced as the active context.

The same applies to the mobile drawer: `setSidebar(false)` toggles a CSS class and sets `aria-expanded`, but does not restore focus to `#menu-btn`.

`<main id="main" tabindex="-1">` already has the `tabindex="-1"` that the fix requires, so the remediation is small:

1. In `route()`, after revealing a view, call `view.focus()` (or `$("#main").focus()`), and keep the hash change in the address bar.
2. In `setSidebar(false)`, return focus to `#menu-btn` when the drawer was open.

Note the deliberate trade-off: moving focus on every route change can be disorienting for users who navigate by hash manually. The standard mitigation is to move focus only for user-initiated navigation, which the router can distinguish by tracking the last navigation trigger.

---

## Names, roles and values

**ACC-02 — palette trigger.** The button contains an SVG icon, a `<span>Jump to…</span>` and a `<kbd>⌘K</kbd>`. At the `max-width: 820px` breakpoint the rules

```css
.palette-trigger span { display: none; }
.palette-trigger kbd  { display: none; }
```

remove every text node. The `<use href="#i-search"/>` SVG contributes no accessible name, and the button has no `aria-label`. The result is a focusable, clickable button with **no accessible name** on exactly the narrow viewports where a mobile user is most likely to need it.

**ACC-03 — palette search input.** `#palette-input` has a `placeholder` ("Jump to a page or action…") but no `<label>` and no `aria-label`. A placeholder is not an accessible name; it disappears on input and is not reliably announced. The surrounding `<dialog>` does have `aria-label="Command palette"`, but the input itself needs a name — the pattern is already close to correct because the input drives `aria-activedescendant` against `role="option"` items in a `role="listbox"`.

**Conforms.** The GitHub link uses `aria-label="Open GitHub repository"`. The menu button uses `aria-label="Open navigation"`. The image-source tablist is `role="tablist"` with `aria-label="Image source"`, and each tab carries `role="tab"` and `aria-selected`.

**ACC-06 — tab panels.** The tabs are not associated with their panels. There is no `aria-controls` on the `role="tab"` elements and no `role="tabpanel"`/`aria-labelledby` on the corresponding regions, so a screen reader announcing the selected tab cannot say which region it controls. Adding the two attributes per tab and a `role="tabpanel"` to the three source regions completes the pattern.

**ACC-09 — no visually-hidden utility.** There is no `sr-only`/`visually-hidden` class in the stylesheet. Anything removed from the visual design with `display: none` is simultaneously removed from the accessibility tree. That is correct behaviour for decorative content, but it means the only way to add visually hidden context is an `aria-label` on an existing element. The repository contains just two `<label>` elements and 25 `aria-label` attributes, so most controls are named by ARIA rather than by associated labels.

---

## Status messages

**Conforms in source (ACC-10).** Four `aria-live="polite"` regions announce changes without stealing focus:

| Region | Element | Announces |
|---|---|---|
| Sample grid | `#sample-grid` | Dataset sample availability |
| Image result | `#image-result-card` | Detection results and counts |
| Video status | `#video-status-card` | Job stage and progress |
| Toasts | `#toasts` | Transient success/error messages |

`aria-atomic="false"` on the toast stack lets several messages coexist.

**Gap.** The model status chip is **not** a live region, and at ≤620 px its text is removed:

```css
.status-chip span:last-child { display: none; }
```

Only the coloured `status-dot` remains (ACC-04). Model availability — the single most important piece of runtime state in the application — is then conveyed by colour alone, which fails SC 1.4.1 and leaves screen-reader users with no information at all. The chip's `data-state` attribute (`loading`, `ready`, `error`, `unavailable`) is present in the DOM and could be exposed with a visually-hidden text node or an `aria-label` that changes with state.

---

## Motion and animation

**Conforms in source (ACC-10).** The reduced-motion block is thorough:

```css
@media (prefers-reduced-motion: reduce) {
  html { scroll-behavior: auto; }
  *, *::before, *::after {
    animation-duration: 0.001ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.001ms !important;
  }
  .reveal { opacity: 1; transform: none; }
  .scanbeam { top: 50%; }
  .hero h2 em, .btn--primary { text-shadow: none; }
}
```

It neutralises the reveal-on-scroll observer, the animated scan beam, smooth scrolling and every transition, rather than only shortening one animation. `html { scroll-behavior: auto; }` also covers the router's `window.scrollTo`.

---

## Forms, labels and help text

- Slider controls use `input[type="range"]` with a custom track and thumb; the filled portion is drawn with a `--pct` custom property updated from the value, so the visual state matches the value.
- Values are shown numerically next to the sliders, and the hint text beneath the controls explains the range.
- The confidence and IoU inputs are clamped to the server's accepted ranges in `store.js` before being sent; the server validates independently and returns 422, which the UI surfaces. The client clamp is a usability measure, not an accessibility control, and the server-side range is what actually protects the API.
- The file input is visually hidden and driven by a styled label, so the control is still keyboard-operable. The dropzone is a `div` with click and keyboard handlers rather than a native button, which is the weakest part of the form markup: it relies on the hidden input for the real semantics.

---

## What was not tested

| Area | Status | Consequence |
|---|---|---|
| Automated scan (axe, Lighthouse) | **NOT RUN** | No independent confirmation of the manual findings |
| Screen-reader session (NVDA, JAWS, VoiceOver) | **NOT RUN** | Live-region and name announcements are unverified in practice |
| 200% / 400% text zoom | **NOT RUN** | `rem`-based sizing suggests it will hold, but it is untested |
| 320 CSS-pixel reflow (SC 1.4.10) | **NOT RUN** | Checked at 390 px only, which is close but not equivalent |
| Windows High Contrast / `forced-colors` | **NOT RUN** | Dark-glass styling is likely to lose borders and gradients |
| Testing with disabled users | **NOT PERFORMED** | No usability validation of the priority fixes |
| Colour-blind simulation | **NOT RUN** | Severity colours are accompanied by text labels, but not formally checked |
| Braille / tactile output | **N/A** | Visual inspection tool |

The absence of a failing test is not a pass. The findings above are a **review of the source**, and a reviewer with assistive technology may find issues that static reading cannot reveal — particularly around focus order, live-region verbosity and the announcement of detection results, which are announced as a whole region rather than per detection.

---

## Prioritised remediation

Ordered by user impact ÷ effort. Effort is a rough engineering estimate, not a schedule commitment.

| # | Fix | Findings | Effort | Impact |
|---|---|---|---|---|
| 1 | Raise `--muted-dim` to ≥4.5:1 (one token) | ACC-01 | Very low (1 line) | High — repairs 35 text styles at once |
| 2 | Add `aria-label` to `#palette-open` and `#palette-input` | ACC-02, ACC-03 | Very low (2 attributes) | High — restores names on mobile |
| 3 | Keep the status chip's text in the accessibility tree at ≤620 px (e.g. `clip` instead of `display:none`) and expose state via `aria-label` | ACC-04 | Low | High — model state becomes perceivable |
| 4 | Move focus to the revealed view on route change; restore focus to `#menu-btn` when the drawer closes | ACC-05 | Low | High — fixes focus order for keyboard and screen-reader users |
| 5 | Add `aria-controls` + `role="tabpanel"` to the image-source tabs | ACC-06 | Low | Medium |
| 6 | Introduce an `sr-only` utility and use it for the chip text instead of `display:none` | ACC-09, ACC-04 | Low | Medium |
| 7 | Strengthen `--border` to meet 3:1 for interactive boundaries, or add a filled/inset indicator to controls | ACC-07 | Medium | Medium |
| 8 | Raise the smallest helper text to a 0.75 rem floor | ACC-08 | Low | Medium |
| 9 | Convert the dropzone `div` into a real `button` (or give it `role="button"` + `tabindex`) | Form semantics | Low | Medium |
| 10 | Announce per-detection results in a dedicated live region rather than the whole result card | Status messages | Medium | Medium |
| 11 | Run axe + a screen-reader pass and record the results in the [manual test log](../09-testing-and-results/manual-test-log.md) | Evidence | Medium | High — converts review into evidence |

---

## How to verify conformance later

1. Run an automated scan (axe or Lighthouse) on `/`, `/#/inspect`, `/#/video` and `/#/system`, and save the JSON output as evidence.
2. Complete a manual keyboard-only pass: `Tab` from a cold load, activate every nav item, open and close the palette, camera dialog and lightbox, and operate the sliders and dropzone.
3. Complete a screen-reader pass with at least one desktop and one mobile reader, checking the status chip, the result card and the video progress region.
4. Repeat the contrast table against the **rendered** colours, not the declared tokens, and record any difference.
5. Test at 200% and 400% zoom, and at a 320 CSS-pixel viewport width.
6. Test with `forced-colors: active` and with reduced motion enabled.
7. Record each check in the [manual test log](../09-testing-and-results/manual-test-log.md) with a status of PASS, FAIL, BLOCKED or NOT RUN, and promote a conformance statement only when the evidence supports it.

Until then, the correct statement is: *the dashboard was reviewed against WCAG 2.2 AA at the source level, ten findings were identified, and no formal conformance testing has been performed.*

---

## Related pages

- [Security and quality index](README.md) — scope, severity definitions and the full finding list.
- [Security and privacy](security-and-privacy.md) — trust boundaries and the controls that are actually implemented.
- [Performance](performance.md) — measured timings and the scalability boundaries of a local single-user tool.
- [Frontend architecture](../04-architecture/frontend-architecture.md) — module structure, DOM-safety patterns and the CSP.
- [Manual test log](../09-testing-and-results/manual-test-log.md) — where the verification steps above are recorded once executed.
- [Limitations](../03-project-documentation/limitations.md) — project-level scope limits.
