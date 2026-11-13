# Poster 43 — Stave Notation: A Universal Diagrammatic Calculus for Bioimage Analysis

**Volodymyr Nechyporuk-Zloy**  
Facility for Imaging by Light Microscopy, Imperial College of Science, Technology and Medicine, London, UK

## Abstract

The remaining hard problem in bioimage analysis is not algorithmic but **structural**: how to specify, validate, and communicate the pipelines that connect raw acquisitions to biological conclusions. We present **Stave Notation**, a diagrammatic calculus in which container lines encode the nested structure of data, boxes encode operations, and the pipeline reads left to right.

Container lines stack to arbitrary depth. A single stroke (`──`) is a list; a double stroke (`══`) is a 2D image with spatial neighbourhood; a triple stroke (`≡≡`) is a volume. Additional lines above these encode time (`Frames:1..T`), high-content hierarchy (`Wells:384, Sites:9, Channels:6`), multi-scale pyramids, or any further nesting—each line is a container level, and their count is the nesting depth. Dependent cardinality (`Points(w):1..Nw`) distinguishes ragged containers from fixed-shape arrays. Cap operators (`⊃` contract, `⊂` extend) and a recursive-depth marker (`⋮`) handle branching structures, meshes, and graphs represented as paired node–edge stacks linked by `&`.

A type row above the value line carries the algebraic identity of each datum. For scalar intensities this is ℝ or ℕ; when geometry matters, Geometric Algebra grades—direction (v̂ ∈ Cl₁(3,0)), orientation (R ∈ Spin(3)), rigid-body pose (M ∈ Cl⁺(3,0,1)), conformal spheres, even complex Clifford algebra for polarisation optics—constrain downstream computation: a `μ` over orientations must compute a Fréchet mean on the group, not an arithmetic mean of angles. Seven morphism types (`ι`, `φ`, `σ`, `μ`, `τ`, `ε`, `ρ`) classify every operation; three connectors (`;`, `&`, `×`) compose them.

We validated the notation across six domains—molecular cell biology, bioengineering, genetics, cancer, neuroscience, immunology—formalising over **100 published pipelines** from **300+ references (2017–2026)**. Every domain-specific pattern (MIL inversion in pathology, cycle stacking in multiplexed imaging, connectome wiring in neuroscience) emerged as a compositional consequence of existing primitives. No extensions were required. The intended workflow is low-tech: draw a stave on paper, photograph it, upload the image with the specification as context to a large language model, which reads structure and types and generates executable code.

**Keywords:** Diagrammatic Calculus, Imaging
