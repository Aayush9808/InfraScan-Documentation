# Speaker notes

**Target duration:** 10–12 minutes including a short demo
**Deck:** 14 slides, GLBITM template
**Rule:** the slides carry only the claim; these notes carry the explanation, evidence and transition.

---

## Before you begin

- Confirm the dashboard is running at `http://127.0.0.1:8000` and the model state is **ready**.
- Keep a backup recording/screenshot set in case the live demo fails.
- Know the two numbers you may be asked to defend: **718 images / 994 instances** and **mAP@0.50 = 0.7111**.
- Know the honest caveats: **MPS only**, **no physical webcam test**, **exports not run**, **dataset/checkpoint rights unresolved**.
- Do not say “we built YOLO.” Say “we integrated and extended an Ultralytics YOLO11n checkpoint.”

---

## Slide 1 — Title (30 seconds)

> “Good morning. We are presenting InfraScan, an AI-powered road inspection and pothole detection system. Our team is Aayush, Kajal, Khushi and Ronak. The project asks a practical question: can a compact object detector be turned into a usable, local inspection workspace that produces evidence rather than just a box on an image?”

Pause. Let the audience read the names. Mention the supervisor only if confirmed.

**Transition:** “I’ll start with what the system actually does.”

---

## Slide 2 — Introduction (60 seconds)

> “InfraScan detects potholes in three input paths: a still image, a dashcam-style video, or a frame captured from the browser camera. The detector is a YOLO11n checkpoint. Around it we built a FastAPI backend and a browser dashboard. For each detection the user sees a bounding box, confidence, the percentage of the frame covered by the box, and a severity band.

> The important design choice is that this is local-first. The image does not go to a cloud service. The model, the API and the dashboard run on the same machine. The second important choice is evidence: the system reports the actual checkpoint, device and runtime, and it refuses to pretend a model is available when it is not.”

If asked “why not just classify an image as pothole/no pothole?”: classification gives one label; detection gives location, count and coverage, which is necessary for inspection.

---

## Slide 3 — Objectives (75 seconds)

> “Our primary objective was a complete inspection loop, not only a model file. The loop is: accept an input, validate it, run inference, convert each box into a severity signal, present the annotated result, and export telemetry.

> The secondary objectives were operational: support CPU, Apple MPS and CUDA when present; fail clearly when the checkpoint or input is invalid; keep the frontend build-free; and document the system well enough to reproduce it.

> We were equally explicit about non-goals. We do not measure pothole depth. We do not track the same pothole across frames. We do not provide GPS coordinates or a maintenance decision. And we do not claim certified road-safety accuracy. Those boundaries are part of the engineering, not an afterthought.”

**Transition:** “That scope determines the technology stack.”

---

## Slide 4 — Hardware and software (60 seconds)

> “The validated environment was Python 3.12.14, PyTorch 2.14.1, Ultralytics 8.4.183 and OpenCV 5.0.0.93, on an Apple M2 using MPS. The backend is FastAPI 0.156.8 with Uvicorn. The frontend is plain HTML, CSS and ES modules, so there is no Node build step.

> The checkpoint is YOLO11n: about 2.58 million parameters, 6.4 GFLOPs, 5.2 MB. The dataset copy contains 4,719 images and 7,340 boxes, but neither the dataset nor the checkpoint is redistributed.

> CUDA is supported by the device-selection code, but we did not benchmark it here. Do not quote an NVIDIA number for this project.”

If asked “why YOLO11n instead of a larger model?”: the nano model keeps local startup, memory and latency practical for a student prototype; the verified test mAP@0.50 is 0.7111 on the 718-image test split, and larger models would need a new training/evaluation run to justify a claim.

---

## Slide 5 — Existing system / related work (75 seconds)

> “The project builds on an upstream YOLO pothole-detection codebase attributed to Bisman Singh. That upstream work gave us the detector foundation and the original training and inference scripts. We preserved the MIT notice and the upstream attribution.

> Our release changed the application layer: the original Streamlit prototype became a FastAPI service plus a custom SPA. We added a shared severity module, background video jobs with progress, telemetry downloads, H.264 output, explicit model lifecycle, upload limits and security headers. We also ran a fresh evaluation and separated verified results from inherited claims.

> So the contribution is integration and evidence discipline around an existing detector, not the invention of YOLO.”

**Transition:** “The literature gives the background for those choices.”

---

## Slide 6 — Literature survey (60 seconds)

> “We used the YOLO line of work for single-stage detection, YOLOv4 for the backbone-neck-head framing and speed/accuracy trade-off, and Maeda et al. for the feasibility of smartphone-based road-damage inspection. The PASCAL VOC work explains the annotation/evaluation conventions that the VOC ingestion script follows. Ultralytics provides the practical training and export APIs.

> These are background references. We are not claiming that InfraScan reproduces their reported numbers. Our own result is the 718-image evaluation shown later.”

Do not read the table row by row. Point to the column that matters and move on.

---

## Slide 7 — Motivation / problem statement (60 seconds)

> “Manual road inspection is visual, repetitive and hard to scale. But the engineering gap is not only detection accuracy. A user needs to inspect the result, understand why a box is labelled critical, and export evidence.

> We also saw a reproducibility problem in prototype documentation: inherited metrics, missing checkpoints and unclear test conditions. So we made failure and evidence first-class: the dashboard shows the real model state, invalid inputs return explicit errors, and the evaluation has a recorded command and environment.

> The question is therefore: can a compact detector plus a transparent inspection layer be useful, local and defensible?”

---

## Slide 8 — Description of project diagram (60 seconds)

> “This is the evolution in four bands. The base work is the upstream YOLO detector. The modifications are the shared severity policy, the FastAPI inference service, background video jobs, the SPA and telemetry. The expected result is a local inspection of image, video or webcam frame with annotated output and downloadable evidence. The conclusion is a reproducible academic prototype, not a certified safety system.”

Trace the architecture strip with a pointer. Emphasize the single pipeline: the dashboard and CLI are not two different models.

---

## Slide 9 — Modules (75 seconds)

> “There are four modules. The detection engine lives in `detect.py`, `infrascan/inference.py` and `infrascan/severity.py`. It handles image/video/camera inference, annotation and telemetry.

> The backend is `infrascan/api.py` plus `config.py` and `jobs.py`: routes, validation, model lifecycle, uploads, background jobs and security headers.

> The frontend is the `web/` tree: a hash-routed SPA with separate modules for inspection, video polling, the webcam dialog, the system page, the command palette and shared UI helpers.

> Model operations are `train.py`, `evaluate.py`, `export.py` and the dataset scripts. That separation lets us explain and test each layer independently.”

If asked for a file, point to the code walkthrough rather than improvising.

---

## Slide 10 — Module explanation (90 seconds)

> “For an image, the browser posts multipart form data to `/api/detect/image`. The API validates confidence and IoU, checks the checkpoint, decodes the image, and runs prediction in a thread pool. The inference service serialises model calls with a lock because one Ultralytics model instance is not safe to call concurrently. It clamps boxes to the image, computes coverage, classifies severity, annotates a copy and writes a JPEG.

> For video, the API returns 202 with a job ID. A background worker runs the existing `detect.run_detection` frame loop, writes an annotated MP4 and telemetry, and the browser polls `/api/jobs/{id}`. Video inference is serialised behind a worker lock so a long clip cannot starve the HTTP server.

> Severity is deliberately simple and transparent: box area divided by frame area. Critical is at least 8 percent, moderate at least 2.5, minor below that. The same function is imported by the dashboard, CLI and camera HUD. It is a geometric heuristic, not a depth measurement.”

Mention the H.264 fallback: the encoder tries `avc1` then `mp4v` so the CLI still produces a file when H.264 encoding is unavailable.

---

## Slide 11 — Screenshots (60 seconds)

> “These are captures from the accepted application. The image view shows the annotated result, the detection table and the coverage-based metrics. The video view shows the job progress, the annotated MP4 and telemetry links. The system page shows the checkpoint, device and runtime versions.

> The recorded image test produced one moderate detection at 7.0 percent coverage. The video test processed 24 frames with 28 frame-level detections and played back as H.264 in Chrome. These are functional checks, not accuracy evidence; the accuracy evidence is the 718-image evaluation.”

Do not read every UI label. Let the audience look.

---

## Slide 12 — Conclusion and scope (75 seconds)

> “The accepted system integrates a YOLO11n checkpoint with a complete local inspection loop. On the 718-image test split it achieved precision 0.7959, recall 0.5996, mAP at 0.50 of 0.7111 and mAP at 0.50 to 0.95 of 0.3938.

> The honest limitations are equally important. Recall is about 60 percent, so the model misses a substantial fraction of test potholes. Severity is a box-area heuristic. There is no tracking, no GPS and no depth. The evaluation is MPS-only. Physical camera, CUDA and export targets were not verified. Dataset and checkpoint rights are unresolved.

> Future work is therefore not ‘add more AI.’ It is: license a representative dataset, improve recall, calibrate severity with expert labels, add tracking, test export targets and run a proper accessibility/security review.”

---

## Slide 13 — References (30 seconds)

> “These are the background references and the upstream attribution. The full bibliography is in the documentation repository. We have kept the project's own evidence separate from the cited literature.”

Do not read the references aloud. Leave them on screen during questions.

---

## Slide 14 — Thank you (30 seconds)

> “Thank you. The code and documentation are linked here. We are happy to take questions, demonstrate a live image inspection, or walk through any module.”

If the demo is next, move to the prepared browser tab. If not, invite questions.

---

## Handling difficult questions

### “Why is recall only 60 percent?”

> “That is the measured result on this test split, and it is a real limitation. The model is a nano detector trained for 15 epochs on a local dataset; missed detections can come from small or partially occluded potholes, unusual lighting and domain shift. We report it rather than hiding it. Improving recall would require more representative data, longer training and threshold analysis, followed by a new evaluation.”

### “Why is the severity threshold 8 percent?”

> “It is an application heuristic chosen to create three interpretable bands, not a physically calibrated risk model. The value is centralised in `severity.py` so it cannot drift across interfaces. It should be validated against expert severity labels before any real maintenance use.”

### “Why not use segmentation?”

> “Segmentation would give a pixel mask and could improve area estimation for irregular potholes, but it also increases annotation cost and inference complexity. This project needed a compact detector and an inspectable box-based workflow. Segmentation is a reasonable future extension, not a claim we can make now.”

### “Did you test on CUDA?”

> “No. The evaluation was on an Apple M2 with MPS. The device-selection code supports CUDA, but we do not quote an NVIDIA benchmark.”

### “Is the dataset public?”

> “The local development copy is excluded from the repository and its provenance/licensing is unresolved. We can show its structure and counts, but we cannot claim redistribution rights.”

### “Which part did each teammate do?”

> “The repository history currently attributes the commits to Aayush. The team-declared roles are in the documentation, and `CONTRIBUTIONS.md` separates role allocation from verified authorship. Each member should present their specific evidence rather than relying on the role title alone.”

---

## Timing plan

| Segment | Slides | Time |
|---|---|---:|
| Opening + problem | 1–3 | 2:45 |
| Stack + context | 4–7 | 4:15 |
| Architecture + modules | 8–10 | 3:45 |
| Evidence + close | 11–14 | 2:45 |
| Demo / Q&A | — | as available |

If running late, shorten slides 5–6 and the module detail on slide 10; never cut the limitations or the verified-results caveat.

---

[Presentation index](README.md) · [Slide content](slide-by-slide-content.md) · [Demo script](demo-script.md) · [Viva preparation](../08-viva-preparation/README.md)
