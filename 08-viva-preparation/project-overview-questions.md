# Project overview questions

Twenty opening questions that every member should be able to answer without notes. The model answers are deliberately concise; expand with the linked chapters when practising.

---

### Q1. What is InfraScan?

InfraScan is a local-first road inspection prototype for detecting potholes in images, video and browser-captured frames. It combines an Ultralytics YOLO11n detector with a FastAPI backend and a custom browser dashboard. The system returns annotated media, confidence, bounding-box coverage, a severity band and downloadable telemetry. It is an academic decision-support tool, not a certified road-safety instrument.

### Q2. What problem does it solve?

Manual road inspection is visual, repetitive and difficult to scale. A raw detector also leaves a gap: the user needs to locate the detection, judge confidence, understand the extent of the box and export evidence. InfraScan closes that gap with a complete local inspection loop rather than a model file alone.

### Q3. What are the primary objectives?

The primary objectives are single-class pothole detection, a usable local dashboard for images/video/camera frames, transparent coverage-based severity, structured telemetry, and a reproducible evaluation. Secondary objectives include device portability (CPU/MPS/CUDA), explicit failure states, a build-free frontend and documentation suitable for defence.

### Q4. What are the explicit non-goals?

InfraScan does not measure pothole depth or volume, track objects across frames, provide GPS/GIS coordinates, detect cracks or other defects, make autonomous maintenance decisions, or claim certified safety accuracy. Those boundaries are stated in the report and presentation.

### Q5. Why “local-first”?

Road images can be sensitive and the project is meant to run on a student laptop without cloud credentials. The model, API and frontend run on the same machine, and the default bind address is `127.0.0.1`. This reduces data-export risk and makes the demo independent of internet connectivity. It does **not** make the app secure for public exposure; there is no authentication.

### Q6. What is the high-level architecture?

A browser SPA sends HTTP requests to a FastAPI backend. The backend validates input and calls an inference service that owns one YOLO11n model instance. Images are processed synchronously; videos become background jobs. A shared severity module converts boxes into bands, and a detector/annotator writes JPEG, MP4, CSV and JSON outputs. FastAPI also serves the static SPA and API docs.

### Q7. Why YOLO11n?

YOLO11n is a compact single-stage detector suited to a local student prototype: the checkpoint is about 5.2 MB, 2.58M parameters and 6.4 GFLOPs. It gives bounding boxes and confidence in one forward pass, which fits the inspection workflow. A larger model might improve accuracy, but that would require a new training and evaluation run before it could be claimed.

### Q8. Why object detection rather than image classification?

Classification would return one label for the whole image. Inspection needs the location, count and size of each pothole. Detection provides boxes, which support annotation, per-detection confidence and coverage-based severity. Segmentation could provide masks, but it increases annotation and runtime cost; it is future work, not the current implementation.

### Q9. What are the inputs and outputs?

Inputs are JPG/JPEG/PNG/WebP/BMP images, MP4/AVI/MOV/MKV video, and a browser-captured frame. Outputs are an annotated JPEG or MP4, detection rows with class/confidence/box/coverage/severity, summary counts, and CSV/JSON telemetry. The API also returns runtime and model-state metadata.

### Q10. How is severity calculated?

Severity is the detection box area divided by the full frame area, multiplied by 100. The thresholds are CRITICAL ≥ 8%, MODERATE ≥ 2.5% and MINOR < 2.5%. The same `classify_severity()` function is imported by the dashboard inference service, CLI and camera HUD. It is a transparent geometric heuristic, not a physical risk model.

### Q11. What do the verified metrics mean?

On 718 test images with 994 pothole instances, the model achieved precision 0.7959, recall 0.5996, mAP@0.50 0.7111 and mAP@0.50–0.95 0.3938. Precision means about 80% of predicted potholes matched ground truth at the chosen operating point; recall means about 60% of ground-truth potholes were found. mAP summarizes precision–recall across thresholds.

### Q12. Why is recall only about 60%?

The checkpoint is a nano model trained for 15 epochs on the available local data. Misses can come from small or partially occluded potholes, unusual lighting, blur, domain shift or the confidence threshold. The result is reported honestly; improving it requires more representative data, longer training and threshold analysis, followed by a new evaluation.

### Q13. What does mAP@0.50 versus mAP@0.50–0.95 mean?

mAP@0.50 averages precision–recall at an IoU threshold of 0.50, a relatively loose localization criterion. mAP@0.50–0.95 averages over ten IoU thresholds from 0.50 to 0.95, so it rewards tighter boxes. The drop from 0.7111 to 0.3938 shows that localization quality, not only detection, limits the model.

### Q14. What was actually tested?

Verified checks include Python compilation, FastAPI import, health/status endpoints, image inference, invalid-image handling, missing-checkpoint behaviour, a 24-frame video job with 28 frame-level detections and H.264 playback, CLI image + telemetry, responsive layouts at four widths, and a synthetic browser webcam test. The full 718-image evaluation is the accuracy evidence.

### Q15. What was not tested?

Physical webcam behaviour, CUDA inference, ONNX/TensorRT/OpenVINO/CoreML/TFLite export, long-video stress, concurrent multi-user load and a formal accessibility audit were not verified. They are marked `BLOCKED` or `NOT RUN`, not “passed.” This distinction is part of the project's evidence discipline.

### Q16. What is the main technical contribution?

The contribution is integration and evidence discipline around an existing detector: a FastAPI service with model lifecycle and validation, a custom SPA, background video jobs, a shared severity policy, telemetry, explicit error states and a fresh reproducible evaluation. The team did not invent YOLO or claim the upstream detector as original work.

### Q17. What changed from the upstream project?

The upstream project provided the YOLO pothole-detection foundation and the original training/inference scripts. This release replaced the Streamlit prototype with FastAPI + SPA, added shared severity, background video processing, H.264 fallback, telemetry, upload limits, security headers, runtime diagnostics and a documented 718-image evaluation. Upstream attribution and the MIT notice are preserved.

### Q18. Why FastAPI instead of Streamlit?

Streamlit is excellent for a quick data app, but the accepted interface needed explicit routes, background jobs, progress polling, upload validation, security headers and a custom responsive UI. FastAPI provides those backend primitives while serving a static SPA from the same origin. The trade-off is more application code and a manual frontend; the benefit is control and testability.

### Q19. Why a custom SPA instead of React or Vue?

The interface is small enough for native ES modules, and a zero-build frontend removes Node/npm, bundler and dependency-update overhead from the academic environment. It also makes the source directly inspectable by examiners. The trade-off is that state management, routing and DOM rendering are hand-written, so discipline and testing are required.

### Q20. How would you summarize the project in one sentence?

> InfraScan turns a compact YOLO11n pothole detector into a local inspection workspace that produces annotated evidence, transparent coverage-based severity and downloadable telemetry, with a verified 718-image benchmark and clearly documented limitations.

---

## Cross-examination prompts

- “You said detection, but could you solve this with segmentation?” → yes, future work; higher annotation/runtime cost; current box-based severity is an approximation.
- “You said local-first, but is it secure?” → local by default, no auth, not hardened for public exposure; TLS/auth/rate limiting would be needed.
- “You said 80% precision, so is it accurate?” → precision is threshold- and split-specific; recall is 60%; the system is a screening prototype.
- “Why not just use a bigger model?” → possible, but needs new training/evaluation; nano was chosen for local practicality.
- “What is the biggest weakness?” → recall and heuristic severity, followed by unresolved dataset/checkpoint rights and untested deployment targets.

---

[Viva index](README.md) · [Computer vision and YOLO](computer-vision-and-yolo.md) · [Verified results](../09-testing-and-results/verified-results.md) · [Documentation homepage](../README.md)
