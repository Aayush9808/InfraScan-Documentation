# Bibliography

The sources behind InfraScan. Section A is the numbered list cited by the [project report](../03-project-documentation/project-report.md) — the same numbers, the same order. Section B is further reading for the viva and for anyone extending the work. Section C records the references the project deliberately does not make.

**Access date for all web sources: 22 October 2026.**

---

## Section A — References cited by the project report

### A.1 Pavement condition assessment standards

[1] ASTM International. (2020). *ASTM D6433-20: Standard practice for roads and parking lots pavement condition surveys*. <https://www.astm.org/d6433-20.html>

### A.2 Road-damage detection with deep learning

[2] Maeda, H., Sekimoto, Y., Seto, T., Kashiyama, T., & Omata, H. (2018). Road damage detection using deep neural networks with images captured through a smartphone. *arXiv*. <https://arxiv.org/abs/1801.09450>

### A.3 Object detection

[3] Ren, S., He, K., Girshick, R., & Sun, J. (2015). Faster R-CNN: Towards real-time object detection with region proposal networks. *arXiv*. <https://arxiv.org/abs/1506.01497>

[4] Redmon, J., Divvala, S., Girshick, R., & Farhadi, A. (2016). You only look once: Unified, real-time object detection. *Proceedings of the IEEE Conference on Computer Vision and Pattern Recognition*, 779–788. <https://doi.org/10.1109/CVPR.2016.91>

[5] Redmon, J., & Farhadi, A. (2018). YOLOv3: An incremental improvement. *arXiv*. <https://arxiv.org/abs/1804.02767>

[6] Bochkovskiy, A., Wang, C.-Y., & Liao, H.-Y. M. (2020). YOLOv4: Optimal speed and accuracy of object detection. *arXiv*. <https://arxiv.org/abs/2004.10934>

[7] Ultralytics. (n.d.). *Ultralytics documentation*. <https://docs.ultralytics.com/>

[8] Ultralytics. (n.d.). *YOLO11*. <https://docs.ultralytics.com/models/yolo11/>

### A.4 Evaluation

[9] Lin, T.-Y., Maire, M., Belongie, S., Hays, J., Perona, P., Ramanan, D., Dollár, P., & Zitnick, C. L. (2014). Microsoft COCO: Common objects in context. *arXiv*. <https://arxiv.org/abs/1405.0312>

### A.5 Web stack

[10] FastAPI. (n.d.). *FastAPI documentation*. <https://fastapi.tiangolo.com/>

[11] Starlette. (n.d.). *Starlette documentation*. <https://www.starlette.io/>

[12] Uvicorn. (n.d.). *Uvicorn documentation*. <https://www.uvicorn.org/>

[13] Mozilla Developer Network. (n.d.). *JavaScript modules*. <https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules>

### A.6 Quality and security

[14] International Organization for Standardization. (2023). *ISO/IEC 25010:2023 — Systems and software Quality Requirements and Evaluation (SQuaRE): Product quality model*. <https://www.iso.org/standard/78176.html>

[15] Mozilla Developer Network. (n.d.). *Content Security Policy (CSP)*. <https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP>

[16] Fielding, R., Nottingham, M., & Reschke, J. (Eds.). (2022). *RFC 9110: HTTP Semantics*. IETF. <https://www.rfc-editor.org/rfc/rfc9110.html>

### A.7 Runtime, devices and codecs

[17] PyTorch. (n.d.). *MPS backend*. <https://pytorch.org/docs/stable/notes/mps.html>

[18] OpenCV. (n.d.). *cv::VideoWriter class reference*. <https://docs.opencv.org/4.x/dd/d9e/classcv_1_1VideoWriter.html>

### A.8 Training

[19] Zhang, H., Cissé, M., Dauphin, Y. N., & Lopez-Paz, D. (2018). mixup: Beyond empirical risk minimization. *arXiv*. <https://arxiv.org/abs/1710.09412>

[20] Loshchilov, I., & Hutter, F. (2017). SGDR: Stochastic gradient descent with warm restarts. *arXiv*. <https://arxiv.org/abs/1608.03983>

### A.9 Licensing and provenance

[21] Ultralytics. (n.d.). *Licensing*. <https://www.ultralytics.com/license>

[22] Bisman-Singh-Dev. (n.d.). *YOLO pothole detection model*. <https://github.com/Bisman-Singh-Dev/YOLO-Pothole-Detection-Model>

### A.10 This project

[23] Aayush9808. (2026). *InfraScan* [Source code]. <https://github.com/Aayush9808/InfraScan>

[24] Aayush9808. (2026). *InfraScan documentation* [Documentation]. <https://github.com/Aayush9808/InfraScan-Documentation>

---

## Section B — Further reading

Not cited by the report; recommended for the viva and for anyone extending the project. Each entry states *why it is worth reading*.

### B.1 Road-damage datasets and benchmarks

- **RDD2020** — Arya, D., Maeda, H., Kashiyama, T., & Sekimoto, Y. (2020). *RD2020: Image dataset for road damage assessment*. IEEE International Conference on Robotics and Automation (ICRA). <https://arxiv.org/abs/2004.09380>
  *Why:* the standard reference for multi-class road damage and a useful contrast to this project's single-class scope.

- **RDD2022** — Arya, D., Maeda, H., Madawa, K., Zhang, D., Liu, L., & Seo, Y. (2022). RDD2022: A large-scale benchmark dataset for road damage detection. *IEEE Transactions on Intelligent Transportation Systems*, 23(11), 14381–14390. <https://arxiv.org/abs/2204.00981>
  *Why:* shows how dataset scale and class balance change what a detector can do — the data-quality argument in one paper.

- **PASCAL VOC** — Everingham, M., Van Gool, L., Williams, C. K. I., Winn, J., & Zisserman, A. (2010). The PASCAL Visual Object Classes (VOC) challenge. *International Journal of Computer Vision*, 88(2), 303–338. <https://doi.org/10.1007/s11263-009-0275-4>
  *Why:* the original detection evaluation protocol; the `ingest_voc_dataset.py` utility in this project targets its XML format.

- **KITTI** — Geiger, A., Lenz, P., Stiller, C., & Urtasun, R. (2013). The KITTI vision benchmark suite. *IEEE Transactions on Pattern Analysis and Machine Intelligence*, 35(7), 1724–1736. <https://doi.org/10.1109/TPAMI.2012.224
  *Why:* the standard reference for autonomous-driving perception benchmarks and their metric definitions.

### B.2 Detection foundations

- **Jocher, G., Chaurasia, A., & Qu, J. (2023). Ultralytics YOLO.** <https://arxiv.org/abs/2305.13696>
  *Why:* the paper behind the library this project calls.

- **Focal loss and RetinaNet** — Lin, T.-Y., Goyal, P., Girshick, R., He, K., & Dollár, P. (2017). Focal loss for dense object detection. *ICCV*. <https://arxiv.org/abs/1708.02002>
  *Why:* the standard answer to class-imbalance questions in detection.

- **Soft-NMS** — Bodla, N., Singh, P., Matsakis, S. A., & Davis, L. S. (2016). Soft-NMS: Non-maximum suppression technique for bounding boxes. *CVPR Workshops*. <https://arxiv.org/abs/1703.02743>
  *Why:* the direct follow-up to a question about why the project uses standard NMS.

### B.3 Metrics and evaluation practice

- **Powers, D. M. (2011). Evaluation: From precision, recall and F-measure to ROC, informedness, markedness and correlation. *Journal of Machine Learning Technologies*, 2(1), 37–63.** <https://doi.org/10.5555/2498981.2498982>
  *Why:* a careful treatment of metric definitions — the source of the project's insistence on reporting precision and recall together.

- **Recht, B., Torralba, A., & Recht, S. B. (2019). Do ImageNet classifiers generalize to ImageNet?**
  *Why:* the clearest demonstration that a clean test split is not the same as generalisation — exactly the caveat attached to the 718-image result.

### B.4 Engineering and machine-learning practice

- **Sculley, D., Holt, G., Golovin, D., Davydov, E., Phillips, T., Ebner, D., Chaudhary, V., Young, M., Crespo, J.-F., & Dennison, D. (2015). Hidden technical debt in machine learning systems. *NeurIPS 28*.** <https://papers.nips.cc/paper/5656-hidden-technical-debt-in-machine-learning-systems>
  *Why:* the paper that motivates the project's emphasis on configuration, data collection, serving and monitoring rather than model quality alone.

- **Breck, S., Cai, S., Sculley, D., Slattery, S., & Whiteside, A. (2017). The ML test score: A rubric for ML production readiness and technical debt reduction. *IEEE International Conference on Big Data.*** <https://arxiv.org/abs/1704.02341>
  *Why:* a practical readiness checklist; the project uses it to decide what is *not* yet production-ready.

- **Amershi, S., et al. (2019). Software engineering for machine learning: A case study. *ICSE-SEIP*.** <https://doi.org/10.1109/ICSE-SEIP.2019.00026>
  *Why:* why data and pipeline engineering dominate real ML projects.

- **Bengio, Y., & Grandvalet, J. (2004). The bias-variance tradeoff.**
  *Why:* the classical framing behind the model's size and epoch trade-offs.

### B.5 Python and platform foundations

- **Oliphant, N. (2015). The NumPy array: A structure for efficient numerical computation.** <https://doi.org/10.1109/MCS.2015.3483924>
- **Virtanen, P., et al. (2020). SciPy 1.0: Fundamental algorithms for scientific computing in Python. *Nature Methods*, 17, 261–272.** <https://doi.org/10.1038/s41592-019-0686-2>
- **McKinney, W. (2010). Data structures for statistical computing in Python. *Computing in Science & Engineering*, 12(5), 56–61.** <https://doi.org/10.1109/MCSE.2010.56>
- **Hunter, J. D. (2007). Matplotlib: A 2D graphics environment. *Computing in Science & Engineering*, 9(3), 90–95.** <https://doi.org/10.1109/MCSE.2007.55>
- **Bradski, G. (2000). The OpenCV library. *Dr. Dobb's Journal of Software Tools*.** <https://opencv.org>
- **Paszke, A., et al. (2019). PyTorch: An imperative style, high-performance deep learning library. *NeurIPS 32*.** <https://arxiv.org/abs/1912.01703>
- **Richard Stallman & the GNU Project. (n.d.). *The GNU General Public License, version 3.*** <https://www.gnu.org/licenses/gpl-3.0.html> — the licence under which Ultralytics publishes its open-source components.

### B.6 Accessibility and human factors

- **World Wide Web Consortium. (n.d.). *Web Content Accessibility Guidelines (WCAG) 2.2*.** <https://www.w3.org/TR/WCAG22/>
  *Why:* the target of the project's [source-level accessibility review](../11-security-and-quality/accessibility.md).

- **W3C. (n.d.). *ARIA Authoring Practices Guide — Tabs and Dialog patterns*.** <https://www.w3.org/WAI/ARIA/apg/patterns/tabs/>
  *Why:* the pattern the image-source tabs implement, and the source of the missing `aria-controls` finding.

---

## Section C — References the project deliberately does not make

Naming these matters as much as naming the others. Each of the following would be an attractive citation and each would be a mistake.

| Tempting citation | Why it is not used |
|---|---|
| A state-of-the-art road-damage benchmark score | The datasets, class definitions, splits and metrics differ, so the numbers are not comparable. The report says so explicitly rather than tabulating an invalid comparison. |
| A "95 % accuracy" figure for pothole detection | No such measurement was made in this project. The verified figure is recall 0.5996 on 718 test images. |
| SegNet, U-Net or Mask R-CNN results | No segmentation model was trained or evaluated here. |
| A CUDA or Jetson benchmark | The evaluation ran on Apple MPS; no NVIDIA hardware was used. |
| An ONNX/TensorRT speed-up table | The export path was never executed. |
| A pothole-size or depth estimation model | The severity band is a bounding-box coverage heuristic, not a physical measurement. |
| A Google Street View or Mapillary dataset | No external dataset was used for training or evaluation. |
| A citation for the dataset's provenance | The creators, URLs and licences are unresolved; inventing a citation would be the exact failure this chapter exists to prevent. |

---

## Citation style

The project report uses a numbered style with the reference list at the end. This bibliography uses an author–date style for clarity when the same source is discussed in several chapters. Both styles are acceptable; what matters is that the same source is never given two different identities across the documentation, and that in-text markers always resolve.

---

## Related pages

- [Citation guide](citation-guide.md) — ready-to-use citation formats.
- [Third-party attribution](third-party-attribution.md) — notices the project must carry.
- [Licences and assets](licences-and-assets.md) — the licence matrix and asset provenance.
- [Project report](../03-project-documentation/project-report.md) — the document whose references are listed in Section A.
- [Literature survey](../01-presentation/literature-survey.md) — the presentation-level reading list.
