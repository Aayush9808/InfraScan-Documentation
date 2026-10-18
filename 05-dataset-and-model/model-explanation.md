# Model explanation

The implementation uses the Ultralytics Python API. The exact trained model variant for the excluded checkpoint must be confirmed from the checkpoint metadata; do not infer it solely from README prose or badges.

YOLO-style one-stage detection predicts object locations and classes in a direct pass. This makes it suitable for an interactive prototype, but speed/accuracy trade-offs depend on model variant, input size, hardware, and checkpoint.

The model is not a segmentation model and does not directly estimate depth. Severity labels calculated by the application should be explained as application heuristics, not physical measurements.
