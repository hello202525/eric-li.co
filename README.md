# eric-li.co

Personal website. Static single-page site hosted on GitHub Pages.

## Structure

```
index.html   the entire site (HTML, CSS, JS in one file)
404.html     identical copy (GitHub Pages SPA routing)
CNAME        custom domain config
works/       artwork images for the gallery
```

## Features

### Pages
- **Home**: minimal bio with typographic hierarchy
- **Gallery**: curated art collection in a dark-room grid layout
- **Studio**: freeform ink drawing canvas
- **Work**: individual artwork viewer with parallax

### Easter Eggs

Hidden commands are typed directly on the home screen. There is no visible input field; keystrokes are captured globally via a `keydown` listener that buffers input and pattern-matches against known commands.

| Command | Effect |
|---------|--------|
| `stars` | Procedural starfield with shooting stars. Blocked from re-triggering while active. |
| `void` | Philosophical phrase in large type on black. Also triggered by 15s idle or mobile shake. |
| `mirror` | Webcam to live Sobel edge-detection ink sketch. |
| `play` | Multi-phase space journey: rocket launch → warp travel → Mars fluid sim → zero-g drift game → welcome home. |
| `time` | Generative luxury watch face. |
| `eric` | Opens LinkedIn profile. |
| Artist surnames | Navigate to that artist's work (e.g. `spilliaert`, `goya`) |
| `gallery` | Opens gallery |
| `studio` | Opens studio |

### Play Sequence

A multi-phase interactive experience rendered on Canvas 2D + WebGL2:

1. **Launch**: Rocket lifts off from ground with ~11k parallax stars in 5 layers, procedural exhaust particles, progressive acceleration.
2. **Warp**: First-person hyperspace tunnel with layered streaking stars, atmospheric entry streaks as Mars approaches.
3. **Crash (Mars)**: Interactive WebGL2 fluid plasma simulation. The planet rotates, has textured magma with dark crust/hot pools/fissure lines, tendrils escaping the sphere edges. Click/drag creates shockwaves and swirling vortices.
4. **Drift (Game)**: Third-person astronaut floating through space. Collect glowing orbs for chime sounds. Arrow keys to move with elastic soft boundaries. Parallax star layers, debris, nebula.
5. **Ending**: White wash → sky-blue-white screen → "Welcome home." typewriter text → seamless fade to homepage.

### Hover Interactions
- **"beautiful things"**: letters scatter with random transforms, background shifts to a random tasteful tone, birds fly across the screen in a loose flock. On mouseleave, birds instantly dissolve into fine ash particles that drift down and fade.

### Audio

Procedural Web Audio synthesis throughout the play sequence — launch rumble, warp drone, continuous Mars hum, ambient drift pads, soft pentatonic chimes on orb collection. All generated in real-time, no audio files.

### Mobile
- Shake detection triggers void mode (with iOS 13+ DeviceMotionEvent permission handling)
- 15-second idle timer triggers void mode

## Architecture

### Single-File Design

The entire site lives in one `index.html`. CSS is in a `<style>` block, JavaScript in a `<script>` block. There is no build system, no bundler, no framework. The 404.html is an identical copy so that GitHub Pages serves the same SPA regardless of route.

### Routing

Navigation between pages (home, gallery, studio, work, individual artworks) is handled by toggling `.show` classes on overlay sections. Each page-level container is `position: fixed; inset: 0` with `opacity: 0; pointer-events: none` by default, and transitions to `opacity: 1; pointer-events: auto` when shown.

### Transition System

Every navigation path uses a coordinated 500ms opacity fade with an 800ms `background-color` transition on `<body>`. Immersive modes (void, stars, mirror, play, time) add body classes that trigger CSS-driven background transitions. The `clearWidget()` function is the single cleanup path for all widgets, ensuring no orphaned canvases, animation frames, or event listeners.

### Widget Lifecycle

All interactive widgets follow the same pattern:
1. `fireWidget(kind)` is called, which calls `clearWidget()` first (cleans up any prior widget)
2. Sets `activeWidget = kind` globally
3. Runs the widget function which sets up its own canvas/DOM/listeners
4. Widget's `requestAnimationFrame` loop checks `activeWidget` each frame and self-terminates if it changed
5. `clearWidget()` handles all teardown: cancels RAF, removes canvases, stops media streams, removes event listeners via stored cleanup functions

### WebGL2 Fluid Simulation (Mars)

The crash phase uses a GPU-accelerated fluid simulation:
- **SIM_FRAG**: Navier-Stokes-like simulation with advection, diffusion, pressure correction, curl noise, buoyancy, mouse interaction (gravity pull, energy injection, swirl vortex, shockwave ring on click), global rotation, and multiple orbiting energy sources.
- **DISP_FRAG**: Visualization with temperature-squared color mapping for sharp hot/cold contrast, dark cooled crust, glowing molten pools, fissure lines, crack networks, chromatic aberration, bloom, spherical shading, and tendril rendering beyond the planet edge.
- Ping-pong RGBA16F framebuffers for GPU state.

### Input Blocking

When an immersive widget is active (`stars`, `mirror`, `play`, `time`), the keydown handler rejects all command input. The user must press Escape to exit first.

## Deployment

Pushes to `main` auto-deploy via GitHub Pages. Custom domain `eric-li.co` is configured via the CNAME file.

## Local Development

Open `index.html` in a browser. No server required except for `mirror` which needs HTTPS for webcam access (use `python3 -m http.server` or similar with a self-signed cert).
