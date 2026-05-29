# Birthday ASCII Art Generator 🎂

A browser-based tool that **converts any image into ASCII art** for use as a personalized birthday greeting — and a standalone [`demo.html`](demo.html) featuring a real-time 3D spinning cake rendered entirely in ASCII.

**[▶ Try the Image Tool](tool.html)** | **[▶ Try the 3D ASCII Demo](demo.html)**

---

## 1. Image-to-ASCII Conversion

### 1.1 Core algorithm

ASCII art converts a raster image to a grid of characters by mapping each pixel's **luminance** to a character from a density ramp, ordered from visually sparse (space) to visually dense (`@`):

```
ramp = " .`-_':,;^=+/|\\<>ivxcl{}?JTLF2ICtaek68mG#@"

for each cell (cx, cy) in the output grid:
    sample a block of pixels from the source image at that position
    lum = average(0.299·R + 0.587·G + 0.114·B)   # perceptual luminance
    char = ramp[ floor(lum / 255 × len(ramp)) ]
```

The perceptual weights (0.299, 0.587, 0.114) are the ITU-R BT.601 luma coefficients, matching human sensitivity — green contributes most because the eye has the most green-sensitive cones.

### 1.2 Canvas rendering

The image is decoded onto an off-screen `<canvas>` element via `drawImage()`. Each cell's pixel block is sampled with `getImageData()`, the average luminance is computed, and the corresponding character is appended to a `<pre>` element. Character density per cell is controlled by the configurable character set.

### 1.3 Image sourcing

- **Cake mode:** fetches a random image URL, draws it to the hidden canvas, converts it.
- **Custom mode:** user-typed subject overrides the source query.

### 1.4 Download

`canvas.toBlob()` serializes the rendered `<canvas>` (with the ASCII overlay drawn as text) to a PNG, triggering a `<a download>` click.

---

## 2. 3D ASCII Spinning Cake — `demo.html`

The demo renders a **torus** (mathematically the shape closest to a cake/donut) as ASCII in real time using a **z-buffer ray-casting** technique popularized by Andy Sloane's "donut.c."

### 2.1 Torus parametrization

A point on the torus surface is parameterized by two angles:

```
x = (R₂ + R₁·cos θ)·cos φ
y = (R₂ + R₁·cos θ)·sin φ
z = R₁·sin θ
```

where *R₁* = tube radius, *R₂* = ring radius.

### 2.2 Rotation & perspective projection

The shape is rotated by two independently incrementing angles *A* (around X) and *B* (around Z) per frame using 2D rotation matrices. After rotation, perspective projection maps 3D coordinates to the 2D character grid:

```
ooz = 1 / (z + camera_distance)          # "one over z"
xp  = COLS/2 + K · ooz · x              # projected column
yp  = ROWS/2 − K · ooz · y              # projected row (Y flipped)
```

### 2.3 Z-buffer and shading

A float z-buffer resolves depth: only the front-most surface point at each cell is rendered. Lambertian shading (dot product of surface normal with a fixed light direction) maps to a luminance index into the character ramp — brighter characters for well-lit faces:

```
normal (at θ, φ) = (cos θ · cos φ,  cos θ · sin φ,  sin θ)
luminance = dot(rotated_normal, light_dir)
char = ramp[ clamp(luminance × len(ramp), 0, len-1) ]
```

### 2.4 Colour

Each rendered character is tinted with an HSL colour derived from the surface's Y-coordinate and θ angle, creating the rainbow-iced "birthday cake" appearance. Candle flame pixels are drawn as bright orange `^`/`*` characters at the top of the torus.

---

## 3. Stack

- Vanilla **JavaScript** + **HTML5 Canvas**
- **jQuery** (image fetch helper in the main tool)
- No build step — single page

---

## 4. Run Locally

```bash
# required for cross-origin image requests
python -m http.server 8000
# open http://localhost:8000
```
