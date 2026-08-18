# 06 — Product Requirements

### Purpose

This document translates the scientist workflow into product requirements — structured
around the decisions already made, rather than as a generic software requirements doc.

The application should help a scientist move from a large DGE result (~22,000 genes;
`baseMean`, `log2FoldChange`, `lfcSE`, `pvalue`, `padj`) toward genes, patterns, and
biological questions worth investigating — progressively, rather than requiring them to
begin with a specific gene or predefined hypothesis:

```
DGE Results → Orient → Understand the response → Notice patterns → Explore →
Narrow → Investigate genes → Add context → Decide what deserves follow-up
```

---

## 1. Product Goal

**Primary goal:** help a scientist efficiently navigate a large DGE result and identify
observations worth further investigation — answering *"What is happening in this
experiment, what stands out, and what should I investigate further?"*

**Not** intended to automatically determine which genes are biologically important, which
are therapeutic targets, which pathways are responsible for the response, whether a finding
is clinically meaningful, or what experiment should ultimately be performed. Those remain
scientific judgments.

## 2. Core Product Principle

The scientist should not have to know the answer before using the application:

```
Large result → Overview → Potentially informative pattern → Interactive exploration →
Subset of genes → Individual gene → Biological context → Follow-up question
```

Each stage should preserve the context from the previous stage — if a scientist selects a
gene from a particular pattern, the application should make it possible to understand *"why
am I looking at this gene?"* rather than presenting it as an isolated row.

## 3. Requirement — Experiment Orientation

**User need:** understand what experiment produced the results.

**Provide:** treatment condition, comparison condition, biological system, number of genes
analyzed, relevant experiment metadata, description of the DGE analysis. For this case
study: system = human airway smooth muscle cells; treatment = dexamethasone; comparison =
untreated; analysis = differential gene expression.

**Why it matters:** `log2FC = -2` doesn't mean much without knowing what was treated, what
it was compared against, and what biological system was measured.

**Product boundary:** display experimental context that is actually available; don't invent
missing experimental metadata.

## 4. Requirement — Overall Response Overview

**User need:** *"What happened overall?"*

**Provide:** distribution of `log2FoldChange`, positive vs. negative responses, `padj`
distribution, expression distribution, missing-value counts, gene counts in major response
regions — e.g. *"22,008 genes analyzed; 11,441 positive, 10,567 negative; most genes show
small changes, a smaller subset shows larger changes."*

**Product boundary:** the overview should describe the data; it should not automatically
translate "large response" into "important biological response."

## 5. Requirement — Surface Potentially Informative Patterns

**User need:** *"What stands out?"*

**Provide:** unusually large positive/negative effects, strong adjusted evidence,
combinations of effect + evidence, unusual relationships between expression and response,
potentially uncertain large effects — e.g. *"74 genes show log2FC > 2"*, *"138 genes show
|log2FC| > 2 and padj < 0.05."*

**Principle:** present as *potentially informative observations*, not *scientifically
important findings*, and let the scientist decide whether a region deserves investigation.

**Product boundary:** no unexplained "biological importance" score; if ranking is
introduced, criteria must be visible.

## 6. Requirement — Multidimensional Exploration

**User need:** *"Why is this group interesting?"*

| Dimension | Field |
|---|---|
| Effect | `log2FoldChange` |
| Statistical evidence | `padj` |
| Raw statistical evidence | `pvalue` |
| Expression | `baseMean` |
| Uncertainty | `lfcSE` |

**Allow:** filter, sort, select, compare, zoom into regions, change thresholds, inspect the
resulting gene subset — e.g. moving from "large negative effect + strong adjusted evidence"
to "moderate negative effect + strong adjusted evidence + higher expression."

**Principle:** the scientist controls the definition of the subset; the application
provides the tools for exploration.

## 7. Requirement — Preserve Statistical Meaning

Each measurement should remain interpretable rather than hidden behind simplified labels:

- `baseMean` — average normalized expression signal; expression context, not biological
  importance.
- `log2FoldChange` — magnitude/direction of expression change.
- `lfcSE` — uncertainty around the estimate; large effects should be considered alongside
  uncertainty.
- `pvalue` — statistical evidence against the null hypothesis; not biological importance.
- `padj` — adjusted statistical evidence, particularly important at genome scale.

**Requirement:** displayed values should come with enough explanation that a scientist
doesn't have to remember what each statistic represents.

## 8. Requirement — Dynamic Gene Subsets

**User need:** *"Which genes belong to this pattern?"*

**Provide:** gene ID, relevant statistics, ranking, selection criteria, subset size — e.g.
`|log2FC| > 2 AND padj < 0.05` → 138 genes. The scientist should be able to inspect those
genes without losing the definition of the subset.

**Requirement:** maintain the relationship Pattern → Criteria → Subset for traceability.

## 9. Requirement — Gene-Level Investigation

**User need:** *"What is happening with this gene?"*

**Provide:** gene identifier, expression level, direction, standard error, `pvalue`,
`padj`, rank within subset and within overall result, and *how the scientist arrived at
this gene* (e.g. "Selected from: large negative responses + padj < 0.05 → gene
ENSG00000152583").

## 10. Requirement — Biological Context

**User need:** *"What might this mean biologically?"*

**Potential context:** gene descriptions, gene ontology, pathways, biological processes,
disease associations, drug-target information, literature, internal experimental datasets.

**Architecture principle:** biological context must remain visually distinguishable from
experimental evidence — e.g. `EXPERIMENTAL RESULT` (log2FC, padj, baseMean, lfcSE) vs.
`BIOLOGICAL CONTEXT` (gene function, pathways, literature, disease associations, drug
information). Important for scientific trust.

## 11. Requirement — Follow-Up Support

**User need:** *"What should I investigate next?"*

**Possible next steps:** investigate a pathway, investigate a group of genes, compare
against another dataset, examine literature, investigate a known drug target, identify
validation experiments, compare with another treatment.

**Principle:** provide evidence and options, not prescribe the scientific decision — e.g.
system says *"This gene has a large treatment-associated expression change and strong
statistical evidence,"* not *"This gene should be your next drug target."*

## 12. Requirement — Traceability

For every surfaced pattern or subset, the application should make it possible to determine
what was selected, which measurements were used, which thresholds were applied, and which
genes were included. This becomes particularly important if AI-generated summaries are
eventually introduced — a scientist should be able to move from an AI-generated observation
→ supporting genes → underlying statistics → original DGE result.

## 13. Requirement — Scientist-Controlled Exploration

Scientists should be able to change thresholds, compare regions, inspect individual genes,
return to previous subsets, change the dimension being explored, and decide what deserves
attention. The system should behave more like a **scientific exploration partner** than an
**automated scientific decision-maker**.

## 14. Requirement — Handle Missing Data Transparently

The dataset contains missing `padj` values. The application must distinguish *missing
value* from *weak statistical evidence* from *not significant according to a selected
criterion* — these are not equivalent. The interface should avoid silently converting
missing measurements into negative conclusions.

## 15. Initial Product Requirements Summary

| Workflow stage | User need | System responsibility | Required data | Product boundary |
|---|---|---|---|---|
| Orientation | What am I looking at? | Establish experiment context | Metadata + DGE result | Don't invent context |
| Overall response | What happened overall? | Summarize response landscape | DGE distributions | Describe, don't interpret |
| Pattern discovery | What stands out? | Surface potentially informative patterns | Effect + significance + expression | Don't label biological importance |
| Exploration | Why is this interesting? | Enable multidimensional exploration | All major statistics | Scientist controls criteria |
| Subset | Which genes belong here? | Create persistent subsets | Gene IDs + statistics | Preserve selection logic |
| Gene drill-down | What is happening with this gene? | Show complete gene-level evidence | All gene statistics | Don't isolate one statistic |
| Biological context | What might this mean? | Connect to external knowledge | Annotations + pathways + literature | Separate evidence from context |
| Follow-up | What should I investigate next? | Organize evidence and options | Experimental + contextual evidence | Don't make scientific decisions |

## 16. MVP Product Boundary

```
Experiment overview → Response overview → Pattern visualization →
Interactive exploration → Gene subset → Gene detail
```

Biological context (Gene detail → Biological context → Follow-up support) can be added
after the core exploration workflow is working — keeping the initial product grounded in
information that actually exists in the dataset.

## 17. What We Are Not Building Yet

The initial application will not attempt to: perform RNA-seq alignment, perform
normalization, run DESeq2, generate synthetic biological data, determine drug targets
automatically, diagnose disease, predict clinical outcomes, declare genes biologically
important, replace pathway analysis, or replace scientific judgment.

Focus: **DGE output → Understanding → Exploration → Investigation.**

## 18. Product Hypothesis

> A DGE result contains enough statistical information to support much more than a static
> table, but the information needs to be organized around how scientists actually explore
> results.

```
"I have 22,000 genes." → "I understand the overall response." →
"These patterns stand out." → "This subset is worth examining." →
"This gene is interesting for these specific reasons." →
"Here is the biological context I need to decide whether it deserves follow-up."
```

The product acts as a bridge between analytical output → scientific exploration →
scientific decision-making, without claiming ownership of the final scientific decision.

**Next artifact:** `07_data_to_feature_mapping.md` — for each requirement: what can we
actually build from the columns we have, what additional data would be needed, and what
would have to be simulated or deferred? This distinguishes the client's desired workflow
from what their current data can actually support.
