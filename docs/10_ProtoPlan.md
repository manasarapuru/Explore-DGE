# 10 — Prototype Plan

### Purpose

The prototype is a small, working application that demonstrates how a researcher could
move from a DGE result to an interpretable biological pattern. It is not a bioinformatics
platform and doesn't replace established analysis tools — it exists to make one interaction
model tangible and testable:

> **Receive results → orient → understand the response → notice a pattern → explore →
> narrow → interpret.**

The central question the prototype needs to answer: **can a DGE result be made easier to
explore without hiding the underlying scientific information?** A successful build lets a
user understand what experiment they're looking at, assess the overall response, spot a
potentially interesting pattern, explore it visually, narrow to a gene subset, inspect
individual genes and their supporting evidence, and trace any view back to the original
result. Depth of reasoning matters more here than feature count.

### MVP Boundary

The MVP covers a single experiment and a single primary comparison, built on a prepared DGE
result table: gene-level statistics, interactive filtering, core DGE visualizations, gene
selection, pattern exploration, gene-level inspection, basic biological annotation where
available, and a clear connection between every visualization and the genes behind it.

It deliberately excludes raw FASTQ processing, alignment, quantification, differential
expression modeling, batch correction, experimental design configuration,
multi-project management, authentication, cloud infrastructure, automated biological
conclusions, autonomous AI analysis, production-grade ingestion, and complex statistical
modeling. These may become real capabilities later, but none of them are needed to
demonstrate the core interaction.

### Design Philosophy

The application should behave like a scientific exploration environment, not a dashboard —
a dashboard answers *"what are the numbers?"*; this needs to answer *"what should I look at
next?"* That means supporting movement across levels of abstraction rather than forcing a
single linear path:

| Level | Question |
|---|---|
| 1 — Experiment | What am I looking at? |
| 2 — Global response | What happened overall? |
| 3 — Pattern | Is there something interesting here? |
| 4 — Subset | Which genes are contributing to that pattern? |
| 5 — Gene | What do I know about this particular gene? |
| 6 — Interpretation | What hypothesis might this pattern support? |

The user should be able to move backward and forward between these levels freely.

### Architecture

For the first prototype, the differential expression analysis itself happens **before** the
application runs — the app consumes a prepared result rather than computing anything
dynamically, which keeps the build focused on the interaction rather than the pipeline.

```
DGE Result Dataset
        ↓
Data Preparation Layer
        ↓
Normalized Application Data
        ↓
                React Frontend
        ↓                ↓                ↓
  Experiment      Global View       Exploration
   Context        of Response          Views
        ↓                ↓                ↓
                Gene Selection
                       ↓
                Gene Inspection
```

### Data Model

A minimum DGE record needs:

| Field | Purpose |
|---|---|
| `gene_id` | Stable identifier |
| `gene_symbol` | Human-readable gene name |
| `log2FC` | Direction and magnitude of expression change |
| `pvalue` | Statistical significance |
| `padj` | Multiple-testing-adjusted significance |
| `baseMean` | Average expression level |
| `annotation` | Optional biological context |

Experiment-level metadata (experiment, comparison, control, treatment, organism, tissue,
timepoint, analysis method) should be modeled separately from gene-level results, not
folded into the same table.

### Data Preparation

Before the interface gets built, the DGE results need to be converted into an
application-friendly format: load the original result, standardize column names, remove or
flag invalid records, ensure numeric fields are numeric, identify missing values, add
derived fields (e.g. `direction`, `significance`, `abs_log2FC`, `gene_label` — computed
from the raw statistics, such as `log2FC > 0 → upregulated`), preserve the original
statistics untouched, and export a clean JSON or CSV. Derived fields exist for interaction
and visualization only — they never replace the original statistical values.

### Screens and Interactions

**Screen 1 — Experiment Orientation.** Establishes context before any analysis begins: what
experiment, what comparison, what biological material, what conditions, what analysis
method. This screen should stay lightweight — its job is orientation, not analysis.

**Screen 2 — Overall Response.** Answers *"what happened globally?"* through summary
statistics (genes tested, significant, up/downregulated), an interactive volcano plot
(effect size, significance, direction, and unusually strong signals, with gene selection
exposing underlying statistics), and a fold-change distribution view showing whether the
response is broadly shifted, balanced, or concentrated in a small number of genes.

**Screen 3 — Pattern Exploration.** Once the researcher understands the overall response,
the key interaction becomes *"I see something interesting — let me investigate it."*
Candidate views include a ranked gene list, MA plot, heatmap, selected-gene table,
expression distribution, and annotation grouping — but the prototype shouldn't show all of
these simultaneously. It should make the next useful step obvious rather than exhaustive.

**Filtering** is one of the most important interactions in the prototype. Researchers should
be able to narrow the result set by adjusted p-value, log2 fold change, expression level,
direction, or gene category (e.g. `padj < 0.05 AND |log2FC| > 1`), with gene counts, plots,
tables, and subsets all updating immediately. Filtering should feel like exploration, not a
destructive operation — the original result set should always remain recoverable.

**Selection** should carry across views: a gene set selected from the volcano plot should
remain available to inspect, compare, filter, or annotate elsewhere in the app, without
requiring the user to manually transfer gene identifiers between screens.

**Gene-level inspection**, opened by selecting a gene, should show identity (symbol, ID),
DGE evidence (log2FC, p-value, padj, baseMean), direction, and biological context where
available (functional annotation, pathway membership, description, relevant categories).
The goal isn't a definitive biological interpretation — it's enough context for the
researcher to decide whether the gene deserves further investigation.

**Pattern ↔ gene interaction should work in both directions.** Pattern → genes: an
interesting pattern leads to a selected subset, which leads to inspecting contributing
genes. Gene → pattern: an interesting gene leads to asking where it sits in the overall
response, and what else behaves similarly. Without the second direction, the interface
collapses into a gene lookup tool rather than an exploration environment.

### State and Traceability

The prototype needs to preserve state across views — current experiment, active filters,
selected genes, current visualization, current subset — so that switching screens doesn't
silently drop the researcher's context (e.g. `padj < 0.05, |log2FC| > 1` with three selected
genes should still be intact after switching from the volcano plot to a heatmap).

Traceability is a related, non-negotiable requirement: if the application states "2,146
genes are significant," the researcher needs to be able to see what threshold and statistic
produced that number, which comparison it came from, and which genes are included. Any gene
appearing in a visualization should be traceable back through: visualization → gene → DGE
statistics → original result. This is what makes the tool trustworthy rather than opaque.

Not everything needs to be dynamic, though — experiment description, method description,
explanatory text, dataset metadata, and basic gene annotations can stay static, while
filters, the volcano plot, gene selection, tables, heatmaps, subset creation, and view
switching need to be interactive. Keeping that split explicit keeps the build manageable.

### Visualization Requirements

| Visualization | Question it supports |
|---|---|
| Volcano plot | What genes show strong/significant changes? |
| MA plot | How does effect size relate to expression level? |
| Ranked table | Which genes stand out numerically? |
| Heatmap | Do selected genes show coherent patterns? |
| Gene panel | What evidence supports this gene? |

Every visualization should answer a recognizable question — the prototype should avoid
adding one just because it's conventional in bioinformatics tooling.

### Suggested Build Order

| Phase | Focus |
|---|---|
| 1 — Data | Prepare the DGE dataset, define the application schema, validate fields, build a mock/fixture dataset if needed |
| 2 — Application shell | Set up the React app, routing/view structure, experiment context, global layout |
| 3 — Orientation | Experiment context panel, comparison metadata, dataset summary |
| 4 — Global response | Summary statistics, volcano plot, gene hover/selection |
| 5 — Exploration | Filtering, ranked gene table, MA plot or equivalent |
| 6 — Gene inspection | Gene detail panel, connection to underlying statistics, biological annotation |
| 7 — Cross-view interaction | Preserve selections across views, connect filters to plots/tables, subset exploration |
| 8 — Refinement | Layout polish, loading/error states, labels, end-to-end walkthrough |

### What This Prototype Is Actually Testing

Not whether the visualization library works — the product hypothesis: **researchers move
through DGE results more effectively when the interface is organized around their
reasoning process rather than around isolated analysis outputs.** That means evaluating
orientation (can they tell what experiment they're viewing?), comprehension (do they
understand the overall response?), discovery (can they identify something worth
investigating?), exploration (can they narrow the result set without losing context?),
interpretation (can they inspect the evidence behind a gene or pattern?), and continuity
(can they move between global patterns, subsets, and individual genes without friction?).

### Definition of Done

The prototype is complete when a researcher can independently move through: *"I understand
the experiment" → "I understand the overall response" → "I noticed something interesting"
→ "I can explore that pattern" → "I narrowed it to these genes" → "I can inspect the
evidence for this gene" → "I can return to the broader response."*

It does not need to prove a biological hypothesis. It needs to demonstrate a working bridge
between data, exploration, evidence, and scientific reasoning.

### Future Expansion

Once the core interaction is validated, natural next steps include multiple experiments and
comparisons, saved gene sets, pathway analysis, external biological database integration,
cross-dataset comparison, interactive expression matrices, AI-assisted pattern discovery,
natural-language exploration, analysis provenance, reproducible sessions, cloud-hosted
datasets, and automated reporting. All of this comes after the core interaction is
validated — the first prototype should stay intentionally small.

### Closing Principle

The prototype isn't trying to answer every biological question — it's trying to make the
next question easier to ask. The core interaction — **understand → notice → explore →
narrow → inspect → recontextualize** — is the actual product being prototyped.
