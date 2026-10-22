# Acknowledgements

InfraScan exists because several people and projects made the work possible, and one of them made it possible before the team existed.

---

## The upstream project

The pothole-detection foundation of this work — the dataset preparation and VOC ingestion utilities, the training and evaluation wrappers, the detection loop and the original dashboard — comes from **Bisman Singh's** open-source pothole detection project, released under the MIT licence.

That project is the reason a student team could start from a working single-class pothole detector rather than from an empty repository. The current release is an adaptation and an extension of it, not a claim to have invented it. The upstream copyright and licence notice is preserved in the code repository, and the lineage is stated in the [project report](../03-project-documentation/project-report.md), the [architecture chapter](../04-architecture/README.md) and the [third-party attribution](third-party-attribution.md).

---

## The institution

**G. L. Bajaj Institute of Technology & Management**, Department of Computer Science & Engineering, Artificial Intelligence — for the academic framework, the final-year project structure, the review process and the institutional templates used for the presentation and poster deliverables.

The **project supervisor** is acknowledged here as a role. The supervisor's name and designation are recorded as `[TO BE CONFIRMED]` across the documentation and must be filled in before submission.

---

## The project team

| Member | Roll number | Acknowledged for |
|---|---|---|
| **Aayush Kumar Shrivastav** | 2301921540002 | Backend architecture and implementation, the FastAPI service, the inference lifecycle, the shared severity policy, the video job system, the custom dashboard, the evaluation run, and the documentation system |
| **Kajal Rawal ("Lucky")** | 2301921520087 | Testing and verification work, evidence discipline, the test log and evaluation records, and presentation and documentation coordination |
| **Khushi Shakya** | 2301921520091 | The frontend single-page application, the interface modules and styling, the accessibility and responsive review, and the API integration layer |
| **Ronak Saini** | 2301921520158 | Dataset preparation and annotation workflow, training and evaluation support, the model and dataset documentation, and the reference and attribution work |

**An important qualification.** These contributions are recorded from the team's own understanding of who did what. Independently verifiable Git authorship currently covers Aayush's commits; the individual contributions of the other three members are **team-declared** and are marked as such in [the contribution matrix](../CONTRIBUTIONS.md). The acknowledgements are written with that distinction in view rather than presenting a single document as proof.

---

## Upstream authors and maintainers

- **The authors of the research this project builds on** — the YOLO family of detectors, Faster R-CNN, the road-damage detection work on smartphone imagery, the PASCAL VOC and COCO evaluation protocols, and the augmentation and scheduling methods used during training. These are listed in full in the [bibliography](bibliography.md) and are what make the detector a detector rather than a demonstration.
- **The maintainers of Ultralytics YOLO**, FastAPI, Starlette, Uvicorn, PyTorch, OpenCV, NumPy, pandas, Matplotlib, seaborn, Pillow, PyYAML and tqdm — the frameworks the application is assembled from. Their work is what made a working inspection system feasible within an academic project.
- **The authors of the Inter and Space Grotesk typefaces**, whose SIL Open Font License makes the dashboard's typography possible under conditions the project meets.
- **The contributors to the W3C and Mozilla documentation** on web components, modules, Content Security Policy and accessibility, which informed the interface and the security review.

---

## The people who made the evidence discipline possible

Much of the value of this project is in what it *refuses* to claim. That comes from a project culture of recording the environment, the command and the limitation for every result, and of marking a check as blocked or not run rather than quietly promoting it.

The verification work — the 718-image evaluation with its recorded environment, the manual test log, the evidence index and the honest limitations section — is what turns a demo into something that can be defended in a viva. That discipline is credited to the whole team, and particularly to the member responsible for testing and evidence, who insisted that unrun checks stay unrun.

---

## Sources of data

The pothole imagery used to train and evaluate the model was inherited from the upstream project and prepared locally by the team. The original creators, source URLs and licences of those images have **not** been established, and the dataset is therefore excluded from the public repository rather than published under an unverified licence.

Acknowledging the dataset's unknown provenance is not a formality. It is the reason the results are reported with their reproducibility limits attached, and it is a genuinely open item for anyone who wants to build on this work with a properly licensed dataset.

---

## Reviewers and audiences

- The **project supervisor** and the department's review panel, whose feedback shaped the final scope.
- The **examiners and reviewers** of the final-year evaluation, for the questions that pushed the documentation to state its evidence boundaries explicitly.
- The **open-source community** whose public tools and documentation made an academic computer-vision project of this scope achievable without a budget.

---

## A note on what is not acknowledged

No commercial organisation, sponsor, vendor or platform is acknowledged, because none contributed to the work. The project used only open-source software and its own machine, and no external service was required to run, train or evaluate it. The application is entirely local, with no cloud dependency, no analytics and no third-party script.

If a future version adds a dependency on a hosted service or a commercial dataset, this page should be updated to reflect it — and the licence review in [licences and assets](licences-and-assets.md) completed before it ships.

---

## Related pages

- [Third-party attribution](third-party-attribution.md) — the formal notices.
- [Licences and assets](licences-and-assets.md) — the licence matrix.
- [Contributions](../CONTRIBUTIONS.md) — declared roles versus verified authorship.
- [Bibliography](bibliography.md) — the full reference list.
- [Citation guide](citation-guide.md) — how to cite the project.
