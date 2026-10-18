# Inference workflow

1. User supplies an image, video, or camera source.
2. The application validates the source and checkpoint availability.
3. A compatible Ultralytics checkpoint is loaded.
4. The source is processed frame by frame where applicable.
5. Predictions are filtered and rendered.
6. Severity and telemetry logic records supported fields.
7. The dashboard displays or offers the generated output.

If `weights/best.pt` is absent, the release candidate reports the missing dependency and does not silently download or substitute a model.
