# Frontend architecture

- [Why there is no build step](#why-there-is-no-build-step)
- [SPA shell and routing](#spa-shell-and-routing)
- [Module responsibilities](#module-responsibilities)
- [State and event model](#state-and-event-model)
- [Polling and refresh behaviour](#polling-and-refresh-behaviour)
- [XSS escaping and DOM safety](#xss-escaping-and-dom-safety)
- [CSP compatibility](#csp-compatibility)
- [Responsive layout](#responsive-layout)
- [Error and offline states](#error-and-offline-states)
- [Accessibility summary](#accessibility-summary)
- [Browser support](#browser-support)

## Why there is no build step

The frontend is hand-written HTML, CSS and native ES modules under `web/`. FastAPI serves it directly:

```text
web/
├── index.html
└── assets/
    ├── css/app.css
    ├── js/
    │   ├── app.js
    │   ├── api.js
    │   ├── store.js
    │   ├── ui.js
    │   ├── inspect.js
    │   ├── video.js
    │   ├── camera.js
    │   ├── system.js
    │   └── palette.js
    ├── fonts/                 # Inter + Space Grotesk, OFL, local woff2
    └── brand/favicon.svg
```

The browser loads `app.js` with `<script type="module" src="/assets/js/app.js">`; its `import` statements fetch the other modules natively. There is no npm, bundler, transpiler, CSS preprocessor or separate dev server.

**Why this is appropriate here:**

| Benefit | Consequence for this project |
|---|---|
| Zero install for reviewers | `pip install -r requirements.txt` is the only dependency step; no Node version drift |
| Same origin by construction | No CORS, proxy or environment-specific API base URL |
| Small, auditable surface | Every module is readable in the repository; no generated bundle |
| Simple deployment | The repository's `web/` directory is the deployment |
| Fast local iteration | Edit a file, reload the browser; no build watch process |

**Trade-offs, stated honestly:**

- No tree-shaking, minification or automatic cache busting. Asset URLs are not content-hashed; after editing CSS/JS, use a hard reload if the browser reuses a cached file.
- No TypeScript or framework reactivity. DOM updates are explicit functions, and state changes are coordinated by a tiny event emitter.
- Browser support is limited to engines that understand ES modules, native `<dialog>`, `:has()`, `String.prototype.replaceAll`, `Intl.NumberFormat` and `matchMedia`. There are no polyfills or transpiled fallbacks.
- There is no component test harness or frontend test suite in the repository. The documented evidence is manual browser testing (Chrome, including synthetic camera media) plus source inspection.

## SPA shell and routing

`web/index.html` contains the entire shell and all four view sections in the initial HTML:

| View | `data-view` | Main content |
|---|---|---|
| Overview | `overview` | Hero, runtime strip, capability cards, pipeline explanation, severity scale |
| Image inspection | `inspect` | Source picker (upload/dataset/webcam), result card, detection table, downloads |
| Video analysis | `video` | Video upload, job progress, annotated result and telemetry downloads |
| System | `system` | Checkpoint/runtime metadata, severity policy, API list, CLI reference, reload |

Only one view is visible at a time. The others carry the `hidden` attribute. There is no server-side route per view and no client-side template compilation: the markup is already in the document, and the modules toggle `hidden`, text content, classes and small `innerHTML` fragments.

### Hash router

`app.js` implements a deliberately small router:

```js
const ROUTES = {
  overview: { title: "Overview", eyebrow: "Workspace" },
  inspect:  { title: "Image inspection", eyebrow: "Image workspace" },
  video:    { title: "Video analysis", eyebrow: "Video workspace" },
  system:   { title: "System", eyebrow: "Runtime" },
};
```

- The route is read from `location.hash`, stripped of `#/` and any query string.
- An unknown or empty route resolves to `overview`.
- `route()` normalises the URL with `history.replaceState(null, "", "#/" + name)`. It does not push a new history entry for an unknown hash.
- Navigation uses ordinary `<a href="#/inspect">` links, so browser back/forward works between visited hashes.
- The active nav item gets `aria-current="page"`; the page `<h1>` and document title are updated.
- The window scrolls to the top and the reveal observer is reattached for the visible view.

Because routing is hash-based, the server only needs to serve `/`. A direct request to `/inspect` is not a supported URL; the canonical form is `/#/inspect`.

### Reveal animation

`IntersectionObserver` adds `.is-visible` to `.reveal` elements when they enter the viewport (threshold `0.06`, bottom root margin `-40px`). If `IntersectionObserver` is unavailable, every `.reveal` is made visible immediately. The CSS `prefers-reduced-motion` block disables the transition and forces `.reveal` to its visible state.

## Module responsibilities

| Module | Owns | Key behaviours |
|---|---|---|
| `app.js` | Boot, router, status polling, sidebar, settings sync, command definitions | Imports all feature modules; schedules `/api/status`; renders the model chip and overview values; builds the command palette list |
| `api.js` | HTTP boundary | `ApiError` with status; network failures become status `0`; parses `{detail}` from error JSON; `FormData` for image/video; `encodeURIComponent` for job IDs |
| `store.js` | Single shared state object and event bus | Loads/clamps `conf` and `iou` from `localStorage`; emits `status`, `offline`, `settings`, `samples`, `session`, `latency`; increments session counters |
| `ui.js` | Reusable rendering and browser helpers | `$`/`$$`, `escapeHtml`, formatters, severity chips, severity summary bar, toast stack, clipboard, CSV generation, animated numbers, button busy state |
| `inspect.js` | Image workspace | Source tabs, drag/drop, sample selection, webcam hand-off, threshold labels, run/retry, annotated/original toggle, table rendering, CSV download, lightbox |
| `video.js` | Video workspace | File selection and object-URL preview, upload, job polling every 850 ms, progress/ETA, result metrics, downloads, failure state |
| `camera.js` | Webcam modal | `getUserMedia`, preview, single-frame canvas capture as JPEG, track cleanup on close |
| `system.js` | System view | Checkpoint/runtime fields, severity policy table, reload button, copy-to-clipboard for CLI commands |
| `palette.js` | Command palette | Native modal dialog, fuzzy-ish token matching, keyboard selection, focus restoration, `⌘K`/`Ctrl+K` |

### Boot order

`app.js` `init()` runs after `DOMContentLoaded` (or immediately if the document is already parsed):

1. `initInspect()` — wires image UI, camera, samples and status listeners.
2. `initVideo()` — wires video UI and status listeners.
3. `initSystem()` — wires reload and copy buttons.
4. `initPalette(buildCommands)` — wires the command dialog.
5. `initSettings()` — wires the two range inputs and reset.
6. `initSidebar()` — mobile drawer and Escape handling.
7. `initRouter()` — installs the reveal observer, `hashchange` listener and initial route.
8. Subscribes to `settings`, `session` and `latency`; adds `infrascan:refresh-status`, `visibilitychange`, then calls `refreshStatus()` and `loadSamples()`.

The initial UI therefore shows "Connecting…" until the first status request resolves. `state.apiOnline` starts as `true`, but the run buttons remain disabled until `state.status.model.state === "ready"`.

## State and event model

`store.js` exports one mutable `state` object. There is no framework, proxy or immutable update layer.

```js
export const state = {
  apiOnline: true,
  status: null,
  samples: null,
  settings: loadSettings(),   // {conf, iou}
  image: null,                // selected file/sample metadata + preview URL
  imageResult: null,
  imageView: "annotated",
  video: null,                // selected video file + object URL
  job: null,
  lastLatency: null,
  session: { scans: 0, videos: 0 },
};
```

| Event | Emitted by | Listeners | Effect |
|---|---|---|---|
| `status` | `setStatus()` after a successful `/api/status` | `app.js`, `inspect.js`, `video.js` (through `updateRunState`) | Re-renders runtime values; enables/disables run buttons |
| `offline` | `setOffline()` on a failed status request | `inspect.js`, `video.js` | Disables run buttons; `app.js` also calls `renderOffline()` directly |
| `settings` | `setSettings()`, `resetSettings()` | `app.js` (`syncSettings`), `inspect.js`, `video.js` | Updates slider labels and "parameters changed" notes |
| `samples` | `setSamples()` | `inspect.js` | Re-renders the dataset grid |
| `session` | `incrementSession("image"|"video")` | `app.js` | Animates the session scan count |
| `latency` | `setLatency(ms)` after image inference | `app.js` | Updates the hero latency readout |

### Persistence

Only inference settings persist, under the `localStorage` key `infrascan.settings.v2`:

```json
{ "conf": 0.25, "iou": 0.45 }
```

`loadSettings()` clamps `conf` to `[0.05, 1.0]` and `iou` to `[0.10, 1.0]`, falling back to defaults on invalid JSON or unavailable storage. `resetSettings()` removes the key and emits defaults. Session counters, selected files and job state are **not** persisted; a page reload clears them. Dataset samples are loaded once at boot and are not refreshed unless the page is reloaded.

### Why an event bus instead of direct calls

Feature modules are initialized independently and need to react to status/settings changes without importing each other. `on()` returns an unsubscribe function, though none of the feature modules currently use it because they live for the page lifetime. Listener exceptions are caught and logged so one failing subscriber does not break the rest.

## Polling and refresh behaviour

### Status polling

`app.js` uses a recursive `setTimeout`, not `setInterval`, so a slow request cannot create overlapping polls:

```js
const STATUS_INTERVAL = 15000;   // ready
const LOADING_INTERVAL = 2500;   // starting/loading
const OFFLINE_INTERVAL = 5000;   // API unreachable
```

- Only one status request is in flight. A refresh requested while polling sets `queuedRefresh`; the next run is triggered after the current one finishes.
- On success, `setStatus()` marks the API online, renders the chip/runtime values and schedules the next poll based on `model.state`.
- On failure, `setOffline()` marks the API offline, renders the offline state, shows one error toast (`offlineNotified` prevents a storm), and retries every 5 seconds.
- When the connection recovers, a "Connection restored" toast is shown once.
- `visibilitychange` triggers an immediate refresh when the tab becomes visible again, so a backgrounded tab does not display stale data indefinitely.
- A custom `infrascan:refresh-status` event lets the System reload button request an immediate refresh.

`GET /api/status` is not free: it scans `dataset/images/test` and queries installed package versions. On a local SSD the cost is small; on a dataless/iCloud directory it can stall. The frontend does not cache the response beyond the in-memory `state.status`.

### Video job polling

`video.js` polls `GET /api/jobs/{id}` every **850 ms** after the upload returns `202`:

- It stops on `completed` or `failed`.
- A network/API error stops polling and shows "Lost connection to the local server" in the error panel.
- Progress is rendered from `progress.fraction`, `frame`, `total_frames` and `stage`. ETA is calculated client-side from elapsed time since the upload response and the current fraction; it is an estimate, not a server guarantee.
- The job continues on the server if the tab is closed, but the browser loses the polling chain and the result links. The UI advises keeping the tab open.
- A `beforeunload` prompt is installed while a video is busy. Image inference does not install one; it is expected to be short.

There is no request timeout or `AbortController` in `api.js`. A request that never settles leaves the relevant `busy` flag set until the browser/network stack rejects it. This is a reliability gap for a flaky local network, but not normally visible on loopback.

## XSS escaping and DOM safety

The frontend renders a mixture of static markup and server data. The main defence is explicit escaping, with CSP as a second layer.

| Location | Data source | Handling |
|---|---|---|
| Detection table (`inspect.js`) | `id`, `bbox`, `severity`, numbers | `escapeHtml` for id/coords; `severityChip` escapes severity; numbers are formatted |
| Sample grid (`inspect.js`) | `name`, `url` | `escapeHtml` on name, URL, `data-name` and `alt` |
| Palette items (`palette.js`) | command labels/hints (local constants) | `escapeHtml` before `innerHTML` |
| Toasts (`ui.js`) | error messages, server detail | fixed markup via `innerHTML`; message inserted with `textContent` |
| Error panels | server detail | `textContent` |
| Runtime fields (`system.js`) | server status values | `textContent` via `setText` |
| Severity policy table (`system.js`) | `row.name`, `row.description` | `row.name` and description are inserted with `innerHTML` **without escaping**; these are server constants today |
| Severity summary (`ui.js`) | numeric counts | widths computed from numbers; labels from fixed strings |
| Result image `src` / download `href` | server-relative URLs | set as attributes; the server controls these URLs |

The unescaped severity-policy description is a latent weakness: it is not attacker-controlled in the current server, but if that endpoint ever returned user-influenced text, it would become an HTML-injection path. The correct fix is to build the table with `textContent` or escape the description.

CSP helps because `script-src 'self'` blocks inline event-handler attributes and external scripts, but CSP does not make `innerHTML` safe in general. The code should continue to treat every filename, telemetry string and future API field as untrusted.

## CSP compatibility

The shell is deliberately compatible with the API's restrictive policy:

| Requirement | Implementation |
|---|---|
| No inline scripts | Only `<script type="module" src="/assets/js/app.js">` |
| No CDN assets | Fonts, CSS and JS are local |
| No `eval` / `new Function` | None used |
| Inline styles allowed | The policy includes `style-src 'self' 'unsafe-inline'`; the SPA sets `style="width: …"` and `style="--w: …"` attributes |
| Images | `img-src 'self' data: blob:` covers local images, the CSS data-URI noise and object-URL previews |
| Media | `media-src 'self' blob:` covers local video responses and object-URL previews |
| Network | `connect-src 'self'` matches all relative `fetch` calls |
| Fonts | `font-src 'self'` matches the bundled woff2 files |

The one place this policy is in tension with a shipped feature is **Swagger UI** at `/api/docs`, which FastAPI normally renders with CDN assets and an inline bootstrap script. See [Backend and API](backend-and-api.md#openapi-and-interactive-docs) and [Troubleshooting](../06-setup-and-execution/troubleshooting.md#api-docs-csp). The SPA itself does not depend on that page.

## Responsive layout

`web/assets/css/app.css` defines the layout; the breakpoints are:

| Breakpoint | Main changes |
|---|---|
| `max-width: 1280px` | Tighter page/topbar padding; hero gap reduced; scanner height reduced; pipeline becomes one column |
| `max-width: 1120px` | Hero becomes one column; inspect/video grids become one column; result/video side panels stack |
| `max-width: 1024px` | Sidebar becomes an off-canvas drawer translated left; workspace loses the 274 px margin; menu and close buttons appear; system grid becomes one column |
| `max-width: 820px` | Two-column stat strip; single-column capability/split/pipeline/CLI; view header stacks; result metrics two columns; palette trigger becomes icon-only |
| `max-width: 620px` | Topbar eyebrow hidden; status-chip text hidden (dot remains); hero and card padding reduced; sample grid two columns; action rows stack; video result side becomes flex column |
| `max-width: 420px` | Further page padding reduction; palette trigger hidden; result metrics two columns; one scanner tag hidden |

The mobile sidebar is opened by `#menu-btn`, which sets `.sidebar-open` on `.app` and `aria-expanded` on the button. The scrim closes it; Escape also closes it. There is **no focus trap or `inert` treatment** for the off-canvas drawer, so keyboard focus can move into the page behind it. This is recorded in the [accessibility audit](../11-security-and-quality/accessibility.md#summary-of-findings).

`body { overflow-x: hidden; }` masks accidental horizontal overflow. The recorded layout checks at 390, 820, 1024 and 1440 px found no page-level horizontal overflow; that does not prove every internal table or long filename wraps correctly.

## Error and offline states

| State | Trigger | UI behaviour |
|---|---|---|
| Connecting | Initial load, before first `/api/status` | Chip "Connecting"; sidebar "Connecting…"; run buttons disabled; overview stats show placeholders |
| API offline | `fetch` rejection or status poll failure | Chip "API offline"; sidebar "Offline"; toast once; overview "API unreachable"; run buttons disabled with "restart run_app.py" guidance; retry every 5 s |
| Model loading | `model.state` is `starting`/`loading` | Chip "Loading model"; run buttons disabled; note "Model is loading — this can take a moment"; status poll every 2.5 s |
| Checkpoint missing | `model.state === "unavailable"` | Chip "No checkpoint"; System page shows the configured path and error; run buttons disabled; no automatic substitute |
| Model load error | `model.state === "error"` | Chip "Model error"; System error panel; run buttons disabled; reload button available |
| Invalid image | `400` from API | Image error alert with the server detail, Retry button and toast |
| Oversized upload | Client-side check or `413` | Toast or error alert; the file is not submitted if the browser check catches it |
| Video failed | Job `failed` or polling error | Video error panel with job error; no fake completion; "Process another" resets |
| Samples unavailable | `/api/samples` fails or directory absent | Empty dataset panel tells the user to add images or upload; upload path still works |
| Parameters changed after result | `lastResult.conf/iou` differ from current settings | "Parameters changed — run again to apply" note |
| Connection restored | Successful status after an offline period | "Connection restored" toast once |

The app shell is **not** a service worker/PWA. If the server is down before the first page load, the browser cannot load the SPA at all. "Offline" in the UI means "the already-loaded page cannot reach the local API".

## Accessibility summary

The shell has a stronger baseline than many student dashboards:

- `lang="en"`, a descriptive `<title>`, viewport meta and semantic `<header>`, `<aside aria-label>`, `<nav>`, `<main id="main" tabindex="-1">` and `<footer>`.
- A real skip link (`.skip-link` → `#main`).
- A global `:focus-visible` outline and native `<dialog>` elements for the palette, lightbox and camera.
- Labelled range inputs with `<output>` values; `aria-current` on navigation; `aria-live` regions for sample grid, image result card, video status and toasts; `role="alert"` error panels; `aria-busy` on busy buttons.
- Keyboard support for the image tabs (arrow keys), dropzones (Enter/Space), palette (arrows/Home/End/Enter/Escape) and native dialogs.
- `prefers-reduced-motion` handling in CSS and in the number animation.

Known gaps are documented in detail in [Accessibility](../11-security-and-quality/accessibility.md). The most important are:

- `--muted-dim` (`#5f748c`) is below 4.5:1 against the base background and worse on translucent surfaces, and it is used for small helper text.
- The palette search input has no accessible label and no combobox semantics; its `outline: none` removes the focus indicator.
- Mobile hides the status-chip text, leaving a colour-only dot with no accessible name.
- The mobile sidebar does not trap focus or make the background inert.
- Route changes update the `<h1>`/title but do not move focus or announce the new view.
- Image source tabs lack `aria-controls`/tabpanel roles.

No formal WCAG conformance, screen-reader test or automated accessibility scan has been recorded. Do not claim certification.

## Browser support

The frontend targets current evergreen browsers. It requires:

- ES modules and dynamic `import()` only implicitly through static imports;
- native `<dialog>` and `showModal()`;
- `:has()` in CSS (`body:has(dialog[open])`);
- `String.prototype.replaceAll`, `Intl.NumberFormat`, `matchMedia`;
- `URL.createObjectURL`, `FormData`, `fetch`;
- `navigator.mediaDevices.getUserMedia` for webcam capture;
- `IntersectionObserver` for reveal animation (graceful fallback if absent).

No browser-support matrix or cross-browser regression run is recorded in the repository. The verified dashboard checks used Chrome; the synthetic camera test also used Chrome's media device. Treat Firefox/Safari behaviour, older browsers and mobile browsers as `[TO BE CONFIRMED]` rather than claiming universal support.
