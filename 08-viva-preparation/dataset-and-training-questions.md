# Dataset and training questions

Twenty questions for dataset, training, evaluation and provenance. Kajal and Ronak should be able to answer the data-quality questions; Aayush should also be able to explain the optimization and evaluation pipeline.

---

### Q1. What dataset was used?

The local development copy contains **4,719 images** and **7,340 pothole boxes**, split into train, validation and test. It is a single-class dataset with a `pothole` label. The exact upstream creators, source URLs and licences are **not fully verified**, so the documentation describes the local structure and counts without claiming redistribution rights.

### Q2. What are the split counts?

| Split | Images | Instances | Background images |
|---|---:|---:|---:|
| Train | 3,281 | 5,353 | 18 |
| Validation | 720 | 993 | 2 |
| Test | 718 | 994 | 2 |
| **Total** | **4,719** | **7,340** | **22** |

These counts were checked directly and match the evaluation log's test count of 718 images, 994 instances and 2 backgrounds.

### Q3. What is the directory structure?

```text
dataset/
├── images/{train,val,test}/
└── labels/{train,val,test}/
```

Images and label files are paired by stem: `foo.jpg` corresponds to `foo.txt`. The YAML uses relative paths under `path: ./dataset`. The dataset is excluded from Git, so a fresh clone cannot run evaluation until the data is supplied.

### Q4. What is the YOLO label format?

Each line is:

```text
class_id x_center y_center width height
```

Coordinates are normalized to the image width/height, in `[0, 1]`. Class ID `0` is `pothole`. An empty label file means a background image. This differs from VOC XML (absolute pixel boxes) and COCO JSON (absolute coordinates plus image metadata).

### Q5. How does `prepare_dataset.py` work?

It expects a table with image, mask and split columns. For each mask, it finds external contours with OpenCV, filters contours below a minimum area, computes the bounding rectangle, normalizes the box to the image size, and writes a YOLO label file. It copies/resizes the source image and skips missing/unreadable pairs. This is a **mask-to-box** conversion, not a segmentation-label exporter.

### Q6. How does `ingest_voc_dataset.py` work?

It reads VOC-style XML annotations, extracts `bndbox` coordinates, converts absolute `(xmin, ymin, xmax, ymax)` to normalized center/width/height, copies images, writes labels and creates an 80/10/10 train/val/test split. It supports archive-style ingestion. Because the split is random, it should be checked for near-duplicate leakage across splits.

### Q7. What is a background image and why include it?

A background image has no pothole; its label file is empty. It provides negative examples, so the model sees road scenes that should produce no detection. False positives on backgrounds directly hurt precision. The local test split includes two such images, which is a very small sample for measuring background false-positive behaviour.

### Q8. How was the model trained?

The checkpoint metadata records **15 epochs**, batch **16**, image size **640**, optimizer `auto`, initial LR **0.003**, cosine LR schedule (`cos_lr=True`, `lrf=0.01`), seed 0, `close_mosaic=3`, and device `0` (CUDA). The training ran in a Windows path embedded in the checkpoint metadata. `train.py` is the project wrapper; its defaults are not necessarily the exact values used for this checkpoint, so the metadata is the authoritative record.

### Q9. What is the difference between `train.py` defaults and the checkpoint metadata?

`train.py` is a reusable CLI. It has its own defaults (for example, a default model and learning rate) and exposes arguments. The checkpoint metadata records what was actually used for this model. In a viva, do not confuse “the script can be run with X” with “the reported checkpoint used X.” Quote the metadata for the reported result.

### Q10. What augmentation was used?

The metadata records augmentation values including mosaic, HSV jitter, translation/scale, flips, mixup, and `close_mosaic=3`. These increase variation and can improve robustness, but they can also create unrealistic combinations. The project did not run an augmentation ablation, so it cannot claim each transform contributed to the final mAP.

### Q11. Why use a validation split?

Validation is used during development to monitor generalization and tune choices without touching the test set. The test split is held out for the final evaluation. The verified metrics are from the **test** split, not the validation split; do not mix the numbers.

### Q12. What is data leakage?

Leakage occurs when information from the test set influences training or model selection, or when near-duplicate images appear in different splits. It inflates the reported result. The local split counts are verified, but the project has not run a formal near-duplicate audit. This is an honest limitation, especially if the dataset was assembled from multiple sources.

### Q13. How do you check label quality?

Check that every image has a matching label (or is intentionally background), every class ID is valid, every coordinate is in `[0, 1]`, boxes have nonzero width/height, boxes are not degenerate or outside the image, and the box visually covers the pothole. `scripts/visualize_annotations.py` draws labels for inspection. These checks are necessary before trusting a training run.

### Q14. Why is class imbalance not a major issue here?

There is only one foreground class, so there is no multi-class imbalance. There is still a foreground/background imbalance: most image regions are negative. The detector handles this through its training objective and hard-negative mining in the loss, but the project did not run a class-balance ablation. Empty-label backgrounds are included to expose the model to negatives.

### Q15. What is the evaluation command?

The verified run used:

```bash
python evaluate.py \
  --weights weights/best.pt \
  --data <test-data.yaml> \
  --split test \
  --imgsz 640 \
  --batch 16 \
  --device mps
```

The normal repository path uses `data.yaml`; the verified run used a temporary data copy because the iCloud-backed dataset had evicted files. The command, split and environment are part of the evidence.

### Q16. What were the verified results?

Precision **0.7959**, recall **0.5996**, mAP@0.50 **0.7111**, mAP@0.50–0.95 **0.3938**, on **718 images / 994 instances / 2 backgrounds**. Preprocess **0.17 ms**, inference **3.82 ms**, postprocess **10.75 ms** per image on an Apple M2 with MPS. These are not CUDA numbers.

### Q17. Why is mAP@0.50–0.95 much lower?

The stricter metric requires tighter box overlap across ten IoU thresholds. A model can find the pothole but produce a loose or shifted box, which still passes IoU 0.50 but fails 0.75+. Small, irregular or ambiguous potholes make precise localization difficult. The gap is evidence about localization quality, not a reason to hide the stricter number.

### Q18. How would you improve the result?

Collect more representative and licensed data, audit labels and duplicates, train longer with a controlled schedule, try a larger YOLO variant, tune confidence/NMS on validation only, and evaluate on a held-out test set. Each change requires a new run and a new evidence record. Do not claim improvement from a plan alone.

### Q19. What are the dataset/licensing risks?

The dataset's original creators, stable URLs, licences and redistribution permissions are unresolved. The checkpoint's metadata points to a training path but does not grant redistribution rights. The repository excludes both assets. For a public release or paper, the team would need documented permission or a replacement dataset with a clear licence.

### Q20. What would a stronger dataset chapter include?

A provenance table per source (creator, URL, licence, count, collection method), annotation guidelines, inter-annotator agreement, duplicate/near-duplicate analysis, split-generation script and seed, class/size/lighting distributions, and a held-out field test. The current chapter is honest about which of these are missing.

---

## Cross-examination prompts

- “Did you create the dataset?” → The team assembled/prepared the local copy and scripts, but original source ownership and licences are unverified. Do not claim authorship of every image.
- “Why only 15 epochs?” → That is what the checkpoint metadata records; the reason is not documented. Longer training is a future experiment, not a proven improvement.
- “Is the validation set used for the final metric?” → No; the verified metrics are test-split metrics.
- “Did you check duplicates?” → Not formally; this is a known gap.
- “Can you publish the checkpoint?” → Not without clarifying Ultralytics licensing and the training-data rights.

---

[Viva index](README.md) · [Dataset provenance](../05-dataset-and-model/dataset-provenance.md) · [Training and evaluation](../05-dataset-and-model/training-and-evaluation.md) · [Verified results](../09-testing-and-results/verified-results.md)
