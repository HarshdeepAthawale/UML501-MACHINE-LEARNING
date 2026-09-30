# Tracker

Single source of truth for task status. Task details live in [ImplementationPlan.md](ImplementationPlan.md); this file only tracks state.

**Status key:** `todo` · `doing` · `review` (PR open) · `done` · `blocked` (say why in Notes)
**Owner key:** H = Harshdeep, G = Girikshit, H+G = shared

Update the row in the same PR that finishes the task (put the PR number in Notes).

## Summary

| Phase | Tasks | Done | Status |
|---|---|---|---|
| P0 Foundations | 10 | 0 | not started |
| P1 Backend core | 11 | 0 | not started |
| P2 Agents & orchestration | 13 | 0 | not started |
| P3 Frontend foundation | 7 | 0 | not started |
| P4 War room & results | 11 | 0 | not started |
| P5 Dashboard & pages | 10 | 0 | not started |
| P6 Hardening & deploy | 7 | 0 | not started |
| P7 ML models | 10 | 0 | not started |
| P8 Evaluation & demo | 5 | 0 | not started |

## External dependencies

| Item | Requested on | Status | Notes |
|---|---|---|---|
| FaceForensics++ access form | – | todo | Request in P0-09; approval can take days |
| Celeb-DF v2 access form | – | todo | |
| DFDC (Kaggle) | – | todo | Download a subset only |
| Anthropic API key for the Judge | – | todo | See [OpenQuestions.md Q-05](OpenQuestions.md#q-05) |
| GPU access (Colab / Kaggle / lab) | – | todo | Needed from Week 9 |

## Phase 0 — Foundations

| ID | Task | Owner | Status | Notes |
|---|---|---|---|---|
| P0-01 | Monorepo structure | H | todo | |
| P0-02 | API skeleton + tooling + `/health` | H | todo | |
| P0-03 | Web skeleton + tooling | G | todo | |
| P0-04 | CI workflow | H | todo | |
| P0-05 | Branch protection, PR template, labels | H | todo | |
| P0-06 | `.env.example` + Setup verified on both laptops | H+G | todo | |
| P0-07 | Freeze API contract v1 | H+G | todo | |
| P0-08 | Design tokens + fonts + theme toggle | G | todo | |
| P0-09 | Submit dataset access requests | H+G | todo | |
| P0-10 | Demo set + manifest | G | todo | |

## Phase 1 — Backend core

| ID | Task | Owner | Status | Notes |
|---|---|---|---|---|
| P1-01 | Settings, logging, request ID, errors, CORS | H | todo | |
| P1-02 | DB models + initial migration | H | todo | |
| P1-03 | IDs + local storage | H | todo | |
| P1-04 | `POST /media` | H | todo | |
| P1-05 | Media read/delete + Range | H | todo | |
| P1-06 | Job runner | H | todo | |
| P1-07 | Event bus + SSE | H | todo | |
| P1-08 | Analyses endpoints | H | todo | |
| P1-09 | Stub pipeline emitting full event sequence | H | todo | Unblocks P4 |
| P1-10 | Rate limits, readiness, retention | H | todo | |
| P1-11 | Backend tests | H | todo | |

## Phase 2 — Agents & orchestration

| ID | Task | Owner | Status | Notes |
|---|---|---|---|---|
| P2-01 | Preprocessing | H | todo | |
| P2-02 | Detector protocol + mocks | H | todo | |
| P2-03 | Agent base + persistence | H | todo | |
| P2-04 | Visual agent | H | todo | |
| P2-05 | Temporal & Frequency agent | H | todo | |
| P2-06 | Audio agent | H | todo | |
| P2-07 | LangGraph graph + routing | H | todo | |
| P2-08 | Fusion + tests | H | todo | |
| P2-09 | Judge agent (Claude) | H | todo | |
| P2-10 | Rule-based Judge | H | todo | |
| P2-11 | Agents / frames / artifacts endpoints | H | todo | |
| P2-12 | PDF report | H | todo | |
| P2-13 | Agent contract + graph tests | H | todo | |

## Phase 3 — Frontend foundation

| ID | Task | Owner | Status | Notes |
|---|---|---|---|---|
| P3-01 | App shell + route skeletons | G | todo | |
| P3-02 | Background system + base components | G | todo | |
| P3-03 | Typed API client + Query | G | todo | |
| P3-04 | `/analyze` upload flow | G | todo | |
| P3-05 | Forensic components v1 | G | todo | |
| P3-06 | Landing v1 | G | todo | |
| P3-07 | Frontend tests | G | todo | |

## Phase 4 — War room & results

| ID | Task | Owner | Status | Notes |
|---|---|---|---|---|
| P4-01 | SSE hook + store | G | todo | |
| P4-02 | War-room layout | G | todo | |
| P4-03 | Streaming Judge panel | G | todo | |
| P4-04 | Verdict reveal | G | todo | |
| P4-05 | Summary tab | G | todo | |
| P4-06 | Visual tab | G | todo | |
| P4-07 | Temporal tab | G | todo | |
| P4-08 | Audio tab | G | todo | |
| P4-09 | Metadata + Raw tabs | G | todo | |
| P4-10 | Actions + error states | G | todo | |
| P4-11 | Responsive + reduced motion | G | todo | |

## Phase 5 — Dashboard & secondary pages

| ID | Task | Owner | Status | Notes |
|---|---|---|---|---|
| P5-01 | Metrics API + dashboard | H+G | todo | |
| P5-02 | History | G | todo | |
| P5-03 | Model registry seed + API | H | todo | |
| P5-04 | Leaderboard UI | G | todo | |
| P5-05 | 3D explorer (synthetic) | G | todo | |
| P5-06 | About page | H+G | todo | |
| P5-07 | Settings | H+G | todo | |
| P5-08 | Print report + PDF button | G | todo | |
| P5-09 | Landing 3D hero + stats | G | todo | |
| P5-10 | Command palette | G | todo | |

## Phase 6 — Hardening & deploy

| ID | Task | Owner | Status | Notes |
|---|---|---|---|---|
| P6-01 | Playwright E2E | H+G | todo | |
| P6-02 | Security checklist | H+G | todo | |
| P6-03 | Performance pass | H+G | todo | |
| P6-04 | Docker + compose | H | todo | |
| P6-05 | Demo deployment | H+G | todo | |
| P6-06 | Pre-computed demo analyses | H+G | todo | |
| P6-07 | Tag v0.6.0 | H+G | todo | |

## Phase 7 — ML models

| ID | Task | Owner | Status | Notes |
|---|---|---|---|---|
| P7-01 | Data pipeline + splits | H+G | todo | |
| P7-02 | Baselines | H+G | todo | |
| P7-03 | GenD fine-tune | H+G | todo | |
| P7-04 | Effort | H+G | todo | |
| P7-05 | Grad-CAM | H+G | todo | |
| P7-06 | Temporal classical classifier | H+G | todo | |
| P7-07 | Audio model | H+G | todo | |
| P7-08 | Stacking + calibration | H+G | todo | |
| P7-09 | Exports → registry/embeddings | H+G | todo | |
| P7-10 | Real detectors wired in | H+G | todo | |

## Phase 8 — Evaluation & demo

| ID | Task | Owner | Status | Notes |
|---|---|---|---|---|
| P8-01 | Robustness panel | H+G | todo | P2 |
| P8-02 | Live webcam mode | H+G | todo | P2 |
| P8-03 | Final report | H+G | todo | |
| P8-04 | Slides + demo script | H+G | todo | |
| P8-05 | Rehearsals + v1.0.0 | H+G | todo | |
