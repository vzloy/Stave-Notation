# Stave Notation for Image Analysis

**Version: 2026-11-10_1002**

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

### 1. Cell counting

```
DAPI ≡≡ ℝ ── ; σ: Otsu ; μ: Count → Records: Count ∈ ℕ
```

### 2. Spot counting per cell (FISH)

```
DAPI ≡≡ ℝ ;  σ: Threshold → NucleiID ∈ ℕ
   ×
Cy5 ≡≡ ℝ  ;  σ: SpotDetect → SpotID ∈ ℕ

Both ; μ: Children → Records: ParentID, SpotCount ∈ ℕ
```

### 3. Colocalization

```
Ch1: GFP ══ ℝ ┐
               ├── μ: Pearson/Manders → {P, M1, M2} ∈ ℝ
Ch2: mCh ══ ℝ ┘
```

### 4. Ratio imaging (FRET / Calcium)

```
F340 ══ ℝ ┐
           ├── φ: ÷ pixel → Ratio ══ ℝ:[0..1]
F380 ══ ℝ ┘
; μ: FieldMeas per frame → Records: MeanRatio ∈ ℝ
```

### 5. Tracking

```
DAPI ; t:1..100 ; ══ ℝ
; σ per frame ; μ: TimeAndCenter
→ Detections & Links ({1..D}² × {move,split,merge})
; μ: MotionFeatures → Records: Speed, MSD ∈ ℝ
```

### 6. 3D volume segmentation

```
Channels: DAPI, Ki67 ; ≡≡ ℝ
; σ: Cellpose3D ; μ: Meas → Records: Vol, SurfArea, Ki67int ∈ ℝ
```

### 7. HCS plate (384-well)

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

σ preserves spatial (still 1024×1024, type ℝ→ℕ).
Objects:N created by μ. Each τ peels one container level.
Six levels in → one level out.

### 8. Neuron branching (Sholl)

```
GFP ══ ℝ ; φ: Denoise ; σ: Threshold ; ρ: Skeletonize
→ Junctions & Branches ({1..J}²) & Paths
; μ: Sholl & μ: BranchMeas
→ Radii: Intersections ∈ ℕ & Records: Length, Order ∈ ℝ
```

### 9. Tissue analysis (cell adjacency)

```
DAPI ══ ℝ ; σ: Cellpose ; μ: ObjectMeas + Voronoi
→ Cells & Adjacency ({1..N}²)
; μ: GraphFeatures → Records: Degree, Clustering ∈ ℝ
```

### 10. Oriented surface (curvature) — GA

```
MembraneGFP ≡≡ ℝ ; σ: Cellpose3D ; ρ: Marching cubes
→ Vertices (v ∈ ⊛¹G³) & Faces (B ∈ ⊛²G³)
; μ: Curvature → Records: MeanCurv, GaussCurv ∈ ℝ
```

### 11. Light sheet fusion — GA

```
View₁ ≡≡ v∈⊛¹G³ × View₂ ≡≡ v∈⊛¹G³
; φ: Rotor R₁₂ on View₂ ; ⊃ fuse → Fused ≡≡ v∈⊛¹G³
```

### 12. Polarisation — complex Clifford

```
Channels: H,V ; ══ J∈G²(ℂ)
; φ: Waveplate(δ) ; μ: Stokes → {S₀,S₁,S₂,S₃} ∈ ℝ
```

### 13. Tiling / stitching

```
Points:n ; ══ ℝ
  & StageCoords: (x,y) ∈ ℝ²
  × Overlap: 10%
; ρ: Stitch → ξY,X: n×1024 ══ ℝ
```

### 14. FRAP (bleach recovery)

```
GFP ; t:1..T ; ══ ℝ
  & ROI: (y₀,x₀,r) ∈ ℝ³
  & BleachEvent: t_bleach ∈ ℕ
; μ: MeanIntensity per frame
; τ: Normalise ; τ: Fit → {τ_half, MobileFraction, I∞} ∈ ℝ
```

### 15. FLIM (fluorescence lifetime)

Per-pixel (no binning): box below ══, spatial passes over.
```
══ ; τ_bins:256 Photons∈ℕ → φ: PhasorFFT → Columns:G,S ∈ ℝ²
```

With spatial binning: box spans ══, uses neighbours.
```
══ ; τ_bins:256 Photons∈ℕ → μ: SpatialBin+PhasorFit → ══(64) G,S ∈ ℝ²
```

Rule 3 decides placement.

### 16. Wound healing / scratch assay

```
Phase ; t:1..T ; ══ ℝ
; σ: Threshold per frame → {cell,gap} ∈ {0,1}
; μ: GapArea per frame → Records: GapArea, GapWidth ∈ ℝ
; τ: Fit → {ClosureRate, T_half} ∈ ℝ
```

### 17. PALM/STORM (single molecule localisation)

```
Frames:1..10000 ; ══ ℝ
; σ: SpotDetect ; μ: GaussFit
→ Localisations: x,y,frame,intensity,precision ∈ ℝ

Then either:
; ρ: Render → ══(2560) ℝ (ρ: table→image, reverse of μ)
or:
; & DBSCAN Clusters ({1..L}²) ; μ: ClusterFeatures
→ Records: Npoints, Radius, Density ∈ ℝ
```

### 18. Spectral unmixing

```
Channels: λ₁..λ₃₂ ; ══ ℝ
  × Reference spectra
; φ: LinearUnmix (below ══, per pixel)
→ Components: DAPI,GFP,mCherry,AutoFluor ; ══ ℝ:[0..1]
```

### 19. Organoid morphology (nested objects)

```
Channels: DAPI,Ecad,Laminin ; ≡≡ ℝ
; σ: Cellpose3D → CellID ∈ ℕ
; σ: OrganoidDetect → OrganoidID ∈ ℕ
→ Organoids & Cells (OrganoidID∈{1..K} = foreign key)
; μ: OrganoidFeatures → Records: CellCount, Vol, Sphericity, Polarity
```

### 20. Batch processing (map over files)

```
Files:1..F
  Channels: DAPI, GFP ; ══ ℝ
  &
  {condition:Text, replicate:Text}
; (σ ; μ) per file
; τ: GroupBy condition ; τ: Statistics
→ Records: Condition, Mean, SD, N
```

Files:K appears ONCE at the top. Both image and
metadata branches sit inside it. Condition metadata
via &. Statistics across replicates is τ.

### 21. Multi-scale pyramid (OME-Zarr)

```
Dataset ; Levels:0..4
  Channels: DAPI,GFP ; ══(4096/2ˡ) ℝ
  &
  Transforms: scale_y, scale_x ∈ ℝ
; σ at level 2 (coarse) ; ρ: Upsample to level 0
; μ: Precise measurement at full res
→ Records: Area, Int ∈ ℝ (calibrated via & transforms)
```

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
