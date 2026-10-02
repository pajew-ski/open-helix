# open helix

A Shepard-Risset glissando in the browser. A tone that keeps rising, or falling, and never arrives, with every parameter of the illusion on one page and a spectrogram that draws it as it plays. One HTML file with everything in it, nothing else. In English and German, chosen by the browser's language.

**Site**: [pajew-ski.github.io/open-helix](https://pajew-ski.github.io/open-helix/)

## How it works

Several sine tones a fixed interval apart glide in the same direction at the same speed. A bell curve over the logarithmic frequency axis sets their loudness: quiet at the bottom, loudest in the middle, quiet again at the top. When the highest tone fades out, it wraps around and fades back in at the bottom. The ear follows the pitch class, which keeps moving, and loses track of the register, which stays where it is. Gliding, that is the Shepard-Risset glissando; in discrete steps, the Shepard scale.

The spacing does not have to be an octave. Any other interval gives a pattern that repeats only after that interval, and the result sounds less like one endless tone and more like a slowly turning texture. On top of the motion sit the other dimensions of the sound: waveform and harmonics, a binaural beat between the ears, tremolo, panning by pitch or in a slow circle, pink noise, reverb, and filters.

The page draws the sound on a logarithmic frequency axis. The Model view plots each component as the engine computes it, so the diagonal stripes of the illusion are visible directly. The Analyzer view shows the actual output spectrum after filters and reverb. Beside it, the envelope with the current components as dots.

Everything is synthesized while it plays, in an AudioWorklet where the browser has one and a ScriptProcessor where it does not. Nothing is downloaded, nothing is sent anywhere. The settings are kept in `localStorage`.

| Group | Parameters |
| --- | --- |
| Motion | direction (up, down, swinging), time per octave, swing period, steps per octave, step transition, acceleration, tempo swing |
| Structure | number of components, spacing in cents, center frequency, envelope and its shape, spectral tilt |
| Sound | waveform, number of harmonics, harmonic rolloff, binaural beat, tremolo rate, depth, and left-right phase |
| Space and mix | pan by pitch, auto-pan, pink noise, reverb mix and length, high-pass, low-pass |
| Session | duration, fade in, fade out, view, time window, volume |

## Using it

1. Start quiet. Use headphones for the binaural beat, panning, and tremolo phase.
2. Pick a preset as a starting point: Shepard-Risset, Falling drone, Riser, Shepard scale, Trance, or Pendulum.
3. Change one thing at a time and watch the spectrogram. Components and spacing change the texture most, time per octave and direction change the motion.
4. Set a duration if the session should fade out and stop by itself; at zero it runs until you stop it. Space starts and pauses.

## Before you use it

This is a listening tool, not a medical device, and it does not treat anything. The effect does not need volume, and fast runs with many components feel more intense than expected. Do not use it while driving or operating anything that needs your attention. With epilepsy, a pacemaker, or a psychiatric condition, ask a clinician first. Stop if you feel unwell.

## Running it locally

```bash
git clone https://github.com/pajew-ski/open-helix.git
cd open-helix
open index.html
```

The whole app is `index.html`; copy that one file anywhere and it runs. There is no build step and no dependency. Opened straight from disk, some browsers refuse the AudioWorklet and the page falls back to the ScriptProcessor; served over HTTP it uses the worklet. Any static host serves it as is; on GitHub Pages, deploy from the root of `main`. The footer links adapt to a fork automatically.

As a Home Assistant add-on, the page ships through the [Home Assistant Apps Collection](https://github.com/pajew-ski/home-assistant-apps-collection), together with its sibling apps. Every change to `index.html` on `main` becomes a new add-on version automatically.

The page speaks English and German. It shows German when the browser's first language is German and English otherwise; `?lang=de` or `?lang=en` overrides that.

Everything here was built by a coding agent from [AGENTS.md](AGENTS.md), which is the design and behavior spec of the tool. It is a sibling of [open entrainer](https://github.com/pajew-ski/open-entrainer) and [open desensitizer](https://github.com/pajew-ski/open-desensitizer).

## License

Public domain under the [Unlicense](LICENSE). Copy it, change it, sell it, build on it. Tools for the ear should be free.
