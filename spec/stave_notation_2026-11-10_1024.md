# Stave Notation for Image Analysis

**Version: 2026-11-10_1024**

A diagrammatic calculus for image analysis pipelines,
grounded in the cartesian double category Dbl(Img).

Four structural primitives. Seven morphism types. Three rules.

Inspired by Vincent Abbott's Neural Circuit Diagrams
(TMLR 2024) — visual grammar where lines = functors.
Categorical semantics (type row, morphism types, container
nesting, &, ×, Rule 3, double-category structure) is original.

Literature connections confirmed:
- Sheet diagrams (Comfort, Delpeuch, Hedges, arXiv:2010.13361) — two monoidal products nesting
- Tape diagrams (Bonchi et al., arXiv:2210.09950) — string diagrams of string diagrams
- Functorial boxes (Melliès, CSL 2006) — box contains sub-diagram, wires cross boundary
- String diagrams for double categories (Myers, arXiv:1612.02762) — horizontal pipeline, vertical container depth

Validated: ~1,100 references, 20 fields, ~330 composites, 0 new primitives needed.
The structural grammar of bioimage analysis is finite and closed.

---

## Structural Primitives

```
⊂    copy       diagonal Δ : X → X×X
&    fibred     product in the fibre (co-indexed, shared container)
×    plain      product with no indexing constraint (independent)
□    terminal   discard  ! : X → 1
```

⊂ (opening left, diverge) = copy. The defining primitive
of cartesian structure. One wire in, two wires out.

⊃ DROPPED. Was Abbott's contraction symbol for bilinear
neural operations (attention, dot-products). In Dbl(Img)
every contraction is a morphism (μ etc.), not a structural
primitive. Not needed.

### Lines — container depth

```
──    wire      1D container (list, time series, table rows)
══    matrix    2D container (spatial grid — has neighbourhood)
≡≡    volume    3D container
```

Strokes = dimensions. ══ is literally two ── stacked
(Y and X axes). Count of lines above the type row =
nesting depth of containers.

---

## Three Rules

**Rule 1 — Lines are data, boxes are functions.**

Every visual element is one or the other. No decorative
boxes or rectangles. Every element carries semantic meaning.

**Rule 2 — Pipeline flows left to right.**

Sequential composition (;) is horizontal. Like a musical
score — left to right, no backtracking.

**Rule 3 — The box sits at the level it operates on.**

A theorem: containers are functors. Pixel-level functions
lift automatically; structure-consuming functions cannot be
pushed down. The box position is DETERMINED, not chosen.

Box height on paper = which fibre levels of the double
category the 2-cell spans:
- Short box = local operation on one branch
- Tall box = reads across & branches (e.g. μ reading PixelSize)

---

## & vs ×

Both are products. The difference is co-indexing.

```
&    fibred product     shared container above both branches
×    plain product      no shared container, independent
```

The shared container in & appears ONCE, above both branches.
Never repeated. Repeating it (e.g. Files:K on both sides)
means two independent collections = ×, not &.

```
Files:K                          ← shared container, once
  ══ ℝ ──                        ← image branch
  &                              ← & sits INSIDE Files:K
  {PixelSize:ℝ, Condition:Text}  ← metadata branch
  ──
```

& and × are related by the fibration:
- × restricted to the same fibre = &
- & is ×, plus the information that both share a container
- Split dissolves & into × (forgets co-indexing)
- Merge introduces & from × (adds shared container)

Both nest freely at any depth. The test is always:
"does it share an index with the data?"

---

## Drawing Rules

**Line continuity.** Lines run continuously left to right
(like musical stave lines). Boxes sit ON lines they
transform. Lines passing a box are unaffected. Lines
starting at a box = data created. Lines ending at a
box = data consumed.

**Box attachment.** Boxes attach at branch level, not to
individual wires. Inner & (e.g. Vertices & Faces inside
a mesh) is NOT attached to the box edge. The inner
structure is drawn on the output side as part of the
branch, preserving hierarchy.

**Hierarchy is spatial.** Between boxes, the hierarchy is
the spatial arrangement of lines on the page: which
lines are above which, which are indented under which
container. The nesting just IS. No box needed to maintain it.

**At a box, hierarchy can change:**
- Lines entering and exiting → transformed (type change)
- Lines entering and not exiting → consumed (μ collapsed it)
- Lines exiting but not entering → created (σ, ρ made it)
- Inner & appearing on output side → box created nesting

---

## Two Kinds of Box Input

Every input to a box is either & or ×.

```
&    linked       shares index     calibration, coordinates,
                                   conditions that vary per element

×    independent  no shared index  parameters, constants,
                                   models, thresholds
```

On paper × is drawn as a dashed line (- - -).
× and dashed line are interchangeable.

---

## Data Types

### Scalar

```
ℝ              real (intensity, area, distance)
ℕ              natural (ObjectID, label, count)
ℝ: [0..2¹⁶)   ranged real (pixel value with bit depth)
{0,1}          binary (presence/absence)
Text           string (filename, condition)
```

### Coordinate

```
(y,x) ∈ ℝ²    2D point
(x,y,z) ∈ ℝ³  3D point
```

### Record (flat product of named fields)

```
{Area:ℝ, Circ:ℝ, FileName:Text}
```

No container of its own — depth 0. Use for grouped
metadata: {FileName:Text, PixelSize:ℝ, Objective:Text}.
Separate & branches only needed when components have
different container structure.

### Reference (foreign key into another container)

```
ℕ ∈ {1..N}        index into parent
ℕ³ ∈ {1..V}³      triple of indices (mesh face → vertices)
{1..N}²            adjacency (graph edges)
```

### Geometric Algebra (aspirational — constrains computation)

```
Cl₁(3,0)        direction (grade-1 vector)
Spin(3)          orientation (rotor, forces geometric product)
Cl⁺(3,0,1)      pose (PGA motor)
```

### Category-enriched (names the target category)

```
(ℝ², ω)    symplectic (forces symplectomorphisms)
(ℝ, g)     Riemannian (forces diffeomorphisms)
P(ℝ)       statistical (probability, forces non-expansive maps)
ℝ_max      tropical semiring
```

### Type row

Type annotation sits between container lines and value
line. Answers "what kind of value lives at each position."
Catches value-level mistakes: Gaussian blur on ℕ labels,
arithmetic mean on Spin(3) rotors.

---

## Morphism Types

### ι — introduce

```
Brings data into the pipeline
Creates container lines (no input wires, only output)
ι: ReadND2, ι: WatchFolder, ι: Acquire
```

### φ — filter/transform

```
Type-preserving: ℝ → ℝ, ℕ → ℕ
Box sits below container lines (Rule 3: spatial passes over)
fmap — applied per element, container structure unchanged
φ: GaussianBlur, φ: BackgroundSubtract, φ: Clean
```

### σ — segment

```
Type-changing: ℝ → ℕ
Intensity becomes labels (label mask, same spatial grid)
PRESERVES spatial structure — output is still 1024×1024
Consumes Channels if it selects one
Does NOT create Objects:N (that is μ's job)
σ: Cellpose, σ: Threshold, σ: StarDist
```

### μ — measure

```
Structure-collapsing: consumes container lines
Spatial (══) → flat record ({Area, Circ})
Creates Objects:N by grouping pixels by ObjectID
Reads & branches for calibration (PixelSize → μm²)
Tall box — spans multiple levels
μ: ObjectMeas, μ: IntensityStats, μ: RegionProps
```

### τ — restructure

```
Changes container organisation, not values
Reindex, group, sort, accumulate, join, reduce
Operates at the container level it restructures
τ: GroupBy, τ: Accum, τ: Join, τ: Z′-factor, τ: DoseResponse
```

### ρ — change representation

```
Same data, different mathematical structure
Can change number of & branches (one in, two out or vice versa)
The ONLY morphism that rewires & structure
Point list → triangle mesh (creates inner &)
Table → image (PALM/STORM reconstruction)
Graph → tree or tree → graph
ρ: Triangulate, ρ: Render, ρ: Skeletonise
```

### ε — export

```
Sends data out of the pipeline
Save file, send notification, write to database
Consumes wires (no output, or side-effect only)
ε: CSV, ε: Save, ε: Slack, ε: GoogleSheets
```

### Summary table

```
       Changes type?  Changes structure?  Changes & branches?
φ      no             no                  no
σ      ℝ→ℕ           no (preserves ══)   no
μ      any            consumes container  no
τ      no             reorganises         no
ρ      any            any                 YES
ι      —              creates from nothing —
ε      —              consumes to nothing  —
```

---

## Calibration as &

```
══ ℝ ──                          ← image (has spatial structure)
&
{PixelSize:ℝ} ──                 ← metadata (flat, no container)
```

μ reads PixelSize through &. Output is in μm² not px².
The & guarantees they belong to the same file.

---

## Image and Table: Same Structure, 3 Lines

```
Image:                       Table:

Channels: DAPI               Records: 1..N
──────────────               ──────────────
ξy,x: 1024                  Columns: ID, Area, Int
══════════════               ──────────────
ℝ: [0..2¹⁶]                 ℝ | ℕ | text
──────────────               ──────────────
```

μ changes ══ to ──. Same depth, different middle line.

---

## Dependent Cardinality

```
Points: 25         fixed. ND array
Points(w): 1..Nw   variable. Container
```

---

## Recursive / Variable Depth

```
⋮    W-type (well-founded tree, variable nesting)
```

Used for: quadtrees, octrees, lineage trees, tissue
hierarchies, OME-Zarr pyramids. Every path from root
to leaf is finite, but the tree isn't uniform.

---

## Graphs

No special symbol needed. Two &-linked branches where
edge data references the node index set via the type row:

```
Nodes: 1..N                  Edges: 1..E
──────────────               ──────────────
Columns: data     &          Columns: src, tgt
──────────────               ──────────────
ℝ                            {1..N}²
──────────────               ──────────────
```

---

## Composition and Named Pipelines

### Sequential composition (;)

Any ;-chain can be named and used as a single box:

```
SegMeas = σ: Cellpose ; μ: Measure
```

Type signature determined by first input and last output.
Each internal function keeps its own × parameters.
When collapsed, selected × exposed in parentheses:
SegMeas(Model, MinSize).

Composites nest: FacilityBatch calls SegMeas which
calls σ. Expansion is referentially transparent —
the composite IS its expansion.

### Common named composites

```
Preprocess      φ: Denoise ; φ: BgSub           ══ ℝ → ══ ℝ
CleanSeg        σ ; φ: Clean ; φ: FillHoles     ══ ℝ → ══ ℕ
SegMeas         σ ; μ: Measure                  ══ ℝ → ── ℝ
TrackAnalyse    σ/frame ; μ ; τ: Link ; μ       ══ ℝ → ── ℝ
HCSPipeline     σ ; μ ; τ: Accum ; τ: Reduce    ══ ℝ → ── ℝ
DoseResponse    τ: Group ; τ: Fit ; τ: Z′       ── ℝ → ── ℝ
QualityGate     ⊂ ; τ: Filter ; τ: Filter       ── → ── × ──
```

---

## Protocols

Each protocol shows vertical stave snapshots at structural
transition points, connected by ; morphism chains.
Input stave → ; morphisms → Output stave.


### 1. Cell counting

Input:
```
Channels: DAPI
──────────────
ξy,x: 1024
══════════════
ℝ: [0..2¹⁶]
──────────────
```

; σ: Otsu

σ output (spatial preserved, type changed):
```
Channels: DAPI
──────────────
ξy,x: 1024
══════════════
ObjectID ∈ ℕ
──────────────
```

; μ: Count

Output:
```
Records: 1
──────────────
Columns: Count
──────────────
ℕ
──────────────
```


### 2. Spot counting per cell (FISH)

Two independent (×) input channels:
```
Channels: DAPI               Channels: Cy5
──────────────               ──────────────
ξy,x: 1024       ×          ξy,x: 1024
══════════════               ══════════════
ℝ: [0..2¹⁶]                 ℝ: [0..2¹⁶]
──────────────               ──────────────
```

; σ: Threshold (on DAPI) × σ: SpotDetect (on Cy5)

After σ:
```
DAPI                         Cy5
──────────────               ──────────────
ξy,x: 1024       ×          ξy,x: 1024
══════════════               ══════════════
NucleiID ∈ ℕ                SpotID ∈ ℕ
──────────────               ──────────────
```

; μ: Children (assigns spots to parent nuclei)

Output:
```
Records: 1..N
──────────────
Columns: ParentID, SpotCount
──────────────
ℕ
──────────────
```


### 3. Colocalization

Input (two channels, & = same spatial grid):
```
Ch1: GFP
──────────────
ξy,x: 1024
══════════════
ℝ: [0..2¹⁶]
──────────────
&
Ch2: mCherry
──────────────
ξy,x: 1024
══════════════
ℝ: [0..2¹⁶]
──────────────
```

; μ: Pearson + Manders (spans both & branches)

Output:
```
Records: 1
──────────────
Columns: P, M1, M2
──────────────
ℝ
──────────────
```


### 4. Ratio imaging (FRET / Calcium)

Input:
```
Ch: F340
──────────────
ξy,x: 1024
══════════════
ℝ: [0..2¹⁶]
──────────────
&
Ch: F380
──────────────
ξy,x: 1024
══════════════
ℝ: [0..2¹⁶]
──────────────
```

; φ: ÷ pixel (per-pixel ratio, below ══)

After φ:
```
Ratio
──────────────
ξy,x: 1024
══════════════
ℝ: [0..1]
──────────────
```

; μ: FieldMeas per frame

Output:
```
Records: 1..T
──────────────
Columns: MeanRatio
──────────────
ℝ
──────────────
```


### 5. Tracking

Input:
```
DAPI
──────────────
t: 1..100
──────────────
ξy,x: 1024
══════════════
ℝ: [0..2¹⁶]
──────────────
```

; σ: Cellpose per frame ; μ: TimeAndCenter

After μ (graph: detections & links):
```
Detections: 1..D             Links: 1..L
──────────────               ──────────────
Columns: x, y, t  &         Columns: d₁, d₂, type
──────────────               ──────────────
ℝ                            {1..D}² × {move,split,merge}
──────────────               ──────────────
```

; μ: MotionFeatures

Output:
```
Records: 1..K
──────────────
Columns: Speed, MSD
──────────────
ℝ
──────────────
```


### 6. 3D volume segmentation

Input:
```
Channels: DAPI, Ki67
──────────────
ξz,y,x: 64, 512, 512
≡≡≡≡≡≡≡≡≡≡≡≡≡≡
ℝ: [0..2¹⁶]
──────────────
```

; σ: Cellpose3D

After σ (spatial preserved, ≡≡ unchanged):
```
DAPI
──────────────
ξz,y,x: 64, 512, 512
≡≡≡≡≡≡≡≡≡≡≡≡≡≡
ObjectID ∈ ℕ
──────────────
```

; μ: Measure

Output:
```
Records: 1..N
──────────────
Columns: Vol, SurfArea, Ki67int
──────────────
ℝ
──────────────
```


### 7. HCS plate (384-well)

Input (6 container levels):
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
```

; σ: Cellpose (consumes Channels, preserves ══, type ℝ→ℕ)

After σ (5 levels, spatial still 1024×1024):
```
Plate: 384-well
──────────────
Wells: w ⊆ {A1..P24}
──────────────
Points(w): 1..Nw
──────────────
ξy,x: 1024
══════════════
ObjectID ∈ ℕ
──────────────
```

; φ: Clean ; μ: ObjectMeas (consumes ══, creates Objects:N)

After μ (4 levels, spatial gone):
```
Plate: 384-well
──────────────
Wells: w ⊆ {A1..P24}
──────────────
Points(w): 1..Nw
──────────────
Objects: N
──────────────
{Area, Circ} ∈ ℝ
──────────────
```

; τ: Accum (collapses Objects)

After τ (3 levels):
```
Plate: 384-well
──────────────
Wells: w ⊆ {A1..P24}
──────────────
Points(w): 1..Nw
──────────────
{CellCount, MeanArea} ∈ ℝ
──────────────
```

; τ: Join wells (collapses Points)

After τ (2 levels):
```
Plate: 384-well
──────────────
Wells: w ⊆ {A1..P24}
──────────────
{CellCount, MeanArea} ∈ ℝ
──────────────
```

; τ: Z′-factor ; τ: DoseResponse → EC₅₀ (collapses Wells)

Output (1 level):
```
Plate: 384-well
──────────────
{Z′: ℝ, EC₅₀: ℝ}
──────────────
```

; ε: Save

Six levels in → one level out. Each morphism peels one level.


### 8. Neuron branching (Sholl)

Input:
```
GFP
──────────────
ξy,x: 2048
══════════════
ℝ: [0..2¹⁶]
──────────────
```

; φ: Denoise ; σ: Threshold ; ρ: Skeletonize

After ρ (image → graph, creates inner &):
```
Junctions: 1..J
──────────────
Columns: y, x
──────────────
ℝ²
──────────────
&
Branches: 1..B
──────────────
Columns: j₁, j₂
──────────────
{1..J}²
──────────────
&
Paths: 1..P
──────────────
Columns: y, x
──────────────
ℝ²
──────────────
```

; μ: Sholl & μ: BranchMeas

Output (two & tables):
```
Radii: 1..R
──────────────
Columns: Intersections
──────────────
ℕ
──────────────
&
Records: 1..B
──────────────
Columns: Length, Order
──────────────
ℝ
──────────────
```

ρ changes representation (image → graph).
Branches reference Junctions via {1..J}² (foreign key).


### 9. Tissue analysis (cell adjacency)

Input:
```
DAPI
──────────────
ξy,x: 4096
══════════════
ℝ: [0..2¹⁶]
──────────────
```

; σ: Cellpose ; μ: ObjectMeas + Voronoi

After μ (graph: cells & adjacency):
```
Cells: 1..N
──────────────
Columns: Area, Type
──────────────
ℝ
──────────────
&
Adjacency: 1..E
──────────────
Columns: c₁, c₂
──────────────
{1..N}²
──────────────
```

; μ: GraphFeatures

Output:
```
Records: 1..N
──────────────
Columns: Degree, Clustering
──────────────
ℝ
──────────────
```


### 10. Oriented surface (curvature) — GA

Input:
```
MembraneGFP
──────────────
ξz,y,x: 128, 512, 512
≡≡≡≡≡≡≡≡≡≡≡≡≡≡
ℝ: [0..2¹⁶]
──────────────
```

; σ: Cellpose3D ; ρ: Marching cubes

After ρ (creates inner &, GA types):
```
Vertices: 1..V
──────────────
Columns: pos
──────────────
v ∈ ⊛¹G³
──────────────
&
Faces: 1..F
──────────────
Columns: v₁, v₂, v₃
──────────────
B ∈ ⊛²G³ (normal)
──────────────
```

; μ: Curvature

Output:
```
Records: 1..N
──────────────
Columns: MeanCurv, GaussCurv
──────────────
ℝ
──────────────
```


### 11. Light sheet fusion — GA

Input (two views, ×):
```
View₁
──────────────
ξz,y,x: 512
≡≡≡≡≡≡≡≡≡≡≡≡
v ∈ ⊛¹G³
──────────────
×
View₂
──────────────
ξz,y,x: 512
≡≡≡≡≡≡≡≡≡≡≡≡
v ∈ ⊛¹G³
──────────────
```

; φ: Rotor R₁₂ on View₂ (GA rotation, below ≡≡) ; μ: Fuse

Output:
```
Fused
──────────────
ξz,y,x: 512
≡≡≡≡≡≡≡≡≡≡≡≡
v ∈ ⊛¹G³
──────────────
```


### 12. Polarisation — complex Clifford

Input:
```
Channels: H, V
──────────────
ξy,x: 1024
══════════════
J ∈ G²(ℂ)
──────────────
```

; φ: Waveplate(δ) (below ══, per pixel) ; μ: Stokes

Output:
```
Records: 1
──────────────
Columns: S₀, S₁, S₂, S₃
──────────────
ℝ
──────────────
```


### 13. Tiling / stitching

Input (image & stage coords, × overlap parameter):
```
Points: n
──────────────
ξy,x: 1024
══════════════
ℝ: [0..2¹⁶]
──────────────
&
Points: n
──────────────
Columns: stageX, stageY
──────────────
(x,y) ∈ ℝ²
──────────────
×
Overlap: 10%
──────────────
```

; ρ: Stitch

Output:
```
ξY,X: n × 1024
══════════════
ℝ: [0..2¹⁶]
──────────────
```

& = stage coordinates (share Points index).
× = overlap (same for all tiles).
ρ: n tiles → one large image.


### 14. FRAP (bleach recovery)

Input (image with two & metadata):
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
ROI: 1
──────────────
Columns: y₀, x₀, r
──────────────
ℝ³
──────────────
&
BleachEvent: 1
──────────────
Columns: t_bleach
──────────────
ℕ
──────────────
```

; μ: MeanIntensity per frame (within ROI, reads & for ROI)

After μ:
```
Records: 1..T
──────────────
Columns: t, MeanInt
──────────────
ℝ
──────────────
```

; τ: Normalise (pre-bleach = 1.0) ; τ: Fit (I(t) = I∞(1 - e^(-t/τ)))

Output:
```
Records: 1
──────────────
Columns: τ_half, MobileFraction, I∞
──────────────
ℝ
──────────────
```


### 15. FLIM (fluorescence lifetime)

Input:
```
DAPI
──────────────
ξy,x: 256
══════════════
τ_bins: 256
──────────────
Photons ∈ ℕ
──────────────
```

Per-pixel phasor (no binning):
; φ: Phasor FFT (below ══, spatial passes over)

Output (spatial preserved):
```
DAPI
──────────────
ξy,x: 256
══════════════
Columns: G, S
──────────────
ℝ²
──────────────
```

With spatial binning:
; μ: SpatialBin + PhasorFit (spans ══, uses neighbours)

Output (spatial reduced):
```
DAPI
──────────────
ξy,x: 64
══════════════
Columns: G, S
──────────────
ℝ²
──────────────
```

Rule 3 decides: needs neighbours → spans ══.


### 16. Wound healing / scratch assay

Input:
```
Phase contrast
──────────────
t: 1..T
──────────────
ξy,x: 1024
══════════════
ℝ: [0..2¹⁶]
──────────────
```

; σ: Threshold per frame

After σ (spatial preserved, type changed):
```
Phase contrast
──────────────
t: 1..T
──────────────
ξy,x: 1024
══════════════
{cell, gap} ∈ {0,1}
──────────────
```

; μ: GapArea per frame

After μ:
```
Records: 1..T
──────────────
Columns: t, GapArea, GapWidth
──────────────
ℝ
──────────────
```

; τ: Fit (closure rate)

Output:
```
Records: 1
──────────────
Columns: ClosureRate, T_half
──────────────
ℝ
──────────────
```


### 17. PALM/STORM (single molecule localisation)

Input:
```
Frames: 1..10000
──────────────
ξy,x: 256
══════════════
ℝ: [0..2¹⁶]
──────────────
```

; σ: SpotDetect per frame ; μ: GaussFit per spot

After μ (massive collapse: 10000 × ══ → table):
```
Localisations: 1..L
──────────────
Columns: x, y, frame, intensity, precision
──────────────
ℝ
──────────────
```

Then either:

**Path A** — render super-resolution image:
; ρ: Render (table → image, reverse of μ)

```
ξy,x: 2560 (10× oversampled)
══════════════
ℝ: [0..1] (density)
──────────────
```

**Path B** — cluster analysis:

After DBSCAN (creates graph via &):
```
Localisations: 1..L
──────────────
Columns: x, y, prec
──────────────
ℝ
──────────────
&
Clusters: 1..C
──────────────
Columns: l₁, l₂
──────────────
{1..L}²
──────────────
```

; μ: ClusterFeatures

```
Records: 1..C
──────────────
Columns: Npoints, Radius, Density
──────────────
ℝ
──────────────
```


### 18. Spectral unmixing

Input:
```
Channels: λ₁..λ₃₂
──────────────
ξy,x: 512
══════════════
ℝ: [0..2¹⁶]
──────────────
×
Reference spectra
──────────────
Columns: DAPI, GFP, mCherry, AutoFluor
──────────────
ℝ (per λ)
──────────────
```

; φ: LinearUnmix (below ══, per pixel)

Output (top line changes: wavelengths → components):
```
Components: DAPI, GFP, mCherry, AutoFluor
──────────────
ξy,x: 512
══════════════
ℝ: [0..1] (abundance)
──────────────
```

× = reference spectra (same for all pixels).
φ below ══ = spatial passes over.


### 19. Organoid morphology (nested objects)

Input:
```
Channels: DAPI, Ecad, Laminin
──────────────
ξz,y,x: 200, 1024, 1024
≡≡≡≡≡≡≡≡≡≡≡≡≡≡
ℝ: [0..2¹⁶]
──────────────
```

; σ: Cellpose3D (on DAPI) ; σ: OrganoidDetect (on Ecad)

After σ (nested objects, bipartite &):
```
Organoids: 1..K
──────────────
Columns: Volume, Sphericity
──────────────
ℝ
──────────────
&
Cells: 1..N
──────────────
Columns: OrganoidID, CellID
──────────────
ℕ × ℕ
──────────────
```

OrganoidID ∈ {1..K} = foreign key linking cells to organoids.

; μ: OrganoidFeatures

Output:
```
Records: 1..K
──────────────
Columns: CellCount, Volume, Sphericity,
         LumenPresent, Polarity
──────────────
ℝ | {0,1}
──────────────
```


### 20. Batch processing (map over files)

Input (Files:K once at top, metadata via &):
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
  {condition: Text, replicate: Text}
  ──────────────
```

; (σ ; μ) per file (metadata passes over)

After σ ; μ per file:
```
Files: 1..F
──────────────
  {CellCount, MeanArea} ∈ ℝ
  ──────────────
  &
  {condition: Text, replicate: Text}
  ──────────────
```

; τ: GroupBy condition ; τ: Statistics

Output:
```
Conditions: 1..C
──────────────
Columns: Condition, Mean, SD, N
──────────────
ℝ | Text
──────────────
```

Files:K once at top. & branches indented inside it.
τ reads condition through & → groups F files into C conditions.


### 21. Multi-scale pyramid (OME-Zarr)

Input (image & transforms share Levels index):
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
```

; σ at level 2 (coarse, 1024×1024)

After σ:
```
Level 2
──────────────
ξy,x: 1024
══════════════
ObjectID ∈ ℕ
──────────────
```

; ρ: Upsample ROIs to level 0

After ρ:
```
Level 0 (ROIs only)
──────────────
ξy,x: 4096
══════════════
ℝ: [0..2¹⁶]
──────────────
```

; μ: Precise measurement at full resolution

Output:
```
Records: 1..N
──────────────
Columns: Area, Int (at full res)
──────────────
ℝ (calibrated via & transforms)
──────────────
```

Coarse detection, precise measurement.
ρ maps coordinates between levels via & transforms.

---

## Mathematical Foundation

Stave Notation = string diagram calculus of the
cartesian double category Dbl(Img).

Horizontal direction = pipeline flow (;)
Vertical direction = container depth (fibration)

The cartesian structure gives:
- ⊂ = diagonal (copy)
- & = fibred product (product in the fibre)
- × = plain product (product in the total category)
- □ = terminal (discard)
- π₁, π₂ = projections (implicit, derived from layout)

× restricted to the same fibre = &. Both are products
at different levels of the fibration. Same axiom,
different fibres.

Boxes are 2-cells. Box height = vertical span.
Rule 3 reads off which edges of the square are active.

All seven power properties (types, compositionality,
Rule 3, compression, self-hosting, container nesting,
whiteboard compatibility) are theorems about this one
structure, not independent design choices.

---

## Agent Integration

Morphism box = agent task. Type signature = contract.

```
Agent reads container structure → auto-selects tool
Agent reads type row → auto-calculates parameters
Agent reads & vs × → validates connections
```

Multi-agent composition:
- ; = handoff (sequential)
- & = parallel with shared context (co-indexed)
- × = independent parallel

Meta-agent (Level 3): biological question → pipeline
design → execution → feedback. Closes the acquisition
↔ analysis loop with intelligent microscopy.

---

## Delivery

Draw on paper → photograph → upload to LLM with spec
as context. The LLM IS the parser, type checker, and
code generator. No formal grammar, visual editor, or
standalone software needed. Paper is the editor, phone
camera is the scanner, the LLM is the compiler.

---

## Platform Translation

```
Box   napari/scikit-image        ImageJ                GA3
ι     tifffile.imread()          open()                Src.Channels
φ     skimage.filters.*          run("Median...")       Pre.Smooth
σ     threshold_otsu()/cellpose  setAutoThreshold()     Seg.Threshold
μ     regionprops_table()        Analyze Particles      Meas2.ObjectFeatures
τ     pd.merge(), groupby()      (manual)               Data.Reduce
ε     df.to_csv()                saveAs("Results")      Sink.SaveTables
ρ     marching_cubes()           —                      —
```

Design on paper. Execute anywhere.
