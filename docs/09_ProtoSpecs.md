# 09 — UI / Prototype Specification

### Purpose

Translates the application user flow into a concrete interface. The prototype should let a
scientist move from a large DGE result toward potentially informative genes and patterns
without requiring them to begin with a specific gene or predefined hypothesis. Priorities:
orientation, pattern discovery, exploration, progressive narrowing, statistical context,
traceability, scientist-controlled interpretation.

---

## 1. Application Structure

Five primary views:

```
1. Experiment Overview → 2. Response Landscape → 3. Pattern Exploration →
4. Gene Set → 5. Gene Detail
```

A persistent nav element (`Overview > Response > Explore > Genes > Gene Detail`) keeps the
scientist oriented within the workflow.

## 2. Screen 1 — Experiment Overview

**Purpose:** *"What experiment am I looking at?"*

Shows: treatment, comparison, biological system, genes analyzed, and a short explanation of
what the result contains (effect = `log2FoldChange`, expression = `baseMean`, uncertainty =
`lfcSE`, evidence = `pvalue`/`padj`). Deliberately not chart-heavy — its job is orientation,
not analysis. The scientist should be able to answer: what treatment, compared with what, in
what system, how many genes, what kind of data.

## 3. Screen 2 — Response Landscape

**Purpose:** *"What happened overall?"* — the first analytical screen.

Layout, top to bottom:

- Header: experiment name, 22,008 genes, positive (11,441) vs. negative (10,567) counts
- Two-up: effect distribution (histogram) + statistical evidence distribution
- Expression distribution (baseMean)
- "Potentially informative regions" — pattern cards: Large positive, Large negative,
  Strong evidence, Large + significant

**Visual hierarchy:**
- Level 1 — how many genes, how is the response distributed
- Level 2 — where are potentially informative regions
- Level 3 — individual genes within those regions

The interface should avoid showing a giant gene table as the first analytical element.

### Pattern cards

```
┌──────────────────────┐   ┌──────────────────────┐
│ Large positive        │   │ Large negative        │
│ responses              │   │ responses              │
│ 74 genes               │   │ 109 genes              │
│ Explore →              │   │ Explore →              │
└──────────────────────┘   └──────────────────────┘
```

Wording stays neutral: "Large negative responses," not "Important downregulated genes";
"Strong statistical evidence," not "Significant genes worth pursuing." These are entry
points, not conclusions.

## 4. Screen 3 — Pattern Exploration

Example: the scientist selects "Large negative responses" (109 genes) and lands in an
exploration workspace with:

- Filters: `log2FC [< -2.0]`, `padj [< 0.05]`, `baseMean [Any]`
- Two-up: effect-vs-evidence plot + selection summary (gene count + current criteria)
- A "Selected Genes" table below (gene, log2FC, padj, baseMean, lfcSE)

**Interactive exploration:** the plot is an exploration surface — hover, select, zoom,
adjust thresholds, inspect the subset, sort the table. Changing `log2FC < -2.0` to `< -1.5`
updates the count live (109 → 214 genes).

**Statistical context:** selecting a gene surfaces its full stats (baseMean, log2FC, lfcSE,
pvalue, padj), with an info control that explains each measurement in place (e.g.
"`log2FoldChange` — Estimated magnitude and direction of expression change"; "`padj` —
Adjusted statistical evidence accounting for multiple testing") — without forcing the
scientist away from the analysis.

**Selection context:** whenever a gene is selected, the interface preserves why — e.g.
"Selected from: Large Negative Responses; Current criteria: log2FC < -2, padj < 0.05" —
and this stays visible in the gene detail view.

## 5. Screen 4 — Gene Set

**Purpose:** *"Which genes are represented by this pattern?"*

A dedicated exploration view, not just a CSV replacement — header (count + active filter),
search-within-set, sort control, and a table of gene / effect / evidence / expression.

**Table design:** prioritize the dimensions that help scientists evaluate a gene. Initial
columns: gene, `log2FoldChange`, `padj`, `baseMean`, `lfcSE`. `pvalue` remains available but
doesn't need to dominate the initial table. Sorting is user-controlled.

## 6. Screen 5 — Gene Detail

**Purpose:** *"What is happening with this gene?"*

- **DGE Evidence:** `log2FoldChange`, `padj`, `pvalue`, `baseMean`, `lfcSE`
- **Selection context:** pattern name + criteria that led here
- **Biological Context:** placeholder — "Not currently available in source dataset — [ Add
  annotation sources ]"

### Biological context — future layer

The initial prototype doesn't need a full biological knowledge system, but the interface
reserves space for it: Gene → Description → Biological processes → Pathways → Disease
associations → Drug targets → Literature — clearly distinguished from the DGE evidence.

## 7. Navigation

A breadcrumb represents the exploration path, e.g.:

```
Dexamethasone Response > Large Negative Responses > 109 Genes > ENSG00000152583
```

Clicking a previous level returns the scientist to that state.

## 8. Saved Exploration

Eventually the scientist can save an observation as a reusable object:

```
Saved Observation
Name: Large negative response
Criteria: log2FC < -2, padj < 0.05
Genes: 109
Created from: Dexamethasone vs Untreated
```

## 9. Future Comparison Capability

The current case study has only one treatment comparison, but the interface shouldn't
foreclose future comparison (e.g. Compound A / B / C → response comparison). Explicitly out
of MVP scope — the current application should not fabricate additional drug data.

## 10. AI Layer — Future Consideration

AI is not the starting point of the interface. Once the deterministic exploration workflow
is established, AI could assist with: summarizing observed patterns, explaining why a
subset was surfaced, answering questions about the current dataset, connecting selected
genes to approved knowledge sources, and suggesting additional analyses — always operating
**on top of** traceable evidence:

```
DGE data → Deterministic analysis → Visible evidence → AI assistance
```

rather than `DGE data → AI interpretation → Unverifiable conclusion`.

## 11. MVP Screen Map

```
Experiment Overview → Response Landscape → Pattern Exploration →
Gene Set → Gene Detail
```

## 12. Prototype Success Criteria

The prototype should let a user accomplish the following without opening the original CSV:

- **Orientation** — "I understand what experiment I'm looking at."
- **Overview** — "I understand the broad response."
- **Discovery** — "I can see several potentially informative regions."
- **Exploration** — "I can investigate a region using multiple measurements."
- **Narrowing** — "I can define a subset of genes."
- **Investigation** — "I can understand why a particular gene was selected."
- **Traceability** — "I can see how I arrived at this gene."

It does not need to prove biological significance — it needs to demonstrate that the path
from large DGE result → informed exploration is more usable than working directly from a
raw result table.

## 13. Design Principle

Organize around **what does the scientist want to know next?**, not **what visualization
can we make from this column?** The resulting product should feel like an exploration
environment, not a collection of bioinformatics plots.

## 14. Next Step

Once the interface structure is settled, translate it into a visual prototype and decide
exact chart types, layout, interactions, filtering behavior, table behavior, navigation
behavior, and responsive behavior — then implementation.

---

### Where the case study stands

```
DATA (22,008 DGE results)
   ↓
DATA REASONING — What do these measurements mean?
   ↓
SCIENTIST WORKFLOW — What questions does a scientist ask?
   ↓
PRODUCT REQUIREMENTS — What should the system help with?
   ↓
DATA MAPPING — What can our actual data support?
   ↓
USER FLOW — How does the scientist move through it?
   ↓
UI SPECIFICATION — What does the application actually look like?
```

This is the final planning document in the case study. The next step is deciding exactly
which screens to build first, and which interactions need to work against the current
dataset, before moving from planning documents into the actual application build — see
`PORTFOLIO_CASE_STUDY.md` for the current status of that build and the prioritized next
steps.
