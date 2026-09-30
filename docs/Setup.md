# Local Setup

From a clean clone to a running app. Written for **Windows 11** (our laptops); macOS/Linux commands are the same unless noted.

> Status: the app code doesn't exist yet (Phase 0 creates it). This doc describes the target setup; P0-06 verifies it on both laptops and fixes anything that's wrong.

## 1. Prerequisites

| Tool | Version | Install (Windows) | Check |
|---|---|---|---|
| Git | 2.40+ | `winget install Git.Git` | `git --version` |
| Node.js | 22 LTS or newer | `winget install OpenJS.NodeJS.LTS` | `node -v` |
| pnpm | 9+ | `corepack enable` then `corepack prepare pnpm@latest --activate` | `pnpm -v` |
| Python | 3.12 | `winget install Python.Python.3.12` | `py -3.12 --version` |
| uv | latest | `winget install astral-sh.uv` | `uv --version` |
| FFmpeg | 6+ (includes ffprobe) | `winget install Gyan.FFmpeg` — restart the terminal afterwards | `ffmpeg -version` |
| Docker Desktop | optional | For Postgres and container builds | `docker --version` |

Keep the repo **outside OneDrive-synced folders** if you can (e.g. `C:\dev\UML501-MACHINE-LEARNING`). OneDrive syncing `node_modules`, `.venv`, and `data/` is slow and can lock files mid-build. If you keep it in OneDrive, at least mark those folders as "Free up space"/excluded.

## 2. Clone

```bash
git clone https://github.com/HarshdeepAthawale/UML501-MACHINE-LEARNING.git
```

```bash
cd UML501-MACHINE-LEARNING
```

## 3. Backend (`apps/api`)

```bash
cd apps/api
```

```bash
uv sync
```

```bash
copy .env.example .env
```

(`cp .env.example .env` on macOS/Linux.) Then edit `.env` — at minimum set `ANTHROPIC_API_KEY` if you want the LLM Judge; without it the rules Judge is used.

```bash
uv run alembic upgrade head
```

```bash
uv run python scripts/fetch_assets.py
```

(Downloads the small YuNet face-detector ONNX file into `data/assets/`.)

```bash
uv run python scripts/seed_models.py
```

```bash
uv run fastapi dev app/main.py --port 8000
```

Open http://localhost:8000/docs for the interactive API docs and http://localhost:8000/api/v1/health/ready for the readiness check.

## 4. Environment variables

### `apps/api/.env`

| Variable | Default | Description |
|---|---|---|
| `APP_ENV` | `development` | `development \| test \| production`. Enables `mock_scenario` and `/admin/purge` in development only |
| `DATABASE_URL` | `sqlite+aiosqlite:///../../data/app.db` | Postgres: `postgresql+asyncpg://user:pass@host:5432/aletheia` |
| `DATA_DIR` | `../../data` | Root for media, artifacts, assets |
| `CORS_ORIGINS` | `http://localhost:3000` | Comma-separated allowlist |
| `API_ACCESS_TOKEN` | *(empty)* | If set, every request needs `Authorization: Bearer <token>` |
| `MAX_UPLOAD_MB_IMAGE` / `_VIDEO` / `_AUDIO` | `20` / `200` / `50` | Upload caps |
| `MAX_VIDEO_SECONDS` / `MAX_AUDIO_SECONDS` | `120` / `300` | Duration caps |
| `MAX_IMAGE_PIXELS` | `40000000` | Rejects decompression bombs |
| `NUM_FRAMES` | `32` | Default frames sampled per video |
| `JOB_CONCURRENCY` | `1` | Parallel analyses |
| `MAX_QUEUED_JOBS` | `20` | Above this, `POST /analyses` → 503 |
| `DETECTOR_VISUAL` / `_TEMPORAL` / `_AUDIO` | `mock-visual` / `mock-temporal` / `mock-audio` | Detector registry keys |
| `ANTHROPIC_API_KEY` | *(empty)* | Judge LLM. Server-side only. Never commit it |
| `JUDGE_ENABLED` | `true` | `false` → always use the rules Judge |
| `JUDGE_MODEL` | `claude-opus-5-5` | |
| `JUDGE_EFFORT` | `medium` | `low \| medium \| high` |
| `JUDGE_TIMEOUT_S` | `60` | |
| `AGENT_LLM_OBSERVATIONS` | `false` | Optional Claude observations for specialist agents |
| `RETENTION_DAYS` | `7` | Uploaded media auto-delete |
| `LOG_LEVEL` | `INFO` | |
| `LOG_FORMAT` | `pretty` | `pretty \| json` |

### `apps/web/.env.local`

| Variable | Default | Description |
|---|---|---|
| `NEXT_PUBLIC_API_BASE_URL` | `http://localhost:8000/api/v1` | Browser → API base |
| `NEXT_PUBLIC_APP_NAME` | `Aletheia` | Display name |
| `API_ACCESS_TOKEN` | *(empty)* | Only if the API requires a token; used server-side by Next route handlers to proxy requests so the token isn't exposed in the browser |

## 5. Frontend (`apps/web`)

In a second terminal:

```bash
cd apps/web
```

```bash
pnpm install
```

```bash
copy .env.example .env.local
```

```bash
pnpm gen:api
```

(Needs the API running; regenerates `src/lib/api/generated/schema.d.ts`.)

```bash
pnpm dev
```

Open http://localhost:3000.

## 6. Everyday commands

| Task | Backend (`apps/api`) | Frontend (`apps/web`) |
|---|---|---|
| Run dev server | `uv run fastapi dev app/main.py --port 8000` | `pnpm dev` |
| Tests | `uv run pytest` | `pnpm test` |
| Tests with coverage | `uv run pytest --cov=app` | `pnpm test -- --coverage` |
| E2E | – | `pnpm e2e` (needs both servers) |
| Lint | `uv run ruff check .` | `pnpm lint` |
| Format | `uv run ruff format .` | `pnpm format` |
| Type check | `uv run pyright` | `pnpm typecheck` |
| New migration | `uv run alembic revision --autogenerate -m "msg"` | – |
| Regenerate API types | – | `pnpm gen:api` |
| Add a dependency | `uv add <pkg>` | `pnpm add <pkg>` |

## 7. Docker (optional)

From the repo root:

```bash
docker compose up --build
```

Starts `api` on :8000, `web` on :3000, and (with the `postgres` profile) Postgres on :5432:

```bash
docker compose --profile postgres up --build
```

## 8. Troubleshooting

| Symptom | Fix |
|---|---|
| `ffprobe not found` / readiness shows `ffmpeg: fail` | FFmpeg isn't on `PATH`. Restart the terminal after installing; check `where ffprobe` |
| `uv` uses the wrong Python | `uv python pin 3.12` in `apps/api` |
| CORS error in the browser | Add the web origin to `CORS_ORIGINS` and restart the API |
| `pnpm gen:api` fails | Start the API first; check `NEXT_PUBLIC_API_BASE_URL` |
| SQLite "database is locked" | Only one API process should use the SQLite file; stop duplicate servers |
| Slow installs / file-lock errors | Move the repo out of OneDrive (see §1) |
| Judge always shows `RULES` | `ANTHROPIC_API_KEY` missing or invalid, or `JUDGE_ENABLED=false`; check `/health/ready` |
