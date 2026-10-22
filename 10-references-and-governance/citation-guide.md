# Citation guide

Ready-to-use citation formats for the project and everything it depends on, with the rules for citing a result rather than a tool.

- [Citing the project](#citing-the-project)
- [Citing the upstream project](#citing-the-upstream-project)
- [Citing the model and the dataset](#citing-the-model-and-the-dataset)
- [Citing the evaluation result](#citing-the-evaluation-result)
- [Citation keys and where they live](#citation-keys-and-where-they-live)
- [Rules that prevent citation errors](#rules-that-prevent-citation-errors)
- [Related pages](#related-pages)

---

## Citing the project

### APA 7

> Shrivastav, A. K., Rawal, K., Shakya, K., & Saini, R. (2026). *InfraScan: AI-powered road inspection and pothole detection system* [Source code and documentation]. GitHub. <https://github.com/Aayush9808/InfraScan>

### IEEE

> A. K. Shrivastav, K. Rawal, K. Shakya, and R. Saini, "InfraScan: AI-powered road inspection and pothole detection system," GitHub, 2026. [Online]. Available: https://github.com/Aayush9808/InfraScan

### BibTeX

```bibtex
@software{infrascan2026,
  author    = {Shrivastav, Aayush Kumar and Rawal, Kajal and Shakya, Khushi and Saini, Ronak},
  title     = {{InfraScan}: AI-Powered Road Inspection and Pothole Detection System},
  year      = {2026},
  version   = {2.0.0},
  url       = {https://github.com/Aayush9808/InfraScan},
  license   = {MIT},
  note      = {Source code and documentation}
}
```

### As a technical report

> A. K. Shrivastav, K. Rawal, K. Shakya and R. Saini, *InfraScan: AI-Powered Road Inspection and Pothole Detection System — Final Year Project Report*, G. L. Bajaj Institute of Technology & Management, 2026.

**Placeholders to fill before submission:** the group ID, the supervisor's name and designation, and the department's official report-number format.

---

## Citing the upstream project

The upstream project by Bisman Singh is the foundation of the pothole-detection model and training pipeline. Any report that describes the work as newly conceived must credit it.

> Bisman-Singh-Dev. (n.d.). *YOLO pothole detection model* [Source code]. GitHub. <https://github.com/Bisman-Singh-Dev/YOLO-Pothole-Detection-Model>

If a comparison is drawn, use the neutral form: *"Building on the upstream pothole-detection project by Bisman Singh, this release replaces the Streamlit prototype with a FastAPI backend and a custom dashboard…"* — not *"our YOLO model"*.

---

## Citing the model and the dataset

### The model

The checkpoint is **not redistributed** by this project, and its provenance is unresolved. That constrains the citation:

- **Do not** cite a model card, DOI or dataset record for `weights/best.pt`. None exists.
- **Do** cite the architecture family: Ultralytics YOLO11 — <https://docs.ultralytics.com/models/yolo11/>.
- **Do** state the checkpoint identity: `weights/best.pt`, SHA-256 `f3490f953b4ac2aa0205c206c151be8f2da281da99708713b182492a43c7751c`, with the note that it is excluded from distribution and that its redistribution permission is unresolved.

### The dataset

The same rule applies with less room:

- The local dataset's original creators, source URLs and licences are **not established**. The documentation therefore describes the *local copy* — structure, class, counts, split sizes — and does not claim provenance.
- Until provenance is resolved, the honest citation form is:

> *A local pothole image collection (4,719 images; 7,340 annotated instances; single class `pothole`), prepared and split by the project team. The upstream source and licence terms remain unverified and are recorded as an open item in the project documentation.*

- If an examiner asks for a dataset citation, say plainly: *"The dataset provenance is unresolved — the source is inherited from the upstream project and the licence has not been confirmed. It is excluded from the repository for that reason."* That answer is stronger than a plausible-looking citation.

---

## Citing the evaluation result

A result is citable only together with the conditions that produced it. The citable unit is:

> *YOLO11n checkpoint (SHA-256 `f3490f95…3c7751c`) evaluated on a held-out 718-image test split (994 pothole instances, 2 background images) at 640 px and batch 16 on Apple M2 with MPS: precision 0.7959, recall 0.5996, mAP@0.50 0.7111, mAP@0.50–0.95 0.3938. Python 3.12.14, PyTorch 2.14.1, Ultralytics 8.4.183.*

**Never cite the metrics without the environment.** A number detached from the checkpoint, the split and the device is not reproducible and invites the question "on what?".

**Never cite the timing without the batch size.** The 3.82 ms inference figure is amortised over a batch of 16; quoting it as a single-image latency misrepresents it by roughly an order of magnitude. The end-to-end warm request time for a single image was 66.2 ms.

---

## Citation keys and where they live

| Key | Used in | Appears in |
|---|---|---|
| `[1]`–`[24]` | The project report | The report's own reference list, mirrored as Section A of the [bibliography](bibliography.md) |
| `PROJECT_FACTS.md` | Every chapter that states a fact | The canonical fact sheet |
| `CONTRIBUTIONS.md` | Team and contribution statements | The role-versus-authorship matrix |
| `TC-01`–`TC-25` | Testing chapters | The [manual test log](../09-testing-and-results/manual-test-log.md) |
| `SEC-01`–`SEC-18` | Security and quality chapters | The [security and quality index](../11-security-and-quality/README.md) |
| `ACC-01`–`ACC-10` | Accessibility findings | [Accessibility](../11-security-and-quality/accessibility.md) |

When a new numbered reference is added to the report, add it to Section A of the bibliography in the same change, and renumber nothing silently.

---

## Rules that prevent citation errors

1. **Cite the source, not the summary.** If a claim comes from a blog post about a paper, cite the paper.
2. **Do not cite what you did not run.** A citation can establish that a technique exists; only an execution can establish that it works here.
3. **Distinguish inherited from produced.** The detector architecture, the dataset and the training script are inherited; the FastAPI service, the SPA, the shared severity policy, the job system, the telemetry and the evaluation are produced by this team.
4. **No citation laundering.** Restating a claim does not verify it. If the project did not test it, the text says the project did not test it — the [bibliography](bibliography.md#section-c--references-the-project-deliberately-does-not-make) lists the tempting citations that are deliberately absent.
5. **Attribute the code you reused.** The upstream project is credited in the code repository's licence notice and in every document that describes the lineage.
6. **Use "adapted from" precisely.** The current release is adapted from the upstream project, not a clean-room implementation of it.
7. **When in doubt, cite the repository.** A commit hash is a stronger citation than a URL that can change.

---

## Related pages

- [Bibliography](bibliography.md) — the full reference list.
- [Third-party attribution](third-party-attribution.md) — the notices the project must carry.
- [Licences and assets](licences-and-assets.md) — the licence matrix and redistribution position.
- [Verified facts](../PROJECT_FACTS.md) — the numbers the citation formats above quote.
- [Verified results](../09-testing-and-results/verified-results.md) — the result being cited.
