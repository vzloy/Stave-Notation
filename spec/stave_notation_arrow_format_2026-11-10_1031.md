# Stave Notation — Arrow Format

**Version: 2026-11-10_1031**

Alternative rendering when boxes are hard to draw.
Stave snapshots connected by labeled arrows.
Same semantics, no box-drawing characters needed.

Format:

```
stave snapshot
      |
      | morphism: Name (what changes)
      ↓
stave snapshot
```

---

## 1. Cell counting

```
Channels: DAPI
──────────────
ξy,x: 1024
══════════════
ℝ: [0..2¹⁶]
──────────────
      |
      | σ: Otsu — Channels consumed, spatial preserved, ℝ→ℕ
      ↓
ξy,x: 1024
══════════════
ObjectID ∈ ℕ
──────────────
      |
      | μ: Count — spatial consumed
      ↓
Records: 1
──────────────
Count ∈ ℕ
──────────────
```


## 2. Segmentation and measurement

```
Channels: DAPI
──────────────
ξy,x: 1024
══════════════
ℝ: [0..2¹⁶]
──────────────
      |
      | σ: Cellpose — Channels consumed, spatial preserved, ℝ→ℕ
      ↓
ξy,x: 1024
══════════════
ObjectID ∈ ℕ
──────────────
      |
      | μ: Measure — spatial consumed, Objects:N created, reads PixelSize via &
      ↓
Objects: N
──────────────
{Area:ℝ(μm²), Circ:ℝ, MeanInt:ℝ}
──────────────
```


## 3. Segmentation with calibration (&)

```
Channels: DAPI
──────────────
ξy,x: 1024
══════════════
ℝ: [0..2¹⁶]
──────────────
&
{PixelSize:ℝ(μm)}
──────────────
      |
      | σ: Cellpose — touches image only, metadata passes over
      ↓
ξy,x: 1024
══════════════
ObjectID ∈ ℕ
──────────────
&
{PixelSize:ℝ(μm)}                ← unchanged, passed over σ
──────────────
      |
      | μ: Measure — spans both & branches, reads PixelSize
      ↓
Objects: N
──────────────
{Area:ℝ(μm²), Circ:ℝ}           ← calibrated via &
──────────────
```


## 4. Channel split (× independence)

```
Channels: {DAPI, A488}
──────────────
ξy,x: 1024
══════════════
ℝ: [0..2¹⁶]
──────────────
      |
      | τ: Split — dissolves Channels container, & becomes ×
      ↓
DAPI                             A488
──────────────                   ──────────────
ξy,x: 1024                      ξy,x: 1024
══════════════    × (independent) ══════════════
ℝ: [0..2¹⁶]                     ℝ: [0..2¹⁶]
──────────────                   ──────────────
      |                                |
      | σ: Cellpose                    | σ: Cellpose
      ↓                                ↓
DAPI                             A488
──────────────                   ──────────────
ξy,x: 1024                      ξy,x: 1024
══════════════    × (independent) ══════════════
ObjectID ∈ ℕ                     ObjectID ∈ ℕ
──────────────                   ──────────────
```


## 5. Colocalization (&)

```
Ch1: GFP                        Ch2: mCherry
──────────────                   ──────────────
ξy,x: 1024                      ξy,x: 1024
══════════════   & (co-indexed)  ══════════════
ℝ: [0..2¹⁶]                     ℝ: [0..2¹⁶]
──────────────                   ──────────────
      |
      | μ: Pearson + Manders — spans both & branches
      ↓
Records: 1
──────────────
{P:ℝ, M1:ℝ, M2:ℝ}
──────────────
```


## 6. Ratio imaging (FRET / Calcium)

```
Ch: F340                         Ch: F380
──────────────                   ──────────────
ξy,x: 1024                      ξy,x: 1024
══════════════   & (co-indexed)  ══════════════
ℝ: [0..2¹⁶]                     ℝ: [0..2¹⁶]
──────────────                   ──────────────
      |
      | φ: ÷ pixel — per-pixel, below ══
      ↓
ξy,x: 1024
══════════════
ℝ: [0..1]
──────────────
      |
      | μ: FieldMeas — per frame, spatial consumed
      ↓
Records: 1..T
──────────────
MeanRatio ∈ ℝ
──────────────
```


## 7. Tracking

```
DAPI
──────────────
t: 1..100
──────────────
ξy,x: 1024
══════════════
ℝ: [0..2¹⁶]
──────────────
      |
      | σ: Cellpose per frame — spatial preserved, ℝ→ℕ
      ↓
t: 1..100
──────────────
ξy,x: 1024
══════════════
ObjectID ∈ ℕ
──────────────
      |
      | μ: TimeAndCenter — spatial consumed, creates detections
      ↓
Detections: 1..D                 Links: 1..L
──────────────                   ──────────────
Columns: x, y, t  & (graph)     Columns: d₁, d₂, type
──────────────                   ──────────────
ℝ                                {1..D}² × {move,split,merge}
──────────────                   ──────────────
      |
      | μ: MotionFeatures — graph consumed
      ↓
Records: 1..K
──────────────
{Speed:ℝ, MSD:ℝ}
──────────────
```


## 8. HCS plate (384-well)

```
Plate: 384-well                  ← level 1
──────────────
Wells: w ⊆ {A1..P24}            ← level 2
──────────────
Points(w): 1..Nw                 ← level 3
──────────────
Channels: DAPI, GFP              ← level 4
──────────────
ξy,x: 1024                      ← level 5
══════════════
ℝ: [0..2¹⁶]                     ← level 6
──────────────
      |
      | σ: Cellpose — Channels consumed, spatial preserved, ℝ→ℕ
      ↓
Plate: 384-well
──────────────
Wells: w ⊆ {A1..P24}
──────────────
Points(w): 1..Nw
──────────────
ξy,x: 1024                      ← still 1024×1024
══════════════
ObjectID ∈ ℕ                     ← 5 levels
──────────────
      |
      | φ: Clean
      ↓
      |
      | μ: ObjectMeas — spatial consumed, Objects:N created
      ↓
Plate: 384-well
──────────────
Wells: w ⊆ {A1..P24}
──────────────
Points(w): 1..Nw
──────────────
Objects: N                       ← 4 levels
──────────────
{Area, Circ} ∈ ℝ
──────────────
      |
      | τ: Accum — Objects consumed
      ↓
Plate: 384-well
──────────────
Wells: w ⊆ {A1..P24}
──────────────
Points(w): 1..Nw                 ← 3 levels
──────────────
{CellCount, MeanArea} ∈ ℝ
──────────────
      |
      | τ: Join wells — Points consumed
      ↓
Plate: 384-well
──────────────
Wells: w ⊆ {A1..P24}            ← 2 levels
──────────────
{CellCount, MeanArea} ∈ ℝ
──────────────
      |
      | τ: Z′-factor + DoseResponse — Wells consumed
      ↓
Plate: 384-well                  ← 1 level
──────────────
{Z′:ℝ, EC₅₀:ℝ}
──────────────
      |
      | ε: Save
      ↓
```


## 9. FRAP (bleach recovery)

```
GFP
──────────────
t: 1..T
──────────────
ξy,x: 512
══════════════
ℝ: [0..2¹²]
──────────────
&
ROI: {y₀, x₀, r} ∈ ℝ³
──────────────
&
BleachEvent: {t_bleach} ∈ ℕ
──────────────
      |
      | μ: MeanIntensity per frame — reads ROI via &
      ↓
Records: 1..T
──────────────
{t:ℕ, MeanInt:ℝ}
──────────────
      |
      | τ: Normalise (pre-bleach = 1.0)
      ↓
      |
      | τ: Fit — I(t) = I∞(1 − e^(−t/τ))
      ↓
Records: 1
──────────────
{τ_half:ℝ, MobileFraction:ℝ, I∞:ℝ}
──────────────
```


## 10. PALM/STORM

```
Frames: 1..10000
──────────────
ξy,x: 256
══════════════
ℝ: [0..2¹⁶]
──────────────
      |
      | σ: SpotDetect per frame
      ↓
      |
      | μ: GaussFit per spot — massive collapse: 10000 × ══ → table
      ↓
Localisations: 1..L
──────────────
{x:ℝ, y:ℝ, frame:ℕ, intensity:ℝ, precision:ℝ}
──────────────
```

Path A — super-resolution image:
```
      |
      | ρ: Render — table → image (reverse of μ)
      ↓
ξy,x: 2560 (10× oversampled)
══════════════
ℝ: [0..1] (density)
──────────────
```

Path B — cluster analysis:
```
      |
      | τ: DBSCAN — creates graph via &
      ↓
Localisations: 1..L             Clusters: 1..C
──────────────                   ──────────────
{x, y, prec} ∈ ℝ  & (graph)    {l₁, l₂} ∈ {1..L}²
──────────────                   ──────────────
      |
      | μ: ClusterFeatures
      ↓
Records: 1..C
──────────────
{Npoints:ℕ, Radius:ℝ, Density:ℝ}
──────────────
```


## 11. Contour → Mesh (ρ creates inner &)

```
Objects: N
──────────────
Points: Mₙ
──────────────
(y, x) ∈ ℝ²
──────────────
      |
      | ρ: Triangulate — 1 branch in, 2 branches out
      ↓
Objects: N
──────────────
  Vertices: V                    Faces: F
  ──────────────                 ──────────────
  (x,y,z) ∈ ℝ³   & (inner)     (v₁,v₂,v₃) ∈ {1..V}³
  ──────────────                 ──────────────
```

ρ is the only morphism that changes & branch count.
Faces reference Vertices via {1..V}³ (foreign key).


## 12. Batch processing (map over files)

```
Files: 1..F
──────────────
  Channels: DAPI, GFP
  ──────────────
  ξy,x: 1024
  ══════════════
  ℝ: [0..2¹⁶]
  ──────────────
  &
  {condition:Text, replicate:Text}
  ──────────────
      |
      | σ ; μ per file — metadata passes over
      ↓
Files: 1..F
──────────────
  {CellCount:ℕ, MeanArea:ℝ}
  ──────────────
  &
  {condition:Text, replicate:Text}
  ──────────────
      |
      | τ: GroupBy condition + Statistics — reads condition via &
      ↓
Conditions: 1..C
──────────────
{Condition:Text, Mean:ℝ, SD:ℝ, N:ℕ}
──────────────
```

Files:K once at top, both branches indented inside.
τ reads condition through & → groups F files into C conditions.


## 13. Spectral unmixing

```
Channels: λ₁..λ₃₂
──────────────
ξy,x: 512
══════════════
ℝ: [0..2¹⁶]
──────────────
×
Reference spectra: {DAPI, GFP, mCherry, AutoFluor} ∈ ℝ
──────────────
      |
      | φ: LinearUnmix — per-pixel (below ══), × enters independently
      ↓
Components: DAPI, GFP, mCherry, AutoFluor
──────────────
ξy,x: 512
══════════════
ℝ: [0..1] (abundance)
──────────────
```


## 14. Organoid morphology (nested objects)

```
Channels: DAPI, Ecad, Laminin
──────────────
ξz,y,x: 200, 1024, 1024
≡≡≡≡≡≡≡≡≡≡≡≡≡≡
ℝ: [0..2¹⁶]
──────────────
      |
      | σ: Cellpose3D ; σ: OrganoidDetect
      ↓
Organoids: 1..K                  Cells: 1..N
──────────────                   ──────────────
{Vol, Spher} ∈ ℝ  & (bipartite) {OrgID, CellID} ∈ ℕ×ℕ
──────────────                   ──────────────

OrgID ∈ {1..K} = foreign key

      |
      | μ: OrganoidFeatures
      ↓
Records: 1..K
──────────────
{CellCount:ℕ, Vol:ℝ, Sphericity:ℝ, LumenPresent:{0,1}, Polarity:ℝ}
──────────────
```


## 15. Tissue analysis (cell adjacency)

```
DAPI
──────────────
ξy,x: 4096
══════════════
ℝ: [0..2¹⁶]
──────────────
      |
      | σ: Cellpose
      ↓
      |
      | μ: ObjectMeas + Voronoi — creates graph via &
      ↓
Cells: 1..N                     Adjacency: 1..E
──────────────                   ──────────────
{Area, Type} ∈ ℝ  & (graph)    {c₁, c₂} ∈ {1..N}²
──────────────                   ──────────────
      |
      | μ: GraphFeatures — graph consumed
      ↓
Records: 1..N
──────────────
{Degree:ℝ, Clustering:ℝ}
──────────────
```

---

## Drawing guide

On paper with pen:

1. Write the stave snapshot vertically (container labels + lines)
2. Draw an arrow ↓ below it
3. Write the morphism name next to the arrow
4. Write what changes in parentheses
5. Write the next stave snapshot

For & : write both branches side by side, write & between them
For × : write both branches side by side, write × or dashed line between them

The arrow replaces the box. Everything else stays the same.
