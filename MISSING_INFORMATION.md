# Missing information and evidence gaps

This checklist is intentionally narrow. Most project facts are now verified in [`PROJECT_FACTS.md`](PROJECT_FACTS.md); the items below are the ones that cannot be filled from the current source tree, checkpoint metadata, dataset copy or Git history.

## Academic / submission fields

- [ ] Official GLBITM **group ID**.
- [ ] Supervisor's full name and official designation.
- [ ] Confirmation of the official spelling of **Aayush Kumar Shrivastav** on college documents.
- [ ] Confirmation of whether **Kajal Rawal** should appear as “Kajal Rawal” or “Kajal Rawal (Lucky)” on the PPT/report.
- [ ] Confirmation of the official project title used by the department (the documentation currently uses “InfraScan — AI-Powered Road Inspection and Pothole Detection System”).
- [ ] Confirmation of the session/batch printed in the GLBITM templates (currently shown as 2026–2027 / Batch 2023).

## Individual contribution evidence

The repository history currently attributes all commits to Aayush Kumar Shrivastav / `Aayush9808`. The other roles are team-declared. Before the viva, attach the evidence listed in [`CONTRIBUTIONS.md`](CONTRIBUTIONS.md):

- [ ] Kajal — test execution log, review notes, documentation edits or screenshots she personally captured.
- [ ] Khushi — specific frontend components/design decisions she owned, plus browser test notes or review evidence.
- [ ] Ronak — dataset collection/preparation log, label-validation notes, split counts and provenance records.
- [ ] Aayush — evaluation log, checkpoint hash, architecture/demo walkthrough and release checklist (partly present in this repository).

Do not replace these with guessed claims. A precise “I owned this component and here is the evidence” is stronger than a broad unsupported claim.

## Dataset and checkpoint governance

- [ ] Original dataset names, creators and stable source URLs for every source.
- [ ] Dataset licence and permission for derived labels and processed copies.
- [ ] Permission to redistribute the checkpoint.
- [ ] Confirmation of the checkpoint's training-data provenance beyond the embedded Windows path in its metadata.
- [ ] Written permission or licence review for any third-party images used in the PPT, poster or report.
- [ ] Decision on whether the checkpoint/dataset will be shared privately with examiners or kept local.

## Testing evidence still missing

- [ ] Live physical-webcam test on the presentation machine (currently synthetic-device only).
- [ ] CUDA inference benchmark and memory measurement (test machine was Apple M2).
- [ ] ONNX export and runtime test.
- [ ] TensorRT export and runtime test.
- [ ] TorchScript, OpenVINO, CoreML and TFLite export tests.
- [ ] Long-video stress test (duration, file size, memory, thermal behaviour).
- [ ] Multi-user/concurrent-request stress test.
- [ ] Formal accessibility audit (keyboard-only, screen reader, contrast measurement).
- [ ] Automated test suite (none exists; manual test log is the current evidence).

## Template files

- [ ] Confirm that the local `GLBITM PPT Format.pptx` and `GLBITM POSTER Format.pptx` are the current approved versions.
- [ ] Confirm whether the department requires a specific citation style (the report uses APA-style references).
- [ ] Confirm whether a PDF export or printed poster is required in addition to Markdown content.
- [ ] Confirm whether the poster requires the group ID before printing.

## How to close a gap

1. Obtain the artefact or permission.
2. Record the date, owner and exact evidence location.
3. Update `PROJECT_FACTS.md` if the item changes a project fact.
4. Update the relevant chapter and `09-testing-and-results/evidence-index.md`.
5. Commit the change with a message that names the evidence source.

Do not silently convert a `[TO BE CONFIRMED]` item into a definitive statement without evidence.
