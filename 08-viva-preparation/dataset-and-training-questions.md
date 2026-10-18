# Dataset and training questions

1. Why separate train/validation/test? Different roles and unbiased assessment.
2. What does one class mean? The configured class is `pothole`.
3. Why normalize labels? Resolution-independent YOLO representation.
4. What is leakage? Evaluation information influencing training.
5. Why verify dataset license? Code license does not grant dataset rights.
6. Why is exact model variant unresolved? The checkpoint is excluded.
7. What should a reproducible training record include? Versions, data hash, config, command, seed, and checkpoint hash.
8. Why are inherited metrics not enough? They were not freshly reproduced here.
