
# Touch + Tilt FM Synth

This example demonstrates how to use touch and tilt controls in a web browser to manipulate a polyphonic FM synthesizer, using Csound running in WebAssembly (WASM).

## Goals
This project integrates real-time touch interaction and motion sensors to control a three-voice FM synth:

1. Load and initialize the Csound WASM engine  
2. Compile a polyphonic FM synthesis instrument  
3. Handle multitouch input to control pitch and amplitude  
4. Use device tilt (via DeviceOrientation) to modulate FM depth and ratio  
5. Add stereo random panning to spatialize the voices  

## CsoundObj API Usage
This app uses the following CsoundObj methods:

1. `Csound()` – Create a new Csound engine object  
2. `.setOption()` – Configure real-time audio and buffer sizes  
3. `.compileOrc()` – Compile FM synth instrument code  
4. `.start()` – Start the audio engine  
5. `.inputMessage()` – Schedule a persistent instrument (`i1 0 -1`)  
6. `.setControlChannel()` – Dynamically control FM parameters  

## JavaScript Overview
All application logic is contained in the `<script type="module">` section of `index.html`. No external build tools or frameworks are required. This includes:

- Asynchronous initialization of Csound  
- UI event listeners for touch and slider inputs  
- Motion permission and sensor event handling  

### Touch Handling
```javascript
pad.addEventListener("touchstart", handleTouch);
pad.addEventListener("touchmove", handleTouch);
pad.addEventListener("touchend", handleTouchEnd);
```
Each touch is mapped to a fixed musical scale (Y axis → pitch) and X axis (amplitude). Touches are assigned to one of three voices. When the touch ends, the corresponding amplitude fades out smoothly via `portk()`.

### Tilt Mapping
```javascript
const gammaNorm = Math.abs(gamma) / 90;
const betaNorm = (beta + 90) / 180;
const fmIndex = gammaNorm * 0.8;
const ratios = [0.5, 1, 1.5, 2, 3, 4];
const fmRatio = ratios[Math.min(Math.floor(betaNorm * ratios.length), ratios.length - 1)];
```
- Gamma (left/right tilt) controls FM **index**  
- Beta (forward/backward tilt) selects one of several musical FM **ratios**  

### Csound Instrument
```csound
instr 1
  kamp1 = portk(chnget:k("amp1"), krelease)
  kfreq1 = chnget:k("freq1")
  ...
  kfmRatio = portk(chnget:k("fmRatio"), 0.1)
  kfmFreq1 = kfreq1 * kfmRatio
  ...
  amod1 oscili kfmIndex * kfmFreq1, kfmFreq1
  acar1 oscili kamp1, kfreq1 + amod1
```
The modulation frequency is dynamically linked to the carrier: `modFreq = carrier * ratio`, ensuring harmonic overtones. Three identical voices are mixed together.

### Stereo Panning
```csound
apan1 random 0, 1
aleft = acar1 * apan1
aright = acar1 * (1 - apan1)
```
Each voice is randomly panned across the stereo field on each reinitialization. Voices are mixed and scaled to prevent clipping.

## HTML Layout
```html
<div id="starter">
  <button id="startAllBtn">▶️ Start Synth + Tilt</button>
  <input id="releaseSlider" type="range" min="0.01" max="0.3" />
</div>
<div id="xy-pad"></div>
```
A button starts the synth and requests motion permissions. A slider controls the release smoothing time for amplitude changes.

## Permissions
iOS requires:
- **User gesture** to enable motion sensing (via `DeviceOrientationEvent.requestPermission()`)  
- **HTTPS origin** for both motion and microphone access  

These are handled as soon as the user presses the start button.

## Deployment
You can host this app as a static website:

### GitHub Pages
1. Push your files to a new repo  
2. Go to **Settings > Pages** and choose the main branch root  
3. Visit `https://your-username.github.io/your-repo-name`  

## Testing & Debugging
Open the browser console to test real-time values and issue direct Csound messages:
```js
csound.inputMessage("i1 0 1")
csound.setControlChannel("fmIndex", 0.5)
```

## Conclusion
This app showcases a full-featured, mobile-first FM synth powered by WebAssembly and gesture input. You can touch and tilt to explore harmonics, motion-based modulation, and spatial sound. It's a minimal but expressive interface for exploring interactive audio in the browser.

---

**Happy tilting and tweaking! 🎛️📱**
