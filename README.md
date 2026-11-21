# mini-inkling-2-cow-test

🎥 **Story:** [Watch it live](https://joeppie.github.io/mini-inkling-2-cow-test/) · [play locally](index.html)

> [!NOTE]
> **Redaction note.** The benchmark section below is a redacted rendering. Every figure is public benchmark data (success rates, tokens, wall times, turns, first-token latency, decode speed, tool errors) and every machine-specific detail has been stripped — no device models, GPU/CPU names or memory footprints, no ports or protocols, no artifact paths or directories, no seed or engine-config internals. Where any detail differs from the source, it is missing, not wrong. The story section is complete and unredacted; nothing in the story contains machine data at all.

## The story

**Greta** walks past the fence of her pasture, meets a lost calf in the next county, walks through the shallows with it, and comes home. Ten segments, each a self-contained inline SVG (viewBox 960×540, aspect-fit letterbox).

### Spec

- **Segments:** Title → The Notice → The Walk Out → The Meadow → The Wood → The Lost Calf → The Dog → The Crossing → The Return → Home
- **Animation:** walk cycle (leg swing + body bob) in two scenes, tail sway, standing bobbing, shivering calf, scrolling meadow background, grass sway, drifting clouds, twinkling stars, flapping bird, falling leaves, animated creek ripples, moon glow, breathing farmer, moonlit dusk.
- **Dialogue bubbles** are placed procedurally, never hand-placed. A canvas text measurement sets the box width; a priority-ordered candidate search then enforces three rules:
  1. The box stays inside the frame (margins from the viewBox edges),
  2. The box does not overlap any character on the scene (the speaker included),
  3. The box does not overlap any other bubble visible at the same time.
  The bubble's tail points at the speaker's position (nearest edge). If no candidate passes, the least-bad position is used and still never leaves the frame.
- **Transitions:** iris circle closes/opens with a glowing ring pulse and a white flash at the hand-off.
- **Controls:** auto-advance, pause/play, prev/next, restart, clickable segment dots, keyboard (← / → / space), progress bar, `#s=N` deep-link to a specific scene (pins the scene, animations keep running), `prefers-reduced-motion` support.

### How to view

Open `index.html` directly, or visit the published site (GitHub Pages). Append `#s=N` to the URL to start at segment `N` (1–10): `#s=1` is the Title, `#s=3` is the Walk Out.

---

## Benchmark comparison

<img src="assets/agentic-compare.png" alt="Inkling 2 Mini vs Swift 1.5-NVFP4 agentic benchmark card" width="100%">

**Same harness · same served model · 5 tasks × 4 samples = 20 runs per arm · one client at a time · warm prompt cache.**

Success = the harness re-running the pristine verifier with exit 0 — never the model's own success claim. 95% confidence intervals are Wilson intervals.

| Metric | Inkling 2 Mini | Swift 1.5-NVFP4 | Wins |
| --- | --- | --- | --- |
| Success | 16 / 20 (0.80 · CI 0.58–0.92) | 18 / 20 (0.90 · CI 0.70–0.97) | Swift |
| Median completion tokens | 1,938 (min 371 · max 12,962) | 3,326 (min 634 · max 11,713) | Inkling |
| Median wall per run | 5.4 s (min 1.2 · max 33.0 s) | 29.9 s (min 7.9 · max 87.5 s) | Inkling |
| Median turns | 6.5 (range 4–12) | 8.0 (range 4–12) | Inkling |
| Median TTFT (first turn) | 0.09 s (range 0.09–0.10 s) | 0.94 s (range 0.88–1.04 s) | Inkling |
| Tool errors | 0 (total per 20 runs) | 2 (total per 20 runs) | Inkling |
| Dataops success | 0 / 4 (CI 0.00–0.49) | 2 / 4 (CI 0.15–0.85) | Swift |
| Median decode (bench window) | 484 t/s (234.5–670.0) | 136 t/s (10.3–208.8) | Inkling |
| Runs hitting the 12-turn cap | 4 (dataops 4) | 7 (dataops 3 · refactor 3 · trace 1) | Inkling |

### Per-task

| Task | Inkling 2 Mini (success · med tokens · med wall · turns) | Swift 1.5-NVFP4 (success · med tokens · med wall · turns) |
| --- | --- | --- |
| bugfix | 4 / 4 · 564 · 1.5 s · 4.5 | 4 / 4 · 889 · 10.9 s · 4.5 |
| dataops | 0 / 4 · 4,897 · 15.5 s · 12.0 | 2 / 4 · 5,077 · 50.9 s · 12.0 |
| implement | 4 / 4 · 1,938 · 4.7 s · 4.0 | 4 / 4 · 2,026 · 18.1 s · 5.5 |
| refactor | 4 / 4 · 9,458 · 23.4 s · 8.5 | 4 / 4 · 10,720 · 83.8 s · 12.0 |
| trace | 4 / 4 · 1,902 · 5.2 s · 6.5 | 4 / 4 · 2,430 · 24.0 s · 9.0 |

### How they ran

Same engine and same harness: 5 coding tasks (a contract bug, implementing primes.py including the 10001st prime, distinct-5xx analysis in a noisy log, a CSV report refactor, a 3-file KeyError + slicing trace) × 4 samples = 20 runs per arm, seeded task order, one client at a time, warm prompt cache. Both arms run the same per-run caps (token budget per turn, a 12-turn cap, a 600-second wall cap, and a shell/verify cap) — identical for both arms, unchanged packs, warm server. Quantization and engine features differ per arm as designed; nothing about the harness changed between arms.

---

## Provenance

This repository was not produced in a single pass — it is the product of an interactive, back-and-forth working session between **Joep Lijnen (@Joeppie)** and an agentic coding assistant, across several turns of review and rework. The honest account of that process:

**Round 1 — the story itself.** A ~2,100-word tale about a cow that walks out of her pasture and meets a lost calf. It was then turned into an animated SVG product: a 10-segment sequence (Title → The Notice → The Walk Out → The Meadow → The Wood → The Lost Calf → The Dog → The Crossing → The Return → Home) with a warp-style iris transition, dialogue bubbles, and an auto-advance/pause toolbar.

**Round 2 — review and rework.** The first render showed the cow with poorly-fitting art and unreadable balloons: two symptoms, one root cause — CSS transforms on the same element as its positioning attribute made every balloon jump to the origin. The cow was redrawn (two eyes, hooves, horns, ears, udder, tuft tail), the bubble structure was fixed, and the font sizes were raised. The user then reported that the live view was stuttering because a game was running in parallel — at which point verification moved from screenshots to real-time DevTools.

**Round 3 — the procedural placement procedure.** The review ended with a firm ask: stop verifying with headless-Chrome screenshots and instead *calculate* where every balloon lands. The build now computes every box from the scene's own facts — canvas text measurement, a safe frame, a priority-ordered candidate search (above the speaker, frame-clamped, side-of, below, corner), an anti-overlap rule against the speaker's footprint and any simultaneously-visible balloons, and a tail that always points at the speaker. Because `slice` cropping could silently hide content near the viewBox edge, the story now uses a letterbox (`meet`) so nothing is ever hidden by the layout.

**Round 4 — identity.** The commit initially carried a persona name, which the user flagged. The commit was rewritten to carry their real GitHub identity — name of the public profile with GitHub's noreply email, so no personal contact details appear on the public page.

**Round 5 — publishing.** A redacted `index.html` + this README + a redacted benchmark card were committed, published with `gh`, and enabled as GitHub Pages.

### The prompt (verbatim)

> *"Once completed place a redacted (anonimized) version of the html together with a markdown readme, containing the specs and the earlier session created PNG comparing swift and inkling models to a new github repo (with pages) called mini-inkling-2-cow-test, include this prompt and the timings/statistics, but strip any information regarding computer-specific paths or any other sensitive information."*

### Tools used

- **Node.js** — real-time DevTools-protocol probes against headless Chromium (frame checks, bubble timings, walk-cycle tracking, page errors), and a compute-loop over the placement procedure against every scene's declared facts.
- **Python 3 + Pillow** — re-rendered the redacted benchmark card from the raw comparison data.
- **Headless Chromium** — real-time captures and layout checks (frozen-animation screenshots were rejected as a verification method).
- **gh CLI** — repo creation and GitHub Pages.

**Honest limits.** The story's animations (walk cycle, sway, ripples) are standard CSS and are verified by eye and in real-time rendering, not by any static figure — the point of the placement procedure is that no *placement* fact depends on visual luck. The README describes the process as it ran; the earlier version of this section, which presented the work as a single pass, was mistaken and is preserved here as a correction, not a polish step.

## Files

- `index.html` — the animated story (self-contained, no dependencies).
- `assets/agentic-compare.png` — redacted benchmark card.
- `README.md` — this file.
