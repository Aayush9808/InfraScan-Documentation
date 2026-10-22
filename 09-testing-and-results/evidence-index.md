# Evidence index

Every substantive claim in this documentation, mapped to the artefact, source location, command or observation that supports it — and to the limitation that bounds it.

> A claim without a row in this table is a claim without evidence. If you add a claim to a chapter, add a row here.

- [How to read this index](#how-to-read-this-index)
- [Model and checkpoint](#model-and-checkpoint)
- [Dataset](#dataset)
- [Evaluation results](#evaluation-results)
- [Functional behaviour](#functional-behaviour)
- [Interface behaviour](#interface-behaviour)
- [Command-line behaviour](#command-line-behaviour)
- [Architecture and code](#architecture-and-code)
- [Team and contribution](#team-and-contribution)
- [Assets and artefacts](#assets-and-artefacts)
- [Claims deliberately absent](#claims-deliberately-absent)
- [Related pages](#related-pages)

---

## How to read this index

| Column | Meaning |
|---|---|
| **Claim** | The statement as it appears in the documentation |
| **Evidence** | The artefact, file, command or observation that supports it |
| **Type** | `Source` (read from the code), `Execution` (a check that was run), `Artefact` (a captured file), `Declared` (stated by the team, not independently verified) |
| **Limitation** | What the evidence does *not* establish |

Evidence types matter when a question is asked in a viva. `Source` supports a statement about the implementation. `Execution` supports a statement about behaviour. `Declared` supports nothing on its own — it must be corroborated, and where it cannot be, it is marked as unverified in the text as well.

---

## Model and checkpoint

| Claim | Evidence | Type | Limitation |
|---|---|---|---|
| The detector is Ultralytics YOLO11n | `weights/best.pt` checkpoint metadata; `PROJECT_FACTS.md` §3 | Source | — |
| The checkpoint has 2,582,347 fused parameters and 6.4 GFLOPs | Checkpoint metadata and `model.info()` output | Execution | A parameter count says nothing about accuracy |
| The checkpoint file is 5.2 MB (5,467,546 bytes) | File on disk | Execution | File size is not load time or memory footprint |
| The checkpoint SHA-256 is `f3490f95…3c7751c` | `shasum -a 256 weights/best.pt` | Execution | Identifies the file; does not grant redistribution rights |
| The model was trained for 15 epochs, batch 16, 640 px, lr0 0.003, `cos_lr=True`, `close_mosaic=3`, device 0 | `train_args` inside the checkpoint | Source | Records the *recorded* run; `train.py` defaults differ and are not evidence of this checkpoint |
| The training data path embedded in the checkpoint points at an upstream Bisman Singh training directory | `train_args["data"]` | Source | Indicates provenance, not licence or permission |
| The dashboard never substitutes another checkpoint when one is missing | `infrascan/api.py` `checkpoint_available()` guard; `POST /api/detect/image` returns 503 | Source + Execution | The CLI scripts *do* fall back to `yolo11s.pt`; the two behaviours must be stated separately |
| `run_app.py --weights` does not change the running process's checkpoint path | `run_app.py` imports `infrascan.config` before `main()` parses arguments; `config.WEIGHTS_PATH` is resolved at import | Execution | Use `INFRASCAN_WEIGHTS=…` before launch; `--weights` is only reliable in a `--reload` child process |
| Severity is CRITICAL ≥ 8 %, MODERATE ≥ 2.5 %, MINOR < 2.5 % of frame area | `infrascan/severity.py` | Source + Execution | A geometric heuristic, not depth, volume or structural risk |

---

## Dataset

| Claim | Evidence | Type | Limitation |
|---|---|---|---|
| The local dataset holds 4,719 images and 7,340 pothole boxes across 22 background images | Direct count of `dataset/images/*` and `dataset/labels/*` | Execution | Counts the local copy only |
| Train 3,281 / val 720 / test 718 images | Directory counts; the evaluation log independently reports 718 test images | Execution | — |
| Test split has 994 instances and 2 background images | Evaluation log | Execution | Only 2 background images — too few to characterise false positives |
| The label format is `<class_id> <x_center> <y_center> <width> <height>`, normalised to `[0,1]` | `data.yaml`; a sampled label file | Source + Execution | — |
| There is exactly one class, `pothole` | `data.yaml` (`nc: 1`) | Source | — |
| A VOC ingestion utility and a mask-to-box preparation utility exist | `scripts/ingest_voc_dataset.py`, `scripts/prepare_dataset.py` | Source | Utilities existing is not the same as the split being produced by them |
| The dataset's original creators, source URLs, licences and redistribution permissions | **Not established** | — | Marked `[TO BE CONFIRMED]`; the dataset must not be described as freely reusable |

---

## Evaluation results

| Claim | Evidence | Type | Limitation |
|---|---|---|---|
| P 0.7959, R 0.5996, mAP@0.50 0.7111, mAP@0.50–0.95 0.3938 | The 718-image evaluation run on the local test split | Execution | Apple MPS, one split, one checkpoint |
| The evaluation used 718 images / 994 instances / 2 backgrounds | Evaluation log | Execution | — |
| Preprocess 0.17 ms, inference 3.82 ms, postprocess 10.75 ms per image | Evaluation log speed block | Execution | Batch-16 amortised, MPS only |
| The evaluation ran at 640 px, batch 16, on `mps` | The recorded command | Execution | Not comparable with a CUDA run |
| The environment was Apple M2, macOS 27.0.1, Python 3.12.14, PyTorch 2.14.1, Ultralytics 8.4.183 | `PROJECT_FACTS.md` §5 | Execution | — |
| The PR curve shown in the documentation comes from this run | `assets/evaluation/pr-curve.png` | Artefact | A curve without its command and environment is not evidence; the context is on this page |

---

## Functional behaviour

| Claim | Evidence | Type | Limitation |
|---|---|---|---|
| The application and package compile | `python -m compileall` over the application and package | Execution | Compilation proves syntax, not behaviour |
| `app` imports and is titled `InfraScan API` | `import infrascan.api` | Execution | — |
| The dashboard root returns HTTP 200 and serves the SPA | `GET /` | Execution | — |
| `/api/health` returns `{"status":"ok","model_state":"ready"}` | `GET /api/health` | Execution | Liveness of the process, not of the model, if the model is unavailable |
| One image produced 1 MODERATE detection at 7.0 % coverage | `POST /api/detect/image` response | Execution | A single image; it says nothing about precision or recall |
| The warm image request took 66.2 ms | The recorded API response | Execution | One machine, one image, warm cache |
| An invalid image returns HTTP 400 with a decode message | `POST /api/detect/image` with non-image bytes | Execution | — |
| Out-of-range thresholds are rejected | `_validate_thresholds()`; 422 responses | Source + Execution | — |
| With the checkpoint absent the model state is `unavailable`, the dashboard still serves, and detection returns an actionable error | `/api/status`; `POST /api/detect/image` → 503 | Execution | — |
| A video job processed 24 frames and produced 28 frame-level detections | `GET /api/jobs/{id}` | Execution | One short clip; no sustained-throughput claim |
| The annotated video is H.264 and played in Chrome (`readyState 4`) | Browser playback check | Execution | One browser, one clip; the `mp4v` fallback path is untested |
| CSV and JSON telemetry are produced for a video job | `/api/jobs/{id}/telemetry.csv` and `.json` | Execution | — |
| Path traversal against the sample route is rejected | `GET /api/samples/../../etc/passwd` → 404 | Execution | A spot check, not a fuzzing campaign |
| Security headers are present | `X-Content-Type-Options`, `X-Frame-Options`, `Referrer-Policy`, CSP, `Cache-Control` on API responses | Source + Execution | Header presence is not a vulnerability assessment |
| The SPA serves from the same origin as the API | `web/index.html` served by the FastAPI app; relative `/api/...` calls | Source | — |

---

## Interface behaviour

| Claim | Evidence | Type | Limitation |
|---|---|---|---|
| The layout is responsive at 390, 820, 1024 and 1440 px with no page-level horizontal overflow | Manual checks at four widths | Execution | Four widths are not a device matrix; touch and zoom were not tested |
| The interface supports image upload, dataset samples and webcam capture | `web/assets/js/inspect.js` | Source + Execution | — |
| The command palette opens with `⌘K`/`Ctrl+K` and supports arrow/Enter/Escape | `web/assets/js/palette.js` | Source | No screen-reader verification |
| Severity labels are rendered from the API's severity policy, not recomputed in JavaScript | `GET /api/status` → severity policy; `web/assets/js/system.js` | Source | — |
| Webcam capture works with a synthetic media device | Chrome synthetic media device test | Execution | **Not** a physical-camera result; physical camera is `BLOCKED` |
| The interface is accessible | **Not established as a conformance claim** | Source review | Ten source-level findings are open; see [accessibility](../11-security-and-quality/accessibility.md) |

---

## Command-line behaviour

| Claim | Evidence | Type | Limitation |
|---|---|---|---|
| `detect.py` writes an annotated JPEG plus CSV and JSON telemetry | Execution of the CLI | Execution | — |
| The CLI and the dashboard classify severity identically | Both import `classify_severity()` from `infrascan/severity.py` | Source | A shared function is strong evidence, not a measured equivalence test |
| `test_camera.py` compiles, imports and shows `--help` | Execution | Execution | **Not** a camera run |
| `train.py`, `evaluate.py` and `export.py` expose the documented flags | Source | Source | `train.py`'s defaults are not the recorded checkpoint's settings |
| `export.py` produces an ONNX/TensorRT/etc. model | **Not established** | — | Export is `NOT RUN`; no exported artefact exists |

---

## Architecture and code

| Claim | Evidence | Type | Limitation |
|---|---|---|---|
| The stack is FastAPI + Uvicorn serving a zero-build vanilla ES-module SPA | `infrascan/api.py`, `web/index.html`, `web/assets/js/*.js` | Source | — |
| The model is loaded lazily in a background thread during the lifespan | `infrascan/api.py` lifespan; `infrascan/inference.py` `start_model_load()` | Source | Load time was not measured |
| Predictions are serialised by a lock on the single model instance | `_predict_lock` in `infrascan/inference.py` | Source | Concurrency behaviour under load is untested |
| Video jobs are serialised by a worker lock and run off the event loop | `_worker_lock` in `infrascan/jobs.py`; `run_in_threadpool` in the API | Source | No multi-user test |
| The H.264 encoder falls back to `mp4v` if `avc1` cannot initialise | `detect.py` writer setup | Source | The fallback path was not exercised |
| Upload limits are 25 MB (image) and 2 GB (video); reads are chunked at 1 MB | `infrascan/config.py` | Source + Execution | — |
| Results and jobs have a 24-hour TTL, purged at startup | `infrascan/config.py`; `purge_old_artifacts()` | Source | Purging is startup-only, not continuous (SEC-13) |
| The dashboard is served by `run_app.py` / `uvicorn app:app` | `app.py`, `run_app.py` | Source | — |
| The upstream project was a Streamlit prototype that this release replaced | `git show 9242ff3:app.py` in the code repository | Execution (history) | Historical context only; the current application is not Streamlit |

---

## Team and contribution

| Claim | Evidence | Type | Limitation |
|---|---|---|---|
| The team is Aayush Kumar Shrivastav (2301921540002), Kajal Rawal "Lucky" (2301921520087), Khushi Shakya (2301921520091), Ronak Saini (2301921520158) | `CONTRIBUTIONS.md` | Declared | Names and roll numbers are team-declared; only Aayush's Git authorship is independently verifiable |
| Aayush's commits are independently verifiable | `git log main` in the code repository | Execution | See [`CONTRIBUTIONS.md`](../CONTRIBUTIONS.md) for the per-file attribution |
| The other members' individual contributions | **Not independently verifiable** | Declared | Documented as team-declared roles with the evidence gap stated explicitly |
| Group ID and supervisor name/designation | **Not available** | — | Kept as `[TO BE CONFIRMED]` |

---

## Assets and artefacts

| Artefact | Path | Origin | Use |
|---|---|---|---|
| Overview screenshot | `assets/screenshots/overview.webp` | Copied from the code repository's `docs/screenshots/` | Homepage and presentation evidence |
| Image-inspection screenshot | `assets/screenshots/image-inspection.webp` | Captured from the running dashboard at 1440 × 1100 | Shows the detection result panel |
| Video screenshot | `assets/screenshots/video.webp` | Copied from the code repository's `docs/screenshots/` | Shows the job/progress UI |
| System screenshot | `assets/screenshots/system.webp` | Copied from the code repository's `docs/screenshots/` | Shows runtime, severity policy and API reference |
| PR curve | `assets/evaluation/pr-curve.png` | Copied from the code repository's `docs/evaluation/` | The precision–recall curve from the verified run |

The screenshots show the running application. They do **not** prove accuracy on their own; the accuracy evidence is the evaluation run recorded above.

---

## Claims deliberately absent

The following are asserted nowhere in this documentation because no evidence supports them:

- CUDA, FPS or memory-performance figures.
- Export success for any format, or any exported artefact.
- Physical-webcam results.
- Safety certification, compliance or production readiness.
- Depth, volume, structural-risk or "road condition score" outputs.
- GPS or GIS integration, geotagging or map-based triage.
- Multi-object defect classes such as cracks or lane marking.
- Cross-frame tracking, object identity or duplicate merging across frames.
- Dataset or checkpoint redistribution rights.
- A formal security audit, penetration test or dependency audit.
- WCAG conformance.
- Any accuracy figure other than the verified 718-image evaluation.

If any of these appear in a slide, a poster or an answer, they are errors — and they should be corrected rather than defended.

---

## Related pages

- [Verified facts](../PROJECT_FACTS.md) — the canonical evidence boundary.
- [Verified results](verified-results.md) — the measured outcomes.
- [Manual test log](manual-test-log.md) — the 25 executed cases.
- [Test strategy](test-strategy.md) — levels, status vocabulary and the regression checklist.
- [Contribution evidence](../CONTRIBUTIONS.md) — declared roles versus verified authorship.
- [Missing information](../MISSING_INFORMATION.md) — the open academic and provenance fields.
- [Evidence assets](../assets/README.md) — the captured files and their provenance.
