# Phase 9 — Evaluation, Report & Demo

| | |
|---|---|
| **Weeks** | Week 12 (2026-12-21 → 12-24, adjust to the real deadline — [Q-02](../OpenQuestions.md#q-02)) |
| **Owners** | H+G |
| **Depends on** | Phase 8 |
| **Unblocks** | Submission and viva |
| **Release** | `v1.0.0` |

## Goal

Freeze everything, produce the final numbers, write the project report, and deliver a rehearsed demo that works even if the network doesn't. Stretch features (robustness panel, webcam mode) only if the must-haves are finished by mid-week.

## Inputs

- Models, reports, and deployment from Phase 8
- DemoRunbook from Phase 6
- Course requirements for the report and viva ([Q-02](../OpenQuestions.md#q-02), [Q-03](../OpenQuestions.md#q-03))

## Deliverables

- Frozen final evaluation (`ml/results/final/`) with every table and plot used in the report
- Project report (PDF) and slide deck
- Updated DemoRunbook + two rehearsal notes
- Optional: robustness panel, webcam live mode
- `v1.0.0` release

## Must-have tasks

### P9-01 · Final evaluation run

1. Freeze model versions (record run IDs and weight hashes).
2. Run `evaluate.py` once on every test set with the final pipeline: visual models, temporal, audio, stacked + calibrated ensemble.
3. Produce the final tables:
   - **Table 1 — In-domain (FF++ c23 test):** accuracy, AUC, EER per model, frame- and video-level, per manipulation type.
   - **Table 2 — Cross-dataset:** AUC on Celeb-DF v2, DFDC subset (and DF40 subset if done).
   - **Table 3 — Audio:** EER on ASVspoof 2019 LA eval and In-the-Wild.
   - **Table 4 — Ablations:** frozen vs LN tuning; ± alignment/uniformity; single vs ensemble; hand-crafted temporal vs none; calibration before/after (ECE).
   - **Table 5 — Robustness:** AUC under JPEG / blur / resize.
4. State clearly whether each PRD target (M-1 … M-5) was met. Numbers are what they are (Rules R-44).

**Accept:** every number in the report traces to a file in `ml/results/final/` and a run ID.

### P9-02 · Project report

Suggested structure (adapt to the course template):

1. Abstract
2. Introduction — deepfakes, why detection is hard, why generalization matters
3. Related work — CNN baselines, foundation-model detectors (GenD, Effort), audio anti-spoofing, explainable/agentic forensics
4. System design — architecture, the four agents, LangGraph pipeline, SSE, dashboard (screenshots)
5. Methods — data and splits, preprocessing, models, losses (BCE, alignment, uniformity), SVD residual tuning, classical temporal features (FFT, SSIM, optical flow), audio model, stacking, temperature scaling, Grad-CAM
6. Experiments and results — Tables 1–5, ROC curves, reliability diagrams, embedding plot
7. Discussion — in-domain vs cross-dataset gap, failure cases, what the agents add
8. Limitations and ethics — probabilistic output, dataset bias, privacy, misuse
9. Conclusion and future work
10. References; appendix with hyperparameters and the API summary

Each of us writes the sections for the parts we led, then we swap and edit each other's sections so both of us can explain everything.

### P9-03 · Slides and demo script

- 12–15 slides: problem → our approach (4 agents) → live demo → results tables → what we learned → limitations.
- Demo script (7 minutes), from the DemoRunbook:
  1. Landing → "Analyze media" (30 s)
  2. Upload a **real** clip → war room → verdict real (90 s)
  3. Upload a **fake** clip → point at the timeline, heatmap, and Judge evidence (2 min)
  4. Leaderboard: our model vs baselines, in-domain vs cross-dataset (1 min)
  5. About page: fusion math with the sliders (1 min)
  6. Explorer: real/fake clusters (30 s)
  7. Close on limitations (30 s)
- Fallbacks: pre-computed analyses for steps 2–3 if uploads are slow; local docker compose if the network fails; rules Judge if the LLM is down.

### P9-04 · Rehearsals and release

- Two full rehearsals on the actual demo machine (projector resolution, dark theme), timed; fix everything that stutters.
- Final smoke test on the deployment and locally.
- Update [Changelog.md](../Changelog.md) and [Tracker.md](../Tracker.md); tag `v1.0.0`; GitHub Release with the report PDF attached (no datasets or weights).

## Stretch tasks (only after P9-01 … P9-03 are done)

### P9-05 · Robustness panel (P2)

On the results page: "Stress test" button → server re-runs the Visual detector on JPEG q=50, blur σ=2, and 0.5× resize variants of the same frames → small chart of score drift. Reuses the Phase 8 robustness code. New endpoint `POST /analyses/{id}/robustness` (update API.md first).

### P9-06 · Live webcam mode (P2)

`/live`: `getUserMedia` → send ≤ 5 fps JPEG frames (≤ 640 px) over the `WS /live` endpoint → Visual detector only → live score line and face box overlay. Clear "Visual agent only, no Judge" label. Works on localhost or HTTPS only.

## Exit criteria

- [ ] Final tables frozen and traceable
- [ ] Report and slides submitted/ready
- [ ] Two rehearsals done within time; fallbacks tested
- [ ] `v1.0.0` released
- [ ] Both of us can explain every component, the math, and every number

## Viva prep checklist

- [ ] Why log-odds fusion instead of averaging probabilities?
- [ ] Why LN-only tuning generalizes better than full fine-tuning?
- [ ] What does AUC mean, and why is it different from accuracy?
- [ ] Why is cross-dataset performance lower, and what would improve it?
- [ ] What does the Judge LLM do, and why can't it change the verdict?
- [ ] How do you know the heatmaps are meaningful (sanity checks)?
- [ ] How did you avoid test-set leakage?
- [ ] What are the ethical risks of a deepfake detector?

## Risks

| Risk | Mitigation |
|---|---|
| Deadline earlier than planned | Skip stretch tasks; the must-haves fit in 3–4 days |
| Demo-day failure | Local compose + pre-computed analyses + rules Judge |
| Results below target | Report honestly with analysis; the system design and evaluation rigor still carry the project |

## References

[ImplementationPlan.md](../ImplementationPlan.md) · [PRD.md §4](../PRD.md#4-goals-and-success-metrics) · [TestPlan.md §7](../TestPlan.md#7-ml-evaluation-protocol-phase-7-preview) · [Security.md §6](../Security.md#6-privacy-and-data-handling)
