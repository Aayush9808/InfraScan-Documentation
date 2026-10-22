# Testing and limitations questions

Twenty questions on evidence, failure modes and honest scope. These are the questions that separate a polished demo from a defensible project.

---

### Q1. What was the testing strategy?

The project used a layered manual strategy: static checks (compile/import), API checks (health, status, image, invalid input, missing checkpoint), UI and responsive checks, a video job/playback check, a CLI check with telemetry, and a full held-out evaluation on 718 images. There is no automated Python or browser test suite, so the manual test log and evidence index are the audit trail.

### Q2. Why is there no automated test suite?

It was not implemented within the project scope. That is a real limitation: a future contributor could break a route or UI state without a failing CI job. The documentation does not pretend otherwise. The manual test log lists the checks that were actually executed, and the evidence index maps each claim to a source or artefact.

### Q3. What does “PASS” mean in this project?

`PASS` means the check was executed and the observed result matched the expected result on the stated machine/date. It does not mean “the whole system is correct.” `FAIL` means executed but not matching; `BLOCKED` means it could not be executed for an environmental or permission reason; `NOT RUN` means it was never attempted. These labels are defined in the test strategy.

### Q4. How was model accuracy measured?

With the full test split: **718 images, 994 pothole instances, 2 background images**, using the local YOLO11n checkpoint, 640 px, batch 16, MPS. The evaluator reported precision 0.7959, recall 0.5996, mAP@0.50 0.7111 and mAP@0.50–0.95 0.3938. The command and environment are recorded in `PROJECT_FACTS.md`.

### Q5. Why does the environment matter?

Accuracy and speed depend on the checkpoint, split, input size, batch, device and library versions. The verified run used Python 3.12.14, PyTorch 2.14.1, Ultralytics 8.4.183, an Apple M2 and MPS. A CUDA run or a different Ultralytics version could produce different speed and possibly different numeric output. Report the environment with the metric.

### Q6. What did the image functional test show?

The API accepted a test image, ran inference, and returned one **MODERATE** detection with **7.0% coverage** in the recorded sample. It wrote an annotated JPEG and returned a warm latency of about **66.2 ms**. This is a functional integration check, not a new accuracy benchmark; one image cannot estimate precision or recall.

### Q7. What did the video test show?

A short clip was processed as a background job: **24 frames** and **28 frame-level detections**. The output was H.264 and Chrome reported `readyState 4`, meaning it could play. This verifies the job/polling/encode/playback path on that clip; it does not establish throughput for long or high-resolution video.

### Q8. What happens when the checkpoint is missing?

The app still starts and serves the dashboard. `/api/status` reports the model as `unavailable` with an actionable error; detection routes return HTTP 503. This is deliberate: the UI should explain the missing asset rather than silently substitute a different model. The CLI fallback is a separate behaviour and is documented separately.

### Q9. What happens with an invalid image?

The upload is read, OpenCV fails to decode it, `InvalidImageError` is raised, and the API returns HTTP 400 with a decode message. The SPA shows the error without crashing. This verifies the negative path, which is often omitted in student demos.

### Q10. What about oversized files and invalid thresholds?

The upload readers enforce a 25 MB image limit and a 2 GB video limit, returning HTTP 413 when exceeded. Threshold validation returns HTTP 422 for confidence outside 0.05–1.0 or IoU outside 0.10–1.0. The limits are also exposed in `/api/status`.

### Q11. How was responsiveness tested?

At **390, 820, 1024 and 1440 px** widths, checking page-level horizontal overflow, navigation behaviour and panel stacking. This is a targeted manual check, not a full device-lab matrix. Touch behaviour, very large fonts and screen readers were not formally tested.

### Q12. Was the webcam tested?

A browser synthetic-media-device test was executed in Chrome. A **physical** camera on the presentation machine was not independently verified. The docs mark physical camera as `BLOCKED`. If the demo requires it, test it on the exact machine, browser and permission state beforehand.

### Q13. Was the CLI tested?

Yes: image inference produced an annotated JPEG plus CSV/JSON telemetry. The CLI shares `classify_severity()`, so its labels are consistent with the dashboard. The CLI retains a historical fallback to `yolo11s.pt` when `--weights` is omitted; reported results should always pass the explicit checkpoint.

### Q14. Were security headers verified?

The middleware sets `X-Content-Type-Options`, `X-Frame-Options`, `Referrer-Policy`, CSP and no-store on API paths. Their presence is a code-level fact and can be checked with `curl -I`. This is not a penetration test, and the app has no authentication; it must not be exposed directly to the public internet.

### Q15. What regression checks should be rerun after a change?

At minimum: startup with and without a checkpoint; `/api/health` and `/api/status`; image success and invalid image; threshold 422; video job + playback + telemetry; reload; samples; responsive 390/1440; security headers; CLI image/telemetry; and the full evaluation if the model, data or preprocessing changes. The manual test log is the checklist.

### Q16. How reproducible is the result?

The checkpoint SHA-256, evaluation command, split counts, package versions and device are recorded. Given the same checkpoint and data, the result should be reproducible within normal library/hardware variation. The dataset itself is excluded and its provenance is unresolved, so a third party cannot reproduce it from the public repository alone.

### Q17. Could the model be overfitting?

It is possible. The checkpoint trained for 15 epochs on a local dataset; the test split is held out, which is the main guard. But the project did not run a duplicate audit, cross-validation or a field test. The gap between mAP@0.50 (0.7111) and mAP@0.50–0.95 (0.3938) suggests localization weakness, not necessarily overfitting. Any claim about generalization beyond this split is unverified.

### Q18. What are the top limitations?

Recall around 60%, heuristic box-area severity, single-class scope, no tracking/GPS/depth, unresolved dataset/checkpoint rights, MPS-only evaluation, untested CUDA/exports/physical camera, no auth, in-memory jobs, no automated tests, and no formal accessibility/security audit. The full list is in `../03-project-documentation/limitations.md`.

### Q19. What ethical or safety concerns apply?

A user could over-trust a detection or severity label. The system must be presented as a screening/decision-support prototype, not an authoritative road-safety assessment. Privacy matters because road images may contain people, plates or locations; local processing reduces but does not eliminate risk. Dataset consent/licensing and bias in road imagery are unresolved.

### Q20. What testing would you do next?

Add pytest API tests and a browser E2E suite; test on CUDA and export runtimes; test physical cameras and multiple browsers; run a long-video stress test; audit duplicates and label quality; measure calibration and stratified performance by pothole size/lighting; and conduct a formal accessibility and security review. Each would need recorded evidence, not just a plan.

---

## Cross-examination prompts

- “Can you prove the 718-image result?” → Yes, within the recorded environment: command, split counts, metrics, environment and checkpoint hash are in the facts sheet; the PR curve is in the assets.
- “Why should we trust a manual test?” → You should trust the recorded observations, not a generic claim. The limitation is the absence of automation; the evidence and exact steps are documented.
- “Did you test the model on a different road?” → Not formally; domain shift is a known limitation.
- “What would break first in production?” → Authentication/security, long-video resource use, codec/browser variation, and dataset licensing.
- “What is the weakest evidence?” → Individual teammate contributions and dataset provenance are the weakest documented areas; both are explicitly flagged.

---

[Viva index](README.md) · [Test strategy](../09-testing-and-results/test-strategy.md) · [Manual test log](../09-testing-and-results/manual-test-log.md) · [Limitations](../03-project-documentation/limitations.md)
