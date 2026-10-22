# Installation

This page installs the accepted InfraScan application: a FastAPI backend, a static vanilla-JavaScript SPA, and a local Ultralytics YOLO11n inference pipeline. It covers the verified macOS path first, then the platform variations an operator is likely to meet.

- [Prerequisites](#prerequisites)
- [Get the source](#get-the-source)
- [Create a virtual environment](#create-a-virtual-environment)
- [Install dependencies](#install-dependencies)
- [Supply a checkpoint](#supply-a-checkpoint)
- [Optional dataset samples](#optional-dataset-samples)
- [First launch](#first-launch)
- [Verify the installation](#verify-the-installation)
- [Platform notes](#platform-notes)
- [Reinstall and uninstall](#reinstall-and-uninstall)
- [Evidence boundary](#evidence-boundary)
- [Next steps](#next-steps)

## Prerequisites

| Requirement | Recommended | Why |
|---|---|---|
| Python | 3.12.x (3.12.14 verified) | Dependency compatibility; the project has no upper pin |
| pip | Current version for that Python | Resolves wheels correctly |
| Virtual environment | `venv` | Keeps the ML stack out of the system interpreter |
| Disk space | Enough for PyTorch/Ultralytics plus artefacts | The dependency set is large; the checkpoint is 5.2 MB |
| Checkpoint | YOLO11n-compatible `best.pt` with `pothole` | Inference cannot run without it |
| Browser | Current Chrome (verified) | ES modules, `<dialog>`, `:has()`, `fetch`, optional camera |
| Network | Only for installation | Runtime inference is local |

There is **no Node.js, npm, bundler or separate frontend server** to install. The SPA is plain HTML, CSS and ES modules served by FastAPI.

Check your interpreter:

```bash
python3 --version
python3 -m pip --version
```

If `python3` is older than 3.12 on your machine, install a newer interpreter first. The verified environment used Python 3.12.14. Lower versions may work but are `[TO BE CONFIRMED]`.

## Get the source

```bash
git clone https://github.com/Aayush9808/InfraScan.git
cd InfraScan
```

If you received a source archive instead of a Git checkout, extract it and `cd` into the directory containing `app.py`, `run_app.py`, `requirements.txt` and `web/`.

Confirm the expected layout:

```text
InfraScan/
├── app.py
├── run_app.py
├── requirements.txt
├── data.yaml
├── infrascan/
├── web/
├── weights/            # checkpoint goes here (gitignored)
├── dataset/            # optional samples (gitignored)
└── runs/               # runtime output (gitignored)
```

Do not run the application from inside `web/` or `infrascan/`. The launcher changes to the project root itself, but relative CLI commands in this documentation assume the root.

## Create a virtual environment

### macOS / Linux

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
```

### Windows PowerShell

```powershell
py -3.12 -m venv .venv
.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
```

If PowerShell blocks the activation script, use Command Prompt (`.venv\Scripts\activate.bat`) or adjust the execution policy for the current user. Do not disable security policy globally just to run this project.

### Windows Command Prompt

```bat
py -3.12 -m venv .venv
.venv\Scripts\activate.bat
python -m pip install --upgrade pip
```

On Debian/Ubuntu, if `venv` is missing:

```bash
sudo apt install python3-venv
```

After activation, `python` and `pip` should point inside `.venv`:

```bash
python -c "import sys; print(sys.executable)"
```

## Install dependencies

```bash
pip install -r requirements.txt
```

The file contains lower bounds only, so pip resolves the newest compatible versions at install time:

| Package | Declared floor | Role |
|---|---|---|
| `ultralytics` | `>=8.3.0` | YOLO model loading, prediction, training, validation and export |
| `torch`, `torchvision` | `>=2.0.0`, `>=0.15.0` | Deep-learning runtime and device backends |
| `opencv-python` | `>=4.8.0` | Image/video decode, annotation and MP4 writing |
| `pillow` | `>=9.5.0` | Image support used by the ML stack |
| `numpy` | `>=1.24.0` | Array operations |
| `pandas` | `>=2.0.0` | Telemetry CSV |
| `matplotlib`, `seaborn` | `>=3.7.0`, `>=0.12.0` | Ultralytics plots and evaluation output |
| `pyyaml` | `>=6.0` | Dataset configuration |
| `tqdm` | `>=4.65.0` | Progress bars in training/evaluation |
| `fastapi` | `>=0.115.0` | HTTP API |
| `uvicorn[standard]` | `>=0.30.0` | ASGI server |
| `python-multipart` | `>=0.0.9` | Multipart uploads |

There is **no lockfile**. Installing at a later date can produce different versions. For a reproducible demonstration, record the versions from `/api/status` and consider pinning them in your own environment.

### PyTorch and accelerators

The verified run used PyTorch 2.14.1 with Apple MPS. The generic `torch>=2.0.0` install is sufficient for CPU and, on Apple Silicon, normally exposes MPS.

For NVIDIA CUDA, install the PyTorch build that matches your driver from the official PyTorch index before or alongside `requirements.txt`. The CUDA path is implemented (`cuda:0` is selected first) but **was not executed** on this project's development machine. Do not claim CUDA results from this documentation.

```bash
# Example only: choose the CUDA build that matches your machine.
# See https://pytorch.org/get-started/locally/ for the current command.
pip install torch torchvision --index-url <matching-cuda-wheel-index>
pip install -r requirements.txt
```

### OpenCV and video codecs

`opencv-python` wheels bundle FFmpeg but do not guarantee an H.264 encoder on every platform. The application tries `avc1` first and falls back to `mp4v` for the annotated video. `avc1` produced browser-playable output on the verified machine; `mp4v` may not play in every browser. If video playback fails, see [Troubleshooting](troubleshooting.md#video-playback).

## Supply a checkpoint

The checkpoint is deliberately **not redistributed** by the repository and `.gitignore` excludes `weights/`. Its provenance and redistribution permission are unresolved (`[TO BE CONFIRMED]`). Obtain a compatible checkpoint from a source you are authorised to use.

### Default location

```text
weights/best.pt
```

Create the directory if needed:

```bash
mkdir -p weights
# Copy your authorised checkpoint to weights/best.pt
ls -lh weights/best.pt
```

The verified checkpoint is:

| Property | Value |
|---|---|
| Architecture | Ultralytics YOLO11n |
| Task | `detect` |
| Class | `pothole` (class 0) |
| Input size | 640 px |
| File size | 5,467,546 bytes (5.2 MB) |
| SHA-256 | `f3490f953b4ac2aa0205c206c151be8f2da281da99708713b182492a43c7751c` |

If you are reproducing the exact verified results, compare the hash:

```bash
shasum -a 256 weights/best.pt          # macOS/Linux
certutil -hashfile weights\best.pt SHA256   # Windows
```

A different but compatible YOLO checkpoint can run the dashboard, but its classes, accuracy and behaviour will differ. The dashboard reads class names from the checkpoint rather than forcing `pothole`.

### Alternative path

Set `INFRASCAN_WEIGHTS` **before** starting Python:

```bash
export INFRASCAN_WEIGHTS="/absolute/path/to/best.pt"     # macOS/Linux
$env:INFRASCAN_WEIGHTS = "C:\path\to\best.pt"            # PowerShell
```

The variable is read when `infrascan.config` is imported. It is not re-read by the **Reload checkpoint** button. See [Configuration](configuration.md#checkpoint-path) for the `--weights` launcher caveat.

### Optional checkpoint sanity check

Loading the checkpoint directly verifies that PyTorch/Ultralytics can read it and shows the class names:

```bash
python - <<'PY'
from ultralytics import YOLO
m = YOLO("weights/best.pt")
print("task:", m.task)
print("names:", m.names)
PY
```

This loads the model in a separate process and does not test the dashboard. If it fails, the dashboard will also fail to load the checkpoint; fix that first.

> **Security note:** a `.pt` checkpoint is loaded through PyTorch's pickle-based mechanism. Only load checkpoints from sources you trust. See [Security and privacy](../11-security-and-quality/security-and-privacy.md#sec-03--checkpoint-trust).

## Optional dataset samples

The **Dataset** tab in Image inspection reads files from:

```text
dataset/images/test/
```

The API lists up to 24 images (`SAMPLE_LIMIT`), sorted by filename. It is optional: without the directory, the tab shows an empty-state message and uploads still work.

Expected dataset layout for the CLI tools:

```text
dataset/
├── images/
│   ├── train/
│   ├── val/
│   └── test/
└── labels/
    ├── train/
    ├── val/
    └── test/
```

`data.yaml` defines one class and relative paths:

```yaml
path: ./dataset
train: images/train
val: images/val
test: images/test
nc: 1
names:
  - pothole
```

The verified local dataset has 718 test images. It is **not** bundled, and its creators, source URLs and licence are unresolved. Do not redistribute it. The sample picker only needs test images; labels are not read by the dashboard.

## First launch

With the virtual environment active and the checkpoint in place:

```bash
python run_app.py
```

Expected banner (abridged):

```text
  InfraScan · Road Intelligence Dashboard
  Dashboard : http://localhost:8000
  API docs  : http://localhost:8000/api/docs
  Press Ctrl+C to stop.
```

The browser opens after about 1.5 seconds unless `--no-browser` is used. The dashboard is served immediately; the model loads in a background thread. The status chip shows **Loading model** until loading finishes, then **Model ready**.

If the checkpoint is missing, the chip shows **No checkpoint** and the System page displays the expected path. This is a supported state, not a crash.

### Launch variants

```bash
python run_app.py --no-browser
python run_app.py --port 8001
python run_app.py --reload                 # development only
uvicorn app:app --host 127.0.0.1 --port 8000
```

The default host is `127.0.0.1`. Do not use `--host 0.0.0.0` on an untrusted network; there is no authentication or TLS.

## Verify the installation

Run these checks in a second terminal with the server running.

| Check | Command | Expected |
|---|---|---|
| SPA | `curl -s -o /dev/null -w "%{http_code}\n" http://127.0.0.1:8000/` | `200` |
| Liveness | `curl -s http://127.0.0.1:8000/api/health` | `{"status":"ok","model_state":"..."}` |
| Readiness | `curl -s http://127.0.0.1:8000/api/status` | `model.state` is `ready` when loaded |
| API schema | `curl -s -o /dev/null -w "%{http_code}\n" http://127.0.0.1:8000/api/openapi.json` | `200` |
| Assets | `curl -s -o /dev/null -w "%{http_code}\n" http://127.0.0.1:8000/assets/css/app.css` | `200` |

Readiness one-liner:

```bash
curl -s http://127.0.0.1:8000/api/status \
  | python -c "import json,sys; print(json.load(sys.stdin)['model']['state'])"
```

Expected model states:

| State | Meaning | Action |
|---|---|---|
| `starting` | Initial state before a load attempt | Wait or reload |
| `loading` | Loader thread running | Wait; first load can take a while |
| `ready` | Inference available | Proceed |
| `unavailable` | Checkpoint missing or empty | Add `weights/best.pt`; reload/restart |
| `error` | Checkpoint exists but load failed | Read the error on the System page/server log |

### End-to-end image check

Use any road image you have:

```bash
curl -s -X POST http://127.0.0.1:8000/api/detect/image \
  -F "file=@/path/to/road.jpg" \
  -F "conf=0.25" \
  -F "iou=0.45" \
  | python -m json.tool
```

A successful response includes `summary`, `detections`, `inference_ms` and an `image_url`. Open the URL on the same host to fetch the annotated JPEG. A `400` means the file could not be decoded; a `503` means the checkpoint/model is not ready.

### End-to-end video check (optional)

Use a short clip for the first test:

```bash
curl -s -X POST http://127.0.0.1:8000/api/detect/video \
  -F "file=@/path/to/short-clip.mp4" \
  -F "conf=0.25" \
  -F "iou=0.45" \
  | python -m json.tool
```

The response is `202` with a job `id`. Poll it:

```bash
curl -s http://127.0.0.1:8000/api/jobs/<id> | python -m json.tool
```

When `status` becomes `completed`, the `result` object contains video and telemetry URLs. A failed job has `status: "failed"` and an `error` string.

The verified functional check processed 24 frames with 28 detections and produced an H.264 MP4 that Chrome played to `readyState 4`. That does not guarantee every codec/container/browser combination.

## Platform notes

### macOS (verified)

- Apple Silicon uses MPS automatically when available.
- The first model load may be slower than later runs; no load-time benchmark is recorded.
- If MPS inference fails, the image path retries on CPU and records `cpu` until the next reload/restart.
- Browser camera access works on `http://localhost` and `http://127.0.0.1`.

### Linux (expected, `[TO BE CONFIRMED]`)

- Install `python3-venv` if `venv` creation fails.
- For a remote/headless host, run the browser on another machine only with a deliberate, secured network setup. The app has no auth.
- Install the PyTorch build appropriate for CPU or CUDA; the generic wheel may be CPU-only depending on the index.
- H.264 encoder availability varies by OpenCV wheel; the fallback is `mp4v`.

### Windows (expected, `[TO BE CONFIRMED]`)

- Activate with `.venv\Scripts\activate`.
- Long paths and spaces are handled by `pathlib`; quote paths in commands.
- Windows Firewall may prompt if a non-loopback host is used.
- `test_camera.py` uses DirectShow first for camera indices; `detect.py` uses OpenCV's default backend.
- `train.py` sets `workers=0` on Windows.

### Containers

No Dockerfile or Compose file is provided. A container would need the full Python/ML stack, a mounted checkpoint, writable `runs/`, and a deliberate port/loopback decision. Not tested.

## Reinstall and uninstall

### Clean reinstall of dependencies

```bash
deactivate 2>/dev/null || true
rm -rf .venv
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
pip install -r requirements.txt
```

### Remove the application

1. Stop the server.
2. Delete the checkout or virtual environment.
3. Delete `runs/dashboard/` if you no longer need the generated media and telemetry.
4. Retain or destroy the checkpoint/dataset according to their own permissions; they are not managed by the application.

There is no installer, service registration or registry entry to remove.

## Evidence boundary

| Statement | Status |
|---|---|
| These commands install the dependencies declared in the repository | Source-verified |
| The verified environment used Python 3.12.14, PyTorch 2.14.1, Ultralytics 8.4.183, OpenCV 5.0.0.93 | Verified |
| The dashboard serves `/`, `/assets`, `/api/health`, `/api/status` | Verified |
| Image and video inference work end to end | Verified |
| CUDA installation/execution | **NOT RUN** |
| Linux and Windows installation | `[TO BE CONFIRMED]` |
| Physical webcam | **BLOCKED** |
| Export targets | **NOT RUN** |

Do not present the platform-notes expectations as tested results. If you install on a new platform, record the actual versions and commands in your run notes.

## Next steps

- [Configuration](configuration.md) — change paths, port, thresholds and limits.
- [Application usage](application-usage.md) — learn the interface.
- [Troubleshooting](troubleshooting.md) — fix startup or inference failures.
- [Deployment and operations](../04-architecture/deployment-and-operations.md) — runbook and operational limits.
- [Verified results](../09-testing-and-results/verified-results.md) — what the recorded run actually measured.
