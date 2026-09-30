# Phase 4 — War Room & Results

| | |
|---|---|
| **Weeks** | Weeks 4–5 (2026-10-26 → 11-06) |
| **Owner** | G (H pairs on SSE edge cases) |
| **Depends on** | Phase 3 (shell, client, components); Phase 1 stub pipeline (P1-09) |
| **Runs in parallel with** | Phase 2 — switch from stub to real pipeline when P2-07 merges |
| **Unblocks** | Phase 5, and the demo itself |
| **Release** | `v0.4.0` |

## Goal

Build the signature experience: the **live war room** where four agents visibly work in parallel, the verdict reveal, and a rich results view (Summary, Visual, Temporal, Audio, Metadata, Raw) that links every claim to evidence. It must survive refreshes, disconnects, failures, and every mock scenario.

## Inputs

- [Appflow.md §3–4, §7](../Appflow.md#3-the-war-room-analysesid-while-running), [Design.md §5–6](../Design.md#5-motion), [API.md §6](../API.md#6-server-sent-events)
- Stub pipeline (early) → real pipeline (late) from H

## Deliverables

- `lib/sse/useAnalysisStream.ts` + `stores/warRoomStore.ts`
- War-room layout with pipeline stepper, agent grid, live timeline, log, Judge panel
- Verdict reveal animation and transition to results
- Results tabs and all components: `EvidenceTimeline`, `HeatmapViewer`, `SpectrumViewer`, `WaveformPlayer`, `ReportMarkdown`, frame grid
- Actions: cancel, re-run, delete, copy link, download PDF
- Every error/edge state in Appflow §7

## Tasks

### P4-01 · Stream hook and store

- `useAnalysisStream(id)`: `new EventSource(url)` (with `?access_token=` via the Next proxy when auth is on); listen to each event type; parse each payload with a zod discriminated union mirroring [API.md §6](../API.md#6-server-sent-events); unknown/invalid events are logged and ignored (F-04).
- `warRoomStore` (Zustand + immer): `status`, `stage`, `progress`, `agents[name]` (status, progress, logs[], frameScores[], findings[], artifacts[], score, confidence, backend), `verdict`, `judgeText`, `lastSeq`, `connection` (`connecting | open | reconnecting | closed`).
- **Snapshot + stream merge:** hydrate from `GET /analyses/{id}` first; ignore events with `seq ≤ lastSeq`.
- Batch `judge.token` and `agent.frame_score` updates with `requestAnimationFrame` to keep renders ≤ 1 per frame.
- On a terminal event: close the stream, invalidate `['analysis', id]`, let the persisted result replace streamed state.

**Accept:** F-03, F-04 pass; a refresh mid-run restores the exact state (E-02).

### P4-02 · War-room layout

Implement the wireframe in [Appflow.md §3](../Appflow.md#3-the-war-room-analysesid-while-running):

- Header: filename, meta, MOCK badge when any agent is mock, Cancel.
- Left: media player (video/image/audio) with a scan-line overlay while agents run.
- Right: `PipelineStepper` (Preprocess → Agents → Fusion → Judge), 2×2 `AgentCard` grid wired to the store.
- Bottom: live `EvidenceTimeline` (bars grow as frame scores arrive) and the agent log (monospace, auto-scroll unless the user scrolled up, colour-coded by agent).
- "Reconnecting…" banner bound to `connection`.
- `aria-live="polite"` region announcing agent completions and the verdict.

### P4-03 · Judge panel

When the Judge starts, its card expands into a panel; `judgeText` renders progressively as Markdown (`react-markdown` + `remark-gfm`, raw HTML disabled, links sanitized) with a blinking caret. Finding IDs mentioned in the text (`fnd_…`) become chips linking to the evidence.

### P4-04 · Verdict reveal

On `fusion.completed`: gauge springs to p, arc fills with the verdict colour, label stamps in, one pulse ring (≤ 1.2 s total, reduced-motion → instant). On `analysis.completed`: after a 600 ms beat, crossfade the war room into the results layout (same route, no reload).

### P4-05 · Summary tab

Large gauge; confidence; `AgreementIndicator`; per-agent contribution bars (from `verdict.contributions`); the Judge report; key-evidence cards (thumbnail, title, agent, time → click jumps to the right tab and timestamp); caveats; backend badges for every agent.

### P4-06 · Visual tab

- Suspicious-frame grid (top-k by score) → click opens `HeatmapViewer` (crop/full-frame toggle, opacity slider, legend, "Mock saliency" watermark when mock).
- Per-frame score chart (Recharts area, score-scale gradient, threshold lines at 0.35/0.65), synced cursor with the player.
- Findings list filtered to Visual.

### P4-07 · Temporal tab

- `EvidenceTimeline` with one lane per agent + finding segments, two-way synced with the `<video>` (click to seek, `timeupdate` moves the playhead, ←/→ steps sampled frames).
- `SpectrumViewer`: FFT image + radial profile chart vs. a reference band; HF region shaded; the HF energy ratio as a stat.
- Flow-map artifact viewer; SSIM curve with flicker segments highlighted.
- For images: spectrum only.

### P4-08 · Audio tab

`WaveformPlayer` (wavesurfer.js + Regions plugin, dynamically imported) with suspicious segments; spectrogram image aligned to the same time axis; findings list. Shown only when the Audio agent ran.

### P4-09 · Metadata and Raw tabs

Metadata: container/codec table, EXIF key/values, C2PA present/absent, metadata findings. Raw: collapsible, syntax-highlighted JSON of the full analysis with a copy button.

### P4-10 · Actions and error states

Cancel (confirm dialog), Re-run (`POST /rerun` → navigate), Delete (confirm → back to History), Copy link, Download PDF (`/report.pdf`). Implement every row of [Appflow.md §7](../Appflow.md#7-error-and-edge-flows): failed agent card, all-failed error panel with request ID, `RULES` Judge badge tooltip, 404 page, API-unreachable banner.

### P4-11 · Responsive and reduced-motion pass

< 1024 px: player above the grid; < 640 px: single-column cards, log in a bottom drawer, timeline scrolls horizontally inside its own container (no page scroll). Verify reduced motion removes scan lines and pulses.

## Testing

- Store reducer tests: feed recorded event sequences for all five scenarios → assert final state (record them once from the real API into `tests/fixtures/events/*.json`).
- Component tests: timeline sync logic (time ↔ x), heatmap opacity, Judge chip linking.
- E2E (with H): E-01, E-02, E-03, E-04, E-09.

## Exit criteria

- [ ] All five mock scenarios render correctly live **and** after a refresh
- [ ] Mid-analysis refresh restores the war room exactly (E-02)
- [ ] Timeline, player, charts, and heatmaps stay in sync
- [ ] Every Appflow §7 error state is implemented
- [ ] 60 fps timeline scrub with 64 frames; no jank during Judge streaming
- [ ] Works at 375 px; reduced motion respected; axe clean on the results page
- [ ] `v0.4.0` tagged

## End-of-phase demo

Full run of the demo video in the browser: agents light up in parallel, the log streams, the timeline fills, the verdict reveals, the Judge types its report, the page becomes results; scrub the timeline and watch the heatmap follow. Refresh mid-run to show recovery. Run the `agent_fail` scenario to show graceful degradation.

## Risks

| Risk | Mitigation |
|---|---|
| Render storms from high-frequency events | rAF batching, memoized selectors, `React.memo` on cards |
| Stub vs. real event drift | Both emit the same catalogue; zod parsing surfaces drift immediately |
| Video sync bugs with variable frame rate | Always sync by `timestamp_ms`, never by frame index × fps |

## Handoff

- **To Phase 5:** shared components (gauge, timeline, cards) reused by dashboard and history.
- **To Phase 6:** E2E flows ready to automate.

## References

[Appflow.md](../Appflow.md) · [Design.md](../Design.md) · [API.md §6](../API.md#6-server-sent-events) · [TestPlan.md](../TestPlan.md)
