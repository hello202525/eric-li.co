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
| `eric` | Opens LinkedIn profile. |
| Artist surnames | Navigate to that artist's work (e.g. `spilliaert`, `goya`) |
| `gallery` | Opens gallery |
| `studio` | Opens studio |

### Hover Interactions
- **"beautiful things"**: letters scatter with random transforms, background shifts to a random tasteful tone (varies each hover)

### Mobile
- Shake detection triggers void mode (with iOS 13+ DeviceMotionEvent permission handling)
- 15-second idle timer triggers void mode

## Architecture

### Single-File Design

The entire site lives in one `index.html`. CSS is in a `<style>` block, JavaScript in a `<script>` block. There is no build system, no bundler, no framework. The 404.html is an identical copy so that GitHub Pages serves the same SPA regardless of route.

### Routing

Navigation between pages (home, gallery, studio, work, individual artworks) is handled by toggling `.show` classes on overlay sections. Each page-level container is `position: fixed; inset: 0` with `opacity: 0; pointer-events: none` by default, and transitions to `opacity: 1; pointer-events: auto` when shown.

### Transition System

Every navigation path uses a coordinated 500ms opacity fade with an 800ms `background-color` transition on `<body>`. Immersive modes (void, stars, mirror) add body classes (`body.void-mode`, `body.stars-mode`, etc.) that trigger CSS-driven background transitions. The `clearWidget()` function is the single cleanup path for all widgets, ensuring no orphaned canvases, animation frames, or event listeners.

### Widget Lifecycle

All interactive widgets (stars, void, mirror) follow the same pattern:
1. `fireWidget(kind)` is called, which calls `clearWidget()` first (cleans up any prior widget)
2. Sets `activeWidget = kind` globally
3. Runs the widget function which sets up its own canvas/DOM/listeners
4. Widget's `requestAnimationFrame` loop checks `activeWidget` each frame and self-terminates if it changed
5. `clearWidget()` handles all teardown: cancels RAF, removes canvases, stops media streams, removes event listeners via stored cleanup functions

### Stars Widget

Canvas 2D particle system. Background stars drift slowly; shooting stars spawn periodically with velocity streaks rendered via line segments with alpha gradients.

### Mirror Widget

Captures webcam via `navigator.mediaDevices.getUserMedia`, renders each frame to an offscreen canvas, applies a Sobel edge-detection kernel, then draws the result inverted (white edges on black) to a visible canvas.

### Input Blocking

When an immersive widget is active (`stars`, `mirror`), the keydown handler rejects all command input. The user must press Escape to exit first. This prevents accidental command triggers mid-experience.

## Deployment

Pushes to `main` auto-deploy via GitHub Pages. Custom domain `eric-li.co` is configured via the CNAME file.

## Local Development

Open `index.html` in a browser. No server required except for `mirror` which needs HTTPS for webcam access (use `python3 -m http.server` or similar with a self-signed cert).
