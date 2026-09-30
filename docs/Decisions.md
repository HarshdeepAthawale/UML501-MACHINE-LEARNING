# Architecture Decision Records

Short ADRs: what we chose, why, and what we rejected. Status: `proposed` → `accepted` → (`superseded by ADR-x`). Add new ADRs at the bottom; never rewrite an accepted one — supersede it.

Template:

```
## ADR-NNN — Title
Status · Date · Deciders
Context — what forced a decision
Decision — what we chose
Alternatives — what we rejected and why
Consequences — what this makes easier / harder
```

---

<a id="adr-001"></a>
## ADR-001 — Product first on mock detectors, ML second

**Status:** accepted · 2026-09-30 · H, G

**Context:** We want a polished dashboard and 4 agents, and 96–97% accuracy. Training depends on dataset approvals and GPU time we don't control yet.

**Decision:** Build the complete product (backend, agents, UI) against a `Detector` interface with deterministic mock implementations and real signal processing. Swap in trained models in Phase 7.

**Alternatives:** (a) ML first, UI last — risks an ugly rushed UI and blocks one teammate. (b) Build both at once — splits focus and makes integration late and painful.

**Consequences:** Both teammates are productive from week 1; the UI is exercised on every edge case via mock scenarios. Risk: mock output mistaken for real — handled by [Rules R-5/R-6](Rules.md#r-honesty).

<a id="adr-002"></a>
## ADR-002 — FastAPI (Python) backend

**Status:** accepted · 2026-09-30

**Context:** The ML stack (PyTorch, OpenCV, librosa) is Python. The backend must host the models later.

**Decision:** FastAPI + Pydantic v2.

**Alternatives:** Node/NestJS backend + a separate Python inference service — two services, two deploys, more glue. Django — heavier, weaker async and OpenAPI story. Flask — no native async or schema-driven docs.

**Consequences:** One backend language shared with ML; automatic OpenAPI for frontend type generation.

<a id="adr-003"></a>
## ADR-003 — Next.js + Tailwind + shadcn/ui frontend (not Streamlit/Gradio)

**Status:** accepted · 2026-09-30

**Context:** The dashboard has to impress: streaming war room, custom visualizations, 3D, precise motion.

**Decision:** Next.js (App Router) + TypeScript + Tailwind v4 + shadcn/ui + motion + react-three-fiber.

**Alternatives:** Streamlit / Gradio — fast to start but limited layout, animation, and real-time control; looks like every other ML demo. Plain Vite + React — fine, but Next gives routing, fonts, and image handling for free.

**Consequences:** More frontend work, but full control over the experience. A separate deploy for web and API.

<a id="adr-004"></a>
## ADR-004 — Detector interface + registry

**Status:** accepted · 2026-09-30

**Decision:** Agents never import a model directly. They call `registry.get(agent_name)` which returns a `Detector` chosen by the `DETECTOR_*` env var. Each result records `backend` and `backend_version`.

**Consequences:** Swapping mock → GenD → ensemble is a config change; A/B comparisons on the same media are trivial; the UI can badge mock results.

<a id="adr-005"></a>
## ADR-005 — SQLite in dev, Postgres for deployment, via SQLAlchemy + Alembic

**Status:** accepted · 2026-09-30

**Decision:** SQLite (zero setup on Windows) for local development; Postgres when deployed. Portable types only (JSON, VARCHAR + CHECK instead of native enums).

**Alternatives:** Postgres everywhere via Docker — more setup friction for both laptops. MongoDB — our data is relational (analysis → runs → findings).

**Consequences:** Migrations must be tested on both; SQLite's single-writer limit is fine at our concurrency.

<a id="adr-006"></a>
## ADR-006 — In-process asyncio job runner (for now)

**Status:** accepted · 2026-09-30

**Context:** Analyses take 5–60 s and must not block HTTP requests. We run one API process.

**Decision:** An `asyncio.Queue` with N worker tasks inside the API process, job state in the DB, recovery on startup. Behind a `JobQueue` protocol.

**Alternatives:** Celery + Redis — heavy for a single-machine demo, awkward on Windows. `arq` + Redis — good, and our planned upgrade path if we need several processes or a separate GPU worker.

**Consequences:** Simple to run and debug. One process means a crash kills in-flight jobs (they're marked `INTERRUPTED`). Revisit in Phase 7 if GPU inference moves to its own worker.

<a id="adr-007"></a>
## ADR-007 — Deterministic fusion; the LLM Judge explains but doesn't decide

**Status:** accepted · 2026-09-30

**Context:** An LLM could output the verdict directly, but it could hallucinate, drift between runs, or be manipulated through metadata text in the file.

**Decision:** The verdict is computed by weighted log-odds fusion ([TechSpecifications.md §4.7](TechSpecifications.md#47-fusion-the-math)). The Judge receives that verdict and writes the explanation, evidence list, and caveats.

**Alternatives:** LLM decides (less reproducible, weaker ML story, prompt-injection risk). Only rules, no LLM (less impressive explanations).

**Consequences:** Reproducible, testable numbers; the math is ours to explain in the viva; the Judge can't be talked into a different verdict.

<a id="adr-008"></a>
## ADR-008 — Claude as the Judge model, with a rules fallback

**Status:** accepted · 2026-09-30

**Decision:** The Judge uses the Anthropic Python SDK with `claude-opus-5-5` by default (`JUDGE_MODEL` is configurable), streaming output, and keyframes as image input. If there's no key, an API error, or a timeout, a template-based Judge (`backend=rules`) runs instead.

**Alternatives:** A local open VLM (e.g. Qwen-VL) — free per call but needs a GPU at inference time and gives weaker reports on a laptop. Another hosted LLM — no strong reason to switch.

**Consequences:** Costs money per analysis (one call each; cached system prompt; downscaled images). The demo keeps working offline thanks to the fallback. Open question on who provides the key: [Q-05](OpenQuestions.md#q-05).

<a id="adr-009"></a>
## ADR-009 — LangGraph for orchestration

**Status:** accepted · 2026-09-30

**Decision:** The analysis pipeline is a LangGraph `StateGraph` with a parallel fan-out to the three specialist agents and a fan-in to fusion → Judge.

**Alternatives:** Plain `asyncio.gather` — simpler, but we'd hand-roll state merging, routing, and the graph visual. CrewAI / AutoGen — built for chatty LLM-to-LLM agents; our specialists are tool-driven and mostly deterministic.

**Consequences:** A clear graph to show on `/about`; conditional routing by media kind; an extra dependency. If LangGraph gets in the way, the graph is small enough to replace with `asyncio` without touching agents.

<a id="adr-010"></a>
## ADR-010 — Server-Sent Events (not WebSockets) for live updates

**Status:** accepted · 2026-09-30

**Decision:** One-way server → browser updates over SSE, events persisted with sequence numbers for replay.

**Alternatives:** WebSockets — two-way isn't needed, and reconnection/replay is manual. Polling — laggy, and it wastes requests.

**Consequences:** Built-in browser reconnect with `Last-Event-ID`; works through most proxies. WebSockets are reserved for the webcam live mode.

<a id="adr-011"></a>
## ADR-011 — Monorepo with generated API types

**Status:** accepted · 2026-09-30

**Decision:** One repo (`apps/api`, `apps/web`, `ml/`, `docs/`). The frontend generates TypeScript types from FastAPI's OpenAPI schema with `openapi-typescript`.

**Consequences:** Contract drift shows up as a TypeScript error; one PR can change both sides.

<a id="adr-012"></a>
## ADR-012 — Real signal processing in the specialist agents from day one

**Status:** accepted · 2026-09-30

**Decision:** Even before trained models exist, the Temporal & Frequency agent computes real FFT spectra, SSIM flicker, optical flow and bbox jitter; the Audio agent computes real spectrograms and energy features. Only the final *scores* are mocked.

**Consequences:** The visualizations show genuine measurements of the uploaded file; these features become the inputs of the classical temporal classifier in Phase 7 (a good ML-course ablation: hand-crafted features vs. foundation-model features).

<a id="adr-013"></a>
## ADR-013 — Main visual model candidate: GenD, with Effort as the second model

**Status:** proposed · 2026-09-30 (confirm at the start of Phase 7)

**Context:** Research (2025–2026) shows that lightly tuning a strong foundation model generalizes best across datasets.

**Decision:** GenD (CLIP ViT-L/14, LayerNorm-only tuning with alignment + uniformity losses), fine-tuned on FF++ c23. Effort (SVD residual tuning) as the second model; EfficientNet-B4 and Xception as baselines; stacked ensemble for the final score.

**Consequences:** Fits a free T4 GPU; public checkpoints exist for a sanity check. To be revisited if dataset access or compute changes.
