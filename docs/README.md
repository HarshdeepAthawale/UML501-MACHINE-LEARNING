# Aletheia — Documentation Index

> **Aletheia** (Greek: "truth, un-concealment") — a multi-agent deepfake forensics platform.
> UML501 Machine Learning project · Team: Harshdeep Athawale, Girikshit Gupta.
> Working name — see [OpenQuestions.md](OpenQuestions.md#q-01).

## Build order (the one-line version)

**Backend API + 4 AI agents (on mock detectors) → Frontend dashboard → Hardening → swap in real ML models → Evaluation & demo.**
The ML models plug into a detector interface that exists from day one, so nothing in the app changes when the real models arrive. See [ImplementationPlan.md](ImplementationPlan.md).

## Read in this order

| # | Doc | What it answers | Read before |
|---|-----|-----------------|-------------|
| 1 | [PRD.md](PRD.md) | What are we building, for whom, and how do we know it's good? | Anything |
| 2 | [NonGoals.md](NonGoals.md) | What are we deliberately *not* building? | Scoping any feature |
| 3 | [TechSpecifications.md](TechSpecifications.md) | Architecture, stack, modules, agent framework, fusion math | Writing code |
| 4 | [Appflow.md](Appflow.md) | Screens, user journeys, analysis lifecycle | Frontend or API work |
| 5 | [Design.md](Design.md) | Visual language, tokens, components, motion | Any UI work |
| 6 | [Schema.md](Schema.md) | Database tables, storage layout | Backend work |
| 7 | [API.md](API.md) | Every endpoint, request/response shape, SSE events, error codes | Frontend ↔ backend work |
| 8 | [ImplementationPlan.md](ImplementationPlan.md) | Phases, tasks, owners, exit criteria | Planning a week |
| 9 | [Tracker.md](Tracker.md) | Live task status | Every standup |
| 10 | [Rules.md](Rules.md) | Non-negotiable engineering rules (for humans *and* AI assistants) | First commit |

### Supporting docs

| Area | Docs |
|------|------|
| Decisions & history | [Decisions.md](Decisions.md) · [Changelog.md](Changelog.md) |
| Setup & ops | [Setup.md](Setup.md) · [Deployment.md](Deployment.md) |
| Quality & safety | [TestPlan.md](TestPlan.md) · [Security.md](Security.md) |
| Scope control | [NonGoals.md](NonGoals.md) · [OpenQuestions.md](OpenQuestions.md) |
| Team & process | [Contributing.md](Contributing.md) · [Glossary.md](Glossary.md) |

## Doc maintenance rules

- Docs live in the repo and change in the same PR as the code they describe.
- A PR that changes an endpoint must update [API.md](API.md); one that changes a table must update [Schema.md](Schema.md).
- Any choice between two reasonable options gets an ADR in [Decisions.md](Decisions.md).
- Unresolved questions go in [OpenQuestions.md](OpenQuestions.md) — never silently dropped.
- Status lives in [Tracker.md](Tracker.md), not in chat.
