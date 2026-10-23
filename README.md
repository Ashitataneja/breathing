# ebb

A breathing companion where the screen itself breathes. A luminous, organic form
swells and brightens as you breathe in, settles while you hold, and softens as
you let go. Drifting motes gather onto it with each inhale and float back out
with each exhale. Every session you complete leaves a star in your sky.

## Run it

```bash
npm install
npm run dev        # http://localhost:5173
npm test           # unit tests: breath timing + session state machine
npm run build      # typecheck + production build into dist/
```

No backend, accounts or analytics. Preferences and history live in `localStorage`.

## How it works

```
BreathSource ──sample(t)──▶ BreathSignal { fullness, phase }
     ▲                              │
SessionEngine.elapsed()             ├─▶ Garden (canvas renderer, rAF)
(monotonic clock, pause-aware)      ├─▶ Ambience (Web Audio)
                                    └─▶ phase words, haptics
```

- `src/breath/pattern.ts`: patterns, presets, and `sampleBreath(pattern, t)`, a
  pure function from elapsed time to phase and eased fullness.
- `src/breath/sources.ts`: the `BreathSource` interface. `GuidedSource` follows
  a pattern; `OpenSource` is an unpaced, slowly resting presence. A future
  sensor-driven source (microphone, camera, wearable) would implement the same
  interface behind an explicit permission step. Nothing else would change.
- `src/session/SessionEngine.ts`: `idle → running ⇄ paused → complete`. Active
  time is accumulated from `performance.now()`, so the session stays correct in
  background tabs and across pause/resume. A session ends at the end of a
  breath, never mid-exhale. If a tab sleeps past the end, extra time isn't counted.
- `src/render/Garden.ts`: Canvas 2D renderer for the sky, tide, membrane form,
  motes and personal stars. It draws whatever fullness it's given. Its only
  internal state is smoothing.
- `src/components/BreathCanvas.tsx`: one persistent canvas behind every screen,
  driven by whichever screen is active. If Canvas is unavailable, it falls back
  to a DOM glow that follows the same driver.

## Using it

- **Keyboard (in a session):** `Space`/`K` pauses and resumes, `M` turns sound
  on and off, `Esc` ends the session. On the landing screen, `Enter` begins.
- **Touch:** tap to show or hide the controls. Controls fade after a few seconds
  and come back on any movement or keyboard focus.
- **Guidance modes:** *Guided* shows a quiet word each phase. *Quiet* shows
  words for the first two breaths, then only the light. *Open* has no pacing at
  all. The modes are meant to help you gradually practise without guidance.
- **Settings:** rhythm presets, or custom in/hold/out/rest in 0.5 s steps
  (holds can be 0); session length; sound; reduced motion (system, reduced or
  full); vibration where supported.
- The screen is kept awake during sessions where the Wake Lock API exists.

## Limitations and honest notes

- **No breath detection.** In every mode the light follows a rhythm you choose,
  and the app says so in Settings. The microphone and camera are never accessed.
  Detection was left out of the MVP on purpose. Breath sounds are loud on both
  the inhale and the exhale, so a microphone can't reliably tell the phases
  apart without careful signal work, and a misleading "it follows your breath"
  mode would undermine trust.
- Vibration works only where `navigator.vibrate` exists (mostly Android
  browsers). iOS Safari doesn't support it, and the toggle says so.
- Ambient sound is synthesised with Web Audio and starts only after a tap. In a
  background tab the session keeps time correctly, but visuals pause until you
  return (by design: browsers stop rendering hidden tabs).
- History is per-browser. Clearing site data clears your sky.
