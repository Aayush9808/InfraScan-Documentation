# Slide-by-slide content

This is the fill-in copy for the 14-slide GLBITM template. Keep the template's header/footer and visual identity. Text in **bold** is suggested on-slide text; the rest is production guidance.

> **Source of truth:** [`../PROJECT_FACTS.md`](../PROJECT_FACTS.md). If a number is not there, do not put it on a slide.

---

## Slide 1 — Title

**InfraScan — AI-Powered Road Inspection and Pothole Detection System**

**Presented by:**
Aayush Kumar Shrivastav (2301921540002)
Kajal Rawal (2301921520087)
Khushi Shakya (2301921520091)
Ronak Saini (2301921520158)

**Under the supervision of:** `[TO BE CONFIRMED]`
**Session:** 2026–2027 · **Batch:** 2023

**Visual:** small InfraScan mark or dashboard crop, not a full-screen screenshot. Keep the department header and title template intact.

**Do not:** invent the supervisor, group ID or designation.

---

## Slide 2 — Introduction of the project

**InfraScan is a local-first road inspection workspace.**

- Detects **potholes** in images, dashcam clips and browser-captured frames.
- Uses a trained **Ultralytics YOLO11n** checkpoint.
- Adds per-detection **confidence, bounding box, coverage and severity**.
- Presents results in a **FastAPI + vanilla SPA** dashboard.
- Exports **annotated media and CSV/JSON telemetry**.

**One line:** *“The model finds the pothole; InfraScan turns the detection into an inspectable, explainable result.”*

**Visual:** simple input → model → annotated result → telemetry strip.

**Speaker cue:** define pothole detection as object detection, not classification.

---

## Slide 3 — Objectives

**Primary objectives**

1. Detect potholes with a single-class object detector.
2. Provide a usable local dashboard for images, video and camera frames.
3. Convert boxes into a transparent coverage-based severity signal.
4. Produce structured telemetry for review and reporting.
5. Keep inference, evidence and limitations visible to the user.

**Secondary objectives**

- Run on CPU, Apple MPS or CUDA when available.
- Fail clearly when the checkpoint, input or runtime is invalid.
- Keep the frontend build-free and locally hosted.
- Document the system well enough to reproduce and defend.

**Explicit non-goals**

- No depth/volume measurement.
- No GPS/GIS or cross-frame tracking.
- No autonomous maintenance decision.
- No claim of certified road-safety accuracy.

**Visual:** two columns — “What it does” / “What it does not do.”

---

## Slide 4 — Hardware and software requirements

| Layer | Verified requirement |
|---|---|
| Python | 3.12.14 used for validation (3.10+ stated by project) |
| ML runtime | PyTorch 2.14.1, Torchvision 0.29.1, Ultralytics 8.4.183 |
| Vision/data | OpenCV 5.0.0.93, NumPy 2.5.4, pandas 3.0.6, Pillow 13.0.0 |
| Backend | FastAPI 0.156.8, Uvicorn 0.54.0, python-multipart 0.0.32 |
| Model | YOLO11n, 2.58M params, 6.4 GFLOPs, 5.2 MB |
| Inputs | JPG/PNG/WebP/BMP; MP4/AVI/MOV/MKV; browser webcam frame |
| Hardware | CPU works; Apple M2 MPS validated; CUDA supported by code but not benchmarked here |
| Browser | Chromium/Chrome used for responsive and H.264 checks |

**On-slide footer:** “No Node.js or frontend build step required.”

**Visual:** a compact stack diagram, not a wall of version numbers. Put the full table in the report.

---

## Slide 5 — Existing system / related work

**Base work**

- The project derives from the upstream **YOLO Pothole Detection Model** by Bisman Singh.
- Upstream provided the YOLO training/inference foundation and the original pothole workflow.
- The inherited MIT notice and Git history are preserved.

**What this version changed**

- Replaced the Streamlit prototype with a **FastAPI backend + custom SPA**.
- Added a shared severity module used by dashboard, CLI and camera HUD.
- Added background video jobs, progress polling, telemetry downloads and H.264 output.
- Added explicit model lifecycle, error states, upload limits and security headers.
- Produced a fresh **718-image evaluation** and documented the evidence boundary.

**Related tooling:** Ultralytics YOLO, OpenCV, FastAPI, browser-native ES modules.

**Visual:** two-column “Upstream base” vs “InfraScan release” with an arrow.

**Do not:** imply that YOLO was invented by this team.

---

## Slide 6 — Literature survey

Use the template's table style. Five rows fit at readable size.

| S. No. | Year | Author(s) | Work | Method / relevance |
|---:|---:|---|---|---|
| 1 | 2016 | Redmon et al. | You Only Look Once (YOLO) | Single-stage detection; boxes and class scores in one pass |
| 2 | 2020 | Bochkovskiy et al. | YOLOv4 | Speed/accuracy co-design; backbone-neck-head view |
| 3 | 2018 | Maeda et al. | Road damage detection from smartphone images | Demonstrates vision-based road inspection feasibility |
| 4 | 2010 | Everingham et al. | PASCAL VOC challenge | Dataset/annotation evaluation conventions, including VOC XML |
| 5 | 2023– | Ultralytics | YOLO11 documentation | Practical training, validation, prediction and export APIs |

**On-slide note:** “Background references; InfraScan does not reproduce their reported results.”

Full formatted references are in [`literature-survey.md`](literature-survey.md) and [`../10-references-and-governance/bibliography.md`](../10-references-and-governance/bibliography.md).

---

## Slide 7 — Motivation / problem statement

**Problem**

- Road-surface inspection is visual, repetitive and difficult to scale by hand.
- A detector alone is not enough: users need an inspectable result, not just a box on an image.
- Prototypes often hide missing checkpoints or report numbers that cannot be reproduced.

**Motivation**

- Build a **local-first** workflow that keeps road imagery on the machine.
- Make every result traceable: confidence, box, coverage, severity, telemetry.
- Make failure honest: missing model, bad image and unsupported video are visible states.
- Produce a reproducible evaluation rather than inherited claims.

**Research/engineering question**

> Can a compact YOLO11n checkpoint plus a transparent severity and inspection layer provide a usable, locally hosted pothole-screening prototype?

**Visual:** three pain points → three design responses.

---

## Slide 8 — Description of project (diagram)

Use four horizontal bands, matching the template's “BASE WORK / MODIFICATIONS / EXPECTED RESULT / CONCLUSION” structure.

**BASE WORK**
Upstream YOLO pothole detector + dataset/model workflow

**MODIFICATIONS**
Shared severity policy → FastAPI inference service → background video jobs → zero-build SPA → telemetry/error states

**EXPECTED RESULT**
Local inspection of image/video/webcam frame with annotated output, confidence, coverage, severity and downloadable evidence

**CONCLUSION**
A reproducible academic prototype, not a certified road-safety system

**Visual:** include a simple architecture strip:

```text
Image / Video / Webcam
          ↓
 FastAPI + inference service
          ↓
 YOLO11n → boxes → shared severity
          ↓
 Annotated media + telemetry + SPA
```

---

## Slide 9 — Project details: modules

**Module 1 — Detection engine**
`detect.py`, `infrascan/inference.py`, `infrascan/severity.py`
Image/video/camera inference, annotation, severity, telemetry.

**Module 2 — Backend API**
`infrascan/api.py`, `config.py`, `jobs.py`
Routes, validation, model lifecycle, uploads, video jobs, security headers.

**Module 3 — Frontend dashboard**
`web/index.html`, `web/assets/js/*`, `web/assets/css/app.css`
Hash routing, inspection state, video polling, webcam dialog, system page, command palette.

**Module 4 — Model operations**
`train.py`, `evaluate.py`, `export.py`, `scripts/*`
Training, evaluation, export and dataset preparation.

**Visual:** four cards with file paths and one-line responsibilities.

---

## Slide 10 — Project details: explanation

**Image request**
Browser → `POST /api/detect/image` → validate thresholds/checkpoint → decode → YOLO prediction → severity → annotated JPEG → JSON response.

**Video request**
Upload → `202` job → background worker → frame loop → H.264 (or `mp4v` fallback) → progress polling → video + telemetry downloads.

**Shared severity**

```text
coverage = box area / frame area × 100
CRITICAL ≥ 8%   MODERATE ≥ 2.5%   MINOR < 2.5%
```

**Failure model**
Missing checkpoint → `503` / `unavailable`; invalid image → `400`; bad threshold → `422`; unsupported video → `415`; oversized upload → `413`.

**Visual:** two sequence strips (image, video) and one severity band.

**Key sentence:** *“The same severity function runs in the dashboard, CLI and camera HUD, so the label does not change when the interface changes.”*

---

## Slide 11 — Screenshots of result

Choose **two or three** images; do not paste four dense screenshots at full size.

**Option A — image inspection**
`../assets/screenshots/image-inspection.webp`
Caption: “Annotated image with confidence, severity and coverage; 1 MODERATE at 7.0% in the recorded sample.”

**Option B — video analysis**
`../assets/screenshots/video.webp`
Caption: “Background video job with progress, annotated MP4 and telemetry downloads.”

**Option C — system page**
`../assets/screenshots/system.webp`
Caption: “Runtime transparency: checkpoint, device, versions and shared severity policy.”

**Option D — overview**
`../assets/screenshots/overview.webp`
Caption: “Overview with model state, device and session scans.”

**Rules:** crop browser chrome if it adds noise; keep the actual UI; caption the evidence. Never use a generated mockup.

---

## Slide 12 — Conclusion and scope

**Conclusion**

- A YOLO11n checkpoint was integrated into a complete local inspection workflow.
- The accepted dashboard supports images, video jobs, webcam frames and system diagnostics.
- The verified test run achieved **P 0.7959 · R 0.5996 · mAP@0.50 0.7111 · mAP@0.50–0.95 0.3938** on **718 images / 994 instances**.
- The system is transparent about missing assets, blocked tests and heuristic severity.

**Future scope**

- Expand and license a representative dataset; add more road-surface classes.
- Calibrate confidence and severity against expert annotations.
- Add cross-frame tracking/deduplication and optional GPS metadata.
- Test CUDA, export targets and long-video throughput.
- Conduct a formal accessibility and security review before any wider deployment.

**Visual:** “Proven / Limited / Next” three-column close.

---

## Slide 13 — References

Use APA-style entries. Six to eight fit if the font is 16–18 pt. Recommended:

1. Redmon, J., Divvala, S., Girshick, R., & Farhadi, A. (2016). You only look once: Unified, real-time object detection. *CVPR*, 779–788.
2. Bochkovskiy, A., Wang, C.-Y., & Liao, H.-Y. M. (2020). YOLOv4: Optimal speed and accuracy of object detection. *arXiv:2004.10934*.
3. Maeda, H., Sekimoto, Y., Seto, T., Kashiyama, T., & Omata, H. (2018). Road damage detection using deep neural networks with images captured through a smartphone. *arXiv:1801.09450*.
4. Everingham, M., Van Gool, L., Williams, C. K. I., Winn, J., & Zisserman, A. (2010). The PASCAL Visual Object Classes (VOC) Challenge. *IJCV, 88*(2), 303–338.
5. Ultralytics. (n.d.). *YOLO documentation*. https://docs.ultralytics.com/
6. Ultralytics. (n.d.). *Licensing*. https://www.ultralytics.com/license
7. Bisman-Singh-Dev. (n.d.). *YOLO pothole detection model*. https://github.com/Bisman-Singh-Dev/YOLO-Pothole-Detection-Model

**Do not** add a reference you have not actually consulted.

---

## Slide 14 — Thank you

**Thank you**

Questions and demonstration

**Repository:** github.com/Aayush9808/InfraScan
**Documentation:** github.com/Aayush9808/InfraScan-Documentation

**Visual:** dashboard crop or project mark; keep the template footer.

**Close with:** “The contribution is the integration and evidence discipline around a compact detector, not a claim that YOLO was invented here.”

---

[Presentation index](README.md) · [Template audit](template-audit.md) · [Speaker notes](speaker-notes.md) · [Demo script](demo-script.md)
