# API Reference — v1

Base URL: `http://localhost:8000/api/v1` · Format: JSON (UTF-8) · Live docs: `http://localhost:8000/docs` (Swagger) and `/openapi.json`

This file is the **contract**. The frontend generates its types from `/openapi.json` (`pnpm gen:api`), so the Pydantic schemas in `apps/api/app/schemas/` must match this document. Change both in the same PR.

## 1. Conventions

| Topic | Rule |
|---|---|
| Versioning | Path prefix `/api/v1`. Breaking changes → `/api/v2` (none planned). |
| IDs | Prefixed ULIDs: `med_`, `ana_`, `run_`, `fnd_`, `art_`, `mdl_`. Sortable by creation time. |
| Timestamps | ISO 8601 UTC with `Z`, e.g. `2026-10-05T14:03:22.512Z` |
| Probabilities | Floats in [0, 1]. `score` always means **P(fake)**. |
| Durations | Milliseconds, suffix `_ms` |
| Casing | `snake_case` JSON fields |
| Pagination | Cursor based: `?limit=20&cursor=<opaque>` → `{ "items": [...], "next_cursor": "..." \| null }`. `limit` max 100. |
| Auth | None in local dev. If `API_ACCESS_TOKEN` is set, send `Authorization: Bearer <token>` (see [Security.md](Security.md#3-authentication)). SSE accepts `?access_token=` because `EventSource` can't set headers. |
| Request ID | Every response has `X-Request-ID`; clients may send their own. |
| Idempotency | `POST /media` dedupes by SHA-256 of the file contents (returns the existing media with `200`). |

## 2. Errors

All errors use [RFC 9457](https://www.rfc-editor.org/rfc/rfc9457) Problem Details, `Content-Type: application/problem+json`:

```json
{
  "type": "https://aletheia.local/errors/FILE_TOO_LARGE",
  "title": "File too large",
  "status": 413,
  "code": "FILE_TOO_LARGE",
  "detail": "Video uploads are limited to 200 MB (received 312 MB).",
  "instance": "/api/v1/media",
  "request_id": "req_01J9...",
  "errors": null
}
```

`errors` is set for validation failures: `[{ "field": "options.num_frames", "message": "must be ≤ 64" }]`.

| Code | HTTP | When |
|---|---|---|
| `VALIDATION_ERROR` | 422 | Body/query fails schema validation |
| `UNSUPPORTED_MEDIA_TYPE` | 415 | Magic bytes are not an allowed image/video/audio type |
| `FILE_TOO_LARGE` | 413 | Over the per-kind size limit |
| `MEDIA_TOO_LONG` | 422 | Video > `MAX_VIDEO_SECONDS` (120) or audio > 300 s |
| `MEDIA_DECODE_FAILED` | 422 | ffprobe/decoder can't read the file (corrupt, truncated, zero-length) |
| `MEDIA_NOT_FOUND` | 404 | Unknown `media_id` |
| `ANALYSIS_NOT_FOUND` | 404 | Unknown `analysis_id` |
| `ARTIFACT_NOT_FOUND` | 404 | Unknown artifact |
| `ANALYSIS_NOT_CANCELLABLE` | 409 | Cancel requested on a finished analysis |
| `ANALYSIS_NOT_COMPLETE` | 409 | Report requested before completion |
| `QUEUE_FULL` | 503 | More than `MAX_QUEUED_JOBS` (default 20) waiting |
| `RATE_LIMITED` | 429 | Too many requests; `Retry-After` header set |
| `UNAUTHORIZED` | 401 | Missing/invalid token when auth is enabled |
| `INTERNAL_ERROR` | 500 | Unexpected; details only in server logs |

## 3. Shared objects

### Media

```json
{
  "id": "med_01J9Z3K4X8Q2",
  "kind": "video",
  "original_filename": "interview_clip.mp4",
  "mime_type": "video/mp4",
  "size_bytes": 18432211,
  "sha256": "9f2c…",
  "duration_ms": 31240,
  "width": 1920,
  "height": 1080,
  "fps": 29.97,
  "has_audio": true,
  "thumbnail_url": "/api/v1/media/med_01J9Z3K4X8Q2/thumbnail",
  "file_url": "/api/v1/media/med_01J9Z3K4X8Q2/file",
  "metadata": { "container": "mov,mp4", "video_codec": "h264", "audio_codec": "aac", "encoder": "Lavf60.3.100" },
  "created_at": "2026-10-05T14:03:20.001Z",
  "expires_at": "2026-10-12T14:03:20.001Z"
}
```

`kind` ∈ `image | video | audio`. `duration_ms`, `fps`, `has_audio` are `null` for images; `width`/`height` are `null` for audio.

### AnalysisOptions

```json
{
  "agents": ["visual", "temporal", "audio"],
  "num_frames": 32,
  "judge": true,
  "generate_heatmaps": true,
  "mock_scenario": null
}
```

| Field | Type | Default | Rules |
|---|---|---|---|
| `agents` | array of `visual \| temporal \| audio` | all applicable | Agents not applicable to the media kind are skipped, not rejected |
| `num_frames` | int | 32 | 8–64 |
| `judge` | bool | true | false → rule-based summary only |
| `generate_heatmaps` | bool | true | |
| `mock_scenario` | `real \| fake \| uncertain \| disagree \| agent_fail \| null` | null | Accepted only when `APP_ENV=development`; otherwise `VALIDATION_ERROR` |

### Verdict

```json
{
  "label": "fake",
  "fake_probability": 0.912,
  "confidence": 0.84,
  "disagreement": 0.21,
  "agreement": "consensus",
  "weights": { "visual": 0.5, "temporal": 0.3, "audio": 0.2 },
  "contributions": { "visual": 0.61, "temporal": 0.29, "audio": 0.10 }
}
```

`label` ∈ `real | fake | uncertain`. `agreement` ∈ `consensus | mixed | conflict` (from `disagreement` < 0.25 / < 0.5 / ≥ 0.5). `contributions` = each agent's share of |z|. Math: [TechSpecifications.md §4.7](TechSpecifications.md#47-fusion-the-math).

### AgentResult

```json
{
  "id": "run_01J9Z3M0A1",
  "agent": "visual",
  "status": "completed",
  "backend": "mock",
  "backend_version": "mock-1",
  "score": 0.94,
  "confidence": 0.88,
  "summary": "Blending artifacts along the jaw line in 21 of 32 frames.",
  "metrics": { "faces_detected": 32, "mean_blur": 142.7 },
  "findings": [ /* Finding */ ],
  "frame_scores": [ { "frame_index": 0, "timestamp_ms": 0, "score": 0.81, "has_face": true } ],
  "artifacts": [ /* ArtifactRef */ ],
  "started_at": "2026-10-05T14:03:22.100Z",
  "completed_at": "2026-10-05T14:03:27.900Z",
  "duration_ms": 5800,
  "error": null
}
```

- `agent` ∈ `visual | temporal | audio | judge`
- `status` ∈ `pending | running | completed | failed | skipped | cancelled`
- `backend` ∈ `mock | model | llm | rules` — **the UI shows a MOCK badge when this is `mock`**
- `score`/`confidence` are `null` for the Judge and for skipped/failed runs
- `error` when failed: `{ "code": "AGENT_TIMEOUT", "message": "..." }`

### Finding

```json
{
  "id": "fnd_01J9Z3M9",
  "agent": "temporal",
  "kind": "temporal_flicker",
  "title": "Flicker around the mouth region",
  "description": "Frame-to-frame SSIM in the lower face drops sharply at 12.4–13.1 s while the background stays stable.",
  "severity": "high",
  "score": 0.77,
  "time_start_ms": 12400,
  "time_end_ms": 13100,
  "frame_index": 12,
  "region": { "x": 0.41, "y": 0.52, "w": 0.18, "h": 0.12 },
  "artifact_id": "art_01J9Z3N2"
}
```

`severity` ∈ `info | low | medium | high`. `region` uses coordinates normalized to [0,1] relative to the frame. `kind` values: `blending_boundary`, `texture_anomaly`, `frequency_artifact`, `temporal_flicker`, `motion_mismatch`, `face_jitter`, `lip_sync_mismatch`, `voice_artifact`, `spectral_anomaly`, `metadata_note`, `no_face_detected`, `low_quality_input`.

### ArtifactRef

```json
{
  "id": "art_01J9Z3N2",
  "kind": "heatmap",
  "mime_type": "image/webp",
  "url": "/api/v1/artifacts/art_01J9Z3N2",
  "width": 224,
  "height": 224,
  "frame_index": 12,
  "agent": "visual",
  "meta": { "overlay_on": "face_crop" }
}
```

`kind` ∈ `keyframe | face_crop | heatmap | spectrum | flow_map | waveform | spectrogram | report_pdf`.

### Analysis

```json
{
  "id": "ana_01J9Z3M0",
  "media": { /* Media */ },
  "status": "completed",
  "progress": 1.0,
  "stage": "done",
  "options": { /* AnalysisOptions */ },
  "verdict": { /* Verdict */ },
  "agents": [ /* AgentResult ×4 */ ],
  "report": {
    "markdown": "## Verdict: likely manipulated …",
    "key_evidence": [ { "finding_id": "fnd_01J9Z3M9", "why": "…" } ],
    "caveats": ["Low resolution reduces reliability of frequency cues."],
    "backend": "llm",
    "model": "claude-opus-5-5"
  },
  "is_mock": true,
  "pipeline_version": "0.2.0",
  "error": null,
  "created_at": "2026-10-05T14:03:21.000Z",
  "started_at": "2026-10-05T14:03:21.300Z",
  "completed_at": "2026-10-05T14:03:49.700Z",
  "duration_ms": 28400
}
```

- `status` ∈ `queued | running | completed | failed | cancelled`
- `stage` ∈ `queued | preprocessing | agents | fusion | judge | finalizing | done`
- `is_mock` is true if **any** scoring agent used `backend=mock`

`AnalysisSummary` (used in lists) = `id, status, stage, progress, verdict (label, fake_probability, confidence) | null, is_mock, media (id, kind, original_filename, thumbnail_url, duration_ms), created_at, duration_ms`.

## 4. Endpoints

### Health

| Method | Path | Description |
|---|---|---|
| GET | `/health` | Liveness → `{ "status": "ok", "version": "0.2.0" }` |
| GET | `/health/ready` | Readiness → `{ "status": "ready" \| "degraded", "checks": { "database": "ok", "ffmpeg": "ok", "storage": "ok", "detectors": "ok", "judge_llm": "disabled" } }`; returns 503 if any *required* check fails (judge is optional) |

### Media

| Method | Path | Description |
|---|---|---|
| POST | `/media` | Upload a file (multipart, field `file`) |
| GET | `/media/{media_id}` | Media object |
| GET | `/media/{media_id}/file` | Original bytes (supports HTTP `Range` for video seeking) |
| GET | `/media/{media_id}/thumbnail` | WebP thumbnail |
| DELETE | `/media/{media_id}` | Delete media, its analyses, and all files → `204` |

**POST /media**

```http
POST /api/v1/media
Content-Type: multipart/form-data; boundary=…

file=@interview_clip.mp4
```

- `201 Created` + `Media` for a new file; `200 OK` + existing `Media` if the SHA-256 already exists.
- Limits (defaults, configurable): image ≤ 20 MB (JPEG, PNG, WebP), video ≤ 200 MB and ≤ 120 s (MP4, MOV, WebM, MKV), audio ≤ 50 MB and ≤ 300 s (WAV, MP3, M4A, FLAC, OGG).
- Errors: `UNSUPPORTED_MEDIA_TYPE`, `FILE_TOO_LARGE`, `MEDIA_TOO_LONG`, `MEDIA_DECODE_FAILED`, `RATE_LIMITED`.

### Analyses

| Method | Path | Description |
|---|---|---|
| POST | `/analyses` | Start an analysis |
| GET | `/analyses` | List (paginated, filterable) |
| GET | `/analyses/{analysis_id}` | Full analysis (the shape above) |
| GET | `/analyses/{analysis_id}/events` | SSE stream (§6) |
| POST | `/analyses/{analysis_id}/cancel` | Cancel a queued/running analysis → `202` + `Analysis` |
| POST | `/analyses/{analysis_id}/rerun` | New analysis on the same media with the same (or overridden) options → `201` |
| DELETE | `/analyses/{analysis_id}` | Delete analysis + artifacts → `204` |
| GET | `/analyses/{analysis_id}/frames` | Per-frame data for the timeline |
| GET | `/analyses/{analysis_id}/report.pdf` | PDF report (generated on first request, then cached) |

**POST /analyses**

```json
{ "media_id": "med_01J9Z3K4X8Q2", "options": { "num_frames": 32, "judge": true } }
```

→ `202 Accepted` + `Analysis` (`status: "queued"`) and header `Location: /api/v1/analyses/ana_…`. Errors: `MEDIA_NOT_FOUND`, `VALIDATION_ERROR`, `QUEUE_FULL`, `RATE_LIMITED`.

**GET /analyses** query parameters:

| Param | Example | Notes |
|---|---|---|
| `limit`, `cursor` | `limit=20` | |
| `status` | `status=completed` | Repeatable |
| `label` | `label=fake` | `real \| fake \| uncertain` |
| `kind` | `kind=video` | |
| `q` | `q=interview` | Case-insensitive match on filename |
| `from`, `to` | `from=2026-10-01T00:00:00Z` | On `created_at` |
| `sort` | `sort=-created_at` | `created_at`, `-created_at`, `fake_probability`, `-fake_probability` |

→ `{ "items": [AnalysisSummary], "next_cursor": "…" | null, "total": 132 }`

**GET /analyses/{id}/frames**

```json
{
  "analysis_id": "ana_01J9Z3M0",
  "fps": 29.97,
  "frames": [
    {
      "frame_index": 12,
      "timestamp_ms": 12012,
      "has_face": true,
      "face_box": { "x": 0.38, "y": 0.21, "w": 0.24, "h": 0.36 },
      "scores": { "visual": 0.93, "temporal": 0.71 },
      "keyframe_url": "/api/v1/artifacts/art_…",
      "face_crop_url": "/api/v1/artifacts/art_…",
      "heatmap_url": "/api/v1/artifacts/art_…"
    }
  ],
  "segments": [ { "agent": "temporal", "start_ms": 12400, "end_ms": 13100, "severity": "high", "finding_id": "fnd_01J9Z3M9" } ]
}
```

### Artifacts

| Method | Path | Description |
|---|---|---|
| GET | `/artifacts/{artifact_id}` | Raw bytes with correct `Content-Type`; `Cache-Control: public, max-age=31536000, immutable` |

### Agents

| Method | Path | Description |
|---|---|---|
| GET | `/agents` | Registry of the 4 agents and their active detectors |

```json
{
  "items": [
    {
      "name": "visual",
      "display_name": "Visual Forensics",
      "description": "Examines face crops for blending, texture and generator artifacts.",
      "applies_to": ["image", "video"],
      "backend": "mock",
      "backend_version": "mock-1",
      "status": "ready",
      "avg_duration_ms": 5200
    }
  ]
}
```

`status` ∈ `ready | degraded | unavailable`. The Judge reports `backend: "llm"` when a key is configured, else `"rules"`.

### Models (leaderboard)

| Method | Path | Description |
|---|---|---|
| GET | `/models` | Leaderboard rows |
| GET | `/models/{model_id}` | Model card with full metrics and ROC points |
| GET | `/models/{model_id}/embeddings` | 3D embedding points for the explorer, `?dataset=celebdf_v2&limit=2000` |

```json
{
  "id": "mdl_gend_clip_l14",
  "name": "GenD (CLIP ViT-L/14)",
  "agent": "visual",
  "source": "ours",
  "train_dataset": "FF++ c23",
  "params_trainable_m": 0.1,
  "metrics": [
    { "dataset": "FF++ c23 (test)", "split": "in-domain", "auc": 0.991, "accuracy": 0.967, "eer": 0.041 },
    { "dataset": "Celeb-DF v2", "split": "cross-dataset", "auc": 0.93, "accuracy": null, "eer": 0.14 }
  ],
  "roc": { "FF++ c23 (test)": [[0.0, 0.0], [0.01, 0.62], [1.0, 1.0]] },
  "notes": "LayerNorm-only fine-tuning; 32 frames/video.",
  "is_active": true
}
```

`source` ∈ `ours | paper | placeholder`. `paper` rows must include a citation in `notes`; `placeholder` rows exist only until Phases 7–8 replace them and are rendered greyed out as "pending training". Numbers above are illustrative.

### Metrics (dashboard)

| Method | Path | Description |
|---|---|---|
| GET | `/metrics/overview` | KPIs for `/dashboard`, `?range=7d\|30d\|all` |

```json
{
  "range": "7d",
  "totals": { "analyses": 132, "completed": 124, "failed": 3, "cancelled": 5 },
  "verdicts": { "real": 61, "fake": 48, "uncertain": 15 },
  "by_kind": { "image": 40, "video": 71, "audio": 21 },
  "avg_duration_ms": { "image": 4100, "video": 27900, "audio": 9800 },
  "timeseries": [ { "date": "2026-10-05", "analyses": 21, "fake": 8 } ],
  "agents": [ { "agent": "visual", "success_rate": 0.99, "avg_duration_ms": 5200 } ]
}
```

### Settings / admin (local only)

| Method | Path | Description |
|---|---|---|
| GET | `/settings/public` | Non-secret runtime info for the Settings page: limits, judge enabled/model, retention days, `app_env` |
| POST | `/admin/purge` | Delete all media and analyses. Requires `{"confirm": "DELETE ALL"}`. Disabled unless `APP_ENV=development` or a token is set. |

### Live mode (Phase 9, stretch)

| Method | Path | Description |
|---|---|---|
| WS | `/live` | Client sends JPEG frames (≤ 5 fps, ≤ 640 px); server replies `{ "t": 1234, "visual": 0.12, "face_box": {...} }`. Visual agent only, no Judge. |

## 5. Rate limits (per IP, defaults)

| Endpoint | Limit |
|---|---|
| `POST /media` | 20 / minute |
| `POST /analyses`, `/rerun` | 10 / minute |
| Everything else | 300 / minute |

## 6. Server-Sent Events

`GET /api/v1/analyses/{id}/events` · `Content-Type: text/event-stream`

Each message:

```
id: 17
event: agent.progress
data: {"seq":17,"analysis_id":"ana_…","ts":"2026-10-05T14:03:24.120Z","type":"agent.progress","agent":"visual","data":{"progress":0.5,"message":"Scored 16/32 frames"}}
```

- `id` = per-analysis `seq` (1, 2, 3…). Reconnect with `Last-Event-ID: 17` to resume after 17.
- Opening the stream on a finished analysis replays all events, then closes.
- Comment heartbeat `: ping` every 15 s.

| Event type | `agent` | `data` fields | Emitted when |
|---|---|---|---|
| `analysis.queued` | – | `position` | Job enqueued |
| `analysis.started` | – | `media_kind`, `planned_agents` | Worker picks the job |
| `stage.changed` | – | `stage`, `progress` | Pipeline stage changes |
| `preprocess.progress` | – | `step` (`probe \| frames \| faces \| audio \| metadata`), `progress`, `message` | During preprocessing |
| `agent.started` | ✓ | `backend`, `backend_version` | Agent begins |
| `agent.progress` | ✓ | `progress`, `message` | Periodically |
| `agent.log` | ✓ | `message`, `level` (`info \| warn`) | Human-readable reasoning lines for the war room |
| `agent.finding` | ✓ | `Finding` | Each finding |
| `agent.frame_score` | ✓ | `frame_index`, `timestamp_ms`, `score` | Per scored frame (video) — drives the live timeline |
| `agent.artifact` | ✓ | `ArtifactRef` | Artifact saved |
| `agent.completed` | ✓ | `score`, `confidence`, `summary`, `duration_ms` | Agent done |
| `agent.skipped` | ✓ | `reason` | Not applicable / disabled |
| `agent.failed` | ✓ | `error` `{code, message}` | Agent error (analysis may continue) |
| `fusion.completed` | – | `Verdict` | Fusion done |
| `judge.token` | `judge` | `text` | Streaming report text delta |
| `judge.completed` | `judge` | `backend`, `model`, `key_evidence`, `caveats` | Judge done |
| `analysis.completed` | – | `verdict`, `duration_ms` | Terminal |
| `analysis.failed` | – | `error` `{code, message}` | Terminal |
| `analysis.cancelled` | – | – | Terminal |

Agent error codes: `AGENT_TIMEOUT`, `DETECTOR_ERROR`, `NO_FACES`, `NO_AUDIO`, `LLM_UNAVAILABLE` (Judge falls back to rules; not fatal).

## 7. Example session

```bash
curl -F "file=@clip.mp4" http://localhost:8000/api/v1/media
# → {"id":"med_01J9...","kind":"video",...}

curl -X POST http://localhost:8000/api/v1/analyses \
  -H "Content-Type: application/json" \
  -d '{"media_id":"med_01J9..."}'
# → 202 {"id":"ana_01J9...","status":"queued",...}

curl -N http://localhost:8000/api/v1/analyses/ana_01J9.../events
# → stream of events until analysis.completed
```
