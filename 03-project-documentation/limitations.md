# Limitations

This chapter is deliberately uncomfortable. A final-year report is stronger when it states what the system cannot do than when it hides those gaps behind a favourable metric. The limitations below are separated into **verified** findings (checked against the source, the local checkpoint, the dataset or the recorded runs) and **unverified** risks (plausible but not measured). Neither list is minimised.

## 1. Status vocabulary

| Status | Meaning |
|---|---|
| **VERIFIED** | Confirmed by a recorded check, the source tree or inspected metadata |
| **UNVERIFIED** | Not measured; may be true or false, but cannot be claimed |
| **BLOCKED** | The check could not run in the available environment |
| **NOT RUN** | The capability exists but was not exercised |
| **BY DESIGN** | An explicit scope decision, not an accidental omission |

## 2. The headline limitations

1. **Recall is 59.96%.** On the 718-image test split, the checkpoint missed roughly two in five annotated potholes at the operating point summarised by the evaluation. A missed pothole is not inspected.
2. **Localisation is loose.** mAP@0.50 is 0.7111, but mAP@0.50–0.95 falls to 0.3938. The detector finds many potholes, but its boxes do not align tightly enough for measurement-grade use.
3. **Severity is box area, not severity.** A large box on a nearby, shallow patch can outrank a small box on a deep, dangerous hole. Camera distance and perspective dominate the score.
4. **One class only.** Cracks, ruts, raveling, patches, manholes and edge failures are invisible to the model.
5. **No tracking.** A pothole appears once per frame. Video counts are frame-level detections, not unique defects.
6. **No geolocation.** Telemetry cannot be placed on a map or joined to a road asset register without a separate manual step.
7. **No authentication.** The API is safe only on a trusted local machine. Binding to `0.0.0.0` exposes unauthenticated detection and file endpoints to the network.
8. **No automated tests.** Acceptance relied on manual checks; regressions are not caught by CI.
9. **The evidence is one run.** One checkpoint, one split, one Apple MPS environment, no confidence intervals or repeated seeds.
10. **The artefacts are not redistributable.** Dataset and checkpoint provenance and licences are unresolved; both are excluded from the repositories.

## 3. Dataset limitations

### 3.1 Verified facts and what they do not prove

| Fact | Verified value | What it does not prove |
|---|---|---|
| Total images | 4,719 | That the images cover diverse geography, weather, lighting, camera types or road surfaces |
| Train / val / test | 3,281 / 720 / 718 | That the split is leakage-free or stratified by location |
| Pothole instances | 7,340 | That every pothole is labelled, or that labels are consistently tight |
| Background images | 22 (18 / 2 / 2) | That the model is well calibrated on defect-free roads |
| Class count | 1 (`pothole`) | That other distress types are absent from the imagery |
| Label format | Normalised YOLO `class xc yc w h` | That the original annotations were produced to a published protocol |

### 3.2 Background scarcity

Only **22 of 4,719 images (about 0.47%)** have empty label files. Background images are how a detector learns what *not* to fire on. With two background images in the test split, the reported precision mostly reflects pothole-containing scenes; false positives on clean asphalt, shadows, patches, puddles, lane markings, drain covers and repaired surfaces are **not adequately characterised**. A road with no potholes may still produce detections, and this evaluation would barely notice.

### 3.3 Provenance and licensing are unresolved

The local structure, counts and label format are verified. The following are **[TO BE CONFIRMED]**:

- the original creators and stable source URL for every image;
- the licence of each source and the right to redistribute;
- whether every local image actually came from the named upstream sources;
- how the splits were generated and whether near-duplicate frames cross split boundaries;
- whether the annotations were reviewed by a second annotator;
- whether the dataset contains the same road scene in more than one split.

The dataset is therefore **not** a reusable benchmark and must not be cited as one. A high score on an unverified split is weaker evidence than the number alone suggests.

### 3.4 Label quality and class definition

There is no documented annotation guideline in the repository. "Pothole" is not a precise visual category: a shallow depression, a repaired patch, a utility cut, a shadow and a wet stain can all look similar. Without an inter-annotator agreement study or a label audit, the ground truth itself has unknown noise. Box tightness also affects mAP@0.50–0.95 directly; loose or inconsistent boxes depress the strict metric even when the detector is visually reasonable.

### 3.5 Domain coverage is unknown

The dataset's distribution across:

- countries, cities and road classes;
- asphalt, concrete, gravel and unpaved surfaces;
- daylight, dusk, night, rain, fog and glare;
- dashcam, phone, action camera and fixed CCTV viewpoints;
- pothole size, depth and age;

is not documented. The model's behaviour under domain shift is therefore **UNVERIFIED**.

### 3.6 Duplicate and leakage risk

Video-derived datasets often contain near-identical consecutive frames. If such frames straddle train and test, the evaluation leaks. The repository has no deduplication step and no scene-level split record. This risk is **UNVERIFIED** and could inflate the reported metrics.

## 4. Detection and task limitations

### 4.1 Single class

The checkpoint has one class, `pothole` (class 0). It cannot distinguish:

- longitudinal, transverse and alligator cracking;
- rutting, shoving, raveling and bleeding;
- patched or repaired surfaces;
- manholes, drains and utility cuts;
- edge drop-offs and shoulder failures.

A road with severe cracking but no visible pothole can be scored as clean. Any multi-distress use case requires new labels, a new checkpoint and a new evaluation.

### 4.2 Severity is a geometric heuristic

Severity is computed as:

```text
coverage_pct = ((x2 - x1) × (y2 - y1)) / (frame_width × frame_height) × 100
```

| Band | Coverage |
|---|---:|
| CRITICAL | ≥ 8.0% |
| MODERATE | ≥ 2.5% and < 8.0% |
| MINOR | < 2.5% |

This is **not** a measurement of depth, volume, structural damage, ride quality or risk. It has several failure modes:

- **Perspective bias.** A pothole close to the camera occupies more pixels than the same pothole farther away. A dashcam at a different height or angle changes the band.
- **Focal-length bias.** A wide-angle lens shrinks the frame coverage of a fixed defect; a telephoto lens enlarges it.
- **Box-shape bias.** A long, thin crack-like region and a compact deep hole can share the same area but not the same hazard.
- **Occlusion.** A partially occluded pothole yields a smaller box and may be downgraded.
- **False boxes.** A large false positive can be labelled CRITICAL with complete internal consistency.
- **No calibration.** The 8.0% and 2.5% thresholds are policy choices, not empirically validated safety thresholds. No study in this project relates them to repair cost, injury risk or ride quality.

The label is useful for **triage and relative ordering within a consistent camera setup**, not for engineering decisions.

### 4.3 No tracking or deduplication

Video inference is frame-by-frame. There is no IoU tracker, Kalman filter, ByteTrack, re-identification or temporal smoothing. Consequences:

- the same pothole is counted in every frame in which it is detected;
- `detections_total` is a detection count, not a pothole count;
- a brief false positive appears in the summary as a real record;
- there is no track id in telemetry, so downstream deduplication must be done externally;
- the "critical" count for a clip can be dominated by one large pothole seen for many frames.

This is **BY DESIGN** for scope, but it materially limits the meaning of video summaries.

### 4.4 No geolocation or GIS

Telemetry contains no latitude, longitude, road name, chainage, heading, timestamp or frame time. The system cannot produce a map, a GIS layer, a route-level report or a distance-normalised defect density. Any spatial claim requires an external GPS log aligned to frames, which the project does not implement.

### 4.5 Perspective, occlusion and scene variation

A bounding box is a 2D projection. The detector can fail or drift under:

- strong shadows, wet reflections, sun glare and night-time headlights;
- rain, fog, dust and motion blur;
- road markings, tar seams, shadows and dark patches that resemble holes;
- potholes filled with water, gravel or debris;
- partial occlusion by vehicles, pedestrians or vegetation;
- unusual camera pitch, roll or fisheye distortion;
- very small defects at distance and very large defects that exceed the receptive field.

These are **UNVERIFIED** failure modes in the sense that the evaluation does not stratify by them, even though they are well-known in road-damage literature.

### 4.6 Confidence is not calibrated probability

`confidence` is the model's detection score. It is not a calibrated probability that the box is a pothole. A score of 0.75 does not mean "75% chance"; it means the detector ranked this candidate above others under its training objective. The dashboard exposes the score and a threshold, but does not calibrate it, plot reliability diagrams or report expected calibration error. Users should not interpret confidence as a risk percentage.

### 4.7 Threshold sensitivity

Confidence and NMS IoU are user-adjustable. Lowering confidence surfaces more candidates and usually raises recall at the cost of precision; raising IoU keeps more overlapping boxes. The reported evaluation is a single operating profile, not a sweep. The dashboard does not show a precision-recall curve, warn that a threshold change invalidates the previous result (it does note "parameters changed" in the UI), or recompute metrics after a change.

## 5. Evaluation limitations

### 5.1 One run, one split, one environment

The verified metrics come from a single evaluation of one checkpoint on the 718-image test split, using Apple MPS, batch 16 and image size 640. There are:

- no repeated runs with different seeds;
- no confidence intervals or standard deviations;
- no bootstrap resampling;
- no per-class metrics (there is only one class);
- no per-size (small/medium/large) breakdown;
- no per-scene or per-geography breakdown;
- no comparison against a baseline or a second architecture;
- no cross-dataset or external validation.

The numbers are reproducible in principle, but they are a point estimate, not a distribution.

### 5.2 Metrics can mislead without context

- **Precision 0.7959** does not mean 79.59% of real potholes are found; that is recall.
- **Recall 0.5996** means a substantial fraction of annotated potholes are missed at the summarised operating point.
- **mAP@0.50 0.7111** is lenient about box placement; a box that covers the pothole loosely can still count.
- **mAP@0.50–0.95 0.3938** shows that strict localisation is much weaker.
- The reported precision/recall are curve summaries, not necessarily the dashboard's default confidence of 0.25. A user operating at 0.25 may see a different trade-off.

Presenting only mAP@0.50 would flatter the system. This documentation reports all four values together.

### 5.3 Background and false-positive measurement

With two background images in the test split, the evaluation cannot estimate a meaningful false-positive rate on defect-free roads. Precision on pothole scenes is not the same as precision in deployment, where most frames may contain no pothole.

### 5.4 Speed measurements are not end-to-end

The evaluation reports 0.17 ms preprocess, 3.82 ms inference and 10.75 ms postprocess per image. These are Ultralytics validation timings on MPS, not dashboard latency. The dashboard's `inference_ms` includes lock wait but excludes image decode, annotation and JPEG write. The recorded API response was 66.2 ms, which includes more of the request path but is still a single observation, not a benchmark. No throughput, memory, thermal or sustained-load study was performed.

### 5.5 Training provenance is incomplete

Checkpoint metadata records 15 epochs, batch 16, image size 640, optimizer `auto`, initial LR **0.003**, cosine LR with `lrf=0.01`, device `0` (CUDA), seed 0 and `close_mosaic=3`. The repository's `train.py` defaults are 50 epochs, `yolo11s.pt` and initial LR **0.01**. The released checkpoint is **YOLO11n**. Therefore:

- the exact training command is not recoverable from the repository defaults;
- the metadata path points to a Windows machine (`C:\Users\Bisman Singh\...`), not the evaluation machine;
- the checkpoint may have been produced outside the current scripts or with overrides;
- the checkpoint's provenance and redistribution permission are unresolved;
- metadata records Ultralytics 8.4.108, while evaluation used 8.4.183.

A reader cannot fully reproduce training from the repository alone. Evaluation is reproducible given the same checkpoint and data; training is not.

### 5.6 No statistical testing

No significance tests, effect sizes or uncertainty estimates accompany the metrics. Claims such as "model A is better than model B" cannot be supported because no B was trained or evaluated under the same protocol.

## 6. Deployment and operational limitations

### 6.1 Local-only, single-process design

The application is one Uvicorn process with in-memory job state. It is not designed for:

- multiple concurrent users;
- horizontal scaling or load balancing;
- persistent job queues across restarts;
- high availability or failover;
- container orchestration (no Dockerfile or Compose file);
- cloud storage or object-store backends;
- TLS termination or reverse-proxy configuration.

Restarting the server clears all job records. Output files remain on disk until a later startup purges those older than 24 hours. A user who restarts mid-job loses the job's status and may leave a partial directory until cleanup.

### 6.2 Export targets are unverified

`export.py` exposes ONNX, TensorRT, TorchScript, OpenVINO, CoreML and TFLite. **None was run during acceptance.** Export often requires extra toolchains (CUDA/TensorRT, OpenVINO runtime, Xcode/CoreML tools, TensorFlow Lite converter), and success is not guaranteed by the presence of the wrapper. No exported model was benchmarked, validated for numerical equivalence or tested in a target runtime. Claims that InfraScan "runs on mobile" or "supports TensorRT" are unsupported without that evidence.

### 6.3 Video codec and container fragility

The pipeline depends on the local OpenCV/FFmpeg build. It tries `avc1` then `mp4v`. If neither encoder opens, the job fails with a runtime error. Even when `mp4v` succeeds, browsers may not play the file. There is:

- no external FFmpeg transcode step;
- no server-side playback verification;
- no audio preservation (the writer is video-only);
- no frame-rate normalisation beyond the source FPS;
- no rotation metadata handling;
- no support for HLS, DASH, MKV playback in the browser (MKV is accepted for upload but the output is MP4).

The verified 24-frame H.264 playback in Chrome is a single positive case, not a codec matrix.

### 6.4 Resource consumption

- A 2 GB upload can fill disk; there is no quota or free-space check.
- Video processing creates a second model instance while the dashboard image model is loaded; memory use can exceed a small laptop's budget.
- Image and video inference can compete for the same GPU/MPS device; there is no scheduler or priority.
- Long jobs have no cancel button. Closing the tab does not stop the server-side worker.
- Job directories are removed only at startup after the TTL, not continuously.
- There is no rate limit, per-IP quota or queue length cap.

### 6.5 No observability stack

Logs go to standard output via Python logging. There are no metrics, traces, structured log fields, dashboards, alerting or error aggregation. Diagnosing a failure requires terminal access and the server log. The `/api/health` endpoint reports liveness and model state but not queue depth, disk usage or memory.

### 6.6 No CI, packaging or automated deployment

There is no `tests/` directory, pytest suite, GitHub Actions workflow, pre-commit configuration, lockfile, Dockerfile or installer. `requirements.txt` uses lower bounds (`>=`), so two installs can resolve different versions. The verified environment is recorded, but it is not enforced by the repository.

## 7. Security and privacy limitations

### 7.1 No authentication or authorisation

Every endpoint is unauthenticated. Anyone who can reach the port can:

- upload images and video;
- read annotated results and telemetry if they know or guess a UUID/job id;
- reload the model;
- consume CPU/GPU and disk.

UUIDs are unguessable in practice, but they are not an access-control mechanism. There are no users, roles, sessions, API keys or audit records.

### 7.2 Network exposure

The default bind is `127.0.0.1`, which is appropriate for a local tool. The launcher permits `--host 0.0.0.0`, which exposes the unauthenticated API to the LAN. There is no warning beyond the README, no TLS, no host allow-list and no reverse-proxy integration. A user who does this on an untrusted network accepts the risk.

### 7.3 CSRF and cross-origin requests

There is no CSRF token. The API accepts `multipart/form-data` POSTs, which a malicious page can submit cross-origin without reading the response. The practical impact is limited by the local bind and the same-origin default, but a browser on the same machine could be tricked into queuing large jobs or reloading the model. `SameSite` cookies are irrelevant because there are no cookies. A restrictive CORS policy is absent, but CORS is not a CSRF defence.

### 7.4 Checkpoint loading is code execution

Ultralytics loads `.pt` checkpoints through PyTorch. PyTorch checkpoint files can contain pickle payloads, and unpickling can execute arbitrary code. The dashboard and CLI therefore trust the checkpoint file completely. There is no signature verification, hash pinning at load time, sandbox or safe-format enforcement. **Only load checkpoints from a source you trust.** The recorded SHA-256 is an integrity aid for the documented checkpoint, not a security control for arbitrary files.

### 7.5 Observability and diagnostics

Error messages are intentionally generic at the HTTP boundary (`500` hides tracebacks), which is good for information leakage but bad for remote debugging. There is no request id in responses, no structured error code, no audit trail of who uploaded what. The server log is the only diagnostic surface, and it is not rotated by the application.

### 7.6 Dependency and supply-chain risk

The project depends on Ultralytics, PyTorch, Torchvision, OpenCV, NumPy, pandas, FastAPI, Uvicorn and python-multipart, all with lower-bound versions. There is no lockfile, hash verification, vulnerability scan or SBOM. A compromised or incompatible dependency can break the app or introduce risk. The frontend avoids npm, which removes one supply chain, but the Python stack remains substantial.

### 7.7 Privacy

Road imagery can contain:

- faces and identifiable pedestrians;
- vehicle number plates;
- homes, businesses and location cues;
- vulnerable infrastructure details.

Local processing reduces third-party exposure, but it does not anonymise the data. Uploaded files are stored under `runs/dashboard/jobs/{id}/input*`; results under `runs/dashboard/results/`; telemetry may contain image filenames. There is no encryption at rest, access control, retention policy beyond the 24-hour startup purge, or consent workflow. A shared machine leaves artefacts readable by other local users unless filesystem permissions are managed externally. This is a **privacy-by-deployment** argument, not a privacy-by-design guarantee.

### 7.8 Model and dataset licensing

The repository is MIT-licensed, but:

- Ultralytics YOLO is AGPL-3.0 by default; the checkpoint metadata records AGPL-3.0;
- the dataset's licence and redistribution rights are unresolved;
- the checkpoint's provenance and redistribution permission are unresolved;
- bundled fonts are SIL OFL (separate terms);
- dependencies have their own licences.

MIT on the repository does not relicense the model, the dataset or Ultralytics. Anyone redistributing the system must review those terms independently. See [third-party attribution](../10-references-and-governance/third-party-attribution.md).

## 8. Usability and accessibility limitations

| Area | Limitation | Status |
|---|---|---|
| **Image batch** | SPA inspects one image at a time; directory processing is CLI-only | BY DESIGN |
| **Video cancellation** | No cancel/stop control; a job runs to completion or failure | VERIFIED |
| **Job history** | In-memory only; restart clears records; no list of past jobs | VERIFIED |
| **Progress accuracy** | Depends on container frame count; some videos show frame numbers without a percentage | VERIFIED |
| **Error language** | Messages are technical ("checkpoint", "IoU", "multipart"); no glossary inline | VERIFIED |
| **Localisation** | UI is English-only; no RTL or translation layer | VERIFIED |
| **Camera** | Requires permission and a secure context; physical camera not tested | BLOCKED |
| **Accessibility** | Skip link, ARIA labels, focus styles and reduced-motion handling exist; no formal WCAG 2.2 audit, screen-reader test or colour-contrast measurement | UNVERIFIED |
| **Mobile** | Responsive at 390 px, but video upload and camera capture on mobile browsers are not tested | UNVERIFIED |
| **Offline** | The app is local, but "offline" means no internet; it still needs the local server running | BY DESIGN |
| **Large files** | 2 GB uploads may time out on slow disks or browsers; no resumable upload | VERIFIED |
| **Onboarding** | Requires a checkpoint and Python environment; no bundled demo model | BY DESIGN |

## 9. Ethical and societal limitations

1. **Automation bias.** A coloured CRITICAL chip can be treated as an authoritative repair order. The interface must not imply more certainty than the model has; the documentation repeats this, but the UI cannot enforce it.
2. **Misallocation of maintenance.** If severity scores are used to rank sites, camera perspective and pothole size bias can direct resources toward visually large but less dangerous defects, or toward roads that happened to be surveyed with a favourable camera.
3. **Coverage equity.** Roads with less camera coverage, unpaved surfaces or unusual lighting may be under-detected. A model trained on an unverified geographic distribution can systematically fail outside it.
4. **Privacy of public space.** Even local processing captures people and places. Retention and access controls are deployment responsibilities that the project does not automate.
5. **Environmental cost.** Training and inference consume energy; the project does not measure or offset it.
6. **Dual use.** The same detector can support maintenance or surveillance. The project does not impose a use policy beyond the local-only design.
7. **Academic honesty.** Presenting an unverified dataset or a single-run metric as a general result would misrepresent the evidence. This documentation deliberately avoids that.

## 10. Unverified claims register

The following statements are **not supported** by the current evidence and must not appear in the report, poster, viva or README without new evidence:

| Claim | Why it is unsupported | What would close the gap |
|---|---|---|
| "The model is 79.59% accurate" | Precision is not accuracy; mAP and recall differ | Report all four metrics with definitions |
| "It detects potholes with 71% accuracy" | mAP@0.50 is not accuracy | Explain mAP and the operating point |
| "It works on any road, anywhere" | No cross-dataset or geographic validation | External test set and stratified results |
| "Severity reflects danger/depth" | Severity is box area only | Depth/IMU validation study |
| "It counts potholes in a video" | Frame-level detections, no tracking | Tracker and deduplication evaluation |
| "It provides GPS/GIS output" | No coordinates in telemetry | GPS ingest and map export |
| "It runs on NVIDIA/TensorRT/mobile" | Exports not run; CUDA path blocked | Export and device benchmark matrix |
| "It is production-ready/secure" | No auth, TLS, tests or hardening | Security review, tests, deployment architecture |
| "The dataset is public/reusable" | Provenance and licence unresolved | Documented creator, licence and permission |
| "The checkpoint can be redistributed" | Permission unresolved | Written permission and licence record |
| "The dashboard was written by Khushi" | No independent commit evidence | Design/test artefacts and team agreement |
| "Kajal/Ronak performed specific tests" | No signed logs in Git | Dated test logs and review notes |
| "It was trained on this laptop" | Metadata says CUDA device 0; evaluation was MPS | Training log and machine record |
| "15 epochs reproduce the checkpoint" | Script defaults and metadata differ | Exact training command and log |
| "It replaces manual inspection" | Decision support only; recall 59.96% | Prospective field study and human factors review |

## 11. Mitigations and residual risk

| Risk | Mitigation already present | Residual risk |
|---|---|---|
| False positives | Adjustable confidence; visible boxes; human review | No calibrated threshold; background scarcity |
| False negatives | Adjustable confidence; recall reported honestly | 40% missed at summarised operating point; no recall guarantee |
| Oversized uploads | 25 MB / 2 GB caps; chunked video writes | Disk exhaustion and long uploads still possible |
| Concurrent load | Locks; thread pool; background jobs | No queue cap, priority or cancellation |
| Missing checkpoint | Explicit `unavailable` state; no substitution | User cannot detect without weights |
| Path traversal | `Path(name).name` checks; UUID outputs | No formal security test suite |
| Network exposure | Loopback default; headers | `0.0.0.0` opt-in; no auth/TLS |
| Privacy | Local processing; TTL cleanup | No encryption, access control or consent |
| Untrusted checkpoint | None | Arbitrary code execution on load |
| Stale artefacts | Startup TTL purge | Long-running process accumulates; restart deletes |
| Regression | Manual acceptance checks | No automated tests or CI |
| Data rights | Artefacts excluded from Git | No provenance record |
| Licensing | MIT preserved; attribution documented | AGPL and dataset terms unresolved |

## 12. Evidence that would close the gaps

A credible next evidence package would include:

1. **Dataset provenance record** — creators, URLs, licences, permissions, collection dates, annotation guideline and inter-annotator agreement.
2. **Leakage audit** — perceptual hashing or embedding-based duplicate detection, with a scene-level split.
3. **Expanded background set** — defect-free and distractor-heavy images, with a false-positive rate per kilometre or per frame.
4. **Repeated evaluation** — multiple seeds/checkpoints, confidence intervals, per-size and per-condition strata.
5. **Calibration study** — reliability diagram, expected calibration error, threshold selection rationale.
6. **Tracking evaluation** — unique-pothole counts, ID switches, false tracks.
7. **Field pilot** — a small prospective study with surveyor review, comparing model flags with manual inspection.
8. **Security review** — auth/TLS design, CSRF/rate-limit tests, checkpoint signing, dependency audit.
9. **Automated test suite** — pytest/httpx tests for routes, jobs, severity and error paths; CI workflow.
10. **Export and device matrix** — ONNX/TensorRT/OpenVINO/CoreML/TFLite runs with numerical parity and latency.
11. **Accessibility audit** — WCAG 2.2 AA review with screen-reader and keyboard testing.
12. **Licensing opinion** — written position on AGPL, dataset and checkpoint redistribution.

## 13. Summary

InfraScan is a working, locally deployable inspection workspace with a verified single-class detector, a coherent API, a usable dashboard and honest failure states. It is **not** a certified inspection instrument, a multi-class distress survey, a tracking system, a GIS product, a production service or a validated safety tool. Its most important number is not 0.7111 mAP; it is the combination of **59.96% recall**, **two background test images**, **one evaluation run** and **no field validation**. Those facts define the boundary within which the system can be trusted — and the work required to move beyond it.
