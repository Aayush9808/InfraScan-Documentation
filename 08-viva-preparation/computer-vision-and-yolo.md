# Computer vision and YOLO questions

Twenty-five questions on the detector, the metrics and the limits of the visual pipeline. Answers are written for an undergraduate viva, with the precise caveats an examiner is likely to probe.

---

### Q1. What is object detection?

Object detection predicts both **where** objects are and **what** they are. The output is a set of bounding boxes, each with a class label and a confidence score. It differs from classification (one label for the whole image) and segmentation (a pixel-level mask).

### Q2. Why is this a single-class detection problem?

The project scope is potholes only. `data.yaml` defines `nc: 1` and `names: [pothole]`, so the model learns one foreground class. Background regions are implicit negatives; two test images have empty label files and act as background examples. Multi-class road-damage detection is future work.

### Q3. What is a single-stage detector?

A single-stage detector predicts boxes and class scores in one network pass, without a separate region-proposal stage. YOLO is the canonical family. The trade-off is usually faster inference and a simpler pipeline, with potentially weaker localization on difficult small objects compared with multi-stage detectors.

### Q4. At a high level, what are the parts of YOLO11?

The Ultralytics YOLO11 detector has a **backbone** that extracts multi-scale features, a **neck** that fuses those features, and a **detection head** that predicts box coordinates and class scores. The exact block types and channel widths are Ultralytics implementation details; the project uses the pretrained nano variant and does not modify the architecture.

### Q5. Is YOLO11 anchor-based or anchor-free?

The Ultralytics YOLO11 detection head is anchor-free in the modern Ultralytics lineage: it predicts boxes directly rather than matching hand-specified anchor boxes. For the viva, the safe explanation is that the head predicts objectness/class scores and box offsets at multiple feature levels; do not invent layer-level details beyond the installed documentation.

### Q6. What do “2.58M parameters” and “6.4 GFLOPs” mean?

Parameters are learned weights and biases; the fused checkpoint has **2,582,347**. GFLOPs estimates computational cost for one forward pass at the reference input size; the model reports **6.4 GFLOPs**. Parameters measure model size/capacity, not accuracy; GFLOPs measure arithmetic work, not wall-clock latency.

### Q7. Why is the input size 640?

640 px is the Ultralytics default and the size recorded in the checkpoint metadata and evaluation command. It balances detail for potholes against memory and latency. A larger input could help small objects but increases computation; a smaller input is faster but may lose small potholes. The project did not run a multi-resolution ablation, so it cannot claim 640 is optimal.

### Q8. What is confidence?

Confidence is the model's score for a predicted box/class. It is used as a ranking and filtering signal. It is **not** a calibrated probability that the box is a pothole. The dashboard default threshold is 0.25; lowering it increases detections (and false positives), raising it increases precision but can reduce recall.

### Q9. What is IoU?

Intersection over Union is the area of overlap between a predicted box and a ground-truth box divided by their union area. It ranges from 0 to 1. A prediction is a true positive at a given IoU threshold if it matches an unmatched ground-truth box of the same class with IoU at least that threshold.

### Q10. What is NMS?

Non-Maximum Suppression removes overlapping duplicate detections. Candidates are sorted by confidence; the highest-scoring box is kept, and boxes with IoU above a threshold relative to it are suppressed. The dashboard exposes NMS IoU (default 0.45). Too low can merge distinct nearby potholes; too high can leave duplicates.

### Q11. What is precision?

Precision = TP / (TP + FP). It answers: of the detections the model reported, how many were correct? The verified test precision is **0.7959**. Precision depends on the confidence threshold and the matching rule, so it must always be reported with the evaluation setting.

### Q12. What is recall?

Recall = TP / (TP + FN). It answers: of the real potholes, how many did the model find? The verified test recall is **0.5996**, meaning about 60% were matched at the evaluation operating point. This is the project's most important accuracy limitation; a screening tool that misses 40% of test potholes is not a certified inspection instrument.

### Q13. What is Average Precision?

For one class, Average Precision is the area under the precision–recall curve, usually computed across confidence thresholds. It summarizes the trade-off between finding positives and avoiding false positives. The PR curve in `../assets/evaluation/pr-curve.png` is the visual evidence.

### Q14. What is mAP@0.50?

Mean Average Precision at IoU 0.50 averages AP across classes; here there is one class, so it is effectively AP@0.50. A prediction counts as correct if its IoU with ground truth is at least 0.50. The verified value is **0.7111**.

### Q15. What is mAP@0.50–0.95?

It averages mAP over ten IoU thresholds: 0.50, 0.55, …, 0.95. It rewards tighter localization. The verified value is **0.3938**, much lower than mAP@0.50. That gap says the boxes often overlap enough at a loose threshold but are not precise at stricter thresholds.

### Q16. Why report both mAP values?

mAP@0.50 is easier to compare with many detection papers, while mAP@0.50–0.95 is stricter and more informative about localization. Reporting both prevents a misleadingly optimistic single number. The project reports both and does not claim a new state of the art.

### Q17. What does the PR curve show?

It plots precision against recall as the confidence threshold changes. A curve that stays high as recall increases is better. The exact curve shape depends on the test split, confidence sweep and IoU criterion. The image is evidence from the verified run; it is not a generic illustration.

### Q18. What is a background image in this dataset?

A background image has no pothole instances, so its label file is empty. It teaches the model that a road scene without a pothole should produce no detections, and it contributes false positives if the model fires. The test split contains two background images.

### Q19. What augmentation was used?

The checkpoint metadata records Ultralytics defaults such as mosaic, HSV jitter, translation/scale, flips and `close_mosaic=3`. `train.py` exposes a set of augmentation flags. The exact effective values are those in the checkpoint's `train_args`, not assumptions from a different run. Augmentation improves robustness but can create unrealistic pothole geometry if pushed too far.

### Q20. What is transfer learning here?

The training starts from a pretrained YOLO11n checkpoint and fine-tunes it on the pothole dataset, rather than training from random weights. This is why `train.py` defaults to a pretrained model. The project's reported checkpoint is a fine-tuned result; the exact pretrained parent is not independently documented beyond the metadata.

### Q21. What is the difference between training and inference?

Training updates weights using labelled examples and a loss. Inference freezes the weights and predicts on new inputs. The dashboard only performs inference; `train.py` is a separate CLI path. A model can be good at training loss and still generalize poorly, which is why the held-out test split matters.

### Q22. Why does device selection try CUDA → MPS → CPU?

The code is portable across environments. CUDA is preferred on NVIDIA hardware, MPS on Apple Silicon, and CPU is the fallback. If a preferred device fails at prediction time, the inference service can retry on CPU. The verified evaluation used MPS; CUDA behaviour is supported by code but **not benchmarked** here.

### Q23. What is the difference between FP32 and FP16?

FP32 uses 32-bit floating point; FP16 uses 16-bit and can be faster/lower-memory on supported hardware, with some numerical risk. Ultralytics can use half precision on GPU. The verified evaluation settings should be reported exactly; do not claim FP16 speed unless it was measured.

### Q24. What is a checkpoint?

A checkpoint is a serialized model state plus metadata. The local `best.pt` contains the YOLO11n weights, class names, training arguments and version information. It is **not** the training dataset, and it is not redistributed by the repository. Its SHA-256 is recorded in `../PROJECT_FACTS.md` for identification.

### Q25. Where does the model fail visually?

Likely failure modes include small or distant potholes, partial occlusion, shadows, wet reflective surfaces, motion blur, unusual pavement textures, extreme lighting and domain shift from the training images. These are hypotheses grounded in the task and the measured recall, not a formal failure-mode study. A proper study would stratify the test set by size, lighting and road type.

---

## Follow-up traps

- **“If precision is 80%, why not deploy it?”** Precision is split- and threshold-specific; recall is 60%; severity is heuristic; there is no safety certification, authentication or field validation.
- **“Why not train longer?”** The checkpoint metadata records 15 epochs; longer training might help or overfit. It requires a controlled experiment and a fresh evaluation, not an assumption.
- **“Can confidence be used as risk?”** No. It is a model score, not calibrated probability or physical severity.
- **“Why is mAP@0.50–0.95 so low?”** Loose IoU matches are easier; strict localization is harder. Small/irregular potholes and box ambiguity contribute, but the project has not run an ablation to quantify causes.
- **“Is 640 px enough for small potholes?”** Not always. It is a trade-off; a tiling or higher-resolution experiment would be needed to claim improvement.

---

[Viva index](README.md) · [Project overview](project-overview-questions.md) · [Dataset and training](dataset-and-training-questions.md) · [Model explanation](../05-dataset-and-model/model-explanation.md)
