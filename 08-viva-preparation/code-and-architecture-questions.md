# Code and architecture questions

Twenty-five questions that move from the request path into the implementation. Use these to practise naming the exact file, function and failure mode.

---

### Q1. What is the application entrypoint?

`app.py` is a three-line ASGI entrypoint: it imports `app` from `infrascan.api`, so Uvicorn or `run_app.py` can serve it. `run_app.py` is the convenience launcher: it parses host/port/weights/reload flags, sets `INFRASCAN_WEIGHTS` when supplied, and starts Uvicorn. The dashboard and API are one process.

### Q2. What happens during FastAPI startup?

`infrascan/api.py` defines a lifespan context. It cleans old result/job directories, calls `inference.start_model_load()` in a background thread, and yields control so the server can accept requests immediately. The model load is deliberately asynchronous so the UI can render a “loading” state instead of blocking startup.

### Q3. How does the model state machine work?

`infrascan/inference.py` tracks states such as `starting`, `loading`, `ready`, `unavailable` and `error`. `model_status()` exposes the state, checkpoint path, device, classes and error message. The dashboard reads `/api/status`; if the state is `unavailable`, detection routes return an actionable `503`.

### Q4. Why is the model loaded lazily in a thread?

Loading a PyTorch checkpoint can take seconds. Doing it during a request would make the first user wait and could time out. A background thread loads it while the SPA renders; a lock protects the state and load. This is a UX and concurrency decision, not a claim that model loading is parallelized.

### Q5. Walk through an image request.

`POST /api/detect/image` accepts multipart `file`, `conf` and `iou`. `_validate_thresholds()` checks the ranges, `config.checkpoint_available()` checks the file, `_read_upload()` enforces the 25 MB limit, and `run_in_threadpool(inference.predict_image, ...)` moves CPU/GPU work off the event loop. The result contains detections, summary, severity counts, latency, model metadata and a URL for the annotated JPEG.

### Q6. How are uploads validated?

`_read_upload()` reads in chunks, aborts with HTTP 413 when the byte count exceeds the limit, and closes the file. The image route then decodes the bytes with OpenCV; a decode failure raises `InvalidImageError`, mapped to HTTP 400. There is no extension whitelist on image upload because the decoder is the real validation; the UI advertises the supported formats.

### Q7. How are confidence and IoU validated?

`_validate_thresholds()` enforces confidence 0.05–1.0 and IoU 0.10–1.0. Out-of-range values raise HTTP 422 with a message naming the allowed range. Defaults are 0.25 and 0.45. The same values are passed to Ultralytics prediction and recorded in the result.

### Q8. What does an image response contain?

The response includes `detections` (class, confidence, bbox, coverage percentage, severity), `summary` counts, `severity_counts`, image width/height, latency, model metadata, `annotated_url`, and the original filename added by the route. The SPA renders the table and annotated image from this response; it does not infer severity in JavaScript.

### Q9. How is path traversal prevented?

For sample images, the route rejects any `name` where `Path(name).name != name` or the extension is not allowed, then checks that the resolved file exists. Result images require a plain filename ending in `.jpg` and check the results directory. The job outputs use a server-generated job ID and a fixed filename, not a user path. These checks keep user input from becoming an arbitrary filesystem path.

### Q10. How do video jobs work?

`POST /api/detect/video` validates thresholds, extension and checkpoint, creates an in-memory job with a UUID, saves the upload under the job directory, and starts a background thread. It returns HTTP 202 with the job object. The browser polls `/api/jobs/{id}` for status/progress, then requests `/video`, `/telemetry.csv` or `/telemetry.json` when complete.

### Q11. Where is video inference executed?

`infrascan/jobs.py` calls `detect.run_detection()` in a worker thread, wrapped by a module-level `_worker_lock`. The lock serialises video jobs so multiple long clips do not run concurrently and exhaust memory/CPU. The HTTP event loop remains responsive because the work is outside it.

### Q12. How is progress tracked?

`update_progress()` updates a job dictionary under `_jobs_lock` with frame, total frames, fraction, stage and ETA. The detection loop calls a progress callback. The SPA polls at an interval and renders a progress bar/ETA. Jobs are in memory, so they do not survive a process restart; results on disk are cleaned at startup after the TTL.

### Q13. What happens if the video encoder cannot use H.264?

The detection writer tries the `avc1` fourcc first and falls back to `mp4v` if the local OpenCV build cannot initialize it. The output is still an MP4, but browser playback depends on the codec support. The verified test used H.264 and Chrome reported `readyState 4`; the fallback is a compatibility path, not a guarantee for every browser.

### Q14. What does `severity.py` do?

It defines the two thresholds and `classify_severity(coverage_pct)`. The function is deliberately tiny and pure, so the dashboard, `detect.py` and `test_camera.py` all import the same policy. The API exposes the policy in `/api/status`, and the SPA displays it. No client-side reimplementation should drift from this module.

### Q15. How is the annotation produced?

`infrascan/inference.py` uses OpenCV to draw the box and label on a copy of the decoded image, then encodes a JPEG. `detect.py` uses a similar drawing path plus an optional HUD for video/camera. The annotated image is evidence, not the model output itself; the raw detections remain in JSON/CSV.

### Q16. How does device selection work?

The inference service chooses CUDA when available, otherwise Apple MPS, otherwise CPU. If prediction fails on an accelerator, it can retry on CPU and record the actual device. The verified evaluation used MPS. Do not claim a CUDA result without running it; the device string in `/api/status` is the evidence for the current process.

### Q17. What is the CLI fallback issue?

The dashboard refuses to substitute a checkpoint when `weights/best.pt` is missing. The standalone CLI scripts retain the historical Ultralytics fallback to `yolo11s.pt` when `--weights` is omitted. That is documented in the main README; for reported results, always pass `--weights weights/best.pt` and verify the hash. The difference is intentional in the current code and must not be described as “the app silently downloads a model.”

### Q18. How is the frontend structured?

`web/index.html` defines the shell, views and inline SVG icons. `web/assets/js/app.js` bootstraps the app, routing, status polling and shared UI. Feature modules are split by responsibility: `inspect.js`, `video.js`, `camera.js`, `system.js`, `palette.js`, `api.js`, `store.js` and `ui.js`. CSS and local fonts live under `web/assets/`.

### Q19. How does routing work?

The SPA uses hash-based navigation (`#/overview`, `#/inspect`, etc.), so it works when served as static files without server-side route rewrites. `app.js` listens for hash changes, shows the matching view, updates navigation state and focuses the view. Unknown hashes fall back to a known route.

### Q20. How does state management work?

`store.js` holds a small observable state object for status, settings, samples, session counts and latency. `on()` registers handlers and `emit()` notifies them. It is not Redux; the app is small enough for a hand-written store. Settings persist in `localStorage` with clamped values, so invalid stored values cannot break the UI.

### Q21. How does the SPA prevent XSS?

`ui.js` provides escaping helpers and the result tables build DOM/text nodes rather than injecting raw API strings as HTML. The API also sets a restrictive Content-Security-Policy. This is defence in depth: even a malicious filename or error string should not become executable markup. It is not a formal security audit.

### Q22. How are static files served?

FastAPI serves `web/index.html` at `/`, the SPA assets under `/assets`, and the favicon. The API and SPA share an origin, so the SPA can use relative `/api/...` URLs and there is no CORS configuration to maintain. Cache headers are explicit: the index is `no-cache`, static assets can be cached, API responses are `no-store`.

### Q23. Which security headers are set?

Middleware adds `X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY`, `Referrer-Policy: no-referrer`, a restrictive CSP, and `Cache-Control: no-store` on API paths. The CSP permits local scripts/styles/fonts and the inline SVG/icon setup; any new third-party script would require a deliberate policy change. There is still no authentication.

### Q24. What are the main HTTP error codes?

`400` invalid image/decode; `404` missing sample/result/job/output; `409` job not finished; `413` upload too large; `415` unsupported video extension; `422` invalid thresholds; `503` missing model, model load failure or timeout; `500` unexpected upload failure. The SPA maps these into actionable messages rather than a generic “something went wrong.”

### Q25. Why is there no database?

The app is a local, single-user academic tool. Jobs and results are ephemeral; in-memory jobs plus on-disk result directories are sufficient. A database would add schema, migrations and operational burden without solving a current requirement. The trade-off is no persistence across restarts and no multi-user isolation, which is explicitly documented as a limitation.

### Q26. Does `run_app.py --weights` actually change the checkpoint?

Not reliably, and this is a real ordering issue in the launcher rather than a hypothetical one. `run_app.py` imports `infrascan.config` near the top of the file to obtain `PROJECT_ROOT`, and `config.WEIGHTS_PATH` is resolved when that module is imported — before `main()` parses the command-line flags. In a normal launch Uvicorn then imports `app:app` in the same process, so the already-cached configuration module is reused and the path is unchanged. Two working alternatives: set the environment variable before launching (`INFRASCAN_WEIGHTS=/path/to/model.pt python run_app.py`), or pass `--reload`, whose child process re-imports the configuration with the variable already inherited from the parent.

I state this as source-derived reasoning, not as a browser-verified test: it follows from the import order visible in `run_app.py` against `infrascan/config.py`, and it is recorded in [`../06-setup-and-execution/configuration.md`](../06-setup-and-execution/configuration.md). The practical consequence is that a demo which silently ignored `--weights` would still be showing the default checkpoint while the operator believes otherwise — which is exactly why the System page prints the resolved path, and why checking that path is part of the startup routine.

---

## Cross-examination prompts

- “What happens if two image requests arrive at once?” → FastAPI runs them in a threadpool; `_predict_lock` serialises access to the single Ultralytics model.
- “What happens if the process dies during a video job?” → The in-memory job is lost; the uploaded file/results remain until cleanup; the UI shows a failed/unknown job after restart. There is no durable queue.
- “Could a filename escape the results directory?” → The sample/result routes reject path separators and require allowed suffixes; job outputs use server-generated paths.
- “Why not use Celery/Redis?” → Overkill for a local single-process academic tool; the added services would complicate setup and are not needed for the verified scope.
- “What is the most fragile part?” → Long video jobs and codec/browser compatibility, because they depend on the local OpenCV build, file size and hardware.

---

[Viva index](README.md) · [Backend and API](../04-architecture/backend-and-api.md) · [Code walkthrough](../03-project-documentation/code-walkthrough.md) · [Documentation homepage](../README.md)
