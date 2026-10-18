# Kajal Rawal (Lucky): testing and presentation guide

## Planned scope

Reproducible checks, test evidence, limitations, presentation support, and result interpretation. Confirm completed work before presenting it as personal contribution.

## Test categories

- Syntax and import checks.
- Dashboard startup and missing-checkpoint behavior.
- Invalid input and unsupported format checks.
- Image/video inference only with authorized assets.
- Camera checks only with available hardware.
- Secret, path, artifact, and staged-diff scans.

## Questions

1. What is a smoke test? A quick check of basic startup behavior.
2. What is integration testing? Checking components together.
3. Why record commands? Reproducibility.
4. What is PASS? Executed with evidence.
5. What is BLOCKED? Required asset or environment unavailable.
6. Why not call blocked inference a pass? That would be inaccurate.
7. What is a negative test? Deliberately invalid input.
8. Why test missing weights? It is a common release condition.
9. What is regression? A previously working behavior breaking.
10. Why check `git diff --check`? Whitespace quality.
11. What is an evidence index? Map from claim to artifact/test.
12. How do you test video? Use a known input and inspect output/telemetry.
13. Why test camera separately? Hardware and permissions differ.
14. What is reproducibility? Another user can repeat the procedure.
15. Why distinguish historical metrics? They are not newly verified.
16. What is a limitation? A known boundary of validity.
17. What is a false negative? A pothole missed by the detector.
18. What is a false positive? A non-pothole labeled as one.
19. How should screenshots be labeled? Input, checkpoint, date, environment.
20. What is presentation risk? Claiming more than evidence supports.

## Short explanation

“My testing role is to separate evidence from assumption. I report PASS only for checks that ran, BLOCKED when assets or hardware are absent, and NOT RUN when no execution occurred. This makes the project easier to defend because every result has a command and limitation.”
