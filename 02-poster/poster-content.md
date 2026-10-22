# Poster copy

Final copy for the GLBITM poster. Replace the bracketed academic fields before printing. Keep the template's header, column structure and institutional footer.

---

## Header

**SESSION (2026–2027) · BATCH 2023**
**Final Year Project Evaluation**

---

## Title block

**InfraScan — AI-Powered Road Inspection and Pothole Detection System**

**Team Members**
Aayush Kumar Shrivastav (2301921540002)
Kajal Rawal (2301921520087)
Khushi Shakya (2301921520091)
Ronak Saini (2301921520158)

**Supervisor:** `[TO BE CONFIRMED]`
**Group ID:** `[TO BE CONFIRMED]`

---

## Abstract

*Target: 90–120 words.*

> Road-surface inspection is repetitive, visual and difficult to scale manually. InfraScan is a local-first pothole inspection prototype that combines a compact Ultralytics YOLO11n detector with a FastAPI backend and a browser dashboard. Users inspect still images, dashcam-style video or browser-captured frames; the system returns annotated media, bounding boxes, confidence, frame-coverage severity and downloadable telemetry. A fresh evaluation on a 718-image test split produced precision 0.7959, recall 0.5996, mAP@0.50 0.7111 and mAP@0.50–0.95 0.3938 on an Apple M2 using MPS. The project is an academic decision-support prototype, not a certified road-safety instrument; severity is a geometric heuristic, and dataset/checkpoint permissions remain unresolved.

---

## Introduction

*Target: 80–110 words.*

> Potholes are a visible road-maintenance problem, but a detector alone is not an inspection workflow. Users need to see where the model fired, how confident it is, how much of the frame is affected, and what evidence can be exported. InfraScan addresses that gap by integrating detection, severity scoring, annotation and telemetry in one locally hosted application. The browser interface supports image upload, dataset samples, webcam frames and background video jobs. The same Python severity policy is shared by the dashboard, command-line tools and camera HUD, so a result keeps the same label when the interface changes. The system deliberately exposes missing checkpoints, invalid inputs and untested capabilities instead of hiding them.

---

## Objectives & Motivation

*Target: 4–6 bullets.*

- Detect potholes in images, video frames and browser-captured frames with a compact single-class detector.
- Turn bounding boxes into a transparent, shared **coverage-based severity** signal.
- Provide a local-first dashboard for inspection, annotation review and telemetry download.
- Keep runtime evidence visible: checkpoint, device, model state and versions.
- Evaluate on a recorded test split instead of relying on inherited metrics.
- **Non-goal:** depth/volume measurement, GPS tracking, autonomous maintenance decisions or certified safety accuracy.

---

## Methodology

*Target: 90–120 words.*

> A YOLO11n checkpoint (2.58M parameters, 6.4 GFLOPs, 5.2 MB) performs single-class pothole detection at 640 px. The FastAPI backend validates uploads and thresholds, loads the model lazily during application lifespan, and serialises prediction calls around one Ultralytics model instance. Image requests return an annotated JPEG and a JSON detection list. Video requests become background jobs with progress polling, an annotated MP4 (H.264 preferred, `mp4v` fallback) and CSV/JSON telemetry. Severity is calculated as box area divided by frame area: **CRITICAL ≥ 8%**, **MODERATE ≥ 2.5%**, **MINOR < 2.5%**. The frontend is a zero-build vanilla ES-module SPA served by the same FastAPI process. No cloud service is required.

---

## Workflow diagram

Create a horizontal five-stage graphic in the lower band.

```text
[ Input ]
Image · video · webcam frame
        ↓
[ Validation ]
format · size · confidence · IoU · checkpoint
        ↓
[ YOLO11n inference ]
boxes · confidence · class
        ↓
[ Shared severity + annotation ]
coverage % · band · annotated media · telemetry
        ↓
[ Output ]
dashboard · CSV/JSON · MP4 · API response
```

Add one feedback arrow from **Output** back to **Inspection** labelled “review / re-run”. Keep arrows thick enough to read at print size. Do not embed a screenshot in this band.

---

## Experimentation & Results

*Use a compact table and one figure. Do not add rows that are not verified.*

| Metric | Verified result |
|---|---:|
| Test images | 718 |
| Test instances | 994 |
| Background images | 2 |
| Precision | 0.7959 |
| Recall | 0.5996 |
| mAP@0.50 | 0.7111 |
| mAP@0.50–0.95 | 0.3938 |
| Inference speed | 3.82 ms/image |
| Preprocess / postprocess | 0.17 / 10.75 ms/image |

**Environment:** Apple M2, macOS 27.0.1, Python 3.12.14, PyTorch 2.14.1, Ultralytics 8.4.183, MPS, batch 16, 640 px.

**Functional checks:** image inference produced one moderate detection at 7.0% coverage; invalid image returned HTTP 400; missing checkpoint produced an unavailable state; a 24-frame video produced 28 frame-level detections and H.264 playback; responsive layouts were checked at 390, 820, 1024 and 1440 px.

**Figure:** use `../assets/evaluation/pr-curve.png` at a legible size. Caption: “Precision–recall curve for the verified 718-image test run; mAP@0.50 = 0.7111.”

**Do not add:** CUDA FPS, export benchmarks, physical-webcam results or a second dataset’s accuracy.

---

## Conclusion

*Target: 60–90 words.*

> InfraScan demonstrates that a compact YOLO11n detector can be integrated into a complete local inspection workflow with transparent severity, annotated evidence and downloadable telemetry. The verified test result is useful but limited: recall is about 60%, severity is heuristic, and the evaluation is MPS-only. Dataset and checkpoint rights, CUDA performance, export targets and physical-camera behaviour remain open. The project is therefore a reproducible academic prototype and a foundation for further data, calibration and deployment work — not a certified road-safety product.

---

## References

*Use 4–6 entries at 12–14 pt. These fit the template.*

1. Redmon, J., Divvala, S., Girshick, R., & Farhadi, A. (2016). You only look once: Unified, real-time object detection. *CVPR*, 779–788.
2. Bochkovskiy, A., Wang, C.-Y., & Liao, H.-Y. M. (2020). *YOLOv4: Optimal speed and accuracy of object detection*. arXiv:2004.10934.
3. Maeda, H., Sekimoto, Y., Seto, T., Kashiyama, T., & Omata, H. (2018). *Road damage detection using deep neural networks with images captured through a smartphone*. arXiv:1801.09450.
4. Everingham, M., Van Gool, L., Williams, C. K. I., Winn, J., & Zisserman, A. (2010). The PASCAL Visual Object Classes (VOC) challenge. *IJCV, 88*(2), 303–338.
5. Ultralytics. (n.d.). *YOLO documentation*. https://docs.ultralytics.com/
6. Bisman-Singh-Dev. (n.d.). *YOLO pothole detection model*. https://github.com/Bisman-Singh-Dev/YOLO-Pothole-Detection-Model

---

## Footer

Keep the template's institutional line. Add, if space permits:

> Code: github.com/Aayush9808/InfraScan · Documentation: github.com/Aayush9808/InfraScan-Documentation

---

[Poster index](README.md) · [Template audit](template-audit.md) · [Layout guide](poster-layout-guide.md) · [Verified results](../09-testing-and-results/verified-results.md)
