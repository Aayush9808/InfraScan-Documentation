# Application usage

After installing dependencies and placing an authorized checkpoint:

```bash
streamlit run app.py
```

The dashboard exposes image inspection, video analysis, and system-oriented views. Video uploads are processed through the existing detection pipeline and temporary handling logic. The exact output and telemetry availability should be demonstrated using a real test run.

Command-line examples:

```bash
python detect.py --source path/to/image.jpg --weights weights/best.pt --save
python detect.py --source path/to/video.mp4 --weights weights/best.pt --save
```
