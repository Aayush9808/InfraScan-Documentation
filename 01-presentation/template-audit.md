# GLBITM presentation template audit

**Source file:** `GLBITM PPT Format.pptx` (workspace root, not committed)
**Audited:** 22 October 2026
**Deck size:** 13.33 × 7.50 in (16:9), 14 slides
**Theme fonts:** Calibri Light (headings) and Calibri (body)
**Institutional footer:** “Department of Computer Science & Engineering – Artificial Intelligence”

The template is an academic shell. Keep its header, footer, slide order and restrained institutional styling. Replace the sample text and the sample literature row; do not redesign the deck into a product pitch.

## Slide map and constraints

| Slide | Template heading | What the template expects | InfraScan fill |
|---:|---|---|---|
| 1 | Title slide | Department, project title, students + roll numbers, supervisor + designation, session | InfraScan title; four team members; supervisor `[TO BE CONFIRMED]`; session 2026–2027 |
| 2 | Introduction of the project | Basic definition and foundational aspects | Local-first pothole inspection, inputs, outputs, one pipeline/two interfaces |
| 3 | Objectives | What the project does, modules covered, challenges solved/remaining | Five objectives + explicit non-goals |
| 4 | Hardware & software requirements | Specifications | Verified Python/torch/Ultralytics/OpenCV/FastAPI stack; CPU/MPS/CUDA; browser |
| 5 | Existing system / related work | Base work and related tools/technology | Upstream YOLO pothole project; Streamlit prototype replaced by FastAPI + SPA; what gap remains |
| 6 | Literature survey | Table with paper, author, method, conclusion, future work | 5–7 verified references in the template's table style |
| 7 | Motivation / problem statement | Why the existing work needs enhancement | Inconsistent manual inspection, lack of local tooling, weak evidence/UX in prototypes |
| 8 | Description of project (diagram) | Base work → modifications → expected result → conclusion | Four-box architecture/evolution diagram |
| 9 | Project details — modules | Module list | Detection, inference/service, API, frontend, ops/telemetry |
| 10 | Project details — explanation | Explanation of modules | Request flow, video job flow, severity policy, failure states |
| 11 | Screenshots of result | Real result screenshots | Overview, image inspection, video, system (choose 2–4) |
| 12 | Conclusion & scope | Conclusion and future scope | Verified outcome + limitations + future work |
| 13 | References | APA-style references | 6–8 entries, same style as the template |
| 14 | Thank you | Closing | Repositories, demo close, Q&A |

## Typography and layout rules

- **Title:** Calibri Light, large; keep the template's title position and department header.
- **Body:** Calibri, 20–28 pt for bullets; 16–18 pt for table cells.
- **Minimum on-slide font:** 16 pt. If it does not fit, cut text.
- **One idea per slide.** The slide is a visual aid; the explanation belongs in the speaker notes.
- **Use the template's footer** on every content slide.
- **Colour:** retain the institutional palette. A restrained teal accent is acceptable for InfraScan diagrams; do not replace the college identity with the dashboard's dark theme.
- **Images:** use screenshots with a thin border and a one-line caption that states the view and the fact it demonstrates. Do not stretch images out of aspect ratio.
- **Tables:** no more than four columns on slides. Move detailed tables to the report.
- **Equations:** only one equation on a slide (severity coverage), with a plain-language explanation.

## Mapping the current project to the template

The template says “mini project” and contains sample text from an unrelated IoT paper. Replace those placeholders completely:

- Title: **InfraScan — AI-Powered Road Inspection and Pothole Detection System**
- Subtitle: **Local-first YOLO11n detection with a FastAPI + SPA inspection workspace**
- Existing-system slide: distinguish the upstream detector from the accepted dashboard/architecture.
- Literature slide: use the verified table in [`literature-survey.md`](literature-survey.md), not the template's IoT example.
- Results slide: use only verified screenshots and the 718-image metrics.
- Conclusion: state clearly that the project is an academic decision-support prototype.

## Template QA checklist

- [ ] 14 slides retained (or a documented reason for merging).
- [ ] Department header and footer present.
- [ ] All four names and roll numbers spelled correctly.
- [ ] Supervisor/group ID filled or visibly marked `[TO BE CONFIRMED]`.
- [ ] No sample text from the template remains.
- [ ] Every screenshot has a caption and is from the accepted build.
- [ ] Metrics match [`../PROJECT_FACTS.md`](../PROJECT_FACTS.md).
- [ ] No slide claims an unexecuted test.
- [ ] References fit on one slide at readable size.
- [ ] Export checked as PDF on the presentation machine.
- [ ] Fonts available or embedded; no missing-glyph boxes.
- [ ] Videos/images play from the local machine without internet.

---

[Presentation index](README.md) · [Slide content](slide-by-slide-content.md) · [Speaker notes](speaker-notes.md) · [Documentation homepage](../README.md)
