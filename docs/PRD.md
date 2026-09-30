# PRD — Aletheia: Multi-Agent Deepfake Forensics

| Field | Value |
|---|---|
| Status | Draft v0.1 |
| Owners | Harshdeep Athawale, Girikshit Gupta |
| Course | UML501 — Machine Learning |
| Last updated | 2026-09-30 |

## 1. Problem

Deepfakes (face swaps, reenactments, lip-sync, fully synthetic faces, cloned voices) are now cheap to make and hard to spot. Most academic detectors are a single black-box classifier that outputs a number with no explanation, and they are known to lose accuracy on data unlike their training set.

People evaluating a suspicious video need three things a single score does not give them:

1. **A verdict they can trust** — calibrated probability, not a raw logit.
2. **Evidence** — *where* in the frame and *when* in the timeline the manipulation shows up, across both video and audio.
3. **An explanation in plain language** — why the system believes what it believes, including when its signals disagree.

## 2. Vision

Upload an image, video, or audio clip. Four specialist AI agents examine it in parallel — **Visual**, **Temporal & Frequency**, **Audio**, and a **Judge** that weighs their evidence — and the dashboard shows their work live, ending in a verdict, an evidence timeline, heatmaps, and a downloadable forensic report.

## 3. Users

| Persona | Who | What they need |
|---|---|---|
| **Evaluator** (primary) | Our professor / viva panel | A polished, reliable live demo; visible ML rigor (metrics, baselines, math); honest reporting of limits |
| **Analyst** | A student or journalist checking a clip | Fast upload → clear verdict → evidence they can point at → exportable report |
| **Builder** | Us (Harshdeep, Girikshit) | Swappable detectors, reproducible runs, a leaderboard comparing models |

## 4. Goals and success metrics

### 4.1 Product goals (Phases 0–6, before real ML)

| ID | Goal | Metric / acceptance |
|---|---|---|
| G-1 | End-to-end analysis works for image, video, and audio | 100% of the manual test cases in [TestPlan.md](TestPlan.md) pass |
| G-2 | Live agent "war room" | First SSE event reaches the UI ≤ 1 s after job start; every agent's state change is visible |
| G-3 | Fast enough to demo | 30 s 1080p video completes in ≤ 45 s on a laptop CPU with mock detectors |
| G-4 | Beautiful, responsive UI | Lighthouse Performance ≥ 90, Accessibility ≥ 95 on `/dashboard`; usable at 375 px width |
| G-5 | Reliable demo | 20 consecutive demo-set analyses with zero crashes; clear error states for bad input |
| G-6 | Honest by construction | Every result produced by a mock detector shows a visible **MOCK** badge (see [Rules.md](Rules.md#r-honesty)) |

### 4.2 ML goals (Phase 7+, targets — not promises)

| ID | Goal | Target |
|---|---|---|
| M-1 | In-domain accuracy (FF++ c23 test split, frame-level → video-level) | **≥ 96% accuracy, ≥ 0.98 AUC** |
| M-2 | Cross-dataset generalization (train FF++, test Celeb-DF v2) | ≥ 0.90 AUC (reported honestly whatever it is) |
| M-3 | Beat baselines | Main model beats Xception and EfficientNet-B4 on both M-1 and M-2 |
| M-4 | Calibration | ECE ≤ 0.05 after temperature scaling |
| M-5 | Audio | EER ≤ 5% on ASVspoof 2019 LA eval; report In-the-Wild EER |

## 5. Features

Priority: **P0** = required for the demo, **P1** = strongly wanted, **P2** = stretch.

| ID | Feature | Priority | Phase |
|---|---|---|---|
| F-01 | Upload image / video / audio (drag-and-drop, validation, progress bar) | P0 | 1, 3 |
| F-02 | Analysis job with 4 agents orchestrated as a graph | P0 | 2 |
| F-03 | Live war room: per-agent status, streamed reasoning, progress | P0 | 4 |
| F-04 | Verdict: real / fake / uncertain + calibrated probability + confidence | P0 | 2, 4 |
| F-05 | Frame-score timeline for video (fake probability per sampled frame) | P0 | 4 |
| F-06 | Heatmap overlay on keyframes (Grad-CAM once ML lands) | P0 | 4 |
| F-07 | Frequency spectrum view (real FFT of frames, computed from day one) | P1 | 4 |
| F-08 | Audio waveform + spectrogram + suspicious segments | P1 | 4 |
| F-09 | Judge report: plain-English forensic write-up, streamed token-by-token | P0 | 2, 4 |
| F-10 | PDF report export | P1 | 5 |
| F-11 | History: list, search, filter, delete analyses | P0 | 5 |
| F-12 | Overview dashboard: KPIs, verdict distribution, recent activity, agent health | P0 | 5 |
| F-13 | Model leaderboard: our models vs baselines, ROC curves, per-dataset metrics | P1 | 5 (shell), 7–8 (real data) |
| F-14 | 3D embedding explorer (real vs fake clusters) | P2 | 5 (shell), 8 (real data) |
| F-15 | "How it works" page: pipeline diagram + the math, for the viva | P1 | 5 |
| F-16 | Command palette (Ctrl/⌘+K) | P2 | 5 |
| F-17 | Live webcam mode | P2 | 9 |
| F-18 | Metadata / provenance check (EXIF, C2PA presence) | P1 | 2 |
| F-19 | Robustness panel (re-run under JPEG / blur / resize, show score drift) | P2 | 9 |

## 6. User stories (P0)

- As an **analyst**, I drop a video on the page and immediately see an upload progress bar, then the war room, so I know the system is working.
- As an **analyst**, I see each agent's status (queued → running → done / failed / skipped) and a one-line summary of what it found.
- As an **analyst**, I see the final verdict with a gauge and a confidence value, and I can tell at a glance whether the agents agreed.
- As an **analyst**, I scrub a timeline to see which moments of the video look most manipulated, and I see the heatmap on the matching frame.
- As an **analyst**, I read the Judge's explanation, which cites specific evidence from the other agents.
- As an **evaluator**, I open the leaderboard and see how our model compares with published baselines, with dataset and metric clearly labeled.
- As an **evaluator**, I can tell which numbers come from a real model and which from a mock or a paper.
- As a **builder**, I switch an agent from the mock detector to a real model by changing configuration only.

## 7. Scope by phase

See [ImplementationPlan.md](ImplementationPlan.md) and the step-by-step guides in [phases/](phases/). Summary:

| Phase | Outcome |
|---|---|
| 0 | Repo, tooling, CI, docs |
| 1 | Backend core: media upload, analyses, job runner, SSE |
| 2 | 4 agents on mock detectors + real signal processing, LangGraph orchestration, Claude-powered Judge |
| 3 | Frontend foundation: design system, layout, upload flow |
| 4 | War room + results visualizations |
| 5 | Dashboard, history, leaderboard shell, about page, PDF |
| 6 | Hardening: tests, security, performance, Docker, deploy |
| 7 | ML I: data pipeline, baselines, GenD + Effort visual models, Grad-CAM |
| 8 | ML II: temporal classifier, audio model, learned fusion + calibration; no mocks left |
| 9 | Evaluation, report, demo rehearsal, stretch features |

## 8. Assumptions

- Development happens on Windows laptops without a dedicated GPU; ML training uses Colab / Kaggle GPUs or a college lab machine.
- The Judge agent calls the Claude API; someone on the team supplies an API key. Without a key, a rule-based Judge takes over.
- Dataset access (FaceForensics++, Celeb-DF) requires request forms that take days to be approved. **Submit these in Phase 0**, even though ML is later.

## 9. Risks

| Risk | Impact | Mitigation |
|---|---|---|
| ML accuracy lands below 96% | Headline goal missed | Report in-domain and cross-dataset separately; use an ensemble; the dashboard is still strong |
| Dataset access delayed | Phase 7 blocked | Request access in week 1; DeepfakeBench / GenD preprocessed copies as fallback |
| GPU scarcity | Slow training | LayerNorm-only tuning (GenD) fits a free T4; cache face crops |
| Scope creep in the UI | Core flow not finished | P0 first; P2 only after Phase 6 exit criteria |
| Demo-day network failure | Judge LLM unavailable | Rule-based Judge fallback + pre-computed demo analyses |
| Mock results mistaken for real | Credibility loss with the evaluator | MOCK badge rule, `backend` field on every result |
