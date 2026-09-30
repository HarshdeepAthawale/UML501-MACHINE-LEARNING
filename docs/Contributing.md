# Contributing

How the two of us (and any AI assistant we use) get changes into `main`. Engineering rules are in [Rules.md](Rules.md).

## 1. Workflow

1. Pick a task in [Tracker.md](Tracker.md); set it to `doing` with your initial.
2. Branch from an up-to-date `main`.
3. Commit small, focused changes (Conventional Commits).
4. Push and open a PR early (draft is fine) using the PR template.
5. The other person reviews within 24 hours.
6. Squash-merge when CI is green and the review is approved. Delete the branch.
7. Mark the task `done` in [Tracker.md](Tracker.md) (in the same PR).

## 2. Branch naming

```
<type>/<task-id>-<short-description>
```

| Type | Use for |
|---|---|
| `feat/` | New functionality |
| `fix/` | Bug fixes |
| `docs/` | Documentation only |
| `refactor/` | Code change without behaviour change |
| `test/` | Tests only |
| `chore/` | Tooling, deps, config, CI |
| `perf/` | Performance improvements |
| `ml/` | Training, evaluation, data scripts (Phases 7–8) |

Examples: `feat/P1-04-media-upload`, `feat/P4-02-war-room-layout`, `fix/sse-replay-duplicates`, `docs/api-frames-endpoint`.

## 3. Commit messages — Conventional Commits

```
<type>(<scope>): <summary in imperative mood, ≤ 72 chars>

<optional body: what and why, wrapped at 72>

<optional footer: Refs P1-04 · BREAKING CHANGE: …>
```

- **Types:** `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`, `revert`
- **Scopes:** `api`, `web`, `agents`, `pipeline`, `judge`, `db`, `sse`, `ui`, `design`, `ml`, `docs`, `ci`, `deps`
- Examples:
  - `feat(api): stream uploads to disk with per-kind size limits`
  - `feat(agents): add FFT azimuthal spectrum to temporal agent`
  - `fix(sse): avoid duplicate events on reconnect`
  - `docs(api): document /analyses/{id}/frames response`
- Breaking API changes: `feat(api)!: rename score to fake_probability` + `BREAKING CHANGE:` footer + Changelog entry.
- One logical change per commit. Don't mix formatting with logic.

## 4. Pull requests

**Title:** same format as a commit message (it becomes the squash commit).

**Template** (`.github/pull_request_template.md`):

```markdown
## What
<!-- One or two sentences. Link the task: P1-04 -->

## Why

## How to test
<!-- Commands or clicks; include a screenshot/GIF for UI changes -->

## Checklist
- [ ] Tests added/updated and passing locally
- [ ] Lint, format, typecheck pass
- [ ] API.md / Schema.md updated if the contract or schema changed
- [ ] `pnpm gen:api` re-run if the API changed
- [ ] Changelog updated (user-visible change)
- [ ] Tracker updated
- [ ] UI: both themes, 375 px, keyboard, reduced motion checked
- [ ] No secrets, datasets, or large files added
- [ ] Mock results still badged (if touching results UI)
```

**Size:** aim for < 400 changed lines (excluding generated files). Split bigger work into stacked PRs.

## 5. Code review

The reviewer checks, in order:

1. **Correctness** — does it do what the task says; edge cases; errors handled?
2. **Contract** — API/schema docs match; types regenerated?
3. **Rules** — [Rules.md](Rules.md) respected (honesty rules especially)?
4. **Tests** — meaningful, not just coverage padding?
5. **Readability** — would the other person be able to explain this in the viva?

Comment prefixes: `blocker:` (must fix), `suggestion:` (optional), `question:`, `nit:`. Approve when there are no blockers. The author resolves conversations.

## 6. Definition of done

A task is done when: merged to `main`, CI green, tests cover the new logic, docs updated, tracker updated, and the other person has seen it working (demo on Friday counts).

## 7. Using AI coding assistants

- Point the assistant at [Rules.md](Rules.md) and the relevant docs before it writes code (R-41).
- You own every line you commit: read and understand AI-generated code before pushing it. You need to be able to explain it in the viva.
- Never paste API keys, dataset files, or other people's media into an assistant.
- AI-written commits follow the same message and PR conventions.

## 8. Labels

`phase:0` … `phase:9`, `area:api`, `area:web`, `area:agents`, `area:ml`, `area:docs`, `type:bug`, `type:feature`, `priority:P0/P1/P2`, `blocked`, `run-e2e`.

## 9. Repository settings (set up in P0-05)

- `main` protected: PR required, 1 approval, CI must pass, linear history (squash merge only), no force pushes.
- Secret scanning + push protection on.
- Dependabot enabled for pip/uv, npm, GitHub Actions.
