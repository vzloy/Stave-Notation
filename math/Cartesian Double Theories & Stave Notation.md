# Cartesian Double Theories & Stave Notation

How Lambert & Patterson's framework provides the mathematical foundations for Stave Notation — and what it unlocks next. \
Reference: M. Lambert & E. Patterson, *Cartesian double theories: A double-categorical framework for categorical doctrines*, arXiv:2310.05384v3 (2024)

[Overview](#overview)· [What is a CDT?](#what-is)· [Generators & Relations](#generators)· [The Mapping](#mapping)· [Models = Pipelines](#models)· [Cartesian Structure](#cartesian)· [Families & Containers](#families)· [Restriction Sketches](#restriction)· [Equipment of Models](#equipment)· [Remaining Machinery](#remaining)· [Implications](#implications)

## 1. Overview — Why This Paper Matters

Stave Notation's memory file records: *"Stave Notation = string diagram calculus of a cartesian double category Dbl(Img)."* This was established through iterative design — the four structural primitives (⊂ & × □), the seven morphism types (ι φ σ μ τ ρ ε), and the three rules all emerged from practical image analysis needs, then were recognised as categorical structures.

Lambert & Patterson's paper provides **the general theory of exactly this kind of structure**. A *cartesian double theory* is a small, strict cartesian double category 𝕋 presented by generators and relations. A *model* is a cartesian lax double functor 𝕋 → 𝕊 into a semantics. The paper proves that every such theory has a (virtual) double category of models — with lax transformations, modules, and multimodulations — giving the full mathematical ecology of a doctrine.

### The headline contribution

The Stave Notation spec file (the 2100-line markdown document with primitives, morphisms, rules, and protocols) **is** a presentation by generators and relations of a cartesian double theory. Every concrete image analysis pipeline **is** a model of that theory. Lambert–Patterson provides the proof that this is mathematically sound and the roadmap for what the theory gives you: a full double category of models, an equipment structure, and a restriction sketch mechanism for universal properties.

## 2. What Is a Cartesian Double Theory?

A **double category** 𝔻 has four kinds of data: *objects*, *arrows* (vertical morphisms), *proarrows* (horizontal morphisms), and *cells* (2-dimensional squares). Composition works in two independent directions: arrows compose vertically, proarrows compose horizontally, and cells compose in both.

Definition (Lambert–Patterson, Def. 6.1)

A **cartesian double theory** is a small, strict cartesian double category 𝕋.

A **model** of 𝕋 in a cartesian double category 𝕊 is a cartesian lax double functor F : 𝕋 → 𝕊. The receiving double category 𝕊 is called the **semantics**.

When the semantics is 𝕊pan (the double category of sets, functions, spans and maps of spans), models recover familiar categorical structures: categories, monoidal categories, multicategories, and more.

The word *lax* is crucial. A lax double functor preserves composition only up to coherent comparison cells (the *laxators*). In pipeline terms: composing two functions and then applying the functor may differ from applying the functor to each function and then composing — but the difference is tracked by a specific, coherent data transformation. This is exactly what happens when you compose pipeline steps: intermediate normalisation, padding, and bookkeeping mean strict preservation would be too rigid.

The word *cartesian* means the double category has finite products (in the sense of Grandis and Paré's limits in double categories). For Stave, this is the ⊂, &, ×, □ structure — the ability to copy, pair, separate, and discard data.

Key insight

**A doctrine specifies what extra structure a category can have.** Just as an ordinary theory (like the theory of groups) specifies what extra structure a set can have, a doctrine specifies what extra structure a category can have. Categories with finite products, monoidal categories, multicategories — each is determined by a different doctrine. Lambert–Patterson's framework lets you write down any such doctrine as a small double category with generators and relations, instead of building it from 2-monads.

## 3. Presentation by Generators and Relations

The paper's central contribution is showing that doctrines can be presented by **generators and relations** — just like groups. Instead of giving the full multiplication table of a group (impractical for infinite groups), you list the generators and the equations they satisfy. Similarly, instead of constructing a doctrine as a 2-monad on **Cat** (which requires heavy categorical machinery), you list the generating objects, arrows, proarrows, and cells, plus the equations they satisfy.

Examples from the paper

| Theory | Generators | Models are |
| --- | --- | --- |
| Unit theory (3.3) | Terminal 𝟙 | Categories |
| Walking arrow (3.4) | 𝟙 + one arrow 0→1 | Functors |
| Families (6.3) | Object x + proarrow p : 1 ⇸ x | Copresheaves (indexed families) |
| Monoids (6.4) | x + arrows ⊗ : x² → x, I : 1 → x + assoc/unit cells | Strict monoidal categories |
| Pseudomonoids (6.7) | Monoids + associator/unitor invertible cells | (Weak) monoidal categories |
| Finite products (6.18) | Restriction sketch with ⊗\*, I\*, projection and deletion cells | Cartesian categories |
| Cartesian promonoids (6.20) | Promonoids + restriction cells for all σ : \[m\] → \[n\] | Cartesian multicategories |

Each row in this table is a different doctrine — a different answer to "what extra structure can a category have?" The paper shows they all arise as models of cartesian double theories presented by generators and relations. This is the method Stave Notation needs.

Connection to Stave

**The Stave spec file IS a generators-and-relations presentation.** The four structural primitives are generators for the cartesian structure. The seven morphism types are generating cells. The three rules are relations (equations between composites of cells). The 21 protocols are worked examples showing that these generators suffice — they aren't additional axioms, they're theorems about the generating set.

## 4. The Mapping — Paper to Notation

Here is the precise correspondence between Lambert–Patterson's categorical machinery and Stave Notation's design elements.

| Lambert–Patterson | Stave Notation | Concrete example |
| --- | --- | --- |
| **Object** in 𝕋 | Data type in the type row | ℝ, ℕ, Text, ∧¹G³, Spin(3) |
| **Arrow** (vertical morphism) | Container functor | y ∘ x (spatial grid), t (time), Obj (objects) |
| **Proarrow** (horizontal morphism) | Data-flow wire — the lines carrying data between boxes | ══ ℝ ── flowing left to right through the pipeline |
| **Product of proarrows** (& / ×) | & = fibred product (co-indexed parallel flows) × = plain product (independent parallel flows) | Image & Metadata (shared index), View₁ × View₂ (independent) |
| **Cell** (2-dimensional square) | Morphism box (ι φ σ μ τ ρ ε) | σ: Cellpose, μ: Measure, φ: GaussianBlur |
| **Horizontal composition** of cells (along proarrows) | ; (sequential pipeline composition) | σ ; μ = SegMeas |
| **Vertical composition** of arrows | Container nesting (stacking lines) | z ∘ y ∘ x (three container levels) |
| **Laxator** (composition comparison) | Intermediate data rearrangement | Buffer between σ output and μ input |
| **Diagonal** Δ : x → x × x | ⊂ (copy) | One image copied to two branches |
| **Terminal** ! : x → 1 | □ (discard) | Wire that ends = data dropped |
| **Product** x × y | & (fibred) or × (plain) | Image & Calibration (shared index) |
| **Projection** π : x × y → x | Implicit (wire that ends) | Metadata branch ends after μ reads it |
| **Model** F : 𝕋 → 𝕊pan | Concrete pipeline | SegMeas, HCSPipeline, FacilityBatch |
| **Lax natural transformation** α : F ⇒ G | Pipeline adaptation | Swap Cellpose for StarDist in SegMeas |
| **Module** (bimodule/profunctor) | Data bridge between pipelines | Shared calibration table read by two workflows |
| **Modulation** | Transformation between data bridges | Changing calibration format between workflows |

## 5. Models = Pipelines — The Lax Functor Perspective

In Lambert–Patterson's framework, a concrete image analysis pipeline is a **model** — a cartesian lax double functor F : 𝕋Img → 𝕊pan. Let's unpack what this means component by component.

Orientation

**Proarrows are the wires, not the boxes.** In the double category, proarrows run horizontally (the pipeline direction). They carry data between stages. The morphism boxes (ι φ σ μ τ ρ ε) are *cells* — 2-dimensional squares with proarrows on top/bottom and arrows (container functors) on the sides. Bénabou's foundational observation: a lax functor F : 𝟙 → 𝕊pan sends the identity proarrow to a span C ← Mor(C) → C — the set of morphisms of the resulting category. Proarrows, when evaluated by a model, give the actual data transformations. The & and × operators are *products* of proarrows (parallel data flows), not proarrows themselves.

### What the functor assigns

```
The lax functor F for the pipeline "SegMeas"

F on objects (types):
  F(ℝ)  =  float32                    ← actual memory layout
  F(ℕ)  =  uint16                     ← actual label type

F on arrows (containers):
  F(y ∘ x)  =  {0..1023} × {0..1023}  ← actual pixel grid
  F(Obj)    =  {1..N}                  ← actual object count

F on proarrows (& and ×):
  F(&)  =  span linking image and metadata by shared File index
  F(×)  =  cartesian product (independent inputs)

F on cells (morphisms):
  F(σ)  =  cellpose.models.Cellpose(model_type='cyto2')
  F(μ)  =  skimage.measure.regionprops_table()

F on laxators (composition mediators):
  F(σ;μ)  ≅  F(σ) ; F(μ)   but the ≅ involves:
    - label cleanup (remove border objects)
    - type cast (float32 → uint16 labels)
    - memory management (release raw image after σ)
```

The **laxators** are the key. Strict functors would require F(σ ; μ) = F(σ) ; F(μ) on the nose. But in practice, composing pipeline steps involves intermediate bookkeeping — cleanup, type casting, memory management. The lax functor framework makes this explicit: the laxator Fσ,μ is a specific, coherent cell that witnesses "how the composite differs from the composition." The coherence axioms (associativity and unitality of laxators) say this bookkeeping is consistent — you get the same result regardless of how you parenthesise the pipeline.

Why lax and not strict?

**Strictness would force all intermediate data to match exactly.** In practice, σ: Cellpose outputs a float32 label image but μ: regionprops expects uint16 labels. The laxator converts between them. If the functor were strict, this conversion would be invisible — you'd need the types to match on the nose. Laxity makes the conversion explicit and trackable. This is why Stave uses ; (sequential composition) rather than strict equality: the semicolon IS the laxator.

### Different models of the same theory

Two pipelines that follow the same abstract pattern (σ ; μ) but use different tools are **two models of the same theory**:

Model F: SegMeas(Cellpose)

```
F(σ) = cellpose.Cellpose(cyto2)
F(μ) = regionprops_table()
F(y∘x) = 1024 × 1024
F(ℝ) = float32
F(ℕ) = uint16
```

Model G: SegMeas(StarDist)

```
G(σ) = stardist.StarDist2D(fluo)
G(μ) = regionprops_table()
G(y∘x) = 1024 × 1024
G(ℝ) = float32
G(ℕ) = uint16
```

A **lax natural transformation** α : F ⇒ G (Lambert–Patterson, Definition 7.1) is a systematic comparison between these two models. For each object x in the theory, αx maps F(x) to G(x). For each proarrow m : x ⇸ y, the naturality comparison cell says how F's and G's action on m relate. In pipeline terms: α says "for each data type and each container, here is how Cellpose's version and StarDist's version correspond" — and the naturality condition guarantees this correspondence is consistent across the whole pipeline.

## 6. The Cartesian Structure — ⊂ & × □

Lambert–Patterson's Theory 6.14 introduces the **theory of cartesian objects**: the theory of monoids augmented with the internal diagonal Δ, projections π and π', and the terminal arrow ! and its unit η. Theory 6.18 extends this to the **theory of finite products** using restriction sketches.

This is precisely the structure Stave Notation's four primitives embody:

```
Lambert–Patterson Theory 6.14 (Cartesian objects) generators:

  Δ : x → x²     diagonal         ⟺   ⊂  copy       Δ : X → X × X
  π : x² → x     projection       ⟺   (implicit)   wire ends = projection
  π': x² → x     projection       ⟺   (implicit)   second wire ends
  ε : x → 1      deletion         ⟺   □  discard   ! : X → 1
  η : 1 → x      unit             ⟺   (implicit)   terminal → object

The two product structures:

  x × y in the theory  with shared context  ⟺  &  fibred product
  x × y in the theory  without context      ⟺  ×  plain product

Proposition 6.19: A model of 𝕋_Fp is precisely a
cartesian category with chosen products and terminal object.
```

The paper proves (Proposition 6.19) that a model of the theory of finite products is **precisely** a category with chosen binary products and a chosen terminal object. This means Stave's four primitives aren't ad hoc design choices — they are the *axioms that make Dbl(Img) cartesian*. Removing any one would break the cartesian structure.

Confirmation

**The removal of ⊃ was categorically correct.** The Stave memory records that ⊃ (Abbott's contraction) was dropped because "in Dbl(Img) every contraction is a morphism (μ etc.) not a structural primitive." Lambert–Patterson confirms this: in a cartesian double category, contraction is *derived* from the cartesian structure (diagonal + composition), not a primitive. The four remaining primitives ⊂ & × □ are exactly the cartesian generators — no more, no less.

## 7. Theory of Families = Container Lines

One of the simplest examples in the paper has the deepest resonance with Stave. The **theory of families** (Theory 6.3) is freely generated by a single object x and a single proarrow p : 1 ⇸ x.

Theory 6.3 — Copresheaves (families)

A strict model in 𝕄at is a set X together with a family of sets indexed by X, denoted P : X → **Set**.

A model in 𝕄at is a category C together with a copresheaf P : C → **Set**.

This is **exactly** what a container line represents in Stave Notation:

```
A container line IS a model of the theory of families

Container line:       ξy,x: 0..1023
                      ══════════════

Interpretation:       The proarrow p : 1 ⇸ x is the spatial grid.
                      For each pixel position (i,j) ∈ {0..1023}²,
                      there is a set P(i,j) = the data at that pixel.

More container lines = more families composed:

z: 0..10              ← family of z-slices
──────────────
y,x: ξ 0..1023        ← family of pixel positions per slice
══════════════
ℝ: [0..2¹⁶]          ← values at each position
──────────────

This is the composite functor z ∘ y ∘ x ∘ ℝ
Three proarrows composed horizontally: p_z ⊙ p_yx ⊙ p_ℝ
```

The connection runs deep. The paper shows that proarrow composition in 𝕊pan is by pullback — and in Stave, composing container lines vertically is exactly nesting one indexed family inside another. The thick-line notation (z ∘ y ∘ x ∘ ℝ) is the functor composition that the paper's proarrow composition formalises.

The duality

**The theory of presheaves is the dual.** Generated by p : x ⇸ 1, its models are presheaves (contravariant functors). In Stave, this doesn't arise directly — image analysis pipelines are covariant (data flows forward). But the observation connects to the "remaining machinery" item about the Grothendieck construction: the total space of a family p : 1 ⇸ x in 𝕊pan is precisely the Grothendieck construction ∫ P, which is what ⋮ (recursive depth, W-type) represents.

## 8. Restriction Sketches = Container Predicates

One of the most recent additions to Stave Notation is the container predicate syntax: `ContainerName: var | Predicate`. Lambert–Patterson's **restriction sketches** (Definition 6.17) provide exactly the categorical framework for this.

Definition 6.17 — Restriction sketch

A restriction sketch is a cartesian double theory 𝕋 together with a distinguished subset of cells 𝕋res, closed under the operations in Lemma 4.13. A model of (𝕋, 𝕋res) in a cartesian equipment 𝕊 is a model F : 𝕋 → 𝕊 such that for every cell α ∈ 𝕋res, its image F(α) is a **restriction cell** in 𝕊.

In a cartesian equipment, a restriction cell witnesses a universal property — it says "this proarrow factors uniquely through a subobject." This is exactly what a container predicate does:

```
Container predicate as restriction cell

Without predicate:
  ξy,x: 0..1023                  full spatial domain
  ══════════════

With predicate (restriction):
  ξy,x | (y-y₀)² + (x-x₀)² < r²  restricted to circular ROI
  ══════════════

Categorically: the predicate defines a subobject S ↪ X,
and the restricted proarrow p|_S factors through S.
The restriction cell witnesses this factorisation.

More examples of restriction = predicates:

  t | t < t_bleach               temporal gate (pre-bleach frames)
  Objects: n | Area(n) > 50 μm²  object filter (large cells only)
  Wells: w ∈ {A1..A12}           subset of plate wells
```

The paper's Theory 6.18 (finite products) uses restriction sketches to express universal properties: the universal property of binary products and terminal objects are stated as "the following cells are restriction cells." This connects directly to Stave's design: the structural primitives ⊂ and □ satisfy universal properties (copying is unique, discarding is unique), and these universal properties are precisely restriction conditions.

What this unlocks

**Restriction sketches give predicates a precise categorical semantics.** Currently, Stave's predicates are informal: "restrict the domain." With the restriction sketch framework, each predicate is a cell in a distinguished set 𝕋res, and the closure conditions (Lemma 4.13) guarantee that predicates compose correctly: restricting a restricted container gives the intersection, and restriction commutes with pipeline composition. This means a type checker could verify predicate consistency mechanically.

## 9. The Virtual Equipment of Models

The paper's culminating results (Theorem 10.3, Corollaries 10.6–10.11) show that every cartesian double theory has a **virtual double category** — and in the right conditions, a **virtual equipment** — of models. This captures the full ecology of a facility's pipelines.

Theorem 10.3 — Virtual double category of lax functors

For any double categories 𝔻 and 𝔼, there is a unital virtual double category **Lax**ℓ(𝔻, 𝔼) whose objects are lax double functors 𝔻 → 𝔼, arrows are lax natural transformations, proarrows are modules, and multicells are multimodulations.

Corollary 10.11 — Virtual equipment of models

Let 𝕋 be a cartesian double theory and 𝕊 be a cartesian equipment. Then there is a virtual equipment of models of 𝕋 in 𝕊, with (lax, pseudo, or strict) maps between models, bimodules between models, and multitransformations between those.

For Stave Notation, this equipment structure captures everything a facility manager would want to formalise about how pipelines relate to each other:

| Equipment layer | Stave interpretation | Example |
| --- | --- | --- |
| **Objects** = models | Concrete pipelines | SegMeas, HCSPipeline, FacilityBatch, TrackAnalyse |
| **Arrows** = lax transformations | Pipeline adaptations | Swap Cellpose for StarDist, change threshold method |
| **Proarrows** = bimodules | Data bridges between pipelines | Shared calibration table, cross-reference lookup between workflows |
| **Cells** = multimodulations | Transformations of data bridges | Changing calibration format, migrating shared data between versions |
| **Restrictions** | Subpipeline extraction | "Run SegMeas on just the DAPI channel" = restrict the model to a subtheory |

The **virtual** qualifier means proarrow composition may not always exist (composing modules between lax functors is subtle). The paper gives sufficient conditions for representability (Proposition 10.4, Corollary 10.7): when the domain is a 2-category or a bicategory and the semantics is a cartesian equipment, the virtual double category is representable — meaning module composition always exists. For Stave, this holds when the theory is purely 2-categorical (which it is for the core theory without general proarrows).

## 10. The Remaining Machinery — What Gets Absorbed

The Stave memory file lists three items of "remaining machinery to absorb": the interchange law, companions/conjoints, and the Grothendieck construction. Here is what Lambert–Patterson provides for each.

### ✅ Interchange law

The interchange law for horizontal composition of 2-cells is **built into the axioms** of the virtual double category of models (Theorem 10.3). The proof verifies interchange explicitly for the binary composite of multimodulations (p. 92–94). In Stave terms: when two pipeline adaptations (lax transformations) are composed horizontally, the result is independent of the order — this is guaranteed by the framework.

### ✅ Companions and conjoints

Companions and conjoints arise in any equipment (Section 4). The paper defines cartesian equipments (Definition 4.1) and shows they have the companion/conjoint structure needed for the proarrow ↔ arrow interplay. Corollary 10.10 establishes the virtual equipment of cartesian lax functors. In Stave terms: every container functor (arrow) has a companion proarrow and a conjoint proarrow — these are the "induced data bridges" that any container nesting gives you for free.

### 🔶 Grothendieck construction (partial)

The Grothendieck construction — what ⋮ (recursive depth, W-type) represents — is not directly treated in this paper. However, the theory of families (Theory 6.3) and the profunctor perspective (Section 2, Definition 2.13–2.15) provide the categorical context. The total space of a family p : 1 ⇸ x in 𝕊pan, obtained by the Grothendieck construction, gives a category whose objects are pairs (x ∈ X, s ∈ P(x)) — elements of the indexed family. This is the categorical content behind variable-depth containers. A full treatment would need the theory of *displayed double categories* or *fibrations of double categories*, which is an active research direction the paper points toward in its conclusion.

## 11. Implications for the Project

### For the book (*Principles of Intelligent Microscopy*)

Citation and framing

Lambert–Patterson should be cited as providing the mathematical foundations for the claim that Stave Notation is a string diagram calculus of a cartesian double theory. The generators-and-relations method validates the "spec file = presentation, pipeline = model" architecture. The paper is by Michael Lambert (UMass Boston) and **Evan Patterson** (Topos Institute) — Patterson is also behind **AlgebraicJulia/Catlab**, the practical categorical computing framework, which demonstrates that formal categorical structures can be computed with, not just theorized about.

### For the spec

Three refinements the paper suggests

#### 1. Make restriction cells explicit

Identify which cells in the Stave theory are restriction cells. At minimum: the universal properties of ⊂ and □, and every container predicate (`| Predicate`). This gives the theory the structure of a restriction sketch (Definition 6.17), which unlocks the theory of categories with finite products (Proposition 6.19) as a specific model class.

#### 2. The theory of promonoids for multi-input morphisms

The paper's Theory 6.9 (promonoids, whose models are multicategories) is the right framework for μ morphisms that take multiple inputs. A measurement μ that reads both the label image and the calibration metadata through & is a multi-ary morphism — it takes two proarrows as input and produces one. The cartesian promonoid theory (Theory 6.20) gives this multi-ary structure the cartesian properties (copying and discarding inputs) that Stave already uses informally.

#### 3. Use the virtual equipment for pipeline comparison

The virtual equipment structure (Corollary 10.11) gives a formal framework for comparing pipelines (lax transformations), bridging data between them (modules), and transforming those bridges (multimodulations). This is the machinery needed for systematic pipeline validation: "does swapping Cellpose for StarDist give comparable results?" is a question about whether a specific lax transformation α exists and what its components look like.

### For the notation itself

### Nothing needs to change

**The paper confirms that the current design is categorically sound.** The four structural primitives are the axioms of a cartesian double category. The seven morphism types are generating cells. The three rules are equations in a presentation. The 21 protocols are theorems — consequences of the generators, not additional axioms.

The contribution is purely foundational: a precise statement of *what* Stave Notation *is* in the hierarchy of categorical doctrines. It sits alongside Lawvere theories (which formalise algebraic structure on sets) at the double-categorical level — formalising what extra structure a category (of image data types and transformations) can have.

### The Patterson / AlgebraicJulia connection

Evan Patterson's other work — the **AlgebraicJulia** ecosystem and **Catlab** — is directly relevant to the "LLM as parser" vision. Catlab already has computational implementations of:

- Double categories and lax functors as data structures
- Presentation of theories by generators and relations
- Computation with string diagrams (ACSet rewriting)

This means the Lambert–Patterson framework has an *implementation pathway*. The Stave spec could in principle be loaded into Catlab as a cartesian double theory, and concrete pipelines could be verified as models. The LLM would still be the primary interface (paper → photograph → LLM → code), but a Catlab backend could provide formal verification that the generated pipeline is a valid model of the spec.

### Paper's broader vision — and Stave's place in it

The paper's conclusion (§11) identifies two directions:

Direction 1: More structure on theories

"The theory of pseudomonoids should likewise be interpretable in any monoidal double category or even in any double multicategory."

**Stave angle:** As Stave absorbs more sophisticated composition patterns (adaptive pipelines, feedback loops, agentic meta-pipelines), the theory may need monoidal or multicategorical structure beyond the cartesian. Lambert–Patterson's framework is designed for exactly this kind of extension.

Direction 2: Higher morphisms

"We must go beyond the two-dimensional framework developed here to a fully three-dimensional structure encompassing… lax double functors, lax transformations, modules, and multimodulations."

**Stave angle:** A facility managing hundreds of pipelines needs to compare not just individual pipelines but *families* of pipelines — "all our HCS workflows" vs "all our tracking workflows." This is inherently three-dimensional: theories, models, and maps between them form a 2-category, and the passage to doctrines-of-doctrines is the paper's stated future direction.

## References

**\[LP24\]** M. Lambert and E. Patterson, "Cartesian double theories: A double-categorical framework for categorical doctrines," arXiv:2310.05384v3, April 2024.

**\[Abbott24\]** V. Abbott and A. Zardini, "Neural Circuit Diagrams: Robust Diagrams for the Communication, Implementation, and Analysis of Deep Learning Architectures," TMLR, 2024. arXiv:2404.00249.

**\[Comfort+20\]** C. Comfort, A. Delpeuch, and J. Hedges, "Sheet diagrams for bimonoidal categories," arXiv:2010.13361, 2020.

**\[Bonchi+22\]** F. Bonchi et al., "Tape diagrams for monoidal categories," arXiv:2210.09950, 2022.

**\[Melliès06\]** P.-A. Melliès, "Functorial boxes in string diagrams," CSL 2006, Springer LNCS 4207.

**\[Myers16\]** D. J. Myers, "String diagrams for double categories and equipments," arXiv:1612.02762, 2016.

**\[Patterson\]** E. Patterson, AlgebraicJulia/Catlab: [algebraicjulia.github.io/Catlab.jl](https://algebraicjulia.github.io/Catlab.jl/)

**\[GrandisPare99\]** M. Grandis and R. Paré, "Limits in double categories," Cahiers de topologie et géométrie différentielle catégoriques, 40(3):162–220, 1999.

Stave Notation — FILM, Imperial College London — November 2026