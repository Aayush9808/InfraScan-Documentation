# Dataset provenance

> **Status: partially verified.** The local files, counts and label format are verified. The original creators, source URLs, licences and redistribution permissions are **not** verified. This page separates those two layers deliberately.

The main repository's inherited documentation describes a multi-source pothole dataset and ships preparation utilities for both a mask/segmentation-style source and a Pascal VOC-style archive. The current local `dataset/` is a YOLO detection dataset. Those statements are consistent with a merged, multi-source dataset, but they are not a provenance record.

## 1. What is present

The dataset is excluded from Git by `.gitignore` (lines 41–43) and is present only on the development machine. The verified structure is:

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

| Split | Images | Pothole instances | Background images (empty labels) |
|---|---:|---:|---:|
| Train | 3,281 | 5,353 | 18 |
| Validation | 720 | 993 | 2 |
| Test | 718 | 994 | 2 |
| **Total** | **4,719** | **7,340** | **22** |

Source: local directory enumeration and label-file counts, corroborated by the evaluation log summarised in [`../PROJECT_FACTS.md`](../PROJECT_FACTS.md) §4–§5.

`data.yaml` defines one class and relative split paths:

```yaml
path: ./dataset
train: images/train
val: images/val
test: images/test
nc: 1
names:
- pothole
```

Labels use the Ultralytics YOLO detection format:

```text
<class_id> <x_center> <y_center> <width> <height>
```

Coordinates are normalised to `[0, 1]` relative to image width and height. A sample local test label contains:

```text
0 0.441406 0.690625 0.364063 0.159375
0 0.554688 0.349219 0.743750 0.257812
```

The first line is used as a worked example in [annotation and preprocessing](annotation-and-preprocessing.md).

## 2. Filename families

The local test split is **not** a single uniform naming family. Enumerating the filenames gives five families:

| Filename family | Train | Val | Test | Total |
|---|---:|---:|---:|---:|
| `pothole_dataset_1_train_image*` | 407 | 45 | 57 | 509 |
| `pothole_dataset_1_valid_image*` | 73 | 15 | 4 | 92 |
| `pothole_dataset_2_*` | 781 | 98 | 98 | 977 |
| `pothole_dataset_3_*` | 1,488 | 496 | 492 | 2,476 |
| `voc_potholes*` | 532 | 66 | 67 | 665 |
| **Total** | **3,281** | **720** | **718** | **4,719** |

These counts come from `ls dataset/images/{train,val,test}` on the local copy. The family labels are **filename heuristics**, not proof of the original source.

What the filenames show:

- The test split contains `pothole_dataset_1_*`, `pothole_dataset_2_*`, `pothole_dataset_3_*` and `voc_potholes*` files. Examples include `pothole_dataset_1_train_image10.jpg`, `pothole_dataset_2_00001.jpg`, `pothole_dataset_3__0081_...jpg` and `voc_potholes103.jpg`.
- The token `train` or `valid` inside a filename is an **inherited upstream split token**, not the current split. For example, `pothole_dataset_1_train_image*` files appear in train, validation and test; `pothole_dataset_1_valid_image*` files also appear in all three.
- All 22 empty-label background images in the inspected local copy belong to the `pothole_dataset_1_train_image` family.
- The `voc_potholes*` images retain original dimensions in sampled files (for example `467 × 300` and `400 × 400`), while sampled `pothole_dataset_1_*` images are `640 × 640`. This is consistent with two different preparation paths, but does not prove which path produced every file.

> **Do not use filename prefixes to infer the split, the source dataset, or whether an image is synthetic or real.** The prefixes are useful for auditing; they are not labels.

## 3. What can be verified

| Claim | Evidence | Strength |
|---|---|---|
| Local folder structure and split names | `dataset/images/*`, `dataset/labels/*`; `data.yaml` | Direct inspection |
| Image and label counts | Directory enumeration; `PROJECT_FACTS.md` §4 | Direct inspection |
| Background-image count | Empty `.txt` files: 18 / 2 / 2 | Direct inspection |
| One class named `pothole` | `data.yaml` lines 5–7; label class IDs | Direct inspection |
| Normalised YOLO label format | Sample labels; preparation scripts | Direct inspection |
| Multiple filename families | Filename enumeration table above | Derived from filenames |
| Checkpoint training-data path | Checkpoint metadata; see §5 | Metadata inspection |
| Test metrics | 718-image evaluation; `PROJECT_FACTS.md` §5 | Executed evaluation |
| Dataset/checkpoint excluded from Git | `.gitignore` lines 41–43 | Source inspection |

## 4. What cannot be verified

| Missing field | Why it matters | Status |
|---|---|---|
| Original dataset name(s) | Determines what may be cited and what the counts mean | `[TO BE CONFIRMED]` |
| Creator(s) and curator(s) | Required for attribution and permission requests | `[TO BE CONFIRMED]` |
| Stable source URL(s) | Required to reproduce the collection | `[TO BE CONFIRMED]` |
| Licence for each source | Determines whether images and derived labels may be shared | `[TO BE CONFIRMED]` |
| Permission for processed copies | The local JPEGs and YOLO labels are derivative artifacts | `[TO BE CONFIRMED]` |
| Permission for redistribution | Neither repository grants it | `[TO BE CONFIRMED]` |
| Per-file manifest | Needed to map every image to a source and split | Not present |
| Exact split protocol | The local split does not match one clean 80/10/10 run; see §6 | `[TO BE CONFIRMED]` |
| Duplicate/leakage audit | Filename tokens are inherited; byte-identical or near-duplicate images may cross splits | Not recorded |
| Synthetic vs real composition | Inherited prose mentions multiple sources; no per-file source-type field exists | `[TO BE CONFIRMED]` |
| Privacy/consent review | Road images may contain faces, plates or identifiable locations | Not recorded |
| Collection dates and locations | Not present in the local tree | `[TO BE CONFIRMED]` |

No source URL, creator list or licence is invented on this page. If a future provenance record is added, it should be linked here and in [`../PROJECT_FACTS.md`](../PROJECT_FACTS.md).

## 5. Checkpoint metadata data path

The checkpoint metadata records a Windows training path. The exact string recovered from `weights/best.pt` is:

```text
C:\Users\Bisman Singh\.gemini\antigravity-ide\scratch\YOLO-Pothole-Detection-Model\data.yaml
```

This is evidence of the **training environment** and of the upstream project name. It is not evidence that:

- the current local dataset is identical to the dataset used for training;
- the dataset may be redistributed;
- the named upstream repository owns every image;
- the checkpoint may be shared.

The path also shows why a Windows-to-macOS reproduction needs an explicit `--data` path. `data.yaml` is portable because its `path` is relative (`./dataset`), but the embedded training path is not.

## 6. Split arithmetic and why it matters

The local split proportions are:

| Split | Images | Share |
|---|---:|---:|
| Train | 3,281 | 69.5% |
| Validation | 720 | 15.3% |
| Test | 718 | 15.2% |

These are close to **70/15/15**, not 80/10/10.

The VOC ingestion script (`scripts/ingest_voc_dataset.py`) uses 80/10/10 with integer flooring and assigns the remainder to test. For 4,719 matched pairs it would produce:

```text
train = int(4719 × 0.80) = 3775
val   = int(4719 × 0.10) =  471
test  = 4719 − 3775 − 471 = 473
```

That does **not** match the local `3,281 / 720 / 718`. Therefore the local dataset was not produced by a single clean 80/10/10 VOC ingestion run over all 4,719 current images. Possible explanations include multiple ingestion runs, merging with the mask/contour pipeline, a later resplit, or additional data. The exact history is `[TO BE CONFIRMED]`.

This is also why the chapter does not describe the split as "the VOC split" or "the upstream split". It is simply the **local split used for the recorded evaluation**.

## 7. Inherited conflation of sources

The word "conflation" here means that several preparation paths and naming families have been merged into one local dataset. It does not mean the images are bad; it means the provenance cannot be reconstructed from the current tree alone.

Evidence of merging:

1. **Two preparation scripts.** `prepare_dataset.py` converts grayscale masks to contours and then bounding boxes. `ingest_voc_dataset.py` converts Pascal VOC XML bounding boxes. They write different filename prefixes (`<stem>.jpg` versus `voc_<stem>.jpg`) and different image handling (square resize versus original dimensions).
2. **At least five filename families** in the local tree, including `pothole_dataset_1_*`, `pothole_dataset_2_*`, `pothole_dataset_3_*` and `voc_potholes*`.
3. **Inherited split tokens.** `train_image` and `valid_image` appear in every current split, so the upstream split boundaries were not preserved.
4. **Mixed image dimensions.** Sampled `pothole_dataset_1_*` files are `640 × 640`; sampled `voc_potholes*` files are smaller and non-square. Normalised labels make them coexist, but they did not follow one resize path.
5. **Split arithmetic mismatch.** A single 80/10/10 VOC run would not produce the local counts.
6. **Checkpoint path.** The metadata points to an upstream `YOLO-Pothole-Detection-Model` working directory, not to the current repository layout.

The inherited prose may describe synthetic, real and VOC sources. The current tree does **not** contain a per-file field that identifies which images are synthetic, which are real photographs, or which VOC archive they came from. Do not state a synthetic/real ratio as fact.

## 8. Missing provenance fields

A usable provenance record would need, at minimum:

- [ ] Source dataset name and version.
- [ ] Creator/curator names and contact or institution.
- [ ] Stable URL or DOI for each source.
- [ ] Licence text and redistribution permission.
- [ ] Permission for derived labels, resized copies and annotated outputs.
- [ ] Per-file manifest: original filename, source, licence, split, SHA-256.
- [ ] Split-generation command and seed.
- [ ] Duplicate/leakage audit across splits.
- [ ] Synthetic/real/source-type field per image.
- [ ] Privacy review for faces, number plates and identifiable locations.
- [ ] Date of collection and any consent terms.
- [ ] Decision on whether examiners may receive a private copy.

Until these exist, the honest description is:

> "A local, single-class pothole detection dataset assembled from inherited material. It contains 4,719 images and 7,340 annotated pothole boxes across train/validation/test splits of 3,281/720/718. The original sources and redistribution rights are not verified."

## 9. Reference to the upstream project

The repository's `LICENSE` file is an MIT licence with the copyright line:

```text
Copyright (c) 2026 Bisman Singh
```

The inherited upstream repository is:

<https://github.com/Bisman-Singh-Dev/YOLO-Pothole-Detection-Model>

That link is preserved as **attribution for inherited material**, not as proof that every image, label or weight originated there. The upstream repository's own dataset and model terms must be checked separately. See [licensing and attribution](licensing-and-attribution.md) and [third-party attribution](../10-references-and-governance/third-party-attribution.md).

## 10. Citation and redistribution guidance

- Cite the **local dataset description** and the counts in this chapter, not an invented public dataset.
- Do not claim CC BY, CC0, MIT or any other licence for the images or labels.
- Do not upload `dataset/` to GitHub, cloud storage or a public release.
- Do not share the checkpoint as if it were a clean, licensed model release.
- If an examiner needs the data, prefer a private, permissioned copy with a written explanation of the unresolved provenance.
- Keep the distinction between **the dataset used for training** and **the local copy used for evaluation** explicit. The checkpoint metadata path is not a dataset manifest.

For the evaluation evidence that uses this split, see [`../09-testing-and-results/verified-results.md`](../09-testing-and-results/verified-results.md).
