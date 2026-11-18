# NIS-Elements AR Image Processing Nodes — Stave Notation

**Source:** https://www.nisoftware.net/NikonSaleApplication/Help/Docs-AR/eng_ar/node.imageprocessing_section.html
**Date:** 2026-11-18 15:46 UTC
**Notation version:** 2026-11-18_1439

Every node classified by morphism type (ι φ σ μ τ ρ ε), typed in
Haskell form with explicit sort boundary (`:`), and annotated
with box placement (Rule 3) and Para inputs (×).

Convention:
- `forall s.` → box below container lines (structure-polymorphic)
- Uppercase `S` → box spans that level (structure-specific)
- `× P` → Para parameter (independent, dashed line on paper)
- `& M` → fibred input (shared index)
- `spans ══` → box needs 2D neighbourhood context
- `spans ≡≡` → box needs 3D neighbourhood context
- `below ══` → per-pixel, no neighbourhood needed

---

## Master Table

```
Node                    Stave   Span        Val type
─────────────────────────────────────────────────────────────
DECONVOLUTION
  Deconvolution         φ       spans ══    ℝ → ℝ
  Fluo. Unmixing        φ       below ══    ℝ → ℝ  (channels reorganise)

BACKGROUND
  Auto Shading Corr.    φ       spans ══    ℝ → ℝ
  Rolling Ball          φ       spans ══    ℝ → ℝ
  Shading Correction    φ       spans ══    ℝ → ℝ
  Subtract Background   φ       spans ══    ℝ → ℝ  (& binary mask)
  Subtract Constant     φ       below ══    ℝ → ℝ
  Ax Shading Corr.      φ       spans ══    ℝ → ℝ

CONTRAST
  Auto Contrast         φ       below ══    ℝ → ℝ
  Contrast              φ       below ══    ℝ → ℝ
  Gauss-Laplace         φ       spans ══    ℝ → ℝ
  Local Contrast        φ       spans ══    ℝ → ℝ
  Sharpen               φ       spans ══    ℝ → ℝ
  Sharpen Slightly      φ       spans ══    ℝ → ℝ
  Ultra Details         φ       spans ══    ℝ → ℝ
  Unsharp Mask          φ       spans ══    ℝ → ℝ

CONVOLUTION
  Common Filters        φ       spans ══    ℝ → ℝ
  Gaussian Filters      φ       spans ══    ℝ → ℝ
  Golay Filter          φ       spans ══    ℝ → ℝ
  Mexican Hat           φ       spans ══    ℝ → ℝ

DENOISING
  Denoising             φ       spans ══+── ℝ → ℝ  (3D: reads neighbours in t/z)
  Denoising (fast)      φ       spans ══    ℝ → ℝ  (2D only)
  Destriping            φ       spans ══    ℝ → ℝ
  Low Pass Filter       φ       spans ══    ℝ → ℝ
  Median                φ       spans ══    ℝ → ℝ
  Smooth                φ       spans ══    ℝ → ℝ
  Smooth (fast)         φ       spans ══    ℝ → ℝ

DETECTION
  DIC Objects           φ       spans ══    ℝ → ℝ
  Edges                 φ       spans ══    ℝ → ℝ
  Filaments             σ       spans ══    ℝ → ℕ  (2D/3D filament detection)
  Gabor Edge Detect     φ       spans ══    ℝ → ℝ
  Gradient Morpho       φ       spans ══    ℝ → ℝ
  Peaks                 φ       spans ══    ℝ → ℝ
  Regional Maxima       σ       spans ══    ℝ → {0,1}
  Regional Minima       σ       spans ══    ℝ → {0,1}
  Valleys               φ       spans ══    ℝ → ℝ

INSERT
  Gradient Image        φ       below ══    ℝ → ℝ

MORPHOLOGY
  Open                  φ       spans ══    ℝ → ℝ  (also ℕ → ℕ on labels)
  Close                 φ       spans ══    ℝ → ℝ  (also ℕ → ℕ on labels)
  Erode                 φ       spans ══    ℝ → ℝ  (also ℕ → ℕ)
  Dilate                φ       spans ══    ℝ → ℝ  (also ℕ → ℕ)
  Fill Holes            φ       spans ══    ℝ → ℝ  (also {0,1} → {0,1})

LINEAR MORPHOLOGY
  Linear Open           φ       spans ══    ℝ → ℝ
  Linear Close          φ       spans ══    ℝ → ℝ
  Linear Erode          φ       spans ══    ℝ → ℝ
  Linear Dilate         φ       spans ══    ℝ → ℝ

LINEAR MORPHOLOGY 3D
  Linear Open Z         φ       spans ≡≡    ℝ → ℝ
  Linear Close Z        φ       spans ≡≡    ℝ → ℝ
  Linear Erode Z        φ       spans ≡≡    ℝ → ℝ
  Linear Dilate Z       φ       spans ≡≡    ℝ → ℝ

LABELS & CLASSES
  Smooth                φ       spans ══    ℕ → ℕ  (majority filter on labels)

TRANSFORMATIONS
  Add Borders           τ       spans ══    ℝ → ℝ
  Binning               τ       spans ══    ℝ → ℝ
  Crop                  τ       spans ══    ℝ → ℝ
  Change Canvas         τ       spans ══    ℝ → ℝ
  Fit Size              τ       spans ══    ℝ → ℝ
  Flip                  φ       spans ══    ℝ → ℝ  (coordinate permutation)
  Move                  φ       spans ══    ℝ → ℝ  (coordinate shift)
  Resize to Calibration τ       spans ══    ℝ → ℝ
  Resize to Ref         τ       spans ══    ℝ → ℝ  (× reference image)
  Rotate                τ       spans ══    ℝ → ℝ
```

**Count by morphism type:**

```
φ   53   (filter/transform — structure- and type-preserving)
σ    4   (segment/detect — structure-preserving, type-changing)
τ    8   (transform — container-restructuring)
μ    0   (no measurement nodes on this page)
ρ    0   (no representation-change nodes on this page)
ι    0   (no data-loading nodes on this page)
ε    0   (no export nodes on this page)
```

The Image Processing section is overwhelmingly φ. This is expected:
image processing = pixel-level transforms that preserve spatial
structure. σ, μ, τ, ρ, ι, ε live in other GA3 node categories
(Segmentation, Measurement, Data, Source, Sink).

---

## Typed Signatures

### Deconvolution

```haskell
-- Deconvolution
-- Auto-selects 2D or 3D method from image dimensionality and licence.
-- Spans ══ (or ≡≡ for 3D): PSF-based, reads spatial neighbours.
φ_Deconvolution :: forall s. s : ℝ → s : ℝ

-- Fluo. Unmixing (Nikon AX Confocal + DUX, widefield)
-- Per-pixel spectral decomposition. Box below ══.
-- Input channels → output components (channel container reorganises).
-- × SeparationStrength, × NDyes.
-- Requires GA3 + Local Option licence.
φ_FluoUnmixing :: forall s.
    (Channels s : ℝ) × {Strength: ℝ, NDyes: ℕ}
    → Components s : ℝ
```

### Background

```haskell
-- Auto Shading Correction
-- Spans ══: estimates background from image statistics.
φ_AutoShadingCorr :: forall s. s : ℝ → s : ℝ

-- Rolling Ball
-- Spans ══: rolls sphere over intensity surface.
-- × Radius (pixels, > largest object), × Signal (Bright|Dark).
φ_RollingBall :: forall s.
    (s : ℝ) × {Radius: ℕ, Signal: Bright | Dark}
    → s : ℝ

-- Shading Correction
-- Spans ══: polynomial background model.
-- × Smoothness, × CorrType (Light|Black|Average).
φ_ShadingCorr :: forall s.
    (s : ℝ) × {Smoothness: ℝ, CorrType: Light | Black | Average}
    → s : ℝ

-- Subtract Background (notable: & mask input)
-- Spans ══. Uses a binary mask sharing the spatial grid.
-- & BinaryMask (shared ξy,x), × MaskType (Background|Objects).
φ_SubtractBg :: forall s.
    (s : ℝ) & (s : {0,1}) × {MaskType: Background | Objects}
    → s : ℝ

-- Subtract Constant
-- Below ══: per-pixel arithmetic, no neighbours.
-- × Value.
φ_SubtractConst :: forall s.
    (s : ℝ) × {Value: ℝ}
    → s : ℝ

-- Ax Shading Correction (Confocal AX)
-- Spans ══. Uses pre-acquired correction image.
-- × CorrectionImage (same dimensions, independent = Para), × Strength.
φ_AxShadingCorr :: forall s.
    (s : ℝ) × {CorrImage: Y ∘ X : ℝ, Strength: ℝ}
    → s : ℝ
```

### Contrast

```haskell
-- Auto Contrast
-- Below ══: LUT remapping, per-pixel.
-- × Low, × High.
φ_AutoContrast :: forall s.
    (s : ℝ) × {Low: ℝ, High: ℝ}
    → s : ℝ

-- Contrast
-- Below ══: intensity mapping (LUT), per-pixel.
-- × Type (Linear|Log|Exp|Equalized|Gamma), × Gamma, × Range.
φ_Contrast :: forall s.
    (s : ℝ) × {Type: Linear | Log | Exp | Equalized | Gamma,
                Gamma: ℝ, Range: ℝ × ℝ}
    → s : ℝ

-- Gauss-Laplace
-- Spans ══: second-derivative sharpening at large scale.
-- × Power: [1.01..2], default 1.25.
φ_GaussLaplace :: forall s.
    (s : ℝ) × {Power: ℝ}
    → s : ℝ

-- Local Contrast
-- Spans ══: adaptive contrast from local statistics.
-- × Degree, × Radius.
φ_LocalContrast :: forall s.
    (s : ℝ) × {Degree: ℝ, Radius: ℕ}
    → s : ℝ

-- Sharpen
-- Spans ══: kernel-based sharpening.
φ_Sharpen :: forall s. s : ℝ → s : ℝ

-- Sharpen Slightly
-- Spans ══: gentle kernel sharpening.
φ_SharpenSlightly :: forall s. s : ℝ → s : ℝ

-- Ultra Details
-- Spans ══: multi-channel detail enhancement + denoising.
-- × Noise (denoising strength).
φ_UltraDetails :: forall s.
    (s : ℝ) × {Noise: ℝ}
    → s : ℝ

-- Unsharp Mask
-- Spans ══: Gaussian blur subtracted from original.
-- × Power: [0.01..1], × Area (smoothing extent).
φ_UnsharpMask :: forall s.
    (s : ℝ) × {Power: ℝ, Area: ℝ}
    → s : ℝ
```

### Convolution

```haskell
-- Common Filters (Sobel / Prewitt edge detection)
-- Spans ══: directional convolution kernel.
-- × Kernel (Sobel|Prewitt), × Direction (N|S|E|W).
φ_CommonFilters :: forall s.
    (s : ℝ) × {Kernel: Sobel | Prewitt,
                Direction: N | S | E | W}
    → s : ℝ

-- Gaussian Filters (blur, first/second derivatives)
-- Spans ══: Gaussian kernel or derivative thereof.
-- × Sigma, × FilterType (Blur|∂x|∂y|∂x²|∂y²).
-- Output is floating-point.
φ_GaussianFilters :: forall s.
    (s : ℝ) × {Sigma: ℝ,
                FilterType: Blur | ∂x | ∂y | ∂x² | ∂y²}
    → s : ℝ

-- Golay Filter
-- Spans ══: second-order polynomial fitting.
-- × Size (steps), × FilterType (Smooth|VEdges|HEdges|Edges).
φ_GolayFilter :: forall s.
    (s : ℝ) × {Size: ℕ,
                FilterType: Smooth | VEdges | HEdges | Edges}
    → s : ℝ

-- Mexican Hat
-- Spans ══: 5×5 Laplacian-of-Gaussian. No parameters.
φ_MexicanHat :: forall s. s : ℝ → s : ℝ
```

### Denoising

```haskell
-- Denoising (3D neighbourhood)
-- Spans ══ AND ── above (reads neighbouring frames/slices).
-- If data is t ∘ Y ∘ X : ℝ, box spans t, Y, X levels.
-- × Method (Original|Regression|Fusion|Bayes|Iterative), × Power.
φ_Denoising :: forall s.
    (s : ℝ) × {Method: Original | Regression | Fusion
                      | Bayes | Iterative,
                Power: ℝ}
    → s : ℝ
-- Note: despite forall s, the box spans all spatial+temporal
-- levels because it reads 3D neighbourhoods including adjacent
-- frames. Rule 3: identity on containers, active on values,
-- but READS container structure for context.

-- Denoising (fast) — 2D only, current frame
-- Spans ══ only (no temporal neighbours).
φ_DenoisingFast :: forall s.
    (s : ℝ) × {Method: Original | Regression | Fusion
                      | Bayes | Iterative,
                Power: ℝ}
    → s : ℝ

-- Destriping (VisiTech iSIM artefact removal)
-- Spans ══: directional frequency filter.
-- × Strength.
φ_Destriping :: forall s.
    (s : ℝ) × {Strength: ℝ}
    → s : ℝ

-- Low Pass Filter
-- Spans ══: frequency-domain cutoff.
-- × KeepLevel (minimum detail size to retain).
φ_LowPass :: forall s.
    (s : ℝ) × {KeepLevel: ℕ}
    → s : ℝ

-- Median
-- Spans ══: rank filter.
-- × Radius.
φ_Median :: forall s.
    (s : ℝ) × {Radius: ℕ}
    → s : ℝ

-- Smooth
-- Spans ══: iterative neighbourhood averaging.
-- × Matrix (structuring element), × Count (iterations).
φ_Smooth :: forall s.
    (s : ℝ) × {Matrix: SE, Count: ℕ}
    → s : ℝ

-- Smooth (fast) — Gaussian approximation
-- Spans ══.
-- × Sigma.
φ_SmoothFast :: forall s.
    (s : ℝ) × {Sigma: ℝ}
    → s : ℝ
```

### Detection

```haskell
-- DIC Objects
-- Spans ══: DIC-specific enhancement for downstream σ.
-- × Angle (illumination angle).
φ_DICObjects :: forall s.
    (s : ℝ) × {Angle: ℝ}
    → s : ℝ

-- Edges
-- Spans ══: generic edge enhancement.
φ_Edges :: forall s. s : ℝ → s : ℝ

-- Filaments (neuron/vessel detection)
-- σ: type changes ℝ → ℕ (filament labels).
-- Spans ══ (2D) or ≡≡ (3D).
-- × DiameterMin, × DiameterMax, × Steps, × Contrast.
σ_Filaments :: forall s.
    (s : ℝ) × {DiamMin: ℝ, DiamMax: ℝ, Steps: ℕ, Contrast: ℝ}
    → s : ℕ

-- Gabor Edge Detect
-- Spans ══: oriented frequency filter.
-- × Orientation, × Amplitude, × Filter (Real|Imag|Both),
-- × UseValue (Positive|Negative).
φ_GaborEdge :: forall s.
    (s : ℝ) × {Orientation: ℝ, Amplitude: ℝ,
                Filter: Real | Imag | Both,
                UseValue: Positive | Negative}
    → s : ℝ

-- Gradient Morpho
-- Spans ══: Dilate − Erode (morphological gradient).
-- × Matrix (SE), × Count.
φ_GradientMorpho :: forall s.
    (s : ℝ) × {Matrix: SE, Count: ℕ}
    → s : ℝ

-- Peaks (white top-hat)
-- Spans ══: TopHat = Image − Open(Image).
-- × Matrix (SE), × Count.
φ_Peaks :: forall s.
    (s : ℝ) × {Matrix: SE, Count: ℕ}
    → s : ℝ

-- Regional Maxima
-- σ: detects local maxima → binary markers.
-- Spans ══.
-- × Matrix (SE), × Count.
σ_RegionalMaxima :: forall s.
    (s : ℝ) × {Matrix: SE, Count: ℕ}
    → s : {0,1}

-- Regional Minima
-- σ: detects local minima → binary markers.
-- Spans ══.
-- × Matrix (SE), × Count.
σ_RegionalMinima :: forall s.
    (s : ℝ) × {Matrix: SE, Count: ℕ}
    → s : {0,1}

-- Valleys (black top-hat)
-- Spans ══: Close(Image) − Image.
-- × Matrix (SE), × Count.
φ_Valleys :: forall s.
    (s : ℝ) × {Matrix: SE, Count: ℕ}
    → s : ℝ
```

### Insert

```haskell
-- Gradient Image
-- Below ══: per-pixel overlay from a generated gradient.
-- × GradientTable, × Angle.
φ_GradientImage :: forall s.
    (s : ℝ) × {GradientTable: ℝ → ℝ, Angle: ℝ}
    → s : ℝ
```

### Morphology

```haskell
-- Open (Erode ; Dilate)
-- Spans ══. Removes small bright objects.
-- × Matrix (SE), × Count.
-- Note: also valid on label images (ℕ → ℕ).
φ_Open :: forall s a. (a ∈ {ℝ, ℕ, {0,1}}) =>
    (s : a) × {Matrix: SE, Count: ℕ}
    → s : a

-- Close (Dilate ; Erode)
-- Spans ══. Fills small dark holes.
-- × Matrix (SE), × Count.
φ_Close :: forall s a. (a ∈ {ℝ, ℕ, {0,1}}) =>
    (s : a) × {Matrix: SE, Count: ℕ}
    → s : a

-- Erode
-- Spans ══. Shrinks bright areas.
-- × Matrix (SE), × Count.
φ_Erode :: forall s a. (a ∈ {ℝ, ℕ, {0,1}}) =>
    (s : a) × {Matrix: SE, Count: ℕ}
    → s : a

-- Dilate
-- Spans ══. Grows bright areas.
-- × Matrix (SE), × Count.
φ_Dilate :: forall s a. (a ∈ {ℝ, ℕ, {0,1}}) =>
    (s : a) × {Matrix: SE, Count: ℕ}
    → s : a

-- Fill Holes
-- Spans ══. Flood-fills enclosed dark regions.
-- (Holes touching image border are preserved.)
φ_FillHoles :: forall s a. (a ∈ {ℝ, {0,1}}) =>
    s : a → s : a
```

### Linear Morphology (directional SE)

```haskell
-- All four: same as standard morphology but with
-- linear (1D directional) structuring element.
-- × Matrix (direction), × Count.

φ_LinearOpen  :: forall s. (s : ℝ) × {Matrix: SE, Count: ℕ} → s : ℝ
φ_LinearClose :: forall s. (s : ℝ) × {Matrix: SE, Count: ℕ} → s : ℝ
φ_LinearErode :: forall s. (s : ℝ) × {Matrix: SE, Count: ℕ} → s : ℝ
φ_LinearDilate :: forall s. (s : ℝ) × {Matrix: SE, Count: ℕ} → s : ℝ
```

### Linear Morphology 3D (Z-axis operations)

```haskell
-- Same morphological operations applied along Z axis.
-- Spans ≡≡ (needs z-neighbours, operates on 3D volume structure).

φ_LinearOpenZ   :: Z ∘ Y ∘ X : ℝ → Z ∘ Y ∘ X : ℝ    -- spans ≡≡
φ_LinearCloseZ  :: Z ∘ Y ∘ X : ℝ → Z ∘ Y ∘ X : ℝ
φ_LinearErodeZ  :: Z ∘ Y ∘ X : ℝ → Z ∘ Y ∘ X : ℝ
φ_LinearDilateZ :: Z ∘ Y ∘ X : ℝ → Z ∘ Y ∘ X : ℝ
-- Note: Z uppercase — these require specific 3D volume
-- structure. The box spans ≡≡ but preserves all levels.
```

### Labels & Classes

```haskell
-- Smooth (majority filter on label/class images)
-- Spans ══: replaces each pixel with mode in neighbourhood.
-- × Radius.
-- Type row: ℕ → ℕ (label images, NOT intensity).
φ_SmoothLabels :: forall s.
    (s : ℕ) × {Radius: ℕ}
    → s : ℕ
```

### Transformations

```haskell
-- Add Borders
-- τ: changes spatial extent (Y₁ ∘ X₁ → Y₂ ∘ X₂ where Y₂>Y₁, X₂>X₁).
-- × Top, × Left, × Right, × Bottom (pixels), × FillValue: [0..255].
τ_AddBorders ::
    (Y₁ ∘ X₁ : ℝ)
    × {Top: ℕ, Left: ℕ, Right: ℕ, Bottom: ℕ, Fill: ℝ}
    → Y₂ ∘ X₂ : ℝ

-- Binning
-- τ: reduces spatial resolution by aggregation.
-- × Factor, × Method (Max|Min|Mean|Sum|SumNoNoise).
τ_Binning ::
    (Y₁ ∘ X₁ : ℝ) × {Factor: ℕ, Method: Max | Min | Mean | Sum}
    → Y₂ ∘ X₂ : ℝ
-- where Y₂ = Y₁/Factor, X₂ = X₁/Factor

-- Crop
-- τ: extracts sub-region.
-- × Width, × Height, × StartX, × StartY.
τ_Crop ::
    (Y₁ ∘ X₁ : ℝ) × {W: ℕ, H: ℕ, X₀: ℕ, Y₀: ℕ}
    → Y₂ ∘ X₂ : ℝ
-- where Y₂ = H, X₂ = W

-- Change Canvas
-- τ: crops symmetrically around centre.
-- × Width, × Height.
τ_ChangeCanvas ::
    (Y₁ ∘ X₁ : ℝ) × {W: ℕ, H: ℕ}
    → Y₂ ∘ X₂ : ℝ

-- Fit Size
-- τ: downscale to fit square, preserve aspect ratio.
-- × Size, × Method (interpolation).
τ_FitSize ::
    (Y₁ ∘ X₁ : ℝ) × {Size: ℕ, Method: Interp}
    → Y₂ ∘ X₂ : ℝ

-- Flip
-- φ: coordinate permutation. Structure preserved (same dimensions).
-- Spans ══: needs the coordinate domain to compute reflection.
-- × Horizontal: Bool, × Vertical: Bool.
φ_Flip :: forall s.
    (s : ℝ) × {Horizontal: Bool, Vertical: Bool}
    → s : ℝ

-- Move
-- φ: coordinate shift. Structure preserved.
-- Spans ══.
-- × XOffset, × YOffset, × Wrap: Bool.
φ_Move :: forall s.
    (s : ℝ) × {XOffset: ℕ, YOffset: ℕ, Wrap: Bool}
    → s : ℝ

-- Resize to Calibration
-- τ: resamples to match a target pixel size.
-- × Method (interpolation).
τ_ResizeToCal ::
    (Y₁ ∘ X₁ : ℝ) × {Method: Interp}
    → Y₂ ∘ X₂ : ℝ

-- Resize to Ref
-- τ: resamples image A to match dimensions of reference B.
-- × RefImage (independent — not co-indexed), × Method.
τ_ResizeToRef ::
    (Y₁ ∘ X₁ : ℝ) × {Ref: Y₂ ∘ X₂ : ℝ, Method: Interp}
    → Y₂ ∘ X₂ : ℝ

-- Rotate
-- τ: rotates around centre (may change canvas dimensions).
-- × Angle.
τ_Rotate ::
    (Y₁ ∘ X₁ : ℝ) × {Angle: ℝ}
    → Y₂ ∘ X₂ : ℝ
```

---

## Stave Examples

### 1. Subtract Background (& mask input)

The only node in this set with a fibred (&) input.
The binary mask shares the spatial grid with the image.

```
ξy,x: 0..1023                 ξy,x: 0..1023
══════════════                 ══════════════
ℝ: [0..2¹⁶]       &          {0,1}
──────────────                 ──────────────
                  × MaskType: Background

; φ: SubtractBg

ξy,x: 0..1023
══════════════
ℝ: [0..2¹⁶]
──────────────
```

& because the mask shares ξy,x with the image — same pixels.
× MaskType is independent (same value for all pixels).


### 2. Denoising 3D (spans ══ + ──)

Reads neighbouring frames. Box must span the time container.

```
t: 1..T
──────────────
ξy,x: 0..1023         × Method: Bayes
══════════════         × Power: 0.5
ℝ: [0..2¹⁶]
──────────────

; φ: Denoising         ← box spans ── and ══

t: 1..T
──────────────
ξy,x: 0..1023
══════════════
ℝ: [0..2¹⁶]
──────────────
```

Expanded box view:
```
t: 1..T          reads t-1,t,t+1   t: 1..T
──────────────                      ──────────────
ξy,x: 0..1023   reads neighbours   ξy,x: 0..1023
══════════════                      ══════════════
ℝ               φ: Denoise         ℝ
──────────────                      ──────────────
```

The box spans from ℝ up through ══ and ──. All container
levels pass through (identity), but the morphism reads
neighbouring values across both t and ξy,x.

Compare with Denoising (fast): same, but the ── (time)
level is NOT spanned — only the current frame's ══ is read.


### 3. Fluo. Unmixing (channel reorganisation)

Per-pixel, below ══. The channel container reorganises.

```
Channels: λ₁..λ₃₂                   × Strength: 0.8
──────────────                       × NDyes: 4
ξy,x: 0..511
══════════════
ℝ: [0..2¹⁶]
──────────────

; φ: FluoUnmixing      ← below ══ (per pixel)

Components: DAPI, GFP, mCherry, AutoFluor
──────────────
ξy,x: 0..511
══════════════
ℝ: [0..1]
──────────────
```

The box sits below ══ — spatial passes over unchanged.
The top container line changes (Channels → Components).
Value range narrows (bit-depth → abundance [0..1]).
Per-pixel linear algebra: 32 wavelengths → 4 components.


### 4. Binning (spatial τ)

τ changes the spatial container dimensions.

```
ξy,x: 0..1023         × Factor: 2
══════════════         × Method: Mean
ℝ: [0..2¹⁶]
──────────────

; τ: Binning

ξy,x: 0..511
══════════════
ℝ: [0..2¹⁶]
──────────────
```

Container changed: 1024×1024 → 512×512.
Value type preserved: ℝ → ℝ.
τ because the container structure is reorganised (2×2 → 1).


### 5. Morphological Open on a label mask

Morphology nodes are polymorphic in value type.

On intensity image (ℝ):
```
ξy,x: 0..1023         × Matrix: disk(3)
══════════════         × Count: 1
ℝ: [0..2¹⁶]
──────────────

; φ: Open

ξy,x: 0..1023
══════════════
ℝ: [0..2¹⁶]
──────────────
```

On label mask (ℕ):
```
ξy,x: 0..1023         × Matrix: disk(3)
══════════════         × Count: 1
ObjectID ∈ ℕ
──────────────

; φ: Open

ξy,x: 0..1023
══════════════
ObjectID ∈ ℕ
──────────────
```

Same φ, same parameters. The type row (ℝ vs ℕ) determines
which underlying operation runs. Rule 3: box spans ══ in
both cases because morphological operations need neighbours.


### 6. Regional Maxima as σ (type change)

Detection with type change — intensity → binary markers.

```
ξy,x: 0..1023         × Matrix: disk(5)
══════════════         × Count: 1
ℝ: [0..2¹⁶]
──────────────

; σ: RegionalMaxima

ξy,x: 0..1023
══════════════
{0,1}
──────────────
```

σ because the type changes (ℝ → {0,1}): intensities become
binary markers at maxima positions. Spatial ══ preserved —
still 1024×1024, same grid. This is the Detection analogue
of Threshold: same spatial structure, different value meaning.

Downstream in a typical pipeline:
```
; σ: RegionalMaxima ; σ: Watershed ; μ: Measure
```


---

## Composition: NIS-Elements preprocessing composites

Typical NIS-Elements preprocessing chains expressed as named
composites in Stave:

```haskell
-- Standard preprocessing
Preprocess = φ_RollingBall(Radius=50, Signal=Dark)
           ; φ_Median(Radius=1)
           ; φ_AutoContrast(Low=0.1, High=99.9)

Preprocess :: forall s. s : ℝ → s : ℝ

-- Deconvolution preprocessing
DeconvPrep = φ_SubtractConst(Value=100)
           ; φ_Deconvolution

DeconvPrep :: forall s. s : ℝ → s : ℝ

-- Morphological cleanup (on binary/label images)
CleanMask = φ_Open(Matrix=disk(2), Count=1)
          ; φ_Close(Matrix=disk(2), Count=1)
          ; φ_FillHoles

CleanMask :: forall s. s : {0,1} → s : {0,1}

-- Spot detection chain
SpotDetect = φ_RollingBall(Radius=5, Signal=Dark)
           ; σ_RegionalMaxima(Matrix=disk(3), Count=1)

SpotDetect :: forall s. s : ℝ → s : {0,1}

-- Spatial downsampling chain
Downsample = τ_Binning(Factor=2, Method=Mean)
           ; τ_Crop(W=256, H=256, X₀=0, Y₀=0)

Downsample :: Y₁ ∘ X₁ : ℝ → Y₂ ∘ X₂ : ℝ
```

Every composite preserves the Rule 3 typing: an all-φ chain
(like Preprocess) stays `forall s` — the whole composite box
sits below the container lines. A chain containing τ
(like Downsample) has uppercase structure — the box spans ══.

---

## GA3 Platform Translation (expanded)

The spec's GA3 column maps categories. Here is the expanded
mapping from individual NIS-Elements AR image processing nodes
to GA3 recipe node names:

```
Stave         NIS-Elements AR Node         GA3 Recipe Node
─────────────────────────────────────────────────────────────
φ             Deconvolution                Pre.Deconvolution
φ             Fluo. Unmixing               Pre.FluoUnmixing
φ             Auto Shading Corr.           Pre.AutoShadingCorr
φ             Rolling Ball                 Pre.RollingBall
φ             Shading Correction           Pre.ShadingCorr
φ             Subtract Background          Pre.SubtractBg
φ             Subtract Constant            Pre.SubtractConst
φ             Ax Shading Correction        Pre.AxShadingCorr
φ             Auto Contrast                Pre.AutoContrast
φ             Contrast                     Pre.Contrast
φ             Gauss-Laplace                Pre.GaussLaplace
φ             Local Contrast               Pre.LocalContrast
φ             Sharpen                      Pre.Sharpen
φ             Unsharp Mask                 Pre.UnsharpMask
φ             Common Filters               Pre.CommonFilters
φ             Gaussian Filters             Pre.GaussianFilters
φ             Golay Filter                 Pre.GolayFilter
φ             Mexican Hat                  Pre.MexicanHat
φ             Denoising                    Pre.Denoising
φ             Denoising (fast)             Pre.DenoisingFast
φ             Median                       Pre.Median
φ             Smooth                       Pre.Smooth
φ             Open / Close / Erode / Dilate  Pre.Morphology
φ             Fill Holes                   Pre.FillHoles
σ             Filaments                    Seg.Filaments
σ             Regional Maxima              Seg.RegionalMaxima
σ             Regional Minima              Seg.RegionalMinima
τ             Binning                      Pre.Binning
τ             Crop                         Pre.Crop
τ             Rotate                       Pre.Rotate
```
