# Architecture

This chapter describes the accepted InfraScan implementation: a **FastAPI backend**, a **zero-build vanilla ES-module single-page application (SPA)**, and a shared **Ultralytics YOLO11n** inference pipeline, all served from the same local origin.

> **Authority:** implementation claims here are checked against the source tree and [`PROJECT_FACTS.md`](../PROJECT_FACTS.md). Where a behaviour was not exercised, the page says so instead of presenting it as verified.

## Architecture in one paragraph

`run_app.py` starts Uvicorn, which imports the FastAPI application from `infrascan/api.py` through `app.py`. At startup the application creates its runtime directories, removes expired result/job artefacts, and starts a **background model-loading thread**. The browser downloads `web/index.html` and its ES modules from the `/assets` mount; the same FastAPI process serves `/api/*`, so there is no separate frontend server, proxy, CORS configuration, or Node build. Image requests are decoded and run through a single shared YOLO model in a worker thread, with inference serialised by a lock. Video requests are uploaded to a per-job directory and processed by a background thread that runs the `detect.py` frame loop under a separate worker lock. Both paths import the same `infrascan/severity.py` policy. Results are annotated JPEG/MP4 files plus CSV/JSON telemetry under `runs/dashboard/`. The default deployment is a single-user workstation bound to `127.0.0.1:8000`, with no authentication.

## Navigate this chapter

| Page | What it answers |
|---|---|
| [System architecture](system-architecture.md) | Context, components, deployment topology, trust boundaries, startup, model lifecycle, threads and locks |
| [Backend and API](backend-and-api.md) | FastAPI design, middleware, error handling, every route, validation, request/response examples, curl |
| [Frontend architecture](frontend-architecture.md) | SPA shell, hash router, module responsibilities, state/events, polling, XSS escaping, CSP, responsive layout |
| [Inference workflow](inference-workflow.md) | Image, video and live-camera paths; frame loop; confidence/NMS; severity; annotation; telemetry; codec and device fallbacks |
| [Component interactions](component-interactions.md) | Sequence diagrams for image inference, video jobs, model reload and missing-checkpoint failure |
| [Data flow](data-flow.md) | Data at rest and in motion, upload handling, result/job directories, telemetry fields, retention, privacy |
| [Deployment and operations](deployment-and-operations.md) | Supported local deployment, launch commands, environment variables, ports, platform notes, logs, backup, non-goals |

## At a glance

| Property | Accepted implementation |
|---|---|
| Backend | FastAPI application in `infrascan/api.py`, ASGI entrypoint `app.py` |
| Frontend | Hand-written HTML, CSS and ES modules in `web/`; no npm, bundler or transpiler |
| Serving model | API and SPA mounted on the same Uvicorn origin; `/assets` is a `StaticFiles` mount |
| Default bind | `127.0.0.1:8000` |
| Model | Ultralytics YOLO11n, single `pothole` class, 640 px input, checkpoint supplied locally |
| Image inference | `POST /api/detect/image`, synchronous, JSON response plus annotated JPEG URL |
| Video inference | `POST /api/detect/video` returns `202`; background job polled via `GET /api/jobs/{id}` |
| State | In-memory model state and job registry; files on disk under `runs/dashboard/` |
| Authentication | None |
| Persistence | Local filesystem only; no database, cloud storage or accounts |
| Runtime tested | macOS on Apple M2 MPS, Python 3.12.14 (see [`PROJECT_FACTS.md`](../PROJECT_FACTS.md)) |

## Suggested reading order

- **New to the codebase:** this page → [System architecture](system-architecture.md) → [Backend and API](backend-and-api.md) → [Frontend architecture](frontend-architecture.md).
- **Operator or demonstrator:** [Deployment and operations](deployment-and-operations.md) → [Installation](../06-setup-and-execution/installation.md) → [Application usage](../06-setup-and-execution/application-usage.md).
- **Reviewing quality claims:** [Performance](../11-security-and-quality/performance.md) → [Security and privacy](../11-security-and-quality/security-and-privacy.md) → [Accessibility](../11-security-and-quality/accessibility.md).

## Related chapters

- [Setup and execution](../06-setup-and-execution/README.md) — install, configure, run and troubleshoot.
- [Security and quality](../11-security-and-quality/README.md) — threat model, measured performance, accessibility audit.
- [Verified results](../09-testing-and-results/verified-results.md) — the evidence boundary for tests and benchmarks.
