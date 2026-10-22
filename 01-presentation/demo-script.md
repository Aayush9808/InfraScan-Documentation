# Demo script

**Goal:** demonstrate the real accepted system in 5–7 minutes without relying on the internet, a physical webcam, or an unverified export.

## Pre-demo checklist (do this 15 minutes before)

- [ ] Application starts from the repository root.
- [ ] `weights/best.pt` exists and is the compatible checkpoint.
- [ ] `python run_app.py --no-browser` is running on `http://127.0.0.1:8000`.
- [ ] `http://127.0.0.1:8000/api/health` returns `model_state: "ready"`.
- [ ] A test image is available at `dataset/images/test/` (or use a local road image).
- [ ] A short authorized video is available locally (MP4/AVI/MOV/MKV) if demonstrating video.
- [ ] Browser zoom is 100%; dashboard is at a 1440×900 or 1280×800 window.
- [ ] Backup screenshots are open in another tab.
- [ ] No terminal contains secrets, personal paths or unrelated projects.
- [ ] Close DragonX/other unrelated applications and notifications.

## Start the application

```bash
cd "/Users/aayus/Desktop/College final year/InfraScan"
source ~/.venvs/infrascan/bin/activate   # or your own venv
python run_app.py --no-browser
```

Then open <http://127.0.0.1:8000>.

If port 8000 is busy:

```bash
python run_app.py --no-browser --port 8010
```

Use the actual port consistently in the demo.

## Demo sequence

### 1. Overview (45 seconds)

Point out:

- Model status = **Ready** (or explain `unavailable` if that is the state).
- Device = **Apple GPU · MPS** on this machine.
- Checkpoint name and size.
- Session scans counter.
- Severity scale: critical ≥ 8%, moderate ≥ 2.5%, minor < 2.5%.

Say: “The runtime panel is reading the actual loaded model, not a hard-coded status.”

### 2. Image inspection (90 seconds)

1. Open **Image inspection**.
2. Choose **Dataset** → pick a sample, or **Upload** a local road image.
3. Leave confidence 0.25 and IoU 0.45.
4. Click **Run inspection**.
5. When the result appears, point to:
   - annotated box and label,
   - detection table (confidence, severity, coverage, bbox),
   - summary metrics,
   - **Annotated / Original** toggle.
6. Download the CSV (optional; show the button).

Say: “The recorded test sample produced one moderate detection at 7.0% coverage. The number is box area divided by frame area; it is a heuristic, not depth.”

### 3. Video analysis (90 seconds, optional)

1. Open **Video analysis**.
2. Upload a short local clip.
3. Click **Analyse video**.
4. Show progress/ETA and the job stage.
5. When complete, play the annotated MP4.
6. Point to detections, critical count, frames, processing time and telemetry downloads.

Say: “Video is a background job, not a blocking request. Inference is serialised so one long clip cannot starve the API. The encoder prefers H.264 for browser playback and falls back to `mp4v` if the local OpenCV build cannot encode H.264.”

If the clip is long or the machine is slow, skip live processing and show the verified video screenshot; do not wait silently.

### 4. System page (45 seconds)

- Checkpoint path, task, classes, device, inference size and file size.
- Python/PyTorch/Ultralytics/OpenCV versions.
- Severity policy.
- **Reload checkpoint** button.

Say: “This page is the evidence surface: it reports the actual runtime rather than a marketing claim.”

### 5. CLI fallback (45 seconds, optional)

In a second terminal:

```bash
cd "/Users/aayus/Desktop/College final year/InfraScan"
python detect.py \
  --source dataset/images/test/pothole_dataset_1_train_image10.jpg \
  --weights weights/best.pt \
  --output /tmp/infrascan_demo
```

Show the annotated JPEG and telemetry files. Say: “The CLI shares the severity module, so the classification is consistent with the dashboard.”

### 6. API check (30 seconds, optional)

```bash
curl -s http://127.0.0.1:8000/api/health
curl -s http://127.0.0.1:8000/api/status | head -c 500
```

Do not paste a huge JSON blob on screen; show the model state and one or two fields.

## What to show if something fails

| Failure | Recovery |
|---|---|
| Dashboard does not load | Show backup screenshot; check terminal log; restart on port 8010 |
| Model state `unavailable` | Explain the checkpoint path; do not fake a result; show the system page error |
| Image upload rejected | Use a JPG/PNG sample; explain the 25 MB and format limits |
| Video takes too long | Stop; show the verified video screenshot and explain the job design |
| Browser cannot play MP4 | Mention the H.264/`mp4v` fallback and use the downloaded file if needed |
| Webcam unavailable | Say physical camera was not verified; show the synthetic-device test evidence, not a claim |
| No dataset sample | Upload a local image instead; the sample picker is optional |

## Closing script

> “That is the complete loop: input, local inference, transparent severity, annotated evidence and telemetry. The verified evaluation is separate from this demo: 718 test images, mAP at 0.50 of 0.7111. We are happy to walk through the code or the limitations.”

## What not to demo

- A physical webcam unless it has been tested on that exact machine.
- TensorRT/ONNX/CoreML export unless it was run and verified.
- A cloud URL or internet-dependent model download.
- A dataset image whose redistribution permission is unknown, in a public recording.
- Any metric that is not in [`../PROJECT_FACTS.md`](../PROJECT_FACTS.md).

---

[Presentation index](README.md) · [Speaker notes](speaker-notes.md) · [Verified results](../09-testing-and-results/verified-results.md) · [Troubleshooting](../06-setup-and-execution/troubleshooting.md)
