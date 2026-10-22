# Contribution evidence and responsibility matrix

This page separates three things that are often blurred in student projects:

1. **Assigned responsibility** — what the team agreed each member would own.
2. **Repository evidence** — what the current Git history and source tree can independently support.
3. **Presentation evidence** — test logs, design reviews, dataset records or demonstration notes that a member can present at the viva but that are not stored in Git.

No one should claim that a role assignment automatically proves authorship, or that a single Git author proves that nobody else contributed. The honest position is to show the evidence that exists and state what still needs to be attached.

## Current evidence

| Member | Assigned responsibility | Repository evidence today | Status of individual contribution claim |
|---|---|---|---|
| **Aayush Kumar Shrivastav** (2301921540002) | Team leader; core YOLO model; system architecture and integration | **All 6 main-repo commits** (`9242ff3`, `24bd606`, `b0d0649`, `d67ea1c`, `4ddcff8`, `89143ad`) and **every documentation commit**, listed newest-first through `34558c0` in `PROJECT_FACTS.md` §10 (the list is extended by the commit that updates it; `git log` is authoritative); FastAPI backend, SPA, README, evaluation run, tests and Git pushes are attributable to the configured Aayush identity | **Verified in repository history** for the integration, release and documentation work listed above |
| **Kajal Rawal (“Lucky”)** (2301921520087) | Dataset testing, validation and documentation support | No commit in the current main or docs history is independently attributable to Kajal. The verification runs exist, but their execution/ownership is not recorded in Git | **Assigned role; individual authorship not independently verified.** Attach a signed test log, review notes or presentation evidence before claiming specific tests |
| **Khushi Shakya** (2301921520091) | Frontend development and dashboard | The frontend source exists and is reviewed, but commit `b0d0649` is authored by Aayush. No separate Khushi commit or design-record file exists | **Assigned role; individual authorship not independently verified.** Attach component ownership notes, design-review evidence or test sign-off |
| **Ronak Saini** (2301921520158) | Dataset collection, preparation and documentation | The local dataset, counts and preparation scripts exist; the dataset is excluded from Git. No Ronak commit or provenance record is present | **Assigned role; individual authorship not independently verified.** Attach collection/preparation logs, label-validation evidence and provenance records |

## What is unambiguously verified

- The current project uses the FastAPI + SPA architecture documented in [`PROJECT_FACTS.md`](PROJECT_FACTS.md).
- The local YOLO11n checkpoint and 718-image test split produced the verified metrics in `PROJECT_FACTS.md`.
- The source tree contains the files described in the architecture and code-walkthrough chapters.
- The repository history records the commits listed above under the Aayush identity.
- The team names, roll numbers and assigned responsibilities were supplied by the team.

## What must be attached before the viva

Use this checklist for each member. A checked box is not evidence by itself; link the artefact.

### Kajal — dataset testing, validation and documentation support

- [ ] Test matrix showing which checks she executed, with dates and commands.
- [ ] Review notes for dataset labels, split integrity or result tables.
- [ ] Screenshots/log excerpts she personally captured.
- [ ] Documentation sections she wrote or edited, with a commit or shared-document history.
- [ ] A short oral explanation of PASS / FAIL / BLOCKED and the verified image/video tests.

### Khushi — frontend development and dashboard

- [ ] List of frontend components she designed, implemented or reviewed.
- [ ] Design rationale for the SPA state model, responsive layout or accessibility choices.
- [ ] Browser test notes at named viewports.
- [ ] Commit history or pull-request review evidence if the work was paired/integrated by Aayush.
- [ ] A short oral walkthrough of `app.js`, `inspect.js`, `video.js` or `camera.js`.

### Ronak — dataset collection, preparation and documentation

- [ ] Dataset source/provenance record with creator names and stable URLs (or a clear “unverified” note).
- [ ] Label-conversion log from `scripts/prepare_dataset.py` or `scripts/ingest_voc_dataset.py`.
- [ ] Split counts and a sample annotation inspection.
- [ ] Documentation sections he wrote or edited, with a shared-document history.
- [ ] Permission/licence status; do not claim redistribution rights that are not documented.

### Aayush — leadership, YOLO, architecture and integration

- [ ] Evaluation command, log and checkpoint hash.
- [ ] Architecture walkthrough using the current FastAPI + SPA diagram.
- [ ] API and frontend integration demonstration.
- [ ] Explanation of the H.264 fallback and shared severity module.
- [ ] Statement of what remains unverified (CUDA, exports, physical camera, dataset rights).

## How to talk about contribution honestly

Use language like:

> “My assigned role was frontend development. The repository contains the SPA, and I can walk through the components I worked on; the integration commit itself was made by Aayush. Here is my design/test evidence.”

Avoid:

> “I wrote the entire dashboard” — unless there is evidence and the team agrees on that claim.

The project is stronger when the contribution story matches the evidence. Examiners often ask follow-up questions about the exact file, test or design decision; a precise, modest claim is easier to defend than a broad one.
