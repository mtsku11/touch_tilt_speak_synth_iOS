# Touch Tilt Speak FM Synth iOS

A browser-based Csound instrument for iPhone and iPad. It combines multitouch performance, device tilt, and microphone/vocoder input inside a single WebAssembly-powered page.

Live app: <https://mtsku11.github.io/touch_tilt_speak_synth_iOS/>

## Interaction

- Touch position controls a three-voice FM synth: vertical position selects pitch and horizontal position controls amplitude.
- Device tilt controls FM colour, with gamma mapped to modulation index and beta mapped to modulation ratio.
- Microphone input shapes the FM carrier through a phase-vocoder filter, so speech or breath can colour the synthetic tone.
- `Gesture Glide` controls smoothing between touch changes.
- `Voice Colour` blends in the vocoder path.

## Audio Notes

The synth runs in Csound via `csound.js`. The current version reduces the noisy behaviour that can happen on phone microphones by:

- lowering the default vocoder blend from full wet to a more balanced setting;
- reducing microphone and vocoder gain staging;
- adding DC blocking, high-pass filtering, and low-pass smoothing around the microphone and output paths;
- reducing extreme tilt-driven FM index and ratio values;
- soft-limiting the final output;
- keeping the instrument silent until a touch is active.

These changes keep the vocal/FM behaviour intact while making the instrument less prone to feedback, hiss, and harsh spectral bursts.

## Files

- `index.html` contains the interface, Csound orchestra, and touch/tilt/mic control mapping.
- `js/csound.js` is the bundled Csound WebAssembly wrapper used by the page.

## Running Locally

Serve the folder with a local HTTP server, then open the page on a device/browser that supports microphone and orientation permissions.

```bash
python3 -m http.server 8080
```

Then open `http://localhost:8080/`.
