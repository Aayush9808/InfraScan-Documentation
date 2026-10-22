# Frontend and integration questions

Twenty questions for Khushi and anyone demonstrating the dashboard. Answers should be tied to the actual modules under `web/assets/js/` and the API routes they call.

---

### Q1. What is the frontend stack?

It is a **zero-build SPA**: `web/index.html`, handcrafted CSS in `web/assets/css/app.css`, and native JavaScript ES modules in `web/assets/js/`. There is no React, Vue, Node build step or separate frontend server. FastAPI serves the files and the API from the same origin.

### Q2. How is the SPA organised?

`app.js` boots the app, manages hash routing, status polling, navigation and shared UI. Feature modules are separated: `inspect.js` (image workflow), `video.js` (job workflow), `camera.js` (webcam dialog), `system.js` (runtime page), `palette.js` (command palette), `api.js` (fetch wrapper), `store.js` (state/events) and `ui.js` (DOM/formatting helpers). CSS and fonts are local assets.

### Q3. Why hash routing?

A hash route (`#/inspect`) is resolved entirely in the browser. The server does not need a catch-all route or rewrite rule for every view, which keeps deployment simple and works when the SPA is served as static files. The trade-off is less elegant URLs and no server-side rendering.

### Q4. How does the app load?

The browser requests `/`, receives `index.html`, then loads the module entrypoint and CSS. `app.js` initialises the store, binds navigation, reads the initial hash, calls `/api/status`, and renders the model/runtime state. If the API is unreachable, it switches to an explicit offline state rather than showing stale “ready” data.

### Q5. How does the API wrapper work?

`api.js` wraps `fetch`, sets JSON headers where appropriate, parses error payloads, and throws a typed `ApiError` with status/detail. Feature modules catch that error and show an actionable message. This avoids scattering raw `fetch` calls and inconsistent error handling across the UI.

### Q6. How does image inspection work?

`inspect.js` supports upload, dataset sample and webcam capture. It validates the selection, shows a preview, sends multipart form data to `/api/detect/image` with `conf` and `iou`, disables the run button while waiting, then renders the annotated image, detection table, counts, severity badges, coverage and latency. It also supports an original/annotated toggle and CSV download.

### Q7. How are dataset samples loaded?

The UI calls `/api/samples`; the API lists up to 24 test images from `dataset/images/test` and returns names, URLs and sizes. The SPA renders tiles and converts a selected sample into a `File`/blob for the same image request path. If the dataset directory is absent, the picker shows an unavailable state; upload and webcam remain usable.

### Q8. How does webcam capture work?

`camera.js` uses `navigator.mediaDevices.getUserMedia` with video constraints, draws a frame to a canvas, converts it to a blob, and hands it to the image workflow. It handles permission denial, missing devices and insecure-context errors. The browser test used Chrome's synthetic media device; a physical camera was **not** independently verified.

### Q9. How does video analysis work?

`video.js` uploads a file to `/api/detect/video`, receives HTTP 202 with a job ID, and starts polling `/api/jobs/{id}`. It renders stage, frame progress, percentage and ETA. On completion it sets the annotated MP4 URL and enables telemetry/video downloads. On failure it shows the server's error and a retry path.

### Q10. Why polling instead of WebSockets/SSE?

A video job is long-running but progress updates are infrequent. Polling is simpler, survives ordinary proxy/browser conditions and needs no extra protocol. The trade-off is a small amount of repeated traffic. For a local single-user tool, that is a reasonable engineering choice; a production multi-user system might use SSE or a queue.

### Q11. How is state shared between modules?

`store.js` holds a small observable state: model status, settings, samples, session counters, latency and offline state. Modules subscribe with `on()` and update through setters/`emit()`. It is intentionally not a large state framework. The UI reads from this state and re-renders targeted elements rather than rebuilding the whole page.

### Q12. How are settings persisted?

Confidence, IoU and similar preferences are stored in `localStorage`. On load, values are clamped to the server's allowed ranges so a corrupted or malicious stored value cannot bypass validation. The API still validates independently; client-side clamps are a UX measure, not a security boundary.

### Q13. How does the command palette work?

`palette.js` opens on `⌘K`/`Ctrl+K`, lists navigation and actions, filters by keyboard input, supports arrow/Enter/Escape, and restores focus when closed. It is a convenience layer over the same routes/actions, not a separate state. Keyboard handling must not trap focus or interfere with form inputs.

### Q14. How is the UI kept accessible?

The HTML uses landmarks, labels, buttons with accessible names, focus-visible styles, dialog semantics for the webcam/lightbox, `aria-live` regions for status/errors, and reduced-motion support. Keyboard and responsive checks were manual; there is no formal WCAG certification or screen-reader audit. Gaps are documented in the accessibility chapter.

### Q15. How does the layout respond?

The layout uses CSS grid/flex and breakpoints; verification covered **390, 820, 1024 and 1440 px** with no page-level horizontal overflow. At narrow widths the navigation collapses, panels stack and tables become scrollable or simplify. Responsive checks are not the same as a full cross-browser/device matrix.

### Q16. How is XSS prevented?

API-provided strings are inserted with text nodes or escaped through UI helpers, not concatenated into `innerHTML` as raw markup. The CSP restricts script sources. This is defence in depth, but it is not a formal penetration test. Filenames and error messages are still treated as untrusted.

### Q17. Why are fonts local?

The fonts (Inter and Space Grotesk) are bundled under `web/assets/fonts/` with their OFL licence files. This avoids a third-party font request, keeps the app usable offline and makes the CSP simpler. It also means the repository carries the font licence obligations.

### Q18. What happens when the backend is offline?

`app.js` catches status failures and calls the offline state. The UI disables actions that need the API, shows a clear “local API unreachable” message and keeps the shell usable. It must not keep showing a green “ready” chip from cached data. Recovery occurs on the next successful status poll or manual reload.

### Q19. How are errors surfaced?

The API returns structured `detail` messages. `api.js` converts them to `ApiError`; feature modules display a specific message (invalid image, too large, unsupported video, model unavailable, job not finished). The UI distinguishes validation, server and network failures rather than showing one generic toast.

### Q20. What is the biggest frontend limitation?

It is a single-user local dashboard with no authentication, no server-side rendering, no automated browser test suite and no formal accessibility certification. State is in memory/localStorage, and video jobs do not persist across a server restart. Those are documented scope boundaries, not hidden assumptions.

---

## Cross-examination prompts

- “Why not use React?” → The app is small and the zero-build constraint is valuable for an academic/local tool; React would add a toolchain and dependency surface without a current requirement.
- “How do you know the UI matches the API?” → `api.js` is the single wrapper; routes are documented in `backend-and-api.md`; manual integration checks are recorded in the test log.
- “What if a user uploads a 3 GB video?” → The API streams to disk and enforces the 2 GB limit with HTTP 413; the UI must show the error and clean up.
- “Does the webcam work on every browser?” → No claim; it needs HTTPS or localhost, permission and a device. Only a synthetic Chrome test was recorded.
- “Is the dashboard mobile-first?” → It is responsive and checked at 390 px, but the primary use case is a desktop inspection workstation; mobile is supported for layout, not validated as a full field workflow.

---

[Viva index](README.md) · [Frontend architecture](../04-architecture/frontend-architecture.md) · [Application usage](../06-setup-and-execution/application-usage.md) · [Documentation homepage](../README.md)
