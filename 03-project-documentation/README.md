# Project documentation

This directory is the narrative and academic core of the InfraScan documentation. It explains **what the accepted system is, why it was built, how it works, what was verified, and what it cannot yet claim**.

The pages were rebuilt for the accepted **FastAPI + zero-build single-page application (SPA) + YOLO11n** implementation. Earlier revisions of this directory described a Streamlit prototype; those descriptions are obsolete and must not be used in the report, viva or poster.

> **Snapshot:** 22 October 2026. The authoritative fact register is [`../PROJECT_FACTS.md`](../PROJECT_FACTS.md). If a linked page elsewhere in this repository still reflects an earlier revision, `PROJECT_FACTS.md` prevails for factual values.

## Reading order

| # | Page | Question it answers | Best for |
|---:|---|---|---|
| 1 | [Project overview](project-overview.md) | What is InfraScan, who is it for, and what does it actually do? | First read; external readers; poster and presentation framing |
| 2 | [Problem statement and objectives](problem-statement-and-objectives.md) | What precise problem is solved, and how is success defined? | Faculty review; requirement traceability; viva |
| 3 | [Implementation details](implementation-details.md) | Which modules exist, how do they interact, and why were key decisions made? | Technical review; code walkthrough preparation |
| 4 | [Limitations](limitations.md) | What is unverified, weak, unsafe or out of scope? | Examiners; honest claims; risk planning |
| 5 | [Technical glossary](technical-glossary.md) | What do the detection, backend and media terms mean? | Revision; non-specialist readers |
| 6 | [Project report](project-report.md) | What is the complete academic report suitable for submission? | Faculty submission; full chapter-by-chapter account |

### Suggested paths

- **Examiner or faculty member:** overview → problem statement → report → limitations.
- **Technical reviewer:** implementation details → [architecture](../04-architecture/README.md) → [code walkthrough](code-walkthrough.md) → limitations.
- **Team member preparing for the viva:** glossary → implementation details → [viva preparation](../08-viva-preparation/README.md) → report Appendix B.
- **Reader checking a claim:** [`../PROJECT_FACTS.md`](../PROJECT_FACTS.md) → [test evidence](../09-testing-and-results/README.md) → limitations.

## What changed from the old pages

| Old description | Accepted description |
|---|---|
| Streamlit single-script dashboard (`app.py`) | FastAPI application (`infrascan/api.py`) serving a same-origin SPA |
| Dashboard reran the script for each interaction | Explicit HTTP API, background model load, background video jobs |
| Severity logic lived inside `detect.py` | Shared `infrascan/severity.py` imported by CLI and dashboard |
| No stable API surface | Documented REST endpoints, OpenAPI schema and Swagger UI |
| Unverified or inherited benchmark values | One reproducible 718-image evaluation with recorded environment |
| Generic "future work" limitations | Evidence-bounded PASS / FAIL / BLOCKED / NOT RUN register |

## Documentation conventions

- **Verified** means checked against the current source tree, the running application, the local checkpoint metadata, the local dataset or the recorded evaluation.
- **BLOCKED** means the check could not run in the available environment (for example, CUDA on an Apple M2 machine).
- **NOT RUN** means the capability exists in source but was not exercised during acceptance.
- **[TO BE CONFIRMED]** marks institutional or provenance data that the team has not supplied.
- Metrics are reported with the precision given in the evaluation record; they are not rounded up or presented as production accuracy.

## Related sections

- [Architecture](../04-architecture/README.md) — system, component and inference views.
- [Dataset and model](../05-dataset-and-model/README.md) — provenance, annotation, training and checkpoint facts.
- [Setup and execution](../06-setup-and-execution/README.md) — installation, configuration and usage.
- [Testing and results](../09-testing-and-results/README.md) — test strategy and evidence.
- [References and governance](../10-references-and-governance/README.md) — bibliography, attribution and licensing.
- [Contribution evidence](../CONTRIBUTIONS.md) — what Git history does and does not prove.
