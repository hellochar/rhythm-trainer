# CLAUDE.md

Guidance for working on this repository.

## What this is

A browser rhythm sight-reading trainer. It generates random notated rhythms, plays them on a synthesized cowbell, and leaves a silent repeat of the same bar for the user to play back, optionally scoring their taps. See README.md for user-facing behavior.

## Architecture

The entire app is one file, `index.html`, with inline CSS and vanilla JavaScript. There are no dependencies, no build step, and no framework. Keep it that way unless the user asks otherwise. Deployment is GitHub Pages serving the file as-is.

The script is organized into sections, in this order:

- **Rhythm cell library (`CELLS`)** — rhythm figures grouped by difficulty tier. Each cell fills 1 or 2 beats and holds a list of events `{d, g, rest?, flag?, bs?}` where `d` is duration in beats and `g` is a glyph code (`q`, `e`, `s`, `h`, `dq`, `qr`, `er`). Cells can carry `beam`, `bracket`, and `tn` (tuplet number). `bs` splits a cell into separate beam subgroups.
- **Generation** — `genMeasure(diff, bpb)` fills `bpb` beats from a weighted cell pool; 2-beat cells only start on odd-numbered beats (`pos % 2 === 0`) and only where two beats remain, so odd meters group as 2+2+1; measures regenerate until they have at least 2 onsets. `genRhythm` flattens cells into events with absolute beat times and builds the onset list. The beats per measure ride along on the rhythm as `r.bpb`, and everything downstream reads it from there rather than assuming 4.
- **Notation rendering** — `renderRhythm` builds an SVG string. Layout is time-proportional: `xOf(beat)` maps a beat position to an x coordinate. `setLayout(bpb)`, called at the top of `renderRhythm`, sets the module-level `LB` and `MW` so measure width grows with the meter; since `xOf` reads those, it is only valid for the rhythm most recently rendered. The staff opens with a stacked time signature in the `SIGW` gutter between the opening barline and the first note. Beams, partial (secondary) sixteenth beams, tuplet numbers, and quarter-triplet brackets are drawn per beam group. The SVG contains a `#playhead` line and an empty `#markers` group that scoring appends into.
- **Audio** — Web Audio only. `cowbell` is two square oscillators (562 and 845 Hz) through a bandpass; `click` is a short sine blip. Everything routes through a `master` gain node, and `killAudio` disconnects it, which is how Stop silences already-scheduled sounds. Metronome beats go through `met`, which routes to a `metBus` gain node in front of `master`. They are always scheduled; the checkbox only rides `metBus` between 0 and 1 via `syncMet`. That is what lets the toggle affect beats that were scheduled a bar ago, and it is the pattern to copy for any other option that gates already-scheduled audio.
- **Flow** — `startRun` schedules a count-in of one measure and the first cycle; `scheduleCycle` schedules one rhythm's listen pass plus its optional second-pass sounds and returns `{rhythm, listen, user, end}` times. A `requestAnimationFrame` loop (`loop`) drives phase changes, the playhead, and miss detection by comparing `ctx.currentTime` against the cycle times. In continuous mode, the next rhythm is generated and scheduled while the user pass is still running (`state.pending`), then swapped in at the boundary.
- **Scoring** — real-time. Each tap is matched greedily to the nearest unhit expected onset within a window (`state.win`, derived from the smallest gap between onsets). Markers are drawn immediately via `addMark`. Scoring is gated on the `score my taps` checkbox (`scoringOn()`), off by default.
- **Calibration** — plays ten clicks at 100 bpm, takes the median tap delay, and writes it into the offset field. The offset in milliseconds is subtracted from taps before matching.

## Rules and gotchas

- All timing must use the Web Audio clock (`ctx.currentTime`). Never time playback or scoring with `Date.now`, `performance.now`, or `setTimeout` beyond rough UI scheduling.
- Do not use `localStorage` or `sessionStorage`. Settings live in the DOM controls and reset on reload. This is intentional, because the file also runs inside the Claude artifact viewer where storage APIs fail.
- Schedule audio ahead of time. Anything scheduled at a timestamp in the past plays immediately and sounds wrong. In continuous mode, the pending cycle exists so its audio is scheduled a full bar early.
- Because of that lead time, a checkbox read inside `scheduleCycle` does not affect audio that is already scheduled, so flipping it mid-run looks dead for up to two bars. If a toggle needs to respond immediately, schedule the sound unconditionally and gate it with a gain node, the way the metronome does.
- Beats per measure is not 4. Read it from `r.bpb` (or `state.rhythm.bpb`) rather than hard-coding a bar length, and remember the count-in, the metronome accent, the beat numbers, and the playhead clamp all depend on it.
- Tuplet durations are fractions like `1/3` and `2/7`, so beat positions accumulate floating point error within a cell. Comparisons on beat positions should use tolerances, not equality.
- Partial sixteenth beams: a stub pointing left is drawn only when the sixteenth has no sixteenth neighbor before it. A previous version drew a stub past the last sixteenth's stem in the sixteenth-sixteenth-eighth figure; do not reintroduce that.
- Re-query `#playhead` and `#markers` after any `renderRhythm` call, since rendering replaces the SVG and old element references go stale.
- The Start button doubles as the user gesture that unlocks audio on mobile. Any new entry point that plays sound must also come from a user gesture.

## Style

- Match the existing code: vanilla JS, `const`/arrow style, no classes, sections marked with `/* ---------- name ---------- */` comments.
- Colors come from CSS custom properties on `:root`. Use `var(--...)` in both CSS and inline SVG attributes rather than hard-coded hex values.
- UI copy is plain and literal. Do not add hype words or decorative text.

## Testing

There is no test suite. To check changes:

1. Syntax-check the script block, for example by extracting it and running `node --check`.
2. Open `index.html` in a browser and run one cycle at each difficulty, including Crazy, to confirm notation renders and audio lines up with the playhead.
3. Toggle `score my taps` and confirm the pad, legend, and results appear, taps draw markers in real time, and calibration still fills the offset field.
