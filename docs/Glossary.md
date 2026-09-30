# Glossary

Terms and acronyms used across the docs and the code. Alphabetical within each section.

## Deepfakes and forensics

| Term | Meaning |
|---|---|
| **Artifact (forensic)** | A visible or statistical trace left by manipulation: blending seams, texture smoothing, grid patterns in the frequency spectrum, flicker. (In our API, "artifact" also means a saved image/file produced by an agent — see below.) |
| **Blending boundary** | The seam where a swapped face is pasted onto the target; often shows colour or sharpness mismatch |
| **C2PA** | Coalition for Content Provenance and Authenticity — a standard for signed provenance metadata embedded in media |
| **Cheapfake** | Manipulation without AI (speed changes, cuts, mislabeling) |
| **Deepfake** | Media where a person's face or voice is synthesized or altered by deep learning |
| **Entire face synthesis** | A face that doesn't exist, generated from scratch (StyleGAN, diffusion) |
| **EXIF** | Metadata embedded in image files (camera, software, timestamps) |
| **Face reenactment** | Driving a target face's expressions/pose from a source actor (e.g. Face2Face) |
| **Face swap** | Replacing one person's face with another's (e.g. DeepFakes, FaceSwap, FaceShifter) |
| **Lip-sync deepfake** | Mouth region altered to match new audio (e.g. Wav2Lip) |
| **Optical flow** | Per-pixel motion between consecutive frames; inconsistent flow in the face vs. background can reveal manipulation |
| **rPPG** | Remote photoplethysmography — the faint pulse signal visible in skin colour; often disrupted in fakes |
| **Voice cloning / TTS spoof** | Synthetic speech imitating a real speaker |

## Datasets

| Term | Meaning |
|---|---|
| **ASVspoof 2019 LA** | Standard audio anti-spoofing dataset (Logical Access: TTS and voice conversion attacks) |
| **c23 / c40** | FaceForensics++ compression levels (H.264 constant rate factor 23 = light, 40 = heavy) |
| **Celeb-DF v2** | High-quality celebrity deepfake videos; standard cross-dataset test |
| **Deepfake-Eval-2024** | In-the-wild deepfakes collected from social media in 2024; shows the gap between benchmarks and reality |
| **DeepfakeBench** | Open benchmark codebase with dozens of detectors and preprocessed datasets |
| **DF40** | Benchmark with 40 deepfake techniques, including diffusion-based ones |
| **DFDC** | Facebook/Meta Deepfake Detection Challenge dataset; large and diverse |
| **FF++ (FaceForensics++)** | The standard training set: 1,000 real YouTube videos, each manipulated by 4 methods (Deepfakes, Face2Face, FaceSwap, NeuralTextures) |
| **In-the-Wild (audio)** | Real-world audio deepfake dataset of public figures |

## ML and math

| Term | Meaning |
|---|---|
| **AASIST** | Graph-attention audio anti-spoofing model |
| **Alignment / uniformity loss** | Contrastive-style losses: pull same-class features together; spread all features evenly on the unit sphere (used by GenD) |
| **AUC (ROC-AUC)** | Area under the ROC curve; probability a random fake scores higher than a random real. 0.5 = chance, 1.0 = perfect |
| **Calibration** | How well predicted probabilities match observed frequencies (a "0.8" should be right 80% of the time) |
| **CLIP** | Vision-language model trained on image–text pairs; its image encoder is a strong starting point for detection |
| **Cross-dataset** | Train on one dataset, test on a different one — measures generalization |
| **DINOv3** | Self-supervised vision foundation model (Meta) |
| **ECE** | Expected Calibration Error — average gap between confidence and accuracy across probability bins |
| **EER** | Equal Error Rate — the error rate at the threshold where false-accept and false-reject rates are equal (common in audio) |
| **Effort** | Detector that uses SVD to freeze a pretrained model's main weight directions and tune only the rest (ICML 2025) |
| **Ensemble / stacking** | Combining several models; stacking trains a small model (e.g. logistic regression) on their outputs |
| **FFT / DCT** | Fast Fourier / Discrete Cosine Transform — move an image into the frequency domain, where generator artifacts often show up |
| **Fine-tuning** | Continuing to train a pretrained model on our task |
| **GenD** | Detector that fine-tunes only LayerNorm parameters of a foundation model (CLIP / DINOv3 / PE) with alignment-uniformity losses (WACV 2026) |
| **Grad-CAM** | Heatmap of which image regions most influenced a prediction (gradients × activations) |
| **In-domain** | Test data from the same distribution as the training data |
| **LayerNorm tuning** | Updating only the normalization layers' scale/shift parameters (~0.03% of weights) |
| **Logit / log-odds** | log(p / (1 − p)); our fusion averages these rather than raw probabilities |
| **LoRA** | Low-Rank Adaptation — parameter-efficient fine-tuning via small low-rank weight updates |
| **ROC curve** | True-positive rate vs. false-positive rate across all thresholds |
| **SSIM** | Structural Similarity Index — similarity between two images; we use it frame-to-frame to detect flicker |
| **SVD** | Singular Value Decomposition — factorizes a matrix into orthogonal directions ranked by importance |
| **Temperature scaling** | Calibration method: divide logits by a learned T before the sigmoid/softmax |
| **t-SNE / UMAP** | Dimensionality reduction for visualizing high-dimensional embeddings in 2D/3D |
| **ViT** | Vision Transformer |
| **XLS-R / wav2vec 2.0** | Self-supervised speech models; used as the front-end for audio deepfake detection |

## Our system

| Term | Meaning |
|---|---|
| **Agent** | One of the four analysis units: Visual Forensics, Temporal & Frequency, Audio, Judge |
| **Agent run** | One agent's execution within one analysis (`agent_runs` table) |
| **Analysis** | One run of the full pipeline on one media file |
| **Artifact (API)** | A file produced during analysis: keyframe, face crop, heatmap, spectrum, spectrogram, PDF |
| **Backend (of a result)** | What produced a score: `mock`, `model`, `llm`, or `rules` |
| **Detector** | The scoring component an agent calls; swappable via the registry |
| **Disagreement** | The largest gap between any two agents' P(fake) |
| **Finding** | A specific, located piece of evidence (what, where, when, how severe) |
| **Frame score** | P(fake) for one sampled frame |
| **Fusion** | Deterministic combination of agent scores into the verdict (weighted log-odds) |
| **Judge** | The fourth agent: explains the fused verdict in plain language (Claude, or the rules fallback) |
| **Mock detector** | A placeholder that produces deterministic, plausible scores so the product can be built before models are trained; always badged |
| **Mock scenario** | Development-only option that forces a mock outcome (`real`, `fake`, `uncertain`, `disagree`, `agent_fail`) |
| **P(fake)** | The probability the media is manipulated; the meaning of every `score` field |
| **Rules Judge** | Template-based report used when the LLM is unavailable or disabled |
| **Verdict** | Final label (`real`, `fake`, `uncertain`) + probability + confidence |
| **War room** | The live analysis screen showing agents working in real time |

## Engineering

| Term | Meaning |
|---|---|
| **ADR** | Architecture Decision Record ([Decisions.md](Decisions.md)) |
| **Alembic** | Database migration tool for SQLAlchemy |
| **LangGraph** | Library for building agent workflows as graphs with shared state |
| **OpenAPI** | Machine-readable API description; FastAPI generates it, the frontend generates types from it |
| **Problem Details (RFC 9457)** | Standard JSON error format we use for every API error |
| **SSE** | Server-Sent Events — one-way server-to-browser event stream over HTTP |
| **ULID** | Sortable unique ID; ours are prefixed (`ana_…`, `med_…`) |
