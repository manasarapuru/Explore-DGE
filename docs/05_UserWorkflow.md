# 05 — User Workflow

### Purpose

This document models how a scientist might interact with a DGE result after the upstream
RNA-seq analysis has already been completed. The starting point is a DGE results table
containing ~22,000 genes from the dexamethasone-vs-untreated comparison.

```
Receive DGE results → Orient to the experiment → Understand the overall response →
Notice potentially informative patterns → Explore a pattern → Narrow to a subset
of genes → Investigate individual genes → Add biological context →
Decide what deserves follow-up
```

The purpose is to understand what information a scientist needs at each stage and where an
application could reduce friction. The application is not assumed to make the scientific
decision for the user — it should help the scientist move efficiently from a large
analytical output toward observations worth investigating.

---

## 1. Receive DGE Results

**Scientist question:** *"What am I looking at?"*

The scientist receives the output of a differential expression analysis (~22,000 genes,
with `baseMean`, `log2FoldChange`, `lfcSE`, `pvalue`, `padj`). At this point they have the
result but may not yet have a useful mental model of what it contains. The first task isn't
to find a gene — it's to establish the context of the experiment.

- **Data needed:** gene identifiers, experimental comparison, treatment/control conditions,
  number of genes tested, DGE measurements, experiment metadata.
- **Current friction:** a DGE output is often just a large table that doesn't explain what
  the experiment showed overall, how large the response was, how many genes changed
  substantially, how strong the statistical evidence was, or where the most informative
  regions of the result may be.
- **Product opportunity:** provide enough experiment-level context that the scientist can
  immediately understand *"this is the result I am looking at, and this is what the
  experiment was testing"* — before asking them to interpret individual results.

## 2. Orient to the Experiment

**Scientist question:** *"What happened overall?"*

Understanding the overall transcriptional response to dexamethasone requires: number of
genes analyzed, distribution of log2 fold changes, positive-vs-negative counts, `padj`
distribution, expression-level distribution, and overall response magnitude.

- **Current friction:** ~22,000 rows don't immediately communicate the shape of the
  experiment's response; the scientist must manually summarize or visualize the data.
- **Product opportunity:** an experiment-level overview — response direction, magnitude
  distribution, statistical evidence distribution, counts of strongly changing / strongly
  supported genes, notable dataset characteristics. The purpose is not to decide what's
  important — it's to give the scientist a mental map before they explore.

## 3. Understand the Overall Response

**Scientist question:** *"How strong and widespread is the response?"*

Was the response broad or limited, mostly small changes or a few large ones, mostly
increases or decreases? This requires looking across `log2FoldChange` (magnitude/direction),
`padj` (evidence), `baseMean` (expression context), and `lfcSE` (uncertainty) together —
there is no single measurement that answers *"how important is this response?"*

- **Current friction:** the answers exist in the data but are distributed across thousands
  of rows and several columns.
- **Product opportunity:** present the major dimensions together — effect-size
  distribution, statistical-evidence distribution, expression distribution, and the
  relationship between effect size and statistical evidence.

## 4. Notice Potentially Informative Patterns

**Scientist question:** *"What stands out?"*

Examples: unusually large positive/negative changes, strong statistical evidence,
combinations of effect + evidence, groups of genes in similar regions of the response
space, highly expressed genes with unexpected changes, interesting results with unusual
uncertainty. **The scientist may not know the gene names in advance** — they may begin with
a pattern rather than a specific gene.

- **Current friction:** manually inspecting plots, sorting tables, adjusting thresholds,
  repeatedly comparing columns — discovery becomes dependent on knowing exactly what to
  look for.
- **Product opportunity:** surface potentially informative patterns without presenting them
  as definitive biological conclusions — e.g. *"A small group of genes shows large negative
  expression changes with strong adjusted statistical evidence"* rather than *"This is what
  you should conclude."* These become entry points for exploration.

## 5. Explore a Pattern

**Scientist question:** *"Why is this group interesting?"*

The scientist moves from the experiment-level view toward a subset — e.g. strongly
down/upregulated genes, statistically strong results, a specific combination of effect size
and evidence, or a region of a visualization.

- **Current friction:** reproducing a relationship between measurements (*"show me genes
  with substantial changes, but only where the statistical evidence is strong"*) often
  means switching between plots, filters, spreadsheets, and scripts.
- **Product opportunity:** interactive exploration of the result space, where the scientist
  changes how a pattern is defined and immediately sees which genes belong to that region.
  **The scientist should be able to explore the dimensions of the data rather than being
  forced into one predetermined definition of importance.**

## 6. Narrow to a Subset of Genes

**Scientist question:** *"Which genes belong to this pattern?"*

```
22,008 genes → potentially informative pattern → several hundred genes → dozens of genes
```

- **Data needed:** gene identifiers, effect sizes, adjusted p-values, expression levels,
  standard errors, ranking/filtering criteria.
- **Current friction:** repeatedly exporting, filtering, sorting, or manipulating tables to
  preserve a subset; the reasoning behind the subset becomes hard to track.
- **Product opportunity:** make subsets persistent and inspectable — why a gene entered the
  subset, which criteria produced it, how many genes are included, how they compare to one
  another (e.g. *"Large negative response AND strong adjusted evidence → 74 genes"*).

## 7. Investigate Individual Genes

**Scientist question:** *"What is happening with this gene?"*

The scientist wants: how large the response is, direction, statistical strength, expression
signal, estimate precision, and how the gene compares to others in the experiment.

- **Data needed:** gene identifier, `baseMean`, `log2FoldChange`, `lfcSE`, `pvalue`,
  `padj`, position/ranking within the overall result.
- **Current friction:** statistical information may be in a spreadsheet while biological
  information about the gene lives elsewhere (genome browsers, gene databases, pathway
  resources, literature, internal knowledge bases).
- **Product opportunity:** a focused gene-level view that preserves statistical context and
  lets the scientist return to the pattern or subset the gene was selected from.

## 8. Add Biological Context

**Scientist question:** *"What might this mean biologically?"*

The DGE result describes transcriptional changes, not what they mean biologically. Useful
additional information: gene function, pathway membership, biological processes, disease
associations, drug targets, literature, prior experimental evidence, other omics data.

- **Current friction:** this information is scattered across resources — identify a gene,
  search a database, inspect pathway info, search literature, compare against the DGE
  result, repeat for additional genes. A fragmented investigation workflow.
- **Product opportunity:** a biological context layer connected to the DGE result (DGE
  result → gene → annotation → pathway → literature → drug/disease context) — with external
  information kept **clearly separated** from the original experimental evidence: *what
  this experiment measured* vs. *what external biological knowledge suggests.*

## 9. Decide What Deserves Follow-Up

**Scientist question:** *"What should I investigate next?"*

Possible follow-ups: investigate a gene experimentally, examine a pathway, compare with
another dataset, validate a transcriptional response, check whether a target is already
known, investigate a mechanism, compare with another treatment, examine patient/disease
data, design a follow-up experiment.

- **Current friction:** the decision is fragmented across statistical evidence, effect
  size, gene function, pathway information, literature, drug information, and existing
  knowledge — assembled manually before deciding whether a result is worth pursuing.
- **Product opportunity:** a structured transition from observation to investigation. Not
  *"This gene is the answer"* but *"Here is why this result may be worth investigating, and
  here is the evidence and context you can use to decide what to do next."*

## 10. Complete Scientist Workflow

```
Receive DGE Results
   → Orient to Experiment            ("What am I looking at?")
   → Understand Response             ("What happened overall?")
   → Notice Patterns                 ("What stands out?")
   → Explore Pattern                 ("Why is this interesting?")
   → Narrow to Subset                ("Which genes are here?")
   → Investigate Gene                ("What is happening here?")
   → Add Biological Context          ("What might this mean?")
   → Decide Follow-Up                ("What should I explore?")
```

## 11. Workflow Summary

| Stage | Scientist question | Data needed | Current friction | Product opportunity |
|---|---|---|---|---|
| Orientation | What happened overall? | DGE distribution | 22k rows don't provide context | Overview |
| Pattern discovery | What stands out? | Effect, significance, expression | Scientist must manually find patterns | Surface patterns |
| Exploration | Why is this group interesting? | Multiple measurements | Hard to compare dimensions | Interactive exploration |
| Drill-down | What is happening with this gene? | Gene-level statistics | Context scattered across outputs | Gene detail |
| Interpretation | What might this mean biologically? | Annotation, pathway, literature | Requires external investigation | Context layer |
| Follow-up | What should I investigate next? | Evidence + context | Decision is fragmented | Guided next steps |

## 12. Key Product Principle

The application should **not** be designed around the assumption that the scientist already
knows which gene they want to investigate. Instead:

```
Observation → Pattern → Exploration → Subset → Gene → Context → Follow-up
```

The central workflow problem: **how do we help a scientist move from a large DGE result to
an evidence-backed area of investigation without requiring them to already know what to
search for?** This becomes the foundation for the application design.

**Next artifact:** `06_product_requirements.md`
