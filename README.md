# README: Touch Tilt Speak FM Synth

## Overview

This is a web-based FM synthesizer built with HTML/CSS for UI, JavaScript for interaction and control, and Csound (compiled to WebAssembly via `csound.js`) as the audio engine. The user interacts through touch, tilt (accelerometer), and voice input, creating a unique multi-modal musical interface.

## HTML

The HTML provides the structure and interface of the app:

### Key Elements:

* `<h2>`, `<p>`: UI text headers and prompts.
* `<button id="startAllBtn">`: Starts the synthesizer and requests device permissions (microphone, orientation).
* `<input type="range">`: Sliders for `releaseTime` and `vocoderMix`, controlling envelope speed and morph mix ratio.
* `<div id="xy-pad">`: Interactive pad where touch X/Y determines amplitude and frequency of voices.

### Fast / Slow Slider:

* `releaseSlider` adjusts the `releaseTime` in the Csound engine, affecting portamento (smoothing of parameter changes).

### Vocoder Mix Slider:

* `vocoderMixSlider` sets a blend between the raw FM sound and the vocoded voice input using Csound's `pvsfilter`.

## CSS

Styles the UI with a retro synth aesthetic (black background, lime green monospace text).

### Highlights:

* `#xy-pad`: Bordered interactive area for touch input.
* `input[type="range"]`: Consistent slider width and alignment.
* `body`: Uses flexbox to center and align all elements vertically.

## JavaScript

Acts as the glue connecting the UI to the Csound WASM engine.

### Initialization Flow:

1. On click of `#startAllBtn`:

   * Requests microphone access.
   * Requests device orientation access (for tilt controls).
   * Loads `csound.js` (a wrapper around the Csound WebAssembly engine).
   * Initializes the Csound engine and starts the instrument.

### Csound Interaction:

* Sets control channels: `releaseTime`, `vocoderMix`, `ampN`, `freqN`, `fmIndex`, `fmRatio`.
* Handles touch to determine which "voice" is played, assigning touch IDs to three available voices.
* Tilt controls modify:

  * `fmIndex`: Modulation index (how intense FM modulation is).
  * `fmRatio`: Ratio between modulator and carrier frequencies.

## Csound (within JavaScript `csound.compileOrc(...)`)

This is the core audio engine, written in the Csound orchestral language, compiled to WebAssembly via `csound.js`.

### Block-by-Block Breakdown:

```csound
sr = 48000
ksmps = 8
nchnls = 2
0dbfs = 1
```

* Standard Csound setup for:

  * Sample rate (48 kHz),
  * k-cycle size (`ksmps = 8`),
  * Stereo output,
  * Normalized audio (0dB = 1.0 amplitude).

```csound
instr 1
```

* Defines Instrument 1, which stays on and processes continuously.

### Control Channels:

```csound
krelease chnget "releaseTime"
kmix chnget "vocoderMix"
```

* `chnget`: Reads values from named control channels set by JavaScript sliders.

### FM Synth Voice Definitions:

```csound
kamp1 = portk(chnget:k("amp1"), krelease)
kfreq1 = portk(chnget:k("freq1"), krelease)
...
```

* Smooths abrupt parameter changes using `portk` (controlled by `releaseTime`).
* Reads amplitude and frequency for each of the 3 FM voices.

### FM Modulation:

```csound
kfmFreq1 = kfreq1 * kfmRatio
amod1 oscili kfmIndex * kfmFreq1, kfmFreq1
acar1 oscili kamp1, kfreq1 + amod1
```

* Basic FM: Carrier + Modulator, where:

  * Modulator = oscillator (`amodN`)
  * Carrier = oscillator modulated by `amodN`
  * `kfmIndex` scales modulation depth
  * `kfmRatio` adjusts the frequency of the modulator relative to the carrier

### Microphone Input (Voice):

```csound
ain inch 1
amic = ain*5
```

* Takes microphone input (scaled for gain).

### PVOC Analysis (Vocoder):

```csound
fs_mod pvsanal amic, 1024, 256, 1024, 1
fs_car pvsanal acar, 1024, 256, 1024, 1
fs_morph pvsfilter fs_car, fs_mod, 1, 5
```

* `pvsanal`: Converts signals to phase vocoder spectra.
* `pvsfilter`: Applies the spectral envelope of the modulator (`amic`) to the carrier (`acar`)—this is the vocoder effect.
* `pvsynth`: Converts the processed spectrum back to audio.

### Output Mixing:

```csound
amix = (aout*3 * kmix) + (acar * (1 - kmix))
outs amix, amix
```

* Mixes vocoder output (`aout`) with clean FM signal (`acar`) using the mix control `kmix`.

## How Csound.js Works

### `csound.js` + WebAssembly

* `csound.js` is a JavaScript wrapper for the Csound WebAssembly module.
* Internally, it:

  * Loads a `.wasm` version of Csound.
  * Exposes functions like `compileOrc`, `start`, `setControlChannel`, and `inputMessage` for JavaScript control.
  * Runs entirely in the browser—no audio server required.
  * Enables real-time synthesis with low latency.

## Summary

This app is a browser-based FM synthesizer and vocoder, integrating:

* Real-time microphone input and spectral morphing
* FM synthesis with 3 voices
* Dynamic control via touch (X/Y)
* Tilt (device orientation) for FM parameters
* All running in the browser with no plugins, thanks to Csound compiled to WebAssembly.


