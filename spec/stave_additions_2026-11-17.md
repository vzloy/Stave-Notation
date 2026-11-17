# Stave Notation Additions — 2026-11-17

## 1. Para Construction (× as Parametrised Morphism)

**Source:** Femić, "Para construction for double categories" (arXiv 2510.26465, 2025) — lifts Gavranović's Para construction from 1-categories to double categories.

**Decision:** × parameter inputs are Para arrows (P, f) : P × X → Y in Para_M(Dbl(Img)). No new symbol needed — × already IS Para. Reparametrisation (collapsing a composite's internal × to exposed × with defaults) is a 2-cell in Para. Side outputs (quality metrics, auto-chosen thresholds) are coPara: X → P × Y, expressed via existing ⊂ branching.

**Key property:** Para does not add symbols to the notation. The four structural primitives (⊂ & × □), three line types (── ══ ≡≡), seven morphisms (ι φ σ μ τ ρ ε), and three rules remain unchanged. Same pattern as graphs (absorbed into &), predicates (absorbed into |), thick lines (absorbed into ∘).

## 2. Pipeline Recursion (⋮ for Iteration)

**Decision:** ⋮ marks pipeline iteration, reusing the existing recursion symbol. Visual convention: inside a composite box, the pipeline name appears twice with ⋮ between them — standard mathematical "this pattern continues" notation.

```
┌─ Acquire ; Analyse ; Decide | Cond ─┐
│               ⋮                      │
└─ Acquire ; Analyse ; Decide          ┘
```

**Disambiguation from data ⋮:** no ambiguity because position distinguishes — ⋮ on a container line = recursive data depth (W-type, trees); ⋮ inside a composite box between pipeline labels = pipeline iteration. Visual context is different on paper.

**Algebraic form:** ⋮ does NOT appear in the algebraic/Haskell notation. The recursion is expressed via guards instead:

```haskell
SmartAcquire = Acquire ; Analyse ; Decide
                 | Count(rare) ≥ 10 : stop
                 | else             : update(×)
```

The `| else : update(×)` says "feed new parameters back and repeat". The `| condition : stop` says when to terminate. The guard IS the recursion in the algebraic form. Two representations, one meaning.

**Rule 2 preserved:** no backward arrows anywhere. In the visual stave, the box contains iterations stacked vertically. In the algebra, guards express the recursion. Everything flows left to right.

## 3. Guards on Morphisms (| for Piecewise Functions)

**Decision:** | extends from containers and types to morphisms. Three levels, one symbol:

```
Container:  Objects: n | Area(n) > 50μm²       (which elements)
Type:       ℝ | ℕ | Text                        (which kind)
Morphism:   φ | v > 150 : brighten              (which branch)
              | v > 50  : identity
              | else    : darken
```

**Practical necessity:** thresholds, LUTs, quality gates, conditional processing — all require piecewise morphisms. Even a basic LUT (intensity → colour) is a morphism defined by predicate guards on the input:

```
φ | v ∈ [0, 50)    : ℝ → RGB(blue)
  | v ∈ [50, 150)  : ℝ → RGB(green)
  | v ∈ [150, 256) : ℝ → RGB(red)
```

**Categorically:** a piecewise morphism is a copairing [f₁, f₂, f₃] : X₁ + X₂ + X₃ → Y, where predicates partition the input into cases via coproduct injections. This is case analysis in a category with coproducts.

## 4. Coherent Logic Internal to Dbl(Img)

**Discovery:** the existing structural primitives, read logically, give Stave an internal coherent logic. This was not added — it was already present:

| Stave | Logic | Category theory |
|-------|-------|-----------------|
| & | ∧ (conjunction) | fibred product |
| × | ∧ (independent conjunction) | cartesian product |
| ⊂ | contraction (use a fact twice) | diagonal Δ |
| □ | ⊤ (tautology / weakening) | terminal |
| \| on container | subobject selection | pullback along P : X → Ω |
| \| on type | ∨ (disjunction) | coproduct |
| \| on morphism | case analysis | copairing |
| μ | ∃ ("some element satisfies…") | left adjoint to substitution |
| ; | ⇒ (modus ponens) | horizontal composition |

**Position in the logic hierarchy:**

- Regular logic (∧, ∃, =) ✓
- Coherent logic (+ ∨) ✓ — the | type alternative and morphism guards
- Heyting logic (+ ⇒, ∀) — partially (⇒ implicit in ;)
- Full topos — theoretical ceiling, not needed for paper notation

## 5. Distributive Cartesian Double Category

**Compatibility check:** guards on morphisms (coproducts) are compatible with the existing cartesian structure (products) if the **distributivity law** holds:

```
& distributes over |
(A | B) & C  ≅  (A & C) | (B & C)
```

This holds for image data because containers are orthogonal to case analysis — the spatial grid doesn't care which branch the value took. The coproducts are **extensive** (cases are disjoint, splits are stable under container change).

**Precise name:** Dbl(Img) is a **distributive cartesian double category** — cartesian (Patterson) + cocartesian (coproducts from |) + distributive (products distribute over coproducts). Distributive categories are well-known (Cockett 1993, Carboni-Lack-Walters 1993); lifting to double categories is a natural extension via Lambert-Patterson's framework, potentially a new contribution.

## 6. Haskell `where` for Composite Internals

**Decision:** adopt `where` from Haskell for naming parts of composites:

```haskell
HCSPipeline = Preprocess ; SegMeas ; Statistics
  where SegMeas    = σ(Model) ; μ(MinSize)
        Preprocess = φ(Flatfield) ; φ(Gaussian)
        Statistics = τ ; μ(Mean, SD)
```

**Rule:** `where` is for naming, not for logic. Each `where` binding is a named composite. Guards and predicates live on the morphisms, not in the `where` clause. Structure (;, where) and conditions (|) stay separate.

## 7. Smart Microscopy as Para + Guards

**Application:** NIS-Elements NIS7 (Jobs + GA3, from LIM/Nikon) for rare event detection formalised as a guarded recursive composite:

```haskell
SmartAcquire = Acquire ; Analyse ; Decide
                 | Count(rare) ≥ 10 : stop
                 | else             : update(×)
  where Acquire = ι(Jobs) × {Obj, Exp, Tiles, Z-range}
        Analyse = SegMeas | SNR > 5
        SegMeas = σ(Model) ; μ(MinSize)
        Decide  = φ(Score)
```

**Visual form (on paper):**

```
┌─ Acquire ; Analyse ; Decide | Count(rare) ≥ 10 ─┐
│               ⋮                                   │
└─ Acquire ; Analyse ; Decide                       ┘
      × S₀ = {Obj:10×, Exp:100ms, Tiles:overview}
```

Jobs = parameter space M. GA3 = the Stave pipeline. Guards handle termination and quality gating. The stave spec IS the smart microscopy protocol.

## 8. Four Representations of a Pipeline

Adding the Haskell/algebraic form with `where` and guards gives four equivalent representations:

```
1. Visual stave     — draw on paper, see every level and box placement
2. Thick line       — one wire per data flow, algebra label above
3. Type expression  — ∘ and & with → (compact signatures)
4. Haskell form     — guards, where, type constraints (for LLM code generation)
```

## 9. Arrow/ArrowLoop Correspondence

| Stave | Arrow | Haskell type |
|-------|-------|--------------|
| ; | >>> | a b c → a c d → a b d |
| × | *** | a b c → a d e → a (b,d) (c,e) |
| & | &&& | a b c → a b d → a b (c,d) |
| ⊂ | arr id &&& arr id | a b (b,b) |
| □ | arr (const ()) | a b () |
| ⋮ (visual) / guards (algebra) | loop | a (b,s) (c,s) → a b c |

Stave without Para = Arrow. Stave with Para = ArrowLoop. ArrowLoop is the same typeclass Yampa (FRP) uses for signal processing with feedback — microscopy time-lapse with adaptive acquisition IS FRP.

## 10. Literature References Added

- **Femić** (arXiv 2510.26465, 2025) — Para construction for double categories
- **Gavranović** (2024) — Categorical Foundations of Gradient-Based Learning (original Para for 1-categories)
- **Cockett** (1993), **Carboni-Lack-Walters** (1993) — Distributive categories
- **Hughes** (2000) — Generalising Monads to Arrows (Arrow/ArrowLoop)
- **Courtney & Elliott** (2001) — Yampa / FRP with ArrowLoop
