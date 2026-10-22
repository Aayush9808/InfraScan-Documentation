# Manual test log

Twenty-five executed cases covering compilation, the API contract, the failure paths, the image and video workflows, the interface and the model evaluation. Each record states its precondition, the exact command or step, what was expected, what was observed, the status and the evidence.

> **Status vocabulary and the rules for promoting a case are defined in the [test strategy](test-strategy.md#status-vocabulary).** Nothing here is `NOT RUN`; the unexecuted checks are listed separately at the end of the page and are deliberately **not** written as cases.

- [Environment](#environment)
- [How to read a case](#how-to-read-a-case)
- [Static and import checks](#static-and-import-checks)
- [Severity policy](#severity-policy)
- [API contract](#api-contract)
- [Image workflow](#image-workflow)
- [Failure paths](#failure-paths)
- [Video workflow](#video-workflow)
- [Interface](#interface)
- [Command line](#command-line)
- [Model evaluation](#model-evaluation)
- [Checks that were not executed](#checks-that-were-not-executed)
- [Related pages](#related-pages)

---

## Environment

All cases were executed on the machine recorded in [`../PROJECT_FACTS.md`](../PROJECT_FACTS.md) §5: Apple M2 (arm64), macOS 27.0.1, Python 3.12.14, PyTorch 2.14.1, Ultralytics 8.4.183, OpenCV 5.0.0.93, FastAPI 0.156.8, Uvicorn 0.54.0, device `mps`, input size 640, batch 16, with the local checkpoint `weights/best.pt` (SHA-256 `f3490f953b4ac2aa0205c206c151be8f2da281da99708713b182492a43c7751c`).

**Server used for the API cases:** `python run_app.py --host 127.0.0.1 --port 8000`, started from the repository root with the checkpoint present, unless the case states otherwise.

## How to read a case

| Field | Meaning |
|---|---|
| **Precondition** | The state required before running the case |
| **Command / steps** | Exactly what was executed, so a reader can repeat it |
| **Expected** | The behaviour the design and the documentation promise |
| **Observed** | What actually happened |
| **Status** | PASS, PASS (synthetic), FAIL, BLOCKED or NOT RUN |
| **Evidence** | Where the result is recorded, or the artefact that supports it |

---

## Static and import checks

### TC-01

**Python compilation of the application and package sources**

| Field | Detail |
|---|---|
| Precondition | Repository checked out; virtual environment active |
| Command | `python -m compileall app.py run_app.py detect.py train.py evaluate.py export.py test_camera.py infrascan/` |
| Expected | Compilation completes with no syntax errors in any file |
| Observed | `compileall` completed for the application and package files; no `SyntaxError` reported |
| Status | **PASS** |
| Evidence | `PROJECT_FACTS.md` §6; [test strategy](test-strategy.md) coverage matrix |

### TC-02

**Package import**

| Field | Detail |
|---|---|
| Precondition | Environment active; repository root as the working directory |
| Command | `python -c "import infrascan, infrascan.api, infrascan.config, infrascan.inference, infrascan.jobs, infrascan.severity"` |
| Expected | All modules import without side effects or errors |
| Observed | All modules imported successfully |
| Status | **PASS** |
| Evidence | `PROJECT_FACTS.md` §6 |

### TC-03

**FastAPI application object**

| Field | Detail |
|---|---|
| Precondition | As TC-02 |
| Command | `python -c "from infrascan.api import app; print(app.title)"` |
| Expected | The ASGI app is importable and titled `InfraScan API` |
| Observed | `app` imported; title `InfraScan API` |
| Status | **PASS** |
| Evidence | `PROJECT_FACTS.md` §6 |

---

## Severity policy

### TC-04

**Severity band boundaries**

| Field | Detail |
|---|---|
| Precondition | `infrascan.severity` importable |
| Command | Call `classify_severity(box_area, frame_area)` for coverage 0 %, 2.49 %, 2.5 %, 7.99 %, 8.0 % and 20 % against a frame area of 10000 px² |
| Expected | MINOR below 2.5 %; MODERATE from 2.5 % up to but excluding 8 %; CRITICAL at 8 % and above |
| Observed | Labels matched the documented thresholds at every boundary, including the exact 2.5 % and 8.0 % values |
| Status | **PASS** |
| Evidence | `infrascan/severity.py`; [model and severity policy](../05-dataset-and-model/model-explanation.md) |

### TC-05

**Degenerate frame area**

| Field | Detail |
|---|---|
| Precondition | As TC-04 |
| Command | `classify_severity(100, 0)` and `classify_severity(0, 0)` |
| Expected | A zero or invalid frame area is handled defensively and returns a zero-coverage MINOR result instead of raising `ZeroDivisionError` |
| Observed | Returned `MINOR` with `0.0` coverage and no exception |
| Status | **PASS** |
| Evidence | `infrascan/severity.py` docstring and implementation |

---

## API contract

### TC-06

**Health endpoint**

| Field | Detail |
|---|---|
| Precondition | Server running with the checkpoint present |
| Command | `curl -s http://127.0.0.1:8000/api/health` |
| Expected | HTTP 200 with `{"status":"ok","model_state":"ready"}` |
| Observed | `{"status":"ok","model_state":"ready"}` |
| Status | **PASS** |
| Evidence | `PROJECT_FACTS.md` §6, §7 |

### TC-07

**Status payload completeness**

| Field | Detail |
|---|---|
| Precondition | Server running |
| Command | `curl -s http://127.0.0.1:8000/api/status` |
| Expected | The response contains app metadata, model state and device, runtime versions, the severity policy, media limits, threshold ranges and sample availability |
| Observed | All sections were present, including the severity bands and the 25 MB / 2 GB limits |
| Status | **PASS** |
| Evidence | `PROJECT_FACTS.md` §7; `infrascan/api.py` `/api/status` |

### TC-08

**Model state transitions**

| Field | Detail |
|---|---|
| Precondition | Server running with the checkpoint present |
| Command | Observe `/api/health` at startup, then `POST /api/model/reload`, then observe again |
| Expected | The state progresses through loading to `ready`, and a forced reload returns to `ready` without breaking the process |
| Observed | The state settled on `ready`; the reload endpoint returned a successful status and the model remained available |
| Status | **PASS** |
| Evidence | `infrascan/inference.py` `start_model_load()`, `reload_model()`; `PROJECT_FACTS.md` §6 |

### TC-09

**Sample listing**

| Field | Detail |
|---|---|
| Precondition | Server running; the local test split is present under `dataset/images/test` |
| Command | `curl -s http://127.0.0.1:8000/api/samples` |
| Expected | A list of test images with names, URLs and sizes, capped at 24 entries |
| Observed | The sample list was returned and rendered in the dataset picker |
| Status | **PASS** |
| Evidence | `PROJECT_FACTS.md` §7; `infrascan/api.py` `_sample_files()` |

---

## Image workflow

### TC-10

**Image inference**

| Field | Detail |
|---|---|
| Precondition | Server running with the checkpoint present; one test-split image selected |
| Command | `curl -s -F "file=@<test-image>" -F "conf=0.25" -F "iou=0.45" http://127.0.0.1:8000/api/detect/image` |
| Expected | HTTP 200 with a detection list, a summary, severity counts, an `annotated_url`, and the image dimensions |
| Observed | 1 MODERATE detection at 7.0 % coverage; the summary reported 1 detection, 0 critical, 1 moderate, 0 minor; an annotated JPEG was written |
| Status | **PASS** |
| Evidence | `PROJECT_FACTS.md` §6; [verified results](verified-results.md#functional-checks) |

### TC-11

**Warm image latency**

| Field | Detail |
|---|---|
| Precondition | Server running; the model already loaded; at least one prior request completed |
| Command | Repeat the TC-10 request and read the reported `inference_ms` and the observed round-trip time |
| Expected | A warm request completes in well under a second, with a model-only figure reported separately from the total |
| Observed | 66.2 ms recorded in the API response for the warm request |
| Status | **PASS** |
| Evidence | `PROJECT_FACTS.md` §6; [performance](../11-security-and-quality/performance.md#why-the-dashboard-is-slower-than-382-ms) |

### TC-12

**Annotated result retrieval**

| Field | Detail |
|---|---|
| Precondition | A completed TC-10 request |
| Command | `GET` the `annotated_url` returned by the detection call |
| Expected | The annotated JPEG is served with an image content type and is retrievable by the SPA |
| Observed | The annotated image was returned and displayed in the result panel with its bounding box and label |
| Status | **PASS** |
| Evidence | `PROJECT_FACTS.md` §6; `infrascan/api.py` `/api/results/{filename}` |

---

## Failure paths

### TC-13

**Invalid image payload**

| Field | Detail |
|---|---|
| Precondition | Server running |
| Command | POST random bytes with a `.jpg` filename to `/api/detect/image` |
| Expected | HTTP 400 with an actionable decode message; the server stays healthy |
| Observed | HTTP 400 with a decode message; the dashboard displayed the specific error rather than a generic failure |
| Status | **PASS** |
| Evidence | `PROJECT_FACTS.md` §6; `infrascan/inference.py` `InvalidImageError` |

### TC-14

**Threshold validation**

| Field | Detail |
|---|---|
| Precondition | Server running |
| Command | POST with `conf=1.5`, then with `iou=0.05` |
| Expected | HTTP 422 naming the permitted range for the offending parameter |
| Observed | Both were rejected with range-specific messages |
| Status | **PASS** |
| Evidence | `infrascan/api.py` `_validate_thresholds()`; `infrascan/config.py` `CONF_*`, `IOU_*` |

### TC-15

**Oversized upload rejection**

| Field | Detail |
|---|---|
| Precondition | Server running |
| Command | POST a payload larger than `MAX_IMAGE_BYTES` to `/api/detect/image` |
| Expected | HTTP 413 once the 25 MB limit is exceeded, with the read aborted rather than buffering the whole file |
| Observed | The size limit was enforced and the request was rejected |
| Status | **PASS** |
| Evidence | `infrascan/api.py` `_read_upload()`; `infrascan/config.py` `MAX_IMAGE_BYTES` |

### TC-16

**Unsupported video extension**

| Field | Detail |
|---|---|
| Precondition | Server running |
| Command | POST a file with an extension outside the allowed video set to `/api/detect/video` |
| Expected | HTTP 415 before any processing begins |
| Observed | The unsupported extension was rejected |
| Status | **PASS** |
| Evidence | `infrascan/api.py` `/api/detect/video`; `infrascan/config.py` `VIDEO_EXTENSIONS` |

### TC-17

**Missing checkpoint behaviour**

| Field | Detail |
|---|---|
| Precondition | Server running; the configured checkpoint temporarily renamed so it cannot be found |
| Command | `GET /api/status`, `GET /api/health`, `GET /`, then `POST /api/detect/image` |
| Expected | The dashboard still loads; the model state is `unavailable` with instructions; detection returns HTTP 503; no alternative model is substituted |
| Observed | Model state `unavailable`; the dashboard was still served; the detection call returned an actionable error; no substitute checkpoint was used |
| Status | **PASS** |
| Evidence | `PROJECT_FACTS.md` §6; [verified results](verified-results.md#functional-checks) |

### TC-18

**Path traversal rejection**

| Field | Detail |
|---|---|
| Precondition | Server running |
| Command | Request `/api/samples/../../etc/passwd` and a result path containing a directory separator |
| Expected | HTTP 404 with no filesystem content in the response |
| Observed | The request was rejected and no file content was disclosed |
| Status | **PASS** |
| Evidence | `infrascan/api.py` `/api/samples/{name}` and `/api/results/{filename}`; [security and privacy](../11-security-and-quality/security-and-privacy.md) |

---

## Video workflow

### TC-19

**Video job lifecycle**

| Field | Detail |
|---|---|
| Precondition | Server running with the checkpoint present; a short clip available |
| Command | POST the clip to `/api/detect/video`, then poll `GET /api/jobs/{id}` until completion |
| Expected | HTTP 202 with a job id; the job moves from queued to running to complete; progress reports frames, percentage and ETA |
| Observed | 24 frames were processed and 28 frame-level detections were reported; the job completed and exposed its outputs |
| Status | **PASS** |
| Evidence | `PROJECT_FACTS.md` §6; `infrascan/jobs.py` `run_video_job()` |

### TC-20

**Annotated video playback**

| Field | Detail |
|---|---|
| Precondition | A completed TC-19 job |
| Command | `GET /api/jobs/{id}/video` and play the result in Chrome |
| Expected | An MP4 is served and plays in the browser |
| Observed | H.264 output; Chrome reported `readyState 4`, meaning the media was decodable and playable |
| Status | **PASS** |
| Evidence | `PROJECT_FACTS.md` §6; `detect.py` writer with the `avc1` → `mp4v` fallback |

### TC-21

**Telemetry export**

| Field | Detail |
|---|---|
| Precondition | A completed TC-19 job |
| Command | `GET /api/jobs/{id}/telemetry.csv` and `GET /api/jobs/{id}/telemetry.json` |
| Expected | Per-frame detection rows in both formats, each row carrying the box, confidence, severity and coverage |
| Observed | Both exports were produced and the download links were enabled in the interface |
| Status | **PASS** |
| Evidence | `PROJECT_FACTS.md` §6; `detect.py` telemetry writers |

---

## Interface

### TC-22

**Responsive layout**

| Field | Detail |
|---|---|
| Precondition | Server running; browser available |
| Command | Load the dashboard at viewport widths 390, 820, 1024 and 1440 px and inspect each view |
| Expected | No page-level horizontal overflow; the navigation collapses and panels stack at narrow widths; the content remains usable |
| Observed | All four widths were checked; no page-level horizontal overflow was found |
| Status | **PASS** |
| Evidence | `PROJECT_FACTS.md` §6; [frontend architecture](../04-architecture/frontend-architecture.md) |

### TC-23

**Webcam capture (synthetic device)**

| Field | Detail |
|---|---|
| Precondition | Browser with a synthetic media device enabled; server running |
| Command | Open the webcam panel, grant the permission, and capture a frame into the image workflow |
| Expected | A frame is captured, converted to a blob and sent through the same image detection path as an upload |
| Observed | The synthetic device produced a capturable frame that was accepted by the detection endpoint |
| Status | **PASS (synthetic)** |
| Evidence | `PROJECT_FACTS.md` §6. The qualifier is part of the result: a physical camera was **not** independently tested |

---

## Command line

### TC-24

**CLI image inference with telemetry**

| Field | Detail |
|---|---|
| Precondition | Checkpoint present; the working directory is the repository root |
| Command | `python detect.py --source <image> --weights weights/best.pt --save` |
| Expected | An annotated JPEG is written, together with `pothole_telemetry.csv` and `pothole_telemetry.json` containing per-detection rows |
| Observed | The annotated JPEG plus both telemetry files were written; the severity labels matched the dashboard's for the same image, as both call `classify_severity()` |
| Status | **PASS** |
| Evidence | `PROJECT_FACTS.md` §6; `detect.py` `run_detection()` |

---

## Model evaluation

### TC-25

**Held-out test-split evaluation**

| Field | Detail |
|---|---|
| Precondition | Checkpoint present with the recorded SHA-256; the test split available (read from a temporary copy to avoid iCloud-evicted files) |
| Command | `python evaluate.py --weights weights/best.pt --data <test-data.yaml> --split test --imgsz 640 --batch 16 --device mps` |
| Expected | The evaluator runs over the full test split and reports precision, recall, mAP@0.50, mAP@0.50–0.95 and the per-image speed breakdown |
| Observed | 718 images, 994 instances, 2 backgrounds; P 0.7959, R 0.5996, mAP@0.50 0.7111, mAP@0.50–0.95 0.3938; preprocess 0.17 ms, inference 3.82 ms, postprocess 10.75 ms per image |
| Status | **PASS** |
| Evidence | `PROJECT_FACTS.md` §5; [verified results](verified-results.md#model-evaluation); `assets/evaluation/pr-curve.png` |

---

## Checks that were not executed

These are recorded so the coverage is complete and honest. They are **not** cases, because no execution was recorded for them.

| Check | Status | Blocker or reason |
|---|---|---|
| Physical camera capture | **BLOCKED** | Requires local camera hardware and browser permission |
| CUDA inference, FPS and memory | **BLOCKED** | The test machine is Apple M2; the MPS path was used |
| ONNX / TensorRT / OpenVINO / CoreML / TFLite export and re-run | **NOT RUN** | The export path was never executed |
| Sustained long-video throughput | **NOT RUN** | Only a 24-frame clip was processed |
| Peak memory and memory growth | **NOT RUN** | No memory profiling was performed |
| Cold-start and model-load timing | **NOT RUN** | Startup was not timed |
| Concurrent and multi-user behaviour | **NOT RUN** | No load test was performed |
| Automated regression suite | **NOT RUN** | No suite exists in the repository; the plan is in the [test strategy](test-strategy.md#automation-plan) |
| Security penetration test / dependency audit | **NOT RUN** | Source review only — see [security and privacy](../11-security-and-quality/security-and-privacy.md) |
| WCAG conformance and screen-reader session | **NOT RUN** | Source-level review only — see [accessibility](../11-security-and-quality/accessibility.md) |
| Evaluation on any dataset other than the local test split | **NOT RUN** | The dataset's provenance and licence are unresolved |

---

## Related pages

- [Test strategy](test-strategy.md) — levels, status vocabulary, coverage matrix and the regression checklist.
- [Verified results](verified-results.md) — the measured outcomes with their interpretation.
- [Evidence index](evidence-index.md) — claim-to-artefact mapping for the whole documentation.
- [Verified facts](../PROJECT_FACTS.md) — the canonical evidence boundary.
- [Performance](../11-security-and-quality/performance.md) — what the timings measure and what they do not.
- [Limitations](../03-project-documentation/limitations.md) — the project-level scope statement.
