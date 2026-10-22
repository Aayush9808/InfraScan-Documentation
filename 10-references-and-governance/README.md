# References and governance

How InfraScan is cited, what it is built on, what it is allowed to reuse, and the rules that keep those statements honest.

- [Purpose](#purpose)
- [Chapter contents](#chapter-contents)
- [Citation policy](#citation-policy)
- [Licence summary](#licence-summary)
- [Governance of the documentation](#governance-of-the-documentation)
- [Known attribution gaps](#known-attribution-gaps)
- [What this chapter does not claim](#what-this-chapter-does-not-claim)
- [Related pages](#related-pages)

---

## Purpose

Two failure modes destroy the credibility of an academic project before an examiner finishes reading:

1. **Inherited claims.** Citing a metric that was never measured here, or a method that was never run.
2. **Unattributed borrowing.** Using someone else's dataset, checkpoint, code or trained weights without saying so.

This chapter addresses both. It records the sources the project actually depends on, states the licence position for each, and marks the places where attribution is still unresolved.

The rule applied throughout: **an asset that cannot be attributed is not described as reusable.** Where the provenance is unknown, the documentation says so and the asset is excluded from the public repository.

---

## Chapter contents

| Page | Purpose |
|---|---|
| [Bibliography](bibliography.md) | The numbered sources cited by the [project report](../03-project-documentation/project-report.md), plus categorised further reading and the references deliberately not used |
| [Citation guide](citation-guide.md) | How to cite the project, its code, its documentation, the upstream project and the model, with ready-to-use formats |
| [Third-party attribution](third-party-attribution.md) | Every external dependency, asset and upstream source, and the notice each requires |
| [Licences and assets](licences-and-assets.md) | The licence matrix, the asset provenance table and the redistribution position for the dataset and checkpoint |
| [Acknowledgements](acknowledgements.md) | The people, institutions and upstream authors the project depends on |

---

## Citation policy

1. **The report's numbered list is the citation source of truth.** In-text markers like `[8]` in the [project report](../03-project-documentation/project-report.md) resolve against the reference list in that same document. The [bibliography](bibliography.md) mirrors it and adds further reading; it does not renumber it.
2. **Every citation resolves to a URL.** No entry points to a page that requires a login, a paywall or a personal contact.
3. **Citations support context, not results.** A citation may explain how pothole detection is normally approached; it may never stand in for a number this project did not measure.
4. **Prefer the primary source.** The original paper or specification is cited rather than a summary of it.
5. **Document access dates** for sources without a fixed publication date, because web documentation changes. The current list records **22 October 2026** as the access date.
6. **No citation laundering.** Restating a claim in different words does not make it independently verified. If the project did not run it, the text says the project did not run it.

---

## Licence summary

| Component | Licence / status | Redistribution |
|---|---|---|
| InfraScan source code | MIT (see the code repository `LICENSE`) | Permitted, with the notice preserved |
| Upstream project code by Bisman Singh | MIT — copyright notice preserved in the code repository | Permitted with the notice |
| This documentation | Academic project documentation; no separate licence declared | Treated as all-rights-reserved unless stated otherwise |
| Ultralytics YOLO (code, weights, `yolo11n.pt`) | **AGPL-3.0 or a commercial Enterprise licence**, per the Ultralytics licensing page | Do not redistribute Ultralytics weights or AGPL-covered code outside the stated terms |
| The trained checkpoint `weights/best.pt` | **Unresolved.** Produced with AGPL-covered tooling; the training data's rights are unknown | **Not redistributed.** Excluded from the repository |
| The pothole dataset | **Unresolved.** Original creators, source URLs and licences are not established | **Not redistributed.** Excluded from the repository |
| Inter and Space Grotesk fonts | SIL Open Font License 1.1 — licence files bundled in `web/assets/fonts/` | Permitted with the notice |
| GLBITM presentation and poster templates | Institutional templates held outside the repository; subject to the institution's usage rules | **Not redistributed.** Deliberately not committed |
| Python dependencies | Individual licences (BSD, Apache-2.0, MIT and others) — see the [licence matrix](licences-and-assets.md) | Permitted, subject to each package's terms |
| Screenshots and evaluation images in `assets/` | Produced by the team from the running application | Permitted with this documentation |

The single most important line in that table is the checkpoint. Ultralytics publishes its models under AGPL-3.0 with an Enterprise alternative, and a checkpoint trained with that tooling inherits questions the project cannot answer on its own. The project therefore **ships the code, not the weights**, and says so plainly.

---

## Governance of the documentation

| Rule | Where it is enforced |
|---|---|
| Every implementation claim is tied to the source tree or to a recorded check | [Evidence index](../09-testing-and-results/evidence-index.md) |
| `PROJECT_FACTS.md` is the canonical fact sheet; where a chapter disagrees with it, it wins | [`PROJECT_FACTS.md`](../PROJECT_FACTS.md) |
| Unverified academic fields stay marked `[TO BE CONFIRMED]` | [Missing information](../MISSING_INFORMATION.md) |
| Blocked and not-run checks are labelled, never promoted to PASS | [Test strategy](../09-testing-and-results/test-strategy.md#status-vocabulary) |
| Declared contributions are separated from independently verified authorship | [Contributions](../CONTRIBUTIONS.md) |
| Changes are made in small, meaningful commits with a reason in the message | The repository history |
| Evidence assets are copies of real artefacts, never mock-ups | [Assets](../assets/README.md) |

**Who may change what.** The team may correct any statement, but a correction that adds a new *claim* must come with its evidence in the same commit. A correction that removes a claim needs no evidence — it only needs to be right.

---

## Known attribution gaps

These are unresolved and are recorded in [Missing information](../MISSING_INFORMATION.md) as well:

| Gap | Consequence |
|---|---|
| The original creators, stable URLs and licences of the pothole images | The dataset cannot be described as freely reusable or redistributed |
| The exact split-generation history of the local dataset | Near-duplicate leakage across splits cannot be excluded |
| Permission to redistribute the trained checkpoint | The checkpoint stays out of the repository |
| The final academic metadata — group ID, supervisor name and designation | Kept as `[TO BE CONFIRMED]` on the cover of every deliverable |
| The recorded evidence for three team members' individual contributions | Their roles are documented as team-declared, not as independently verified authorship |

None of these gaps is hidden behind confident prose. Each is stated at the point where a reader would otherwise assume it had been resolved.

---

## What this chapter does not claim

- It does not grant any licence over the dataset or the checkpoint.
- It does not assert that the upstream project was the sole source of the pothole images.
- It does not assert that the checkpoint is legally distributable.
- It does not claim a security or privacy audit of any dependency.
- It does not replace a legal review of the redistribution position. For a published paper or a public release, a maintainer should confirm the dataset and checkpoint terms first.

---

## Related pages

- [Bibliography](bibliography.md)
- [Citation guide](citation-guide.md)
- [Third-party attribution](third-party-attribution.md)
- [Licences and assets](licences-and-assets.md)
- [Acknowledgements](acknowledgements.md)
- [Project report](../03-project-documentation/project-report.md)
- [Contributions](../CONTRIBUTIONS.md)
- [Missing information](../MISSING_INFORMATION.md)
