# Testing and results

> **Evidence policy:** every PASS on these pages is traceable to an executed check recorded in [`../PROJECT_FACTS.md`](../PROJECT_FACTS.md) §5–§6. Source inspection is labelled as such. A feature being implemented is **not** the same as a test passing. No dates, logs or metrics are invented.

This chapter records what was tested on the accepted InfraScan implementation, what the results mean, what was not run, and how to reproduce the checks.

## Navigation

| Page | Purpose |
|---|---|
| [Test strategy](test-strategy.md) | Test levels, environment, PASS/FAIL/BLOCKED/NOT RUN definitions, design techniques and regression checklist |
| [Verified results](verified-results.md) | The measured 718-image evaluation, image/video/CLI checks, UI behaviour and explicit non-results |
| [Evidence index](evidence-index.md) | Claim → artifact/source/command/limitation, including screenshots, PR curve, line references and commit hashes |
| [Manual test log](manual-test-log.md) | 25 reproducible manual cases with preconditions, steps, expected, observed, status and evidence |

Related: [dataset and model](../05-dataset-and-model/README.md), [training and evaluation](../05-dataset-and-model/training-and-evaluation.md), [architecture](../04-architecture/system-architecture.md), [limitations](../03-project-documentation/limitations.md), [contribution evidence](../CONTRIBUTIONS.md).

## Result snapshot

| Area | Verified result | Status |
|---|---|---|
| Static compilation | Application and package files compiled | **PASS** |
| FastAPI import | `app` imported; title `InfraScan API` | **PASS** |
| Dashboard root | HTTP 200; SPA served | **PASS** |
| Health | `{"status":"ok","model_state":"ready"}` | **PASS** |
| Image inference | 1 MODERATE detection; 7.0% coverage; 66.2 ms | **PASS** |
| Invalid image | HTTP 400 with an actionable decode message | **PASS** |
| Missing checkpoint | Model state `unavailable`; dashboard still served; 503 on detection | **PASS** |
| Video job | 24 frames processed; 28 frame-level detections | **PASS** |
| Video playback | H.264 output; Chrome `readyState 4` | **PASS** |
| CLI image + telemetry | Annotated JPEG plus CSV and JSON telemetry | **PASS** |
| Responsive layout | 390, 820, 1024 and 1440 px; no page-level horizontal overflow | **PASS** |
| Webcam capture | Chrome synthetic media device | **PASS (synthetic)** |
| 718-image evaluation | P 0.7959; R 0.5996; mAP@0.50 0.7111; mAP@0.50–0.95 0.3938 | **PASS** |
| Physical camera | Requires local hardware/permission | **BLOCKED** |
| CUDA execution | Test machine is Apple M2; MPS path used | **BLOCKED** |
| Export formats | Documented but not exercised | **NOT RUN** |
| Automated test suite | No suite exists in the repository | **NOT RUN** |

The full evaluation environment and metric table are in [verified results](verified-results.md). The raw evidence mapping is in [evidence index](evidence-index.md).

## What this chapter does not claim

- It does not claim CUDA latency, FPS or memory figures.
- It does not claim ONNX, TensorRT, OpenVINO, CoreML or TFLite export success.
- It does not claim physical-webcam behaviour on the examiner's machine.
- It does not claim production readiness, safety certification, GPS/GIS capability, depth/volume measurement or cross-frame tracking.
- It does not claim that the 718-image split is leakage-free.
- It does not claim that the recorded metrics generalise beyond the local split.
- It does not claim that a source-code feature was exercised just because the code exists.

## Status language

| Status | Meaning |
|---|---|
| **PASS** | Executed; expected result observed; evidence recorded in `PROJECT_FACTS.md` §5–§6. |
| **PASS (synthetic)** | Executed with a synthetic device or data source; the qualifier is part of the result. |
| **FAIL** | Executed; the observed result differs from the expected result. |
| **BLOCKED** | Could not execute because hardware, permission or an external dependency was unavailable. |
| **NOT RUN** | No execution is recorded. Source inspection or a design description may exist, but it is not a pass. |

When a check is `NOT RUN`, the manual log still records the expected behaviour and the source evidence so a future tester can execute it. It is never silently promoted to PASS.

## Evidence boundary

The testing evidence was collected on one development machine:

- Apple M2, macOS 27.0.1, Python 3.12.14.
- Ultralytics 8.4.183, PyTorch 2.14.1, MPS.
- Local checkpoint `weights/best.pt` with the recorded SHA-256.
- Local 718-image test split.

It is sufficient to show that the pipeline works end to end and that the checkpoint has learned the pothole class on this split. It is not a multi-machine, multi-browser, load, security or accessibility certification.

For the implementation context, see [architecture](../04-architecture/system-architecture.md) and [inference workflow](../04-architecture/inference-workflow.md). Where those pages and `PROJECT_FACTS.md` disagree, `PROJECT_FACTS.md` is authoritative.
