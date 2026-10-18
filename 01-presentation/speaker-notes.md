# Speaker notes

## Opening

Introduce InfraScan as an engineering integration project, not as a claim of independently developing YOLO. State that the source repository preserves upstream attribution and that the public release excludes unverified dataset and checkpoint artifacts.

## Technical explanation

An input is accepted by the detection workflow, passed to an Ultralytics model, and converted into annotated output. A confidence threshold filters weak detections; non-maximum suppression removes overlapping duplicate boxes. The dashboard presents actual runtime results and reports a clear missing-checkpoint state when the model is unavailable.

## Defending results

Say “verified by the listed test” only when an evidence row exists in `09-testing-and-results/verified-results.md`. Historical metrics in the inherited README are not new experiments. If asked for accuracy, explain that a fresh evaluation requires the excluded dataset and checkpoint.

## Likely questions

- Why object detection instead of classification? Detection provides location and confidence, not only an image-level label.
- Why Streamlit? It provides a lightweight Python interface around the existing inference functions.
- Why is the model not in the public repository? Redistribution permission has not been established.
- Can it replace road inspectors? No. It is an assistive prototype with dataset and domain limitations.
