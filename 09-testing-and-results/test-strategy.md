# Test strategy

How InfraScan was tested, what each status word means, and what would need to change to claim more.

- [Objectives](#objectives)
- [Scope](#scope)
- [Test levels](#test-levels)
- [Status vocabulary](#status-vocabulary)
- [Test environment](#test-environment)
- [Test design techniques](#test-design-techniques)
- [Coverage matrix](#coverage-matrix)
- [Regression checklist](#regression-checklist)
- [Automation plan](#automation-plan)
- [Evidence rules](#evidence-rules)
- [Related pages](#related-pages)

---

## Objectives

The testing effort had three goals, in priority order:

1. **Prove the pipeline works end to end.** A single image must travel from the browser through validation, inference, severity scoring and annotation and come back as a usable result; a video must become a playable file with telemetry.
2. **Prove the failure paths are safe.** A missing checkpoint, an undecodable image, an oversized upload and an out-of-range threshold must produce an actionable error rather than a crash, a wrong answer or a silently substituted model.
3. **Measure the model honestly.** The accuracy claim must come from a fresh evaluation on the held-out test split in a recorded environment, not from numbers inherited from an earlier project description.

It deliberately did **not** aim at coverage, load, security certification or cross-platform compatibility. Those are named in the [not-measured](../11-security-and-quality/performance.md#what-is-not-measured) tables rather than left implied.

## Scope

| In scope | Out of scope |
|---|---|
| Static validity of the Python sources | Formal code coverage metrics |
| API contract and error behaviour | Automated regression suite |
| Model accuracy on the local test split | Generalisation to unseen roads/regions |
| Image and video end-to-end flows | Long-video and multi-hour stability |
| Dashboard rendering and responsiveness | Full device and browser matrix |
| CLI image and telemetry output | Distributed or multi-user operation |
| Security headers and path handling | Penetration testing, SAST/DAST, dependency audit |
| Accessibility at the source level | Screen-reader sessions, WCAG conformance |

---

## Test levels

| Level | What it examines | Representative check |
|---|---|---|
| **L1 — Static** | Syntax and importability without executing the ML stack | `python -m compileall` over the application and package; `import infrascan.api` |
| **L2 — Unit-equivalent** | A single function's behaviour in isolation | Severity banding: coverage 0.0 → MINOR, 2.5 → MODERATE, 8.0 → CRITICAL |
| **L3 — Component** | One service in isolation | `/api/health` and `/api/status` responses; `model_status()` transitions |
| **L4 — Integration** | Two or more components working together | Image upload → model → annotated JPEG → result URL renders in the SPA |
| **L5 — End-to-end** | A complete user journey | Video upload → 202 → poll → completion → playback → telemetry download |
| **L6 — System** | The deployed artefact in its real environment | The 718-image evaluation run on the recorded machine |
| **L7 — Exploratory** | Behaviour outside the written expectations | Responsive layout at 390/820/1024/1440 px; synthetic webcam capture |

Not every level was executed for every concern. The coverage matrix records exactly what was covered and where the gaps are.

---

## Status vocabulary

These terms are used consistently across this chapter, the [evidence index](evidence-index.md) and the [manual test log](manual-test-log.md).

| Status | Definition | May be promoted to PASS? |
|---|---|---|
| **PASS** | The check was executed in the recorded environment and the observed result matched the expected result | — |
| **PASS (synthetic)** | Executed, but against a synthetic or simulated source rather than the real device/dataset | Only by re-running against the real source |
| **FAIL** | Executed; the observed result did **not** match the expected result | Only by fixing the defect and re-running |
| **BLOCKED** | Could not be executed because a required hardware, permission, licence or external dependency was unavailable | Only by removing the blocker |
| **NOT RUN** | No execution is recorded. Source inspection or a design description may exist, but that is not evidence of behaviour | Only by executing the check |

Three rules follow from this table and are applied throughout the documentation:

- **A feature that exists in the code is not a test that passed.** `export.py` exists; no export was executed, so export performance is `NOT RUN`.
- **A blocked check is never silently dropped.** Physical webcam, CUDA and export targets are listed with their blockers.
- **A PASS is always traceable** to a command, an observation or an artefact recorded in [`../PROJECT_FACTS.md`](../PROJECT_FACTS.md) §5–§6.

---

## Test environment

Every recorded result was produced on one machine. The environment is part of the result.

| Component | Value |
|---|---|
| Machine | Apple M2 (arm64) |
| OS | macOS 27.0.1 (Darwin 27.0.1) |
| Python | 3.12.14 |
| PyTorch | 2.14.1 |
| Torchvision | 0.29.1 |
| Ultralytics | 8.4.183 |
| OpenCV | 5.0.0.93 |
| NumPy | 2.5.4 |
| pandas | 3.0.6 |
| Pillow | 13.0.0 |
| FastAPI | 0.156.8 |
| Uvicorn | 0.54.0 |
| python-multipart | 0.0.32 |
| Accelerator | `mps` (Apple GPU) |
| Batch / input size | 16 / 640 |
| Checkpoint | `weights/best.pt`, SHA-256 `f3490f953b4ac2aa0205c206c151be8f2da281da99708713b182492a43c7751c` |

The environment is not reproducible from the public repository alone: the checkpoint and dataset are excluded from Git by policy, and the dataset's licence is unresolved. See [dataset provenance](../05-dataset-and-model/dataset-provenance.md).

**Implication for any future run:** record the machine, the package versions, the checkpoint hash, the split and the command. Metrics from a different environment are not comparable and must not be merged into the same results table.

---

## Test design techniques

### Equivalence partitioning

Inputs were grouped into classes with expected equivalent behaviour, and one representative per class was exercised:

| Partition | Representative | Expected |
|---|---|---|
| Valid image (decodable) | A test-set image | 200 + detection payload |
| Valid image, no pothole | A background image or a road scene | 200 + zero detections |
| Non-image bytes with an image name | Random bytes as `.jpg` | 400 decode error |
| Oversized image | > 25 MB | 413 |
| Valid video container | Short MP4 | 202 + job |
| Unsupported video extension | `.txt` renamed `.mp4` | 415 |
| Threshold out of range | `conf=1.5`, `iou=0.05` | 422 |
| Checkpoint absent | Renamed `best.pt` | 503 + instructions |

### Boundary value analysis

The boundaries that matter in this system are the upload limits and the severity thresholds:

- Image size: 0 bytes, just under 25 MB, exactly 25 MB, just over 25 MB.
- Video size: just under 2 GB, just over 2 GB.
- Confidence: 0.05 (min), 0.25 (default), 1.0 (max), 0.049, 1.001.
- IoU: 0.10 (min), 0.45 (default), 1.0 (max), 0.099, 1.001.
- Severity coverage: just under 2.5 %, exactly 2.5 %, just under 8 %, exactly 8 %.

### State-transition testing

The model lifecycle has observable states (`starting` → `loading` → `ready`, and `unavailable`/`error`). Each transition was exercised: a normal load to `ready`, a load with the checkpoint renamed to produce `unavailable`, and a forced reload via `POST /api/model/reload`.

### Negative and abuse testing

Malformed input, missing files, wrong methods, hostile filenames (path traversal attempts) and out-of-range parameters were all treated as first-class cases, because an inspection tool that silently accepts bad input produces misleading evidence.

### Equivalence of the shared policy

The severity policy has exactly one implementation (`infrascan/severity.py`), imported by the dashboard inference service, `detect.py` and `test_camera.py`. A label inconsistency between the dashboard and the CLI would corrupt the evidence trail, so the sharing is a design control as well as a convenience.

---

## Coverage matrix

| Concern | Level | Status | Where |
|---|---|---|---|
| Python syntax | L1 | **PASS** | [Test log TC-01](manual-test-log.md#tc-01) |
| Package import | L1 | **PASS** | TC-02 |
| API app object and title | L1 | **PASS** | TC-03 |
| Severity banding logic | L2 | **PASS** | TC-04 |
| Zero/invalid frame area | L2 | **PASS** | TC-05 |
| Liveness endpoint | L3 | **PASS** | TC-06 |
| Status payload completeness | L3 | **PASS** | TC-07 |
| Model state transitions | L3 | **PASS** | TC-08 |
| Sample listing | L3 | **PASS** | TC-09 |
| Image inference | L4 | **PASS** | TC-10 |
| Image latency (warm) | L4 | **PASS** | TC-11 |
| Annotated result retrieval | L4 | **PASS** | TC-12 |
| Invalid image handling | L4 | **PASS** | TC-13 |
| Threshold validation | L4 | **PASS** | TC-14 |
| Oversized upload rejection | L4 | **PASS** | TC-15 |
| Unsupported video extension | L4 | **PASS** | TC-16 |
| Missing checkpoint behaviour | L4 | **PASS** | TC-17 |
| Path traversal rejection | L4 | **PASS** | TC-18 |
| Video job lifecycle | L5 | **PASS** | TC-19 |
| Video playback | L5 | **PASS** | TC-20 |
| Telemetry export | L5 | **PASS** | TC-21 |
| CLI image + telemetry | L5 | **PASS** | TC-24 |
| Responsive layout | L7 | **PASS** | TC-22 |
| Webcam capture | L7 | **PASS (synthetic)** | TC-23 |
| Model accuracy on the test split | L6 | **PASS** | TC-25 |
| Physical camera | L7 | **BLOCKED** | Requires local hardware and permission |
| CUDA execution | L6 | **BLOCKED** | Test machine is Apple M2 |
| Export runtimes | L6 | **NOT RUN** | Export path never executed |
| Concurrent/multi-user load | L6 | **NOT RUN** | No load test performed |
| Security audit | L6 | **NOT RUN** | No penetration test or SAST/DAST |
| Accessibility conformance | L7 | **NOT RUN** | Source review only — see [accessibility](../11-security-and-quality/accessibility.md) |

---

## Regression checklist

Run this after any change to the backend, the model path, the frontend or the shared severity policy. The first eight items take under a minute and catch the majority of realistic regressions.

**Startup and model state**

1. Launch the app; confirm the dashboard loads and the model chip reaches `ready` (or `unavailable` with instructions if the checkpoint is absent).
2. `GET /api/health` → `{"status":"ok","model_state":"ready"}`.

**Image path**

3. `GET /api/status` → app, model, runtime, severity policy, media limits and thresholds all present.
4. POST a valid image → 200, one or more detections, `annotated_url` present, annotated JPEG retrievable.
5. POST random bytes as `.jpg` → 400 with a decode message.
6. POST with `conf=1.5` → 422 naming the allowed range. Repeat with `iou=0.05` → 422.

**Video path**

7. POST a short MP4 → 202 with a job id; poll to completion; fetch `/video` (plays) and `/telemetry.csv`.
8. POST a file with an unsupported extension → 415.

**Security and paths**

9. `GET /api/samples/../../etc/passwd` → 404, and the response contains no filesystem content.
10. `curl -I http://127.0.0.1:8000/api/health` → `X-Content-Type-Options`, `X-Frame-Options`, `Referrer-Policy`, CSP present.

**Frontend**

11. Load `/#/inspect`, `/#/video`, `/#/system` directly; each renders without a console error.
12. Resize to 390 px and 1440 px; no page-level horizontal overflow; the status chip still shows its text at 390 px.
13. Open the command palette with `⌘K`/`Ctrl+K`; arrow to a command and press Enter; Escape restores focus.

**CLI and shared policy**

14. `python detect.py --source <image> --weights weights/best.pt --save` → annotated JPEG plus CSV and JSON telemetry.
15. Confirm the CLI's severity labels match the dashboard's for the same image (both call `classify_severity`).

**Only when the model, data or preprocessing changes**

16. Re-run the full evaluation and archive the new metrics alongside the previous run; never overwrite a result in place.

---

## Automation plan

No automated suite exists. This is the highest-value gap in the project's engineering quality, and the plan below is the concrete route to closing it.

| Priority | Suite | Framework | Cases | Notes |
|---|---|---|---|---|
| 1 | API contract | `pytest` + `httpx` | ~25 | Use FastAPI's `TestClient`; cover the status codes in the coverage matrix |
| 2 | Severity policy | `pytest` | ~8 | Pure function; table-driven over the band boundaries |
| 3 | Upload and path handling | `pytest` | ~10 | Malformed bytes, oversize, traversal, bad extensions |
| 4 | Model lifecycle | `pytest` | ~5 | With a stubbed checkpoint; no GPU required |
| 5 | Frontend smoke | Playwright | ~8 | Load each route, run one detection, check the error states |
| 6 | End-to-end video | Playwright | ~2 | Upload a fixture clip, poll, assert the output exists |

Practical notes:

- The API suite can run **without a checkpoint** by asserting the `unavailable` path, so it stays fast and CI-friendly.
- A separate, slower job can run the real model and the evaluation on a tagged runner with the dataset mounted.
- The fixtures needed are small: a 1-pixel decodable image, a text file with an image extension, a 1-second silent MP4, and a truncated MP4.
- The 25 manual cases in the [test log](manual-test-log.md) are written as test-log entries precisely so that they can be converted into automation one by one.

---

## Evidence rules

1. **Record the command.** A result without the exact invocation cannot be reproduced.
2. **Record the environment.** Machine, accelerator and package versions.
3. **Record the observed output.** Paste the response or the number, not a paraphrase.
4. **Record the limitation.** Every result is bounded by the split, device and checkpoint it was produced with.
5. **Never merge environments.** An MPS number and a CUDA number never share a column.
6. **Never promote a not-run check.** It stays `NOT RUN` until executed.
7. **Keep the raw artefact.** The PR curve, the evaluation summary and the screenshots live in [`../assets/`](../assets/) and are indexed in the [evidence index](evidence-index.md).

---

## Related pages

- [Verified results](verified-results.md) — the measured outcomes.
- [Evidence index](evidence-index.md) — claim-to-artefact mapping.
- [Manual test log](manual-test-log.md) — 25 executed cases.
- [Performance](../11-security-and-quality/performance.md) — timings and what was not measured.
- [Accessibility](../11-security-and-quality/accessibility.md) — the source-level WCAG review.
- [Verified facts](../PROJECT_FACTS.md) — the canonical evidence boundary.
