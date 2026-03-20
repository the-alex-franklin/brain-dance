# Brain Dance
### Neural Music Interface — Design Spec & Roadmap v0.3

---

## Vision

The brain already has one voice box. This is a second one.

The first translates thought into speech — words, tone, inflection. It's been there since birth, refined over a lifetime, and most people never think about it. This system is the same idea applied to music. A speaker that doesn't play music *at* you, but speaks *for* you — a direct extension of your nervous system into sound.

A bidirectional learning system that maps real-time EEG data from a Neurosity Crown to musical parameters in Ableton Live. The performer's mental state drives the music. Over time, the system learns the performer's unique neural signatures and the performer learns what their mind can control. The interface becomes increasingly personal — nobody else's brain would play it the same way.

---

## Stack

```
Crown (EEG hardware)
  ↓ Built-in OSC output — UDP port 9000
  ├── Raw mode: 8-channel EEG at 256Hz
  └── Perform mode: focus/calm/band-power at 4Hz

Deno/TypeScript (intelligence layer — phases 2+)
  ↓ OSC messages (npm:node-osc)

Max for Live — ES5 JS (dumb receiver)
  ↓ LiveAPI parameter mapping via Macro knobs

Ableton Live Suite (audio engine, VSTs, plugins)
```

> **Key discovery:** The Crown has built-in OSC support. It can stream directly to Max for Live without Deno in the middle. This collapses the MVP stack considerably — Deno enters the picture in Phase 2 as the intelligence layer, not Phase 1.

---

## Dependencies

| Tool | Notes |
|------|-------|
| Neurosity Crown | Hardware — has device emulator for dev without physical device |
| Ableton Live Suite | Requires Suite or Standard + Max for Live license (~$99) |
| Max for Live | Bundled with Suite |
| npm:node-osc | OSC for Deno — TypeScript types included, ESM native |
| npm:@neurosity/sdk | Crown SDK — Node/JS native, works via Deno Node compat |
| Deno | Intelligence layer runtime |

---

## Milestone 1 — The Lemon Test (MVP)

**Goal:** Think about biting into a sour lemon. Hear a bass note.

**Stack for this milestone:** Crown → OSC (built-in) → Max for Live. No Deno.

The Crown's built-in Perform Mode OSC output streams processed focus/calm/band-power metrics at 4Hz directly over UDP port 9000. Max for Live receives this via `udpreceive`. No custom code in the signal path yet — just configuration.

### Steps

1. **Configure Crown OSC output**
   - Enable Perform Mode in the Crown SDK/app
   - Confirm UDP packets arriving on port 9000

2. **Build Max for Live receiver (ES5 JS)**
   - `udpreceive 9000` object listens for Crown OSC
   - Parse incoming band-power values
   - Identify which band/channel spikes on the lemon event (likely gamma)
   - Threshold detection: value exceeds threshold → fire MIDI note
   - Debounce to prevent double-triggers

3. **Ableton session**
   - Single instrument track with a bass patch
   - Max for Live device on the track
   - Macro knob as the single control point

4. **Calibration**
   - Run 10 lemon imagination sessions
   - Log which band/channel produces the strongest, most consistent response
   - Tune threshold accordingly

### Max for Live ES5 JS Notes

- No async — use `Task` object for timing
- No modules — single file, everything in scope
- `LiveAPI` object for Ableton parameter access
- Keep it dumb — no logic here beyond threshold + debounce
- Macro knobs as the single control surface — Max maps to Macros, Macros map to everything else

**Done when:** Sit down, think about a lemon, hear a bass note. Reliably.

---

## Milestone 2 — Continuous State Mapping

**Goal:** Mental state continuously shapes a live performance in real time.

New in this milestone: Deno enters the stack as the intelligence layer between Crown and Ableton.

The Crown's raw OSC output is sufficient for discrete event detection (Milestone 1), but continuous nuanced mapping benefits from custom signal processing — windowed FFT, band power normalization, smoothing. That lives in Deno.

### Steps

1. **Crown → Deno bridge**
   - Connect to Crown SDK via `npm:@neurosity/sdk` in Deno
   - Subscribe to raw brainwaves (256Hz, 8 channels)
   - Alternatively: use Crown's built-in Raw Mode OSC and receive in Deno

2. **State classifier (Deno/TS)**
   - Moving window FFT on EEG stream
   - Extract band power: delta, theta, alpha, beta, gamma per channel
   - Normalize to 0–1 per band
   - Output continuous state vector at ~4Hz

3. **Parameter mapper**
   - Map state vector to Ableton parameter ranges
   - JSON-configurable mapping table
   - Smoothing/interpolation to prevent jitter

4. **Expand Max for Live receiver**
   - Receive continuous OSC parameter stream from Deno
   - Map to multiple Macro knobs simultaneously
   - Multiple parameters updating in real time

### Mental State → Musical Parameter Map *(starting point)*

| State | EEG Signature | Musical Parameter |
|-------|--------------|-------------------|
| Focus | Beta increase (13–30Hz) | Tempo, rhythmic tightness |
| Calm / flow | Alpha increase (8–13Hz) | Reverb depth, filter openness |
| Arousal | Gamma increase (30+Hz) | Intensity, distortion, velocity |
| Fatigue | Theta increase (4–8Hz) | Density reduction, sparse texture |

**Done when:** A pre-composed Ableton session responds fluidly and meaningfully to mental state in real time.

---

## Milestone 3 — MIDI as Intent Signal

**Goal:** Physical input devices feed the AI layer alongside EEG — not mapped directly to output.

A keyboard, pad controller, or any MIDI device becomes an intent signal, not a controller. The AI ingests MIDI alongside EEG and the output is a function of both. The same key pressed in a focus state sounds different than in a calm state.

### Why This Matters

- EEG alone is noisy and slow — ~250ms latency minimum, variable signal quality
- MIDI alone is just a keyboard — fast and precise but no mental context
- Together: MIDI provides intentional, fast, precise input; EEG provides continuous subconscious context
- The combination is more expressive than either alone

### Architecture

```
MIDI device → Deno MIDI listener
Crown EEG  → Deno Crown bridge
                ↓
        Combined ingestion layer
      (state vector + MIDI event)
                ↓
      Context-aware output mapper
                ↓
          OSC → Ableton
```

**Done when:** The same physical key produces meaningfully different output depending on measured mental state.

---

## Milestone 4 — Bidirectional Learning

**Goal:** The system learns the performer. The performer learns the system.

### Performer → System

- Session recorder captures: EEG state vector + MIDI events + musical parameter state + timestamp
- After each session, offline process analyzes correlations
- Builds a personal neural signature model — unique to this performer's brain
- Classifier improves over time: less noise, better response, higher confidence

### System → Performer

- Real-time visual feedback UI (Deno/Fresh or plain HTML)
  - Current detected mental state
  - Model confidence indicator
  - Session history graph
- Performer learns which mental states produce which musical effects
- Intentional mental control becomes possible over weeks and months

### Claude API Integration

Higher-level session intelligence layer:

- Analyzes session logs for patterns across multiple sessions
- Suggests composition or mapping changes based on what the performer's brain responds to
- Long-term arc: *"your focus state tends to appear 2 minutes in — the arrangement could anticipate that"*

**Done when:** The classifier is measurably more accurate after 10 sessions than after 1. Performer can deliberately invoke a mental state and hear the system respond.

---

## Neural Fingerprint Portability

The fingerprint built here is not music data — it's a portrait of how this specific brain works. It has value far beyond this application. Treat it accordingly.

### Principles

- **Application-agnostic** — normalized format, not coupled to Ableton, music parameters, or any specific use case
- **Portable** — open format (JSON), lives locally on the performer's machine, never in a third-party cloud
- **Versioned** — model improves over time, old versions preserved not overwritten
- **Labeled** — each session tagged with context: application, date, duration, mental state annotations, confidence scores
- **Exportable** — single command produces a self-contained fingerprint package importable by any future application

### What Gets Exported

- Per-band baseline profiles (resting state per channel)
- Event signatures (labeled neural events and their EEG fingerprints)
- Session history metadata
- Model weights and classifier thresholds
- Confidence scores per signature

There is no garbage data here. Data that is noise for this application is signal for the next one. The neural fingerprint accumulates value across every application built on top of it — a coding focus tool, a meditation tracker, a future interface that doesn't exist yet. Every session makes the fingerprint richer regardless of what it was recorded for.

---

## Future Scope / Stretch Goals

### Intent Detection (Bereitschaftspotential)

The brain generates a measurable "readiness potential" 500ms–1 second before a conscious voluntary movement. The system learns the neural precursor to decisions — not "you pressed this key" but "you decided to go up in pitch."

Over enough sessions, the model detects intent before any physical action. The keyboard confirms and trains the model. Eventually the keyboard becomes optional. The performer puts it down.

### The Full Instrument

- Pre-composed pieces with neural performance parameters baked into the arrangement
- Live improvisation mode — full generative response to mental state, no pre-composition
- Multi-session arc — pieces that evolve over weeks as the model deepens
- Collaborative performance — two performers' neural states interact to shape shared output

### Publication / Performance

- Live performance context: the instrument as a concert piece
- Open source release: the architecture as a platform others can build on
- Academic framing: a new class of instrument — bidirectional, personal, irreproducible

---

## Open Questions

### Resolved

| Question | Answer |
|----------|--------|
| Crown SDK + Deno compatibility | ✅ Crown SDK is Node/JS native — works via Deno Node compat. Also has built-in OSC output, may not need SDK at all for MVP. |
| OSC library for Deno | ✅ `npm:node-osc` — TypeScript types included, ESM native, actively maintained. |
| Ableton version | ✅ Requires Live Suite, or Live Standard/Intro + Max for Live license (~$99). |

### Active

- Crown device emulator: usable for development without physical hardware? *(Neurosity offers emulators — worth testing)*
- MIDI input in Deno: evaluate `npm:midi` or `npm:easymidi` for cross-platform MIDI listener
- Latency budget across full stack: Crown → Deno → OSC → Ableton → audio. Target <100ms (phase-locked) for discrete events.
