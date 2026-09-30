# Aletheia — Multi-Agent Deepfake Forensics

UML501 Machine Learning project by **Harshdeep Athawale** and **Girikshit Gupta**.

Upload an image, video, or audio clip. Four AI agents examine it in parallel — **Visual Forensics**, **Temporal & Frequency**, **Audio**, and a **Judge** that weighs their evidence — and a live dashboard shows their work, ending in a calibrated verdict, an evidence timeline, heatmaps, and a downloadable forensic report.

## Status

📄 **Planning complete — Phase 0 (foundations) is next.** No application code yet.

| Phase | What | Status |
|---|---|---|
| 0 | Repo, tooling, CI | next |
| 1–2 | Backend API + 4 agents (mock detectors, real signal processing, Claude Judge) | planned |
| 3–5 | Dashboard: upload, live war room, results, history, leaderboard | planned |
| 6 | Hardening + deployment | planned |
| 7 | Real ML models (GenD / Effort / EfficientNet-B4, XLS-R + AASIST) | planned |
| 8 | Evaluation, report, demo | planned |

## Documentation

Start at **[docs/README.md](docs/README.md)**. Key docs:

- [PRD](docs/PRD.md) — what and why
- [Technical specification](docs/TechSpecifications.md) — architecture, agents, fusion math
- [API reference](docs/API.md) — endpoints, SSE events, errors
- [Implementation plan](docs/ImplementationPlan.md) and [Tracker](docs/Tracker.md)
- [Engineering rules](docs/Rules.md) — read before your first commit

## Stack

**Backend:** Python 3.12 · FastAPI · SQLAlchemy + Alembic · LangGraph · Claude API · OpenCV · FFmpeg
**Frontend:** Next.js · TypeScript · Tailwind CSS · shadcn/ui · Motion · react-three-fiber
**ML (Phase 7):** PyTorch · CLIP ViT-L/14 · DeepfakeBench

## Getting started

See [docs/Setup.md](docs/Setup.md).
