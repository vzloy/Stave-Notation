# Stave Notation — n8n Integration

Every n8n node maps to one of seven morphisms plus
two caps and two input types. No exceptions.

---

## Complete n8n Node Mapping

### Triggers and Input (ι: introduce)

```
n8n node                 Stave    Notes
──────────────────────────────────────────────────
Manual trigger           ι        user action
Cron / Schedule          ι        timed, × schedule param
Webhook                  ι        HTTP event
Watch folder             ι        filesystem event
Email trigger            ι        inbox event
Kafka / MQTT             ι        message queue event
Form submission          ι        user input
Database trigger         ι        row change event
Read file                ι        from disk
HTTP Request GET         ι        from API
Google Drive read        ι        from cloud
OMERO fetch              ι        from repository
RSS feed                 ι        from web
FTP download             ι        from server
S3 get object            ι        from bucket
Database SELECT          ι        from query
Error trigger            ι        error event
Redis GET                ι        from cache
```

### Value Transform (φ: pixel-level / preserve structure)

```
n8n node                 Stave    Notes
──────────────────────────────────────────────────
Set node                 φ        change values, same rows
Code (value transform)   φ        JS/Python on values
Rename keys              φ        rename columns
Type conversion          φ        cast data types
Crypto / Hash            φ        transform values
Date/Time format         φ        transform values
HTML extract             φ        text → structured
XML / JSON parse         φ        format → structured
Markdown to HTML         φ        transform format
Text operations          φ        string manipulation
Math operations          φ        compute on values
Regex extract            φ        pattern → values
LLM generate text        φ        prompt → text
Database UPDATE          φ        change values in place
```

### Classification (σ: segment / label)

```
n8n node                 Stave    Notes
──────────────────────────────────────────────────
Text Classifier          σ        label data
Sentiment analysis       σ        text → category
LLM classify             σ        prompt → category
Code (classification)    σ        compute labels
```

### Aggregation (μ: measure / collapse)

```
n8n node                 Stave    Notes
──────────────────────────────────────────────────
Summarize                μ        collapse rows → stats
Aggregate (with func)    μ        GROUP BY + SUM/AVG/COUNT
LLM summarise            μ        collapse text → summary
Code (aggregation)       μ        custom collapse
```

### Restructure (τ: preserve depth, change arrangement)

```
n8n node                 Stave    Notes
──────────────────────────────────────────────────
Filter                   τ        keep matching rows
Sort                     τ        reorder rows
Item Lists               τ        group, split, aggregate
Remove duplicates        τ        filter unique
Limit                    τ        keep first N
Concatenate              τ        append rows (UNION)
Pivot                    τ        reshape table
Unpivot / Melt           τ        reshape table
Flatten                  τ        nested → flat
Edit fields              τ        add/remove/rename columns
Wait / Pause             τ        identity (delay, no data change)
Compare datasets         τ        match / no-match split
```

### Output (ε: eliminate / save / notify)

```
n8n node                 Stave    Notes
──────────────────────────────────────────────────
Write file               ε        to disk
Google Drive write       ε        to cloud
S3 put object            ε        to bucket
FTP upload               ε        to server
OMERO upload             ε        to repository
Google Sheets push       ε        to sheet
Database INSERT          ε        to database
Database DELETE          ε        remove data
Redis SET                ε        to cache
Slack send               ε        notify
Email send               ε        notify
Telegram send            ε        notify
Discord send             ε        notify
SMS send                 ε        notify
Webhook response         ε        respond
Stop and error           ε        error exit
```

### Representation Change (ρ: convert encoding/format)

```
n8n node                 Stave    Notes
──────────────────────────────────────────────────
CSV → JSON               ρ        data format change
JSON → XML               ρ        data format change
Excel → JSON             ρ        data format change
PDF extract              ρ        document → structured
Image resize             ρ        spatial representation
Compress / Zip           ρ        storage representation
Base64 encode/decode     ρ        encoding change
```

### Branching and Merging (caps: ⊂ ⊃ &)

```
n8n node                 Stave    Notes
──────────────────────────────────────────────────
IF                       ⊂ ; τ    copy then filter each branch
Switch                   ⊂ ; τ    copy then filter N branches
Split in batches         ⊂        distribute to parallel
Merge (combine)          ⊃ or &   depends on shared key
  - append mode          τ        concatenate (UNION)
  - merge by key         &        shared index JOIN
  - combine all          ⊃        full contraction
Try / Catch              ⊂ ; τ    route success / failure
Loop                     ⊂ ; τ    output feeds back to input
```

### AI / LLM Nodes (morphism depends on what it computes)

```
n8n node                 Stave    Determined by function
──────────────────────────────────────────────────
LLM: generate text       φ        value → value (same rows)
LLM: classify            σ        value → label
LLM: summarise           μ        many → one (collapse)
LLM: decide branch       ⊂ ; τ    copy then route
Agent (iterative)        ⊂ ; τ    loop: decide, act, evaluate
Vector store read        ι        retrieve
Vector store write       ε        persist
```

### Meta / Flow Control

```
n8n node                 Stave    Notes
──────────────────────────────────────────────────
NoOp                     ── wire  identity (no box)
Wait                     ── wire  identity with delay
Execute Workflow         ;        sequential composition
Sub-workflow             ;        nested composition
```

### Parameters (× independent inputs)

```
n8n concept              Stave    Notes
──────────────────────────────────────────────────
Credentials              ×        same for all items
Environment variables    ×        same for all items
Static values            ×        same for all items
Retry count              ×        error handling config
Timeout                  ×        execution config
Batch size               ×        parallelism config
```

---

## Summary: Seven Morphisms Cover Everything

```
ι   introduce    triggers, reads, fetches, events
φ   pixel-level  value transforms, format changes
σ   segment      classify, label, categorise
μ   measure      aggregate, summarise, collapse
τ   restructure  filter, sort, group, pivot, wait
ε   eliminate    write, send, notify, upload, delete
ρ   represent    format conversion, encoding change
```

Plus structural operations:

```
⊂   copy         split, branch, fan-out
⊃   contract     merge, join, combine
&   link         shared key merge
×   parameter    config, credentials, constants
;   sequential   workflow composition
```

---

## Facility Workflows in Stave Notation

### 1. Nightly batch with quality gate

```
ι: WatchFolder(/incoming/)
──────────────
Columns: path, modified
──────────────
text
──────────────

; ι: ReadND2 per file

Files: 1..F
──────────────
Channels: DAPI, GFP
──────────────
ξy,x: 1024
══════════════
ℝ: [0..2¹⁶]
──────────────

       &

Files: 1..F
──────────────
Columns: SNR, Saturation%
──────────────
ℝ
──────────────

; ⊂ ; τ: Filter per branch

PASS (SNR > 3):               FAIL (SNR ≤ 3):
Files: 1..P                   Files: 1..Q
──────────────                ──────────────
(pipeline continues)          Columns: path, SNR
                              ──────────────
; σ ; μ: ObjectMeas           text
                              ──────────────
Records: 1..N
──────────────                ; ε: Slack(#qc-failures)
Columns: File, Count, Area    ; ε: MoveFolder(/rejected/)
──────────────
ℝ | text
──────────────

; τ: Aggregate per file
; ε: CSV(/results/{date}.csv)
; ε: GoogleSheets("Experiment Log")
; ε: Slack(#facility, "Done: {P} passed, {Q} rejected")
```

### 2. Weekly instrument health report

```
ι: Cron(Monday 07:00)
──────────────
Columns: timestamp
──────────────
text
──────────────

; ι: ReadCalibrationImages per instrument

Instruments: {AX, NSPARC, SIM}
──────────────
ξy,x: 1024
══════════════
ℝ: [0..2¹⁶]
──────────────

       &

Instruments: {AX, NSPARC, SIM}
──────────────
Columns: expected_resolution, expected_uniformity
──────────────
ℝ
──────────────

; μ: FieldUniformity ; μ: PSF width

Records: {AX, NSPARC, SIM}
──────────────
Columns: Instrument, Uniformity%, PSFnm, Status
──────────────
ℝ | text
──────────────

; ⊂ ; τ per branch

OK:                            DRIFT:
; ε: GoogleSheets              ; ε: Email(engineer,
  ("Health Log")                 "URGENT: {Instrument} drift")

; ε: Slack(#instruments, "Weekly QC: {P} OK, {Q} attention")
```

### 3. OMERO ingest + auto-annotate

```
ι: Webhook(OMERO, "new import")
──────────────
Columns: datasetID, userID
──────────────
text
──────────────

; ι: HTTP GET(OMERO API)

Files: 1..F                  Files: 1..F
──────────────               ──────────────
Channels: varies   &         Columns: filename, date, objective
──────────────               ──────────────
ξy,x: varies                text
══════════════               ──────────────
ℝ: [0..2¹⁶]
──────────────

; σ: Cellpose ; μ: Count per image

Records: 1..F
──────────────
Columns: imageID, CellCount, MeanArea, Channels
──────────────
ℝ | text
──────────────

; τ: GenerateTags (count>100→"dense", DAPI→"nuclear")

Tags: 1..T
──────────────
Columns: imageID, tag
──────────────
text
──────────────

; ε: HTTP POST(OMERO, annotate)
; ε: Slack(#data, "Imported {F}, auto-tagged")
```

### 4. Experiment results → PDF → PI email

```
ι: FormSubmit(experiment_id, PI_email)
──────────────
Columns: experiment_id, PI_email
──────────────
text
──────────────

; ι: ReadCSV(/results/{experiment_id}/)

Records: 1..N
──────────────
Columns: Well, CellCount, MeanInt, Ratio
──────────────
ℝ | text
──────────────

       &

PlateLayout: 1
──────────────
Columns: Well, Condition, Concentration
──────────────
text | ℝ
──────────────

; τ: GroupBy Well ; τ: Statistics
; τ: DoseResponse ; τ: Z′-factor

Results: 1
──────────────
Columns: EC₅₀, Z′, MaxEffect
──────────────
ℝ
──────────────

; ε: GeneratePDF(table + plot)
; ε: Email(PI_email, "EC₅₀ = {EC₅₀}")
; ε: GoogleDrive(/shared/{PI}/{id}/)
```

### 5. Disk space monitor + auto-archive

```
ι: Cron(hourly)
──────────────
Columns: timestamp
──────────────
text
──────────────

; ι: CheckDisk(/data/)

DiskStatus: 1
──────────────
Columns: total_TB, used_TB, percent
──────────────
ℝ
──────────────

; ⊂ ; τ per branch

OK (< 85%):                  WARNING (≥ 85%):
; (no action)                 ; ι: ListOldFiles(> 90 days)

                              OldFiles: 1..N
                              ──────────────
                              Columns: path, size, age, owner
                              ──────────────
                              text | ℝ
                              ──────────────

                              ; ε: Email(each owner, "7 day warning")
                              ; (wait 7 days)
                              ; ε: MoveToArchive
                              ; ε: Slack("Freed {size} TB")
```

### 6. Live timelapse monitor

```
ι: Webhook(microscope, "frame acquired")
──────────────
Columns: frame_path, t
──────────────
text | ℕ
──────────────

; ι: ReadFrame

Frame: 1
──────────────
ξy,x: 512
══════════════
ℝ: [0..2¹²]
──────────────

; σ: QuickThreshold ; μ: Count + MeanInt

Record: 1
──────────────
Columns: t, CellCount, MeanInt
──────────────
ℝ
──────────────

; τ: AppendToLog

TimeSeries: 1..t
──────────────
Columns: t, CellCount, MeanInt
──────────────
ℝ
──────────────

; ⊂ copy three checks

Drift?                   Death?                Confluent?
; τ: Δ(MeanInt, 10)     ; τ: Δ(CellCount)    ; τ: last Count

If |Δ| > 20%:           If Δ < -50%:          If count > max:
; ε: Slack("Focus       ; ε: Slack("Cell      ; ε: Slack("Confluent
   drift t={t}")           death t={t}")          at t={t}")
                                                ; ε: HTTP(microscope,
                                                   "stop")
```

### 7. Multi-user booking → acquire → results

```
ι: CalendarWebhook(instrument booked)
──────────────
Columns: user, instrument, date, protocol
──────────────
text
──────────────

; ι: LookupProtocol(protocol)

                    ×

Protocol
──────────────
Columns: morphisms (Stave MD file)
──────────────
text
──────────────

; (user acquires data during slot)

ι: WatchFolder(/incoming/{user}/{date}/)
──────────────
Columns: path
──────────────
text
──────────────

; Execute protocol (LLM interprets Stave → Python)

Records: 1..N
──────────────
Columns: (from protocol)
──────────────
ℝ | text
──────────────

; ε: CSV(/results/{user}/{date}/)
; ε: Email(user, "Results ready")
; ε: GoogleSheets("Facility Log",
    {user, instrument, date, N_images, N_cells})
; ε: Invoice(user, hours × rate)
```

---

## Structural Summary

n8n adds three things the notation already handles:

```
n8n concept              Stave equivalent
────────────────────────────────────────────────
WHEN to run              ι with × schedule parameter
WHAT IF failure          ⊂ ; τ (branch success/fail)
WHERE to output          ε (multiple sinks via ⊂)
```

Stave is WHAT to compute. n8n is WHEN, WHERE, and
WHAT IF. Same notation, different operational concerns.


---

## Stave Type as n8n Data Contract

n8n currently passes data blind — it doesn't know if
it's sending a 3D volume to a 2D tool. Stave's type
information fixes this. Every image gets a Stave type
tag at ι, propagated and validated at every step.

### Type tag structure

ι: ReadAndType extracts the Stave type from file
metadata and attaches it as & linked data:

```
{
  "path": "/data/experiment.nd2",
  "stave": {
    "lines": [
      {"name": "Channels", "members": ["DAPI","GFP"], "stroke": 1},
      {"name": "t", "cardinality": 100, "stroke": 1},
      {"name": "ξy,x", "cardinality": [1024,1024], "stroke": 2},
      {"name": "ℝ", "range": [0,65535], "stroke": 1}
    ],
    "links": [
      {"type": "&", "target": "calibration",
       "data": {"pixelSize": 0.065, "unit": "μm"}}
    ]
  }
}
```

### Automatic tool selection from Stave type

n8n reads stroke count and routes to the correct tool:

```
stroke == 2 (══, 2D):
    DAPI present → HTTP Cellpose(model="cyto2")
    no DAPI      → HTTP StarDist(model="versatile")

stroke == 3 (≡≡, 3D):
    → HTTP Cellpose3D(model="cyto3")

lines contains "t":
    → apply σ per frame (map over t)
    → not σ on whole block

ℝ range [0,255]:
    → 8-bit, adjust threshold
ℝ range [0,65535]:
    → 16-bit, different normalisation
```

### Automatic parameter calculation

```
σ: Cellpose:
  pixel_size = stave.links["calibration"]["pixelSize"]
  → diameter = 30μm / 0.065 μm/px = 461 pixels
  Without stave: user guesses. With stave: calculated.

φ: Gaussian:
  user says "smooth 0.5μm"
  → sigma = 0.5 / 0.065 = 7.7 pixels

μ: Measure:
  → area in μm² not px²
  → perimeter in μm not pixels
  Automatic unit conversion from & calibration.
```

### Type validation at every connection

```
Connection           Type contract
──────────────────────────────────────────────────
ι → σ               must have ══ or ≡≡, must have ℝ
σ → μ               must have ══ or ≡≡, must have ℕ (labels)
σ → φ               must have ══ or ≡≡, must have ℕ
μ → τ               must have ── (table), columns defined
τ → τ               same ── depth, columns may change
τ → ε               any table type
```

n8n validates at each connection. Type error → workflow
stops BEFORE processing, not after.

Example validation:

```
σ output → another σ input:
  σ expects ℝ intensity, got ObjectID ∈ ℕ
  → BLOCK: "Type error: segmenting a label image"

μ output → σ input:
  σ expects ══ spatial, got ── table
  → BLOCK: "Type error: cannot segment a table"
```

### Dimensionality routing

```
Input stave type         n8n routes to
────────────────────────────────────────────
≡≡ (3D volume)          Cellpose3D, regionprops_3D, marching_cubes
══ + t (time-lapse)     σ per frame, then tracking
══ (single 2D)          standard pipeline
── (table only)         τ-chain, skip all image steps
```

One IF node reading stroke count replaces pages of
user configuration.

### Channel-aware routing

```
Input channels           n8n action
────────────────────────────────────────────
DAPI present            use for nuclear σ
GFP present             use for μ intensity measurement
DAPI + GFP + mCherry    μ: colocalization(GFP, mCherry)
Count > 3               route to φ: spectral unmixing first
Count == 1              simple single-channel pipeline
```

### Complete type-driven workflow

```
ι: WatchFolder
──────────────
Columns: path
──────────────
text
──────────────

; ι: ReadAndType

File: 1
──────────────
(stave type attached)
──────────────

       &

StaveType: 1
──────────────
Columns: strokes, channels, hasTime, calibration
──────────────
varies
──────────────

; ⊂ ; τ: Route by strokes

══ (2D):                    ≡≡ (3D):
  τ: IF hasTime               σ: Cellpose3D
                                (diameter from cal)
  No time:     Has time:
  σ: Cellpose  σ per frame
  (diameter    τ: Link
   from cal)   tracking

; (all branches produce same output type)

; τ: IF channels has multiple markers

Yes:                         No:
μ: Coloc + ObjectMeas        μ: ObjectMeas

; (merge)

Records: 1..N
──────────────
Columns: (from whichever path)
──────────────
ℝ (in μm units from calibration)
──────────────

; ε: CSV
; ε: Slack("Processed: {strokes} data,
            {N} objects, tool={which σ}")
```

### What type-aware routing gives a facility

```
Without stave types:              With stave types:
───────────────────────           ───────────────────────
User configures every param       calculated from metadata
2D tool on 3D data = crash        routes to correct tool
Wrong channel = wrong results     selects DAPI for σ
Pixel vs μm confusion            auto unit conversion
Silent failures                   type errors caught early
One pipeline per experiment       one pipeline handles all
```

The stave type is the missing metadata layer that makes
n8n intelligent about image data. Without it, n8n is a
dumb HTTP caller. With it, n8n becomes a type-aware
routing engine.


---

## Composites as n8n Sub-workflows

Named composites map directly to n8n sub-workflows.
Each composite = one "Execute Workflow" node:

```
n8n main workflow:

┌────────────┐  ┌──────────────────┐  ┌──────────┐
│ WatchFolder│→ │ Execute Workflow: │→ │ Slack    │
│ (ι)        │  │ SegMeas          │  │ (ε)      │
└────────────┘  │ params:          │  └──────────┘
                │   Model=cyto2    │
                │   MinSize=50     │
                └──────────────────┘

n8n sub-workflow "SegMeas":

┌──────────────┐  ┌──────────┐  ┌──────────────┐
│ HTTP:         │→ │ HTTP:    │→ │ Return       │
│ Cellpose      │  │ Measure  │  │ results      │
│ (σ)           │  │ (μ)      │  │              │
└──────────────┘  └──────────┘  └──────────────┘
```

### Parameters as sub-workflow inputs

n8n sub-workflows accept input variables.
These ARE the × parameters:

```
Sub-workflow inputs:         Stave equivalent:
  Model: string              × Model
  MinSize: number            × MinSize
  Calibration: object        & Calibration (if shared index)
```

### Library of facility sub-workflows

```
Sub-workflow       Stave composite         Reused by
────────────────────────────────────────────────────────
Preprocess         φ ; φ                   every pipeline
SegMeas            σ ; μ                   batch, HCS, single
QualityGate        ⊂ ; τ ; τ              nightly batch
DoseResponse       τ ; τ ; τ ; τ           HCS analysis
InstrumentQC       ι ; μ ; ⊂ ; τ          weekly health check
AutoAnnotate       σ ; μ ; τ              OMERO ingest
```

Each sub-workflow is tested once, reused everywhere.
Change Cellpose version → update one sub-workflow →
all pipelines use the new version.

### Composites solve the n8n maintenance problem

```
Without composites:           With composites:
────────────────────          ────────────────────
12 workflows, each has        12 workflows call SegMeas
  their own σ+μ nodes         SegMeas defined once

Update cellpose model:        Update cellpose model:
  edit 12 workflows            edit 1 sub-workflow
  test 12 workflows            test 1, all inherit

Bug in measurement:           Bug in measurement:
  find which of 12 has it      fix SegMeas, done
```

Composites are not optional syntactic sugar.
They are the maintenance strategy.
