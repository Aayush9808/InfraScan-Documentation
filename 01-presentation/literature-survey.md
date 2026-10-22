# Literature survey

This table follows the GLBITM literature-survey format and is limited to sources that can be checked. The references are **background**, not evidence that InfraScan reproduced their reported results.

| S. No. | Date / year | Author(s) | Title / publication | Conceptual framework | Methodology | Conclusion | Future research / relevance |
|---:|---|---|---|---|---|---|---|
| 1 | 2016 | Redmon, Divvala, Girshick & Farhadi | *You Only Look Once: Unified, Real-Time Object Detection*, CVPR 2016 | Single-stage object detection | Predict bounding boxes and class probabilities directly from the full image in one network pass | Established a direct, real-time detection formulation | Better localization, small-object handling and modern backbone/neck designs; conceptual basis for YOLO-family detectors |
| 2 | 2020 | Bochkovskiy, Wang & Liao | *YOLOv4: Optimal Speed and Accuracy of Object Detection*, arXiv:2004.10934 | Accuracy/speed co-design | Backbone, neck, head and training improvements (CSPDarknet, PANet, mosaic, etc.) | Strong real-time detection trade-offs | Hardware/domain-specific tuning; useful vocabulary for backbone–neck–head explanations |
| 3 | 2018 | Maeda, Sekimoto, Seto, Kashiyama & Omata | *Road Damage Detection Using Deep Neural Networks with Images Captured Through a Smartphone*, arXiv:1801.09450 | Mobile road inspection | Deep detection/classification on smartphone road images | Demonstrated feasibility of vision-based road assessment | Larger, geographically diverse datasets; motivates pothole detection but does not validate InfraScan's checkpoint |
| 4 | 2010 | Everingham, Van Gool, Williams, Winn & Zisserman | *The PASCAL Visual Object Classes (VOC) Challenge*, IJCV 88(2), 303–338 | Dataset/annotation benchmark | Standardized images, bounding boxes, XML annotations and evaluation protocols | Established shared conventions for detection evaluation | Explains the VOC XML format used by `scripts/ingest_voc_dataset.py`; not a licence for any particular dataset |
| 5 | 2014 | Lin et al. | *Microsoft COCO: Common Objects in Context*, ECCV 2014 | Large-scale detection/segmentation benchmark | Rich annotations and COCO-style evaluation | Established widely used detection evaluation conventions | Background for mAP/precision–recall interpretation; InfraScan uses Ultralytics evaluation, not a COCO claim |
| 6 | 2023– | Ultralytics | *Ultralytics YOLO documentation* | Practical detection framework | Training, validation, prediction, export APIs | Provides the implementation tooling used by this project | Version-sensitive; cite the installed version (8.4.183) rather than assuming behaviour across releases |
| 7 | 2024– | Jocher, Chaurasia & Qiu | *Ultralytics YOLO11* documentation/repository | Modern YOLO family | Detection models and training/export workflows | Describes the YOLO11 model family used here | The checkpoint metadata and loaded model confirm YOLO11n; do not claim architectural details that are not documented |

## Formatted references

Bochkovskiy, A., Wang, C.-Y., & Liao, H.-Y. M. (2020). *YOLOv4: Optimal speed and accuracy of object detection*. arXiv. https://arxiv.org/abs/2004.10934

Everingham, M., Van Gool, L., Williams, C. K. I., Winn, J., & Zisserman, A. (2010). The PASCAL Visual Object Classes (VOC) challenge. *International Journal of Computer Vision, 88*(2), 303–338. https://doi.org/10.1007/s11263-009-0275-4

Jocher, G., Chaurasia, A., & Qiu, J. (2024). *Ultralytics YOLO11* [Computer software]. https://github.com/ultralytics/ultralytics

Lin, T.-Y., Maire, M., Belongie, S., Hays, J., Perona, P., Ramanan, D., Dollár, P., & Zitnick, C. L. (2014). Microsoft COCO: Common objects in context. In *Computer Vision — ECCV 2014* (pp. 740–755). Springer. https://doi.org/10.1007/978-3-319-10602-1_48

Maeda, H., Sekimoto, Y., Seto, T., Kashiyama, T., & Omata, H. (2018). *Road damage detection using deep neural networks with images captured through a smartphone*. arXiv. https://arxiv.org/abs/1801.09450

Redmon, J., Divvala, S., Girshick, R., & Farhadi, A. (2016). You only look once: Unified, real-time object detection. In *Proceedings of the IEEE Conference on Computer Vision and Pattern Recognition* (pp. 779–788). https://doi.org/10.1109/CVPR.2016.91

Ultralytics. (n.d.). *Ultralytics documentation*. https://docs.ultralytics.com/

Ultralytics. (n.d.). *Licensing*. https://www.ultralytics.com/license

## How to use this in the viva

- If asked “what is YOLO?”, use reference 1 and explain the single-pass idea.
- If asked “why these terms backbone/neck/head?”, use reference 2.
- If asked “why road damage is a computer-vision problem?”, use reference 3.
- If asked “where did the XML format come from?”, use reference 4; explain that the VOC format is a convention, not a dataset licence.
- If asked “what is mAP?”, use reference 5 for the evaluation concept, then point to the verified Ultralytics output.
- If asked “which YOLO version?”, use the checkpoint metadata + reference 7; say YOLO11n, not “YOLOv8” or “YOLOv5.”

## Caveats

- Access date for online sources in this repository: **22 October 2026**.
- The literature survey does not establish the licence of the local dataset or checkpoint.
- Citation details should be checked against the publisher before final submission.
- Do not add a paper merely to increase the row count. A shorter verified table is stronger.

---

[Presentation index](README.md) · [Slide content](slide-by-slide-content.md) · [Bibliography](../10-references-and-governance/bibliography.md)
