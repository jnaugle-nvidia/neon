# NEON//EQ
<img width="1919" height="907" alt="image" src="https://github.com/user-attachments/assets/de941f0b-cc01-4440-80a8-6e113bae1335" />

<img width="1919" height="909" alt="Screenshot 2026-08-11 231637" src="https://github.com/user-attachments/assets/b84dcb5d-ae7c-49a7-87f3-6307e3a12733" />

A cyberpunk graphic equalizer for Spotify. One self-contained HTML file, zero dependencies, no build step — it taps the actual audio leaving your machine and runs a real 8192-point FFT on it, rendered as a CRT-flavored LED matrix with beat-driven glitch effects.

**File:** `neon-eq.html` · **Run it:** open in Chrome or Edge. That's the whole install.

![NEON//EQ](https://img.shields.io/badge/deps-0-00e8ff) ![](https://img.shields.io/badge/build-none-ff2fb0) ![](https://img.shields.io/badge/file-1-ffc247)

---

## Quick start: piping Spotify in

Spotify has no visualizer API, and its audio-analysis endpoints are closed to new apps — so NEON//EQ reads the sound itself via Chrome's screen-capture pipeline. No drivers, no virtual cables.

**Spotify desktop app:**
1. Start playback in Spotify.
2. Click **▶ SYSTEM AUDIO**.
3. In the picker choose the **Entire Screen** tab, click your screen, and — the step everyone misses — tick **"Also share system audio"** at the bottom left.
4. Share. Music keeps playing through your speakers; the bars light up.

**Spotify web player:** same button, but pick the **Chrome Tab** tab, select the `open.spotify.com` tab, and tick **"Also share tab audio."**

**No capture available?** **MIC** listens through your microphone (speakers out loud, a phone in the room, a turntable on line-in). **DEMO** runs a synthetic 120 BPM track with no permissions at all — useful for tuning the look.

> If you shared a surface but forgot the audio checkbox, the app detects the missing track and tells you instead of sitting dark.

---

## Controls

### Sources
| Button | Effect |
|---|---|
| ▶ System Audio | Capture what the machine is playing (the Spotify path) |
| Mic | Analyze the microphone |
| Demo | Synthetic track, no permissions |
| Stop | Back to demo, release the capture |

### Selects
| Control | Options |
|---|---|
| BANDS | 16 · 24 · **31** · 48 · 64 log-spaced bands, 25 Hz → 16 kHz |
| PALETTE | 17 gradients (list below) |
| CELL | Square · Hex · Triangle · Skull · Heart |

### Sliders
| Slider | Range | What it does |
|---|---|---|
| GAIN | −12 … +30 dB | Manual level. With AGC on, acts as a bias on top of the auto gain |
| DECAY | 60 … 900 ms | Release time of the bars (attack is fixed fast) |
| BLOOM | 0 … 150% | Neon glow intensity |
| GL AMT | 0 … 200% | Glitch severity: RGB split, shake, slice tears |
| GL SENS | 0 … 100% | How readily a beat triggers effects; 50 = factory calibration |

### Toggles
| Button | Effect |
|---|---|
| AGC | Auto gain — ranges the input so peaks ride just under the top on any master |
| ⏺ Rec | Record a 15 s WebM clip of the display (details below) |
| ◇ Wire | Wireframe cells — outlines instead of fills; caps stay solid |
| Mirror | Reflection below the baseline |
| Glitch | Master switch for beat-driven effects |
| ◱ Full Window | Bars fill the entire window, chrome hidden (move the mouse to get it back) |
| ⛶ Fullscreen | Browser fullscreen on top of whatever mode you're in |

---

## Keyboard

| Key | Action |
|---|---|
| `W` | Full window mode |
| `F` | Fullscreen |
| `D` | Demo source |
| `A` | AGC on/off |
| `C` | Record 15 s clip (press again to stop early) |
| `H` | Cycle cell shape |
| `R` | Wireframe |
| `M` | Mirror |
| `G` | Glitch |
| `P` / `⇧P` | Next / previous palette |
| `1`–`9` | First nine palettes directly |
| `↑` `↓` | Gain ±1 dB |
| `?` | Help panel |
| `ESC` | Close one layer: help → full window → message |

---

## Header readouts

| Field | Meaning |
|---|---|
| SOURCE | SYS AUDIO · MIC IN · DEMO |
| RATE | Capture sample rate |
| PEAK / RMS | Honest dBFS off the time-domain signal (not the bar heights) |
| DOM | Dominant frequency right now |
| BPM | Detected tempo; **dims when the lock is weak**, and the magenta dot beside it pulses on each beat |
| ● LED | Cyan = live capture · red = clipping |

---

## What's under the neon

- **31 log-spaced bands** on ISO third-octave spacing off an 8192-point FFT. Each band blends peak-bin and average-power so tonal content spikes while broadband stays honest.
- **+3.5 dB/octave pink tilt** — the same compensation hardware RTAs use, so treble actually moves instead of hugging the floor.
- **Custom ballistics:** 18 ms attack, slider-set release, 550 ms peak hold, caps fall under gravity. The Web Audio analyser's built-in smoothing is disabled — it turns everything to mush.
- **AGC** tracks the loudest band with a fast-attack/slow-release envelope (so it ranges on beats, not troughs) and glides an auto offset — ~2.5 s up, 0.6 s down — to hold peaks ~4 dB under the ceiling. Frozen during silence so it never amplifies the noise floor. The gain readout shows both parts: `+6 (+4A)`.
- **BPM detection** is spectral-flux onset detection autocorrelated over a 6 s window (lags spanning 60–200 BPM), with windowed harmonic rejection and centre-of-mass peak location. It works on brickwalled masters where energy-threshold detectors go blind — verified within ±2 BPM across 70–180 BPM at every compression level. It needs ~6 s of signal to lock.
- **Beat-driven effects** run on a separate bass-transient detector: bloom flash, chromatic RGB split, screen shake, and horizontal slice tears, scaled by GL AMT and gated by GL SENS.
- **Standby:** after ~8 s below −58 dBFS on a live source, the lit cells fade out leaving the ghost grid and a breathing `STANDBY · NO SIGNAL` marker. Any real signal snaps it back in a quarter second.
- **Wake lock:** while the page is visible it holds a screen wake lock, so a spare monitor running the visualizer won't sleep mid-set. Auto-releases when the tab is hidden.
- **Cell shapes:** hex mode interlocks two half-row-staggered honeycomb columns per band; triangle tessellates point-up/point-down; skull and heart switch to near-square cells so the pictograms stay legible (at the cost of vertical resolution — 16–31 bands is their sweet spot). The skull's eye sockets are genuine holes cut by winding-rule subpaths, so they work in wireframe too.
- **Rendering** is plain canvas 2D, batched one path per row, with the static ghost grid cached to its own layer and re-rendered only when geometry changes. Worst case measured ~1 ms/frame against the 16.7 ms 60 fps budget. Two-stage downsampled bloom, scanlines, film grain, vignette — all drawn on-canvas, so recordings keep the look.

## Recording

**⏺ Rec** (or `C`) captures 15 seconds of the canvas at 60 fps into a WebM (VP9, 8 Mbps) that downloads automatically as `neon-eq-<timestamp>.webm`. When a system-audio or mic capture is live, **the source audio is cloned into the clip** — synced music and bars, ready to share. Press again to stop a take early and keep what you have. Demo mode records video only.

## Palettes

NEON · ACID · INFERNO · ICE · VAPOR · OUTRUN · PLASMA · TOXIC · SPECTRUM · AURORA · MIAMI · ULTRAVIOLET · ROSE GOLD · NOIR (monochrome) · EDGERUNNER (yellow→blue) · HAL 9000 (neon red) · NERF (blue→orange)

Near-complementary gradients (Edgerunner, Nerf) are routed through intermediate hues rather than interpolated straight, which would cross dead grey at the midpoint.

## Privacy

Nothing is uploaded, anywhere, ever. Captured samples go into the FFT and die there. The one thing that ever touches disk is a **⏺ Rec** clip, which goes to your own downloads folder. Settings persist in `localStorage`.

## Requirements & honest limitations

- **Chrome or Edge on desktop.** System-audio capture via `getDisplayMedia` is Chromium-only; the "Also share system audio" checkbox is Windows-specific (on macOS, Chromium can only share *tab* audio — use the Spotify web player there).
- **No track metadata.** Spotify closed its audio-analysis/metadata endpoints to new apps, so title/artist can't be shown without an OAuth app registration and a localhost server — out of scope for a single file.
- Everything on screen (BPM, dominant frequency, levels) is computed live from the signal itself.
