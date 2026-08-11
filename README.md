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
