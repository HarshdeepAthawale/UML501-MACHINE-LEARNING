# Phase 3 — Frontend Foundation

| | |
|---|---|
| **Weeks** | Weeks 2–3 (2026-10-12 → 10-23) |
| **Owner** | G |
| **Depends on** | Phase 0 (web skeleton, tokens, frozen contract) |
| **Runs in parallel with** | Phase 1 (backend core) |
| **Unblocks** | Phase 4 (war room), Phase 5 (pages) |
| **Release** | `v0.3.0` |

## Goal

The app shell, the design system in code, a typed API client, and a real upload flow: a user drops a file on `/analyze`, sees upload progress and a preview, starts an analysis, and lands on `/analyses/[id]`. Plus the first version of every forensic component, built in isolation.

## Inputs

- [Design.md](../Design.md) (tokens, components, motion), [Appflow.md §1–2](../Appflow.md#1-sitemap)
- [API.md](../API.md); from mid-Phase 1, a running API (`openapi.json`)
- Tokens, fonts, theme toggle from P0-08

## Deliverables

- Layout shell with sidebar, top bar, API status, route skeletons for every sitemap route
- Background system and restyled shadcn base components
- `pnpm gen:api` + typed client + TanStack Query setup + problem+json handling
- `/analyze` upload flow end to end
- `/dev/components` gallery with v1 of: VerdictGauge, AgentCard (5 states), StatTile, MockBadge, BackendBadge, FindingCard, ConfidenceBar, AgreementIndicator
- Landing page v1

## Frontend structure

```
src/
├── app/
│   ├── (marketing)/page.tsx            # landing, no sidebar
│   ├── (app)/layout.tsx                # sidebar + top bar shell
│   ├── (app)/dashboard/page.tsx        # skeleton (Phase 5)
│   ├── (app)/analyze/page.tsx
│   ├── (app)/analyses/[id]/page.tsx    # placeholder until Phase 4
│   ├── (app)/{history,models,explore,about,settings}/page.tsx   # skeletons
│   └── dev/components/page.tsx         # component gallery (dev only)
├── components/{ui,forensics,charts,layout}/
├── lib/api/{client.ts,queries.ts,errors.ts,generated/schema.d.ts}
├── lib/utils/{format.ts,score-scale.ts}
├── stores/prefsStore.ts
└── styles/globals.css
```

## Tasks

### P3-01 · App shell and routes

- `(app)` route group layout: collapsible sidebar (icons + labels; collapses to icons < 1280 px; becomes a sheet on mobile), top bar with page title, ⌘K trigger placeholder, `ApiStatusDot` (polls `/health/ready` every 30 s via Query), theme toggle.
- Skeleton pages for every route in [Appflow.md §1](../Appflow.md#1-sitemap) with correct titles (`metadata`) and an `EmptyState`.
- `not-found.tsx` and `error.tsx` styled per Design.md.

**Accept:** every sitemap route resolves; keyboard can reach every nav item; mobile sheet works at 375 px.

### P3-02 · Background and base components

- `Backdrop` component: `--bg`, 32 px grid (CSS background-image), SVG noise at 2%, optional aurora blobs (props) — used by landing and war room.
- Restyle shadcn primitives to tokens (Button variants: primary/secondary/ghost/destructive; Card with inner highlight; Input; Tabs; Tooltip; Dialog; Table; Slider; Switch; Skeleton; Sonner toasts).
- Motion helpers: `fadeUp`, `stagger`, spring presets from Design.md §5; a `useReducedMotion` wrapper that swaps them for crossfades.

**Accept:** no raw hex colours in components (lint rule or grep in CI); both themes look right on `/dev/tokens` and `/dev/components`.

### P3-03 · Typed API client

1. `pnpm add -D openapi-typescript` → script `gen:api`: `openapi-typescript $NEXT_PUBLIC_API_BASE_URL/../openapi.json -o src/lib/api/generated/schema.d.ts` (use the API origin + `/openapi.json`).
2. `client.ts`: `apiFetch<Path, Method>()` typed from `paths`; base URL from env; parses problem+json into an `ApiError` class (`code`, `status`, `detail`, `requestId`, `errors`).
3. `queries.ts`: TanStack Query hooks — `useMedia`, `useAnalysis`, `useAnalyses(filters)` (infinite), `useAgents`, `useReadiness`, `useCreateAnalysis`, `useDeleteAnalysis`, etc. Query keys centralised.
4. `QueryClientProvider` in the root layout; sensible defaults (`staleTime` 10 s, retry only on network/5xx).
5. CI contract check (with H): regenerate types from the committed `openapi.json` snapshot and fail on diff.

**Accept:** changing a field name in the API makes `pnpm typecheck` fail.

### P3-04 · `/analyze` upload flow

1. `UploadDropzone` (`react-dropzone`): accepts the allowlisted MIME types; client-side size limits per kind (from `/settings/public`, with defaults); rejects with an inline message.
2. Local preview immediately via `URL.createObjectURL` (video/image/audio element); revoke on unmount.
3. Upload with `XMLHttpRequest` (for `upload.onprogress`), abortable; show bytes, %, speed.
4. On success show `MediaMetaCard` (duration, resolution, fps, audio yes/no, short SHA-256); if the response was `200` (dedup), show "Already uploaded — view previous analyses".
5. "Advanced" options (collapsed): agent toggles disabled when not applicable (with a tooltip), frames slider 8–64, Judge switch, heatmaps switch; in development only, a scenario select.
6. **Run analysis** → `useCreateAnalysis` → `router.push(/analyses/{id})`.
7. "Try a sample" buttons (once the demo set is loaded server-side; hidden otherwise).
8. Notice under the button: "Keyframes are sent to Claude to write the report" when the Judge is on ([Security.md §6](../Security.md#6-privacy-and-data-handling)).

**Accept:** F-01, F-02, F-07 in [TestPlan.md](../TestPlan.md#frontend) pass; the full flow works against the Phase 1 API.

### P3-05 · Forensic components v1

Build in `components/forensics/`, show every state on `/dev/components`:

| Component | Must support |
|---|---|
| `VerdictGauge` | p ∈ [0,1], label, confidence ring, threshold ticks, sizes `sm`/`lg`, animated and reduced-motion modes |
| `AgentCard` | `queued`, `running` (progress, scan line), `completed` (score pill), `skipped` (reason), `failed` (error); backend badge; last log line; sparkline slot; finding chips |
| `StatTile` | label, value, delta, sparkline |
| `MockBadge`, `BackendBadge` | tooltips per Design.md |
| `FindingCard` | severity, time range, thumbnail, "jump to" action |
| `ConfidenceBar`, `AgreementIndicator` | |

Put the gauge geometry and score→colour mapping in pure functions (`lib/utils/score-scale.ts`, `gauge-geometry.ts`) so they're unit-testable.

**Accept:** F-05, F-06 pass; axe shows no serious violations on the gallery in both themes.

### P3-06 · Landing v1

Hero (Instrument Serif headline, subhead, CTAs "Analyze media" / "How it works"), a **static SVG poster** where the 3D face will go in Phase 5, the four agent cards, a three-step "how it works", footer with team names and course. Uses `Backdrop` with aurora.

**Accept:** Lighthouse Performance ≥ 90 on `/`.

### P3-07 · Tests

Vitest: score scale, gauge geometry, `ApiError` parsing, upload validation, AgentCard state rendering, MockBadge presence rule.

## Exit criteria

- [ ] Upload from `/analyze` to the real API works and lands on `/analyses/[id]`
- [ ] API types are generated, not hand-written; contract check runs in CI
- [ ] All forensic components v1 render every state in both themes
- [ ] No raw colours outside tokens; axe clean on `/`, `/analyze`, `/dev/components`
- [ ] Usable at 375 px
- [ ] `v0.3.0` tagged

## End-of-phase demo

Drop the demo video on `/analyze` → progress bar → preview and metadata card → toggle options → Run → land on the analysis page (still a placeholder). Then flip through `/dev/components` in both themes and with reduced motion on.

## Risks

| Risk | Mitigation |
|---|---|
| API not ready yet | Build against `openapi.json` from the contract and MSW mocks; switch to the real API once P1-04 lands |
| Over-polishing components before data exists | Timebox P3-05; states and correctness first, finesse in Phase 4 |
| Tailwind v4 / shadcn setup friction | Follow the shadcn Tailwind v4 guide; keep tokens as plain CSS variables |

## Handoff

- **To Phase 4:** shell, client, components, and `/analyses/[id]` route ready for the war room.
- **To Phase 5:** skeleton pages and query hooks.

## References

[Design.md](../Design.md) · [Appflow.md](../Appflow.md) · [API.md](../API.md) · [TechSpecifications.md §5](../TechSpecifications.md#5-frontend-design)
