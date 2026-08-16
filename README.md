# Rhythm Trainer

Rhythm sight-reading trainer that runs in the browser. It shows a randomly generated one or two measure rhythm in music notation, plays it once on a synthesized cowbell, and then the same bar comes around again for you to play it back. You can play it back in your head, on a drum practice pad, or by tapping into the app with scoring turned on. A new rhythm follows on the very next beat and the pulse never stops until you press stop.

The whole app is one file, `index.html`, with no dependencies and no build step.

## Running it

Open `index.html` in a browser, or visit the GitHub Pages URL for this repository. Press the **▶** in the middle of the tempo dial to begin, and the same button stops it again. Each start deals a fresh rhythm unless the loop button is on. Browsers require a tap or click before they allow audio, and that button satisfies that. While a run is going the app holds a screen wake lock where the browser supports one, so a phone left alone on a music stand does not dim and sleep between bars.

## How a cycle works

1. A count-in bar plays, one click per beat of the current meter, the beat number flashing above the staff (first cycle only).
2. The rhythm plays on cowbell. With **moving line** on, a gold playhead sweeps the notation.
3. The same bar repeats in silence, the playhead now teal. This is your pass.
4. The next rhythm appears and begins on the next beat, and round it goes again.

The word above the staff tells you which pass you are in. At rest it says nothing at all.

## Retry

**Retry ↺** puts the bar you just missed back in front of you and loops it until you switch it off. It picks the bar you last played, or the one you are playing now if you are more than halfway through your own pass, so you can reach for it the moment a bar goes wrong without having to catch it in time. It also switches on the loop button in the top-right corner of the staff; press that again to go back to fresh rhythms.

## Changing a beat

Tap any beat of the bar on the staff and a picker opens under it, a grid of every figure that could go there. Tap one and it takes that beat's place immediately, notation and sound both, and the picker closes.

A tap means the figure covering that beat, so tapping either half of a two-beat figure offers a replacement for the whole of it, and the beats it occupies are shaded while the picker is open. What you are offered is everything that fits in what is left of the measure, so beat 1 of 4/4 can take a whole note, beat 2 anything up to three beats, and the last beat only the one-beat figures. Every figure in the library is offered, whether or not it is selected in the **Rhythm figures** panel — that panel is for what gets generated, and this is you choosing directly.

The generator will only start a figure where its length divides the beat, which is what keeps a bar it deals you readable, but placing one by hand is not held to that: you can put a half note on beat 2, or a quarter triplet across beats 2 and 3. There are no ties in the notation, so a figure like that is drawn plainly where you put it rather than as the tied pair an engraver would write. Swapping a long figure for a short one leaves the beats it gives up as quarter rests, and swapping a short one for a long one takes over the beats it needs.

Editing a bar switches the loop button on, the way Retry and loading a saved bar do, so the bar you have just built stays in front of you rather than being replaced on the next start. Edit during a run and the music follows within the bar. The picker closes when the bar underneath it changes for any other reason, and Escape or a tap outside closes it too.

## Saved bars

The **☆** in the top-left corner of the staff keeps the bar in front of you, along with the tempo you were reading it at. The star fills in once a bar is saved, and tapping it again drops that bar, so it tells you at a glance whether what you are looking at is one you already kept.

They collect behind the **☆ X saved** button in the top-right corner of the page, which opens a sidebar down the right-hand side. Each bar is drawn as its own small staff, so you pick one out by its notation rather than by a name, with the tempo, the meter and the length underneath it.

Tapping one loads it back: the bar returns to the staff, the tempo goes back to what it was saved at, and the meter and the length come with it. It also switches the loop button on, the way Retry does, so the bar stays in front of you instead of being replaced by a fresh one on the next start. Load one while a run is going and the music restarts on it at its own tempo. The **×** on a card removes it.

On a wide enough window the sidebar sits beside the app, which shifts over to make room, so you can keep the list open while you play. Anywhere narrower it lies over the page instead, and tapping the page outside it, pressing Escape, or loading a bar closes it again. Where it sits alongside, it is still open when you come back; where it covers the page it always starts closed.

Saved bars are kept in browser local storage, separately from the settings, and the newest sixty are held.

## Rhythm figures

The **Rhythm figures** panel lists every figure the generator can use, grouped by kind, each with its notation. A figure has three states, and tapping it moves to the next one:

- **off**, dimmed — never used.
- **selected**, a teal dot — in the pool the generator draws from.
- **guaranteed**, a brass dot with a glow and a light chasing its border — reserved a slot in every rhythm, so it is certain to come up.

Guaranteed figures are how you drill one thing without giving up variety around it: guarantee the sixteenth triplet, leave a few plain figures selected, and every rhythm will contain that triplet and something different around it. Each guaranteed figure claims one random slot, in one measure picked at random, so on a two-measure rhythm it turns up once across the pair rather than twice. If several of them cannot all fit, or one is longer than the bar, whatever does not fit is quietly left out rather than breaking the rhythm.

Triplets, quintuplets and septuplets have a section each. The triplet section also carries the broken triplets, where the three partials are not all sounded: two of them joined into one longer note, or the middle one silent. Both the eighth-note triplet and the quarter-note triplet have those variations.

The **×** on a group clears it, or selects all of it when it is already empty. With nothing selected at all, bars fall back to plain quarter notes.

A figure is only placed where its length divides the beat position, so two-beat figures start on odd-numbered beats and three-beat figures on beats 1 and 4. Figures longer than the bar (a whole note in 3/4, say) are simply never placed, and a beat that nothing fits gets a quarter note. Bars are regenerated until they hold at least two notes, so a selection of only long or silent figures can still yield a sparse bar.

While nothing is playing, the staff previews what your current settings produce. It rerolls when you change something that changes what can be generated — the figures, the meter, the length — and when you press Start, and it stays put for everything else, so a nudge of a volume slider does not throw away the bar you were reading. With the loop button on, Start keeps the bar in front of you instead.

A two-measure rhythm sits on one staff on a wide window and stacks onto two rows when the window is too narrow to read it, which is what phones get. Opening the saved-bars sidebar counts as narrowing the window, so it restacks for that too.

## Settings

- **The tempo dial** sets the beats per minute, from 40 to 200. Turn the dial anywhere on its face to sweep the tempo, tap **‹** and **›** for one beat at a time (hold them to run), or — where there is a keyboard — click the number and type one. The current tempo is on the dial itself and the teal ring shows where it sits in the range. A change takes effect immediately: whatever is playing bends to the new tempo without stopping or losing its place, count-in included.
- **tap tempo** sets the tempo from your own tapping. Tap it a few times in time and the dial follows the spacing of the last few taps; leave off for a couple of seconds and the next tap starts a new measurement. Every tap clicks back at you, whether or not the metronome is on. It sits under the dial on a wide window, and on a phone it moves out to the dial's bottom-right corner at the edge of the page, sized for a thumb.
- **Length** chooses one or two measures per rhythm.
- **Time** sets the time signature: 2/4, 3/4, 4/4, 5/4, 6/4, or 7/4. The quarter note is the beat in all of them, so the tempo means the same thing in every meter. The staff shows the signature, the count-in runs one bar of it, and the metronome accents beat 1. Changing it mid-run takes effect on the next rhythm.
- **The loop button** in the top-right corner of the staff repeats the current bar instead of moving on to a new one. Retry turns it on for you.
- **moving line** shows the playhead sweeping the notation. It runs from the opening barline to the closing one at a steady rate and picks up on the next row when the staff is stacked, so it only ever jumps where the music itself goes back to the top. It is off by default, so nothing moves unless you ask for it; it can be toggled during a run.
- **The two sound rows** — a cowbell for the rhythm and a metronome for the click — each have an icon and a volume slider. Tapping an icon mutes that sound outright and the row dims; moving its slider brings it back. Both are live: they reach sounds that were already scheduled, so a change is audible on the next note. Both scales mean the same thing, so 100 on one is as loud as 100 on the other. The metronome is on by default at 30, so it sits under the cowbell. On a wide enough window the rows are labelled `rhythm` and `metronome`.
- **cowbell on my pass** plays the rhythm again, quieter, during your pass so you can hear your offset against it. Like the volumes and the mutes, it takes effect the moment you tick it, on the pass already under way.
- **score my taps** turns on tap tracking, and reveals the tap pad, the legend, the results line and the calibration panel. It is currently commented out in `index.html` while the app is being tried without it; uncommenting that one line brings all of it back.
- **sound on my taps** plays a soft click on each tap. It only ever applies to a scored tap, so it appears with the rest of the scoring controls and is hidden while tap scoring is parked.

## Scoring

Tap scoring is commented out at the moment, so the controls below are not on the page. What follows describes it as it works when the **score my taps** line in `index.html` is uncommented.

With **score my taps** on, a tap pad appears and the space bar also registers taps. Each tap is matched to the nearest unplayed note within a timing window, and a colored dot with the error in milliseconds appears under that note as you play. Green means within 40 ms, amber within 90 ms, and red beyond that. Missed notes get a hollow ring, and extra taps get an ✕. The matching window shrinks automatically when notes are closer together.

## Saved settings

Every control, including the figure selection and the volume sliders, is written to browser local storage and restored on the next visit. Saved bars go to a key of their own beside it. Nothing is sent anywhere. If storage is unavailable the app still runs, it just starts from the defaults each time and saved bars last only as long as the tab does.

## Input delay calibration

Keyboards, touchscreens, and audio output all add latency, so raw tap timestamps land late. The calibration panel at the bottom handles this, and it only appears when **score my taps** is on, since that is the only thing the offset affects. Press **Run tap test** and tap along with ten steady clicks. The median delay of your taps becomes the offset, and that offset is subtracted from your taps before scoring. You can also type an offset by hand. Negative values mean you tend to tap early.

## Technical notes

- All timing uses the Web Audio clock (`AudioContext.currentTime`), not `setTimeout` or `Date.now`, so playback and scoring stay sample-accurate. Tap tempo is measured on the same clock.
- Changing the tempo mid-run works out where the music has got to in beats, drops the schedule, and lays it down again from that same position at the new beat length, fading the old graph out over 20 ms so the swap does not click.
- The cowbell is two detuned square oscillators through a bandpass filter, in the style of the 808 cowbell.
- The cowbell and the metronome each run through their own gain node, which is why the volume sliders and the mute buttons affect sounds that were scheduled a bar in advance. Sounds are written at full scale and a single output gain provides the headroom, so equal slider positions really are equal loudness.
- The notation is hand-drawn SVG rather than a music font, so rests and flags look stylized.
- The next rhythm's audio is scheduled while you play the current one, which is how the transition lands with no gap. Because of that head start, anything that changes what the next bar should be — the loop button, the meter, the length, the figures — lays that bar down again rather than waiting for it to come round.
- The screen wake lock uses the Screen Wake Lock API where it exists, and is simply skipped where it does not. It is taken when a run starts, dropped when the run stops or ends, and re-taken when you come back to the tab.
