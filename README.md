# Stave Notation

Add advanced pure and applied mathematics to your image
analysis by drawing on paper something like this:

```
Channels: DAPI               Channels: DAPI                Records: 1..N
───────────────               ───────────────               ───────────────
ξy,x: 1024     σ: Cellpose   ξy,x: 1024     μ: Measure    Columns: Area, Int
═══════════════ ──────────▶   ═══════════════ ──────────▶   ───────────────
ℝ: [0..2¹⁶]                  ObjectID ∈ ℕ                  ℝ
───────────────               ───────────────               ───────────────
```

Behind this sketch: category theory (containers are functors,
Rule 3 is a theorem), geometric algebra (oriented types that
enforce correct geometry), and a universal pipeline algebra
validated across 20 fields of bioimage analysis. In front of
it: pen, paper, and a phone camera.

**Three rules. Four primitives. Every bioimage pipeline.**

> Design on paper. Execute anywhere.

---

## The problem

Every imaging facility runs the same kinds of pipeline — segment
cells, measure features, filter by quality, fit a dose-response
curve — but every platform describes them differently. There is
no shared language. A pipeline drawn for one tool cannot be read
by another, and the structural logic of the workflow is buried
in platform-specific menus and scripts.

## The solution

Lines represent where data lives. Boxes represent what happens
to it. Three composition operators connect them. That's the
whole notation — and it is enough to describe every published
bioimage-analysis pipeline we have found.

```
──    wire      1D container (list, time series, table rows)
══    matrix    2D container (spatial grid — has neighbourhood)
≡≡    volume    3D container
□     box       transformation
```

Three rules govern the notation:

1. **Name above line. Bottom line closes.** Every container is
   labelled; the bottom line carries the value type.
2. **Strokes = dimensions.** `══` is literally two `──` stacked
   (Y and X axes).
3. **Box sits at the level it transforms.** This is a theorem,
   not a convention — containers are functors, and vertical
   position is *determined*, not chosen.

Seven morphism types (`ι φ σ μ τ ε ρ`) and three operators
(`;` sequential, `&` linked, `×` independent) compose into any
pipeline you need.

→ Full specification: [`spec/stave_notation.md`](spec/stave_notation.md)

---

## What you can do with it

**Sketch a pipeline on paper.** The notation is designed for
pen and paper. No software required. A stave is readable by
a colleague and by an LLM — the same diagram does both jobs.

**Generate code for any platform.** Upload a stave (Markdown
or photo) to a language model with the spec as context. Each
morphism maps to one function call; each type constrains the
code. The same stave produces Python, ImageJ macro, or
NIS-Elements GA3 — your choice.

```
Box  napari / scikit-image      ImageJ                NIS-Elements GA3
───  ────────────────────────   ────────────────────  ──────────────────
ι    tifffile.imread()          open()                Src.Channels
φ    skimage.filters.*          run("Median...")       Pre.Smooth
σ    threshold_otsu() / Cellpose setAutoThreshold()   Seg.Threshold
μ    regionprops_table()        Analyze Particles     Meas2.ObjectFeatures
τ    pd.merge(), groupby()      (manual)              Data.Reduce
ε    df.to_csv()                saveAs("Results")     Sink.SaveTables
ρ    marching_cubes()           —                     —
```

**Name and reuse pipelines.** Any sequential chain can be named
and collapsed into a single box. Composites are referentially
transparent — the name *is* its expansion.

```
SegMeas     = σ: Cellpose ; μ: Measure            ══ ℝ → ── ℝ
TrackAnalyse = σ/frame ; μ ; τ: Link ; μ           ══ ℝ → ── ℝ
HCSPipeline = σ ; μ ; τ: Accum ; τ: Reduce        ══ ℝ → ── ℝ
```

**Reason about structure.** The notation has a formal semantics
grounded in category theory — containers are functors, boxes are
2-cells in a double category, and Rule 3 is a theorem
(functorial lift). You don't need the mathematics to use the
notation, but it's there when you want it.

→ Categorical semantics: [`math/categorical_semantics.md`](math/categorical_semantics.md)
→ Geometric algebra types: [`math/geometric_algebra.md`](math/geometric_algebra.md)

---

## Validated across 20 fields

The notation has been tested against published pipelines
(2018–2026) in 20 fields of bioimage analysis. Each field
review draws real workflows as staves, defines named composites,
and asks: do we need new primitives? The answer, across ~1,100
references and ~330 named composites, has always been no.

| # | Field | Refs | Composites | What makes it distinctive |
|---|-------|-----:|:----------:|--------------------------|
| 1 | Molecular cell biology | 54 | 6 | The invariant 7-morphism pipeline |
| 2 | Bioengineering | 43 | 10 | Physics-based φ, cross-modal ρ |
| 3 | Genetics | 53 | 8 | Barcode decode ρ, genotype–phenotype & |
| 4 | Cancer | 42 | 12 | MIL inversion ρ;μ, genotype decoder |
| 5 | Immunology | 85 | 38 | Cycle stacking, categorical lift |
| 6 | Neuroscience | — | — | Cross-representational ρ |
| 7 | Microbiology | 63 | 18 | Mother-machine constraint, colony hierarchy |
| 8 | Drug discovery | 49 | 22 | Profile as universal intermediate |
| 9 | Cardiology | 50 | 17 | Temporal container universality |
| 10 | Endocrinology | 50 | 18 | Cross-body modality dispersion |
| 11 | Gastroenterology & Hepatology | 52 | 20 | Massive temporal τ:Filter |
| 12 | Neurology | 54 | 21 | Cross-representational ρ dominance |
| 13 | Nephrology | 50 | 19 | Multi-stain &, seven-order nested σ |
| 14 | Rheumatology | 50 | 23 | Multi-joint τ, scoring ρ |
| 15 | Physiology | 51 | 14+ | Temporal container universality, label-free ρ |
| 16 | Urology | 51 | 32 | Modality dispersion, real-time ρ |
| 17 | Virology | 55 | 19 | Sub-resolution object, ρ-dominance |
| 18 | Plant science | 90 | 14 | Boundary-first σ, ρ-heavy pipelines |
| 19 | Developmental biology | 65 | 15 | τ-dominance, evolving container hierarchy |
| 20 | Lung science | 70 | 26 | Volumetric τ:Classify, fractal σ cascade |

The structural diversity between fields lives in the *patterns
of composition* — which morphisms dominate, how containers nest,
what `&` links — not in the vocabulary itself.

→ Full reviews: [`reviews/`](reviews/)

---

## Origin

Stave Notation adapts Vincent Abbott's functor string diagrams
([Neural Circuit Diagrams](https://openreview.net/forum?id=RyZB4qXEgt),
TMLR 2024; [arXiv:2404.00249](https://arxiv.org/abs/2404.00249))
from deep learning to microscopy. Abbott contributed the visual
grammar and the key insight that lines are functors. The
extensions — type row, multiple morphism types, container nesting
depth, the `&` operator, Rule 3, and the double-category
structure — are original.

The name is by analogy with musical staves: lines carry meaning,
reading is left to right, and vertical position encodes
information.

---

## Repository structure

```
stave-notation/
├── README.md
├── LICENSE
├── CONTRIBUTING.md
│
├── spec/
│   └── stave_notation.md             formal specification (21 protocols)
│
├── math/
│   ├── categorical_semantics.md      functors, double categories, Rule 3
│   ├── geometric_algebra.md          GA types, grades, the cap
│   └── metric_geometry.md            metric structure on stave spaces
│
└── reviews/
    ├── 01_molecular_cell_biology.html
    ├── 02_bioengineering.html
    ├── ...
    └── 20_lung_science.html
```

---

## Quick start

1. Read the [specification](spec/stave_notation.md) — 21 worked
   protocols, from cell counting to whole-slide pyramids.
2. Look at a [field review](reviews/) in your area.
3. Draw a pipeline. Use the spec as an LLM context window.
   Generate code.

---

## Citation

```
Nechyporuk-Zloy, V. (2026). Stave Notation: A Diagrammatic
Calculus for Image Analysis Pipelines. GitHub repository.
https://github.com/<owner>/stave-notation
```

And the foundational visual grammar:

```
Abbott, V. (2024). Neural Circuit Diagrams: Robust Diagrams
for the Communication, Implementation, and Analysis of Deep
Learning Architectures. Transactions on Machine Learning
Research. https://openreview.net/forum?id=RyZB4qXEgt

Abbott, V. & Zardini, G. (2024). arXiv:2404.00249.
```

---

## License

[CC BY 4.0](LICENSE) — share and adapt for any purpose, with
attribution.
