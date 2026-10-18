# Implementation details

The following map is based on the public source tree.

| File | Purpose | Main interactions |
|---|---|---|
| `app.py` | Streamlit interface, model availability handling, image/video controls, status and downloads | Calls detection functions and displays outputs |
| `detect.py` | Core image/video/camera inference, rendering, severity logic, telemetry | Uses Ultralytics, OpenCV, NumPy, pandas |
| `train.py` | Training entry point and training arguments | Uses dataset YAML and Ultralytics training |
| `evaluate.py` | Evaluation entry point | Uses checkpoint and dataset split |
| `export.py` | Model export entry point | Converts a compatible model to selected formats |
| `test_camera.py` | Camera-oriented utility | Uses OpenCV capture and detection |
| `run_app.py` | Launcher convenience wrapper | Starts Streamlit |
| `scripts/ingest_voc_dataset.py` | VOC XML to YOLO label conversion | Requires explicit source/archive paths |
| `scripts/prepare_dataset.py` | Dataset preparation and split workflow | Requires explicit source paths |
| `scripts/visualize_annotations.py` | Annotation visualization | Requires dataset images and labels |
| `data.yaml` | Dataset path, splits, class count, and class name | Consumed by training/evaluation |

## Detection concepts

For boxes A and B, intersection over union is:

`IoU = area(A ∩ B) / area(A ∪ B)`.

Confidence expresses model belief in a detection; it is not a calibrated probability by default. Non-maximum suppression keeps stronger overlapping boxes according to an IoU rule.
