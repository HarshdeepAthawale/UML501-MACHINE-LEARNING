# Changelog

All notable changes to Aletheia. Format based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/); versions follow [Semantic Versioning](https://semver.org/).

While we are `0.x`, the minor version tracks the phase: `0.1` = Phase 1 done, `0.2` = Phase 2, … `0.6` = product complete on mock detectors, `0.7`–`0.8` = real models, `1.0` = final evaluation + demo.

Every entry says whether it is **breaking** for the other side of the API (frontend ↔ backend) or for stored data.

## How to add an entry

- Add lines under **Unreleased** in the PR that makes the change.
- Sections: `Added`, `Changed`, `Deprecated`, `Removed`, `Fixed`, `Security`.
- Prefix breaking changes with **BREAKING:** and say what the other side must do.
- On release: rename **Unreleased** to `[x.y.z] — YYYY-MM-DD`, tag `vX.Y.Z`, start a new **Unreleased**.

## [Unreleased]

### Added
- Project documentation set under `docs/`: PRD, technical specification, app flow, design system, data schema, API contract, implementation plan, tracker, engineering rules, ADR log, setup, deployment, test plan, security, non-goals, open questions, contributing guide, glossary.
- Step-by-step phase guides under `docs/phases/` (Phase 0 – Phase 9).

### Changed
- Plan restructured from 9 to 10 phases: the ML phase is split into Phase 7 (data pipeline + visual models) and Phase 8 (temporal, audio, fusion, calibration); evaluation & demo becomes Phase 9. Release plan gains `0.8.0`.

## Release plan

| Version | Milestone | Phase |
|---|---|---|
| 0.1.0 | Backend core: upload, analyses, SSE with a stub pipeline | 1 |
| 0.2.0 | Four agents on mock detectors + Claude Judge | 2 |
| 0.3.0 | Frontend shell + upload flow | 3 |
| 0.4.0 | War room + results | 4 |
| 0.5.0 | Dashboard, history, leaderboard shell, about, settings | 5 |
| 0.6.0 | Hardened, deployed, product complete on mocks | 6 |
| 0.7.0 | Real visual detector (GenD / Effort) with Grad-CAM | 7 |
| 0.8.0 | Temporal + audio models, learned fusion, calibration — no mocks left | 8 |
| 1.0.0 | Final evaluation, report, demo | 9 |
