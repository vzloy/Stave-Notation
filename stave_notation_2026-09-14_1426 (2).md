# Stave Notation for Image Analysis

A diagrammatic calculus for image analysis pipelines,
inspired by Vincent Abbott's Neural Circuit Diagrams
(TMLR 2024) — where wires represent axes, dashed lines
represent tuples, and operations change the shape of data.

Stave Notation adapts this framework from deep learning
to microscopy, adding Penrose graphical calculus structure
and geometric algebra as the native data type.

Key correspondences with Abbott's Neural Circuit Diagrams:

```
Abbott NCD                     Stave Notation
─────────────────────────────────────────────────
Wires = axes                   Lines = container levels
Dashed lines = tuples          Dashed = decomposition / ×
Operations change shape        Boxes change container structure
Broadcasting over axes         Functorial lift (Rule 3)
Inner product contracts        ⊃ cap contracts lines
Outer product extends          ⊂ cap extends lines
```

Reference: V. Abbott, "Neural Circuit Diagrams: Robust
Diagrams for the Communication, Implementation, and
Analysis of Deep Learning Architectures," Transactions
on Machine Learning Research, 2024.
https://openreview.net/forum?id=RyZB4qXEgt
https://github.com/vtabbott/Neural-Circuit-Diagrams

Three rules. Four primitives.

---

## Primitives

```
──    wire      1D container
══    matrix    2D container (spatial, has neighbourhood)
≡≡    volume    3D container
⊃⊂   cap       one shape — direction determines operation
□     box       transform
```

The cap is one shape. Read left to right:

```
⊃  opening right     contract     grade ↓ (GA inner product)
⊂  opening left      extend       grade ↑ (GA outer product)
```

---

## Rules

**1. Name above line. Bottom line closes.**

```
Channels: DAPI, GFP
───────────────────────
ξy,x: 1024
═══════════════════════
ℝ: [0..2¹⁶]
───────────────────────
```

**2. Strokes = dimensions.**

```
──  1D     ══  2D     ≡≡  3D
```

Dashed line between axes means decomposition — an
explicit claim that the matrix separates into independent
axes. This LOSES spatial structure. Not shorthand.

**3. Box sits at the level it transforms.**

This is a theorem: containers are functors, pixel-level
functions lift automatically, structure-consuming
functions cannot be pushed down. □

---

## Two kinds of box input

Every input to a box is either & or ×. The test:
does it share an index with the data?

```
&    linked       shares index     calibration, coordinates,
                                   conditions that vary per element

×    independent  no shared index  parameters, constants,
                                   models, thresholds
```

On paper × is drawn as a dashed line (- - -).

Both & and × inputs enter from the LEFT side of the box.
Data flows left to right through the box. Supporting
inputs join from below:

```
                          ┌──────────┐
& ── Condition/Metadata   │          │
                          │          │
──── Data ───────────────▶│  □ Box   │──▶ Output
                          │          │
× - - Parameter/Model     │          │
                          └──────────┘
```

Control flow and dynamic branching use the same two
inputs. The test is always the same — does it share
an index?

```
Use &:                           Use ×:

LaserPower(t): per frame         Threshold: 100
minSNR(w): per well              Model: Cellpose weights
QualityGate(w): per well         BlendingMode: linear
Schedule(t): per timepoint       MaxIterations: 200
CellType(region): per ROI        OutputFormat: TIFF
```

Example — quality gate (& condition filters data):

```
QualityGate                  Points: n
──────────────               ──────────────
Columns: minSNR   &          ξy,x: 1024
──────────────               ══════════════
ℝ: 3.0                      ℝ: [0..2¹⁶]
──────────────               ──────────────

; τ: Filter (keep if SNR > minSNR)

Points: m ≤ n
──────────────
ξy,x: 1024
══════════════
ℝ: [0..2¹⁶]
──────────────

& because gate shares Points index.
n tiles filtered to m tiles.
```

Example — model selection (× parameter, same for all):

```
                             DAPI
× - - Model                 ──────────────
      ──────────────         ξy,x: 1024
      Columns: weights       ══════════════
      ──────────────         ℝ: [0..2¹⁶]
      ℝ                     ──────────────
      ──────────────

; σ: Cellpose(model=Model)

DAPI
──────────────
ξy,x: 1024
══════════════
ObjectID ∈ ℕ
──────────────

× because model is same for all pixels.
```

Example — adaptive acquisition (& condition with
time dependency):

```
Condition(t-1)               DAPI (frame t)
──────────────               ──────────────
Columns: LaserPower &        t: 1..T
──────────────               ──────────────
ℝ                            ξy,x: 512
──────────────               ══════════════
                             ℝ: [0..2¹²]
                             ──────────────

; σ ; μ: Count ; τ: Decide

Condition(t)
──────────────
Columns: LaserPower
──────────────
ℝ
──────────────

& because condition depends on frame index.
Condition(t) feeds next frame's acquisition.
Feedback = time-dependent &.
```

---

## Calibration as &

```
DAPI                         Calibration: 1
──────────────               ──────────────
ξy,x: 1024       &          Columns: pixelSize, unit
══════════════               ──────────────
ℝ: [0..2¹⁶]                 0.065 μm/px | μm
──────────────               ──────────────
```

μ reads calibration through &. Output is in μm² not px².

---

## Image and table: same structure, 3 lines

```
Image:                       Table:

Channels: DAPI               Records: 1..N
──────────────               ──────────────
ξy,x: 1024                  Columns: ID, Area, Int
══════════════               ──────────────
ℝ: [0..2¹⁶]                 ℝ | ℕ | text
──────────────               ──────────────
```

μ changes ══ to ──. Same depth, different middle:

```
Objects: 1..N                     Records: 1..N
──────────────                    ──────────────
ξy,x: 1024       μ: Measure      Columns: Area, Int
══════════════    ──────────▶     ──────────────
ObjectID ∈ ℕ                      ℝ
──────────────                    ──────────────
```

---

## Data types

Use simple types by default. Use GA when geometry matters.

```
Simple         GA equivalent          When to use GA
───────────────────────────────────────────────────────────
ℝ              s ∈ ⊛⁰Gⁿ              never (scalar is scalar)
ℕ              —                      never
{0,1}          —                      never
text           —                      no GA equivalent
(x,y) ∈ ℝ²    v ∈ ⊛¹G²              when direction matters
(x,y,z) ∈ ℝ³  v ∈ ⊛¹G³              registration, flow fields
Δ₂ triangle    B ∈ ⊛²G³              oriented surfaces, curvature
Δ₃ tet         T ∈ ⊛³G³              oriented volumes
—              S ∈ ⊛¹G⁴˙¹             conformal spheres (CGA)
—              J ∈ G²(ℂ)             polarisation optics
—              M ∈ Gⁿ                full multivector (stress, DTI)
```

### GA grades

```
⊛⁰Gⁿ     scalar         ⊛¹Gⁿ     vector
⊛²Gⁿ     bivector       ⊛³Gⁿ     trivector
Gⁿ        multivector
```

### Common algebras

```
G²    2D plane       G³    3D space       G⁴˙¹   conformal 3D
G²(ℂ) complex        G³˙¹   spacetime
```

### GA products map to the cap

```
Inner a · b    grade ↓    ⊃    projection, flux
Outer a ∧ b    grade ↑    ⊂    span, surface from edges
Geometric ab   mixed      □    rotation, reflection
```

### Rotors

```
R = e^(Bθ/2)    rotation without matrices, any dimension
                 φ-morphism: below ≡≡, structure passes over
```

---

## Why Rule 3 is a theorem

Abbott's container (S ◁ P) is a functor. Any function
on the bottom line lifts for free through all container
levels above. Container lines MUST pass over.

The reverse is impossible. A box that needs spatial
structure cannot be pushed down to pixel level.

The box position is DETERMINED, not chosen. □

---

## Wire operations

All structural. No arithmetic on wires.

```
⊂ (cap left)   copy/extend    one → two
⊃ (cap right)  contract       two → one
×              parallel       independent (dashed on paper)
&              link           shared index
;              sequential     end to end
⋮              recurse        variable depth (specialist)
```

## Box type from line count (Penrose rank)

```
Lines in = Lines out     endomorphism    (φ, σ)
Lines in > Lines out     contraction     (μ)
Lines in < Lines out     expansion       (⊂, ρ)
Lines in = 0             introduction    (ι)
Lines out = 0            elimination     (ε)
```

---

## Morphism types

```
ι   introduce       load data
φ   pixel-level     change values, preserve structure
σ   segment         change pixel meaning (intensity → ID)
μ   measure         collapse spatial into features
τ   restructure     rearrange table, preserve depth
ε   eliminate       save / export
ρ   representation  convert between spatial encodings
```

---

## The & product

Everything vertical. & connects stacks that share an
index set.

### Image & Table

```
DAPI                         Records: 1..M
──────────────               ──────────────
ξy,x: 1024       &          Columns: ID, Area
══════════════               ──────────────
ObjectID ∈ ℕ                 ℝ
──────────────               ──────────────
```

### Graph (Nodes & Edges)

```
Nodes: 1..N                  Edges: 1..E
──────────────               ──────────────
Columns: data     &          Columns: src, tgt
──────────────               ──────────────
ℝ                            {1..N}²
──────────────               ──────────────
```

### Oriented mesh (GA)

```
Vertices: 1..V               Faces: 1..F
──────────────               ──────────────
Columns: pos       &         Columns: v₁,v₂,v₃
──────────────               ──────────────
v ∈ ⊛¹G³                    B ∈ ⊛²G³ (normal)
──────────────               ──────────────
```

---

## Object representations

```
Raster              ξy,x ══ ObjectID ∈ ℕ
Spots (simple)      Objects ── Columns ── (x,y,z,r) ∈ ℝ⁴
Spots (CGA)         Objects ── Columns ── S ∈ ⊛¹G⁴˙¹
Contour             Objects ── Vertices:k ── (y,x) ∈ ℝ²
Surface mesh        Objects ── Faces:F ── B ∈ ⊛²G³
Volume mesh         Objects ── Cells:C ── T ∈ ⊛³G³
```

## Dependent cardinality

```
Points: 25         fixed. ND array.
Points(w): 1..Nw   variable. Container.
```

---


## Multi-scale spatial hierarchy

Pyramids, quadtrees, octrees — all use existing features.
No new symbols.

### OME-Zarr pyramid

```
Dataset
──────────────
Levels: 0..L
──────────────
Channels: DAPI, GFP
──────────────
ξz(l),y(l),x(l): varies per level
≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡
ℝ: [0..2¹⁶]
──────────────

       &

Transforms: 0..L
──────────────
Columns: scale_z, scale_y, scale_x
──────────────
ℝ
──────────────
```

Spatial extent depends on level: `(l)` dependency.
Transforms & linked — shares Levels index.

### Quadtree (adaptive subdivision)

```
Root: 1
──────────────
Quadrants: 1..4
──────────────
⋮
──────────────
Quadrants: 1..4
──────────────
ξy,x(depth): varies
══════════════
ℝ: [0..2¹⁶]
──────────────
```

⋮ with branching factor 4. Octree = same with 8 and ≡≡.

Or as graph (what software stores):

```
Nodes: 1..N                  Children: 1..E
──────────────               ──────────────
Columns: level,    &         Columns: parent, child
  y₀, x₀, size              ──────────────
──────────────               {1..N}²
ℝ                            ──────────────
──────────────
```

### Structure summary

```
Pyramid      Levels + (l) dependency     dependent cardinality
Quadtree     ⋮ with branching 4          recursive depth
Octree       ⋮ with branching 8          recursive depth
Coord maps   & linked transforms         shared index
Adaptive     graph with &                nodes & children
```

---

## Protocols

### 1. Cell counting

```
DAPI                        DAPI                         Records: 1
──────────────              ──────────────               ──────────────
ξy,x: 1024    σ: Otsu      ξy,x: 1024     μ: Count     Columns: Count
══════════════ ─────────▶   ══════════════ ──────────▶   ──────────────
ℝ: [0..2¹⁶]                ObjectID ∈ ℕ                 ℕ
──────────────              ──────────────               ──────────────
```

### 2. Spot counting per cell (FISH)

```
DAPI                          DAPI
──────────────                ──────────────
ξy,x: 1024    σ: Threshold   ξy,x: 1024
══════════════ ──────────▶    ══════════════
ℝ: [0..2¹⁶]                  NucleiID ∈ ℕ
──────────────                ──────────────

       ×

Cy5                           Cy5
──────────────                ──────────────
ξy,x: 1024    σ: SpotDetect  ξy,x: 1024
══════════════ ──────────▶    ══════════════
ℝ: [0..2¹⁶]                  SpotID ∈ ℕ
──────────────                ──────────────

Both ; μ: Children

Records: 1..N
──────────────
Columns: ParentID, SpotCount
──────────────
ℕ
──────────────
```

### 3. Colocalization

```
Ch1: GFP ══──  ┐
               │  ┌───────────┐     Records: 1
               ├──│μ: Pearson │──▶  ──────────────
               │  │  Manders  │     Columns: P, M1, M2
Ch2: mCh ══── ┘  └───────────┘     ──────────────
                                    ℝ
                                    ──────────────
```

### 4. Ratio imaging (FRET / Calcium)

```
F340 ══── ┐
          │  ┌────────┐     Ratio
          ├──│φ: ÷    │──▶  ══════════
          │  │pixel   │     ℝ: [0..1]
F380 ══── ┘  └────────┘     ──────────

; μ: FieldMeas per frame

Records: 1..T
──────────────
Columns: MeanRatio
──────────────
ℝ
──────────────
```

### 5. Tracking

```
DAPI
──────────────
t: 1..100
──────────────
ξy,x: 1024
══════════════
ℝ: [0..2¹⁶]
──────────────

; σ per frame ; μ: TimeAndCenter

Detections: 1..D             Links: 1..L
──────────────               ──────────────
Columns: x,y,t   &          Columns: d₁,d₂,type
──────────────               ──────────────
ℝ                            {1..D}² × {move,split,merge}
──────────────               ──────────────

; μ: MotionFeatures

Records: 1..K
──────────────
Columns: Speed, MSD
──────────────
ℝ
──────────────
```

### 6. 3D volume segmentation

```
Channels: DAPI, Ki67
──────────────
ξz,y,x: 64, 512, 512
≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡
ℝ: [0..2¹⁶]
──────────────

; σ: Cellpose3D ; μ: Meas

Records: 1..N
──────────────
Columns: Vol, SurfArea, Ki67int
──────────────
ℝ
──────────────
```

### 7. HCS plate (JOBS)

```
Plate: 384-well
──────────────
Wells: w ⊆ {A1..P24}
──────────────
Points(w): 1..Nw
──────────────
Channels: DAPI, GFP
──────────────
ξy,x: 1024
══════════════
ℝ: [0..2¹⁶]
──────────────

; σ: Cellpose ; φ: Clean ; μ: ObjectMeas
; τ: Accum ; τ: Join wells ; τ: Reduce/well
; τ: Z′-factor ; τ: DoseResponse → EC₅₀
; ε: Save
```

### 8. Neuron branching (Sholl)

```
GFP
──────────────
ξy,x: 2048
══════════════
ℝ: [0..2¹⁶]
──────────────

; φ: Denoise ; σ: Threshold ; ρ: Skeletonize

Junctions: 1..J              Branches: 1..B
──────────────               ──────────────
Columns: y,x      &          Columns: j₁,j₂
──────────────               ──────────────
ℝ²                           {1..J}²
──────────────               ──────────────
                                    &
                             Paths: 1..P
                             ──────────────
                             Columns: y,x
                             ──────────────
                             ℝ²
                             ──────────────

; μ: Sholl              & ; μ: BranchMeas

Radii: 1..R             Records: 1..B
──────────────           ──────────────
Columns: Intersections   Columns: Length, Order
──────────────           ──────────────
ℕ                        ℝ
──────────────           ──────────────
```

### 9. Tissue analysis (cell adjacency)

```
DAPI
──────────────
ξy,x: 4096
══════════════
ℝ: [0..2¹⁶]
──────────────

; σ: Cellpose ; μ: ObjectMeas + Voronoi

Cells: 1..N                  Adjacency: 1..E
──────────────               ──────────────
Columns: Area, Type &        Columns: c₁, c₂
──────────────               ──────────────
ℝ                            {1..N}²
──────────────               ──────────────

; μ: GraphFeatures

Records: 1..N
──────────────
Columns: Degree, Clustering
──────────────
ℝ
──────────────
```

### 10. Oriented surface (curvature) — GA

```
MembraneGFP
──────────────
ξz,y,x: 128, 512, 512
≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡
ℝ: [0..2¹⁶]
──────────────

; σ: Cellpose3D ; ρ: Marching cubes

Vertices: 1..V               Faces: 1..F
──────────────               ──────────────
Columns: pos       &         Columns: v₁,v₂,v₃
──────────────               ──────────────
v ∈ ⊛¹G³                    B ∈ ⊛²G³
──────────────               ──────────────

; μ: Curvature

Records: 1..N
──────────────
Columns: MeanCurv, GaussCurv
──────────────
ℝ
──────────────
```

### 11. Light sheet fusion — GA

```
View₁                     View₂
──────────────            ──────────────
ξz,y,x: 512              ξz,y,x: 512
≡≡≡≡≡≡≡≡≡≡≡≡              ≡≡≡≡≡≡≡≡≡≡≡≡
v ∈ ⊛¹G³                 v ∈ ⊛¹G³
──────────────            ──────────────

; φ: Rotor R₁₂ on View₂ ; ⊃ fuse

Fused
──────────────
ξz,y,x: 512
≡≡≡≡≡≡≡≡≡≡≡≡
v ∈ ⊛¹G³
──────────────
```

### 12. Polarisation — complex Clifford

```
Channels: H, V
──────────────
ξy,x: 1024
══════════════
J ∈ G²(ℂ)
──────────────

; φ: Waveplate(δ) ; μ: Stokes

Records: 1
──────────────
Columns: S₀, S₁, S₂, S₃
──────────────
ℝ
──────────────
```

### 13. Tiling / stitching

```
Points: n
──────────────
ξy,x: 1024
══════════════
ℝ: [0..2¹⁶]
──────────────

       &                          ×

Points: n                    Parameter
──────────────               ──────────────
Columns: stageX, stageY      Overlap: 10%
──────────────               ──────────────
(x,y) ∈ ℝ²
──────────────

; ρ: Stitch

ξY,X: n × 1024
══════════════
ℝ: [0..2¹⁶]
──────────────

& = stage coords (share Points index)
× = overlap (same for all tiles)
Points consumed. n tiles → one image.
```

### 14. FRAP (bleach recovery)

```
GFP
──────────────
t: 1..T
──────────────
ξy,x: 512
══════════════
ℝ: [0..2¹²]
──────────────

       &                          &

ROI: 1                       BleachEvent: 1
──────────────               ──────────────
Columns: y₀,x₀,r            Columns: t_bleach
──────────────               ──────────────
ℝ³                           ℕ
──────────────               ──────────────

; μ: MeanIntensity (within ROI, per frame)

Records: 1..T
──────────────
Columns: t, MeanInt
──────────────
ℝ
──────────────

; τ: Normalise (pre-bleach = 1.0)
; τ: Fit (I(t) = I∞(1 - e^(-t/τ)))

Records: 1
──────────────
Columns: τ_half, MobileFraction, I∞
──────────────
ℝ
──────────────

ROI is & (shares spatial index).
BleachEvent is & (shares time index).
T frames → 3 fitted parameters.
```

### 15. FLIM (fluorescence lifetime)

Per-pixel phasor (no spatial binning):

```
DAPI
──────────────────────────────────────────
ξy,x: 256
══════════════════════════════════════════
                ┌──────────────┐
τ_bins: 256     │φ: Phasor FFT │  Columns: G, S
──────────────  └──────────────┘  ──────────────
Photons ∈ ℕ                       ℝ²
──────────────                    ──────────────

Box below ══: per-pixel, spatial passes over.
256 bins → 2 phasor coordinates.
```

With spatial binning (needs neighbours):

```
DAPI
──────────────  ┌──────────────────┐  DAPI
ξy,x: 256      │                    │  ξy,x: 64
══════════════  │ μ: SpatialBin +   │  ══════════════
τ_bins: 256     │    PhasorFit      │  Columns: G, S
──────────────  │                    │  ──────────────
Photons ∈ ℕ    └──────────────────┘  ℝ²
──────────────                        ──────────────

Box spans ══: uses spatial neighbours.
Resolution reduced (256 → 64).
```

Rule 3 decides. Does the operation need spatial
neighbours? If no → below ══. If yes → spans ══.

### 16. Wound healing / scratch assay

```
Phase contrast
──────────────
t: 1..T
──────────────
ξy,x: 1024
══════════════
ℝ: [0..2¹⁶]
──────────────

; σ: Threshold per frame (cell vs gap)

Phase contrast
──────────────
t: 1..T
──────────────
ξy,x: 1024
══════════════
{cell, gap} ∈ {0,1}
──────────────

; μ: GapArea per frame

Records: 1..T
──────────────
Columns: t, GapArea, GapWidth
──────────────
ℝ
──────────────

; τ: Fit (closure rate)

Records: 1
──────────────
Columns: ClosureRate, T_half
──────────────
ℝ
──────────────

σ per frame, μ per frame, τ fits across frames.
```

### 17. PALM/STORM (single molecule localisation)

```
Frames: 1..10000
──────────────
ξy,x: 256
══════════════
ℝ: [0..2¹⁶]
──────────────

; σ: SpotDetect per frame
; μ: GaussFit per spot (coordinates + precision)

Localisations: 1..L
──────────────
Columns: x, y, frame, intensity, precision
──────────────
ℝ
──────────────

10000 frames × ══ collapsed into table.
Massive μ: spatial grid → coordinate list.
```

Two possible continuations:

Render super-resolution image (ρ: table → image):

```
; ρ: Render (histogram of localisations)

ξy,x: 2560 (10× oversampled)
══════════════
ℝ: [0..1] (density)
──────────────

ρ goes TABLE → IMAGE. Reverse direction of μ.
```

Or cluster analysis (graph):

```
Localisations: 1..L          Clusters: 1..C
──────────────               ──────────────
Columns: x,y,prec  &         Columns: l₁,l₂
──────────────               ──────────────
ℝ                            {1..L}² (DBSCAN)
──────────────               ──────────────

; μ: ClusterFeatures

Records: 1..C
──────────────
Columns: Npoints, Radius, Density
──────────────
ℝ
──────────────
```

### 18. Spectral unmixing

```
Channels: λ₁..λ₃₂
──────────────────────────────────────────
ξy,x: 512
══════════════════════════════════════════
                 ┌────────────────┐
ℝ: [0..2¹⁶]     │φ: LinearUnmix  │  ℝ: [0..1]
──────────────── └────────────────┘ ──────────────

                       ×

                 Reference spectra
                 ──────────────
                 Columns: DAPI,GFP,mCherry,AutoFluor
                 ──────────────
                 ℝ (per λ)
                 ──────────────

φ below ══: per pixel, spatial passes over.
Top line changes: wavelengths → components.
Reference spectra enter as × (same for all pixels).

Output:

Components: DAPI, GFP, mCherry, AutoFluor
──────────────
ξy,x: 512
══════════════
ℝ: [0..1] (abundance)
──────────────
```

### 19. Organoid morphology (nested objects)

```
Channels: DAPI, Ecad, Laminin
──────────────
ξz,y,x: 200, 1024, 1024
≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡
ℝ: [0..2¹⁶]
──────────────

; σ: Cellpose3D on DAPI → CellID ∈ ℕ
; σ: OrganoidDetect on Ecad → OrganoidID ∈ ℕ

Nested objects: cells within organoids.

Organoids: 1..K              Cells: 1..N
──────────────               ──────────────
Columns: Volume,   &         Columns: OrganoidID, CellID
  Sphericity                 ──────────────
──────────────               ℕ × ℕ
ℝ                            ──────────────
──────────────

OrganoidID ∈ {1..K} in cell table =
foreign key to organoid. Bipartite &.

; μ: OrganoidFeatures

Records: 1..K
──────────────
Columns: CellCount, Volume, Sphericity,
         LumenPresent, Polarity
──────────────
ℝ | {0,1}
──────────────
```

### 20. Batch processing (map over files)

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

Files: 1..F
──────────────
Columns: condition, replicate
──────────────
text
──────────────

; (σ ; μ) per file

Records: 1..F
──────────────
Columns: Filename, CellCount, MeanArea
──────────────
ℝ | text
──────────────

; τ: GroupBy condition ; τ: Statistics

Records: 1..C (conditions)
──────────────
Columns: Condition, Mean, SD, N
──────────────
ℝ | text
──────────────

Files as top container. Pipeline applied per file.
Condition metadata via &. Statistics across
replicates is τ. This is daily facility workflow.
```


### 21. Multi-scale pyramid (OME-Zarr)

```
Dataset
──────────────
Levels: 0..4
──────────────
Channels: DAPI, GFP
──────────────
ξy,x(l): 4096 / 2ˡ
══════════════
ℝ: [0..2¹⁶]
──────────────

       &

Transforms: 0..4
──────────────
Columns: scale_y, scale_x
──────────────
ℝ
──────────────

; σ at level 2 (coarse, 1024×1024)

Level 2
──────────────
ξy,x: 1024
══════════════
ObjectID ∈ ℕ
──────────────

; ρ: Upsample ROIs to level 0

Level 0 (ROIs only)
──────────────
ξy,x: 4096
══════════════
ℝ: [0..2¹⁶]
──────────────

; μ: Precise measurement at full resolution

Records: 1..N
──────────────
Columns: Area, Int (at full res)
──────────────
ℝ (calibrated via & transforms)
──────────────

Coarse detection, precise measurement.
Level selection = choosing from Levels container.
ρ between levels maps coordinates via & transforms.
```


---

## Composition and Named Pipelines

### Sequential composition (;)

Any ;-chain can be named and used as a single box.
This is function definition — the fundamental
abstraction mechanism.

Define by writing the expansion once:

```
SegMeas = σ: Cellpose ; μ: Measure
```

Use as one box:

```
DAPI                         Records: 1..N
──────────────               ──────────────
ξy,x: 1024    SegMeas        Columns: Area, Int
══════════════ ──────────▶   ──────────────
ℝ: [0..2¹⁶]                 ℝ
──────────────               ──────────────
```

The type signature is determined by the first input
and last output of the chain:

```
SegMeas:      ══ ℝ → ── ℝ     (image → table)
CleanSeg:     ══ ℝ → ══ ℕ     (image → labels)
Preprocess:   ══ ℝ → ══ ℝ     (image → image)
```

### Parameters on composed functions

Each function inside a composite has its own × inputs.
When collapsed, selected parameters are exposed.
The rest become internal defaults:

```
Expanded (all parameters visible):

                ×                        ×

         Model                    MinSize
         ──────────────           ──────────────
         Columns: weights         Columns: min_area
         ──────────────           ──────────────
         ℝ                       50 px
         ──────────────           ──────────────

DAPI                    DAPI
──────────────          ──────────────
ξy,x: 1024   σ:Cellpose ξy,x: 1024   φ:Clean    ...  ; μ:Measure
══════════════          ══════════════
ℝ: [0..2¹⁶]            ObjectID ∈ ℕ
──────────────          ──────────────
```

```
Collapsed (parameters exposed on the composite):

                ×              ×

         Model            MinSize

DAPI                                Records: 1..N
──────────────                      ──────────────
ξy,x: 1024    SegMeas(Model,        Columns: Area, Int
══════════════  MinSize)             ──────────────
ℝ: [0..2¹⁶]                        ℝ
──────────────                      ──────────────
```

Parentheses after the name list the exposed ×
parameters. Parameters not listed use internal
defaults.

### & inputs on composed functions

Some composites need & inputs (linked, shared index).
These are exposed the same way:

```
CalibratedMeas(Calibration) =
    σ: Cellpose ; μ: Measure(Calibration)

DAPI                     Calibration: 1
──────────────           ──────────────
ξy,x: 1024              Columns: pixelSize
══════════════    &      ──────────────
ℝ: [0..2¹⁶]             0.065 μm/px
──────────────           ──────────────

; CalibratedMeas

Records: 1..N
──────────────
Columns: Area (μm²), Int
──────────────
ℝ
──────────────

The & passes through the composite to μ inside.
```

### Common named composites

```
Name              Definition                      Type
──────────────────────────────────────────────────────────────
Preprocess        φ: Denoise ; φ: BgSub           ══ ℝ → ══ ℝ
CleanSeg          σ ; φ: Clean ; φ: FillHoles     ══ ℝ → ══ ℕ
SegMeas           σ ; μ: Measure                  ══ ℝ → ── ℝ
SegMeasClean      σ ; φ: Clean ; μ: Measure       ══ ℝ → ── ℝ
TrackAnalyse      σ/frame ; μ ; τ: Link ; μ       ══ ℝ → ── ℝ
HCSPipeline       σ ; μ ; τ: Accum ; τ: Reduce    ══ ℝ → ── ℝ
DoseResponse      τ: Group ; τ: Fit ; τ: Z′       ── ℝ → ── ℝ
QualityGate       ⊂ ; τ: Filter ; τ: Filter       ── → ── × ──
```

### Nesting — composites contain composites

```
FacilityBatch =
    ι: WatchFolder
  ; Preprocess(σ=2)
  ; SegMeas(Model=cyto2, MinSize=50)
  ; τ: Aggregate
  ; ε: CSV ; ε: Slack

Five lines. Expands to twelve boxes.
Full detail exists but drawn only when needed.
```

### Parameter override at call site

A composite has defaults. Override at the call site:

```
Definition:
  SegMeas(Model=cyto2, MinSize=50)

Call with defaults:
  ; SegMeas

Call with override:
  ; SegMeas(Model=cyto3, MinSize=100)
```

Same as function call with keyword arguments.
Default = × defined inside. Override = × supplied
from outside at the call site.

### Expansion rule

Any named composite can be expanded in place.
The expansion is guaranteed to produce the same
type signature. This is referential transparency:

```
; SegMeas(Model=cyto2)

expands to exactly:

                ×
         Model=cyto2

; σ: Cellpose(model=Model)
; μ: Measure
```

No side effects. Same input → same output.
The composite IS its expansion.

---

## Platform translation

| Box | napari / scikit-image         | ImageJ               | GA3                    | IMARIS              |
|-----|-------------------------------|-----------------------|------------------------|---------------------|
| ι   | `tifffile.imread()`           | `open()`             | `Src.Channels`         | File > Open         |
| φ   | `skimage.filters.*`           | `run("Median...")`   | `Pre.Smooth`           | Image Processing    |
| σ   | `threshold_otsu()` / cellpose | `setAutoThreshold()` | `Seg.Threshold`        | Surfaces / Spots    |
| μ   | `regionprops_table()`         | `Analyze Particles`  | `Meas2.ObjectFeatures` | Statistics tab      |
| τ   | `pd.merge()`, `groupby()`    | *(manual)*           | `Data.Reduce`          | Vantage             |
| ε   | `df.to_csv()`                | `saveAs("Results")`  | `Sink.SaveTables`      | Export              |
| ρ   | `marching_cubes()`           | —                    | —                      | Surface from labels |

GA: `clifford` (Python), `ganja.js` (web), `klein` (C++).

The same primitives describe database query plans
(JOIN = ⊃, GROUP BY = μ, WHERE = τ, INDEX = &).

Design on paper. Execute anywhere.
