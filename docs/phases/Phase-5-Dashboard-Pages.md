# Phase 5 — Dashboard & Secondary Pages

| | |
|---|---|
| **Weeks** | Weeks 6–7 (2026-11-09 → 11-20) |
| **Owners** | G (UI), H (APIs, seed data), H+G (About, Settings) |
| **Depends on** | Phases 2 and 4 |
| **Unblocks** | Phase 6 (hardening needs every route to exist) |
| **Release** | `v0.5.0` |

## Goal

Every route in the sitemap is real: an overview dashboard worth opening the demo with, a searchable history, the model leaderboard and 3D explorer shells (ready for Phase 7–8 data), an About page that doubles as our viva companion, Settings, the printable report, the landing hero in 3D, and a command palette.

## Inputs

- [Appflow.md §5](../Appflow.md#5-other-journeys), [Design.md §7](../Design.md#7-page-level-layouts), [API.md](../API.md) (metrics, models, settings)
- Components from Phases 3–4

## Deliverables

- APIs: `GET /metrics/overview`, `GET /models`, `/models/{id}`, `/models/{id}/embeddings`, `GET /settings/public`, `POST /admin/purge`
- Seed scripts: `scripts/seed_models.py`, `scripts/seed_demo_embeddings.py`, `scripts/load_demo_set.py`
- Pages: `/dashboard`, `/history`, `/models`, `/models/[id]`, `/explore`, `/about`, `/settings`, `/analyses/[id]/report`, landing 3D hero, ⌘K palette

## Tasks

### P5-01 · Metrics API + Dashboard (H: API, G: UI)

**API:** `GET /metrics/overview?range=7d|30d|all` — aggregate SQL over `analyses` and `agent_runs` (counts by status, verdict, kind; average durations; daily timeseries; per-agent success rate and average duration). Index-backed; < 50 ms on 1,000 analyses.

**UI** ([Design.md §7](../Design.md#7-page-level-layouts)):
- 4 `StatTile`s: analyses, fake rate, average processing time, agents healthy (x/4).
- Activity area chart (analyses/day with fake overlay) + range switch.
- Verdict donut (real / fake / uncertain with labels, not colour alone).
- Recent analyses (thumbnail, verdict chip, relative time) → results.
- Agent health strip (backend badge, avg duration, success rate).
- First-run empty state with "Analyze media" and "Try a sample".

### P5-02 · History (G)

Table: thumbnail, filename, kind, verdict chip, P(fake) (score-scale cell), confidence, duration, created. Filters (verdict, kind, status, date range), filename search (debounced 300 ms), sort, infinite scroll (`useInfiniteQuery` + cursor), multi-select bulk delete with confirmation. All filters mirrored in the URL query string (`nuqs` or `useSearchParams`). Row click → results. Skeleton rows while loading.

### P5-03 · Model registry seed + API (H)

- `seed_models.py` inserts:
  - `placeholder` rows for each planned model (GenD, Effort, EfficientNet-B4, Xception, temporal classifier, XLS-R + AASIST, ensemble) with empty metrics.
  - A few `paper` rows with **published** numbers and a full citation in `notes` (verify each number against the paper before adding — Rules R-7, R-44).
- `GET /models` (grouped by agent), `/models/{id}` (metrics + ROC), `/models/{id}/embeddings` (paginated points).
- `seed_demo_embeddings.py`: synthetic 3D clusters under `mdl_placeholder_demo`, flagged synthetic.

### P5-04 · Leaderboard UI (G)

Table grouped by agent: model, source badge (`OURS` / `PAPER` / `PENDING`), train set, trainable params, in-domain AUC/Acc, cross-dataset AUC. Select up to 3 rows → overlay ROC curves (Recharts line chart, diagonal chance line). Placeholder rows greyed with "Training pending (Phase 7)". Paper rows show a citation tooltip. Clear footnote: "In-domain and cross-dataset results are reported separately."

`/models/[id]`: model card layout (architecture summary, training setup, metrics table per dataset, ROC, confusion matrix slot, notes). Mostly placeholders until Phase 8.

### P5-05 · 3D Explorer (G)

`@react-three/fiber` + `drei` (`OrbitControls`, `Points` / instanced spheres). Model + dataset selectors; colour by label (teal/rose), shade by manipulation type; hover tooltip; 2D/3D toggle; "synthetic demo data" banner while only the placeholder set exists. Lazy-load the canvas; show a static fallback on low-end devices and with reduced motion (no auto-rotate).

### P5-06 · About page (H+G)

Our viva companion. Sections:
1. The problem (short).
2. Interactive pipeline diagram (nodes: Preprocess → Visual / Temporal / Audio → Fusion → Judge) — click a node for details. Use the Mermaid export from P2-07 as the source of truth for the structure.
3. One section per agent: what it looks at, the signal processing (FFT, SSIM, optical flow, mel spectrogram), what fools it.
4. The fusion math rendered with KaTeX (`react-katex` or `rehype-katex`), with a small interactive: sliders for three agent scores → live fused p.
5. Datasets and metrics (AUC, EER, ECE) in plain words.
6. Limitations, ethics, and the "not proof" disclaimer.

### P5-07 · Settings (H+G)

**API:** `GET /settings/public` (limits, judge enabled + model name — never the key —, retention days, `app_env`, detector backends); `POST /admin/purge` guarded per [API.md](../API.md#settings--admin-local-only).
**UI:** live readiness checks (from `/health/ready`), limits, Judge status, theme, reduced-motion override (stored in `prefsStore`), retention info, "Delete all data" with typed confirmation (dev only).

### P5-08 · Print report + PDF button (G)

`/analyses/[id]/report`: print-optimised layout (`@media print`, light theme forced, page breaks between sections) mirroring the PDF structure; "Download PDF" uses the server PDF from P2-12. Useful for screenshots in the final report.

### P5-09 · Landing 3D hero + live stats (G)

Replace the static poster with a point-cloud face (~6k points sampled from a face mesh or a depth map), slow rotation, a horizontal scanning light band; points in the band tint teal→rose over a "manipulated" region. Dynamic import with the SVG poster as fallback; pause when off-screen (`IntersectionObserver`); disabled with reduced motion. Below the fold: live stats from `/metrics/overview`.

**Accept:** Lighthouse Performance on `/` stays ≥ 90 (3D loads after LCP).

### P5-10 · Command palette (G)

`cmdk` dialog on ⌘/Ctrl+K: navigate to pages, "Analyze media", recent analyses (from the list query), toggle theme, open docs/About.

### P5-11 · Demo set loader (H)

`scripts/load_demo_set.py` reads `data/demo/manifest.json`, uploads each clip through the service layer with `expires_at = NULL`, and runs an analysis. Powers "Try a sample" and the Phase 6 offline fallback.

## Testing

- API: metrics aggregation on a seeded DB; models endpoints; purge guard (B-26-style).
- UI: history filter ↔ URL sync; leaderboard source badges; placeholder rows can't be mistaken for results.
- E2E: E-05, E-07.
- Lighthouse on `/dashboard` and `/`.

## Exit criteria

- [ ] Every sitemap route is functional with loading/empty/error states
- [ ] Lighthouse Performance ≥ 90, Accessibility ≥ 95 on `/dashboard`
- [ ] Leaderboard and explorer clearly label placeholder/synthetic/paper data
- [ ] About page explains every agent and the fusion math
- [ ] Demo set loads with one command
- [ ] `v0.5.0` tagged

## End-of-phase demo

Open on the landing page (3D hero) → Dashboard with real numbers from the demo set → History filtered to "fake" → open a result → About page: move the fusion sliders → Leaderboard with PENDING rows → ⌘K back to Analyze.

## Risks

| Risk | Mitigation |
|---|---|
| 3D hurts performance | Lazy load after LCP, pause off-screen, fallback poster |
| Paper numbers copied wrong | Each `paper` row reviewed by the other person against the PDF |
| Scope creep (more pages, more charts) | Only what's in Appflow; new ideas go to OpenQuestions |

## Handoff

- **To Phase 6:** a complete product to harden and deploy.
- **To Phase 8:** `model_registry` and `embedding_points` are ready to receive real exports.

## References

[Appflow.md §5](../Appflow.md#5-other-journeys) · [Design.md](../Design.md) · [API.md](../API.md) · [Rules.md §2](../Rules.md#r-honesty)
