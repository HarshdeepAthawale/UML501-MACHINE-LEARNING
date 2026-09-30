# Phase 6 — Hardening & Deploy

| | |
|---|---|
| **Weeks** | Week 8 (2026-11-23 → 11-27) |
| **Owners** | H+G |
| **Depends on** | Phases 1–5 |
| **Unblocks** | Phase 7 (ML work starts on a stable, deployed product) |
| **Release** | `v0.6.0` — "product complete on mock detectors" |

## Goal

Turn a working product into a **reliable** one: automated end-to-end tests, the security checklist done, performance budgets met, containerised, deployed to a demo URL, and a local offline fallback ready. After this phase, the app could be demoed tomorrow even if no model were ever trained.

## Inputs

- [TestPlan.md](../TestPlan.md), [Security.md](../Security.md), [Deployment.md](../Deployment.md)
- Hosting decision ([OpenQuestions Q-06](../OpenQuestions.md#q-06)) and auth decision ([Q-07](../OpenQuestions.md#q-07))

## Deliverables

- Playwright E2E suite + axe checks in CI (`e2e.yml`)
- Security checklist completed with evidence
- Performance report (backend timings, Lighthouse, bundle sizes)
- `apps/api/Dockerfile`, `apps/web/Dockerfile`, `docker-compose.yml`
- Demo deployment with token auth
- Pre-computed demo analyses + offline fallback procedure
- `release.yml` workflow; `v0.6.0` release

## Tasks

### P6-01 · Playwright E2E (H+G)

- `apps/web/e2e/` with fixtures: start the API (`APP_ENV=test`, mock detectors, `JUDGE_ENABLED=false`) and the web app via Playwright `webServer`.
- Implement E-01 … E-09 from [TestPlan.md](../TestPlan.md#e2e-playwright). Use the tiny fixture files from the API tests.
- `@axe-core/playwright` on the key pages in both themes (E-08).
- `e2e.yml`: runs on the `run-e2e` label and nightly; uploads traces on failure.

### P6-02 · Security checklist (H+G)

Work through [Security.md §9](../Security.md#9-security-checklist-per-release) and record evidence in the PR:

- Upload fuzzing: run a small corpus of malformed files (truncated, polyglot, wrong magic, huge dimensions, zero-length) through `POST /media` → only documented errors, no 500s, no leftover temp files.
- Path traversal attempts on artifact and media IDs → 404.
- Prompt-injection test M-13 + the contradiction check.
- Headers verified with `curl -I` (API) and the browser (CSP on web).
- CORS restricted to the web origin in the demo config.
- `pip-audit` and `pnpm audit --prod` added to CI.
- Container runs as non-root with only `data/` writable.

### P6-03 · Performance pass (H+G)

- **Backend:** `scripts/bench.py` runs each demo clip N=3 times and prints per-stage timings from `agent_runs` → compare with [TechSpecifications.md §6](../TechSpecifications.md#6-performance-targets-backend-cpu-mock-detectors). Profile the slowest stage (`py-spy` or `cProfile`) and fix hot spots.
- **Frontend:** `@next/bundle-analyzer`; confirm three/wavesurfer/recharts are split; Lighthouse CI on `/`, `/dashboard`, `/analyses/[id]`; fix LCP/CLS issues.
- Record results in `docs/perf/2026-11-perf.md` (create `docs/perf/`).

### P6-04 · Docker (H)

- `apps/api/Dockerfile`: multi-stage — `uv` builder → `python:3.12-slim` runtime with `ffmpeg` installed, non-root user, `DATA_DIR=/data` volume, healthcheck on `/api/v1/health`, entrypoint runs `alembic upgrade head` then `uvicorn ... --workers 1`.
- `apps/web/Dockerfile`: Next.js `output: "standalone"`, non-root.
- `docker-compose.yml`: `api`, `web`, optional `postgres` profile, named volume for `/data`.

**Accept:** `docker compose up --build` on a clean machine gives a working app.

### P6-05 · Demo deployment (H+G)

Per [Deployment.md](../Deployment.md): deploy the web app to Vercel and the API to the chosen host with `APP_ENV=production`, `API_ACCESS_TOKEN`, restricted CORS, `RETENTION_DAYS=3`, JSON logs. Next.js route handlers proxy API calls and attach the token server-side. Run the post-deploy smoke test ([Deployment.md §6](../Deployment.md#6-post-deploy-smoke-test-5-minutes)).

### P6-06 · Demo fallback (H+G)

- Load the demo set on the deployment and locally (`load_demo_set.py`).
- Write `docs/DemoRunbook.md`: the exact click path for the demo, what to do if the network fails (switch to local compose; Judge falls back to rules), and which pre-computed analyses to open if uploads are slow.

### P6-07 · Release v0.6.0 (H+G)

- `release.yml`: on tag `v*`, build and push the API image to GHCR and create a GitHub Release from the Changelog section.
- Update [Changelog.md](../Changelog.md), bump versions, tag `v0.6.0`.

## Exit criteria

- [ ] E2E suite green in CI; axe clean on key pages in both themes
- [ ] Security checklist complete with evidence
- [ ] Performance budgets met (or deviations documented with reasons)
- [ ] `docker compose up` works on a clean machine
- [ ] Demo URL live with token auth; smoke test passes
- [ ] 20 consecutive demo-set analyses on the deployment with zero failures
- [ ] DemoRunbook written and rehearsed once
- [ ] `v0.6.0` released

## End-of-phase demo

Present the deployed app to someone outside the team (a friend, a TA) using only the DemoRunbook. Note every place they got confused and fix it.

## Risks

| Risk | Mitigation |
|---|---|
| Free hosting sleeps or has cold starts | Warm it before demos; local compose fallback |
| Persistent storage missing on the host | Choose a host with a volume; otherwise SQLite + data on a mounted disk |
| Flaky E2E tests | Wait on UI state, not timeouts; fixture files tiny; retry once in CI only |

## Handoff

- **To Phase 7:** a stable, deployed product; `DETECTOR_*` env vars are the switch; the `ml/` folder is ready.

## References

[TestPlan.md](../TestPlan.md) · [Security.md](../Security.md) · [Deployment.md](../Deployment.md)
