# Annotation and preprocessing

The configuration identifies one class, `pothole`, and uses YOLO-style split paths:

```yaml
path: ./dataset
train: images/train
val: images/val
test: images/test
nc: 1
names:
- pothole
```

YOLO label rows use normalized coordinates:

```text
class_id x_center y_center width height
```

The included preparation scripts support dataset conversion workflows, but they require an explicitly supplied source path and an authorized dataset.
