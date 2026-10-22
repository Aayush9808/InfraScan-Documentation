# Security and quality

This chapter assesses InfraScan as what it is: a **local, single-user, best-effort computer-vision application**, not a certified safety instrument or a production service. It separates controls that exist in the source from risks that remain, and it refuses to turn unmeasured behaviour into a performance or accessibility claim.

- [Scope and method](#scope-and-method)
- [What is being protected](#what-is-being-protected)
- [Findings at a glance](#findings-at-a-glance)
- [How to read severity](#how-to-read-severity)
- [Chapter contents](#chapter-contents)
- [What this chapter does not claim](#what-this-chapter-does-not-claim)
- [Related chapters](#related-chapters)

## Scope and method

The assessment covers the accepted implementation:

| Area | Sources inspected |
|---|---|
| HTTP surface and middleware | `infrascan/api.py` |
| Model lifecycle and annotation | `infrascan/inference.py` |
| Video jobs and worker concurrency | `infrascan/jobs.py` |
| Paths, limits, extensions and TTLs | `infrascan/config.py` |
| Severity policy | `infrascan/severity.py` |
| Frame loop, codec fallback and telemetry | `detect.py` |
| SPA markup, modules and styles | `web/index.html`, `web/assets/js/*.js`, `web/assets/css/app.css` |
| Verified functional evidence | [`PROJECT_FACTS.md`](../PROJECT_FACTS.md) |

Methods used:

- **Source review** of routes, validation, file handling, DOM writes and CSS tokens.
- **Contrast calculation** using the WCAG relative-luminance formula on declared colours, with translucent surfaces approximated by alpha compositing over the base background.
- **Evidence review** of the recorded functional checks (image inference, video job/playback, synthetic webcam, responsive widths).
- **No** penetration test, fuzzing campaign, malware scan, dependency audit, screen-reader session, automated accessibility scan, 200% zoom pass, load test, CUDA test or production deployment was performed.

That means the findings are grounded but not exhaustive. A clean source review does not prove the absence of vulnerabilities, and a passing recorded check does not prove behaviour on another machine.

## What is being protected

| Asset | Why it matters | Where it lives |
|---|---|---|
| Uploaded road images/video | May contain location, people, vehicles or other sensitive context | Browser memory; video also copied to `runs/dashboard/jobs` |
| Annotated results and telemetry | Derived sensitive media and records | `runs/dashboard/results`, `runs/dashboard/jobs` |
| Checkpoint | Valuable model artefact; may carry licence restrictions | `weights/best.pt` or `INFRASCAN_WEIGHTS` |
| Dataset samples | Licensed/private imagery | `dataset/images/test` |
| Local process | Runs with the operator's OS privileges | Uvicorn/Python process |
| Browser session | Holds previews, results and thresholds | Tab memory and `localStorage` |
| Availability | A demo or inspection depends on the local service | Single process, in-memory state |

The primary security objective is **not** to resist a determined internet attacker. It is to keep the application's data and compute local, avoid obvious cross-site and path tricks, and make the trust boundary explicit so an operator does not accidentally expose it.

## Findings at a glance

| ID | Finding | Risk | Status |
|---|---|---|---|
| SEC-01 | No authentication or authorization on any route | High if exposed; Medium on a shared local machine | Accepted / by design |
| SEC-02 | No TLS, HSTS, CSRF token, Origin/Host validation or rate limiting | High if exposed | Open |
| SEC-03 | Checkpoints are loaded through PyTorch's pickle-based loader | High impact, requires operator-supplied file | Accepted / trust boundary |
| SEC-04 | Uploaded media reaches OpenCV/FFmpeg decoders without sandboxing | Medium | Accepted / residual |
| SEC-05 | Image extension is not validated server-side; decode is the only gate | Low–Medium | Open / documented |
| SEC-06 | Video extension is validated but the container is not sniffed | Low–Medium | Open / documented |
| SEC-07 | 2 GB uploads, in-memory jobs and no quota permit local disk/memory pressure | Medium | Open |
| SEC-08 | `system.js` injects `row.description` via `innerHTML` | Low today; latent XSS if server data becomes untrusted | Open |
| SEC-09 | CSP blocks FastAPI's default Swagger UI | Low | Known incompatibility |
| SEC-10 | Webcam close-during-permission race can leave a stream active | Low privacy | Open |
| SEC-11 | No request timeout/abort in `api.js`; hung requests can leave the UI busy | Low | Open |
| SEC-12 | Results/samples are served with a one-hour public cache header | Low on a shared machine | Accepted / documented |
| SEC-13 | Artefact retention is startup-only, not a live 24 h expiry | Low privacy/disk | Open / documented |
| SEC-14 | `--muted-dim` text fails 4.5:1; tiny helper text is widespread | Medium accessibility | Open |
| SEC-15 | Palette input lacks a label/focus indicator; tabs lack complete tab semantics | Medium accessibility | Open |
| SEC-16 | Mobile status chip hides its text, leaving a colour-only dot | Medium accessibility | Open |
| SEC-17 | Drawer and route changes lack focus management | Medium accessibility | Open |
| SEC-18 | No formal accessibility or performance certification; many metrics unmeasured | Informational | Evidence boundary |

The detailed analysis, exploit preconditions and mitigations are in the three chapter pages.

## How to read severity

| Label | Meaning in this chapter |
|---|---|
| **High** | Can lead to code execution, broad data exposure or full loss of control if the stated precondition holds |
| **Medium** | Can expose local data, degrade availability or mislead a user under realistic local conditions |
| **Low** | Limited impact, difficult precondition or defence-in-depth concern |
| **Info** | Evidence gap, documentation limitation or non-vulnerability |

Risk depends on deployment. The same missing authentication is a **Medium** concern on a single-user laptop bound to loopback and a **High** concern on `0.0.0.0` or a shared lab machine. The findings state that distinction rather than assigning a context-free score.

## Chapter contents

| Page | What it covers |
|---|---|
| [Security and privacy](security-and-privacy.md) | Threat model, implemented controls, residual risks, privacy properties, deployment guidance, prioritized recommendations |
| [Performance](performance.md) | Verified evaluation and dashboard timings, what they do and do not measure, concurrency, memory, disk and scalability boundaries |
| [Accessibility](accessibility.md) | Source-level WCAG review, contrast calculations, keyboard/screen-reader gaps, prioritized fixes, conformance boundary |

## What this chapter does not claim

- **No security certification.** There is no penetration test, threat-model sign-off, SAST/DAST report or dependency audit.
- **No production readiness.** The app has no accounts, TLS, rate limiting, isolation or monitoring suitable for a public service.
- **No privacy guarantee.** Files are plain on disk; metadata handling is not fully audited; deletion is not secure erasure.
- **No performance benchmark beyond the recorded values.** In particular, no CUDA, FPS, memory, cold-start, concurrency or large-video benchmark exists.
- **No accessibility certification.** There is no WCAG 2.2 AA/AAA claim, screen-reader pass or automated scan. At least one contrast failure is demonstrable from the CSS.
- **No safety claim.** Severity is a bounding-box heuristic, not an engineering assessment, and the system must not drive maintenance decisions autonomously.

When evidence is missing, the text says `[TO BE CONFIRMED]` or **BLOCKED** and explains what would close the gap.

## Related chapters

- [System architecture](../04-architecture/system-architecture.md) — trust boundaries and component responsibilities.
- [Backend and API](../04-architecture/backend-and-api.md) — routes, headers and validation.
- [Frontend architecture](../04-architecture/frontend-architecture.md) — DOM safety, CSP and accessibility baseline.
- [Data flow](../04-architecture/data-flow.md) — retention, metadata and privacy properties.
- [Deployment and operations](../04-architecture/deployment-and-operations.md) — exposure, logs, backup and non-goals.
- [Verified results](../09-testing-and-results/verified-results.md) — what was actually executed.
- [Limitations](../03-project-documentation/limitations.md) — project-level scope limits.
