# Rhythm Trainer

Rhythm sight-reading trainer that runs in the browser. It shows a randomly generated one or two measure rhythm in music notation, plays it once on a synthesized cowbell, and then the same bar comes around again for you to play it back. You can play it back in your head, on a drum practice pad, or by tapping into the app with scoring turned on. In continuous mode, a new rhythm follows on the very next beat and the pulse never stops.

The whole app is one file, `index.html`, with no dependencies and no build step.

## Running it

Open `index.html` in a browser, or visit the GitHub Pages URL for this repository. Press the **▶** in the middle of the tempo dial to begin, and the same button stops it again. It plays the bar already on the staff — nothing reshuffles it behind your back. Browsers require a tap or click before they allow audio, and that button satisfies that. While a run is going the app holds a screen wake lock where the browser supports one, so a phone left alone on a music stand does not dim and sleep between bars.

## How a cycle works

1. A count-in bar plays, one click per beat of the current meter, the beat number flashing above the staff (first cycle only).
2. The rhythm plays on cowbell. With **moving line** on, a gold playhead sweeps the notation.
3. The same bar repeats in silence, the playhead now teal. This is your pass.
4. In continuous mode, the next rhythm appears and begins on the next beat.

The word above the staff tells you which pass you are in. At rest it says nothing at all.

## Retry

**Retry ↺** puts the bar you just missed back in front of you and loops it until you switch it off. It picks the bar you last played, or the one you are playing now if you are more than halfway through your own pass, so you can reach for it the moment a bar goes wrong without having to catch it in time. It also switches on the loop button in the top-right corner of the staff; press that again to go back to fresh rhythms.

## Rhythm figures

The **Rhythm figures** panel lists every figure the generator can use, grouped by kind, each with its notation. A figure has three states, and tapping it moves to the next one:

- **off**, dimmed — never used.
- **selected**, a teal dot — in the pool the generator draws from.
- **guaranteed**, a brass dot with a glow and a light chasing its border — reserved a slot in every rhythm, so it is certain to come up.

Guaranteed figures are how you drill one thing without giving up variety around it: guarantee the sixteenth triplet, leave a few plain figures selected, and every rhythm will contain that triplet and something different around it. Each guaranteed figure claims one random slot, in one measure picked at random, so on a two-measure rhythm it turns up once across the pair rather than twice. If several of them cannot all fit, or one is longer than the bar, whatever does not fit is quietly left out rather than breaking the rhythm.

The **×** on a group clears it, or selects all of it when it is already empty. With nothing selected at all, bars fall back to plain quarter notes.

A figure is only placed where its length divides the beat position, so two-beat figures start on odd-numbered beats and three-beat figures on beats 1 and 4. Figures longer than the bar (a whole note in 3/4, say) are simply never placed, and a beat that nothing fits gets a quarter note. Bars are regenerated until they hold at least two notes, so a selection of only long or silent figures can still yield a sparse bar.

While nothing is playing, the staff previews what your current settings produce. It rerolls when you change something that changes what can be generated — the figures, the meter, the length — and stays put for everything else, so a nudge of a volume slider or a press of Start does not throw away the bar you were reading.

A two-measure rhythm sits on one staff on a wide window and stacks onto two rows when the window is too narrow to read it, which is what phones get.

## Settings

- **The tempo dial** sets the beats per minute, from 40 to 200. Turn the dial anywhere on its face to sweep the tempo, tap **‹** and **›** for one beat at a time (hold them to run), or — where there is a keyboard — click the number and type one. The current tempo is on the dial itself and the teal ring shows where it sits in the range. A change takes effect immediately: whatever is playing bends to the new tempo without stopping or losing its place, count-in included.
- **tap tempo**, under the dial, sets the tempo from your own tapping. Tap it a few times in time and the dial follows the spacing of the last few taps; leave off for a couple of seconds and the next tap starts a new measurement.
- **Length** chooses one or two measures per rhythm.
- **Time** sets the time signature: 2/4, 3/4, 4/4, 5/4, 6/4, or 7/4. The quarter note is the beat in all of them, so the tempo means the same thing in every meter. The staff shows the signature, the count-in runs one bar of it, and the metronome accents beat 1. Changing it mid-run takes effect on the next rhythm.
- **continuous** keeps generating new rhythms until you press Stop. Unchecked, one rhythm plays once and stops at results.
- **The loop button** in the top-right corner of the staff repeats the current bar instead of moving on to a new one. Retry turns it on for you.
- **moving line** shows the playhead sweeping the notation. It runs from the opening barline to the closing one at a steady rate and picks up on the next row when the staff is stacked, so it only ever jumps where the music itself goes back to the top. It is off by default, so nothing moves unless you ask for it; it can be toggled during a run.
- **The two sound rows** — a cowbell for the rhythm and a metronome for the click — each have an icon and a volume slider. Tapping an icon mutes that sound outright and the row dims; moving its slider brings it back. Both are live: they reach sounds that were already scheduled, so a change is audible on the next note. Both scales mean the same thing, so 100 on one is as loud as 100 on the other. The metronome is on by default at 30, so it sits under the cowbell. On a wide enough window the rows are labelled `rhythm` and `metronome`.
- **cowbell on my pass** plays the rhythm again, quieter, during your pass so you can hear your offset against it.
- **score my taps** turns on tap tracking, and reveals the tap pad, the legend, the results line and the calibration panel. It is currently commented out in `index.html` while the app is being tried without it; uncommenting that one line brings all of it back.
- **sound on my taps** plays a soft click on each tap when scoring is on.

## Scoring

Tap scoring is commented out at the moment, so the controls below are not on the page. What follows describes it as it works when the **score my taps** line in `index.html` is uncommented.

With **score my taps** on, a tap pad appears and the space bar also registers taps. Each tap is matched to the nearest unplayed note within a timing window, and a colored dot with the error in milliseconds appears under that note as you play. Green means within 40 ms, amber within 90 ms, and red beyond that. Missed notes get a hollow ring, and extra taps get an ✕. The matching window shrinks automatically when notes are closer together.

## Saved settings

Every control, including the figure selection and the volume sliders, is written to browser local storage and restored on the next visit. Nothing is sent anywhere. If storage is unavailable the app still runs, it just starts from the defaults each time.

## Input delay calibration

Keyboards, touchscreens, and audio output all add latency, so raw tap timestamps land late. The calibration panel at the bottom handles this, and it only appears when **score my taps** is on, since that is the only thing the offset affects. Press **Run tap test** and tap along with ten steady clicks. The median delay of your taps becomes the offset, and that offset is subtracted from your taps before scoring. You can also type an offset by hand. Negative values mean you tend to tap early.

## Technical notes

- All timing uses the Web Audio clock (`AudioContext.currentTime`), not `setTimeout` or `Date.now`, so playback and scoring stay sample-accurate. Tap tempo is measured on the same clock.
- Changing the tempo mid-run works out where the music has got to in beats, drops the schedule, and lays it down again from that same position at the new beat length, fading the old graph out over 20 ms so the swap does not click.
- The cowbell is two detuned square oscillators through a bandpass filter, in the style of the 808 cowbell.
- The cowbell and the metronome each run through their own gain node, which is why the volume sliders and the mute buttons affect sounds that were scheduled a bar in advance. Sounds are written at full scale and a single output gain provides the headroom, so equal slider positions really are equal loudness.
- The notation is hand-drawn SVG rather than a music font, so rests and flags look stylized.
- In continuous mode, the next rhythm's audio is scheduled while you play the current one, which is how the transition lands with no gap.
- The screen wake lock uses the Screen Wake Lock API where it exists, and is simply skipped where it does not. It is taken when a run starts, dropped when the run stops or ends, and re-taken when you come back to the tab.
