# Design System

The look we're going for: **a forensic lab at night.** Dark, calm, precise — deep ink surfaces, thin luminous lines, instrument-like readouts, and motion that feels like measurement rather than decoration. Every visual effect should read as "the system is examining something", never as a gimmick.

Dark theme is primary (demo and projector); a light theme is supported and must pass the same contrast checks.

## 1. Principles

1. **Evidence over decoration.** Every glow, chart, and animation carries information (a score, a state, a location). If it doesn't, cut it.
2. **Show the work.** The war room makes the agents' process visible; results always link a claim to its evidence.
3. **Honest confidence.** Uncertainty is a first-class state with its own colour, never hidden. Mock data is always badged.
4. **Calm by default, dramatic at the moment of truth.** Quiet UI while working; one purposeful animation at the verdict reveal.
5. **Accessible.** WCAG 2.2 AA contrast, keyboard-complete, colour is never the only signal (always icon + label), `prefers-reduced-motion` respected.

## 2. Color tokens

Defined as CSS variables in `apps/web/src/styles/globals.css` and exposed to Tailwind v4 via `@theme`. Components use **semantic tokens only** — never raw hex.

### 2.1 Neutrals and surfaces

| Token | Dark | Light | Use |
|---|---|---|---|
| `--bg` | `#07090F` | `#F7F8FB` | Page background |
| `--bg-grid` | `rgba(148,163,184,0.06)` | `rgba(15,23,42,0.05)` | Subtle 32 px grid pattern |
| `--surface-1` | `#0D111A` | `#FFFFFF` | Cards |
| `--surface-2` | `#131926` | `#F1F4F9` | Raised panels, inputs |
| `--surface-glass` | `rgba(19,25,38,0.72)` + `backdrop-blur(14px)` | `rgba(255,255,255,0.75)` | Top bar, overlays |
| `--border` | `rgba(148,163,184,0.14)` | `rgba(15,23,42,0.10)` | Hairlines |
| `--border-strong` | `rgba(148,163,184,0.28)` | `rgba(15,23,42,0.20)` | Focused/hovered borders |
| `--text` | `#E6EAF2` | `#0B1220` | Primary text |
| `--text-muted` | `#94A3B8` | `#475569` | Secondary text |
| `--text-faint` | `#64748B` | `#64748B` | Labels, axes |

### 2.2 Brand and semantic colours

| Token | Dark | Light | Meaning |
|---|---|---|---|
| `--brand` | `#8B5CF6` (violet) | `#7C3AED` | Primary actions, focus rings, active nav |
| `--brand-glow` | `rgba(139,92,246,0.35)` | `rgba(124,58,237,0.20)` | Glows on primary elements |
| `--real` | `#2DD4BF` (teal) | `#0F766E` | Authentic verdict / low P(fake) |
| `--fake` | `#FB7185` (rose) | `#BE123C` | Manipulated verdict / high P(fake) |
| `--uncertain` | `#FBBF24` (amber) | `#B45309` | Uncertain verdict, warnings |
| `--info` | `#60A5FA` | `#1D4ED8` | Neutral info, links |
| `--danger` | `#F87171` | `#B91C1C` | Errors, destructive actions |
| `--mock` | `#F59E0B` striped | `#B45309` | MOCK badge |

Teal vs. rose was chosen over green vs. red because it stays distinguishable under the common forms of colour blindness; every verdict is also labelled with text and an icon (✓ shield for real, ⚠ split-face for fake, ? for uncertain).

### 2.3 Agent identity colours

Used for agent cards, timeline lanes, and chart series:

| Agent | Token | Dark | Icon (lucide) |
|---|---|---|---|
| Visual Forensics | `--agent-visual` | `#A78BFA` | `scan-face` |
| Temporal & Frequency | `--agent-temporal` | `#38BDF8` | `activity` |
| Audio | `--agent-audio` | `#F472B6` | `audio-waveform` |
| Judge | `--agent-judge` | `#FDE68A` | `scale` |

### 2.4 Score scale

A continuous diverging scale for P(fake): `--real` (0.0) → `--text-faint` (0.5) → `--fake` (1.0), interpolated in OKLCH. Used for frame-score bars, heatmap legends, and table cells. Heatmaps use the perceptually uniform **inferno** colormap with a legend.

## 3. Typography

| Role | Font | Weights | Notes |
|---|---|---|---|
| UI / body | **Geist Sans** | 400, 500, 600 | Via `next/font` |
| Numbers, code, logs | **Geist Mono** | 400, 500 | `font-variant-numeric: tabular-nums` for all scores |
| Display (hero, verdict word) | **Instrument Serif** (italic) | 400 | Used sparingly: hero headline, verdict label, section intros on `/about` |

Scale (rem): `xs 0.75 · sm 0.875 · base 1 · lg 1.125 · xl 1.25 · 2xl 1.5 · 3xl 1.875 · 4xl 2.25 · display 3.5–5 (clamp)`. Line height 1.5 for body, 1.15 for display. Scores are shown with 2–3 decimals in mono, or as percentages with one decimal (`91.2%`).

## 4. Space, shape, elevation

- **Spacing:** 4 px base (`1 = 4px`). Card padding 20–24 px. Page gutter 16 px mobile, 32 px desktop. Max content width 1440 px.
- **Radius:** `--radius-sm 6px` (chips), `--radius 12px` (cards, inputs), `--radius-lg 20px` (hero panels, dialogs).
- **Elevation:** no heavy drop shadows. Depth comes from surface steps + hairline borders + a 1 px inner top highlight (`inset 0 1px 0 rgba(255,255,255,0.04)`). Active/running elements get a soft outer glow in their semantic colour.
- **Background:** `--bg` + faint grid + a very low-opacity noise texture (SVG, 2% opacity) + one or two blurred radial "aurora" gradients (brand violet, teal) behind the hero and war room only.

## 5. Motion

| Token | Value | Use |
|---|---|---|
| `--dur-fast` | 120 ms | Hover, press |
| `--dur-base` | 220 ms | Panels, tabs, chips entering |
| `--dur-slow` | 420 ms | Page transitions, verdict gauge |
| `--ease-out` | `cubic-bezier(0.16, 1, 0.3, 1)` | Default |
| Spring | `stiffness 260, damping 30` | Gauge needle, cards reordering |

Signature motions:

- **Scan line**: while an agent is running, a thin gradient line sweeps down its card and over the media preview (3 s loop). Stops when done.
- **Streaming log**: new lines fade/slide in 4 px; auto-scroll unless the user has scrolled up.
- **Live timeline**: bars grow from the baseline as `agent.frame_score` events arrive.
- **Verdict reveal**: gauge needle springs to the value, the arc fills in the verdict colour, the label stamps in (scale 1.08 → 1, 220 ms), a single soft pulse ring. Total ≤ 1.2 s.
- **Judge typing**: tokens rendered per animation frame with a blinking caret.

`prefers-reduced-motion: reduce` → no scan lines, no pulses, no parallax; state changes crossfade in 120 ms; the gauge jumps to its value.

## 6. Components

Built on shadcn/ui primitives (Button, Card, Dialog, Tabs, Tooltip, Table, Slider, Switch, DropdownMenu, Sheet, Skeleton, Toast/Sonner, Command). Custom forensic components live in `components/forensics/`.

<a id="verdict-gauge"></a>
### VerdictGauge

A 240° arc gauge. The track is the diverging score scale at 20% opacity; the filled arc uses the verdict colour; the needle points at P(fake). The centre shows the verdict word in Instrument Serif italic, P(fake) in mono beneath it, then confidence as a thin secondary ring. Thresholds (0.35 / 0.65) are marked as ticks. Sizes: `sm` 120 px (tables, cards), `lg` 280 px (results).

### AgentCard

Header: agent icon in its identity colour, name, backend badge. Body: status (progress ring while running, score + mini-bar when done), latest log line, sparkline of frame scores, finding chips. States:

| State | Visual |
|---|---|
| queued | Dimmed, dashed border, "Waiting" |
| running | Identity-colour glow border, scan line, progress ring |
| completed | Solid border, score pill coloured by score scale |
| skipped | Muted, reason text, no score |
| failed | `--danger` border, error message, "details" disclosure |

### EvidenceTimeline

Horizontal lanes (one per agent) under the video player, sharing a time axis. Each lane draws frame-score bars coloured by the score scale; finding segments are drawn as rounded ranges with severity outlines. The playhead is synced both ways with the player (click to seek, play to move). Keyboard: ←/→ step one sampled frame.

### HeatmapViewer

Frame or face crop with a heatmap overlay; opacity slider (0–100%), toggle "crop / full frame", legend. While the detector is mock, a "Mock saliency" watermark sits bottom-right.

### SpectrumViewer

Side-by-side: log-magnitude 2D FFT of the selected face crop and the azimuthally averaged power spectrum curve compared with a reference band from real faces. The high-frequency region is shaded.

### WaveformPlayer

wavesurfer.js waveform with suspicious regions in `--fake` at 25% opacity, spectrogram below, play/pause, and region list.

### Others

`MockBadge` (amber, diagonal stripes, tooltip: "Scores come from a placeholder detector — not a trained model"), `BackendBadge`, `StatTile` (label, big mono number, delta, sparkline), `FindingCard`, `ConfidenceBar`, `AgreementIndicator` (three dots: consensus / mixed / conflict), `UploadDropzone`, `MediaMetaCard`, `ReportMarkdown` (styled prose, evidence references become links), `EmptyState`, `ErrorState`, `ApiStatusDot`.

## 7. Page-level layouts

- **Landing (`/`)**: full-bleed hero. Left: headline in Instrument Serif italic ("See through the synthetic."), a one-line subhead, CTA buttons. Right: a slowly rotating point-cloud face (react-three-fiber, ~6k points) that "scans" with a horizontal light band; points shift from teal to rose where the band passes a manipulated region. Below: the 4 agents as cards, a 3-step "how it works", live stats from `/metrics/overview`. The 3D canvas is lazy-loaded; a static SVG poster renders first.
- **Dashboard**: 4 stat tiles → 2-column grid (activity chart 2/3, verdict donut 1/3) → recent analyses + agent health.
- **War room / results**: see [Appflow.md §3–4](Appflow.md#3-the-war-room-analysesid-while-running). At < 1024 px, the player stacks above the agent grid (2×2 → 1 column at < 640 px); the log collapses into a drawer.

## 8. States checklist (every data view)

Loading → skeletons matching the final layout (no spinners for more than 300 ms of content). Empty → illustration + one clear action. Error → what happened, what to do, request ID. Partial → render what exists and mark what's missing. Mock → badge.

## 9. Accessibility

- Contrast: text ≥ 4.5:1, large text and UI parts ≥ 3:1 in both themes (checked in CI with axe via Playwright).
- Focus: 2 px `--brand` ring with 2 px offset on all interactive elements.
- The war room has an `aria-live="polite"` region announcing agent completions and the final verdict (not every token).
- Charts have text alternatives: a summary sentence plus a "view as table" toggle.
- Hit targets ≥ 40 × 40 px; everything reachable by keyboard; Esc closes dialogs; ⌘/Ctrl+K palette.

## 10. Assets and iconography

- Icons: lucide, 1.5 px stroke, 16/20/24 px.
- Illustrations: simple line art in `--text-faint` with one accent colour.
- Favicon / logo: a stylised eye made of scan lines, violet → teal gradient. SVG only.

## 11. Design workflow

1. New screen → a low-fidelity layout in this doc or a Figma frame → review together → build.
2. New colour or size → add a token here first, then use it.
3. Before merging UI: check both themes, 375 px width, keyboard navigation, reduced motion.
