# Aayush technical guide

## Scope

Planned scope: technical coordination, detection pipeline, integration, debugging, and architecture explanation. Confirm which items were personally completed.

## Walkthrough

Explain how `app.py` calls the inference functionality in `detect.py`, how the checkpoint is located, how image/video/camera sources differ, and how outputs are rendered and exported.

## Viva questions

1. What is object detection? Predicting classes and locations.
2. Why not classification? Classification has no object location.
3. What is IoU? Intersection over union.
4. What is NMS? Duplicate-box suppression.
5. What is confidence? Detection score, not automatically calibrated probability.
6. What is inference? Running learned parameters on new input.
7. Why is a checkpoint needed? It stores learned parameters.
8. Why exclude the checkpoint? Redistribution rights are unverified.
9. What is Streamlit's role? Python UI around the pipeline.
10. What does `data.yaml` define? Dataset root, splits, class count, names.
11. What is telemetry? Structured detection records.
12. What can cause false positives? Texture, shadow, water, markings.
13. What is domain shift? Difference between training and deployment data.
14. Why use video frame processing? Apply image inference over time.
15. What does NMS threshold control? Overlap used to suppress boxes.
16. What is recall? Detected true positives divided by actual positives.
17. What is precision? Correct predictions divided by predicted positives.
18. Why can high confidence still be wrong? Scores are model outputs, not guarantees.
19. What is a limitation of box detection? No depth or physical severity measurement.
20. How would you reproduce results? Pin versions, hash data/checkpoint, record commands.

## Cross-questions

- **Why claim YOLO11?** Only if checkpoint metadata and code confirm it; otherwise state that the Ultralytics variant requires confirmation.
- **Can this replace inspectors?** No, it is an assistive prototype.
- **Why no accuracy number?** The public candidate lacks authorized evaluation assets.
- **What happens when the model is missing?** The dashboard reports an actionable error.
- **Why preserve upstream attribution?** The source derives from the upstream project.

## 60-second explanation

“InfraScan integrates a YOLO-based detection engine with a Streamlit dashboard. The user supplies an image or video, the application validates the input and checkpoint, runs inference, renders pothole boxes, and exposes supported telemetry. I focus on the technical integration and debugging. The project is careful about evidence: the public release excludes the dataset and checkpoint until their permissions are confirmed, so historical metrics are not presented as a new evaluation.”
