# Phase 0 — Foundations

| | |
|---|---|
| **Weeks** | Week 1 (2026-10-05 → 10-09) |
| **Owners** | H (backend, CI, repo), G (frontend, tokens, demo set), H+G (contract, datasets) |
| **Depends on** | Nothing — this is the start |
| **Unblocks** | Phase 1 (H) and Phase 3 (G), which run in parallel |
| **Release** | none (repo reaches a runnable skeleton) |

## Goal

By Friday, both of us can clone the repo on our own Windows laptop, run the API and the web app, open a PR, and watch CI go green. The API contract is agreed and frozen, and the slow external requests (datasets, API key) are already in motion.

## Why this phase matters

Everything later assumes a working toolchain, CI that catches mistakes, and a contract both sides trust. A day lost here saves a week of "works on my machine" and API mismatches later.

## Inputs

- Docs in `docs/` (read [Rules.md](../Rules.md), [TechSpecifications.md §3](../TechSpecifications.md#3-repository-layout), [Setup.md](../Setup.md) first)
- Prerequisites from [Setup.md §1](../Setup.md#1-prerequisites) installed on both laptops
- Repo moved **out of OneDrive** (e.g. `C:\dev\UML501-MACHINE-LEARNING`)

## Deliverables

- Monorepo skeleton: `apps/api`, `apps/web`, `ml/`, `data/` (gitignored), `docs/`
- `GET /api/v1/health` returning `{"status":"ok"}`
- Next.js app rendering the tokenized shell in dark and light themes
- CI workflow running lint, typecheck, and tests for both apps
- Branch protection, PR template, labels
- Frozen API contract v1 (sign-off recorded in [Decisions.md](../Decisions.md) or the PR)
- Dataset access requests submitted; demo set manifest started

## Tasks

### P0-01 · Monorepo structure · H

1. Create folders: `apps/api`, `apps/web`, `ml/{configs,scripts,splits,notebooks}`, and `scripts/` at the repo root for cross-cutting helpers. `data/` stays local only — it's gitignored, so nothing inside it is ever committed.
2. Add `.editorconfig` (LF endings, 2 spaces for TS/JSON/MD, 4 for Python) and `.gitattributes` (`* text=auto eol=lf`) so Windows checkouts don't flip line endings.
3. Commit: `chore: scaffold monorepo folders and editor config`.

**Accept:** a fresh clone shows the tree in [TechSpecifications.md §3](../TechSpecifications.md#3-repository-layout); `git status` is clean after checkout on Windows.

### P0-02 · API skeleton · H

1. `cd apps/api` → `uv init --package --name aletheia-api --python 3.12`.
2. `uv add "fastapi[standard]" pydantic-settings structlog` and `uv add --dev ruff pyright pytest pytest-asyncio httpx pytest-cov`.
3. Create `app/main.py` with an app factory (`create_app()`), `app/config.py` (`Settings` with `APP_ENV`, `CORS_ORIGINS`), and `app/api/v1/health.py` exposing `GET /api/v1/health`.
4. Configure Ruff (line length 100, rules `E,F,I,UP,B,SIM,ASYNC`) and Pyright (`strict` for `app/`) in `pyproject.toml`.
5. Test: `tests/test_health.py` using `httpx.AsyncClient` with `ASGITransport`.
6. Add `.env.example` with every variable from [Setup.md §4](../Setup.md#4-environment-variables) (placeholders only).

**Accept:** `uv run fastapi dev app/main.py` serves `/docs`; `uv run pytest`, `uv run ruff check .`, `uv run pyright` all pass.

### P0-03 · Web skeleton · G

1. `cd apps` → `pnpm create next-app web --ts --tailwind --eslint --app --src-dir --import-alias "@/*"`.
2. `pnpm dlx shadcn@latest init` (base colour neutral, CSS variables on).
3. Add Prettier (+ `prettier-plugin-tailwindcss`), Vitest + Testing Library + jsdom, and scripts: `dev`, `build`, `lint`, `format`, `typecheck` (`tsc --noEmit`), `test`, `gen:api` (placeholder until Phase 3).
4. Enable `"strict": true` and `"noUncheckedIndexedAccess": true` in `tsconfig.json`.
5. One smoke test (`src/app/page.test.tsx`) that renders the home page.

**Accept:** `pnpm dev` serves http://localhost:3000; `pnpm lint`, `pnpm typecheck`, `pnpm test`, `pnpm build` pass.

### P0-04 · CI · H

`.github/workflows/ci.yml` with two jobs on `pull_request` and `push` to `main`:

- **api:** checkout → `astral-sh/setup-uv` → install ffmpeg (`sudo apt-get install -y ffmpeg`) → `uv sync --frozen` → ruff check, ruff format --check, pyright, pytest with coverage.
- **web:** checkout → `pnpm/action-setup` → `actions/setup-node` (Node 22, pnpm cache) → `pnpm install --frozen-lockfile` → lint, typecheck, test, build.

Add a `concurrency` group so new pushes cancel old runs.

**Accept:** a PR shows both checks and they pass.

### P0-05 · Repo settings · H

- Protect `main`: PR required, 1 approval, required checks `api` and `web`, squash merge only, no force push.
- Enable secret scanning + push protection and Dependabot (`.github/dependabot.yml` for `uv`, `npm`, `github-actions`, weekly).
- Add `.github/pull_request_template.md` from [Contributing.md §4](../Contributing.md#4-pull-requests) and create the labels from [Contributing.md §8](../Contributing.md#8-labels).
- Add Girikshit as a collaborator with write access.

**Accept:** a direct push to `main` is rejected; a PR shows the template.

> After this task, all changes go through PRs (Rules R-39).

### P0-06 · Setup verified on both laptops · H+G

Each of us follows [Setup.md](../Setup.md) from a fresh clone and fixes anything wrong in the doc in the same PR.

**Accept:** both laptops run both apps; Setup.md updated with any Windows gotchas found.

### P0-07 · Freeze API contract v1 · H+G

1. Read [API.md](../API.md) and [Schema.md](../Schema.md) together (1 hour).
2. Walk through each screen in [Appflow.md](../Appflow.md) and confirm every piece of data it shows comes from an endpoint.
3. Record changes as a PR to API.md; merge it; note "Contract v1 frozen" in [Changelog.md](../Changelog.md).

**Accept:** no open questions about field names or event types; later changes need a PR that edits API.md first (Rules R-1).

### P0-08 · Design tokens, fonts, theme · G

1. Put every token from [Design.md §2–5](../Design.md#2-color-tokens) into `src/styles/globals.css` as CSS variables for dark (default) and light, mapped via Tailwind v4 `@theme`.
2. Load Geist Sans, Geist Mono, and Instrument Serif via `next/font`.
3. Theme toggle using `next-themes` (`class` strategy, dark default, no flash on load).
4. A temporary `/dev/tokens` page that shows swatches, type scale, radii — used to check both themes.

**Accept:** swatches match Design.md; text contrast ≥ 4.5:1 in both themes (check with the browser's accessibility panel).

### P0-09 · Submit dataset access requests · H+G

- FaceForensics++: fill in the official Google Form linked from the FaceForensics GitHub README; you receive a download script by email.
- Celeb-DF v2: request form linked from the Celeb-DF GitHub page.
- DFDC: Kaggle competition data (accept rules on Kaggle; we'll only download a subset later).
- ASVspoof 2019 LA: public download from the University of Edinburgh DataShare.

Record each request date in [Tracker.md → External dependencies](../Tracker.md#external-dependencies).

**Accept:** all four rows show a date.

### P0-10 · Demo set · G

1. Collect 6–10 clips: our own real recordings (both of us consent), plus clearly licensed public deepfake examples. **No dataset clips** in the public demo ([Security.md §6](../Security.md#6-privacy-and-data-handling)).
2. Store them locally in `data/demo/` (not committed) and commit `scripts/demo_manifest.example.json` describing the format: `{file, sha256, kind, expected_label, source_url, license, notes}`.
3. Share the actual files between us via a private drive.

**Accept:** manifest covers at least: real image, fake image, real video with audio, fake video, audio-only clip, a video without audio.

## Testing in this phase

- `tests/test_health.py` (API) and a page smoke test (web) — enough to prove CI works.
- Deliberately open a PR with a lint error to confirm CI blocks it, then fix it.

## Exit criteria

- [ ] CI green on `main` for both apps
- [ ] `GET /api/v1/health` → 200 on both laptops
- [ ] Web shell renders in both themes with the real tokens and fonts
- [ ] `main` is protected; PR template and labels exist
- [ ] API contract v1 frozen
- [ ] Dataset requests submitted (dates in Tracker)
- [ ] Demo set manifest format committed; clips collected
- [ ] [OpenQuestions](../OpenQuestions.md) Q-01, Q-02, Q-03, Q-04, Q-10 answered or have a date

## End-of-phase demo (Friday, 10 min)

1. Fresh clone on the other person's laptop → both apps run.
2. Open a PR with a failing test → CI red → fix → green → merge.
3. Toggle themes on `/dev/tokens`.

## Risks

| Risk | Mitigation |
|---|---|
| Windows path/line-ending issues | `.gitattributes`, repo outside OneDrive, test on both laptops |
| ffmpeg not on PATH | Readiness check in Phase 1 will flag it; Setup.md troubleshooting |
| Contract debates drag on | Timebox P0-07 to 2 hours; record leftovers in OpenQuestions |

## Handoff

- **To Phase 1 (H):** working API skeleton, settings, CI.
- **To Phase 3 (G):** working web skeleton, tokens, fonts, theme toggle.
- Both: frozen contract in API.md.

## References

[ImplementationPlan.md](../ImplementationPlan.md#phase-0--foundations-week-1) · [Setup.md](../Setup.md) · [Contributing.md](../Contributing.md) · [Rules.md](../Rules.md)
