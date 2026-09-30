# Non-Goals

Things we are deliberately **not** building. If a request matches something here, the answer is "no" unless we first move it out of this list with a PR (and an ADR if it's big). This keeps a two-person, one-semester project finishable.

## Product

| Non-goal | Why |
|---|---|
| **Generating deepfakes** (any face swap, voice clone, or "make a fake" feature) | Ethics and scope. We detect; we don't create. Test fakes come from public datasets. |
| Identifying *who* is in a video (face recognition, identity lookup) | Privacy; not needed to judge authenticity |
| Scraping or monitoring social media / URLs | Legal and scope issues; users upload files themselves |
| Paste-a-URL analysis (YouTube, X, Instagram links) | Terms-of-service and download complexity; P2 at most, after v1.0 |
| Multi-tenant SaaS: teams, billing, quotas, admin consoles | Academic project; single shared deployment |
| Native mobile apps or a browser extension | The responsive web app covers viewing on phones |
| Legal or court-grade forensic certification, chain-of-custody | Our output is probabilistic and explicitly not proof |
| Real-time analysis of live streams at scale | Webcam mode (P2) is a single-user demo only |
| Localization / multiple languages | English only for v1 |
| Detecting general image edits (copy-move, splicing of objects, Photoshop) | Focus is faces and voices (deepfakes), not all image forensics |
| Detecting AI-generated text | Out of domain |

## Engineering

| Non-goal | Why |
|---|---|
| Microservices, Kubernetes, message brokers | One API process + one web app is enough ([ADR-006](Decisions.md#adr-006)) |
| Horizontal scaling / high availability | Demo traffic only |
| Training or fine-tuning an LLM | The Judge uses an existing model via API |
| Letting the LLM decide the verdict | Deterministic fusion decides ([ADR-007](Decisions.md#adr-007)) |
| Giving the LLM tools (web search, code execution, DB access) | Unnecessary, and increases prompt-injection risk |
| Supporting Internet Explorer / legacy browsers | Evergreen browsers only |
| Offline-first / PWA | Not needed for the demo |
| 100% test coverage | Targets in [TestPlan.md](TestPlan.md#2-coverage-targets) are enough |

## ML

| Non-goal | Why |
|---|---|
| Building a new architecture from scratch | We fine-tune strong pretrained models and compare against baselines — that's where the accuracy is |
| Training on data scraped from the web or on users' uploads | Consent and licensing; uploads are never used for training |
| Claiming a single "accuracy" that mixes in-domain and cross-dataset results | Misleading (R-9) |
| Chasing 99%+ on easy datasets (e.g. only "140k real vs fake faces") | Looks good, proves little; evaluators know it |
| Adversarial-attack defence research | Interesting, out of scope; robustness panel only measures drift |
| Pixel-level manipulation masks (segmentation) | Heatmaps/Grad-CAM show *where the model looked*, which is enough |

## Revisit after v1.0 (not forbidden, just not now)

- URL ingestion for public videos (with ToS review)
- Batch analysis of a folder
- Local open-source VLM Judge for fully offline use
- C2PA manifest signing / verification beyond "present or not"
- Browser extension
