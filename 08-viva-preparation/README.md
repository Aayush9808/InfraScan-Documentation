# Viva preparation

This is the **project-wide** question bank. Every team member should be able to answer the overview, architecture, testing and limitation questions. Specialist questions are grouped so each member can go deeper in their assigned area.

The individual guides in [`../07-team-preparation/`](../07-team-preparation/) contain 20+ questions and 10+ difficult follow-ups per member. Use this bank for cross-examination and whiteboard practice.

## Question sets

| Set | Questions | Best for |
|---|---:|---|
| [Project overview](project-overview-questions.md) | 20 | Every member; opening questions |
| [Computer vision and YOLO](computer-vision-and-yolo.md) | 25 | Aayush; detector questions |
| [Code and architecture](code-and-architecture-questions.md) | 25 | Aayush, Khushi; file/module questions |
| [Dataset and training](dataset-and-training-questions.md) | 20 | Ronak, Kajal, Aayush |
| [Frontend and integration](frontend-and-integration-questions.md) | 20 | Khushi, Aayush |
| [Testing and limitations](testing-and-limitations-questions.md) | 20 | Kajal, every member |

## How to use this bank

1. **Answer aloud**, not by reading. Use the answer as a model, then close the file.
2. **Draw the diagram** for architecture questions; do not only recite.
3. **Name the file** when discussing implementation. “The API validates it” is weaker than “`infrascan/api.py` calls `_read_upload()` after checking the content type and size.”
4. **Separate evidence from intention.** Say “verified on 718 test images” or “not run; blocked because…”.
5. **Practise the follow-up.** Most marks are lost when the first answer is correct but the second question exposes a gap.

## The five sentences every member should know

> InfraScan is a local-first pothole inspection system built around an Ultralytics YOLO11n checkpoint. A FastAPI backend performs validation, inference, background video jobs and security headers; a zero-build vanilla JavaScript SPA provides the dashboard. For each detection, the system reports confidence, bounding-box coverage and a shared severity band. On the verified 718-image test split, the model achieved precision 0.7959, recall 0.5996, mAP@0.50 0.7111 and mAP@0.50–0.95 0.3938 on an Apple M2 using MPS. The project is an academic decision-support prototype, not a certified road-safety instrument, and dataset/checkpoint permissions, CUDA performance, physical-webcam behaviour and export targets remain unresolved.

## Rapid-fire round

| Question | Short answer |
|---|---|
| What type of vision task is this? | Single-class object detection (bounding boxes), not classification or segmentation. |
| Which model? | Ultralytics YOLO11n. |
| How many parameters? | 2,582,347 fused parameters; 6.4 GFLOPs. |
| What is the class? | `pothole` (class 0). |
| What is the input size? | 640 px during evaluation. |
| What does the dashboard run on? | FastAPI + Uvicorn, serving a vanilla SPA from the same origin. |
| Why no Node build? | The SPA uses native ES modules, HTML and CSS; no bundler is required. |
| Where is the model loaded? | Lazily during FastAPI lifespan in `infrascan/inference.py`. |
| What happens if the checkpoint is missing? | The dashboard still serves; model state is `unavailable`; detection returns an actionable error. |
| What is the severity rule? | Box area / frame area: critical ≥ 8%, moderate ≥ 2.5%, minor < 2.5%. |
| Is severity a physical measurement? | No; it is a geometric heuristic, not depth or volume. |
| What are the verified metrics? | P 0.7959, R 0.5996, mAP@0.50 0.7111, mAP@0.50–0.95 0.3938. |
| How large is the test split? | 718 images, 994 instances, 2 background images. |
| What device was evaluated? | Apple M2 with MPS. |
| Was CUDA tested? | No. |
| Was physical webcam tested? | No; synthetic-device browser test only. |
| Are exports tested? | No; export commands are documented but not verified. |
| Is there authentication? | No; the app binds to localhost by default. |
| Is the dataset redistributed? | No; it is excluded and its licence/provenance is unresolved. |
| What is the biggest technical limitation? | Recall is about 60%, and severity is heuristic; the system is a screening prototype, not a certified inspection instrument. |

## Whiteboard drills

### Drill 1 — Draw the image path

Browser → `POST /api/detect/image` → threshold/checkpoint validation → decode → inference lock → YOLO prediction → clamp boxes → severity → annotate → JPEG + JSON → SPA result view.

### Drill 2 — Draw the video path

Browser upload → `POST /api/detect/video` → save temp file → `202` job ID → background worker → frame loop → annotated MP4 + telemetry → browser polls `/api/jobs/{id}` → playback/download.

### Drill 3 — Explain the severity function

```text
coverage = ((x2-x1) × (y2-y1)) / (frame_width × frame_height) × 100
CRITICAL >= 8.0
MODERATE >= 2.5 and < 8.0
MINOR < 2.5
```

### Drill 4 — Explain the evidence boundary

What is implemented, what was executed, what is blocked, what is untested, and what is unknown. Use [`../PROJECT_FACTS.md`](../PROJECT_FACTS.md) and [`../09-testing-and-results/evidence-index.md`](../09-testing-and-results/evidence-index.md).

## If you do not know

Say:

> “I don’t have that verified. What I can confirm is [fact]. The missing evidence is [specific test/file]. I would check [specific place] rather than guess.”

That answer is stronger than inventing a number. Examiners reward calibrated honesty.

---

[Documentation homepage](../README.md) · [Team guides](../07-team-preparation/README.md) · [Code walkthrough](../03-project-documentation/code-walkthrough.md) · [Verified facts](../PROJECT_FACTS.md)
