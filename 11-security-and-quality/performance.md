# Performance

What was measured on InfraScan, what those numbers actually mean, and which performance questions remain unanswered.

> **Scope limit:** every figure below comes from the single recorded evaluation run or the recorded functional checks. There is **no** load test, no memory profiling, no cold-start timing, no CUDA benchmark, no long-video benchmark and no export-runtime benchmark. Unmeasured areas are labelled, not estimated.

- [Measured performance](#measured-performance)
- [Reading the evaluation timings](#reading-the-evaluation-timings)
- [Why the dashboard is slower than 3.82 ms](#why-the-dashboard-is-slower-than-382-ms)
- [Video processing](#video-processing)
- [Concurrency and throughput model](#concurrency-and-throughput-model)
- [Startup and model loading](#startup-and-model-loading)
- [Data volume and I/O](#data-volume-and-io)
- [Frontend performance](#frontend-performance)
- [What is not measured](#what-is-not-measured)
- [Optimisation opportunities](#optimisation-opportunities)
- [How to measure properly](#how-to-measure-properly)
- [Related pages](#related-pages)

---

## Measured performance

All values come from the environment recorded in [`../PROJECT_FACTS.md`](../PROJECT_FACTS.md): Apple M2, macOS 27.0.1, Python 3.12.14, PyTorch 2.14.1, Ultralytics 8.4.183, MPS, image size 640, batch 16, checkpoint `weights/best.pt` (2,582,347 fused parameters, 6.4 GFLOPs, 5.2 MB).

| Measurement | Value | Source | Scope of the number |
|---|---:|---|---|
| Preprocess | 0.17 ms/image | Ultralytics `val` output, 718-image test split | Per-image amortised over a batch of 16 |
| Inference | 3.82 ms/image | Same run | Per-image amortised over a batch of 16 |
| Postprocess | 10.75 ms/image | Same run | Includes NMS and box scaling |
| Warm image API round trip | 66.2 ms | Recorded functional check, single image | One image, batch of 1, includes HTTP + decode + draw + disk write |
| Single-image detection | 1 MODERATE detection, 7.0% coverage | Recorded functional check | One image |
| Video job | 24 frames, 28 frame-level detections | Recorded functional check | One short clip; no duration recorded |
| Video playback | H.264, Chrome `readyState 4` | Recorded functional check | Codec/decodability only, not speed |
| CLI image + telemetry | Annotated JPEG + CSV/JSON written | Recorded functional check | Correctness, not timing |
| Model file size | 5.2 MB (5,467,546 bytes) | Checkpoint on disk | Load-time proxy only — actual load time was not timed |

Two things are deliberately absent from this table: **frames per second for video** and **peak memory**. The job API exposes `processing_seconds` and `throughput_fps`, but no long-video run was recorded, so quoting an FPS figure would be extrapolation. Memory was never profiled.

---

## Reading the evaluation timings

The 718-image evaluation reports a per-image cost of roughly **14.7 ms** in total (0.17 + 3.82 + 10.75) at batch 16.

Two properties of that number must be stated whenever it is quoted:

1. **It is batch-amortised.** Ultralytics processes 16 images per forward pass, so the 3.82 ms inference figure is not the latency of a single image. A single request is a batch of one and is materially slower per image.
2. **It excludes everything outside the model.** No HTTP, no multipart parsing, no image decode, no annotation drawing, no JPEG encode, no disk write and no UI work is included.

The postprocess figure being roughly 2.8× the inference figure is normal for YOLO-family models: NMS and coordinate scaling are cheap per box but there are many candidate boxes, and postprocess runs on the host CPU rather than the accelerator.

**A caution about comparability.** MPS timings are not CUDA timings. A different accelerator, a different PyTorch build, a different batch size or a different input size changes these numbers. Any presentation slide that shows a speed must show the device next to it.

---

## Why the dashboard is slower than 3.82 ms

The 66.2 ms warm image request is the more relevant number for a user waiting on the dashboard, and it is roughly 4.5× the batch-amortised evaluation cost. The difference is structural, not a defect:

| Stage | Included in the 66.2 ms | Included in the 3.82 ms |
|---|---|---|
| HTTP + multipart parsing | Yes | No |
| Base64/multipart overhead and copy | Yes | No |
| JPEG decode (`cv2.imdecode`) | Yes | No |
| Model forward pass | Yes | Yes (batched) |
| Box clamping, coverage, severity | Yes | No |
| Box and label drawing | Yes | No |
| JPEG encode (quality 92) and disk write | Yes | No |
| Result payload serialisation | Yes | No |

The `inference_ms` field in the API response is measured with `time.perf_counter()` **around the model call only**, so it isolates the forward pass from the surrounding work. It is the right field for comparing model behaviour across runs; it is not the response time, and the two should never be presented interchangeably.

**What the 66.2 ms does mean:** the whole local loop — upload, validate, decode, infer, score, draw, encode, write, respond — completes in about a tenth of a second on a laptop. For a human inspection workflow, that is comfortably interactive, and it is the number worth quoting when describing the product rather than the model.

---

## Video processing

Video is deliberately **not** handled in the request path.

1. `POST /api/detect/video` validates thresholds, the checkpoint and the file extension, creates an in-memory job, streams the upload to disk in 1 MB chunks and returns **HTTP 202** with a job ID.
2. A background thread calls `detect.run_detection()` with a progress callback.
3. The browser polls `GET /api/jobs/{id}` and renders stage, frame progress, percentage and ETA.
4. On completion the job exposes the annotated MP4, CSV/JSON telemetry, `frames_processed`, `detections_total`, `processing_seconds` and `throughput_fps`.

Consequences for performance:

- The HTTP event loop is never blocked by frame processing, so the status endpoint, the UI and other requests stay responsive while a clip is analysed.
- **Video jobs are serialised.** A module-level worker lock means one video job runs at a time; a second upload waits. This is a deliberate trade-off: parallel video jobs would multiply memory use and accelerator contention on a single-user laptop, and the alternative (a durable queue) is out of scope.
- **Time per frame is dominated by decode + draw + encode**, not by the model. A 640-pixel model call is a small part of a per-frame pipeline that also decodes a frame, runs inference, draws boxes and encodes a frame.
- The recorded check processed **24 frames** and produced **28 frame-level detections** on one short clip. That establishes the job → poll → encode → play path works. It does **not** establish sustained throughput for a long or high-resolution recording, and no such figure is claimed.
- The encoder tries the `avc1` (H.264) fourcc first and falls back to `mp4v`. The fallback is a compatibility path, not a performance feature: `mp4v` output depends on browser codec support.

---

## Concurrency and throughput model

| Work | Concurrency control | Effect |
|---|---|---|
| Model load | One background thread at startup; `ensure_ready(timeout=180)` waits for it | The server accepts requests while the checkpoint loads |
| Image inference | `_predict_lock` serialises access to the single Ultralytics model instance | Two simultaneous image requests queue rather than corrupt or contend for the model |
| Video inference | `_worker_lock` serialises whole jobs | One clip at a time; predictable memory use |
| CPU-bound work in the API | `run_in_threadpool` for prediction and model reload | The event loop stays free to serve the SPA and status polls |
| Uploads | Chunked reads (1 MB) and streamed writes | Memory use does not scale with the whole file for video |
| Static files | Served by FastAPI with explicit cache headers | The SPA is cached by the browser; API responses are `no-store` |

The practical result: **the application is built for one operator, not for concurrent users.** The single-model lock is correct — Ultralytics model objects are not designed for concurrent forward passes — but it means throughput beyond a single user requires a different design (one model per worker process, a queue, or a GPU server).

No concurrent-request or multi-user test was performed, so the behaviour under simultaneous load is **unverified**.

---

## Startup and model loading

The model is loaded **lazily, in a background thread, during the FastAPI lifespan**, not at import time. The consequences:

- The server starts and the dashboard renders immediately, with a "loading" chip, instead of blocking on a checkpoint read.
- `/api/status` reports the state (`starting`, `loading`, `ready`, `unavailable`, `error`) so the UI can explain a missing or broken checkpoint rather than failing opaquely.
- `/api/health` returns `{"status":"ok","model_state":...}` — the process is healthy even when the model is not, which is the behaviour you want from a diagnostic endpoint.

**Not measured:** the actual time to load the checkpoint, the time to first inference after a cold start, and the memory cost of a resident model. A 5.2 MB checkpoint loads quickly in absolute terms, but PyTorch's runtime footprint and the MPS allocator are not captured by file size, so no load-time claim is made here.

---

## Data volume and I/O

| Item | Limit / behaviour |
|---|---|
| Image upload | 25 MB maximum, read into memory in 1 MB chunks, 413 on exceed |
| Video upload | 2 GB maximum, streamed to disk, 413 on exceed |
| Annotated image | Written as JPEG at quality 92 to `runs/dashboard/results/<uuid>.jpg` |
| Annotated video | Written as MP4 to the job directory |
| Telemetry | CSV and JSON written per video job |
| Retention | 24-hour TTL for results and jobs, purged **at startup only** |
| Sample images | Up to 24 files listed from `dataset/images/test` |

Two operational notes:

- **TTL is startup-only.** Artefacts are not expired while the process runs. On a long-lived local instance, results accumulate until the next restart. This is a known gap, recorded as SEC-13 in [security and privacy](security-and-privacy.md).
- **Disk growth is unbounded within a session.** A user running many video jobs will fill the disk with annotated MP4s and telemetry. There is no quota, no size dashboard and no cleanup trigger other than the next startup.

---

## Frontend performance

The SPA is deliberately dependency-free: native ES modules, handcrafted CSS, no bundler, no framework, no runtime CDN. Consequences:

- **No build step** means no build time in the loop and no client dependency tree to audit, but also no code splitting, no tree shaking and no minification.
- **Hash routing** avoids server rewrites and keeps the whole app under one cacheable HTML shell.
- **Status polling is adaptive**: every 15 s when the model is ready, every 2.5 s while it is loading, every 5 s while the API is unreachable. An overlapping-request guard (`polling` / `queuedRefresh`) prevents a slow response from stacking up behind a fast one.
- **Progressive rendering**: the reveal-on-scroll observer adds `is-visible` classes once and then unobserves, so scroll cost does not repeat.
- **The dashboard is not a measured-performance target.** No Lighthouse run, no bundle-size budget, no paint-timing profile and no interaction-latency measurement exist.

The one frontend behaviour that directly affects server load is the polling interval. A longer interval would reduce requests; a shorter one would make state changes feel more immediate. The current values are a judgement call, not a measured optimum.

---

## What is not measured

| Question | Status | What it would require |
|---|---|---|
| Peak memory (idle, image, video) | **NOT RUN** | Process sampling during each workload |
| Cold-start and model-load time | **NOT RUN** | Timed startup from a fresh process |
| Time to first inference | **NOT RUN** | Timer around the first request after load |
| Sustained video FPS | **NOT RUN** | A long clip with `throughput_fps` recorded |
| Behaviour under concurrent requests | **NOT RUN** | Parallel client test against one instance |
| Multi-user behaviour | **NOT RUN** | Two or more browsers driving the same server |
| Long-clip stability (memory growth, leaks) | **NOT RUN** | Multi-hour video runs |
| CUDA performance | **BLOCKED** | NVIDIA hardware; the recorded machine is Apple M2 |
| Export-runtime performance (ONNX/TensorRT/CoreML) | **NOT RUN** | The export code path was never executed |
| Frontend paint and interaction timings | **NOT RUN** | Browser performance profiling |
| Model warm-up behaviour across a long session | **NOT RUN** | Sustained inference with periodic re-timing |

This table is the honest answer to "how fast is InfraScan?". The answer is: **fast enough for a single operator on the recorded machine, and uncharacterised beyond that.**

---

## Optimisation opportunities

These are **proposals**, not implemented features. Nothing here has been built or benchmarked.

| Opportunity | Rationale | Risk |
|---|---|---|
| Reuse decoded frames across the video loop | Avoids a redundant decode per frame where the pipeline already holds the array | Requires care with the OpenCV capture path |
| Batch the video frames | The model already runs efficiently at batch 16 | Changes the frame-by-frame progress semantics and memory profile |
| Skip JPEG re-encode for telemetry-only workflows | The annotated file is the expensive output | Removes a headline feature |
| Shorten the ready-state poll interval to 30 s | Fewer requests on an idle local server | Slower reflection of a manual model reload |
| Purge artefacts on a timer, not only at startup | Bounds disk growth (SEC-13) | Adds a background task and a deletion policy |
| Quantise or export the model | A smaller, faster runtime for deployment | Requires running the export path, which has never been done here |
| Cache the `YOLO` object per device | Avoids repeated loads | Already effectively done via the single model instance |

None of these should be presented as performance improvements until a benchmark shows the delta. The recorded numbers are the baseline such a benchmark would be compared against.

---

## How to measure properly

A repeatable protocol for the gaps above:

1. **Fix the environment** — record device, PyTorch/Ultralytics versions, batch size, input size, power mode and whether other applications were running.
2. **Warm up** — discard the first inference, then time at least 30 iterations, reporting median and 95th percentile rather than a mean.
3. **Separate the stages** — time HTTP, decode, model call, postprocess, draw, encode, write and serialise separately; the API already isolates the model call as `inference_ms`.
4. **Repeat the video run** with at least one long clip, recording `processing_seconds`, `throughput_fps`, `frames_processed` and peak memory.
5. **Load test** with a fixed number of concurrent clients, recording latency percentiles and error rates.
6. **Publish the artefacts** — the evaluation summary, the PR curve and the timing table — in the [evidence index](../09-testing-and-results/evidence-index.md), and record every new check in the [manual test log](../09-testing-and-results/manual-test-log.md).
7. **Never mix environments.** A CUDA number and an MPS number must never appear in the same column of a results table.

---

## Related pages

- [Security and quality index](README.md) — scope, severity definitions and the complete finding list.
- [Security and privacy](security-and-privacy.md) — upload limits, retention and the local trust boundary.
- [Accessibility](accessibility.md) — source-level WCAG review.
- [Verified results](../09-testing-and-results/verified-results.md) — the full evidence record for the figures quoted here.
- [Inference workflow](../04-architecture/inference-workflow.md) — the end-to-end request path.
- [Deployment and operations](../04-architecture/deployment-and-operations.md) — running the service and its operational limits.
- [Model explanation](../05-dataset-and-model/model-explanation.md) — architecture, parameters and GFLOPs.
