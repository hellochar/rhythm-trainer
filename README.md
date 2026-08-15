# Rhythm Trainer

Rhythm sight-reading trainer that runs in the browser. It shows a randomly generated one or two measure rhythm in music notation, plays it once on a synthesized cowbell, and then the same bar comes around again for you to play it back. You can play it back in your head, on a drum practice pad, or by tapping into the app with scoring turned on. In continuous mode, a new rhythm follows on the very next beat and the pulse never stops.

The whole app is one file, `index.html`, with no dependencies and no build step.

## Running it

Open `index.html` in a browser, or visit the GitHub Pages URL for this repository. Press **Start ▶** to begin. Browsers require a tap or click before they allow audio, and the Start button satisfies that.

## How a cycle works

1. A count-in bar plays, one click per beat of the current meter (first cycle only).
2. The rhythm plays on cowbell while a gold playhead sweeps the notation.
3. The same bar repeats in silence while a teal playhead sweeps. This is your pass.
4. In continuous mode, the next rhythm appears and begins on the next beat.

## Settings

- **Tempo** sets the beats per minute.
- **Length** chooses one or two measures per rhythm.
- **Time** sets the time signature: 2/4, 3/4, 4/4, 5/4, 6/4, or 7/4. The quarter note is the beat in all of them, so the tempo means the same thing in every meter. The staff shows the signature, the count-in runs one bar of it, and the metronome accents beat 1. Changing it mid-run takes effect on the next rhythm.
- **Difficulty** picks the pool of rhythm figures. Each tier includes the ones below it.
  - *Basic*: quarter notes, eighth pairs, quarter rests.
  - *Standard*: adds half notes, dotted quarter plus eighth, and offbeat eighths.
  - *Spicy*: adds sixteenth-note figures.
  - *Triplets*: adds eighth-note triplets, sixteenth-note triplets, and quarter-note triplets over two beats.
  - *Crazy*: adds five sixteenths over one beat, five eighths over two beats, and seven eighths over two beats.
- **continuous** keeps generating new rhythms until you press Stop. Unchecked, one rhythm plays once and stops at results.
- **cowbell on my pass** plays the rhythm again, quieter, during your pass so you can hear your offset against it.
- **metronome** plays a quarter-note click through both passes, accented on beat 1. It can be turned on and off during a run and takes effect on the next beat.
- **score my taps** turns on tap tracking. It is off by default.
- **sound on my taps** plays a soft click on each tap when scoring is on.

## Scoring

With **score my taps** on, a tap pad appears and the space bar also registers taps. Each tap is matched to the nearest unplayed note within a timing window, and a colored dot with the error in milliseconds appears under that note as you play. Green means within 40 ms, amber within 90 ms, and red beyond that. Missed notes get a hollow ring, and extra taps get an ✕. The matching window shrinks automatically when notes are closer together.

## Input delay calibration

Keyboards, touchscreens, and audio output all add latency, so raw tap timestamps land late. The calibration panel at the bottom handles this. Press **Run tap test** and tap along with ten steady clicks. The median delay of your taps becomes the offset, and that offset is subtracted from your taps before scoring. You can also type an offset by hand. Negative values mean you tend to tap early.

## Technical notes

- All timing uses the Web Audio clock (`AudioContext.currentTime`), not `setTimeout` or `Date.now`, so playback and scoring stay sample-accurate.
- The cowbell is two detuned square oscillators through a bandpass filter, in the style of the 808 cowbell.
- The notation is hand-drawn SVG rather than a music font, so rests and flags look stylized.
- In continuous mode, the next rhythm's audio is scheduled while you play the current one, which is how the transition lands with no gap.
