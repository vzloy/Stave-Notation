# Stave Notation — Graph Applications

24 graph structures in image analysis, all expressed with
vertical stacks, & linkage, and `{1..N}²` back-references.
No new symbols. GA where geometry demands it.
All tables use 3 lines: rows / columns / data type.

---

### 0. Cell adjacency from label image

```
DAPI
──────────────
ξy,x: 1024
══════════════
ObjectID ∈ ℕ
──────────────

; ρ: Voronoi adjacency

Cells: 1..N                  Adjacency: 1..E
──────────────               ──────────────
Columns: y,x,Area &          Columns: c₁,c₂
──────────────               ──────────────
ℝ                            {1..N}²
──────────────               ──────────────
```

---

### 1. Vascular skeleton with loop detection

```
Endothelial marker
──────────────
ξz,y,x: 256, 512, 512
≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡
ℝ: [0..2¹⁶]
──────────────

; φ: Denoise ; σ: Threshold ; ρ: Skeletonize3D

Junctions: 1..J              Segments: 1..S
──────────────               ──────────────
Columns: x,y,z    &          Columns: j₁,j₂
──────────────               ──────────────
v ∈ ⊛¹G³                    {1..J}²
──────────────               ──────────────
                                    &
                             Paths: 1..P
                             ──────────────
                             Columns: x,y,z
                             ──────────────
                             v ∈ ⊛¹G³
                             ──────────────

; μ: GraphFeatures

Records: 1..J
──────────────
Columns: Degree, LoopCount
──────────────
ℕ
──────────────
```

---

### 2. Neuron skeleton (graph, not tree)

```
GFP
──────────────
ξy,x: 2048
══════════════
ℝ: [0..2¹⁶]
──────────────

; φ: Denoise ; σ: Threshold ; ρ: Skeletonize

Nodes: 1..N                  Edges: 1..E
──────────────               ──────────────
Columns: type,y,x &          Columns: n₁,n₂
──────────────               ──────────────
{soma,junct,end}|ℝ          {1..N}²
──────────────               ──────────────

; τ: Flatten (graph → SWC)

Records: 1..N
──────────────
Columns: ID,ParentID,Type,x,y,z,r
──────────────
ℕ | ℝ
──────────────
```

---

### 3. Gene co-expression network

```
Genes: 1..G
──────────────
Samples: 1..S
──────────────
ℝ (log₂ FC)
──────────────

; μ: Correlation (pairwise Pearson)

Genes: 1..G                  CoExpr: 1..E
──────────────               ──────────────
Columns: GeneName  &          Columns: g₁,g₂,r
──────────────               ──────────────
text                         {1..G}² × ℝ
──────────────               ──────────────

; τ: Filter (|r| > 0.8) ; μ: GraphFeatures

Records: 1..G
──────────────
Columns: Degree, Betweenness, Module
──────────────
ℝ | ℕ
──────────────
```

---

### 4. Spatial k-NN for fibre orientation — GA

```
Fibres: 1..N
──────────────
Columns: orientation
──────────────
v ∈ ⊛¹G²
──────────────

; ρ: k-NN (k=6)

Fibres: 1..N                 Neighbours: 1..E
──────────────               ──────────────
Columns: orientation &       Columns: f₁,f₂,dist
──────────────               ──────────────
v ∈ ⊛¹G²                    {1..N}² × ℝ
──────────────               ──────────────

; μ: Alignment (v₁ · v₂ across edges)

Records: 1..N
──────────────
Columns: LocalAlignment
──────────────
ℝ: [0..1]
──────────────
```

---

### 5. CLEM fluorescence ↔ EM registration — GA

```
Fluorescence                  EM
──────────────               ──────────────
ξy,x: 1024       &          ξy,x: 8192
══════════════               ══════════════
ℝ: [0..2¹⁶]                 ℝ: [0..255]
──────────────               ──────────────

& linked by: φ: Motor M ∈ G³˙¹
(y',x') = M (y,x) M̃
```

---

### 6. Spatial transcriptomics (gene × spot)

```
Spots: 1..S                  Expression: S × G
──────────────               ──────────────
Columns: x,y      &          Columns: Genes 1..G
──────────────               ──────────────
ℝ²                           Count ∈ ℕ
──────────────               ──────────────

; μ: DimReduce (UMAP)

Records: 1..S
──────────────
Columns: UMAP1, UMAP2, ClusterID
──────────────
ℝ | ℕ
──────────────
```

---

### 7. Organelle-to-cell membership

```
Cells: 1..N                  Organelles: 1..M
──────────────               ──────────────
Columns: Area,Type &          Columns: parent,Area,Int
──────────────               ──────────────
ℝ                            {1..N} × ℝ
──────────────               ──────────────

parent ∈ {1..N}: function (each organelle → one cell)

; μ: ChildrenCount

Records: 1..N
──────────────
Columns: MitoCount, LysoCount
──────────────
ℕ
──────────────
```

---

### 8. Multi-resolution field-of-view linking

```
Overview                     ROIs: 1..R
──────────────               ──────────────
ξy,x: 4096       &          Columns: y₀,x₀
══════════════               ──────────────
ℝ: [0..2¹⁶]                 {Overview coords}
──────────────               ──────────────
                                    ;
                             ROI data: per ROI
                             ──────────────
                             ξy,x: 2048
                             ══════════════
                             ℝ: [0..2¹⁶]
                             ──────────────
```

---

### 9. Cross-modal registration with PGA motors — GA

```
Modality₁                     Modality₂
──────────────               ──────────────
ξy,x: 1024                  ξy,x: 1024
══════════════               ══════════════
v ∈ ⊛¹G³                    v ∈ ⊛¹G³
──────────────               ──────────────

; φ: Motor M ∈ G³˙¹ on Modality₂
; ⊃ contract (overlay)

Registered
──────────────
ξy,x: 1024
══════════════
v ∈ ⊛¹G³
──────────────
```

---

### 10. Inter-frame tracking with motors — GA

```
Detections: 1..D             Links: 1..L
──────────────               ──────────────
Columns: pos       &         Columns: d₁,d₂
──────────────               ──────────────
v ∈ ⊛¹G³                    {1..D}² × M ∈ G³˙¹
──────────────               ──────────────

Motor M per link: rotation + translation.

; μ: MotionFeatures

Records: 1..K
──────────────
Columns: Speed, AngVelocity, Chirality
──────────────
ℝ
──────────────
```

---

### 11. Two relations on same cells

```
Cells: 1..N
──────────────
Columns: x,y,Expression
──────────────
ℝ | ℕ
──────────────

       &                          &

SpatialAdj: 1..E₁                ExprSim: 1..E₂
──────────────                   ──────────────
Columns: c₁,c₂                  Columns: c₁,c₂
──────────────                   ──────────────
{1..N}² × dist∈ℝ                {1..N}² × sim∈ℝ
──────────────                   ──────────────

; μ: CombinedGraphFeatures

Records: 1..N
──────────────
Columns: SpatialDeg, ExprDeg, CrossModularity
──────────────
ℝ
──────────────
```

---

### 12. Cell adjacency with relative orientation — GA

```
Cells: 1..N                  Adjacency: 1..E
──────────────               ──────────────
Columns: orientation &       Columns: c₁,c₂
──────────────               ──────────────
v ∈ ⊛¹G²                    {1..N}² × R ∈ ⊛⁰G²+⊛²G²
──────────────               ──────────────

R₁₂ = v₂ v₁⁻¹  (relative rotor)

; μ: OrientationOrder

Records: 1..N
──────────────
Columns: NematicOrder
──────────────
ℝ: [0..1]
──────────────
```

---

### 13. Table → label image foreign key

```
DAPI                         Records: 1..M
──────────────               ──────────────
ξy,x: 1024       &          Columns: ObjectID,Area
══════════════               ──────────────
ObjectID ∈ ℕ                 {ObjectIDs} × ℝ
──────────────               ──────────────

ObjectID in table = foreign key into label image.
```

---

### 14. Surface mesh (Δ₂) — GA

```
Vertices: 1..V               Faces: 1..F
──────────────               ──────────────
Columns: pos       &         Columns: v₁,v₂,v₃
──────────────               ──────────────
v ∈ ⊛¹G³                    {1..V}³ × B ∈ ⊛²G³
──────────────               ──────────────

B = (v₂-v₁) ∧ (v₃-v₁)
|B| = area. B/|B| = normal.
```

---

### 15. Volume mesh (Δ₃) — GA

```
Vertices: 1..V               Cells: 1..C
──────────────               ──────────────
Columns: pos       &         Columns: v₁,v₂,v₃,v₄
──────────────               ──────────────
v ∈ ⊛¹G³                    {1..V}⁴ × T ∈ ⊛³G³
──────────────               ──────────────

T = (v₂-v₁) ∧ (v₃-v₁) ∧ (v₄-v₁)
|T| = 6 × volume. sign(T) = handedness.
```

---

### 16. Skeleton graph ↔ tree conversion (ρ)

```
GRAPH:                           TREE:

Nodes: 1..N                      Nodes: 1..N
──────────────                   ──────────────
Columns: y,x       &             Columns: y,x       &
──────────────                   ──────────────
ℝ²                               ℝ²
──────────────                   ──────────────

Edges: 1..E          ρ:Span     Edges: 1..N-1
──────────────       ──────▶    ──────────────
Columns: n₁,n₂                  Columns: child,parent
──────────────                   ──────────────
{1..N}²                          {1..N}²
──────────────                   ──────────────

E edges → N-1 edges. Cycles removed.
```

---

### 17. Nematic order from edge rotors — GA (μ collapse)

```
Cells: 1..N                  Adjacency: 1..E
──────────────               ──────────────
Columns: orientation &       Columns: c₁,c₂
──────────────               ──────────────
v ∈ ⊛¹G²                    {1..N}² × R ∈ ⊛⁰G²+⊛²G²
──────────────               ──────────────

; μ: NematicOrder (global)

Scalar: 1
──────────────
Columns: S
──────────────
ℝ: [0..1]
──────────────

Both stacks collapse to one scalar.
S=1: aligned. S=0: isotropic.
```

---

### 18. Per-node graph features

```
Nodes: 1..N                  Edges: 1..E
──────────────               ──────────────
Columns: data      &         Columns: n₁,n₂,weight
──────────────               ──────────────
ℝ                            {1..N}² × ℝ
──────────────               ──────────────

; μ: PerNodeFeatures

Records: 1..N
──────────────
Columns: Degree, Clustering, Betweenness
──────────────
ℕ | ℝ
──────────────

& consumed. Edge stack → per-node features.
```

---

### 19. Data fusion (× → & → merge)

```
Source₁                       Source₂
──────────────               ──────────────
Cells: 1..N        ×         Cells: 1..M
──────────────               ──────────────
Columns: FluorFeat           Columns: MassFeat
──────────────               ──────────────
ℝ                            ℝ
──────────────               ──────────────

; ρ: Spatial matching

Matches: 1..K
──────────────
Columns: n,m,confidence
──────────────
{1..N} × {1..M} × ℝ
──────────────

; τ: Join

Records: 1..K
──────────────
Columns: FluorFeat, MassFeat
──────────────
ℝ
──────────────

× → & → single table.
```

---

### 20. Cell tracking with merge and split

```
Detections: 1..D             Links: 1..L
──────────────               ──────────────
Columns: x,y,t,Area &       Columns: d₁,d₂,type
──────────────               ──────────────
ℝ                            {1..D}² × {move,split,merge}
──────────────               ──────────────

Split: one d₁ → two d₂.
Merge: two d₁ → one d₂.
Graph, not tree — merges create cycles.
```

---

### 21. Tissue deformation field — GA

```
Cells_t: 1..N                Cells_t+1: 1..N
──────────────               ──────────────
Columns: pos       &         Columns: pos
──────────────               ──────────────
v ∈ ⊛¹G³                    v' ∈ ⊛¹G³
──────────────               ──────────────

       &

Adjacency: 1..E
──────────────
Columns: c₁,c₂
──────────────
{1..N}² × M ∈ G³˙¹ (deformation motor)
──────────────

M = T · R · S (translate · rotate · strain)

; μ: StrainFeatures

Records: 1..E
──────────────
Columns: Stretch, Shear, Rotation
──────────────
ℝ
──────────────
```

---

### 22. Curvature mismatch between adjacent cells — GA

```
Cells: 1..N                  Adjacency: 1..E
──────────────               ──────────────
Columns: MeanCurv,normal &   Columns: c₁,c₂,ΔB
──────────────               ──────────────
ℝ × B ∈ ⊛²G³               {1..N}² × ⊛²G³
──────────────               ──────────────

|ΔB| = curvature discontinuity at junction.

; μ: MismatchFeatures

Records: 1..N
──────────────
Columns: MeanMismatch, MaxMismatch
──────────────
ℝ
──────────────
```

---

### 23. Community detection on combined graphs

```
Cells: 1..N
──────────────
Columns: x,y,Expression
──────────────
ℝ | ℕ
──────────────

       &                          &

SpatialAdj: 1..E₁                ExprSim: 1..E₂
──────────────                   ──────────────
Columns: c₁,c₂                  Columns: c₁,c₂
──────────────                   ──────────────
{1..N}² × dist∈ℝ                {1..N}² × sim∈ℝ
──────────────                   ──────────────

; τ: CombineEdges (weighted union)
; μ: Leiden community detection

Records: 1..N
──────────────
Columns: CommunityID, Modularity
──────────────
ℕ | ℝ
──────────────
```

---

### 24. Bipartite spot ↔ gene expression

```
Spots: 1..S                  Genes: 1..G
──────────────               ──────────────
Columns: x,y       &         Columns: GeneName
──────────────               ──────────────
ℝ²                           text
──────────────               ──────────────

       &

Detections: 1..D
──────────────
Columns: spot,gene,count
──────────────
{1..S} × {1..G} × ℕ
──────────────

Bipartite: detections reference BOTH index sets.

; μ: PerSpot              × ; μ: PerGene

Records: 1..S              Records: 1..G
──────────────              ──────────────
Columns: TotalCount         Columns: TotalCount
──────────────              ──────────────
ℕ                           ℕ | ℝ
──────────────              ──────────────
```

---


---

## Composites in Graph Workflows

Graph analysis chains are natural candidates for
named composites:

```
GraphAnalyse =
    ρ: Voronoi ; μ: GraphFeatures

SkeletonAnalyse =
    ρ: Skeletonize ; μ: BranchMeas

TrackGraph =
    σ/frame ; μ: Detect ; τ: Link ; μ: MotionFeatures
```

Each composite carries its & internally:

```
GraphAnalyse expands to:

(input: ══ ObjectID)
  ; ρ: Voronoi → Nodes & Edges
  ; μ: GraphFeatures → Records
(output: ── ℝ)

The & between Nodes and Edges is INTERNAL
to the composite. The caller never sees it.
```


## Summary

24 graph applications. Five patterns:

```
Pattern              Shape                    Read as
─────────────────────────────────────────────────────────────
1. Self-ref          A & E(A²)               "A connected to A"
2. Shared index      A & B                   "two views of same things"
3. Multi-relation    A & E₁ & E₂             "same things, two connections"
4. Fusion            A × B ; ρ ; &           "match then merge"
5. Bipartite         A & D & B               "two types linked through D"
```

No new symbols. All 3-line tables. GA where geometry demands it.
