# Verified results

Everything on this page was executed against the accepted implementation in the recorded environment. Anything not executed is listed explicitly and labelled.

> **Evidence boundary:** machine, package versions, checkpoint hash and split are recorded in [`../PROJECT_FACTS.md`](../PROJECT_FACTS.md). Where this page and that file disagree, `PROJECT_FACTS.md` is authoritative.

- [Model evaluation](#model-evaluation)
- [Evaluation environment](#evaluation-environment)
- [How to read the metrics](#how-to-read-the-metrics)
- [Functional checks](#functional-checks)
- [Dashboard behaviour](#dashboard-behaviour)
- [Command-line results](#command-line-results)
- [Explicit non-results](#explicit-non-results)
- [What these results do and do not establish](#what-these-results-do-and-do-not-establish)
- [Related pages](#related-pages)

---

## Model evaluation

A fresh evaluation was run on the full held-out test split using the local YOLO11n checkpoint.

| Metric | Value | Percentage |
|---|---:|---:|
| Precision (P) | 0.7959 | 79.59 % |
| Recall (R) | 0.5996 | 59.96 % |
| mAP@0.50 | 0.7111 | 71.11 % |
| mAP@0.50–0.95 | 0.3938 | 39.38 % |

| Dataset figure | Value |
|---|---:|
| Test images | 718 |
| Test pothole instances | 994 |
| Background images (empty labels) | 2 |

**Command:**

```bash
python evaluate.py \
  --weights weights/best.pt \
  --data <test-data.yaml> \
  --split test \
  --imgsz 640 \
  --batch 16 \
  --device mps
```

**Timings (per image, batch 16):**

| Stage | Time |
|---|---:|
| Preprocess | 0.17 ms |
| Inference | 3.82 ms |
| Postprocess | 10.75 ms |

The evaluation data was read from a temporary copy to avoid iCloud-evicted dataset files. The repository command is identical once `data.yaml` points at an available dataset.

---

## Evaluation environment

| Component | Version |
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
| Device | `mps` (Apple GPU) |
| Batch / input size | 16 / 640 |
| Checkpoint | `weights/best.pt` — YOLO11n, 2,582,347 fused parameters, 6.4 GFLOPs, 5.2 MB |
| Checkpoint SHA-256 | `f3490f953b4ac2aa0205c206c151be8f2da281da99708713b182492a43c7751c` |

**The checkpoint metadata records training on a CUDA device.** The reported metrics are an **Apple MPS evaluation**, not an NVIDIA benchmark. Never present these numbers as CUDA results.

---

## How to read the metrics

| Observation | Correct reading | Incorrect reading |
|---|---|---|
| Precision 0.7959 | About 80 % of reported detections matched ground truth at the evaluation operating point | "The model is 80 % accurate" |
| Recall 0.5996 | About 60 % of the 994 ground-truth potholes were found | "The model detects 60 % of roads" |
| mAP@0.50 0.7111 | Mean average precision at a loose IoU criterion of 0.50 | "71 % of images are correct" |
| mAP@0.50–0.95 0.3938 | Average over ten stricter IoU criteria; the drop shows localisation is the weaker skill | "The mAP is broken" |
| 3.82 ms inference | Per-image cost **amortised over batch 16** on MPS | "An image takes 3.82 ms end to end" |
| 66.2 ms latency | Warm full API round trip for a single image (HTTP, decode, infer, draw, encode, write) | "Inference takes 66.2 ms" |

**The headline number to present is recall.** A screening tool that misses roughly 40 % of test potholes is a prioritisation aid, not a road-safety instrument. This limitation is stated in the [limitations chapter](../03-project-documentation/limitations.md) rather than buried.

The gap between mAP@0.50 (0.7111) and mAP@0.50–0.95 (0.3938) indicates the model finds potholes but localises them loosely. That is the most actionable finding for future work.

---

## Functional checks

Executed against the accepted implementation. Each row maps to a case in the [manual test log](manual-test-log.md).

| # | Check | Status | Observed result |
|---|---|---|---|
| 1 | Python compilation | **PASS** | `compileall` completed for the application and package files |
| 2 | FastAPI import | **PASS** | `app` imported; title `InfraScan API` |
| 3 | Dashboard root | **PASS** | HTTP 200; the SPA was served |
| 4 | Health endpoint | **PASS** | `{"status":"ok","model_state":"ready"}` |
| 5 | Status payload | **PASS** | App, model, runtime versions, severity policy, media limits and thresholds returned |
| 6 | Sample listing | **PASS** | Test images listed for the dataset picker |
| 7 | Image inference | **PASS** | 1 MODERATE detection, 7.0 % coverage; annotated JPEG written |
| 8 | Image latency (warm) | **PASS** | 66.2 ms in the recorded API response |
| 9 | Invalid image | **PASS** | HTTP 400 with an actionable decode message |
| 10 | Threshold validation | **PASS** | Out-of-range confidence/IoU rejected with a range-specific message |
| 11 | Missing checkpoint | **PASS** | Model state `unavailable`; the dashboard was still served; instructions were returned |
| 12 | Video job | **PASS** | 24 frames processed; 28 frame-level detections |
| 13 | Video playback | **PASS** | H.264 output; Chrome reported `readyState 4` |
| 14 | Telemetry export | **PASS** | CSV and JSON telemetry produced for the video job |
| 15 | CLI image + telemetry | **PASS** | Annotated JPEG plus CSV and JSON telemetry written |
| 16 | Responsive layout | **PASS** | 390, 820, 1024 and 1440 px checked; no page-level horizontal overflow |
| 17 | Webcam capture | **PASS (synthetic)** | Chrome synthetic media device; a physical camera was not independently tested |

**Deliberately not substituted:** the dashboard never silently replaces a missing checkpoint with a different model. The CLI scripts retain their historical Ultralytics fallback to `yolo11s.pt` when `--weights` is omitted; that is a different, documented behaviour and must not be confused with the dashboard's `unavailable` state.

---

## Dashboard behaviour

Observed while exercising the interface:

- The dashboard is served from the same origin as the API, so no CORS configuration is involved and the SPA can use relative `/api/...` URLs.
- The model state is surfaced in the UI and in the system page rather than hidden; a missing checkpoint produces an explanation with instructions.
- The image workflow supports upload, dataset samples and webcam capture through one request path.
- The video workflow creates a job, polls for progress, and then enables the annotated video and the telemetry downloads.
- Errors are specific: an invalid image, an oversized upload, an unsupported video, an out-of-range threshold, a missing model and a failed job each produce their own message rather than one generic error.
- The interface is responsive at the four checked widths.

**Not verified:** behaviour on other browsers, on touch devices, with a physical camera, with a screen reader, or under concurrent load.

---

## Command-line results

`python detect.py --source <image> --weights weights/best.pt --save` produced:

- an annotated JPEG for the source image;
- `pothole_telemetry.csv` with per-detection rows;
- `pothole_telemetry.json` with the same records.

The CLI imports `classify_severity()` from `infrascan/severity.py`, so its severity labels are identical to the dashboard's for the same input — a deliberate design control that keeps the evidence trail consistent across interfaces.

`test_camera.py` was refactored to import the same shared policy; compilation, import and `--help` checks passed. **A physical camera run was not performed.**

---

## Explicit non-results

The following are **not** results of this project. Each is either blocked or was never executed.

| Item | Status | Why | What would close it |
|---|---|---|---|
| CUDA inference speed and FPS | **BLOCKED** | The test machine is Apple M2; only the MPS path was exercised | An NVIDIA machine and a re-run of the evaluation |
| ONNX / TensorRT / OpenVINO / CoreML / TFLite export | **NOT RUN** | The export code path was never executed; `export.py` exists but running it is not evidence | Execute the export, then run the exported model and compare outputs |
| Physical webcam behaviour | **BLOCKED** | Requires local hardware and browser permission | A camera and a permission grant on the presentation machine |
| Sustained video throughput | **NOT RUN** | Only a 24-frame clip was processed; no long-video benchmark was recorded | A long clip with `processing_seconds` and `throughput_fps` archived |
| Peak memory / memory growth | **NOT RUN** | No memory profiling was performed | Process sampling during image and video workloads |
| Cold-start and model-load time | **NOT RUN** | Startup was not timed | Timed startup from a fresh process |
| Concurrent and multi-user behaviour | **NOT RUN** | No load test was performed | A concurrent-client test with latency percentiles |
| Model accuracy on any other split or source | **NOT RUN** | The evaluation covered the local test split only | A re-evaluation with the split and environment recorded |
| Generalisation to unseen roads or regions | **NOT RUN** | No external dataset was used | A documented external evaluation |
| Security audit | **NOT RUN** | No penetration test, SAST/DAST or dependency audit | A recorded review — see [security and privacy](../11-security-and-quality/security-and-privacy.md) |
| WCAG conformance | **NOT RUN** | Source review only; no automated scan or screen-reader session | See [accessibility](../11-security-and-quality/accessibility.md#how-to-verify-conformance-later) |
| Automated regression suite | **NOT RUN** | No suite exists in the repository | The plan in the [test strategy](test-strategy.md#automation-plan) |
| Dataset provenance and licence | **NOT RUN** | The source, creators and permissions are unresolved | See [dataset provenance](../05-dataset-and-model/dataset-provenance.md) |

---

## What these results do and do not establish

**They establish that:**

- the model has learned the pothole class and generalises to the held-out split at mAP@0.50 0.7111;
- the full local loop works — upload, validate, infer, score, annotate, store, display, download;
- the documented failure paths behave as designed rather than crashing or silently substituting a model;
- the severity policy is shared, so the dashboard, CLI and camera utility agree;
- the results are reproducible in principle, because the checkpoint hash, command, split and environment are all recorded.

**They do not establish that:**

- the system is suitable for real road-safety decisions;
- the metrics transfer to other hardware, other datasets or other geographies;
- the deployment targets work — exports were never run;
- the interface is accessible to users with assistive technology;
- the application is safe to expose beyond `127.0.0.1`;
- the team can reproduce the evaluation from the public repository alone, because the dataset is excluded and unlicensed.

Those boundaries are the point of the chapter, not a weakness in it. A result that is not bounded cannot be defended.

---

## Related pages

- [Test strategy](test-strategy.md) — levels, status vocabulary and the regression checklist.
- [Manual test log](manual-test-log.md) — the 25 executed cases in detail.
- [Evidence index](evidence-index.md) — claim-to-artefact mapping, including the PR curve and screenshots.
- [Verified facts](../PROJECT_FACTS.md) — the canonical evidence boundary.
- [Performance](../11-security-and-quality/performance.md) — what the timings do and do not measure.
- [Limitations](../03-project-documentation/limitations.md) — the project-level scope statement.
