# Aletheia — Multi-Agent Deepfake Forensics

UML501 Machine Learning project by **Harshdeep Athawale** and **Girikshit Gupta**.

Upload an image, video, or audio clip. Four AI agents examine it in parallel — **Visual Forensics**, **Temporal & Frequency**, **Audio**, and a **Judge** that weighs their evidence — and a live dashboard shows their work, ending in a calibrated verdict, an evidence timeline, heatmaps, and a downloadable forensic report.

## Status

📄 **Planning complete — Phase 0 (foundations) is next.** No application code yet.

| Phase | What | Status |
|---|---|---|
| [0](docs/phases/Phase-0-Foundations.md) | Repo, tooling, CI | next |
| [1](docs/phases/Phase-1-Backend-Core.md) | Backend core: upload, analyses, job runner, live events | planned |
| [2](docs/phases/Phase-2-Agents-Orchestration.md) | 4 agents (mock detectors, real signal processing, Claude Judge) | planned |
| [3](docs/phases/Phase-3-Frontend-Foundation.md) | Frontend foundation: design system, upload flow | planned |
| [4](docs/phases/Phase-4-War-Room-Results.md) | Live war room + results | planned |
| [5](docs/phases/Phase-5-Dashboard-Pages.md) | Dashboard, history, leaderboard, explorer, about | planned |
| [6](docs/phases/Phase-6-Hardening-Deploy.md) | Hardening + deployment | planned |
| [7](docs/phases/Phase-7-ML-Data-Visual-Models.md) | ML I: data pipeline, baselines, GenD / Effort visual models | planned |
| [8](docs/phases/Phase-8-ML-Temporal-Audio-Fusion.md) | ML II: temporal + audio models, learned fusion, calibration | planned |
| [9](docs/phases/Phase-9-Evaluation-Demo.md) | Evaluation, report, demo | planned |

## Documentation

Start at **[docs/README.md](docs/README.md)**. Key docs:

- [PRD](docs/PRD.md) — what and why
- [Technical specification](docs/TechSpecifications.md) — architecture, agents, fusion math
- [API reference](docs/API.md) — endpoints, SSE events, errors
- [Implementation plan](docs/ImplementationPlan.md), [phase guides](docs/phases/) and [Tracker](docs/Tracker.md)
- [Engineering rules](docs/Rules.md) — read before your first commit

## Stack

**Backend:** Python 3.12 · FastAPI · SQLAlchemy + Alembic · LangGraph · Claude API · OpenCV · FFmpeg
**Frontend:** Next.js · TypeScript · Tailwind CSS · shadcn/ui · Motion · react-three-fiber
**ML (Phases 7–8):** PyTorch · CLIP ViT-L/14 · DeepfakeBench

## Getting started

See [docs/Setup.md](docs/Setup.md).
