# Poster layout and print guide

**Canvas:** 10.5 × 14 in portrait (GLBITM template)
**Target:** readable at 1 m, printable at A3/A2 or the department's required size
**Rule:** preserve the template's institutional header, footer and column identity. Adjust text, not the brand.

## 1. Page setup

- Open `GLBITM POSTER Format.pptx` directly; do not create a new document.
- Confirm **Design → Slide Size → Custom Slide Size**: 10.5 in × 14 in, portrait.
- Set margins: **0.45–0.55 in** on all sides. Keep text at least 0.35 in from the trim edge.
- Use a single slide. Do not add a second page.
- Keep the template's header band, section labels and footer line.
- If the print shop requires bleed, export at 100% with a 0.125 in bleed; do not scale the content unpredictably.

## 2. Grid

The template is a three-column layout with a full-width workflow band.

```text
┌──────────────────────────────────────────────────────────────┐
│  SESSION 2026–2027 · BATCH 2023 · FINAL YEAR PROJECT         │
├───────────────┬───────────────────────┬──────────────────────┤
│  GROUP ID     │  ABSTRACT             │  INTRODUCTION        │
│               │                       │                      │
├───────────────┼───────────────────────┼──────────────────────┤
│  METHODOLOGY  │  TITLE + TEAM         │  OBJECTIVES &        │
│               │  + SUPERVISOR         │  MOTIVATION          │
├───────────────┼───────────────────────┼──────────────────────┤
│  EXPERIMENTS  │  CONCLUSION           │  REFERENCES          │
│  & RESULTS    │                       │                      │
├───────────────┴───────────────────────┴──────────────────────┤
│  WORKFLOW DIAGRAM (full width)                               │
├──────────────────────────────────────────────────────────────┤
│  Institutional footer                                        │
└──────────────────────────────────────────────────────────────┘
```

The exact template places the title block in the upper centre/right. Keep the project title, four names, supervisor and group ID grouped so the reader can identify the work immediately.

## 3. Recommended sizes

| Element | Size | Weight / style |
|---|---:|---|
| Session line | 14–16 pt | Small caps or bold |
| Project title | 36–44 pt | Bold, Calibri Light |
| Subtitle | 20–24 pt | Regular |
| Team names | 20–22 pt | Regular; roll numbers 16–18 pt |
| Section headings | 24–28 pt | Bold |
| Body | 16–18 pt | Regular, 1.05–1.15 line spacing |
| Table body | 14–16 pt | Regular; header bold |
| Figure caption | 13–15 pt | Italic |
| References | 12–14 pt | Hanging indent if possible |
| Footer | 12–14 pt | Institutional line |

Do not go below **12 pt** anywhere. If the text does not fit at 16 pt body, cut words. A poster with 12 pt paragraphs is a document, not a poster.

## 4. Section word budgets

| Section | Budget | Priority |
|---|---:|---|
| Abstract | 90–120 words | Essential |
| Introduction | 80–110 words | Essential |
| Objectives & motivation | 4–6 bullets | Essential |
| Methodology | 90–120 words | Essential |
| Results | table + 2 short paragraphs | Essential |
| Conclusion | 60–90 words | Essential |
| References | 4–6 entries | Essential |
| Workflow | 5 stages + labels | Essential |
| Group ID / supervisor | short fields | Must be confirmed |

If space is tight, reduce the introduction and references before reducing body font size.

## 5. Figure treatment

### Screenshot

Use one dashboard screenshot, preferably **image inspection**:

- Crop to the result panel; remove irrelevant browser chrome.
- Keep the annotation, confidence, severity and coverage visible.
- Place on a white panel with a 0.5 pt border.
- Caption: “Annotated inspection result: bounding box, confidence, severity and coverage.”
- Do not stretch the image; preserve aspect ratio.

### PR curve

- Use `../assets/evaluation/pr-curve.png`.
- Place it at a width that keeps axis labels readable (approximately 3.0–3.6 in).
- Caption: “Precision–recall curve, 718-image test split; mAP@0.50 = 0.7111.”
- Do not overlay additional annotations that obscure the curve.

### Workflow diagram

- Draw in PowerPoint shapes, Figma, draw.io or Inkscape; export a PDF/PNG at 300 dpi.
- Five stages: Input → Validation → Inference → Severity/Annotation → Output.
- Use one accent colour for arrows; keep labels horizontal.
- Make the diagram span the full lower band without touching the footer.
- Ensure arrows do not cross text.

## 6. Writing style for a poster

- Use short sentences and concrete nouns.
- Prefer “718 images / 994 instances” over “a large dataset.”
- State the evaluation device when giving speed or accuracy.
- Put the limitation in the conclusion, not in a footnote.
- Use “prototype” or “decision-support” rather than “production-ready.”
- Avoid generic AI claims (“revolutionary”, “state-of-the-art”) unless a cited benchmark supports them.

## 7. Print and export checklist

- [ ] Confirm the department's required poster size; the template is 10.5 × 14 in.
- [ ] Replace group ID and supervisor fields.
- [ ] Spell-check all names, roll numbers and affiliations.
- [ ] Confirm every number against [`../PROJECT_FACTS.md`](../PROJECT_FACTS.md).
- [ ] Embed fonts: **File → Options → Save → Embed fonts** or export PDF with font embedding.
- [ ] Export **PDF/X-4** or a high-quality PDF; keep a PPTX source copy.
- [ ] Check at 100% zoom for text overflow.
- [ ] Print an A4 proof and read it at arm's length; if body text is hard to read, increase size and cut copy.
- [ ] Check colour in grayscale if the printer is monochrome.
- [ ] Confirm the poster does not rely on a QR code or internet link for the core content.
- [ ] Carry a PDF on a USB drive and in cloud storage for the viva.

## 8. Common failure modes

| Problem | Fix |
|---|---|
| Text overflows a column | Cut words; do not shrink below 16 pt body |
| Screenshot is pixelated | Use the original WebP/PNG, not a stretched screenshot |
| Title collides with header | Reduce title to two lines; keep the template header |
| Workflow unreadable | Use five large stages, not fifteen small boxes |
| References dominate | Keep four to six; move the full list to the report |
| Metrics look “too good” | Check device, split and threshold; use only the verified run |
| Group ID missing | Mark `[TO BE CONFIRMED]`; do not invent one |
| Print colours differ | Export PDF, check proof, avoid very dark backgrounds |

---

[Poster index](README.md) · [Template audit](template-audit.md) · [Poster copy](poster-content.md) · [Documentation homepage](../README.md)
