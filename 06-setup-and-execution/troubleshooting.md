# Troubleshooting

| Symptom | Check |
|---|---|
| Missing checkpoint | Place an authorized compatible file at `weights/best.pt` |
| Dashboard starts but inference is unavailable | Expected when the checkpoint is excluded |
| Dataset path error | Provide an authorized dataset matching `data.yaml` |
| Camera fails | Check permissions, device index, and local camera availability |
| Video writer fails | Check OpenCV codecs, source readability, and output permissions |
| Import error | Activate the intended virtual environment and install `requirements.txt` |
