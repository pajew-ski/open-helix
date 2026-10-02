# AGENTS.md

Public GitHub repo, project name **open-helix**. Content: a Shepard-Risset glissando generator in the browser. A bank of sine components glides through a fixed log-frequency span under a bell-shaped envelope, synthesized sample by sample in an AudioWorklet, with a spectrogram that draws it. It is a sibling of **open-entrainer** and **open-desensitizer** and shares its design with **temet-nosce**; the four should look and read as one family.

Target audience: someone curious about the auditory illusion, or someone who wants an endless rising or falling texture for listening, focus, or sound design, without an app, an account, or a download. The page has to explain the illusion in one read and expose every parameter without a settings menu.

## Non-Goals

- No framework, no build tool, no bundler, no package manager
- No second file. The app is `index.html` alone; stylesheet and script are inline so one file can be copied anywhere and run
- No external resource of any kind: no CDN, no web font, no analytics
- No accounts, no network requests, no data leaving the page
- No manual dark/light toggle. Automatic only, via `prefers-color-scheme`
- No manual language switch. Automatic only, via the browser's language, with a URL override
- No color. The design is achromatic; components are told apart by position and opacity, never by hue
- No modal, no collapsible settings panel. Everything is on the page

## Repo Structure

```
/
├── AGENTS.md
├── README.md            (short: what this is, link to the Pages site)
├── LICENSE              (Unlicense)
├── index.html           (the whole app: page, stylesheet and script in one file)
└── .github/workflows/
    └── notify-addon.yml (after each change to index.html on main, asks the Home Assistant Apps Collection to build a new add-on version)
```

GitHub Pages deploys from the root of `main`. The footer derives its GitHub links from the Pages URL, so a fork needs no edit. The Home Assistant add-on is built from `index.html` by the Home Assistant Apps Collection; this repo carries no add-on files, only the workflow that notifies the collection. It needs the secret `APPS_COLLECTION_TOKEN`; without it the collection still picks the change up within the hour.

## Design

The inline stylesheet begins with the token block from temet-nosce, verbatim. It stays verbatim in all projects of the family; a change to the tokens is a change to all of them.

- Color: oklch with chroma 0. Light: bg 98%, surface 94%, border 85%, text 15%, muted 40%. Dark flips the scale under `prefers-color-scheme: dark`. `color-scheme: light dark` on the root so form controls follow.
- Spacing: Fibonacci in pixels, 5 8 13 21 34 55 89 144, as `--space-1` to `--space-8`.
- Type: system-ui. Base 1rem, line-height 1.618, sizes 0.875rem, 1rem, φ, φ², φ³.
- Layout: a `.shell` of 987px max width with 21px side padding. Hero, sections, footer, identical in spacing to the siblings (hero padding 144px top, 55px bottom, 89px top below 800px width). Text columns cap at 42rem.
- Visualization: a bordered surface box, `clamp(300px, 40vw, 420px)` high, split into the spectrogram and a 144px envelope column (89px below 600px width). Both share one logarithmic frequency axis with gridlines at 25 Hz times powers of two. Readouts sit in the top corners of the spectrogram as HUD text on a translucent surface backdrop: label in small caps, value at φ size, detail in muted small text.
- Settings: a row of preset chips, then five bordered panels, Motion, Structure, Sound, Space and mix, Session and display, in an auto-fit grid that wraps to one column below 233px per panel. Panel titles are small caps like the HUD labels. Controls that have no effect under the current settings are dimmed to 40%, never hidden.
- Dock: transport, clock and volume in a bar that sticks to the bottom of the viewport, so the sound stays controllable while the panels scroll.
- Buttons: bordered, transparent. The primary button and a pressed segment are inverted (text-colored surface, background-colored label). Nothing is colored, nothing glows.

## Behavior

- Engine: one function, `makeEngine(sampleRate)`, used both inside the AudioWorklet (its source is serialized into a Blob URL together with `mod` and `envelope`) and on the main thread in a `ScriptProcessorNode` when the worklet fails to load. Blocks of 64 samples, up to 24 components, each with its own phase, gain and frequency interpolated across the block. A component that wraps from the top of the span to the bottom changes frequency at once; its gain is zero there. Components above 0.45 of the sample rate are silenced, below 16 Hz faded out. Harmonics above the Nyquist limit are dropped per component. Loudness is normalized by the mean power over one cycle, so the number of components, the envelope and the waveform do not change the level. The output passes through `tanh`.
- Motion: position in octaves advances by `direction · tempo swing · acceleration / time per octave`. Swinging direction is a sine with the swing period. With steps per octave set, the position is quantized and the transition between steps is a smoothstep over the set fraction.
- Envelopes: Hann, Gauss (width from the shape parameter, zeroed at the edges), triangle, plateau (cosine flanks of the shape width). Tilt adds dB per octave around the center.
- Graph: engine, high-pass, low-pass, then a dry path and a convolver with a generated two-channel noise impulse response (exponential decay, darkening over its length), mixed by equal-power crossfade, into an analyser, the master gain and a fade gain. The fade gain carries fade in, fade out and the 0.3 s stop ramp.
- Parameters: defined once in a schema with key, label, control type, range, default, formatter and an optional `needs` predicate for dimming. Presets reset every parameter except duration, fades, view, time window and volume, then apply their own values. Persisted in `localStorage` under `open-helix-v1`.
- iOS: on Start and Resume the page sets `navigator.audioSession.type = "playback"` where it exists. On iOS the output plays through a `MediaStreamAudioDestinationNode` in an `Audio` element, the category the silent switch does not mute; if that element refuses to play, the output goes to `ctx.destination` and a looping one second silent WAV plays alongside. Pause and Stop pause both.
- Transport: Start, Pause, Stop in the dock; Pause suspends the context, Resume resumes it. Space starts or pauses unless a form control has focus. With a duration set, the fade out begins so that it ends at the duration, and the state reads Finished.
- Drawing: the spectrogram scrolls left by the time window. The Model view draws each component as a line segment from its previous to its current position, opacity from the envelope. The Analyzer view maps FFT bins onto rows of the log axis and paints them in the text color with alpha from level against a slowly decaying peak. The envelope column draws the curve filled in border color, edged in text color, with a dot per component. All canvases read their colors from their own computed style, so they follow the color scheme without a second palette. Device pixel ratio is respected.
- Readouts: State (Ready, Starting, Running, Paused, Finished, Error) and elapsed time, over the duration when one is set. Motion direction with the current rate in cents per second, and the audible range of the span. The footer of the session names the engine mode, sample rate and output route.

## Language

The page ships in English and German in the same file. A small classic script in the head sets `<html lang>` before the first paint: German when the browser's first language starts with `de`, English otherwise; `?lang=de` or `?lang=en` overrides. Without script the page stays English.

- Prose in the markup exists once per language, as sibling elements with `lang="en"` and `lang="de"`. One CSS rule hides every element whose `lang` does not match the root. Short labels follow the same pattern with sibling spans.
- Strings the script writes are given in place as `L(english, german)`. Numbers use a decimal point in English and a decimal comma in German.

## Copy

Plain sentences, present tense, no exclamation marks, no emoji, no em dashes, in both languages. The German avoids direct address where an infinitive does the job. Explain the illusion, name the parameters, say what the tool does not do. Warnings are stated once in a section of their own. Product names are lowercase in headings and the footer, as in temet-nosce. README, AGENTS.md and commit messages are English.

## Files

### `index.html`

One file with four parts: the language script and the `<style>` block in the head, the markup, and one `<script type="module">` at the end of the body (plus the small footer module before it).

Markup: hero with the project name and one sentence. Sections: How it works, Session (visualization, presets, panels, a caption with the engine line), Using it, Before you use it. Footer with the AGENTS.md and source links and the module that rewrites them from the Pages URL. The dock after the footer.

Style block: token block, base rules (hero, sections, controls, buttons, chips, segments, footer), then the visualization, HUD, panels and dock.

Script block: an ES module. No globals beyond what the DOM gives. Sections: engine (mod, envelope, makeEngine, the worklet source), language (LANG, L), parameters (formatters, SCHEMA, MASTER, PRESETS, load, save, engineParams), interface from the schema (el, buildControl, buildUI, syncUI, applyPreset, set), derived quantities (derive), audio (makeIR, silentWav, claimPlayback, startCarrier, releasePlayback, renderEngine, routeOutput, startAudio, applyGraph, pushEngine, stopAudio), transport (setState, elapsed, renderClock, play, pause, stop, tick), drawing (palette, viewRange, layout, clearSpec, drawAxis, drawModel, prepFFT, drawFFT, drawEnv, drawIdle, frame), changes (changed, syncNeeds), and the start-up and event wiring at the bottom.
