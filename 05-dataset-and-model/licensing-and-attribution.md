# Licensing and attribution

> **This is an audit record, not legal advice.** The repository mixes an MIT-labelled application, an AGPL-3.0-by-default detector runtime, an unverified dataset, an excluded checkpoint and SIL-OFL fonts. Those terms do not collapse into one licence. Before distributing or deploying anything, obtain a licence review appropriate to your institution and jurisdiction.

## 1. At a glance

| Component | Where it lives | Stated or recorded terms | Redistribution status |
|---|---|---|---|
| InfraScan repository code | This repository and the main repo | **MIT** (`LICENSE`, copyright 2026 Bisman Singh) | Source is public under MIT; third-party components are not relicensed |
| Upstream project | `Bisman-Singh-Dev/YOLO-Pothole-Detection-Model` | Inherited MIT notice in `LICENSE` | Attribution preserved; upstream terms must be checked |
| Ultralytics runtime | Python dependency | **AGPL-3.0** by default; Enterprise option | Separate terms; not covered by the MIT file |
| YOLO11n checkpoint | `weights/best.pt` | Metadata licence field **AGPL-3.0**; provenance unresolved | **Excluded from Git; do not redistribute without permission** |
| Dataset | `dataset/` | **No verified licence** | **Excluded from Git; do not redistribute** |
| Inter font | `web/assets/fonts/` | **SIL OFL 1.1** | Bundled with licence text |
| Space Grotesk font | `web/assets/fonts/` | **SIL OFL 1.1** | Bundled with licence text |
| Python dependencies | `requirements.txt` | Mixed permissive/copyleft licences | Review installed versions before redistribution |
| GLBITM templates | Outside the repository | Institutional usage rules | Not redistributed here |

## 2. The repository's MIT licence

`LICENSE` contains the standard MIT text with:

```text
Copyright (c) 2026 Bisman Singh
```

MIT is permissive: it allows use, copying, modification, merging, publication, distribution, sublicensing and sale, provided the copyright notice and permission notice are included in copies or substantial portions. The software is provided "as is", without warranty.

What the MIT file does **not** do:

- It does not relicense Ultralytics code or the YOLO11 runtime.
- It does not relicense the dataset.
- It does not establish permission to redistribute the checkpoint.
- It does not override the font licences.
- It does not grant trademark rights or an endorsement.

The current team's modifications sit in a repository that preserves the upstream MIT notice. That preservation is the safe, verifiable position. Do not replace the copyright line or present the whole repository as entirely original work.

## 3. Ultralytics and AGPL-3.0

Ultralytics YOLO is distributed under **AGPL-3.0** by default, with a commercial **Enterprise** option. The checkpoint metadata also records `AGPL-3.0` as its licence field.

AGPL-3.0 is a strong copyleft licence. In broad terms:

- If you modify the AGPL-covered code and distribute it, you must provide the corresponding source under AGPL terms and retain notices.
- AGPL also addresses **network use**: if users interact with a modified version over a network, they must be offered the corresponding source.
- You cannot relicense AGPL-covered code as MIT merely because it sits in an MIT-labelled repository.
- A separate Enterprise licence may be required for closed-source or commercial deployment.

The exact obligations depend on how the code is combined, modified and distributed. The project's local-first dashboard is one thing; a public hosted service or a closed-source product is another. Do not describe the repository as "MIT-licensed" without the qualifier that Ultralytics and the dataset have separate terms.

References:

- <https://www.ultralytics.com/license>
- <https://github.com/ultralytics/ultralytics/blob/main/LICENSE>
- <https://docs.ultralytics.com/models/yolo11/>

## 4. Dataset rights are unresolved

The dataset is the largest unresolved governance item.

What is known:

- It is a local, single-class pothole dataset with 4,719 images and 7,340 boxes.
- It contains multiple filename families and at least two preparation pipelines.
- It is excluded from Git by `.gitignore`.
- No dataset licence file, creator manifest, source URL list or permission record is present in the verified tree.

What is **not** known:

- The original dataset names and creators.
- The licence of each source.
- Whether derived labels and resized JPEGs may be redistributed.
- Whether the images contain personal data (faces, number plates, identifiable locations).
- Whether the upstream project had permission to collect or share the images.

Therefore:

- Do **not** claim the dataset is CC BY, CC0, MIT, public domain or "freely available".
- Do **not** upload `dataset/` to GitHub, cloud storage, a public demo or a paper appendix.
- Do **not** include dataset images in a public poster or slide deck without checking the source rights and privacy.
- For an examiner demo, use a private, permissioned copy and explain the provenance gap.

See [dataset provenance](dataset-provenance.md) for the verified counts and missing fields.

## 5. The checkpoint is excluded and its rights are unresolved

The checkpoint `weights/best.pt`:

- is **not** committed to Git (`weights/` is ignored);
- has SHA-256 `f3490f953b4ac2aa0205c206c151be8f2da281da99708713b182492a43c7751c`;
- records `AGPL-3.0` in its metadata;
- was trained on a dataset whose rights are unresolved.

Do not:

- publish it as a release asset;
- upload it to a model hub;
- bundle it in a Docker image that you distribute;
- present it as a clean, licensed model;
- assume that the MIT repository licence covers the weights.

The safest position is: **the checkpoint is a local research artifact, shared only with explicit permission from the rights holders.** If the team decides to share it with examiners, record the permission and the scope (private evaluation only, no redistribution).

## 6. Font licences

The dashboard bundles:

```text
web/assets/fonts/
├── inter-latin.woff2
├── inter-latin-ext.woff2
├── OFL-Inter.txt
├── space-grotesk-latin.woff2
├── space-grotesk-latin-ext.woff2
└── OFL-SpaceGrotesk.txt
```

Both fonts are under the **SIL Open Font License 1.1**:

- Inter: `Copyright 2020 The Inter Project Authors (https://github.com/rsms/inter)`.
- Space Grotesk: `Copyright 2020 The Space Grotesk Project Authors (https://github.com/floriankarsten/space-grotesk)`.

The OFL allows the fonts to be used, studied, modified and redistributed, including bundling/embedding with software, provided that:

- the licence notice is retained;
- the fonts are not sold by themselves;
- modified versions do not use reserved font names;
- the fonts and derivatives remain under the OFL.

The bundled licence files satisfy the notice requirement. Do not delete them. The OFL does not restrict documents created with the fonts.

Reference: <https://openfontlicense.org>

## 7. Python dependency licences

`requirements.txt` declares the runtime dependencies. The table below records the **declared upstream licence** for the versions in use; it is not a full transitive audit. Verify the installed package metadata before distributing a bundled environment.

| Package | Declared upstream licence | Note |
|---|---|---|
| `ultralytics` | AGPL-3.0 | Strong copyleft; Enterprise option |
| `torch` | BSD-3-Clause | PyTorch |
| `torchvision` | BSD-3-Clause | PyTorch vision |
| `opencv-python` | Apache-2.0 | OpenCV packaging |
| `pillow` | HPND | Historical Permission Notice and Disclaimer |
| `numpy` | BSD-3-Clause | Numerical core |
| `pandas` | BSD-3-Clause | Telemetry CSV/JSON |
| `matplotlib` | PSF-2.0 | Training/evaluation plots |
| `seaborn` | BSD-3-Clause | Plot styling |
| `pyyaml` | MIT | `data.yaml` |
| `tqdm` | MPL-2.0 AND MIT | Progress bars |
| `fastapi` | MIT | HTTP API |
| `uvicorn` | BSD-3-Clause | ASGI server |
| `python-multipart` | Apache-2.0 | Multipart uploads |
| `starlette` | BSD-3-Clause | Transitive FastAPI dependency |
| `pydantic` | MIT | Transitive FastAPI dependency |

Transitive dependencies not listed here may carry additional notices. A redistribution bundle should include the licence texts shipped with the installed packages.

## 8. Citation and attribution guidance

| Situation | Recommended action |
|---|---|
| Academic report or viva | Cite the Ultralytics YOLO11 software and documentation; cite the upstream Bisman Singh project; describe the dataset as local/unverified; include the checkpoint SHA-256. |
| GitHub source distribution | Keep `LICENSE`, the upstream copyright line and the font OFL files. Add a third-party notices file if you add dependencies. |
| Public demo or hosted service using Ultralytics | Review AGPL-3.0 network obligations; offer corresponding source; consider Enterprise licensing. |
| Sharing the checkpoint | Do not, until provenance and permission are resolved. If privately shared, state "no redistribution". |
| Sharing the dataset | Do not, until source licences and privacy are resolved. |
| Publishing screenshots | Check for faces, number plates, addresses or other personal data. The supplied UI screenshots show the application, not road imagery. |
| Publishing the PR curve | The curve is an evaluation artifact; cite the checkpoint hash and the split counts. Do not imply it is a public benchmark. |
| Using the fonts in a poster | OFL allows document use; keep the font licence files in the repository. |

### Suggested attribution sentence

> "InfraScan uses an Ultralytics YOLO11n detector. Ultralytics YOLO is licensed under AGPL-3.0 by default; this project's MIT licence does not replace those terms. The checkpoint and dataset are local research artifacts whose provenance and redistribution rights are unresolved."

### Suggested citation for the model

Ultralytics' YOLO11 documentation requests the following form:

```text
Jocher, G., & Qiu, J. (2024). Ultralytics YOLO11 (Version 11.0.0)
[Computer software]. https://github.com/ultralytics/ultralytics
```

See the [bibliography](../10-references-and-governance/bibliography.md) for formatted references.

## 9. Unresolved items

- [ ] Dataset creator(s), source URL(s) and licence(s).
- [ ] Permission for derived labels and processed images.
- [ ] Permission to redistribute the checkpoint.
- [ ] Confirmation of the checkpoint's training-data provenance beyond the embedded Windows path.
- [ ] Decision on whether the checkpoint/dataset will be shared privately with examiners.
- [ ] Review of AGPL obligations for any public deployment.
- [ ] Review of transitive dependency notices for any bundled distribution.
- [ ] Privacy review of any road imagery used in slides, posters or reports.

Until these are closed, use the language in §8 and keep the artifacts local.
