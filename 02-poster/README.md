# Poster preparation

This section prepares a one-page **GLBITM poster** for InfraScan. The supplied template is a **10.5 × 14 inch portrait** layout with nine content regions. The goal is a readable conference-style poster, not a shrunken report.

## Poster files

1. [Template audit](template-audit.md) — dimensions, regions, typography and constraints.
2. [Poster copy](poster-content.md) — final text for each region, with word-count guidance.
3. [Layout and print guide](poster-layout-guide.md) — grid, font sizes, figure treatment, print checklist.

## Poster story in one sentence

> A compact YOLO11n detector is wrapped in a local FastAPI + browser inspection workspace that turns pothole boxes into annotated media, coverage-based severity and downloadable evidence, with a verified 718-image test result.

## Evidence to show

| Claim | Poster element |
|---|---|
| Model and task | “YOLO11n · single-class pothole detection” |
| Verified accuracy | Precision 0.7959 · Recall 0.5996 · mAP@0.50 0.7111 · mAP@0.50–0.95 0.3938 |
| Test set | 718 images / 994 instances / 2 background images |
| Runtime | Apple M2, MPS, 640 px, batch 16 |
| Product | One dashboard screenshot plus one workflow diagram |
| Honesty | “Academic prototype; not a certified road-safety instrument” |

## Do not put on the poster

- CUDA or NVIDIA performance numbers.
- ONNX/TensorRT/CoreML export claims.
- Physical-webcam test claims.
- Dataset/checkpoint redistribution claims.
- A wall of references; four to six are enough.
- Paragraphs copied from the report.

## Before printing

- Replace `[TO BE CONFIRMED]` group ID and supervisor fields.
- Proofread all names and roll numbers.
- Check the poster at 100% print size, not only on screen.
- Export a PDF with fonts embedded.
- Print a small proof or view at 50% to check readability.
- Keep a digital copy for the viva.

---

[Documentation homepage](../README.md) · [Presentation guide](../01-presentation/README.md) · [Verified facts](../PROJECT_FACTS.md) · [Verified results](../09-testing-and-results/verified-results.md)
