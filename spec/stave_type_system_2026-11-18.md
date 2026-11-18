# Stave Notation — Type System Specification

**Date**: 2026-11-18  
**Status**: Addendum to main specification  
**Depends on**: Existing notation (3 rules, 4 structural primitives, 7 morphisms, 4 representation levels)

---

## 1. Two-sorted type system

Stave Notation adopts a **two-sorted Hindley-Milner type system** as its type discipline. The two sorts correspond to the two dimensions of the double category Dbl(Img).

### 1.1 Sorts

| Sort | Name | What it describes | On the stave |
|------|------|-------------------|--------------|
| `Str` | Structure | Container configuration — WHERE data lives | Container lines (══ ── ≡≡) |
| `Val` | Value | Value type — WHAT kind of data | Type row |

Every data wire in a stave carries a pair `(S : Str, A : Val)`. Container lines are `Str`-sorted, the type row is `Val`-sorted. The sort distinction IS the double category's two dimensions.

### 1.2 Type grammar

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

### 1.3 Why two sorts, not one

Stave's type system is the internal language of a **distributive cartesian double category**, not a cartesian closed category (CCC). The critical difference:

- In a CCC (Haskell): `→` is an exponential — functions are data, stored in containers.
- In Dbl(Img) (Stave): `→` is a morphism arrow — morphisms operate ON containers, they do not live IN them. No `══ (ℝ → ℕ) ──`.

This means: **Stave = Haskell − closedness + a second sort.**

The two sorts are needed because an ordinary (one-sorted) category collapses the container/value distinction, making it impossible to distinguish structure-preserving operations (σ: same container, different type) from structure-changing operations (μ: different container, different type). The sort distinction is what makes Rule 3 a typing rule rather than a drawing convention.

---

## 2. The colon `:` — name : carrier

### 2.1 Semantics

The symbol `:` means **"named this, populated by that"** at every level of the notation. Left of `:` is the **name** (the mathematical structure, determining which operations are valid). Right of `:` is the **carrier** (the specific population of values).

The type row describes the **totality** — the entire set of possible values at every container position, not any single value. `:` is therefore not element-level membership (∈), but a type-level annotation: the typing of the type itself.

### 2.2 Three levels, one symbol

| Level | Left of `:` | Right of `:` | Example |
|-------|------------|-------------|---------|
| **Container** | Container name | Index range | `y,x : 0..1024` |
| **Type** | Mathematical structure | Carrier set | `ℝ : [0..2¹⁶)` |
| **Proarrow** | Container structure (Str) | Value type (Val) | `Y ∘ X : ℝ` |

At every level, `:` separates the **intension** (what kind of thing) from the **extension** (what populates it).

### 2.3 On the stave drawing

Every line on the stave uses `:`:

```
Channels : DAPI          container : population
─────────────────
y,x : 0..1024            container : population
═════════════════
ℝ : [0..2¹⁶)             structure : carrier
─────────────────
```

The visual boundary between container lines (above) and the type row (below) IS the `:` at the proarrow level. The stave already draws it — the algebraic form makes it explicit.

### 2.4 In the algebraic form: sort boundary

The `:` replaces the previously ambiguous final `∘` in type expressions:

| Before (ambiguous) | After (sort boundary explicit) |
|--------------------|-------------------------------|
| `Y ∘ X ∘ ℝ` | `Y ∘ X : ℝ` |
| `Obj ∘ ℝ` | `Obj : ℝ` |
| `(Nodes ∘ ℝ) & (Edges ∘ ℕ²)` | `(Nodes : ℝ) & (Edges : ℕ²)` |

Rule 3 is now **visible in the algebraic form**: check whether the structure left of `:` matches across `→`. Same structure both sides = box below the line; different structure = box spans the line.

---

## 3. The double colon `::` — morphism typing

### 3.1 Semantics

The symbol `::` means **"this morphism has this type"** — a typing judgement on a term (a morphism or pipeline).

### 3.2 Two symbols, two levels

| Symbol | Level | Question answered | Where it appears |
|--------|-------|-------------------|-----------------|
| `:` | Type-level | "What populates this name?" | Stave drawing + algebraic form |
| `::` | Term-level | "What type does this morphism have?" | Algebraic form only |

On the stave, the box itself IS the typing judgement — input wires determine the domain, output wires determine the codomain. `::` is therefore **never drawn** on the stave diagram; it exists only in the algebraic representation.

### 3.3 Syntax

```
morphism_name :: StructIn : ValIn → StructOut : ValOut
```

Examples:

```haskell
φ       :: s : a       →  s : a           -- structure-preserving, type-preserving
σ       :: s : ℝ       →  s : ℕ           -- structure-preserving, type-changing
μ       :: Y ∘ X : ℕ   →  Obj : ℝ         -- structure-reducing
τ       :: Obj : ℝ     →  (Obj : ℝ) & (Edges : ℕ²)  -- structure-creating
ρ       :: Y ∘ X : Bool →  (V : ℝ²) & (E : ℕ²)      -- restructuring
ι       :: s : a       →  s : a           -- identity
ε       :: s : a       →  □               -- discard
```

---

## 4. Rule 3 as a typing rule

### 4.1 The case convention

Haskell's case convention encodes Rule 3:

- **Lowercase** type variable (e.g., `s`, `a`) = **universally quantified** = polymorphic = box sits **below** the line (passes over)
- **Uppercase** concrete type (e.g., `Y ∘ X`, `Obj`) = **specific** = box **spans** the line (operates on it)

### 4.2 Generator types with explicit quantifiers

```haskell
ι :: forall s a.           s : a  →  s : a
φ :: forall s a.           s : a  →  s : a
σ :: forall s.             s : A  →  s : B       -- A, B specific Val types
μ ::                   S₁ : A  →  S₂ : B       -- S₂ < S₁ (reduces)
τ ::                   S₁ : A  →  S₂ : B       -- S₂ > S₁ (creates)
ρ ::                   S₁ : A  →  S₂ : B       -- both may change
ε :: forall s a.           s : a  →  □
```

The `forall s` on φ and σ IS Rule 3 — these morphisms work at any container depth, so the box sits below the lines. The absence of `forall s` on μ, τ, ρ means they require specific container structures, so the box must span those lines.

### 4.3 Constraints

Type-class-style constraints express Rule 3 and other structural requirements:

```haskell
φ :: (Spatial s) => s : a  →  s : a
μ :: (s ≥ Obj)   => s : ℕ  →  Obj : ℝ
```

---

## 5. Formal typing rules

### 5.1 Composition (`;`)

```
Γ ⊢ f : S₁ : A → S₂ : B     Γ ⊢ g : S₂ : B → S₃ : C
─────────────────────────────────────────────────────────
              Γ ⊢ f ; g : S₁ : A → S₃ : C
```

The output type of `f` must unify with the input type of `g` on BOTH sorts.

### 5.2 Lift (Rule 3)

```
           Γ ⊢ f : S : A → S : B
  ───────────────────────────────────────
  Γ ⊢ lift(f) : T ∘ S : A → T ∘ S : B
```

Side condition: `S` must be the SAME on both sides of `→` (structure preserved). This is why μ cannot be lifted — its structure changes.

### 5.3 Parallel `×` (independent)

```
Γ ⊢ f : S₁ : A → S₂ : B    Γ ⊢ g : T₁ : C → T₂ : D
───────────────────────────────────────────────────────
  Γ ⊢ f × g : (S₁ × T₁) : (A × C) → (S₂ × T₂) : (B × D)
```

No constraint on structures — they may differ.

### 5.4 Fibred `&` (shared container)

```
Γ ⊢ f : S : A → S : B    Γ ⊢ g : S : C → S : D
──────────────────────────────────────────────────
      Γ ⊢ f & g : S : (A & C) → S : (B & D)
```

Shared `S` required — this is what distinguishes `&` from `×` as a typing rule.

### 5.5 Copy and discard

```
⊂  : S : A → (S : A) × (S : A)       -- diagonal / copy
□  : S : A → 1                         -- terminal / discard
```

### 5.6 Para (parameter input)

```
        Γ ⊢ f : (S : A) × P → S : B
  ──────────────────────────────────────
       Γ ⊢ para(f) : S : A →[P] S : B
```

### 5.7 Loop (ArrowLoop / `⋮`)

```
  Γ ⊢ f : (S : A) × Q → (S : B) × Q
  ─────────────────────────────────────
       Γ ⊢ loop(f) : S : A → S : B
```

Feedback state `Q` is eliminated — it stays internal to the loop.

### 5.8 Instantiation and generalisation

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

---

## 6. Haskell notation adopted

### 6.1 Adopted

| Haskell convention | Stave usage |
|-------------------|-------------|
| `::` for type signatures | Morphism/pipeline typing |
| Lowercase = polymorphic, Uppercase = concrete | Rule 3 encoding |
| `forall` | Explicit polymorphism |
| `=>` after constraints | Rule 3 and structural constraints |
| `type` keyword | Named composites as type synonyms |
| `@` for type application | Instantiation of polymorphic morphisms |
| `where` clause | Composite internals |
| `\|` guards | Piecewise morphisms (already adopted) |

### 6.2 Not adopted

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

---

## 7. Named composites with types

### 7.1 Type synonyms

```haskell
type Image     = Y ∘ X : ℝ
type LabelMask = Y ∘ X : ℕ
type Graph     = (Nodes : ℝ) & (Edges : ℕ²)
type HCS       = Plate ∘ Wells ∘ Points ∘ Y ∘ X : ℝ
```

### 7.2 Composite signatures

```haskell
segMeas :: Y ∘ X : ℝ → Obj : ℝ
segMeas = σ @Cellpose ; μ @Measure
  where
    σ :: forall s. s : ℝ → s : ℕ
    μ :: Y ∘ X : ℕ → Obj : ℝ
```

### 7.3 Protocols as typed arrows

```haskell
SegMeas  :: Y ∘ X : ℝ  →  Obj : ℝ
HCS      :: Plate ∘ Wells ∘ Points ∘ Y ∘ X : ℝ  →  Stats : ℝ
Tracking :: t ∘ Y ∘ X : ℝ  →  (Det : ℝ) & (Links : ℕ²)
```

---

## 8. Example: Cellpose segmentation expanded

The composite `σ @Cellpose` expands to four internal steps, each with its own typed signature and × parameters:

```haskell
σ @Cellpose :: (s : ℝ) × Params → s : ℕ
σ @Cellpose = φ_norm ; φ_net ; σ_thr ; ρ_trk
  where
    type Params = {model: Weights, diameter: ℝ,
                   flow_thr: ℝ, prob_thr: ℝ}

    φ_norm :: (s : ℝ)         × ℝ       → s : ℝ         -- normalise
    φ_net  :: (s : ℝ)         × Weights  → s : (ℝ² & ℝ)  -- neural net → flows + cellprob
    σ_thr  :: (s : ℝ² & ℝ)   × ℝ       → s : (ℝ² & Bool) -- threshold cellprob
    ρ_trk  :: (s : ℝ² & Bool) × ℝ       → s : ℕ          -- gradient tracking → labels
```

**Observations**:

1. Every internal step has lowercase `s` left of `:` → all preserve spatial structure → ══ passes over the whole composite.
2. The morphism letters reveal what each step does: φ (transform values), φ (neural net = also value transform), σ (type change ℝ→Bool), ρ (representation change — flows+mask into labels).
3. × parameters enter each step independently — this is the Para construction.
4. Type inference propagates through `;`: φ_norm's output `s : ℝ` feeds φ_net's input, etc.
5. The composite's external type `s : ℝ → s : ℕ` is derived from first input and last output.

---

## 9. Relationship to FVDblTT / VETT

The two-sorted HM type system is a practical simplification of Nasu's FVDblTT (2024) and New & Licata's VETT (2023) — the formal internal languages of virtual double categories.

| FVDblTT concept | Stave equivalent |
|----------------|-----------------|
| Tight type `I` | `Str` (container configuration) |
| Tight term `s : I` | Structural operation (⊂ & × □) |
| Protype `α` (loose arrow) | Data wire with type `S : A` |
| Proterm `μ : β` (cell) | Morphism box (ι φ σ μ τ ρ ε) |
| Path protype `s ↛ t` | Type-changing wire `A → B` |
| Composition `⊙` | Pipeline `;` |
| Products `×`, `∧` | `×` and `&` |
| Comprehension `{|α|}` | Container + predicate `\| P` |
| Protype isomorphism `α ≅ β` | `ρ` (representation change) |

The simplification is justified: Stave is a **fixed theory** (7 generators, known rules), not a framework for defining arbitrary double categories. FVDblTT is for proving theorems about arbitrary virtual double categories; Stave only needs to type-check pipelines in one specific double category (Dbl(Img)).

**FVDblTT is to Stave as the Calculus of Constructions is to Haskell** — the foundational theory that justifies the practical type system, not the type system itself.

---

## 10. Design rationale: why HM and not more

| Type system | Economy | Diagrammatic? | Needed now? |
|------------|---------|--------------|-------------|
| **Two-sorted HM** (adopted) | Minimal | ✓ unlabelled wires = polymorphic | ✓ captures Rule 3 |
| Dependent types (Π, Σ) | Heavy | Hard (variable binding across wires) | Not yet — Objects:N is runtime |
| Linear types | Medium | Proof nets | Already implicit in dataflow |
| System F (rank-n) | Medium | Dinaturality | No rank-2 morphisms in imaging |
| GAT | Heavy | Natural for theories | Theoretical backing, not notation |

The two-sorted HM is the **largest type system that still has a clean string-diagram reading**. Adding exponentials (closedness) would lose the diagrammatic side. Dropping the sorts would lose Rule 3 as a typing rule. This is the sweet spot.

---

## Summary of additions

| Addition | Symbol | Level | New? |
|----------|--------|-------|------|
| Two sorts (Str, Val) | — | Type system | Yes |
| Sort boundary | `:` | Type-level (all representations) | Existing `:` given formal role |
| Morphism typing | `::` | Term-level (algebraic form only) | Yes (algebraic only) |
| Polymorphism | `forall`, lowercase/Uppercase | Type-level | Yes |
| Constraints | `=>` | Type-level | Yes |
| Type synonyms | `type` keyword | Algebraic form | Yes |
| Type application | `@` | Algebraic form | Yes |

**Symbol inventory unchanged**: `:` was already on every stave line; `::` appears only in the algebraic form. No new symbols are added to the stave drawing. The case convention (lowercase = polymorphic) is the only visual change, and it was already implicit in how generic pipelines are drawn (unlabelled wires).
