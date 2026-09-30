# Open Questions

Unresolved product and design questions. Each has an owner and a "decide by" point. When resolved, fill in **Resolution**, move it to the Resolved section, and write an ADR in [Decisions.md](Decisions.md) if it changes the architecture.

**Status:** `open` · `investigating` · `resolved`

## Open

<a id="q-01"></a>
### Q-01 — Project name

- **Status:** open · **Owner:** H+G · **Decide by:** end of Phase 0
- **Question:** Keep "Aletheia" or pick another name? It appears in the UI, docs, Docker images, and slides.
- **Notes:** Changing it after Phase 3 means renaming in many places.

<a id="q-02"></a>
### Q-02 — Deadlines and evaluation dates

- **Status:** open · **Owner:** H · **Decide by:** Week 1
- **Question:** What are the real dates for the mid-term review, final demo, and report submission? Does the course weight the report, the demo, or the model results most?
- **Impact:** The 12-week plan in [ImplementationPlan.md](ImplementationPlan.md) assumes the final demo in mid/late December 2026. If it's earlier, cut P2 features and compress Phase 8.

<a id="q-03"></a>
### Q-03 — Does the professor have constraints on tools?

- **Status:** open · **Owner:** H+G · **Decide by:** Week 1
- **Question:** Are pretrained checkpoints / foundation models allowed, or must we train from scratch? Is using an LLM API in the product OK? Any required framework (e.g. scikit-learn component, specific report format)?
- **Impact:** Could change the model plan (ADR-013) and the Judge (ADR-008).

<a id="q-04"></a>
### Q-04 — Ownership split

- **Status:** open · **Owner:** H+G · **Decide by:** end of Phase 0
- **Question:** Is the proposed split (H = backend + agents, G = frontend) right? Both of us need to be able to explain every part in the viva.
- **Proposal:** Keep the split for Phases 1–5, pair on Phase 7, and each person reviews all of the other's PRs so knowledge is shared.

<a id="q-05"></a>
### Q-05 — Claude API key and budget for the Judge

- **Status:** open · **Owner:** H · **Decide by:** start of Phase 2
- **Question:** Who provides the API key, and what monthly spend is acceptable? Do we need a hard cap?
- **Notes:** One Judge call per analysis. Set a spending limit in the Anthropic Console, and use `JUDGE_ENABLED=false` during heavy UI development (the rules Judge is free). Estimate the cost per analysis in Phase 2 from logged token usage.

<a id="q-06"></a>
### Q-06 — Where to host the demo

- **Status:** open · **Owner:** H · **Decide by:** start of Phase 6
- **Options:** Vercel + Hugging Face Spaces (recommended for the mock phase) · Vercel + Render/Railway · lab GPU machine + Cloudflare Tunnel (Phase 7) · local only. See [Deployment.md §3](Deployment.md#3-hosting-options-decide-in-openquestionsmd-q-06).
- **Depends on:** Q-08 (GPU availability).

<a id="q-07"></a>
### Q-07 — Do we need user accounts?

- **Status:** open · **Owner:** H+G · **Decide by:** Phase 5
- **Question:** Is a shared access token enough for the demo, or does the evaluator expect login and per-user history?
- **Default if undecided:** shared token only; `users` table stays unused.

<a id="q-08"></a>
### Q-08 — GPU access for Phase 7

- **Status:** open · **Owner:** H+G · **Decide by:** Week 6
- **Question:** Do we have a lab GPU? Otherwise Colab (session limits) or Kaggle (~30 GPU h/week)? Is paid cloud GPU an option?
- **Impact:** Decides whether we fine-tune GenD ourselves on the full FF++ set or on a subset, and where inference runs in the demo.

<a id="q-09"></a>
### Q-09 — Dataset access status

- **Status:** open · **Owner:** H+G · **Decide by:** Week 1 (submit), Week 8 (must have)
- **Question:** Were FF++ and Celeb-DF v2 approved? If not by Week 8, fall back to the preprocessed copies distributed by DeepfakeBench / GenD — check that their licenses allow it for our use.

<a id="q-10"></a>
### Q-10 — Demo set sourcing

- **Status:** open · **Owner:** G · **Decide by:** end of Phase 0
- **Question:** Which real and fake clips can we show publicly in the demo and slides without license or consent problems? (Dataset clips are research-only and shouldn't appear on a public deployment.)
- **Idea:** Record our own real clips (both of us, with consent), plus clearly licensed public examples of known deepfakes; for fakes of ourselves, *don't* generate them (see [NonGoals.md](NonGoals.md)) — use licensed examples instead.

<a id="q-11"></a>
### Q-11 — Should specialist agents also use the LLM?

- **Status:** open · **Owner:** H · **Decide by:** end of Phase 2
- **Question:** Turn on `AGENT_LLM_OBSERVATIONS` (Claude looks at keyframes for lighting/boundary oddities) or keep specialists purely model/signal-based?
- **Trade-off:** More impressive "agentic" behaviour and richer explanations vs. higher cost, latency, and a less clean ML story.

<a id="q-12"></a>
### Q-12 — Frame sampling default

- **Status:** open · **Owner:** H · **Decide by:** Phase 7
- **Question:** 32 evenly spaced frames (matches GenD's evaluation) vs. fixed fps sampling. Measure accuracy vs. latency once real models exist.

## Resolved

*(none yet)*
