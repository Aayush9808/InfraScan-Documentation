# Code and architecture questions

1. `app.py` is the Streamlit entry point.
2. `detect.py` contains core inference logic.
3. `data.yaml` describes dataset paths/classes.
4. `train.py` starts training.
5. `evaluate.py` starts evaluation.
6. `export.py` supports model export.
7. Dataset scripts require explicit paths.
8. Temporary video handling avoids overwriting uploads.
9. Missing checkpoints produce an explicit error.
10. The UI and inference code remain Python components in one application.
