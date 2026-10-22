# Technical glossary

Precise definitions for the detection, evaluation, backend, media and operational terms used across this documentation. Each entry is one to three sentences. Where a term has a project-specific meaning, that meaning is stated explicitly.

**Quick index by theme:** [Detection and model](#detection-and-model) · [Training](#training) · [Evaluation](#evaluation) · [Runtime and backend](#runtime-and-backend) · [Frontend and security](#frontend-and-security) · [Media and operations](#media-and-operations)

---

## Detection and model

**Anchor-based detector.** A detector that predicts offsets and class scores relative to a predefined set of boxes (anchors) at each feature-map location. Anchors require hand-tuned sizes and aspect ratios; the Ultralytics YOLO11 family used here is not anchor-based.

**Anchor-free detector.** A detector that predicts box coordinates and class scores directly from feature-map locations or points, without predefined anchor boxes. Ultralytics YOLOv8 and YOLO11 use anchor-free detection heads with decoupled classification and regression branches.

**Backbone.** The convolutional network that extracts feature maps from the input image. In a YOLO detector, the backbone feeds the neck and head; its depth and width largely determine parameter count and latency.

**Bounding box.** An axis-aligned rectangle that localises an object, stored in InfraScan as pixel coordinates `[x1, y1, x2, y2]` with the origin at the top-left. A box says *where* a candidate pothole is, not how deep or severe it is.

**Class imbalance.** A training condition in which some classes have far more examples than others. InfraScan has one foreground class and very few background images (22 of 4,719), so the imbalance of concern is foreground versus empty road, not between defect types.

**Classification.** Predicting a category label for an input or region. Classification alone does not locate anything; InfraScan performs *detection*, which combines localisation and classification.

**Confidence score.** The detector's score for a candidate detection, exposed as a float between 0 and 1. It is a ranking score, **not** a calibrated probability that the box contains a pothole.

**Confidence threshold.** The minimum score a candidate must reach before it is returned. InfraScan defaults to 0.25 and accepts 0.05–1.0; lowering it generally surfaces more candidates (higher recall, lower precision).

**Decoupled head.** A detection head with separate branches for classification and box regression rather than one shared branch. Decoupling is a common design in modern YOLO detectors and is part of the YOLO11 family described by Ultralytics.

**Detection.** The task of locating and classifying objects in an image, usually by returning boxes, class labels and scores. InfraScan performs single-class detection for `pothole`.

**Domain shift.** The mismatch between the data a model was trained on and the data it meets in deployment, such as a different country, camera, weather or road material. Domain shift can degrade accuracy even when test-set metrics are good.

**Head (detection head).** The final network stage that converts feature maps into box coordinates, objectness/class scores and, in some architectures, mask coefficients. InfraScan uses the box head of YOLO11n.

**Heuristic.** A practical rule that is not derived from a validated physical model. InfraScan's severity bands are a heuristic based on box area, not a measurement of depth or structural risk.

**IoU (Intersection over Union).** The area of overlap between two boxes divided by the area of their union:

```text
IoU = area(A ∩ B) / area(A ∪ B)
```

IoU measures localisation quality and is also used by non-maximum suppression to decide whether two boxes are duplicates.

**mAP (mean Average Precision).** The mean of average precision across classes, often averaged over IoU thresholds. With one class, mAP equals the average precision for that class at the chosen threshold(s).

**mAP@0.50.** Average precision computed with a detection counted correct when its IoU with a ground-truth box is at least 0.50. It is lenient about box tightness; InfraScan scores **0.7111** on the 718-image test split.

**mAP@0.50:0.95.** The mean of average precision at ten IoU thresholds from 0.50 to 0.95 in steps of 0.05. It rewards tight localisation; InfraScan scores **0.3938**, substantially below its mAP@0.50.

**Neck.** The network stage between backbone and head that fuses feature maps at different scales. Fusion helps detect both small distant potholes and large near-field ones.

**NMS (non-maximum suppression).** A post-processing step that sorts candidate boxes by score and removes boxes that overlap a stronger box by more than a threshold. It reduces duplicate detections of the same pothole.

**NMS IoU threshold.** The overlap above which NMS suppresses a lower-scoring box. InfraScan defaults to 0.45 and accepts 0.10–1.0; higher values keep more overlapping boxes.

**One-stage detector.** A detector that predicts boxes and classes in a single forward pass, without a separate region-proposal stage. YOLO is the canonical one-stage family; two-stage detectors such as Faster R-CNN first propose regions and then classify them.

**Pothole (task definition).** In this project, a road-surface depression annotated with a single bounding box and class id `0`. The model does not distinguish depth, cause, repair state or severity type.

**Segmentation.** Predicting a pixel-level mask for each object or class. InfraScan does **not** segment; its severity coverage uses the area of a rectangular box, which includes background inside the box.

**YOLO (You Only Look Once).** A family of one-stage object detectors introduced by Redmon et al. (2016) that treats detection as a single regression problem over a grid or feature map. The name is now associated with successive Ultralytics implementations, including the YOLO11n checkpoint used here.

**Zero-build.** A frontend approach with no bundler, transpiler or package manager step. InfraScan serves native ES modules and CSS directly from `web/`.

## Training

**Augmentation.** Synthetic transformations applied to training images to improve robustness and reduce overfitting. InfraScan's training wrapper enables HSV jitter, rotation, translation, scale, perspective, horizontal flip, mosaic and mixup.

**Batch size.** The number of training samples processed before one parameter update. The released checkpoint metadata records batch 16.

**Checkpoint.** A saved file containing model weights and often optimizer/training metadata, used to resume training or run inference. InfraScan expects a compatible `best.pt` at `weights/best.pt` or the path in `INFRASCAN_WEIGHTS`.

**Cosine learning-rate schedule.** A schedule that decays the learning rate following a cosine curve from an initial value to a minimum. The training wrapper sets `cos_lr=True` and `lrf=0.01`; checkpoint metadata records the same schedule.

**Early stopping.** Halting training when a validation metric fails to improve for a set number of epochs. The wrapper defaults to patience 15.

**Epoch.** One complete pass over the training set. Checkpoint metadata records 15 epochs; the script default is 50, so the released checkpoint was not produced by the default command.

**Fine-tuning.** Continuing training of a pretrained model on a target dataset with a small learning rate. InfraScan starts from a YOLO11 checkpoint and trains on the pothole dataset.

**HSV augmentation.** Random perturbation of hue, saturation and value channels to simulate lighting and colour variation. The wrapper uses h 0.015, s 0.7 and v 0.4.

**Learning rate.** The step size used by the optimizer when updating weights. Too high causes divergence; too low slows convergence. Checkpoint metadata records an initial rate of 0.003, while `train.py` defaults to 0.01.

**Mixup.** An augmentation that blends two images and their labels linearly to create a synthetic training example. The wrapper uses mixup 0.15.

**Mosaic.** An augmentation that combines four images into one training canvas, exposing the model to more context and object scales. YOLOv4 popularised it; the wrapper uses mosaic 1.0 and closes it near the end of training.

**Transfer learning.** Reusing representations learned on a large source dataset for a new target task. It is why a small pothole dataset can fine-tune a YOLO11n model effectively.

**Validation split.** Data held out from training to tune hyperparameters and select checkpoints. InfraScan's local validation split has 720 images and 993 instances; it is not the test split.

## Evaluation

**AP (Average Precision).** The area under the precision-recall curve for one class at one IoU threshold, summarising the trade-off across confidence thresholds.

**Background image.** A training or test image whose label file is empty, representing a scene with no target objects. InfraScan has only 22 background images across all splits, which limits false-positive measurement.

**Calibration.** The agreement between predicted confidence and observed correctness. A calibrated 0.8 score should be correct about 80% of the time; InfraScan does not calibrate or report calibration error.

**F1 score.** The harmonic mean of precision and recall:

```text
F1 = 2 × (precision × recall) / (precision + recall)
```

It is useful when both false positives and false negatives matter, but it is not reported in the current evaluation.

**False negative (FN).** A ground-truth pothole that the model did not detect above the threshold. False negatives are missed defects and are reflected in recall.

**False positive (FP).** A predicted box that does not match a ground-truth pothole. False positives consume review time and are reflected in precision.

**Ground truth.** The reference annotations against which predictions are scored. Their quality bounds the meaning of every metric; InfraScan's annotation protocol is not documented.

**Precision.** The fraction of returned detections that are correct:

```text
precision = TP / (TP + FP)
```

InfraScan reports **0.7959** on the test split. High precision means fewer false alarms, not fewer misses.

**Recall.** The fraction of ground-truth objects that were detected:

```text
recall = TP / (TP + FN)
```

InfraScan reports **0.5996**, meaning a substantial share of annotated potholes were missed at the summarised operating point.

**Test split.** Data reserved for final evaluation and not used for tuning. InfraScan's test split has 718 images and 994 instances; the reported metrics come from it.

**True positive (TP).** A detection that matches a ground-truth box at the required IoU and class. The definition of "match" depends on the IoU threshold and the evaluator's matching rules.

## Runtime and backend

**ASGI (Asynchronous Server Gateway Interface).** The Python standard interface between async web servers and applications. FastAPI is an ASGI application; Uvicorn is the ASGI server used here.

**Background job.** Work started after an HTTP response and tracked separately, usually by an identifier. InfraScan's video jobs return `202` and a job id, then run in a daemon worker thread.

**CPU fallback.** Retrying inference on the CPU when the selected accelerator fails. InfraScan retries an image prediction on CPU after an accelerated failure and records the device as CPU.

**CUDA.** NVIDIA's parallel-computing platform and API for GPU execution. The checkpoint metadata records training on CUDA device 0; the acceptance machine was an Apple M2, so the CUDA inference path is unverified.

**Data contract.** The agreed shape and meaning of data exchanged between components. InfraScan's contract is the JSON returned by its endpoints; it is implicit (plain dictionaries) rather than enforced by response models.

**FastAPI.** A Python web framework built on Starlette and Pydantic that derives OpenAPI documentation from type hints. InfraScan uses FastAPI for routing, form validation, static serving and the Swagger UI at `/api/docs`.

**Job queue.** A structure that holds pending work until a worker can run it. InfraScan has a job *registry* and a worker lock, but no persistent queue, priority or cancellation; queued jobs are simply threads waiting for the lock.

**Latency.** The time to complete one operation, such as one image prediction. It is distinct from throughput and depends on hardware, input size, batch size and warm-up.

**Local-first.** A design in which computation and storage happen on the user's machine, with no mandatory cloud dependency. InfraScan is local-first: the model, uploads, outputs and telemetry stay on the host.

**MPS (Metal Performance Shaders).** Apple's GPU acceleration framework, exposed in PyTorch through the `mps` backend. The verified evaluation ran on an Apple M2 using MPS.

**Multipart form data.** An HTTP encoding that carries file bytes and ordinary form fields in one request. InfraScan's detection endpoints receive `file`, `conf` and `iou` as multipart form data.

**OpenAPI.** A machine-readable specification of an HTTP API's paths, parameters and schemas. FastAPI generates it at `/api/openapi.json` and renders it through Swagger UI.

**Starlette.** The ASGI framework/toolkit underlying FastAPI. InfraScan imports `run_in_threadpool` from Starlette to run blocking inference without blocking the event loop.

**Swagger UI.** A browser interface that renders an OpenAPI schema and allows interactive requests. InfraScan exposes it at `/api/docs`.

**Throughput.** The amount of work completed per unit time, such as frames per second. It is the inverse of average latency only under steady, parallel-free conditions; InfraScan reports a video throughput estimate from frames divided by elapsed time.

**TTL (time to live).** The age after which stored data is considered stale and eligible for deletion. InfraScan uses 24-hour TTLs for result images and finished job directories, applied at startup.

**Uvicorn.** A fast ASGI server implementation. `run_app.py` launches Uvicorn with `app:app`, a host, a port and optional reload.

## Frontend and security

**CSP (Content Security Policy).** An HTTP response header that restricts which origins may supply scripts, styles, images, fonts, media and connections. InfraScan sets a restrictive same-origin CSP, with `style-src 'unsafe-inline'` because some dynamic widths are inline.

**CSRF (Cross-Site Request Forgery).** An attack that tricks a logged-in browser into sending an unwanted request to another origin. InfraScan has no CSRF token; its main defence is the loopback bind and the absence of authenticated state.

**ES module (ECMAScript module).** A native JavaScript module system using `import` and `export`, loaded by the browser with `<script type="module">`. InfraScan's frontend is a set of ES modules with no bundler.

**Hash routing.** Client-side routing that stores the current view in the URL fragment (for example `#/inspect`) so the browser does not request a new document. InfraScan uses hash routes for Overview, Inspect, Video and System.

**SPA (Single-Page Application).** A web application that loads one HTML document and updates the view client-side. InfraScan's SPA is served from `web/index.html` and composed of vanilla ES modules.

**Zero-build frontend.** See **Zero-build** under Detection and model.

## Media and operations

**Container (MP4).** A file format that holds encoded video, audio, subtitles and metadata. MP4 is a container; H.264 is a codec that can be stored inside it. InfraScan writes an MP4 container with an H.264 or `mp4v` video stream.

**H.264 / AVC.** A widely supported video compression standard that browsers can usually play inside MP4. InfraScan tries the `avc1` encoder first so the annotated video plays in Chrome; the verified 24-frame output used H.264.

**mp4v.** An MPEG-4 Part 2 video codec sometimes available in OpenCV builds. InfraScan falls back to `mp4v` when `avc1` cannot be opened; the resulting file may download but not play in a browser.

**Telemetry.** Structured records describing detections and their context. InfraScan exports CSV and JSON with frame/image, box, confidence, severity and coverage fields; it does not include GPS, timestamps or track ids.

**Severity coverage.** The percentage of the full frame area covered by a detection box:

```text
coverage_pct = box_area / frame_area × 100
```

It drives the CRITICAL/MODERATE/MINOR band. It is a geometric proxy, not a physical severity measurement.

**Snapshot / result artefact.** The annotated JPEG or MP4 written under `runs/dashboard/`. Results are addressed by UUID and expire under the TTL policy.

---

## Acronyms at a glance

| Acronym | Expansion |
|---|---|
| AP | Average Precision |
| ASGI | Asynchronous Server Gateway Interface |
| CSP | Content Security Policy |
| CSRF | Cross-Site Request Forgery |
| CUDA | Compute Unified Device Architecture |
| FN / FP / TP | False Negative / False Positive / True Positive |
| FPS | Frames Per Second |
| HSV | Hue, Saturation, Value |
| IoU | Intersection over Union |
| mAP | Mean Average Precision |
| MPS | Metal Performance Shaders |
| NMS | Non-Maximum Suppression |
| OpenAPI | Open Application Programming Interface specification |
| SPA | Single-Page Application |
| TTL | Time To Live |
| YOLO | You Only Look Once |
