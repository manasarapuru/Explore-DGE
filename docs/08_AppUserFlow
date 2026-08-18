# 08 — Application User Flow

### Purpose

Translates the scientist workflow and product requirements into an application-level user
flow — how a scientist moves through the application from receiving a DGE result to
identifying observations that may deserve further investigation, without requiring them to
begin with a specific gene, pathway, or hypothesis.

```
02 What is in the data?
     ↓
03 What does the data tell us / what questions arise?
     ↓
04 How does the scientist work?
     ↓
05 What should the product do?
     ↓
06 Can our data actually support those capabilities?
     ↓
07 How does the scientist actually move through the application?
```

---

## 1. Core User Flow

```
Receive DGE Results → Orient to the Experiment → Understand the Overall Response →
Notice Potentially Informative Patterns → Explore a Pattern → Narrow to a Subset
of Genes → Investigate Individual Genes → Add Biological Context →
Decide What Deserves Follow-Up
```

The application preserves the scientist's context as they move through these stages —
Pattern → Subset → Gene stays traceable so the scientist knows *why* a particular gene is
being examined.

## 2. Entry Point — Receive DGE Results

The scientist opens the application with a DGE result already in hand (human airway smooth
muscle cells; dexamethasone vs. untreated; ~22,000 genes). The application should
immediately establish what experiment is being viewed, what comparison is being made, how
many genes are represented, and what type of result was provided — the scientist shouldn't
have to open the CSV and interpret column names first.

## 3. Stage 1 — Experiment Orientation

**Question:** *"What am I looking at?"*

```
Dexamethasone Response — Human Airway Smooth Muscle Cells
Treatment: Dexamethasone   Comparison: Untreated   Genes analyzed: 22,008

What does this result contain?
Effect: log2FoldChange   Expression: baseMean
Uncertainty: lfcSE       Evidence: pvalue / padj

[ Explore Response → ]
```

## 4. Stage 2 — Understand the Overall Response

**Question:** *"What happened across the dataset?"*

The scientist sees the response landscape rather than a 22,000-row table: direction of
change (11,441 positive / 10,567 negative), effect distribution (most genes small changes,
a smaller subset larger), statistical-evidence distribution, and expression distribution.
This leads naturally to *"What stands out?"*

## 5. Stage 3 — Notice Potentially Informative Patterns

**Question:** *"What should I pay attention to?"*

The application surfaces candidate patterns without deciding what's biologically important:
large positive/negative effects, strong statistical evidence, combined evidence (large
effect + strong adjusted evidence), expression-aware patterns, and high-uncertainty large
effects.

**Interaction principle:** don't just present "Top 10 genes" — present different ways of
looking at the response, e.g.:

```
Potentially informative regions
Large positive responses    74 genes
Large negative responses    109 genes
Strong statistical evidence ... genes
Large + significant         138 genes
```

The scientist chooses where to explore.

## 6. Stage 4 — Explore a Pattern

The scientist selects, e.g., "Large negative responses." The application opens an
interactive view of that region where the scientist examines effect size, statistical
evidence, expression, and uncertainty together, and can change criteria in real time (e.g.
`log2FC < -2` → `log2FC < -1.5`, updating 109 → 214 genes; then adding `padj < 0.05`).

**Key product behavior:** always show current criteria + number of genes, so filtering
never becomes opaque, e.g.:

```
Current exploration: log2FC < -1.5, padj < 0.05 → 214 genes
```

## 7. Stage 5 — Narrow to a Subset of Genes

**Question:** *"Which genes are represented by this pattern?"*

The resulting subset is presented as a table (gene, log2FC, padj, baseMean, lfcSE), sortable
by effect, significance, or expression. The application preserves the origin of the subset
— pattern name and filter criteria stay visible.

## 8. Stage 6 — Investigate an Individual Gene

**Question:** *"What is happening with this gene?"*

The gene detail view shows expression, effect, uncertainty, and statistical evidence, and
**context preservation** — *"You reached this gene from: Pattern = Large negative response,
Criteria = log2FC < -1.5, padj < 0.05"* — answering *"why am I looking at this gene?"*

## 9. Stage 7 — Add Biological Context

**Question:** *"What might this mean biologically?"*

The application eventually connects gene → function → pathways → biological processes →
disease associations → drug-target information → literature, while visually distinguishing
*experimental evidence (observed in this experiment)* from *external biological context
(known from external sources)* — so contextual information never appears to be an
experimental finding.

## 10. Stage 8 — Decide What Deserves Follow-Up

**Question:** *"Is this worth investigating further?"*

The scientist weighs experimental evidence, statistical evidence, effect size, expression,
uncertainty, and biological context. The application organizes this evidence; it does not
make the final scientific decision. Possible actions: save gene, save subset, save
observation, compare with another experiment, investigate pathway, open literature, export
results, flag for follow-up.

## 11. Navigation Model

The application supports both forward and backward exploration:

```
Overview → Pattern → Subset → Gene → Context
Overview ← Pattern ← Subset ← Gene
```

without losing the current exploration state.

## 12. Persistent Exploration State

The application preserves a chain: Experiment → Pattern → Filters → Subset → Selected
gene — e.g. *"Dexamethasone vs Untreated → Large negative response → log2FC < -1.5, padj <
0.05 → 214 genes → ENSG00000152583"* — and this chain should be recoverable by the
scientist.

## 13. Alternative Entry Points

The primary workflow begins with the overall experiment, but scientists may enter from
different directions:

- **Start from overview:** Experiment → Response → Pattern
- **Start from a gene:** Gene → DGE evidence → Biological context
- **Start from a saved subset:** Saved subset → Gene exploration
- **Start from a biological question (future):** Pathway → Relevant genes → Experiment
  evidence

The MVP does not need to implement every entry point.

## 14. MVP User Flow

```
Experiment Overview → Response Overview → Pattern Exploration →
Interactive Filtering → Gene Subset → Gene Detail
```

Biological context is an initial future extension, not part of the MVP.

## 15. Application State Model

```
Experiment
  ├── Overview
  └── Exploration
        ├── Pattern
        ├── Filters
        └── Gene Subset
              └── Selected Gene
```

Each downstream view inherits context from the upstream exploration.

## 16. Scientist ↔ Application Interaction

```
Scientist asks a question → Application surfaces relevant data →
Scientist notices something → Scientist changes the question →
Application updates the view → Scientist narrows the investigation
```

E.g.: *"What happened?"* → overall response; *"What stands out?"* → large negative
responses; *"Why is this interesting?"* → compare effect + significance + expression;
*"Which genes are involved?"* → gene subset; *"What is happening with this gene?"* → gene
detail; *"What might it mean?"* → biological context; *"Is it worth following up?"* →
scientist judgment.

## 17. FDE Design Principle

The application should not simply convert the existing CSV into a prettier table. The
product is organized around the scientist's sequence of questions:

```
CSV structure ≠ Scientist workflow
Scientist workflow → Product interaction → Data presentation
```

The underlying data remains the evidence layer; the application determines how that
evidence is surfaced and explored.

> **Note on approach:** the homepage isn't specified as "a volcano plot" — that's
> deliberate. We start from the scientist's question ("What happened overall?") and ask what
> visualization best answers *that*. The visualization is a solution to a user problem, not
> the starting point.

## 18. Transition to Prototype Design

Before implementation, define: primary screens, layout of each screen, visual hierarchy,
interactive components, navigation, charts, tables, filtering controls, gene-detail
structure, and what appears automatically vs. on demand.

**Next artifact:** `09_ui_prototype_specification.md`
