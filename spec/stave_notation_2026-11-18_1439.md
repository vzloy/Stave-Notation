# Stave Notation for Image Analysis

**Version: 2026-11-18_1439**

A diagrammatic calculus for image analysis pipelines,
grounded in the distributive cartesian double category Dbl(Img).

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
- Para construction for double categories (Femić, arXiv:2510.26465, 2025) — × as parametrised morphism
- Categorical Foundations of Gradient-Based Learning (Gavranović, 2024) — original Para for 1-categories
- Distributive categories (Cockett 1993; Carboni-Lack-Walters 1993) — products distribute over coproducts
- Generalising Monads to Arrows (Hughes, 2000) — Arrow/ArrowLoop typeclasses
- Yampa / FRP with ArrowLoop (Courtney & Elliott, 2001) — signal processing with feedback
- FVDblTT: A Type Theory for Virtual Double Categories (Nasu, 2024) — formal internal language
- VETT: A Virtual Equipment Type Theory (New & Licata, 2023) — virtual double category type theory

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
             type alternative — "or"
               ℝ | ℕ | text  (column holds any of these)
             morphism guard — piecewise function
               φ | v > 150 : brighten
                 | else    : darken
AND OR NOT   logical connectives — always words, never symbols
               Area > 50 AND Circ > 0.8
∧  ·         GA wedge and inner product — always symbols
               B = e₁ ∧ e₂   (bivector, NOT logical AND)
> < ≥ ≤ = ≠  comparison
∈ ⊆          membership, subset
..           range — inclusive both ends
               0..1023  t: 1..100
where        names parts of composites (Haskell convention)
               SegMeas = σ ; μ where σ = Cellpose(Model)
```

Reading guide for `|` — three levels, one symbol:
- After a container name → "such that" (filter/subobject)
- Between type names → "or" (alternative/coproduct)
- After a morphism name → piecewise (case analysis/copairing)
Context always makes it clear: containers on the left,
types on the bottom line, morphisms in the pipeline.

```
Container:  Objects: n | Area(n) > 50μm²       (which elements)
Type:       ℝ | ℕ | Text                        (which kind)
Morphism:   φ | v > 150 : brighten              (which branch)
              | v > 50  : identity
              | else    : darken
```

Piecewise morphism example — LUT (intensity → colour):

```
φ | v ∈ [0, 50)    : ℝ → RGB(blue)
  | v ∈ [50, 150)  : ℝ → RGB(green)
  | v ∈ [150, 256) : ℝ → RGB(red)
```

Categorically: a copairing [f₁, f₂, f₃] : X₁ + X₂ + X₃ → Y,
where predicates partition the input via coproduct injections.

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

### Haskell notation adopted

| Haskell convention | Stave usage |
|-------------------|-------------|
| `::` for type signatures | Morphism/pipeline typing |
| Lowercase = polymorphic, Uppercase = concrete | Rule 3 encoding |
| `forall` | Explicit polymorphism |
| `=>` after constraints | Rule 3 and structural constraints |
| `type` keyword | Named composites as type synonyms |
| `@` for type application | Instantiation of polymorphic morphisms |
| `where` clause | Composite internals |
| `\|` guards | Piecewise morphisms |
| `:` in types | Sort boundary (Str : Val) |

### Haskell notation NOT adopted

| Haskell feature | Reason |
|----------------|--------|
| `→` as type constructor | No closedness — `→` is morphism arrow only |
| `>>=` / `do`-notation | Stave is Arrow, not Monad |
| `data` / `newtype` | Fixed 7 generators, no user-defined types |
| `class` / `instance` | 7 generators with known types — full class mechanism is overkill |
| `$` / `.` for application/composition | Use `;` and `@` |
| Higher-rank types | No rank-2 morphisms in imaging |
| GADTs | Not needed for a fixed theory |
| Type families | LLM is the evaluator, not a type-level interpreter |

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

### × as Para (parametrised morphism)

× parameter inputs are Para arrows: (P, f) : P × X → Y
in Para_M(Dbl(Img)). No new symbol — × already IS Para.

Reparametrisation (collapsing a composite's internal ×
to exposed × with defaults) is a 2-cell in Para.

Side outputs (quality metrics, auto-chosen thresholds)
are coPara: X → P × Y, expressed via existing ⊂ branching.

Reference: Femić (arXiv 2510.26465, 2025) — lifts
Gavranović's Para construction from 1-categories to
double categories.

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
(natural transformations), and `:` for the sort boundary
between structure (Str) and value (Val).

Reading a stave top-to-bottom = left-to-right, with
`:` marking the boundary between container and value:

```
Stave:                        Algebra:

y,x: 0..1023
══════════════                Y ∘ X : ℝ
ℝ: [0..2¹⁶]
──────────────
```

The `:` replaces the final `∘` — it separates structure
(left) from value (right). Rule 3 is visible: check
whether the Str left of `:` matches across `→`.

The & product composes both branches:

```
y,x: 0..1023
══════════════
ℝ: [0..2¹⁶]                 (Y ∘ X : ℝ) & (Meta : ℝ)
──────────────
&
MetaData
──────────────
ℝ (μm)
──────────────
```

Morphism signatures as natural transformations:

```
σ :: Y ∘ X : ℝ  →  Y ∘ X : ℕ                     segment
μ :: (Y ∘ X : ℕ) & (Meta : ℝ)  →  Obj : ℝ        measure
φ :: Y ∘ X : ℝ  →  Y ∘ X : ℝ                     filter
ρ :: Z ∘ Y ∘ X : ℕ  →  (V : ⋀¹G³) & (F : ⋀²G³)  mesh
```

Common data types as expressions:

```
Y ∘ X : ℝ                           2D image
Z ∘ Y ∘ X : ℝ                       3D volume
(Nodes : ℝ) & (Edges : ℕ²)          graph
(V : ⋀¹G³) & (F : ⋀²G³)            oriented mesh
(Y ∘ X : ℝ) × (Model : ℝ)          image with × parameter
Obj ∘ Cols : ℝ                       measurement table
```

Four representations, one meaning:

```
1. Stave (visual)     draw on paper, see every level and box placement
2. Thick line         one wire per data flow, algebra label above
3. Type expression    ∘ and & with → (compact signatures)
4. Haskell form       guards, where, type constraints (for LLM code generation)
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

## Type System

Stave Notation adopts a **two-sorted Hindley-Milner type
system**. The two sorts correspond to the two dimensions
of the double category Dbl(Img).

### Two sorts

| Sort | Name | What it describes | On the stave |
|------|------|-------------------|--------------|
| `Str` | Structure | Container configuration — WHERE data lives | Container lines (══ ── ≡≡) |
| `Val` | Value | Value type — WHAT kind of data | Type row |

Every data wire carries a pair `(S : Str, A : Val)`.
Container lines are `Str`-sorted, the type row is
`Val`-sorted. The sort distinction IS the double
category's two dimensions.

### Type grammar

```
Str  ::= s              -- type variable (polymorphic container)
       | 1              -- trivial structure (no container)
       | S₁ ∘ S₂       -- functor composition (nesting)
       | S₁ & S₂       -- fibred product (shared parent)
       | S₁ × S₂       -- plain product (independent)

Val  ::= a              -- type variable (polymorphic value)
       | ℝ | ℕ | Bool   -- base types
       | A & B          -- product of values
       | A × B          -- independent values
       | A | B          -- coproduct / type alternative
       | {x : A ∣ P(x)} -- refinement type (via | predicate)
```

### Why two sorts, not one

Stave's type system is the internal language of a
**distributive cartesian double category**, not a
cartesian closed category (CCC). The critical difference:

- In a CCC (Haskell): `→` is an exponential — functions
  are data, stored in containers.
- In Dbl(Img) (Stave): `→` is a morphism arrow —
  morphisms operate ON containers, they do not live IN
  them. No `══ (ℝ → ℕ) ──`.

**Stave = Haskell − closedness + a second sort.**

The two sorts are needed because an ordinary (one-sorted)
category collapses the container/value distinction,
making it impossible to distinguish structure-preserving
operations (σ: same container, different type) from
structure-changing operations (μ: different container,
different type). The sort distinction is what makes
Rule 3 a typing rule rather than a drawing convention.

### The colon `:` — name : carrier

The symbol `:` means **"named this, populated by that"**
at every level of the notation. Left of `:` is the
**name** (the mathematical structure). Right of `:` is
the **carrier** (the specific population of values).

Three levels, one symbol:

| Level | Left of `:` | Right of `:` | Example |
|-------|------------|-------------|---------|
| **Container** | Container name | Index range | `y,x : 0..1024` |
| **Type** | Mathematical structure | Carrier set | `ℝ : [0..2¹⁶)` |
| **Proarrow** | Container structure (Str) | Value type (Val) | `Y ∘ X : ℝ` |

On the stave drawing, every line uses `:`:

```
Channels : DAPI          container : population
─────────────────
y,x : 0..1024            container : population
═════════════════
ℝ : [0..2¹⁶)             structure : carrier
─────────────────
```

The visual boundary between container lines (above) and
the type row (below) IS the `:` at the proarrow level.
The stave already draws it — the algebraic form makes
it explicit.

In the algebraic form, `:` replaces the previously
ambiguous final `∘`:

```
Before (ambiguous):          After (sort boundary explicit):

Y ∘ X ∘ ℝ                   Y ∘ X : ℝ
Obj ∘ ℝ                     Obj : ℝ
(Nodes ∘ ℝ) & (Edges ∘ ℕ²)  (Nodes : ℝ) & (Edges : ℕ²)
```

Rule 3 is now **visible in the algebraic form**: check
whether the structure left of `:` matches across `→`.
Same structure both sides = box below; different
structure = box spans the line.

### The double colon `::` — morphism typing

The symbol `::` means **"this morphism has this type"**.

| Symbol | Level | Question answered | Where it appears |
|--------|-------|-------------------|-----------------|
| `:` | Type-level | "What populates this name?" | Stave drawing + algebraic form |
| `::` | Term-level | "What type does this morphism have?" | Algebraic form only |

On the stave, the box itself IS the typing judgement —
input wires determine the domain, output wires determine
the codomain. `::` is **never drawn** on the stave; it
exists only in the algebraic representation.

Syntax:

```
morphism_name :: StructIn : ValIn → StructOut : ValOut
```

Generator types:

```haskell
φ       :: s : a       →  s : a           -- structure-preserving, type-preserving
σ       :: s : ℝ       →  s : ℕ           -- structure-preserving, type-changing
μ       :: Y ∘ X : ℕ   →  Obj : ℝ         -- structure-reducing
τ       :: Obj : ℝ     →  (Obj : ℝ) & (Edges : ℕ²)  -- structure-creating
ρ       :: Y ∘ X : Bool →  (V : ℝ²) & (E : ℕ²)      -- restructuring
ι       :: □            →  s : a           -- introduction
ε       :: s : a       →  □               -- discard
```

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

### Rule 3 as a typing rule

Haskell's case convention encodes Rule 3:

- **Lowercase** type variable (e.g., `s`, `a`) =
  **universally quantified** = polymorphic = box sits
  **below** the line (passes over)
- **Uppercase** concrete type (e.g., `Y ∘ X`, `Obj`) =
  **specific** = box **spans** the line (operates on it)

Generator types with explicit quantifiers:

```haskell
ι :: forall s a.           □  →  s : a
φ :: forall s a.           s : a  →  s : a
σ :: forall s.             s : A  →  s : B       -- A, B specific Val types
μ ::                   S₁ : A  →  S₂ : B       -- S₂ < S₁ (reduces)
τ ::                   S₁ : A  →  S₂ : B       -- S₂ > S₁ (creates)
ρ ::                   S₁ : A  →  S₂ : B       -- both may change
ε :: forall s a.           s : a  →  □
```

The `forall s` on φ and σ IS Rule 3 — these morphisms
work at any container depth, so the box sits below the
lines. The absence of `forall s` on μ, τ, ρ means they
require specific container structures, so the box must
span those lines.

Constraints express Rule 3 and structural requirements:

```haskell
φ :: (Spatial s) => s : a  →  s : a
μ :: (s ≥ Obj)   => s : ℕ  →  Obj : ℝ
```

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

## Pipeline iteration (⋮)

⋮ also marks pipeline iteration — the same "this pattern
continues" symbol reused for looping composites.

Visual convention: inside a composite box, the pipeline
name appears twice with ⋮ between them:

```
┌─ Acquire ; Analyse ; Decide | Cond ─┐
│               ⋮                      │
└─ Acquire ; Analyse ; Decide          ┘
```

**Disambiguation from data ⋮:** no ambiguity because
position distinguishes:
- ⋮ on a container line = recursive data depth (W-type, trees)
- ⋮ inside a composite box between pipeline labels = pipeline iteration

Algebraic form: ⋮ does NOT appear in the Haskell notation.
The recursion is expressed via guards instead:

```haskell
SmartAcquire = Acquire ; Analyse ; Decide
                 | Count(rare) ≥ 10 : stop
                 | else             : update(×)
```

The `| else : update(×)` says "feed new parameters back
and repeat". The `| condition : stop` says when to terminate.
The guard IS the recursion in the algebraic form.

**Rule 2 preserved:** no backward arrows anywhere. In the
visual stave, the box contains iterations stacked vertically.
In the algebra, guards express the recursion. Everything
flows left to right.

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


### 22. Smart microscopy (rare event detection)

Haskell form:

```haskell
SmartAcquire = Acquire ; Analyse ; Decide
                 | Count(rare) ≥ 10 : stop
                 | else             : update(×)
  where Acquire = ι(Jobs) × {Obj, Exp, Tiles, Z-range}
        Analyse = SegMeas | SNR > 5
        SegMeas = σ(Model) ; μ(MinSize)
        Decide  = φ(Score)
```

Visual form (on paper):

```
┌─ Acquire ; Analyse ; Decide | Count(rare) ≥ 10 ─┐
│               ⋮                                   │
└─ Acquire ; Analyse ; Decide                       ┘
      × S₀ = {Obj:10×, Exp:100ms, Tiles:overview}
```

Jobs = parameter space M. GA3 = the Stave pipeline.
Guards handle termination and quality gating.
The stave spec IS the smart microscopy protocol.

Formalises NIS-Elements NIS7 (Jobs + GA3, from LIM/Nikon)
as a guarded recursive composite with Para parametrisation.

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

### Haskell `where` for composite internals

`where` names parts of composites (Haskell convention):

```haskell
HCSPipeline = Preprocess ; SegMeas ; Statistics
  where SegMeas    = σ(Model) ; μ(MinSize)
        Preprocess = φ(Flatfield) ; φ(Gaussian)
        Statistics = τ ; μ(Mean, SD)
```

Rule: `where` is for naming, not for logic. Each `where`
binding is a named composite. Guards and predicates live
on the morphisms, not in the `where` clause. Structure
(;, where) and conditions (|) stay separate.

### Named composites with types

Type synonyms (the `type` keyword):

```haskell
type Image     = Y ∘ X : ℝ
type LabelMask = Y ∘ X : ℕ
type Graph     = (Nodes : ℝ) & (Edges : ℕ²)
type HCS       = Plate ∘ Wells ∘ Points ∘ Y ∘ X : ℝ
```

Composite signatures with `::`:

```haskell
segMeas :: Y ∘ X : ℝ → Obj : ℝ
segMeas = σ @Cellpose ; μ @Measure
  where
    σ :: forall s. s : ℝ → s : ℕ
    μ :: Y ∘ X : ℕ → Obj : ℝ
```

Protocols as typed arrows:

```haskell
SegMeas  :: Y ∘ X : ℝ  →  Obj : ℝ
HCS      :: Plate ∘ Wells ∘ Points ∘ Y ∘ X : ℝ  →  Stats : ℝ
Tracking :: t ∘ Y ∘ X : ℝ  →  (Det : ℝ) & (Links : ℕ²)
```

### Cellpose expansion — worked example

The composite `σ @Cellpose` expands to four internal
steps, each with its own typed signature and × parameters:

```haskell
σ @Cellpose :: (s : ℝ) × Params → s : ℕ
σ @Cellpose = φ_norm ; φ_net ; σ_thr ; ρ_trk
  where
    type Params = {model: Weights, diameter: ℝ,
                   flow_thr: ℝ, prob_thr: ℝ}

    φ_norm :: (s : ℝ)         × ℝ       → s : ℝ          -- normalise
    φ_net  :: (s : ℝ)         × Weights  → s : (ℝ² & ℝ)   -- neural net
    σ_thr  :: (s : ℝ² & ℝ)   × ℝ       → s : (ℝ² & Bool) -- threshold
    ρ_trk  :: (s : ℝ² & Bool) × ℝ       → s : ℕ           -- tracking
```

Observations:
1. Every step has lowercase `s` left of `:` → all preserve
   spatial structure → ══ passes over the whole composite.
2. Morphism letters reveal what each step does: φ (transform),
   φ (neural net = also transform), σ (type change ℝ→Bool),
   ρ (representation change — flows+mask into labels).
3. × parameters enter each step independently — Para construction.
4. Type inference propagates through `;`: φ_norm's output
   feeds φ_net's input, etc.
5. The composite's external type `s : ℝ → s : ℕ` is derived
   from first input and last output.

---

## Mathematical Foundation

Stave Notation = string diagram calculus of the
distributive cartesian double category Dbl(Img).

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

### Coherent logic internal to Dbl(Img)

The existing structural primitives, read logically,
give Stave an internal coherent logic. This was not
added — it was already present:

```
Stave          Logic                      Category theory
─────────────────────────────────────────────────────────────
&              ∧ (conjunction)            fibred product
×              ∧ (independent conjunction) cartesian product
⊂              contraction (use twice)    diagonal Δ
□              ⊤ (tautology/weakening)    terminal
| on container subobject selection        pullback along P : X → Ω
| on type      ∨ (disjunction)           coproduct
| on morphism  case analysis             copairing
μ              ∃ ("some element…")        left adjoint to substitution
;              ⇒ (modus ponens)          horizontal composition
```

Position in the logic hierarchy:
- Regular logic (∧, ∃, =) ✓
- Coherent logic (+ ∨) ✓ — the | type alternative and morphism guards
- Heyting logic (+ ⇒, ∀) — partially (⇒ implicit in ;)
- Full topos — theoretical ceiling, not needed for paper notation

### Distributive cartesian double category

Guards on morphisms (coproducts) are compatible with
the existing cartesian structure (products) if the
distributivity law holds:

```
& distributes over |
(A | B) & C  ≅  (A & C) | (B & C)
```

This holds for image data because containers are
orthogonal to case analysis — the spatial grid doesn't
care which branch the value took. The coproducts are
extensive (cases are disjoint, splits are stable under
container change).

Precise name: Dbl(Img) is a distributive cartesian
double category — cartesian (Patterson) + cocartesian
(coproducts from |) + distributive (products distribute
over coproducts). Distributive categories are well-known
(Cockett 1993, Carboni-Lack-Walters 1993); lifting to
double categories is a natural extension via
Lambert-Patterson's framework.

### Arrow/ArrowLoop correspondence

```
Stave   Arrow               Haskell type
──────────────────────────────────────────────────────────
;       >>>                 a b c → a c d → a b d
×       ***                 a b c → a d e → a (b,d) (c,e)
&       &&&                 a b c → a b d → a b (c,d)
⊂       arr id &&& arr id  a b (b,b)
□       arr (const ())      a b ()
⋮/guards loop               a (b,s) (c,s) → a b c
```

Stave without Para = Arrow.
Stave with Para = ArrowLoop.
ArrowLoop is the same typeclass Yampa (FRP) uses for
signal processing with feedback — microscopy time-lapse
with adaptive acquisition IS FRP.

### Formal typing rules

#### Composition (`;`)

```
Γ ⊢ f : S₁ : A → S₂ : B     Γ ⊢ g : S₂ : B → S₃ : C
─────────────────────────────────────────────────────────
              Γ ⊢ f ; g : S₁ : A → S₃ : C
```

Output type of `f` must unify with input type of `g`
on BOTH sorts.

#### Lift (Rule 3)

```
           Γ ⊢ f : S : A → S : B
  ───────────────────────────────────────
  Γ ⊢ lift(f) : T ∘ S : A → T ∘ S : B
```

Side condition: `S` must be the SAME on both sides of
`→` (structure preserved). This is why μ cannot be
lifted — its structure changes.

#### Parallel `×` (independent)

```
Γ ⊢ f : S₁ : A → S₂ : B    Γ ⊢ g : T₁ : C → T₂ : D
───────────────────────────────────────────────────────
  Γ ⊢ f × g : (S₁ × T₁) : (A × C) → (S₂ × T₂) : (B × D)
```

#### Fibred `&` (shared container)

```
Γ ⊢ f : S : A → S : B    Γ ⊢ g : S : C → S : D
──────────────────────────────────────────────────
      Γ ⊢ f & g : S : (A & C) → S : (B & D)
```

Shared `S` required — this is what distinguishes
`&` from `×` as a typing rule.

#### Copy and discard

```
⊂  : S : A → (S : A) × (S : A)       -- diagonal / copy
□  : S : A → 1                         -- terminal / discard
```

#### Para (parameter input)

```
        Γ ⊢ f : (S : A) × P → S : B
  ──────────────────────────────────────
       Γ ⊢ para(f) : S : A →[P] S : B
```

#### Loop (ArrowLoop / `⋮`)

```
  Γ ⊢ f : (S : A) × Q → (S : B) × Q
  ─────────────────────────────────────
       Γ ⊢ loop(f) : S : A → S : B
```

Feedback state `Q` is eliminated — stays internal.

#### Instantiation and generalisation

```
-- Instantiation
Γ ⊢ f : forall s a. S[s] : A[a] → T[s] : B[a]
─────────────────────────────────────────────────
Γ ⊢ f @S₀ @A₀ : S[S₀/s] : A[A₀/a] → T[S₀/s] : B[A₀/a]

-- Generalisation (let-polymorphism)
Γ ⊢ f : S : A → T : B    s₁…sₙ, a₁…aₘ ∉ FV(Γ)
──────────────────────────────────────────────────
Γ ⊢ f : forall s₁…sₙ a₁…aₘ. S : A → T : B
```

### Relationship to FVDblTT / VETT

The two-sorted HM type system is a practical
simplification of Nasu's FVDblTT (2024) and New &
Licata's VETT (2023) — the formal internal languages
of virtual double categories.

| FVDblTT concept | Stave equivalent |
|----------------|-----------------|
| Tight type `I` | `Str` (container configuration) |
| Tight term `s : I` | Structural operation (⊂ & × □) |
| Protype `α` (loose arrow) | Data wire with type `S : A` |
| Proterm `μ : β` (cell) | Morphism box (ι φ σ μ τ ρ ε) |
| Path protype `s ↛ t` | Type-changing wire `A → B` |
| Composition `⊙` | Pipeline `;` |
| Products `×`, `∧` | `×` and `&` |
| Comprehension `{|α|}` | Container + predicate `| P` |
| Protype isomorphism `α ≅ β` | `ρ` (representation change) |

The simplification is justified: Stave is a **fixed
theory** (7 generators, known rules), not a framework
for defining arbitrary double categories. FVDblTT is
for proving theorems about arbitrary virtual double
categories; Stave only needs to type-check pipelines
in one specific double category (Dbl(Img)).

**FVDblTT is to Stave as the Calculus of Constructions
is to Haskell** — the foundational theory that justifies
the practical type system, not the type system itself.

### Type system design rationale

| Type system | Economy | Diagrammatic? | Needed now? |
|------------|---------|--------------|-------------|
| **Two-sorted HM** (adopted) | Minimal | ✓ unlabelled wires = polymorphic | ✓ captures Rule 3 |
| Dependent types (Π, Σ) | Heavy | Hard (variable binding across wires) | Not yet — Objects:N is runtime |
| Linear types | Medium | Proof nets | Already implicit in dataflow |
| System F (rank-n) | Medium | Dinaturality | No rank-2 morphisms in imaging |
| GAT | Heavy | Natural for theories | Theoretical backing, not notation |

The two-sorted HM is the **largest type system that
still has a clean string-diagram reading**. Adding
exponentials (closedness) would lose the diagrammatic
side. Dropping the sorts would lose Rule 3 as a typing
rule. This is the sweet spot.

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
