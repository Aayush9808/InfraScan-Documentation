# Slide-by-slide content

## 1. Title slide

**Title:** InfraScan — AI-Powered Road Inspection and Pothole Detection System
**Presented by:** Aayush Kumar Shrivastav (2301921540002), Kajal Rawal (2301921520087), Khushi Shakya (2301921520091), Ronak Saini (2301921520158)
**Supervisor:** [TO BE CONFIRMED]
**Group ID:** [TO BE CONFIRMED]

Keep the title large and use the GLBITM department header. Do not add unverified supervisor details.

## 2. Introduction

InfraScan is a Python and Streamlit workflow for detecting potholes in images, uploaded videos, and camera inputs using an Ultralytics-based object-detection pipeline. It annotates detections and can produce telemetry outputs.

## 3. Objectives

- Detect pothole objects with bounding boxes and confidence scores.
- Support image, video, and camera-oriented workflows exposed by the source code.
- Present results through a dashboard.
- Provide structured output and useful failure messages.
- Document limitations and reproducibility requirements.

## 4. Hardware and software requirements

| Category | Verified requirement |
|---|---|
| OS | macOS was used for validation; cross-platform paths are preferred |
| Python | Python 3.10+ is stated by the project badge; confirm exact deployment version |
| Runtime | PyTorch, torchvision, Ultralytics, OpenCV, Pillow, NumPy, pandas, Streamlit |
| Model | A compatible, lawfully obtained checkpoint at `weights/best.pt` |
| Dataset | Required only for training/evaluation/sample workflows; excluded from release |
| Camera | Required for live camera testing |

Do not claim GPU requirements or performance without a measured experiment.

## 5. Existing system / related work

The work derives from the upstream YOLO Pothole Detection Model repository by Bisman Singh. InfraScan preserves that attribution and adds dashboard presentation, local video processing integration, release documentation, and validation work.

## 6. Literature survey

Use the table in [literature-survey.md](literature-survey.md). Keep source papers distinct from InfraScan results.

## 7. Motivation and problem statement

Manual road inspection is slow, inconsistent, and difficult to scale. InfraScan explores a visual-assistance workflow; it is not a certified road-management or safety system and does not replace human inspection.

## 8. Project description diagram

Show: input image/video/camera → validation and temporary handling → model loading → YOLO inference → boxes/confidence/severity → annotated output and telemetry → Streamlit display/download.

## 9. Project details

Modules: detection engine, Streamlit dashboard, training utility, evaluation utility, export utility, dataset preparation utilities, and camera utility.

## 10. Module explanations

Use the file map in [implementation-details.md](../03-project-documentation/implementation-details.md). Explain actual functions, not generic AI components.

## 11. Results screenshots

Insert only screenshots captured from the source application. At minimum label the checkpoint path, input type, date, and whether the result was reproduced.

## 12. Conclusion and scope

InfraScan demonstrates an integrated pothole-detection workflow. Future work includes verified dataset licensing, fresh evaluation, broader validation, calibration, and deployment testing.

## 13. References

Use the APA-style entries in [bibliography.md](../10-references-and-governance/bibliography.md).

## 14. Thank you

Use the template's closing slide and include the repository URL.
