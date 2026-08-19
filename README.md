# Computer Graphics — CVI 2026-2

Code demos for **Computación Visual Interactiva** (ISIS4823, Universidad de los
Andes, 2026-2), following Marschner & Shirley, *Fundamentals of Computer
Graphics*.

One demo per topic. Each demo is a **single, self-contained HTML file** — no
build step, no dependencies, no server. Open it by double-clicking.

## Demos

| # | Folder | Topic | Chapters |
|---|--------|-------|----------|
| 1 | [`Demo 1 - Math`](./Demo%201%20-%20Math/) | Mathematical foundations | 2, 6, 7 |
| 2 | [`Demo 2 - Raster Images`](./Demo%202%20-%20Raster%20Images/) | Raster images — chromatic aberration | 3 |

### Demo 1 — One 3×3 matrix does everything

An interactive transformation playground. It starts at the identity, so every
change you make is visibly something you added.

- A gradient triangle, rasterized **per pixel with barycentric coordinates** —
  the same computation a GPU runs for every triangle it draws. The inside test
  and the colour gradient fall out of the same three numbers.
- The live **3×3 matrices** are shown as an equation, `M = T · R · H · S`. Each
  slider is tinted the same colour as the matrix cells it writes into, so you can
  see exactly which control touches which entry.
- The red and green arrows are `M·e₁` and `M·e₂` — literally the first two
  columns of the matrix.
- `det(M)` is the area scale factor: negative means the triangle was flipped,
  zero means the plane was crushed onto a line and there is no inverse.
- Switching the composition order to `S · H · R · T` uses the same four factors
  and gives a different matrix — multiplication does not commute — while the
  determinant stays put.

### Demo 2 — Chromatic aberration

Simulates the colour fringing a real lens produces, driven by actual glass
dispersion data rather than an arbitrary offset.

- **Cauchy's equation** `n(λ) = A + B/λ²` gives a refractive index per channel;
  the lensmaker's equation turns that into a focal length, and different focal
  lengths mean different magnification. That is lateral chromatic aberration.
  Pick BK7, BaF10 or SF10 and watch the dispersion change.
- The test chart carries **concentric arcs on the left and radial spokes on the
  right, at the same radii**. The displacement is radial, so the arcs fringe hard
  and the spokes stay clean — the `Δ` view proves it. The small rings at the
  optical centre never fringe at all, because `|Δ| = r(1 − 1/s)` is zero there.
- A draggable **probe** magnifies any edge, and the **channel profile** plots R, G
  and B along that row: the three curves come out shifted relative to each other,
  which is the aberration measured in pixels.
- Two switches connect this to the rest of the chapter: resampling in **linear
  light vs. sRGB** (gamma), and **bilinear vs. nearest** reconstruction (filtering).

Sanity check on the optics: the demo's Abbe-style number for BK7 comes out at
63.7 against a published value of about 64.2.

## How it is written

Plain JavaScript and the Canvas 2D API. No framework, no libraries.

The linear algebra is implemented from scratch — matrix multiplication,
determinant by cofactor expansion, inverse via the adjugate, and the affine
primitives — because that math *is* the topic. The triangle goes through a
hand-written barycentric rasterizer writing straight into an `ImageData` buffer;
Canvas 2D only strokes the overlays.

Conventions used in the code:

- matrices are **row-major**, `m[row][col]`
- points are **column vectors**, so a transform is `p' = M · p`
- therefore `A · B` means *apply B first, then A*

Parameters are declared once in a `PARAMS` array and every control is generated
from that declaration, so adding a slider is a one-line change.

## License

[MIT](./LICENSE)
