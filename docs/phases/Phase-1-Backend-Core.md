# Phase 1 — Backend Core

| | |
|---|---|
| **Weeks** | Weeks 2–3 (2026-10-12 → 10-23) |
| **Owner** | H |
| **Depends on** | Phase 0 |
| **Runs in parallel with** | Phase 3 (frontend foundation) |
| **Unblocks** | Phase 2 (agents), Phase 4 (war room needs SSE — the stub pipeline in P1-09 unblocks it early) |
| **Release** | `v0.1.0` |

## Goal

A file can be uploaded, an analysis created, and a (stub) pipeline streams the complete event sequence to any client over SSE — with persistence, replay after disconnect, cancellation, and every error mapped to the documented problem+json codes.

## Inputs

- API skeleton, CI, and `.env.example` from Phase 0
- [API.md](../API.md) (contract v1), [Schema.md](../Schema.md), [TechSpecifications.md §4](../TechSpecifications.md#4-backend-design)

## Deliverables

- All tables from Schema.md with an initial Alembic migration
- Endpoints: `/health`, `/health/ready`, `/media*`, `/analyses*` (create, list, get, events, cancel, rerun, delete)
- In-process job runner with recovery
- Event bus with DB persistence + SSE with `Last-Event-ID` replay
- Stub pipeline that emits the full event sequence for image/video/audio
- Rate limiting, retention job, structured logging with request IDs
- Tests: ≥ 80% coverage on `services/` and `events/`

## Module map (files created this phase)

```
apps/api/app/
├── main.py                 # app factory: middleware, routers, lifespan (DB, runner, retention)
├── config.py               # Settings
├── api/v1/{health,media,analyses}.py
├── schemas/{common,media,analysis,events,errors}.py
├── db/{engine,session,models,base}.py + repositories/{media,analysis,events}.py
├── services/{media_service,analysis_service}.py
├── jobs/{runner,protocol}.py
├── events/{bus,sse,types}.py
├── pipeline/stub.py        # replaced by the real graph in Phase 2
├── media/probe.py          # ffprobe wrapper
├── storage/{base,local}.py
└── utils/{ids,errors,logging,ratelimit}.py
```

## Tasks

### P1-01 · Settings, logging, errors, CORS

- `Settings` covers every variable in [Setup.md §4](../Setup.md#4-environment-variables) with types and defaults; `get_settings()` cached.
- structlog: pretty console in dev, JSON in prod; a middleware adds `request_id` (from `X-Request-ID` or generated) to the log context and response header.
- `AppError(code, status, detail, errors=None)` + exception handlers that render RFC 9457 problem+json. Map `RequestValidationError` → `VALIDATION_ERROR` with the `errors` array; unexpected exceptions → `INTERNAL_ERROR` (log the traceback, return no internals).
- CORS allowlist from `CORS_ORIGINS`.

**Accept:** hitting an unknown route or sending a bad body returns problem+json with `code` and `request_id`.

### P1-02 · Database

- `uv add sqlalchemy aiosqlite asyncpg alembic`.
- Async engine + `async_sessionmaker`; a `get_session` dependency; SQLite pragmas `journal_mode=WAL`, `foreign_keys=ON`.
- ORM models for every table in Schema.md (enums as `String` + `CheckConstraint`, JSON columns typed via Pydantic on the way in/out).
- `alembic init -t async alembic`; set `render_as_batch=True` for SQLite; generate and **review** the initial migration.
- Repositories expose only what services need (e.g. `AnalysisRepo.list(filters, cursor, limit)`).

**Accept:** `alembic upgrade head` works on SQLite and on a Postgres container (`docker run postgres:16`); downgrade works.

### P1-03 · IDs and storage

- `new_id("ana")` → `ana_` + ULID. Validate prefixes on path params (wrong prefix → 404, not 500).
- `Storage` protocol: `save_stream`, `path_for`, `open`, `delete_prefix`, `exists`. `LocalStorage` rooted at `DATA_DIR`, refusing any path that resolves outside it.

**Accept:** unit tests for ID format and for path-escape attempts (`..`, absolute paths).

### P1-04 · `POST /media`

1. Read the multipart stream chunk by chunk into a temp file, updating SHA-256 and a byte counter; abort with `FILE_TOO_LARGE` the moment the limit for the sniffed kind is passed (sniff after the first 8 KB).
2. Sniff with `filetype` → allowlist → `kind`. Unknown → `UNSUPPORTED_MEDIA_TYPE`.
3. Dedup: if the SHA-256 exists, delete the temp file and return `200` with the existing media.
4. `ffprobe` (see [Security.md §4](../Security.md#4-upload-handling)) → duration, dimensions, fps, codecs, audio presence. Enforce duration and pixel caps. Probe failure → `MEDIA_DECODE_FAILED`.
5. Move into `data/media/{id}/original.{ext}`; create a 480 px WebP thumbnail (video: frame at 10% of duration via ffmpeg; audio: a small waveform PNG).
6. Insert the row with `expires_at = now + RETENTION_DAYS`; return `201`.

**Accept:** test cases B-01…B-09 in [TestPlan.md](../TestPlan.md#backend) pass.

### P1-05 · Media read/delete

- `GET /media/{id}`, `/thumbnail`, `/file` with HTTP Range (use Starlette `FileResponse`, which handles ranges, and set `Content-Type` from the DB).
- `DELETE /media/{id}` removes DB rows (cascade) and `data/media/{id}` + each analysis directory.

**Accept:** B-24, B-28 pass; a `<video>` element can seek in the browser.

### P1-06 · Job runner

- `JobQueue` protocol (`enqueue`, `cancel`, `start`, `stop`, `position`). `AsyncioJobQueue`: `asyncio.Queue`, `JOB_CONCURRENCY` worker tasks started in the FastAPI lifespan.
- Each job gets an `asyncio.Event` for cancellation, stored in a dict by analysis ID.
- Worker: mark `running` → run the pipeline → mark terminal state; any exception → `failed` with `INTERNAL_ERROR` + logged traceback.
- Startup recovery: `running` → `failed` (`INTERRUPTED`); `queued` → re-enqueue in `created_at` order.
- `MAX_QUEUED_JOBS` → `QUEUE_FULL`.

**Accept:** B-14, B-21, B-22 pass.

### P1-07 · Event bus and SSE

- `EventBus.publish(analysis_id, type, agent=None, data={})`: assign the next `seq` (per-analysis counter kept in memory, initialised from `MAX(seq)`), insert into `analysis_events`, then push to subscriber queues.
- `EventBus.subscribe(analysis_id, after_seq)`: an async iterator that first yields persisted events with `seq > after_seq`, then live ones, without gaps or duplicates (subscribe to live first, buffer, then replay from the DB, then drain the buffer skipping seen `seq`).
- SSE endpoint with `sse-starlette`: `id` = seq, `event` = type, `data` = JSON envelope from [API.md §6](../API.md#6-server-sent-events); `: ping` every 15 s; close after a terminal event; honour `Last-Event-ID` header and `?last_event_id=` query.
- `judge.token` events can be many — batch DB inserts (flush every 100 ms or 20 events).

**Accept:** B-11, B-12, B-13 pass; `curl -N` shows a clean stream.

### P1-08 · Analyses endpoints

- `POST /analyses` validates `AnalysisOptions` (including the dev-only `mock_scenario` rule), creates the row, enqueues, publishes `analysis.queued`, returns 202 + `Location`.
- `GET /analyses` with filters, `sort`, cursor pagination (cursor = base64 of `(sort_value, id)`), `total`.
- `GET /analyses/{id}` assembles the full `Analysis` shape (media, agents, verdict, report).
- `POST /cancel` (409 if terminal), `POST /rerun`, `DELETE`.

**Accept:** B-10, B-26, B-29 pass; responses validate against the OpenAPI schema.

### P1-09 · Stub pipeline (unblocks the frontend)

`pipeline/stub.py` emits the **exact** event sequence the real pipeline will, with realistic timing (`asyncio.sleep`), for each media kind and each `mock_scenario`, and writes `agent_runs`, `findings`, `frame_scores`, and a verdict. Placeholder images for artifacts (solid gradients labelled "stub").

**Accept:** G can build the whole war room against it; switching to the real graph in Phase 2 needs no frontend change.

### P1-10 · Rate limits, readiness, retention

- `slowapi` limits from [API.md §5](../API.md#5-rate-limits-per-ip-defaults) → `RATE_LIMITED` with `Retry-After`.
- `/health/ready`: DB `SELECT 1`, `ffprobe -version`, write+delete a temp file in `DATA_DIR`, detector registry (stub for now), Judge configured (optional).
- Retention task in the lifespan: on startup and hourly.

**Accept:** B-23, B-25 pass; readiness shows `ffmpeg: fail` when ffmpeg is removed from PATH.

### P1-11 · Tests

- `tests/conftest.py`: temp `DATA_DIR`, temp SQLite, app with overridden settings, `AsyncClient`, a fixture generator script `tests/fixtures/make_fixtures.py` (uses ffmpeg to synthesize tiny files).
- Unit: IDs, cursor encoding, storage path safety, error mapping.
- Integration: upload → analyze → stream → completed, for each media kind.

**Accept:** coverage ≥ 80% on `services/` and `events/`; CI green.

## Exit criteria

- [ ] The curl walkthrough in [API.md §7](../API.md#7-example-session) works end to end
- [ ] Disconnect/reconnect with `Last-Event-ID` resumes without gaps or duplicates
- [ ] All backend cases B-01…B-14, B-21…B-29 pass
- [ ] Migrations run on SQLite and Postgres
- [ ] `v0.1.0` tagged; Changelog updated

## End-of-phase demo

1. `curl` upload of the demo video → media JSON with duration/fps.
2. Create analysis → `curl -N` the events stream; kill it halfway; reconnect with `Last-Event-ID` → it resumes.
3. Cancel a second analysis mid-run.
4. Show `/docs` (Swagger) with every endpoint documented.

## Risks

| Risk | Mitigation |
|---|---|
| SSE replay race (duplicates/gaps) | Subscribe-then-replay-then-dedupe by `seq`; dedicated test B-12 |
| SQLite locking under concurrent writes | WAL mode, one process, batched token inserts |
| Large uploads blocking the event loop | Chunked async reads; hashing in the same loop is cheap; thumbnails in `to_thread` |

## Handoff

- **To Phase 2:** runner, bus, persistence, stub pipeline interface (`run_pipeline(state, ctx)`) to be replaced by the LangGraph graph.
- **To Phase 3/4 (G):** a running API with the stub pipeline; `openapi.json` for `pnpm gen:api`.

## References

[API.md](../API.md) · [Schema.md](../Schema.md) · [TechSpecifications.md §4](../TechSpecifications.md#4-backend-design) · [Security.md §4](../Security.md#4-upload-handling) · [TestPlan.md](../TestPlan.md)
