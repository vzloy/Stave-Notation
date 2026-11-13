# Stave Notation for Image Analysis

**Version: 2026-11-13_0935**

A diagrammatic calculus for image analysis pipelines,
grounded in the cartesian double category Dbl(Img).

Four structural primitives. Seven morphism types. Three rules.

Inspired by Vincent Abbott's Neural Circuit Diagrams
(TMLR 2024) — visual grammar where lines = functors.
Categorical semantics (type row, morphism types, container
nesting, &, ×, Rule 3, double-category structure) is original.

Key correspondences with Abbott's Neural Circuit Diagrams:

```
Abbott NCD                     Stave Notation
─────────────────────────────────────────────────
Wires = axes                   Lines = container levels
Dashed lines = tuples          Dashed = decomposition / ×
Operations change shape        Boxes change container structure
Broadcasting over axes         Functorial lift (Rule 3)
Outer product extends          ⊂ copy/extend
```

Reference: V. Abbott, "Neural Circuit Diagrams: Robust
Diagrams for the Communication, Implementation, and
Analysis of Deep Learning Architectures," Transactions
on Machine Learning Research, 2024.
https://openreview.net/forum?id=RyZB4qXEgt
https://github.com/vtabbott/Neural-Circuit-Diagrams

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

### Lines — container depth

```
──    wire      1D container (list, time series, table rows)
══    matrix    2D container (spatial grid — has neighbourhood)
≡≡    volume    3D container
```

Strokes = dimensions. ══ is literally two ── stacked
(Y and X axes). Count of lines above the type row =
nesting depth of containers.

### Indexing convention

All container lines carry explicit ranges, not bare
cardinalities. The range IS the domain — no ambiguity
between cardinality and index set.

```
Spatial axes:    ξy,x: 0..1023       0-indexed (pixel offset)
Counting:        Objects: 1..N       1-indexed (enumeration)
Time:            t: 1..100           1-indexed
Fixed set:       Points: 1..25       explicit, not bare "25"
Variable:        Points(w): 1..Nw    dependent (range varies)
Named set:       Channels: {DAPI, GFP}    enumeration
Subset:          Wells: w ⊆ {A1..P24}     subset of named set
```

0-indexed for spatial axes (pixel offsets from origin).
1-indexed for everything else (counting objects, frames,
wells, records). This matches implementation: Python
arrays are 0-indexed, object IDs start at 1.

### Symbol convention

Words for logic, symbols for math. Avoids collision
between logical AND (∧) and GA wedge product (∧).

```
|            "such that" — guard/filter on a container
               Objects: n | Area(n) > 50
             also type alternative — "or"
               ℝ | ℕ | text  (column holds any of these)
AND OR NOT   logical connectives — always words, never symbols
               Area > 50 AND Circ > 0.8
∧  ·         GA wedge and inner product — always symbols
               B = e₁ ∧ e₂   (bivector, NOT logical AND)
> < ≥ ≤ = ≠  comparison
∈ ⊆          membership, subset
..           range — inclusive both ends
               0..1023  t: 1..100
```

Reading guide for `|`:
- After a container name → "such that" (filter)
- Between type names → "or" (alternative)
Context always makes it clear: containers are on the
left, types are on the bottom line.

Examples:
```
Objects: n | Area(n) > 50 AND Circ(n) > 0.8
t | t ≥ t_bleach AND t ≤ t_bleach + 100
Wells: w ∈ {A1..A12} OR w ∈ {B1..B12}
ξy,x | NOT (y² + x² > r²)
```

GA uses symbols:
```
B = (v₂−v₁) ∧ (v₃−v₁)        wedge product (bivector)
a · b                           inner product (scalar)
```

No ambiguity. Easy to hand-write. Matches SQL/Haskell
guard convention for predicates while keeping standard
mathematical notation for algebra.

The cap ⊂ is one shape. Read left to right:

```
⊂  opening left      copy/extend    grade ↑ (GA outer product)
```

---

## Three Rules

**Rule 1 — Lines are data, boxes are functions.**

Every visual element is one or the other. No decorative
boxes or rectangles. Every element carries semantic meaning.

Name above line. Bottom line closes:

```
Channels: DAPI, GFP
───────────────────────
ξy,x: 0..1023
═══════════════════════
ℝ: [0..2¹⁶]
───────────────────────
```

**Rule 2 — Pipeline flows left to right.**

Sequential composition (;) is horizontal. Like a musical
score — left to right, no backtracking.

Strokes = dimensions:

```
──  1D     ══  2D     ≡≡  3D
```

Dashed line between axes means decomposition — an
explicit claim that the matrix separates into independent
axes. This LOSES spatial structure. Not shorthand.

**Rule 3 — The box sits at the level it operates on.**

A theorem: containers are functors. Pixel-level functions
lift automatically; structure-consuming functions cannot be
pushed down. The box position is DETERMINED, not chosen. □

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

### Two kinds of box input

Every input to a box is either & or ×.

```
&    linked       shares index     calibration, coordinates,
                                   conditions that vary per element

×    independent  no shared index  parameters, constants,
                                   models, thresholds
```

On paper × is drawn as a dashed line (- - -).
× and dashed line are interchangeable.

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
Columns: minSNR   &          ξy,x: 0..1023
──────────────               ══════════════
ℝ: 3.0                      ℝ: [0..2¹⁶]
──────────────               ──────────────

; τ: Filter (keep if SNR > minSNR)

Points: m ≤ n
──────────────
ξy,x: 0..1023
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
      ──────────────         ξy,x: 0..1023
      Columns: weights       ══════════════
      ──────────────         ℝ: [0..2¹⁶]
      ℝ                     ──────────────
      ──────────────

; σ: Cellpose(model=Model)

DAPI
──────────────
ξy,x: 0..1023
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
ℝ                            ξy,x: 0..511
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

## Thick lines — functor composition

A thick line represents composed container levels.
Thickness IS composition — no new symbol.

Three equivalent representations of the same data:

```
Expanded (stave):         Composed (spatial):     Fully composed:

z: 0..10                  z, y, x                 z, y, x, ℝ
──────────────            ══════════════════       ══════════════════
y,x: ξ 0..1023       =   ℝ                   =
══════════════            ──────────────
ℝ: [0..2¹⁶]
──────────────
```

The thick line (══════) bundles multiple levels into
one wire. The label lists the composed axes.

### Boxes grow up, never down

A box can always be enlarged to span more levels.
On the upper levels it acts as the identity functor —
it passes the container through unchanged.

```
Thick line:    ══════════▶ □ φ: Gaussian ══════════▶

Expanded:      z: 0..10     id           z: 0..10
               ──────────                ──────────
               y,x: 0..1023  id          y,x: 0..1023
               ══════════                ══════════
               ℝ             φ: Gauss    ℝ
               ──────────                ──────────
```

The Gaussian blur operates on ℝ (bottom level).
On z and y,x it is the identity. The box spans the
thick line, but only one component is active.

This is categorically exact:
- F = F₁ ∘ F₂ ∘ F₃ (thick line = composed functors)
- Morphism α at F₃ lifts to id(F₁) ∘ id(F₂) ∘ α
- Growing up = adding identity components (always valid)
- Pushing down = removing needed structure (impossible)

Rule 3 for thick lines: **every box spans the thick
line. Expand to see which components are identity
and which are active.**

### Two drawing modes on the same paper

```
Overview:    ══════▶ □ σ ══════▶ □ μ ──────▶    (what happens)
Detail:      expand any □ to see the stave       (where the box sits)
```

Thick lines for the pipeline. Expand to the stave
when you need to see which level a box operates on.
Same data, two zoom levels, one notation.

### Algebraic type expressions

The label above a thick line is a functor composition
using standard notation: ∘ for composition, & for
fibred product, × for plain product, → for morphisms
(natural transformations).

Reading a stave top-to-bottom = left-to-right ∘:

```
Stave:                        Algebra:

y,x: 0..1023
══════════════                y ∘ x ∘ ℝ
ℝ: [0..2¹⁶]
──────────────
```

The & product composes both branches:

```
y,x: 0..1023
══════════════
ℝ: [0..2¹⁶]                 (y ∘ x ∘ ℝ) & (Meta ∘ ℝ)
──────────────
&
MetaData
──────────────
ℝ (μm)
──────────────
```

Morphism signatures as natural transformations:

```
σ : y ∘ x ∘ ℝ  →  y ∘ x ∘ ℕ                     segment
μ : (y ∘ x ∘ ℕ) & (Meta ∘ ℝ)  →  Obj ∘ ℝ        measure
φ : y ∘ x ∘ ℝ  →  y ∘ x ∘ ℝ                     filter
ρ : z ∘ y ∘ x ∘ ℕ  →  (V ∘ ⋀¹G³) & (F ∘ ⋀²G³)  mesh
```

Common data types as expressions:

```
y ∘ x ∘ ℝ                           2D image
z ∘ y ∘ x ∘ ℝ                       3D volume
(Nodes ∘ ℝ) & (Edges ∘ {1..N}²)     graph
(V ∘ ⋀¹G³) & (F ∘ ⋀²G³)            oriented mesh
(y ∘ x ∘ ℝ) × (Model ∘ ℝ)          image with × parameter
Obj ∘ {Area, Circ} ∘ ℝ              measurement table
```

Three representations, one meaning:

```
1. Stave (visual)     draw on paper, see every level
2. Thick line         one wire, algebra above it
3. Type expression    ∘ and & — for signatures and type checking
```

---

## Calibration as &

```
DAPI                         Calibration: 1
──────────────               ──────────────
ξy,x: 0..1023       &          Columns: pixelSize, unit
══════════════               ──────────────
ℝ: [0..2¹⁶]                 0.065 μm/px | μm
──────────────               ──────────────
```

μ reads calibration through &. Output is in μm² not px².

Equivalent flat form:

```
══ ℝ ──                          ← image (has spatial structure)
&
{PixelSize:ℝ} ──                 ← metadata (flat, no container)
```

The & guarantees they belong to the same file.

---

## Image and table: same structure, 3 lines

```
Image:                       Table:

Channels: DAPI               Records: 1..N
──────────────               ──────────────
ξy,x: 0..1023                  Columns: ID, Area, Int
══════════════               ──────────────
ℝ: [0..2¹⁶]                 ℝ | ℕ | text
──────────────               ──────────────
```

μ changes ══ to ──. Same depth, different middle:

```
Objects: 1..N                     Records: 1..N
──────────────                    ──────────────
ξy,x: 0..1023       μ: Measure      Columns: Area, Int
══════════════    ──────────▶     ──────────────
ObjectID ∈ ℕ                      ℝ
──────────────                    ──────────────
```

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

### Geometric Algebra

Use simple types by default. Use GA when geometry matters.

```
Simple         GA equivalent          When to use GA
───────────────────────────────────────────────────────────
ℝ              s ∈ ⋀⁰Gⁿ              never (scalar is scalar)
ℕ              —                      never
{0,1}          —                      never
text           —                      no GA equivalent
(x,y) ∈ ℝ²    v ∈ ⋀¹G²              when direction matters
(x,y,z) ∈ ℝ³  v ∈ ⋀¹G³              registration, flow fields
Δ₂ triangle    B ∈ ⋀²G³              oriented surfaces, curvature
Δ₃ tet         T ∈ ⋀³G³              oriented volumes
—              S ∈ ⋀¹G⁴˙¹             conformal spheres (CGA)
—              J ∈ G²(ℂ)             polarisation optics
—              M ∈ Gⁿ                full multivector (stress, DTI)
```

#### GA grades

```
⋀⁰Gⁿ     scalar         ⋀¹Gⁿ     vector
⋀²Gⁿ     bivector       ⋀³Gⁿ     trivector
Gⁿ        multivector
```

#### Common algebras

```
G²    2D plane       G³    3D space       G⁴˙¹   conformal 3D
G²(ℂ) complex        G³˙¹   spacetime
```

#### GA products map to stave operations

```
Inner a · b    grade ↓    μ    contraction, projection
Outer a ∧ b    grade ↑    ⊂    span, surface from edges
Geometric ab   mixed      □    rotation, reflection
```

#### Rotors

```
R = e^(Bθ/2)    rotation without matrices, any dimension
                 φ-morphism: below ≡≡, structure passes over
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

### Compact reference

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

## The & product

Everything vertical. & connects stacks that share an
index set.

### Image & Table

```
DAPI                         Records: 1..M
──────────────               ──────────────
ξy,x: 0..1023       &          Columns: ID, Area
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
v ∈ ⋀¹G³                    B ∈ ⋀²G³ (normal)
──────────────               ──────────────
```

---

## Object representations

```
Raster              ξy,x ══ ObjectID ∈ ℕ
Spots (simple)      Objects ── Columns ── (x,y,z,r) ∈ ℝ⁴
Spots (CGA)         Objects ── Columns ── S ∈ ⋀¹G⁴˙¹
Contour             Objects ── Vertices:k ── (y,x) ∈ ℝ²
Surface mesh        Objects ── Faces:F ── B ∈ ⋀²G³
Volume mesh         Objects ── Cells:C ── T ∈ ⋀³G³
```

## Dependent cardinality

```
Points: 1..25       fixed. ND array.
Points(w): 1..Nw   variable. Container.
```

---

## Container predicates

Every container line already carries a domain restriction —
cardinality IS a predicate. The `| Predicate` syntax makes
this explicit: restrict the domain to a subobject.

```
ContainerName: index_variable | Predicate
```

When `| Predicate` is absent, the full domain is implied
(as now). When present, it restricts the domain — a
subobject in the categorical sense, a WHERE clause in
the database sense, a guard in the Haskell sense.

No new primitives. No new rules. Same stance as Haskell
type signatures: declare what, not how. The LLM reads
the predicate and decides the implementation.

### What is already a predicate

```
Wells: w ⊆ {A1..P24}           finite subset
Points(w): 1..Nw               dependent cardinality
ξy,x: 0..1023                  explicit range (domain is the range)
Channels: {DAPI, GFP}          enumeration
```

### Spatial predicate — FRAP with explicit bleach geometry

```
t: 1..T
──────────────
ξy,x | (y-y₀)² + (x-x₀)² < r²
══════════════
ℝ
──────────────
```

ROI is now part of the type, not metadata via &.
A morphism on this container knows its domain is circular.

### Spatial predicate — tile overlap region

```
Tiles: i,j ∈ {0..Ni}×{0..Nj}
──────────────
ξy,x | y ∈ [0..H), x ∈ [0..W)
══════════════
ℝ
──────────────

; φ: BlendOverlap — operates on ξy,x | x ∈ [W-δ..W)
```

### Temporal predicate — time-gated measurement

Pre-bleach:
```
t | t < t_bleach
──────────────
ξy,x
══════════════
ℝ
──────────────
```

Post-bleach:
```
t | t ≥ t_bleach
──────────────
ξy,x
══════════════
ℝ
──────────────
```

The split is a predicate on the time container, not a τ.

### Object predicate — filtering post-measurement

```
Objects: n | Area(n) > 50 μm²
──────────────
{Area, Circ, MeanInt} ∈ ℝ
──────────────
```

### What a predicate can reference

```
Own indices:        y,x | y² + x² < r²
& branch values:    n | Area(n) > threshold
Constants:          t | t < 10
Other containers:   Points(w): 1..Nw
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

Each protocol shows vertical stave snapshots at structural
transition points, connected by ; morphism chains.
Input stave → ; morphisms → Output stave.


### 1. Cell counting

Input:
```
Channels: DAPI
──────────────
ξy,x: 0..1023
══════════════
ℝ: [0..2¹⁶]
──────────────
```

; σ: Otsu

σ output (spatial preserved, type changed):
```
Channels: DAPI
──────────────
ξy,x: 0..1023
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
ξy,x: 0..1023       ×          ξy,x: 0..1023
══════════════               ══════════════
ℝ: [0..2¹⁶]                 ℝ: [0..2¹⁶]
──────────────               ──────────────
```

; σ: Threshold (on DAPI) × σ: SpotDetect (on Cy5)

After σ:
```
DAPI                         Cy5
──────────────               ──────────────
ξy,x: 0..1023       ×          ξy,x: 0..1023
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
ξy,x: 0..1023
══════════════
ℝ: [0..2¹⁶]
──────────────
&
Ch2: mCherry
──────────────
ξy,x: 0..1023
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
ξy,x: 0..1023
══════════════
ℝ: [0..2¹⁶]
──────────────
&
Ch: F380
──────────────
ξy,x: 0..1023
══════════════
ℝ: [0..2¹⁶]
──────────────
```

; φ: ÷ pixel (per-pixel ratio, below ══)

After φ:
```
Ratio
──────────────
ξy,x: 0..1023
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
ξy,x: 0..1023
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
ξz,y,x: 0..63, 0..511, 0..511
≡≡≡≡≡≡≡≡≡≡≡≡≡≡
ℝ: [0..2¹⁶]
──────────────
```

; σ: Cellpose3D

After σ (spatial preserved, ≡≡ unchanged):
```
DAPI
──────────────
ξz,y,x: 0..63, 0..511, 0..511
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
ξy,x: 0..1023
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
ξy,x: 0..1023
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
ξy,x: 0..2047
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
ξy,x: 0..4095
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
ξz,y,x: 0..127, 0..511, 0..511
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
v ∈ ⋀¹G³
──────────────
&
Faces: 1..F
──────────────
Columns: v₁, v₂, v₃
──────────────
B ∈ ⋀²G³ (normal)
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
ξz,y,x: 0..511
≡≡≡≡≡≡≡≡≡≡≡≡
v ∈ ⋀¹G³
──────────────
×
View₂
──────────────
ξz,y,x: 0..511
≡≡≡≡≡≡≡≡≡≡≡≡
v ∈ ⋀¹G³
──────────────
```

; φ: Rotor R₁₂ on View₂ (GA rotation, below ≡≡) ; μ: Fuse

Output:
```
Fused
──────────────
ξz,y,x: 0..511
≡≡≡≡≡≡≡≡≡≡≡≡
v ∈ ⋀¹G³
──────────────
```


### 12. Polarisation — complex Clifford

Input:
```
Channels: H, V
──────────────
ξy,x: 0..1023
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
ξy,x: 0..1023
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
ξY,X: 0..(n×1024-1)
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
ξy,x: 0..511
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

ROI is & (shares spatial index).
BleachEvent is & (shares time index).
T frames → 3 fitted parameters.


### 15. FLIM (fluorescence lifetime)

Input:
```
DAPI
──────────────
ξy,x: 0..255
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
ξy,x: 0..255
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
ξy,x: 0..63
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
ξy,x: 0..1023
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
ξy,x: 0..1023
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

σ per frame, μ per frame, τ fits across frames.


### 17. PALM/STORM (single molecule localisation)

Input:
```
Frames: 1..10000
──────────────
ξy,x: 0..255
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
ξy,x: 0..2550 (10× oversampled)
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
ξy,x: 0..511
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
ξy,x: 0..511
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
ξz,y,x: 0..199, 0..1023, 0..1023
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
  ξy,x: 0..1023
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
ξy,x: 0..1023
══════════════
ObjectID ∈ ℕ
──────────────
```

; ρ: Upsample ROIs to level 0

After ρ:
```
Level 0 (ROIs only)
──────────────
ξy,x: 0..4095
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
ξy,x: 0..1023    SegMeas        Columns: Area, Int
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
ξy,x: 0..1023   σ:Cellpose ξy,x: 0..1023   φ:Clean    ...  ; μ:Measure
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
ξy,x: 0..1023    SegMeas(Model,        Columns: Area, Int
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
ξy,x: 0..1023              Columns: pixelSize
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
(JOIN = μ, GROUP BY = μ, WHERE = τ, INDEX = &).

Design on paper. Execute anywhere.
