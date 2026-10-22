# Third-party attribution

Everything InfraScan is built on, and the notice each item requires. This is the reference to check before a submission, a release or a paper.

> **Governing rule:** the repository's MIT licence covers the project code. It does **not** automatically cover the dataset, the trained checkpoint, the upstream project's material, the fonts, the institutional templates or the Python dependencies. Each is handled separately below.

- [1. Upstream project](#1-upstream-project)
- [2. Ultralytics YOLO](#2-ultralytics-yolo)
- [3. The trained checkpoint](#3-the-trained-checkpoint)
- [4. The dataset](#4-the-dataset)
- [5. Python dependencies](#5-python-dependencies)
- [6. Bundled fonts](#6-bundled-fonts)
- [7. Interface assets](#7-interface-assets)
- [8. Institutional templates](#8-institutional-templates)
- [9. Standards and specifications](#9-standards-and-specifications)
- [10. Documentation sources](#10-documentation-sources)
- [11. Evidence assets](#11-evidence-assets)
- [Compliance checklist](#compliance-checklist)
- [Related pages](#related-pages)

---

## 1. Upstream project

**Component:** the original pothole-detection project that this work is adapted from.
**Upstream author:** Bisman Singh.
**Upstream repository:** <https://github.com/Bisman-Singh-Dev/YOLO-Pothole-Detection-Model>
**Licence:** MIT.
**Notice status:** **Preserved.** The upstream `LICENSE` and copyright notice are retained in the code repository.

The upstream project provides the pothole-detection foundation: the dataset preparation scripts, the VOC ingestion utility, the training and evaluation wrappers, the detection loop and the original Streamlit dashboard.

The current release replaces the Streamlit interface with a FastAPI backend and a custom dashboard, adds background video jobs, a shared severity policy, telemetry export, upload validation, security headers, runtime diagnostics and a documented evaluation. It is therefore an **adapted and extended** work, not an independent reimplementation, and is described that way throughout the documentation.

**Required in any derivative publication:** retain the upstream copyright and MIT notice.

---

## 2. Ultralytics YOLO

**Component:** the `ultralytics` Python package, the YOLO11 model family and the pretrained `yolo11n.pt` weights used for fine-tuning.
**Licence:** **AGPL-3.0**, with a commercial Enterprise licence as an alternative, per the [Ultralytics licensing page](https://www.ultralytics.com/license).
**Use in this project:** the training and inference library, the model architecture, and the pretrained starting weights for fine-tuning.

Consequences the project accepts and documents:

- The AGPL obligations attach to the **Ultralytics components themselves**, which the project consumes as a dependency rather than vendoring. The `ultralytics` package is installed from PyPI, not copied into the repository.
- The **trained checkpoint** is a separate question from the library's licence and is treated conservatively in [section 3](#3-the-trained-checkpoint).
- The project does not redistribute any Ultralytics-provided weights, and the CLI's historical fallback to `yolo11s.pt` causes a download from Ultralytics at runtime rather than a bundled copy.

**If a commercial product or a closed-source release is ever proposed,** the AGPL implications of the training and inference toolchain must be reviewed before distribution. That review is outside the scope of an academic project and is flagged as an open item.

---

## 3. The trained checkpoint

**Component:** `weights/best.pt` — the fine-tuned YOLO11n pothole detector used for every result in this documentation.
**Size:** 5.2 MB. **SHA-256:** `f3490f953b4ac2aa0205c206c151be8f2da281da99708713b182492a43c7751c`.

**Status: not redistributed.** The file is excluded from the repository by policy.

The reasons are cumulative, and each is independently sufficient to justify exclusion:

1. The checkpoint was produced with AGPL-covered tooling, and the terms for redistributing a *trained artefact* are a separate question from the terms for the library.
2. The training data's licence and redistribution rights are **unresolved** (section 4). Redistribution of a model trained on data of unknown provenance cannot be assumed lawful.
3. The upstream provenance recorded in the checkpoint metadata is a lead, not a verified chain of custody.

The SHA-256 is published so that a specific file can be **identified and integrity-checked**. It is not a signature, a licence or a grant of permission. Anyone wanting to reproduce the results needs to supply a compatible checkpoint and the dataset, or accept that the numbers are not reproducible from the public repository alone.

---

## 4. The dataset

**Component:** the local pothole image collection — 4,719 images, 7,340 annotated instances, one class, split into train/validation/test.
**Status: not redistributed.** The `dataset/` directory is excluded from the repository by policy.

**What is known:**

- The local structure, counts, class definition and label format are verified.
- The upstream repository includes utilities for ingesting a VOC-style dataset and for converting masks into bounding boxes, which is consistent with a multi-source public collection.
- The checkpoint metadata records a training-data path pointing at an upstream Bisman Singh directory.

**What is not known:**

- The original creators of the images, their stable URLs, and the licence of each source collection.
- Whether every local image came from the sources named in the upstream material.
- The split-generation history, and therefore whether near-duplicate images straddle the train/val/test boundary.

Until these are resolved, the dataset is **not** described as freely reusable, and the documentation says exactly that. See [dataset provenance](../05-dataset-and-model/dataset-provenance.md) and [missing information](../MISSING_INFORMATION.md).

---

## 5. Python dependencies

Installed from PyPI rather than vendored; each retains its own licence.

| Package | Licence | Used for |
|---|---|---|
| `ultralytics` | AGPL-3.0 / Enterprise | Training, inference, validation, export |
| `torch`, `torchvision` | BSD-3-Clause | Tensors, device execution, MPS/CUDA backends |
| `opencv-python` | Apache-2.0 (OpenCV 4.5+) | Decoding, drawing, video capture and encoding |
| `pillow` | MIT-CMU | Image handling |
| `numpy` | BSD-3-Clause | Arrays and numerical operations |
| `pandas` | BSD-3-Clause | Telemetry tables and CSV output |
| `matplotlib`, `seaborn` | Matplotlib (PSF-based) / BSD-3-Clause | Evaluation plots, including the PR curve |
| `pyyaml` | MIT | Reading `data.yaml` and configuration |
| `tqdm` | MPL-2.0 / MIT | Progress reporting |
| `fastapi` | MIT | HTTP API, validation, OpenAPI generation |
| `uvicorn[standard]` | BSD-3-Clause | ASGI server |
| `python-multipart` | Apache-2.0 | Multipart form parsing |

The authoritative licence for any package is the one in its own distribution. This table is a summary for orientation, not a substitute for it.

**Notice status:** no dependency source is vendored into the repository, so no dependency's notice file needs to be carried in the source tree. A binary redistribution (a container image, an installer) would require reproducing the notices, and that is flagged in [licences and assets](licences-and-assets.md#before-a-public-release).

---

## 6. Bundled fonts

**Component:** the two web fonts served with the dashboard.
**Licence:** SIL Open Font License 1.1 for both.
**Notice status:** **Preserved.** The licence texts are bundled next to the font files.

| Font | Use | Licence file |
|---|---|---|
| Inter | Body text | `web/assets/fonts/OFL-Inter.txt` |
| Space Grotesk | Display headings | `web/assets/fonts/OFL-SpaceGrotesk.txt` |

The OFL permits bundling and redistribution, including in a commercial product, provided the licence and the reserved font names are included. Both conditions are met by shipping the licence files alongside the `.woff2` files. The fonts are bundled locally rather than fetched from a CDN so the application works offline and the Content Security Policy can stay restrictive.

---

## 7. Interface assets

- **Icons** are original inline SVG symbols defined in `web/index.html`. No icon font, no third-party sprite and no external image service is used.
- **Visual design** — the dark glass surfaces, severity palette, layout and typography scale — is original to this project. The severity colours are project-defined semantics, not borrowed from a vendor design system.
- **No CDN, no web font service, no analytics, no telemetry SDK and no third-party script** is loaded by the dashboard. The Content Security Policy reflects that: the application is self-contained.

---

## 8. Institutional templates

**Component:** the GLBITM presentation and poster templates used to produce the academic deliverables.
**Status: not committed to the repository.** The template binaries live outside the documentation repository and remain subject to the institution's usage rules.

The [presentation guide](../01-presentation/README.md) and [poster guide](../02-poster/README.md) contain the audited structure, the content and the layout instructions, so the deliverables can be rebuilt in the official template without the template file being redistributed. Only the institutional header, footer and section structure are reproduced in the documentation; no template artwork is embedded.

---

## 9. Standards and specifications

- **ASTM D6433-20** — *Standard practice for roads and parking lots pavement condition surveys*. Referenced to explain how professional severity judgements are structured. The standard is **cited, not reproduced**, and InfraScan is not claimed to comply with it.
- **ISO/IEC 25010:2023** — the software product quality model. Used as a framework for separating functional suitability from performance, usability, security, maintainability and portability in the quality discussion. Cited, not reproduced; no certification is claimed.
- **RFC 9110** — *HTTP Semantics*. Referenced for the status-code behaviour implemented in the API.

---

## 10. Documentation sources

The project documentation is written from the source tree, the recorded checks and the project report. It draws on the following public documentation for the frameworks it uses, and cites them where the behaviour originates:

- FastAPI, Starlette and Uvicorn documentation.
- Ultralytics documentation, including the YOLO11 model page and the licensing page.
- PyTorch documentation, including the MPS backend notes.
- OpenCV `cv::VideoWriter` reference.
- Mozilla Developer Network — JavaScript modules and Content Security Policy.
- W3C — Web Content Accessibility Guidelines (WCAG) 2.2 and the ARIA Authoring Practices Guide.

These are cited as *documentation*, not as evidence of this project's behaviour. Where the project states what a component does, that statement comes from the installed version recorded in [`../PROJECT_FACTS.md`](../PROJECT_FACTS.md).

---

## 11. Evidence assets

| Asset | Origin | Notice status |
|---|---|---|
| `assets/screenshots/overview.webp` | Captured from the running dashboard | Original to the project |
| `assets/screenshots/image-inspection.webp` | Captured from the running dashboard | Original to the project |
| `assets/screenshots/video.webp` | Captured from the running dashboard | Original to the project |
| `assets/screenshots/system.webp` | Captured from the running dashboard | Original to the project |
| `assets/evaluation/pr-curve.png` | Generated by the evaluation run (Ultralytics plotting) | Produced by running the AGPL-covered toolchain; retained as evidence of this project's own evaluation |

The PR curve is the output of a tool, not third-party artwork, and it documents the project's own run.

---

## Compliance checklist

Run this before a submission, a release or a publication:

- [ ] The upstream MIT copyright notice is present in the distributed code.
- [ ] The font OFL files travel with the font files.
- [ ] No Ultralytics-provided weights are redistributed.
- [ ] The trained checkpoint is not redistributed, or its permissions have been confirmed in writing.
- [ ] The dataset is not redistributed, or its licences have been confirmed per source collection.
- [ ] The institutional templates are not redistributed.
- [ ] The group ID, supervisor name and designation are filled in, or the placeholders are clearly marked.
- [ ] Any accuracy claim carries the checkpoint hash, the split, the environment and the device.
- [ ] The limitations section is present in whatever is submitted — a result without its boundaries is misleading.

---

## Related pages

- [Licences and assets](licences-and-assets.md) — the licence matrix and redistribution position.
- [Bibliography](bibliography.md) — the reference list.
- [Citation guide](citation-guide.md) — how to cite the project and its dependencies.
- [Acknowledgements](acknowledgements.md).
- [Dataset provenance](../05-dataset-and-model/dataset-provenance.md) — what is known about the data.
