# Deployment

How Aletheia is built, released, deployed, and rolled back. The first real deployment happens in Phase 6 (P6-05).

## 1. Environments

| Environment | Purpose | Web | API | DB | Detectors |
|---|---|---|---|---|---|
| **local** | Development | `pnpm dev` :3000 | `fastapi dev` :8000 | SQLite | mock (Phase 7: real, CPU) |
| **demo** | Professor / viva / sharing | Vercel | Container host (see §3) | Postgres (managed) or SQLite on a volume | mock → real |
| **ml** (Phase 7+) | GPU inference | – | Lab GPU machine or cloud GPU container | Postgres | real, GPU |

There is no separate staging; PR preview deployments of the web app (Vercel previews) point at the demo API.

## 2. Build artifacts

| Artifact | Built by | Contents |
|---|---|---|
| `aletheia-api` Docker image | `apps/api/Dockerfile` | Python 3.12-slim + ffmpeg + app + `uv.lock` deps; runs `alembic upgrade head` then `uvicorn app.main:app --host 0.0.0.0 --port 8000 --workers 1` |
| `aletheia-web` Docker image (optional) | `apps/web/Dockerfile` | Next.js `output: "standalone"` build |
| Vercel build | Vercel Git integration | Next.js app from `apps/web` |
| Model weights (Phase 7) | `ml/` export scripts | Stored outside git (Hugging Face Hub private repo or release assets); downloaded at container start into `DATA_DIR/models/` and verified by SHA-256 |

`--workers 1` is deliberate: the in-process job runner and SSE event fan-out live in one process ([ADR-006](Decisions.md#adr-006)).

## 3. Hosting options (decide in [OpenQuestions.md Q-06](OpenQuestions.md#q-06))

| Option | Good for | Notes |
|---|---|---|
| **Vercel (web) + Hugging Face Spaces, Docker SDK (API)** — *recommended for the mock phase* | Free, public demo URL | Spaces gives a persistent Docker container; GPU hardware can be enabled later. Use a persistent storage volume for `data/` |
| Vercel + Render / Railway / Fly.io (API) | Simple container hosting | Needs a persistent disk for `data/`; free tiers may sleep |
| Lab GPU machine + Cloudflare Tunnel | Phase 7 with real models on a GPU we already have | No inbound ports opened; the tunnel exposes the API on an HTTPS hostname. Set up with the `cf` CLI (`cf cli search "create tunnel"`) and run `cloudflared` on the machine |
| Everything local (docker compose) on the demo laptop | Viva-day fallback | No network dependency except the Judge (rules fallback covers it) |

Demo-day rule: **always have the local docker-compose fallback ready** with the demo set pre-analyzed.

## 4. Configuration per environment

- Secrets (`ANTHROPIC_API_KEY`, `API_ACCESS_TOKEN`, `DATABASE_URL`) are set in the host's secret manager (Vercel env vars, Spaces secrets, etc.), never baked into images.
- Demo sets: `APP_ENV=production`, `API_ACCESS_TOKEN` set, `CORS_ORIGINS` = the Vercel domain only, `RETENTION_DAYS=3`, `LOG_FORMAT=json`.
- The web app on Vercel proxies API calls through Next.js route handlers when a token is needed, so the token stays server-side.

## 5. Release process

1. All Phase exit criteria met; CI green on `main`.
2. Update [Changelog.md](Changelog.md): move **Unreleased** to the new version with today's date.
3. Bump versions: `apps/api/pyproject.toml` (also exposed as `pipeline_version`) and `apps/web/package.json`.
4. PR `chore(release): vX.Y.Z` → review → merge.
5. Tag and push:

```bash
git tag -a vX.Y.Z -m "vX.Y.Z"
```

```bash
git push origin vX.Y.Z
```

6. The tag triggers the release workflow: build and push the API image tagged `vX.Y.Z` and `latest` to GHCR (`ghcr.io/harshdeepathawale/aletheia-api`), create a GitHub Release with the changelog section.
7. Deploy: point the API host at image `vX.Y.Z`; Vercel deploys `main` automatically.
8. Smoke test (below). If it fails → roll back (§7).

## 6. Post-deploy smoke test (5 minutes)

- [ ] `GET /api/v1/health/ready` → `ready` (or `degraded` only because the Judge is disabled on purpose)
- [ ] Web `/dashboard` loads, API status dot is green
- [ ] Upload a demo image → analysis completes → verdict shown
- [ ] Upload a demo video with audio → war room streams all 4 agents → report typed out
- [ ] Refresh mid-analysis → war room restores
- [ ] PDF downloads
- [ ] MOCK badges shown if mock detectors are active

## 7. Rollback

| What broke | Rollback |
|---|---|
| API release | Redeploy the previous image tag (`vX.Y.(Z-1)`); images are immutable per tag |
| Web release | Vercel dashboard → Deployments → previous deployment → "Promote to production" (instant) |
| DB migration | Every migration has a `downgrade()`. Roll back the code first, then `alembic downgrade -1` **only if** the new schema breaks the old code. Take a DB backup before any release with a migration |
| Model weights (Phase 7) | Set `DETECTOR_*` back to the previous `backend_version` (older weights stay on disk) or to `mock-*` as a last resort |
| Judge misbehaving | Set `JUDGE_ENABLED=false` → rules Judge, no redeploy needed beyond a restart |

## 8. Backups and data

- Postgres: daily automated backups from the managed provider (or `pg_dump` cron on a VM), keep 7 days.
- SQLite on a volume: copy `app.db` before each release.
- Uploaded media are temporary by design (retention job) and are **not** backed up.
- Model weights and the demo set are reproducible from `ml/` and `data/demo/manifest.json`.

## 9. CI/CD workflows

| Workflow | Trigger | Steps |
|---|---|---|
| `ci.yml` | PR, push to `main` | API: ruff, pyright, pytest (+coverage). Web: lint, typecheck, vitest, build. Contract check: regenerate types and fail if `git diff` is non-empty |
| `e2e.yml` | PR label `run-e2e`, nightly | Start API (mock detectors, rules Judge) + web, run Playwright + axe |
| `release.yml` | Tag `v*` | Build & push API image to GHCR, create GitHub Release |
