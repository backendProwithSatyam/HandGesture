# HandGesture – Sword Swarm

Control thousands of glowing "flying swords" with your bare hand through the webcam.
Move your hand and the swarm follows; change the gesture and the swarm changes formation.

## Gestures

| Gesture | Effect |
| --- | --- |
| ✋ Open palm | Swords orbit your hand in rotating rings |
| ✊ Fist | Swords gather into a tight core |
| ☝️ Point (index finger) | Swords stream behind your fingertip like a whip |
| ✌️ Two fingers | Swords form a sweeping wave across the screen |
| 🤏 Pinch | Swords burst outward, then hover around your fingers |

No camera? Click **use the mouse instead**, move the mouse and press `1`–`5` to switch gestures.

## Run locally

The webcam only works on `https://` or `localhost` (not by double-clicking the file), so serve the folder:

```bash
python -m http.server 5500
```

Then open http://localhost:5500 in Chrome and click **Start camera**.

## Deploy (free) on Vercel

```bash
npx vercel          # preview deploy (first time asks you to log in)
npx vercel --prod   # production URL
```

Or push to GitHub → vercel.com → **Add New → Project** → import the repo → Framework Preset **Other** → Deploy.

---

## How this project is built

**Type:** a static, front-end-only web app (one `index.html`). No backend, no database, no build step.
Everything (camera, AI hand tracking, graphics) runs in the visitor's browser.

### Stack

| Part | Technology | Why |
| --- | --- | --- |
| Page | HTML + CSS + vanilla JavaScript (ES modules) | No framework needed, loads instantly |
| Camera | `navigator.mediaDevices.getUserMedia` (WebRTC) | Built into every modern browser |
| Hand tracking (AI) | [Google MediaPipe Tasks Vision – Hand Landmarker](https://ai.google.dev/edge/mediapipe/solutions/vision/hand_landmarker), loaded from the jsDelivr CDN | Detects 21 points on the hand in real time, on the GPU |
| Graphics | HTML5 Canvas 2D with additive blending (`lighter`) | Draws ~6000 glowing streaks at 60 fps |
| Hosting | Any static host (Vercel, Netlify, GitHub Pages) | HTTPS is required for the camera |

### How it works (pipeline)

1. **Camera → frames.** The webcam stream plays in a hidden `<video>` element (480×360).
2. **Frames → hand points.** For every new camera frame (`requestVideoFrameCallback`) MediaPipe returns 21 landmarks (wrist, knuckles, finger joints, tips) with x/y in 0–1.
3. **Points → gesture.** `classify()` measures each fingertip's distance from its knuckle, relative to hand size, to decide which fingers are up. Pinch = thumb tip close to index tip (with hysteresis so it doesn't flicker), and is checked before fist so only one gesture is active at a time. A gesture must appear on 2 frames in a row before it switches.
4. **Gesture → target shape.** Each mode gives every sword a target position: rings around the palm, a ball, a trail of past fingertip positions, a sine wave, or an explosion.
5. **Physics.** Every sword is a particle with position + velocity, pulled toward its target by a spring with damping. Each sword has a slightly different "lag", which makes the swarm feel alive.
6. **Rendering.** Each sword is drawn as a short line pointing along its velocity; faster swords are brighter. The canvas is faded a little each frame instead of cleared, which leaves motion trails.
7. **Low latency.** The hand position is predicted forward between camera frames from its velocity, smoothing adapts to speed, and the number of swords lowers automatically if FPS drops below 45.

### Build something similar yourself

1. Create `index.html` with a full-screen `<canvas>` and a hidden `<video>`.
2. Ask for the camera with `getUserMedia` and play it in the video.
3. Load MediaPipe `HandLandmarker` (`runningMode: 'VIDEO'`) and call `detectForVideo()` on each frame.
4. Mirror the x coordinate (`1 - x`) so it feels like a mirror, then map it to screen pixels.
5. Write simple finger rules (tip far from knuckle = finger up) to turn landmarks into gestures.
6. Keep particles in `Float32Array`s, update them with a spring toward a target, draw them with Canvas 2D.
7. Serve over `localhost` while developing, deploy to Vercel/Netlify for a public HTTPS link.

### Ideas to extend

- Two hands (`numHands: 2`) controlling two swarms
- WebGL / Three.js for 50k+ particles or 3D depth using the landmark `z` value
- Sound effects on gesture change
- Custom shapes (text, logos) the swords fly into
