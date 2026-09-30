# Phase 8 — ML II: Temporal, Audio, Fusion & Integration

| | |
|---|---|
| **Weeks** | Week 11 (2026-12-14 → 12-18) |
| **Owners** | H+G (suggested: H on audio + integration, G on temporal classifier + embeddings) |
| **Depends on** | Phase 7 (visual models, validation scores) |
| **Unblocks** | Phase 9 (final evaluation and demo) |
| **Release** | `v0.8.0` — no mock detectors left |

## Goal

Every scoring agent runs a real model, the fusion weights are **learned** instead of hand-set, probabilities are **calibrated**, and the leaderboard and 3D explorer show our real results. At the end of this phase, the MOCK badge no longer appears anywhere in normal use.

## Inputs

- Phase 7 outputs: visual models, per-video validation/test scores, splits
- Temporal features already computed by the Temporal agent (Phase 2: `hf_energy_ratio`, `ssim_mean`, `ssim_min`, `flow_ratio`, `bbox_jitter`)
- ASVspoof 2019 LA; In-the-Wild audio dataset

## Deliverables

- `detectors/temporal_classical.py`, `detectors/xlsr_aasist.py`
- `ml/results/{temporal_report,audio_report,fusion_report}.md`
- `fusion.json` (stacker weights + temperature) loaded by the API
- Real rows in `model_registry` and `embedding_points`
- Updated performance numbers in TechSpecifications

## Tasks

### P8-01 · Temporal classifier (classical features)

1. Run the Temporal agent's feature code over FF++ train/val/test videos with a batch script that imports the **same** functions from `apps/api/app/media/` (no copy-paste — avoids skew).
2. Features: the five metrics plus summary statistics over frames (mean, std, min, max, percentiles).
3. Models: logistic regression (standardized features) and gradient boosting (scikit-learn `HistGradientBoostingClassifier`); 5-fold CV on train, select on val.
4. Report AUC in-domain and on Celeb-DF v2, plus feature importances (permutation importance).
5. Honest framing for the report: this is the "hand-crafted features vs. foundation model" ablation — it's expected to be weaker than GenD; its value is complementary evidence and interpretability.

**Accept:** `temporal_report.md` with metrics and importances; `detectors/temporal_classical.py` serving the chosen model (`joblib` file).

### P8-02 · Audio model: XLS-R + AASIST

1. Start from a published SSL-AASIST checkpoint (wav2vec 2.0 XLS-R front end + AASIST back end) or fine-tune it on ASVspoof 2019 LA train with RawBoost-style augmentation if GPU time allows.
2. Evaluate EER on ASVspoof 2019 LA eval (target ≤ 5%) and on In-the-Wild (report as is — expect worse).
3. Inference: 4 s windows with 50% overlap on 16 kHz audio → per-window scores → suspicious segments (top windows above threshold) + clip score (mean of top-k windows).
4. `detectors/xlsr_aasist.py`; `backend_version` records the checkpoint and whether we fine-tuned.

**Accept:** `audio_report.md`; the Audio agent shows `MODEL` with real segments on the waveform.

### P8-03 · Stacking fusion

1. Build a table of **validation-set** per-video scores: visual (GenD, Effort, baselines), temporal, audio (where the video has audio; FF++ is mostly silent — handle missing audio with an indicator feature or a separate stacker).
2. Fit logistic regression on logits of the agent scores (this is the learned version of the weighted log-odds in [TechSpecifications.md §4.7](../TechSpecifications.md#47-fusion-the-math)); compare with the hand-set weights and a simple average.
3. Export weights to `fusion.json`; the API's fusion loads them (fallback to defaults if absent).

### P8-04 · Calibration

- Temperature scaling on the stacked logit: fit T on validation NLL; apply p = σ(z / T).
- Report ECE (15 bins) and reliability diagrams before/after on val and test (target ECE ≤ 0.05).
- Re-check the `real`/`fake`/`uncertain` thresholds (0.35/0.65) against the calibrated distribution; adjust only using validation data and document the change in an ADR.

### P8-05 · Embeddings for the explorer

- Extract CLS embeddings from the best visual model for a balanced sample (e.g. 2,000 FF++ test + 1,000 Celeb-DF v2 test crops/videos).
- UMAP to 3D (`n_neighbors=30`, `min_dist=0.1`, fixed seed); save with labels and manipulation types.
- Import into `embedding_points` under the real model ID; thumbnails only where licenses allow (otherwise none — the explorer handles missing thumbs).
- Remove or hide the synthetic `mdl_placeholder_demo` set in production.

### P8-06 · Model cards → registry

`export.py` writes `model_card.json` for every trained model (architecture, trainable params, data, splits, metrics per dataset with split type, ROC points, seeds, run IDs, git commit). `scripts/import_model_cards.py` upserts them into `model_registry` as `source=ours`; placeholder rows for trained models are removed. Mark the active model per agent (`is_active`).

### P8-07 · Integration and performance

- Switch all `DETECTOR_*` variables on the demo deployment; run the full demo set; verify no `backend=mock` remains (`is_mock=false`).
- End-to-end check: score a handful of FF++ **test** videos through the API and confirm the scores match `evaluate.py` within tolerance (train/serve skew test).
- Measure end-to-end latency on the deployment hardware; update [TechSpecifications.md §6](../TechSpecifications.md#6-performance-targets-backend-cpu-mock-detectors) with measured numbers.
- If CPU inference is too slow for the demo: GPU host (Q-06/Q-08), fewer frames (Q-12), or a smaller model for live use with the big model for the report.

### P8-08 · Robustness evaluation (offline)

Re-evaluate the final ensemble on FF++ test and Celeb-DF v2 under JPEG (q=90/70/50), Gaussian blur (σ=1/2), and downscale (0.75×/0.5×). Plot AUC drift. (The in-app robustness panel is a Phase 9 stretch that reuses this.)

## Exit criteria

- [ ] Temporal and Audio agents run real models; Visual runs the best model or ensemble
- [ ] Learned fusion + temperature scaling in production; ECE before/after reported
- [ ] Leaderboard shows only `ours` and `paper` rows (no placeholders for trained models)
- [ ] Explorer shows real embeddings
- [ ] Train/serve skew check passes
- [ ] `is_mock=false` for all demo-set analyses on the deployment
- [ ] `v0.8.0` tagged

## End-of-phase demo

Same demo video as in Phase 4 — now every agent shows `MODEL`, the verdict is calibrated, the leaderboard compares our ensemble with baselines and paper numbers, and the explorer shows real/fake clusters from our own model.

## Risks

| Risk | Mitigation |
|---|---|
| FF++ has little audio → stacker can't learn audio weight | Separate audio-present stacker trained on data that has audio, or keep a hand-set audio weight and say so |
| Calibration overfits a small val set | Temperature scaling has one parameter; report test ECE too |
| Deployment too slow with real models | Fewer frames, GPU host, or pre-computed demo analyses |
| Temporal classifier near chance cross-dataset | That's a finding, not a failure — report it; lower its fusion weight |

## Handoff

- **To Phase 9:** frozen models, final metrics, reports, and a fully real product for the final evaluation and demo.

## References

[TechSpecifications.md §4.7](../TechSpecifications.md#47-fusion-the-math) · [TestPlan.md §7](../TestPlan.md#7-ml-evaluation-protocol-phase-7-preview) · [PRD.md §4.2](../PRD.md#42-ml-goals-phase-7-targets--not-promises)
