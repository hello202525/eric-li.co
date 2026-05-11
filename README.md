# eric-li.co

Personal website. Static single-page site hosted on GitHub Pages.

## Structure

```
index.html   — the entire site (HTML, CSS, JS in one file)
404.html     — identical copy (GitHub Pages SPA routing)
CNAME        — custom domain config
```

## Features

### Pages
- **Home** — minimal bio with typographic hierarchy
- **Gallery** — curated art collection in a scattered grid layout
- **Studio** — freeform ink drawing canvas
- **Work** — project showcase

### Easter Eggs

Hidden commands are typed directly on the home screen (no input field — keystrokes are captured globally):

| Command | Effect |
|---------|--------|
| `stars` | Procedural starfield with shooting stars. Blocked from re-triggering while active. |
| `void` | Displays a philosophical phrase in large type on black. Also triggered by 15s idle or mobile shake. |
| `supernova` | WebGL2 GPU fluid simulation — interactive plasma nebula with gravity wells, shockwaves, curl noise, blackbody radiation coloring, chromatic aberration, and filament overlays. Click to inject shockwaves; hold to create gravity wells; move fast to paint plasma trails. |
| `mirror` | Webcam → live Sobel edge-detection ink sketch. |
| `eric` | Opens LinkedIn profile. |

### Hover Interactions
- **"beautiful things"** — letters scatter with random transforms, background shifts to warm parchment
- **"xAI"** — triggers the stars widget with shooting stars
- **"Harvard"** — screen floods deep crimson, text turns white

### Transitions
Every navigation path uses a coordinated 500ms opacity fade with an 800ms background color transition. Immersive modes (void, stars, supernova, mirror) block command input while active.

### Mobile
- Shake detection triggers void mode (with iOS 13+ permission handling)
- 15-second idle timer triggers void mode

## Technical Notes

- **No build system.** Everything is in one HTML file — CSS in a `<style>` tag, JS in a `<script>` tag.
- **WebGL2** is used for the supernova simulation (ping-pong framebuffers, RGBA16F textures, GLSL fragment shaders for Navier-Stokes fluid dynamics).
- **Canvas 2D** is used for the studio drawing tool and the stars widget.
- **Navigator.mediaDevices** is used for the mirror webcam effect.
- **DeviceMotionEvent** is used for mobile shake detection.

## Deployment

Pushes to `main` auto-deploy via GitHub Pages. Custom domain `eric-li.co` is configured via the CNAME file.

## Local Development

Just open `index.html` in a browser. No server required (except for `mirror` which needs HTTPS for webcam access — use `python3 -m http.server` or similar).
