# 11 — Data Specification

### Purpose

This document defines exactly what information the application expects, how it's
represented, and how it connects to the scientist's workflow — so the frontend can turn a
prepared DGE result into an interactive exploration experience without needing to
understand how the original analysis was performed. The guiding principle: **the analysis
produces the evidence; the application organizes the evidence for exploration.**

### Data Boundary

The MVP works entirely from **pre-computed DGE results**. It does not process FASTQ files,
BAM files, raw count matrices, alignment files, sequencing QC files, or
differential-expression modeling inputs — it begins at `Completed DGE Analysis → Prepared
Result Dataset → Prototype Application`. This keeps the prototype focused on the
scientist's reasoning workflow rather than reproducing a computational pipeline.

### Core Data Objects

The prototype is organized around four data objects plus one piece of session state:

| Object | Purpose |
|---|---|
| Experiment | Biological and experimental context |
| Comparison | What two conditions are being compared |
| DGE Results | The quantitative evidence |
| Gene Annotations | Additional biological context (optional) |
| Exploration State | What the researcher is currently doing with the data |

### Experiment and Comparison Metadata

The experiment object provides context before the researcher starts exploring results —
not every field needs to be populated for every dataset, but the application should
distinguish *known*, *unavailable*, and *not applicable*, and should never invent missing
metadata:

```json
{
  "experiment_id": "EXP001",
  "name": "Treatment Response in Neuronal Cells",
  "organism": "Homo sapiens",
  "tissue": "Neuronal cells",
  "cell_type": "iPSC-derived neurons",
  "analysis_method": "DESeq2"
}
```

The comparison object defines the biological question the DGE result represents, and its
**direction must always be explicit** — "Treatment vs Control" and "Control vs Treatment"
produce opposite interpretations of the same log fold change, so the interface should
always display comparison direction rather than assume it:

```json
{
  "comparison_id": "CMP001",
  "name": "Treatment vs Control",
  "condition_a": "Treatment",
  "condition_b": "Control",
  "reference_condition": "Control"
}
```

### DGE Result Schema

Each row is one gene. Minimum schema:

| Field | Type | Required | Supports |
|---|---|---|---|
| `gene_id` | string | Yes | Unambiguous identification |
| `gene_symbol` | string | Yes | Human-readable results |
| `log2FC` | number | Yes | Direction and magnitude of change |
| `pvalue` | number | Yes | Unadjusted statistical evidence |
| `padj` | number | Yes | Multiple-testing-adjusted evidence used for significance filtering |
| `baseMean` | number | No | Expression-level context |
| `annotation` | string | No | Biological context, when available |

```json
{
  "gene_id": "ENSG00000141510",
  "gene_symbol": "GENE1",
  "log2FC": 2.14,
  "pvalue": 0.000004,
  "padj": 0.00008,
  "baseMean": 842.3,
  "annotation": "Example gene description"
}
```

No single field should be interpreted in isolation.

### Derived Fields and Thresholds

The application may compute convenience fields from the original results — these are
presentation aids, not new biological measurements:

```json
{ "direction": "up", "significance": "significant", "abs_log2FC": 2.14 }
```

`direction` follows directly from the sign of `log2FC` (positive → up, negative → down,
zero → unchanged); `abs_log2FC` is just its absolute value; `significance` depends on an
explicit threshold rather than a hardcoded rule.

Thresholds should live in application configuration, separate from the dataset itself:

```json
{ "thresholds": { "padj": 0.05, "absolute_log2FC": 1.0 } }
```

This lets the researcher explore a more permissive or restrictive subset — e.g. tightening
from `padj < 0.05` to `padj < 0.01` — without ever modifying the underlying DGE results.

### Gene Classification

The application can classify genes for filtering and visualization — e.g. splitting on
`padj < 0.05`, then on `|log2FC| ≥ 1` vs. `< 1`, then on direction (up/down) within the
strong-change group. This classification is an **application-level categorization**, not a
new statistical result, and should be presented that way.

### Missing Data and Validation

Missing values (`padj = null`, `baseMean = null`, `annotation = null`) must never silently
become `0` — a missing `padj` is not the same as `padj = 0`, and the UI needs to
communicate "unavailable" clearly rather than let it read as a strong result.

Before data enters the application, the preparation layer should verify that required
columns exist (`gene_id`, `gene_symbol`, `log2FC`, `pvalue`, `padj`), numeric fields
actually contain valid numbers, every record has a gene identifier, and statistical values
are sensible (`0 ≤ pvalue ≤ 1`, `0 ≤ padj ≤ 1`, `baseMean ≥ 0`). Invalid records get
flagged, not silently altered.

**Duplicate gene identifiers** should be detected during preparation, investigated, and
resolved according to the source analysis — with the resolution's provenance preserved.
The frontend should never arbitrarily pick between duplicate records; that's a biological
and statistical decision, and it doesn't belong in the presentation layer.

### The Application Dataset Contract

The frontend should receive one normalized structure rather than depending directly on the
original analysis output format — this becomes the application's primary data contract:

```json
{
  "experiment": { "id": "EXP001", "name": "Treatment Response in Neuronal Cells",
                   "organism": "Homo sapiens", "tissue": "Neuronal cells" },
  "comparison": { "id": "CMP001", "name": "Treatment vs Control",
                   "condition_a": "Treatment", "condition_b": "Control",
                   "reference_condition": "Control" },
  "thresholds": { "padj": 0.05, "absolute_log2FC": 1 },
  "genes": []
}
```

### Exploration State, Selection, and Subsets

Not everything belongs in the dataset — some information represents what the *researcher
is currently doing*, and it should be modeled separately from what the experiment actually
produced:

```json
{
  "filters": { "padj": 0.05, "absolute_log2FC": 1, "direction": "up" },
  "selectedGenes": ["GENE1", "GENE2", "GENE3"],
  "activeView": "heatmap"
}
```

Selected genes should be represented by stable identifiers (Ensembl IDs) rather than gene
symbols alone, to avoid ambiguity, even though symbols remain the readable display label.

A **subset** — a temporary gene collection created during exploration — should preserve
the criteria that produced it, not just the resulting gene list:

```json
{
  "subset": {
    "name": "Strongly Upregulated",
    "gene_ids": ["ENSG00000141510", "ENSG00000123456"],
    "created_from": { "padj": 0.05, "absolute_log2FC": 1 }
  }
}
```

This is what makes lightweight **provenance** possible — the application should always be
able to answer "why is this gene in this subset?" That matters now for basic trust, and
matters more if the prototype later supports saved analyses or AI-assisted interpretation.

### Visualization Data

Every visualization should read from the same underlying DGE records rather than keeping
its own copy — volcano plot, MA plot, gene table, heatmap, and gene detail panel should all
draw from one dataset, so components can't accidentally disagree with each other.

One specific caveat: a standard DGE result table doesn't necessarily contain enough
information for a meaningful expression heatmap, since heatmaps generally need
expression values across samples or conditions (a `gene × sample` matrix), not just the
per-gene summary statistics in the DGE table. If an appropriate expression matrix isn't
available, the MVP should use a different exploration view rather than fabricate
sample-level values just to populate a chart.

### Annotation and External Data

Gene annotations (description, gene type, pathways, functional categories) are an optional
enrichment layer — the core DGE workflow shouldn't require them, so the application keeps
working even when annotation sources differ between datasets. For the MVP, annotations
should come from the prepared dataset, a local lookup file, or a small mock layer, rather
than live external database calls — that keeps the prototype reproducible without adding a
service dependency this early. Live integration with sources like Ensembl or pathway
databases is a reasonable later step, not a V1 requirement.

### Data Flow

```
Original DGE Output → Data Preparation → Validation → Normalization →
Application Dataset → React State → Filters/Selections → Derived Subsets →
Visualizations → Gene Inspection
```

The original DGE output remains the source of truth throughout.

### Data Principles

1. **Preserve the source evidence.** Never overwrite original statistical values.
2. **Separate data from interaction state.** The experiment doesn't change because the
   user changes a filter.
3. **Make assumptions explicit.** Thresholds and classifications should be visible, not
   hidden inside the dataset.
4. **Don't fabricate missing biology.** If a dataset can't support a visualization or
   interpretation, the application should say so rather than invent it.
5. **Keep the frontend independent of the analysis implementation.** The application
   should consume a stable data contract whether the underlying DGE analysis came from
   DESeq2, edgeR, limma, or something else.

### Definition of Done

This specification is complete when it answers: *"If I handed this prototype a DGE result,
exactly what does it need in order to work?"* For the MVP: experiment metadata + comparison
metadata + a validated DGE result table + optional gene annotations + an optional
expression matrix for additional visualizations.

The open question is no longer *what data do we need* — it's *how should the application
itself be structured around this data?* That's the focus of the next document.

**Next artifact:** `12_application_architecture.md`
