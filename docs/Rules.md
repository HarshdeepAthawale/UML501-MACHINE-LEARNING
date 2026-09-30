# Engineering Rules

These rules are non-negotiable for both of us **and for any AI coding assistant** working in this repo (Claude Code, Copilot, Cursor, etc.). If a rule gets in the way, change it here in a PR — don't quietly break it.

## 1. Contract first

- **R-1** [API.md](API.md) is the contract. Change the Pydantic schema and API.md in the same PR; then run `pnpm gen:api` and commit the regenerated types.
- **R-2** The frontend never hand-writes API types. It imports from `lib/api/generated/`.
- **R-3** Schema changes go through an Alembic migration and an update to [Schema.md](Schema.md).
- **R-4** New SSE event types are added to the table in [API.md §6](API.md#6-server-sent-events) and to the zod union in the frontend in the same PR.

<a id="r-honesty"></a>
## 2. Honesty (ML integrity)

- **R-5** Every agent result carries `backend` and `backend_version`. The UI **must** show a MOCK badge for `backend=mock`, and the analysis header must show one when `is_mock` is true. No flag, env var, or CSS may hide it.
- **R-6** Never present mock, synthetic, or paper-reported numbers as our results. Leaderboard rows have a `source` (`ours | paper | placeholder`) and the UI labels it.
- **R-7** Paper numbers need a citation in `notes`.
- **R-8** Report test metrics only on splits the model never saw during training or tuning. Record the split files in `ml/splits/` and commit them.
- **R-9** Report in-domain and cross-dataset metrics separately. Never average them into a single headline number.
- **R-10** The Judge LLM explains the fused verdict; it never sets or changes the number ([ADR-007](Decisions.md#adr-007)).

## 3. Code quality

### Python (`apps/api`, `ml/`)

- **R-11** Python 3.12, fully type-annotated. Pyright strict on `app/`; no `# type: ignore` without a reason comment.
- **R-12** Ruff format + lint must pass. Line length 100.
- **R-13** Layering: routers → services → repositories / pipeline. Routers contain no business logic; services never import FastAPI.
- **R-14** Async all the way in request handlers. CPU-heavy work (decoding, FFT, optical flow, model inference) runs in `asyncio.to_thread` or a process pool — never block the event loop.
- **R-15** External processes (`ffmpeg`, `ffprobe`) are called with an argument list, never `shell=True`, and always with a timeout.
- **R-16** No `print`; use the structlog logger. Never log file contents, API keys, or full request bodies.

### TypeScript (`apps/web`)

- **R-17** `strict: true`. No `any` (use `unknown` + zod). No non-null assertions (`!`) without a comment.
- **R-18** Components use semantic design tokens only — no raw hex colours or arbitrary pixel values outside the token set ([Design.md](Design.md)).
- **R-19** Server state lives in TanStack Query; streaming war-room state in Zustand; everything else is local state. No global context for server data.
- **R-20** Every data view implements loading, empty, error, and partial states ([Design.md §8](Design.md#8-states-checklist-every-data-view)).
- **R-21** Heavy libraries (three, wavesurfer, recharts) are dynamically imported.
- **R-22** Every interactive element is keyboard-reachable and has an accessible name.

### Both

- **R-23** Files stay under ~400 lines; functions under ~60. Split before you exceed them.
- **R-24** Names say what things are: `fake_probability`, not `prob` or `p`. Units in names: `duration_ms`, `size_bytes`.
- **R-25** Comments explain *why*, not *what*. Link the ADR when a decision isn't obvious.
- **R-26** No dead code, commented-out blocks, or TODOs without an owner: `TODO(H): …` or `TODO(G): …`.

## 4. Errors and resilience

- **R-27** All API errors use the problem+json shape and a code from [API.md §2](API.md#2-errors). Don't invent codes without adding them there.
- **R-28** An agent failure must not crash the analysis (see [TechSpecifications.md §4.4](TechSpecifications.md#44-analysis-pipeline-langgraph)).
- **R-29** Every external call (Claude API, ffmpeg, model inference) has a timeout and a defined fallback.
- **R-30** User input is untrusted: validate size, type (magic bytes), and duration server-side even though the client also checks.

## 5. Security and secrets

- **R-31** Secrets only live in `.env` files, which are gitignored. Commit `.env.example` with placeholders.
- **R-32** The Anthropic API key is used by the backend only. It never reaches the browser, logs, or the database.
- **R-33** Never commit datasets, model weights over 50 MB, uploaded media, or `data/`. FaceForensics++ and Celeb-DF are research-only: do not redistribute them or put their frames in the public repo or a public deployment.
- **R-34** Follow the checklist in [Security.md](Security.md) for anything touching uploads, the file system, or the LLM prompt.

## 6. Testing

- **R-35** New backend logic ships with tests. Bug fixes ship with a test that fails before the fix.
- **R-36** Coverage targets: backend `services/`, `pipeline/`, `events/` ≥ 80%; frontend `lib/` and `components/forensics/` ≥ 60%.
- **R-37** Tests never call the real Claude API or need a GPU. Use the stub client and mock detectors.
- **R-38** CI must be green to merge. No skipping hooks or disabling tests to get a merge through.

## 7. Git

- **R-39** No direct pushes to `main` after Phase 0 setup; everything goes through a PR reviewed by the other person ([Contributing.md](Contributing.md)).
- **R-40** Conventional Commits, small focused commits, one concern per PR.

## 8. Rules for AI assistants specifically

- **R-41** Read the relevant docs (this file, [TechSpecifications.md](TechSpecifications.md), [API.md](API.md), [Schema.md](Schema.md)) before writing code.
- **R-42** Don't add dependencies without saying so in the PR description; prefer what's already in the stack table.
- **R-43** Don't change the API contract, schema, or design tokens as a side effect of another task.
- **R-44** Don't fabricate metrics, dataset results, citations, or test output. If something wasn't run, say so.
- **R-45** Update [Tracker.md](Tracker.md) and, for user-visible changes, [Changelog.md](Changelog.md) in the same PR.
