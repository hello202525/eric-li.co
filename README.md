# eric-li.co

Personal website. Static single-page site hosted on GitHub Pages.

## Structure

```
index.html   the entire site (HTML, CSS, JS in one file)
404.html     identical copy (GitHub Pages SPA routing)
CNAME        custom domain config
```

## Features

### Pages
- **Home**: minimal bio with typographic hierarchy
- **Gallery**: curated art collection in a scattered grid layout
- **Studio**: freeform ink drawing canvas
- **Work**: project showcase

### Easter Eggs

Hidden commands are typed directly on the home screen. There is no visible input field; keystrokes are captured globally via a `keydown` listener that buffers input and pattern-matches against known commands.

| Command | Effect |
|---------|--------|
| `stars` | Procedural starfield with shooting stars. Blocked from re-triggering while active. |
| `void` | Philosophical phrase in large type on black. Also triggered by 15s idle or mobile shake. |
| `supernova` | WebGL2 GPU fluid simulation. Interactive plasma nebula. |
| `mirror` | Webcam to live Sobel edge-detection ink sketch. |
| `eric` | Opens LinkedIn profile. |

### Hover Interactions
- **"beautiful things"**: letters scatter with random transforms, background shifts to warm parchment
- **"xAI"**: triggers the stars widget with shooting stars
- **"Harvard"**: screen floods deep crimson, text turns white

### Mobile
- Shake detection triggers void mode (with iOS 13+ DeviceMotionEvent permission handling)
- 15-second idle timer triggers void mode

## Architecture

### Single-File Design

The entire site lives in one `index.html`. CSS is in a `<style>` block, JavaScript in a `<script>` block. There is no build system, no bundler, no framework. The 404.html is an identical copy so that GitHub Pages serves the same SPA regardless of route.

### Routing

Navigation between pages (home, gallery, studio, work, individual artworks) is handled by toggling `.show` classes on overlay `<div>`s. Each page-level container is `position: fixed; inset: 0` with `opacity: 0; pointer-events: none` by default, and transitions to `opacity: 1; pointer-events: auto` when shown. This gives full-page transitions without actual page loads.

### Transition System

Every navigation path uses a coordinated 500ms opacity fade with an 800ms `background-color` transition on `<body>`. Immersive modes (void, stars, supernova, mirror) add body classes (`body.void-mode`, `body.stars-mode`, etc.) that trigger CSS-driven background transitions. The `clearWidget()` function is the single cleanup path for all widgets, ensuring no orphaned canvases, animation frames, or event listeners.

### Widget Lifecycle

All interactive widgets (stars, void, supernova, mirror) follow the same pattern:
1. `fireWidget(kind)` is called, which calls `clearWidget()` first (cleans up any prior widget)
2. Sets `activeWidget = kind` globally
3. Runs the widget function which sets up its own canvas/DOM/listeners
4. Widget's `requestAnimationFrame` loop checks `activeWidget` each frame and self-terminates if it changed
5. `clearWidget()` handles all teardown: cancels RAF, removes canvases, stops media streams, removes event listeners via stored cleanup functions

### Supernova (WebGL2 Fluid Simulation)

The supernova easter egg is a real-time Navier-Stokes fluid simulation running entirely on the GPU:

- **Ping-pong framebuffers**: two RGBA16F textures alternate as read/write targets each frame. Channels encode: `xy` = velocity, `z` = density, `w` = temperature.
- **Simulation shader** (`SIM_FRAG`): performs advection (backward particle trace), diffusion (neighbor averaging), pressure correction (divergence removal), curl noise injection (multi-scale rotational turbulence), buoyancy, and decay per texel per frame.
- **Display shader** (`DISP_FRAG`): reads the simulation state and renders it with blackbody radiation coloring (approximate Planck locus), chromatic aberration (per-channel UV offset near gravity wells), multi-tap bloom with diagonal sampling, wispy filament overlay (6-octave turbulent noise), and ACES filmic tone mapping with shadow/highlight color grading.
- **Interaction**: mouse position is passed as a uniform. Moving the mouse injects density and temperature along the trail. Clicking fires a radial shockwave that persists for 15 frames. Holding creates a strong gravity well (8x multiplier).
- **Requires** `EXT_color_buffer_float` for rendering to float textures.

### Stars Widget

Canvas 2D particle system. Background stars drift slowly; shooting stars spawn periodically with velocity streaks rendered via line segments with alpha gradients. Accepts a `persist` option for the xAI hover interaction.

### Mirror Widget

Captures webcam via `navigator.mediaDevices.getUserMedia`, renders each frame to an offscreen canvas, applies a Sobel edge-detection kernel, then draws the result inverted (white edges on black) to a visible canvas.

### Input Blocking

When an immersive widget is active (`stars`, `void`, `supernova`, `mirror`), the keydown handler rejects all command input. The user must press Escape to exit first. This prevents accidental command triggers mid-experience.

## Deployment

Pushes to `main` auto-deploy via GitHub Pages. Custom domain `eric-li.co` is configured via the CNAME file.

## Local Development

Open `index.html` in a browser. No server required except for `mirror` which needs HTTPS for webcam access (use `python3 -m http.server` or similar with a self-signed cert).
