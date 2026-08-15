# CLAUDE.md

Guidance for working on this repository.

## What this is

A browser rhythm sight-reading trainer. It generates random notated rhythms, plays them on a synthesized cowbell, and leaves a silent repeat of the same bar for the user to play back, optionally scoring their taps. See README.md for user-facing behavior.

## Architecture

The entire app is one file, `index.html`, with inline CSS and vanilla JavaScript. There are no dependencies, no build step, and no framework. Keep it that way unless the user asks otherwise. Deployment is GitHub Pages serving the file as-is.

The script is organized into sections, in this order:

- **Rhythm figure library (`GROUPS`)** — the figures the user can pick, in display groups. Each cell has a stable `id` (used for the checkbox id `fig-<id>` and for saved settings, so do not rename one casually), a weight `w`, a length `beats`, an optional `on` default, and a list of events `{d, g, rest?, flag?, bs?}` where `d` is duration in beats and `g` is a glyph code (`w`, `h`, `dh`, `q`, `dq`, `e`, `de`, `s`, `wr`, `hr`, `qr`, `er`). Cells can carry `beam`, `bracket`, and `tn` (tuplet number). `bs` splits a cell into separate beam subgroups. `ALL_CELLS` flattens the groups; `selectedCells()` reads the checkboxes and falls back to `[FILL]` (a quarter note) when nothing is checked.
- **Generation** — `genMeasure(bpb, cells)` fills `bpb` beats from the selected cells; a cell may only start where its own length divides the beat position (`pos % c.beats === 0`), so 2-beat cells land on odd-numbered beats and odd meters group as 2+2+1. A beat where nothing fits gets `FILL`, so a selection that cannot fill a bar degrades instead of throwing. Measures regenerate until they have at least 2 onsets, and after 30 tries the attempt with the most onsets wins, so an all-rests selection still returns a bar. `genRhythm(measures, bpb, cells)` flattens cells into events via `cellEvents` and builds the onset list. The beats per measure ride along on the rhythm as `r.bpb`, and everything downstream reads it from there rather than assuming 4.
- **Notation rendering** — `renderRhythm` builds the staff (barlines, time signature, beat numbers) and calls `drawEvents(events, xf, y)` for the notation itself; `cellPreview` calls the same `drawEvents` with its own x-mapper to draw the little figures in the library, so glyph work only has to happen once. Layout is time-proportional: `xOf(beat)` maps a beat position to an x coordinate. `setLayout(bpb)`, called at the top of `renderRhythm`, sets the module-level `LB` and `MW` so measure width grows with the meter; since `xOf` reads those, it is only valid for the rhythm most recently rendered. The staff opens with a stacked time signature in the `SIGW` gutter between the opening barline and the first note. Beams, partial (secondary) sixteenth beams, tuplet numbers, and quarter-triplet brackets are drawn per beam group. The SVG contains a `#playhead` line and an empty `#markers` group that scoring appends into.
- **Figure library UI** — `buildFigs` generates the checkbox grid from `GROUPS` (each option is `#opt-<id>` wrapping `#fig-<id>`), and `syncFigs` keeps the dimmed state and the count in the summary current. The `×` on a group clears it, or selects all of it when it is already empty.
- **Audio** — Web Audio only. `cowbell` is two square oscillators (562 and 845 Hz) through a bandpass; `click` is a short sine blip. Everything routes through a `master` gain node, and `killAudio` disconnects it, which is how Stop silences already-scheduled sounds. In front of `master` sit two buses: cowbells go through `bellBus` and metronome beats (`met`) through `metBus`, both driven by `syncVol` from the volume sliders and the metronome checkbox. Metronome beats are always scheduled and only gated by that gain, which is what lets the toggle and the sliders affect sound that was scheduled a bar ago; it is the pattern to copy for any other option that gates already-scheduled audio. The count-in and tap clicks stay on `master` so they are always audible.
- **Flow** — `startRun` schedules a count-in of one measure and the first cycle; `scheduleCycle` schedules one rhythm's listen pass plus its optional second-pass sounds and returns `{rhythm, listen, user, end}` times. A `requestAnimationFrame` loop (`loop`) drives phase changes, the playhead, and miss detection by comparing `ctx.currentTime` against the cycle times. In continuous mode, the next rhythm is generated and scheduled while the user pass is still running (`state.pending`), then swapped in at the boundary; `nextRhythm` returns the same bar again while `replay` is checked. `state.prev` holds the bar before the current one, which is what Retry reaches for.
- **Retry** — `retryTarget` picks the bar to loop: the one you just played (`state.prev`), or the current one once you are past halfway through your own pass. `retryRun` ticks `replay`, installs that bar, and restarts with `single` forced false so it loops regardless of the continuous setting.
- **Settings persistence** — `saveSettings`/`loadSettings` write every `input`/`select` with an id, plus the open state of each `details`, to one localStorage key. Both are wrapped in try/catch because storage throws in the artifact viewer, where the app then simply starts from defaults. Load runs after `buildFigs` so the figure checkboxes exist to restore into.
- **Scoring** — real-time. Each tap is matched greedily to the nearest unhit expected onset within a window (`state.win`, derived from the smallest gap between onsets). Markers are drawn immediately via `addMark`. Scoring is gated on the `score my taps` checkbox (`scoringOn()`), off by default.
- **Calibration** — plays ten clicks at 100 bpm, takes the median tap delay, and writes it into the offset field. The offset in milliseconds is subtracted from taps before matching.

## Rules and gotchas

- All timing must use the Web Audio clock (`ctx.currentTime`). Never time playback or scoring with `Date.now`, `performance.now`, or `setTimeout` beyond rough UI scheduling.
- Settings persist through `localStorage`, but every access must stay inside a try/catch. The file also runs inside the Claude artifact viewer, where storage APIs throw, and the app has to keep working there with defaults.
- Schedule audio ahead of time. Anything scheduled at a timestamp in the past plays immediately and sounds wrong. In continuous mode, the pending cycle exists so its audio is scheduled a full bar early.
- Because of that lead time, a checkbox read inside `scheduleCycle` does not affect audio that is already scheduled, so flipping it mid-run looks dead for up to two bars. If a toggle needs to respond immediately, schedule the sound unconditionally and gate it with a gain node, the way the metronome does.
- Beats per measure is not 4. Read it from `r.bpb` (or `state.rhythm.bpb`) rather than hard-coding a bar length, and remember the count-in, the metronome accent, the beat numbers, and the playhead clamp all depend on it.
- Tuplet durations are fractions like `1/3` and `2/7`, so beat positions accumulate floating point error within a cell. Comparisons on beat positions should use tolerances, not equality.
- Partial sixteenth beams: a stub is drawn only for a sixteenth with no sixteenth on either side, and it points into the group, so it goes right when the sixteenth opens the group and left otherwise. Adjacent sixteenths get a full secondary beam instead. Earlier versions drew a stub past the last sixteenth's stem in the sixteenth-sixteenth-eighth figure, and overhung the first sixteenth's stem in eighth-sixteenth-sixteenth; do not reintroduce either.
- The generator is driven entirely by what the user has checked, so it must survive selections that cannot fill a bar, or that are empty, or that are all rests. Never assume a candidate exists at a given beat position.
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
