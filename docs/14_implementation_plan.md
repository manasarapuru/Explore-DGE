# 14 — Implementation Plan

## From Specification to a Working Application

### Purpose

This is the last planning document before implementation. It turns the architecture and
component specification into an executable build sequence: `what the scientist needs →
what the application should do → what the interface looks like → what components exist →
how we build it`. Implementation proceeds incrementally — at every stage there should be a
working application, not one big build followed by a reveal.

### Implementation Philosophy

1. **Build the scientific workflow first.** Get `Orient → Understand → Notice → Explore →
   Narrow → Inspect` working end to end; visual polish comes after, not during.
2. **Build vertically, not horizontally.** Complete slices of functionality (`Data → Filter
   → Visualization → Gene selection → Gene detail`) beat building every component in
   isolation — a vertical slice produces a working experience early.
3. **Use real data as soon as practical.** Mock data is fine for establishing UI structure,
   but the core prototype should run on the actual prepared DGE dataset early.
4. **Avoid premature infrastructure.** The first implementation runs entirely locally — no
   backend, database, authentication, or cloud deployment required for the MVP.

### Development Stack

React, JavaScript, Vite, Tailwind CSS (or plain CSS), Plotly.js, React Context/`useState`,
local JSON data, Git — with Vitest and React Testing Library for testing, if used.

### Milestones Overview

| # | Milestone | Produces |
|---|---|---|
| 1 | Project Setup | A running dev environment |
| 2 | Data Layer | The app can load the real DGE dataset |
| 3 | Application Shell | Persistent nav and layout |
| 4 | Scientific Overview | Experiment context + overall response summary |
| 5 | Global DGE Exploration | Filtering + volcano plot + gene table, in sync |
| 6 | Gene Investigation | Selecting and inspecting an individual gene |
| 7 | Cross-View Interaction | Selections and subsets persist across views |
| 8 | Refinement | Usability, scientific clarity, testing, validation |

Each milestone should produce something runnable and evaluable — not a partial feature
buried behind others still in progress.

### Milestone 1 — Project Setup

Create the Vite React app, initialize Git, install dependencies (including visualization
libraries — `npm install plotly.js react-plotly.js` — and Tailwind, if used), establish the
basic source structure, and confirm the dev server runs. The application doesn't need to be
scientifically functional yet — opening it and seeing a placeholder ("DGE Explorer —
Prototype") is a sufficient result at this stage.

### Milestone 2 — Data Layer

Add the experiment JSON, comparison metadata, DGE result dataset, and optional annotations
under `src/data/`, then build a small data-access layer so UI components never import the
raw JSON directly:

```javascript
import dgeResults from "../data/dge-results.json";
export function getDGEResults() {
  return dgeResults;
}
```

The exact implementation can evolve, but that indirection matters — it's what lets the
data source change later without touching every component that reads it. Success at this
stage means the application can load and display the experiment name, comparison, and gene
count — no visualization required yet.

### Milestone 3 — Application Shell

Build `App → AppShell → Header, Sidebar, MainContent`. The header shows experiment and
comparison ("Treatment Response in Neuronal Cells / Treatment vs Control"); the sidebar
lists Overview, Global Response, Explore, and Selected Genes; `MainContent` starts as
placeholder views. Success here is a researcher being able to move between the three main
views, even while those views are still mostly empty — the navigation skeleton exists.

### Milestone 4 — Scientific Overview

Build `OverviewView`, `ComparisonSummary`, and `DatasetSummary`, displaying experiment name,
organism, tissue/cell type, comparison, analysis method, gene count, and significant/
upregulated/downregulated counts. These derived statistics should be simple filters over
the dataset:

```javascript
const significantGenes = genes.filter(gene => gene.padj < 0.05);
const upregulatedGenes = significantGenes.filter(gene => gene.log2FC > 1);
const downregulatedGenes = significantGenes.filter(gene => gene.log2FC < -1);
```

with thresholds sourced from application configuration, not hardcoded inline or scattered
across components. Success: a researcher can open the app and answer both "what experiment
am I looking at?" and "what happened overall?"

### Milestone 5 — Global DGE Exploration

This is the first major milestone — the app becomes genuinely interactive. Build
`ResponseView` (`DatasetSummary` + `FilterPanel` + `VolcanoPlot` + `GeneTable`).

Start filtering with three controls — adjusted p-value, absolute log2FC, direction — via a
single filtering function that both the plot and table consume:

```javascript
function filterGenes(genes, filters) {
  return genes.filter(gene => {
    if (gene.padj > filters.padj) return false;
    if (Math.abs(gene.log2FC) < filters.absoluteLog2FC) return false;
    if (filters.direction === "up" && gene.log2FC <= 0) return false;
    if (filters.direction === "down" && gene.log2FC >= 0) return false;
    return true;
  });
}
```

The volcano plot (x: log2FC, y: -log10(padj), one point per gene) needs plotting, hover,
and click-to-select for this milestone — nothing more sophisticated yet. The gene table
needs gene, log2FC, padj, baseMean, with sorting and selection.

**Both must read from the same filtered dataset.** This is the key architecture test for
this milestone: if the plot shows 120 genes while the table shows 87 because they're
running different filtering logic, the architecture is wrong, not just the numbers.

Milestone 5 is done when a researcher can open an experiment, understand the comparison,
see the global response, adjust thresholds, watch results update, and inspect genes both
visually and numerically — a basic but genuinely functional exploration tool.

### Milestone 6 — Gene Investigation

Move the researcher from "I see an interesting gene" to "I can investigate this gene."
Build `GeneDetailPanel` with its four sub-components (`GeneHeader`, `GeneStatistics`,
`GeneAnnotation`, `GeneContext`). Selection should work identically whether triggered from
the volcano plot or the gene table, both updating the same `selectedGene` state. Displayed
values (log2FC, p-value, padj, baseMean, annotation) must come directly from the underlying
DGE record — no re-derivation. The detail panel should sit alongside the current
visualization where practical, so closing it returns the researcher to exactly where they
were, filters and position intact — not a reset.

### Milestone 7 — Cross-View Interaction

This is where the prototype stops being a collection of charts and starts being a coherent
exploration environment. Build a persistent Selected Genes list (add, remove, clear, open
detail) that supports multi-selection from either the volcano plot or the table, and
survives view changes. Build `ExploreView` (`FilterPanel` + `VisualizationToolbar` +
primary visualization + `GeneTable` + `SelectedGenePanel`), then add the MA plot as a second
visualization option — switching between Volcano/MA Plot/Table should never lose the
current selection, which is the real test of whether application state is actually
centralized. Finally, allow turning a selection into a named subset with basic provenance
("Strongly Upregulated — 12 genes — created from Treatment vs Control, padj < 0.05, log2FC
> 1") — this is what starts capturing the reasoning trail, not just the result.

Milestone 7 is done when the researcher can move `global response → filter → notice pattern
→ select genes → create subset → inspect genes → switch views → return to broader context`
without losing state at any step. This loop is the core product hypothesis in working form.

### Milestone 8 — Refinement

With the full workflow functional, this stage is about usability and scientific clarity,
not new features.

**Visual refinement:** spacing, typography, hierarchy, labels, chart sizing, panel
behavior, responsiveness — the interface should read as a scientific tool, not a generic
dashboard.

**Scientific clarity — audit every visualization and label:** Is the comparison direction
obvious? Are thresholds visible? Are statistical values labeled correctly, without
confusing significance with effect size? Is it clear what each visualization represents?
Are missing values handled honestly rather than silently dropped? Scientific accuracy
outranks visual polish if the two ever conflict.

**States:** meaningful empty states ("No genes match the current filters — try increasing
the adjusted p-value threshold, reducing the fold-change threshold, or removing the
direction filter"), clear error states for missing/invalid/failed data, and simple loading
indicators ("Loading experiment...", "Updating results...") — no elaborate animation
needed.

**Testing:** data tests (DGE data loads, required fields exist, filters behave correctly,
summary statistics are correct), component tests (filter changes update results, clicking a
gene opens details, selections persist, detail values match source data), and one full
workflow test simulating open → filter → select → inspect → subset → switch view → return.

**Scientific validation:** before calling the prototype done, manually spot-check several
genes against the source DGE file — confirm the application's displayed `log2FC`/`padj`
for a gene matches the source row exactly. This one check catches most data-transformation
bugs before they become invisible.

**Performance:** check initial load, filter response, chart rendering, table rendering, and
selection behavior at the actual dataset size. If it stays responsive, no optimization work
is needed yet — only address performance once it's a real, observed problem.

### Git Strategy

Commit in meaningful increments that double as a readable project history — e.g. `Initialize
React prototype`, `Add DGE data layer`, `Build application shell`, `Add experiment
overview`, `Add DGE filtering`, `Add interactive volcano plot`, `Add gene table`, `Add gene
detail panel`, `Add gene selection`, `Add exploratory MA plot`, `Refine scientific
workflow`. This also makes the project much easier to walk someone through later — the
commit history becomes a second version of this document.

### What Counts as the MVP

The MVP is done when this interaction works reliably: `open experiment → understand
comparison → view overall DGE response → filter results → inspect volcano plot → select a
gene → inspect gene evidence → return to visualization.` Everything beyond this is an
enhancement, not a requirement.

Reasonable next steps after the MVP, in rough order of value: biological enrichment
(selected genes → pathway analysis → functional categories), richer expression context
(selected genes → expression matrix → heatmap), external annotation sources (gene → Ensembl
or pathway resources → additional context), and AI-assisted exploration (observed pattern →
AI-generated candidate interpretation → evidence shown → researcher evaluates). All of these
should extend the core workflow, not replace it.

### Definition of Done

**Data** — real DGE data loads successfully; experiment metadata displays; comparison
direction is explicit; basic data validation exists.

**Exploration** — summary statistics, filtering, volcano plot, gene table, gene selection,
and gene detail all work; selected genes persist across views.

**Scientific integrity** — application values match the source data exactly; thresholds are
explicit, not hidden; missing data is handled honestly; no unsupported biological claims are
generated by the interface itself.

**User experience** — navigation works; empty and error states exist; the researcher never
loses context when inspecting a gene; the full workflow completes without any manual data
transfer between screens.

### Final Implementation Principle

Every implementation decision should answer one question: **does this make the scientist's
next analytical step easier?** The finished prototype should trace one continuous line —
`data → context → global response → pattern recognition → exploration → gene subset →
evidence → scientific reasoning` — and the point of the whole build isn't to demonstrate how
many technologies got used. It's to show that a real scientific workflow can be understood
deeply enough to turn into software someone would actually want to use.

---

This is the final planning document in the case study — `00` through `14` cover the full
arc from mental model to buildable implementation plan. What comes next is the build
itself; see `PORTFOLIO_CASE_STUDY.md` for current build status and next steps.
