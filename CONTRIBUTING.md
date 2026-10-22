# Contributing to the documentation

The goal is documentation that is **specific, source-grounded and defensible**. A smaller accurate page is better than a larger page that sounds impressive but cannot be verified.

## Source of truth

Before editing, read [`PROJECT_FACTS.md`](PROJECT_FACTS.md). It records the verified architecture, checkpoint metadata, dataset counts, benchmark results and evidence boundary. [`CONTRIBUTIONS.md`](CONTRIBUTIONS.md) records what the Git history can and cannot prove about individual contributions.

If a claim is not supported by `PROJECT_FACTS.md`, the source code, a recorded test, a cited external source or a supplied academic template, mark it `[TO BE CONFIRMED]`.

## Rules

1. **Do not invent** metrics, hardware, training parameters, dataset permissions, supervisor details, screenshots, references or individual contributions.
2. **Use the current architecture**: FastAPI + zero-build vanilla ES-module SPA + YOLO11n. Do not reintroduce Streamlit descriptions.
3. **Separate evidence labels**: `PASS`, `FAIL`, `BLOCKED`, `NOT RUN`, `[TO BE CONFIRMED]`.
4. **Keep upstream attribution** intact: Bisman Singh, the MIT licence and Ultralytics' separate licensing.
5. **Do not add** model weights, datasets, uploaded media, credentials, secrets or template binaries.
6. **Use relative links** for repository files and stable URLs/DOIs for external references.
7. **Check Mermaid syntax** and renderability; avoid exotic features that GitHub does not support.
8. **Keep commands executable** from the repository root unless a different working directory is stated.
9. **Do not commit generated outputs** (`runs/`, result images, temporary logs, virtual environments).
10. **Respect file ownership** when multiple writers are working; do not overwrite another section without coordination.

## Markdown style

- One `#` title per page.
- Use sentence case for headings unless a proper noun requires otherwise.
- Prefer tables for comparisons, code blocks for commands and Mermaid for relationships.
- Put a short navigation footer at the bottom of long pages.
- Use descriptive link text, not “click here”.
- Wrap lines only when it improves readability; there is no hard 80-column rule.
- Avoid emoji-heavy headings. A small number of icons is acceptable in the homepage only.

## Evidence checklist

Before committing a documentation change:

- [ ] Commands and paths match the current source tree.
- [ ] FastAPI routes and status codes match `infrascan/api.py`.
- [ ] The checkpoint and dataset numbers match `PROJECT_FACTS.md`.
- [ ] No unsupported “we achieved” or “tested” claims were added.
- [ ] Internal links resolve.
- [ ] Mermaid blocks render.
- [ ] No absolute local paths, secrets or personal data were added.
- [ ] Template-specific guidance refers to the GLBITM files without committing the binaries.

## Commit style

Use small, meaningful commits, for example:

```text
docs: rewrite architecture for FastAPI and SPA
docs: add 718-image evaluation evidence
docs: expand Khushi frontend guide
fix: correct video job retention wording
```

Do not create empty or artificial commits. Git history is a record of work, not a scoreboard.
