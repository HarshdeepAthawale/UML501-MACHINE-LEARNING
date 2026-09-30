# Phase 7 — ML I: Data Pipeline & Visual Models

| | |
|---|---|
| **Weeks** | Weeks 9–10 (2026-11-30 → 12-11) |
| **Owners** | H+G (pair; suggested lead: H on training, G on evaluation + explainability) |
| **Depends on** | Phase 6 (stable product), dataset approvals ([Q-09](../OpenQuestions.md#q-09)), GPU access ([Q-08](../OpenQuestions.md#q-08)) |
| **Unblocks** | Phase 8 (fusion, calibration, and the remaining detectors) |
| **Release** | `v0.7.0` — real visual detector live |

## Goal

Replace `mock-visual` with **trained, evaluated visual detectors**: reproducible data splits, two baselines, the main model (GenD) and a second model (Effort), real Grad-CAM heatmaps, and honest in-domain and cross-dataset results. This is where the 96–97% target is tested.

## Targets (from [PRD.md §4.2](../PRD.md#42-ml-goals-phase-7-targets--not-promises))

| Metric | Target |
|---|---|
| FF++ c23 test — video-level accuracy | ≥ 96% |
| FF++ c23 test — video-level AUC | ≥ 0.98 |
| Celeb-DF v2 test (cross-dataset) — AUC | ≥ 0.90 (reported whatever it is) |
| Beats Xception and EfficientNet-B4 on both | yes |

## Inputs

- FF++ (c23), Celeb-DF v2, a DFDC subset, and (optional) a DF40 subset
- Research notes: GenD (WACV 2026), Effort (ICML 2025), DeepfakeBench
- `Detector` protocol from Phase 2; `model_registry` from Phase 5

## Deliverables

```
ml/
├── pyproject.toml                  # torch, timm, transformers, open_clip_torch, pytorch-grad-cam, scikit-learn, umap-learn
├── configs/{data,baselines,gend,effort}.yaml
├── splits/{ffpp_train,ffpp_val,ffpp_test,celebdf_test,dfdc_subset}.txt   # committed (R-8)
├── scripts/
│   ├── extract_faces.py            # video → 32 aligned 224×224 face crops
│   ├── build_index.py              # crops → parquet index (path, video_id, label, manipulation, split)
│   ├── train.py                    # config-driven training
│   ├── evaluate.py                 # frame- and video-level metrics per dataset
│   ├── gradcam.py
│   └── export.py                   # weights + model_card.json
├── src/aletheia_ml/{data,models,losses,metrics,explain}/
└── results/                        # metrics CSVs + plots (committed); no weights, no frames
```

Plus `apps/api/app/detectors/{gend,effort,ensemble_visual}.py`.

## Tasks

### P7-01 · ML environment and experiment tracking

- `ml/` as its own uv project (GPU wheels differ from the API). Pin torch/CUDA versions matching Colab/Kaggle.
- Seeds everywhere (`torch`, `numpy`, `random`, dataloader workers); `torch.backends.cudnn.benchmark=False` for final runs.
- Tracking: TensorBoard logs + one row per run appended to `ml/results/runs.csv` (run id, config hash, git commit, dataset, metrics). Keep it simple; no paid services.
- A Colab/Kaggle notebook `ml/notebooks/train_remote.ipynb` that clones the repo, mounts data, and calls `train.py` — logic lives in scripts, not notebooks.

### P7-02 · Data pipeline and splits

1. Download FF++ c23 (original + Deepfakes, Face2Face, FaceSwap, NeuralTextures) with the official script; Celeb-DF v2; a DFDC subset (a few thousand videos, balanced).
2. Use the **official FF++ splits** (720 / 140 / 140 videos per class) and the **official Celeb-DF v2 test list** (518 videos). Write them to `ml/splits/` and commit.
3. `extract_faces.py`: 32 evenly spaced frames per video, RetinaFace detection, largest face, 1.3× margin, align, resize 224×224, save JPEG. Same logic as the API's preprocessing where possible (document any difference — train/serve skew matters).
4. `build_index.py` → parquet index; sanity report: counts per split/label/manipulation, missing-face rate.
5. Class imbalance: FF++ has 4 fakes per real → balanced sampling per batch (and keep real/fake **pairs** together where possible — GenD found paired data helps).
6. Fallback if approvals are late: DeepfakeBench / GenD preprocessed releases (check their terms first).

**Accept:** index report committed; no video appears in more than one split (automated check).

### P7-03 · Baselines (EfficientNet-B4, Xception)

- Train with DeepfakeBench configs (or `timm` equivalents in our trainer) on FF++ c23 train, select on val, evaluate on FF++ test + Celeb-DF v2 + DFDC subset.
- Frame-level and video-level (mean of frame probabilities) metrics.

**Accept:** results rows with `source=ours` for both baselines. If they're far below published numbers, find out why before moving on (usually preprocessing or leakage).

### P7-04 · Main model: GenD (CLIP ViT-L/14)

- Backbone: CLIP ViT-L/14 image encoder, **all weights frozen except LayerNorm** scale/shift; linear head on the normalized CLS embedding.
- Loss: BCE + α·alignment + β·uniformity on the unit hypersphere (paper values α=0.1, β=0.5 as the starting point).
- Augmentations: horizontal flip, small affine, Gaussian blur, colour jitter, JPEG compression (quality 60–100).
- Optimiser AdamW, cosine schedule, small LR for LN params; mixed precision; batch size to fit a T4 (gradient accumulation if needed).
- Early stop on val AUC. **3 seeds** for the final configuration.
- Sanity check: evaluate the public `yermandy/GenD_CLIP_L_14` checkpoint on our Celeb-DF split to validate our preprocessing — but the leaderboard's `OURS` row must be **our** fine-tuned model (the public checkpoint gets a `paper` row with a citation).

**Accept:** FF++ test and Celeb-DF v2 metrics logged for 3 seeds (mean ± std).

### P7-05 · Second model: Effort (SVD residual tuning)

Use the official Effort implementation (CLIP ViT-L/14; SVD of each linear layer; principal components frozen, residual subspace trained, orthogonality regulariser). Same data, splits, augmentations, and evaluation as GenD.

### P7-06 · Explainability: Grad-CAM for ViTs

- `pytorch-grad-cam` with a reshape transform (drop CLS token, reshape 16×16 patch tokens) targeting the last block's LayerNorm; compare with attention rollout and keep whichever gives cleaner maps.
- Output: 224×224 saliency in [0,1] per crop → the existing heatmap rendering in the Visual agent.
- Sanity checks: maps change when the model weights are randomized (model-randomization test); document a few good and bad examples.

### P7-07 · Visual detectors in the API

- `detectors/gend.py`, `detectors/effort.py`, `detectors/ensemble_visual.py` implementing `Detector` (batch all crops in one forward pass; GPU if available, CPU otherwise; `torch.inference_mode()`).
- Weights downloaded at startup from the private weights location into `DATA_DIR/models/`, SHA-256 verified ([Deployment.md §2](../Deployment.md#2-build-artifacts)).
- `backend="model"`, `backend_version="gend-clip-l14@ffpp-c23-<run-id>"`.
- Switch with `DETECTOR_VISUAL=gend-clip-l14` — the MOCK badge disappears automatically for this agent.
- Measure inference time on CPU and GPU for 32 crops; update [TechSpecifications.md §6](../TechSpecifications.md#6-performance-targets-backend-cpu-mock-detectors).

### P7-08 · Visual evaluation report

`ml/results/visual_report.md` (+ plots): metrics per dataset and per FF++ manipulation, ROC curves, confusion matrices at 0.5 and at the EER threshold, frozen-vs-LN-tuning ablation, with/without alignment-uniformity ablation, failure-case gallery (only from datasets whose license allows it in a private report). Export ROC points and metrics into `model_card.json` for the leaderboard.

## Exit criteria

- [ ] Splits committed; leakage check passes
- [ ] Two baselines + GenD + Effort trained and evaluated on FF++ test and Celeb-DF v2 (DFDC subset if time)
- [ ] In-domain and cross-dataset results reported **separately**, with seeds and std for GenD
- [ ] Real Grad-CAM heatmaps in the war room
- [ ] `DETECTOR_VISUAL` switched to the best model on the demo deployment
- [ ] `v0.7.0` tagged

## End-of-phase demo

Run the demo set through the app: the Visual agent now shows `MODEL` instead of `MOCK`, heatmaps are Grad-CAM, and the leaderboard has `OURS` rows beside the baselines. Walk through the visual evaluation report.

## Risks

| Risk | Mitigation |
|---|---|
| Dataset access still pending | Preprocessed releases fallback; start with Celeb-DF if FF++ is late |
| GPU time limits (Colab/Kaggle) | LN-only tuning is cheap; cache face crops once; checkpoint every epoch |
| Accuracy under 96% in-domain | Ensemble (Phase 8), more frames per video, tune threshold on val — and report honestly if still short |
| Train/serve preprocessing skew | Same crop logic in `ml/` and the API; test by scoring FF++ test videos through the API end to end |
| Test-set leakage | Committed splits, automated overlap check, test set evaluated only for final numbers |

## Handoff

- **To Phase 8:** per-video validation-set scores from each visual model (for stacking), embeddings for the explorer, model cards.

## References

[PRD.md §4.2](../PRD.md#42-ml-goals-phase-7-targets--not-promises) · [TestPlan.md §7](../TestPlan.md#7-ml-evaluation-protocol-phase-7-preview) · [Decisions ADR-013](../Decisions.md#adr-013) · [Rules.md §2](../Rules.md#r-honesty)
