# Presentation preparation

This section turns the verified project into a **14-slide GLBITM presentation** and a short live demonstration. It follows the supplied `GLBITM PPT Format.pptx` rather than replacing its institutional identity.

## Start here

1. Read [the template audit](template-audit.md) — slide count, dimensions, typography and section order.
2. Fill the deck from [slide-by-slide content](slide-by-slide-content.md).
3. Rehearse with [speaker notes](speaker-notes.md).
4. Practise the [demo script](demo-script.md) and prepare the fallback.
5. Keep [the literature survey](literature-survey.md) separate from the project's own results.

## The one-sentence story

> InfraScan turns a YOLO11n pothole detector into a local inspection workspace: upload an image or dashcam clip, get annotated results and coverage-based severity, and inspect the same evidence in a FastAPI + SPA interface or from the CLI.

## Evidence rules for the presentation

- Use **0.7959 / 0.5996 / 0.7111 / 0.3938** only for the verified 718-image test run.
- State that the evaluation was on **Apple M2 / MPS**, batch 16, 640 px.
- Do not present CUDA, TensorRT, ONNX or physical-webcam results as tested.
- Do not show the dataset or checkpoint as redistributed; both are excluded and their permissions are unresolved.
- Do not attribute the whole implementation to every team member. Use [`../CONTRIBUTIONS.md`](../CONTRIBUTIONS.md).
- Prefer screenshots from `../assets/screenshots/`; they are captures of the accepted application.

## Slide deck at a glance

| Slide | Template section | InfraScan content |
|---:|---|---|
| 1 | Title | Project title, team, supervisor, session |
| 2 | Introduction | What the system does and why it is local-first |
| 3 | Objectives | Primary, secondary and non-goals |
| 4 | Hardware & software | Verified runtime, model, dataset, browser |
| 5 | Existing system / related work | Upstream YOLO project + gap addressed |
| 6 | Literature survey | YOLO, road-damage and dataset references |
| 7 | Motivation / problem statement | Inspection inconsistency, scale, evidence gap |
| 8 | Description of project (diagram) | Base work → modifications → result |
| 9 | Project details — modules | Detection, backend, SPA, operations |
| 10 | Project details — explanation | Request/job flows and shared severity |
| 11 | Screenshots of result | Real dashboard, video, system, metrics |
| 12 | Conclusion & scope | What is proven, what is limited, future work |
| 13 | References | APA-style, verified links |
| 14 | Thank you | Repository + demo close |

## What not to put on slides

- Full code listings.
- Long paragraphs copied from the report.
- Fake dashboards, invented metrics or unlabelled dataset images.
- A claim that the model is “production-ready” or “certified.”
- A claim that the system measures pothole depth, ride quality or structural risk.
- A claim that the dataset/checkpoint is open-source or redistributable.

## Presentation links

- [Template audit](template-audit.md)
- [Slide-by-slide content](slide-by-slide-content.md)
- [Speaker notes](speaker-notes.md)
- [Demo script](demo-script.md)
- [Literature survey](literature-survey.md)
- [Project report](../03-project-documentation/project-report.md)
- [Verified results](../09-testing-and-results/verified-results.md)

---

[Documentation homepage](../README.md) · [Poster guide](../02-poster/README.md) · [Viva preparation](../08-viva-preparation/README.md)
