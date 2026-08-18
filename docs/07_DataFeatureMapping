# 07 — Data to Feature Mapping

Maps the product requirements to the data currently available, to determine: what the
current DGE dataset can support; what can be derived from existing measurements; what
requires additional biological/experimental data; what should be deferred rather than
fabricated; and where an FDE would need to go back to the client with questions.

The key FDE question: **given what the client has provided, what can we actually support,
what requires additional data, and what should we explicitly defer?**

---

## 1. Available Dataset

~22,000 gene-level DGE results.

**Available:** Gene ID, `baseMean`, `log2FoldChange`, `lfcSE`, `pvalue`, `padj`.

**Not available in current dataset:** gene symbols, gene descriptions, pathway
annotations, Gene Ontology terms, literature, drug-target annotations, patient data,
additional treatment conditions, additional compounds, dose information, treatment
duration, individual sample-level expression data, biological replicates, raw RNA-seq
reads.

This distinction matters because not every desired product capability can be supported by
the current dataset.

## 2. Feature Mapping Philosophy

- **Type A — Directly supported:** implementable from data already present (e.g. "show
  genes with |log2FC| > 2" — no additional data required).
- **Type B — Derived:** not a column, but calculable from available data (e.g. "number of
  genes with padj < 0.05").
- **Type C — Requires additional data:** cannot be responsibly implemented from the
  current dataset alone (e.g. "show pathways associated with selected genes" — the DGE
  result has no pathway information).

## 3. Experiment Orientation

**Current support:** partial. The DGE file's five statistical columns don't carry
experimental metadata; the comparison (dexamethasone vs. untreated, human airway smooth
muscle cells) is known from the source experiment but not represented in the table itself.

**Product implication:** an experiment metadata layer, separate from the DGE table. For
this prototype, metadata comes from the documented source experiment.

**FDE question for a real deployment:** "Where does the authoritative experimental
metadata live, and how should it be associated with each DGE result?"

## 4. Overall Response Overview

**Current support:** fully supported (`log2FoldChange`, `pvalue`, `padj`, `baseMean`,
`lfcSE`). Can derive: gene counts, positive/negative changes, effect-size distribution,
`padj` distribution, expression distribution, uncertainty distribution, threshold
proportions, missing-value counts.

**Example:** 22,008 genes — 11,441 positive log2FC, 10,567 negative log2FC; 138 genes with
|log2FC| > 2 and padj < 0.05.

**Product implication:** buildable without additional biological databases.

## 5. Pattern Discovery

**Current support:** fully supported. Candidate patterns: large effects (`|log2FC| >
threshold`), strong statistical evidence (`padj < threshold`), combined evidence, high
expression + response, high-effect/high-uncertainty observations.

**Product implication:** the system can surface candidate patterns without claiming
biological importance.

## 6. Multidimensional Exploration

**Current support:** fully supported across effect size (`log2FoldChange`), statistical
evidence (`padj`, `pvalue`), expression (`baseMean`), and uncertainty (`lfcSE`).

**Product implication:** the interface should avoid reducing DGE interpretation to a
single ranking — different measurements answer different questions.

## 7. Dynamic Gene Subsets

**Current support:** fully supported (e.g. `log2FC > 2 AND padj < 0.05`, or `log2FC < -1
AND padj < 0.01 AND baseMean > 100`).

**Product implication:** every subset should retain its filtering criteria (e.g. "138 genes
selected — criteria: |log2FC| > 2, padj < 0.05"), for reproducibility and clarity.

## 8. Gene-Level Investigation

**Current support:** supported statistically, limited biologically. Can provide gene ID,
`baseMean`, `log2FC`, `lfcSE`, `pvalue`, `padj`, and can calculate ranking, percentile,
subset position, distance from thresholds.

**Cannot currently provide (without an external source):** gene function, pathway, disease
association, drug-target status, literature evidence.

## 9. Biological Context

**Current support:** not supported by the DGE dataset alone.

**Additional data required:** e.g. Ensembl, NCBI Gene, Gene Ontology, Reactome, MSigDB,
Open Targets, PubMed, drug-target databases.

**Product implication:** biological context should be a separate data layer — DGE dataset +
gene annotation + pathway information + literature → contextualized investigation.

**FDE questions for a real client:** "Which knowledge sources are approved for scientific
interpretation?" and "Should the system use public databases, internal knowledge bases, or
both?"

## 10. Follow-Up Support

**Current support:** partial. DGE data provides effect magnitude, statistical evidence,
expression level, and uncertainty — but follow-up decisions need biological function,
pathway, prior evidence, drug relevance, and experimental feasibility, none of which the
current dataset has.

**Product implication:** the MVP should support evidence organization rather than
recommendation generation.

## 11. Traceability

**Current support:** fully supported for data-derived patterns (e.g. "Large negative
effects: log2FC < -2 AND padj < 0.05 → 109 genes"). The application can preserve filter
criteria, source columns, selected genes, and ranking logic.

**Product implication:** every automated observation should be explainable.

## 12. Missing Data

The dataset has missing values for some adjusted p-values. The application must
distinguish *missing* from *not significant* from *significant*, and must not silently
convert `NaN` into `padj = 1` unless there's a scientifically justified and explicitly
documented reason.

## 13. What We Can Build Now

Based entirely on the current dataset: experiment overview (dexamethasone vs. untreated,
human airway smooth muscle cells); response overview (gene count, effect/significance/
expression distributions); interactive response landscape (log2FC vs. statistical
evidence); pattern surfacing (large positive/negative effects, strong statistical
evidence, combined effect + evidence); interactive filtering (effect, significance,
expression, uncertainty); gene table (gene, baseMean, log2FC, lfcSE, pvalue, padj); and a
gene detail view.

## 14. What Requires Additional Data

| Capability | Additional requirement |
|---|---|
| Gene symbols | Gene annotation |
| Gene descriptions | Gene annotation |
| Pathways | Pathway database |
| GO terms | GO annotation |
| Literature | Literature source |
| Drug targets | Drug-target database |
| Disease associations | Disease database |
| Patient comparison | Patient-level dataset |
| Multiple drug comparison | DGE results for additional treatments |
| Dose response | Dose-associated experimental data |
| Time-course analysis | Multiple time points |
| Sample-level QC | Sample-level expression/count data |

## 15. What This Means for the MVP

The current dataset is sufficient to prototype the central FDE problem: **how can we make a
large DGE result easier for a scientist to orient to, explore, and narrow down?** The MVP
does not need to solve the entire biological interpretation problem:

```
22,000 genes → understand the response → see what stands out →
explore dimensions → define a subset → investigate genes
```

Biological context is added as a second layer.

## 16. FDE Perspective: Data Gaps Become Client Questions

The absence of data isn't just a technical limitation — it creates discovery questions:

- **Experimental context:** What metadata accompanies each DGE result? How are treatment,
  control, cell type, dose, and time point represented?
- **Biological annotation:** What annotation sources do your scientists currently use? Are
  there approved internal knowledge bases?
- **Literature:** Should scientists be able to move from a gene directly into literature
  evidence?
- **Drug discovery:** Are scientists trying to identify drug-responsive genes, compare
  compounds, validate targets, or understand mechanisms?
- **Multi-experiment analysis:** Will scientists eventually need to compare multiple
  experiments?
- **Internal data:** Do you have additional DGE datasets that should eventually be analyzed
  alongside public datasets?

These determine what a production system would eventually need.

## 17. Data → Product Boundary

```
CURRENT DATA
DGE Statistics (baseMean, log2FoldChange, lfcSE, pvalue, padj)
        ↓
DATA EXPLORATION
Overview | Patterns | Filtering
        ↓
Gene subsets
        ↓
Gene detail
        ↓
FUTURE LAYER
Annotation | Pathways | Literature | Drug data
```

A clear boundary between what the current evidence tells us and what additional information
would be needed to interpret that evidence.

## 18. Key FDE Takeaway

The important finding isn't just that the dataset has five useful columns — it's that the
dataset naturally supports a progressive exploration workflow:

```
baseMean       → How much expression signal is present?
log2FoldChange → How large and in which direction is the change?
lfcSE          → How uncertain is that estimated change?
pvalue         → How much statistical evidence before adjustment?
padj           → How strong is the evidence after multiple-testing correction?
```

No single measurement answers *"Is this gene important?"* — the scientist needs to examine
the measurements together, then bring in biological context. That is the product
opportunity:

```
Statistical evidence → Pattern → Gene subset → Gene → Biological context →
Scientific judgment
```

The application supports the scientist's reasoning without replacing it.

**FDE distinction demonstrated by this document:** the client asks for a capability → we
inspect the available data → we determine what is actually possible → we identify the
missing data → those gaps become discovery questions. More representative of an FDE
workflow than "we have a DGE CSV, so let's make a dashboard."

**Next artifact:** `08_application_user_flow.md` — mapping what the scientist would
actually *see and do*, from the moment they open the application.
