# 12 — Application Architecture

## Translating the Spec into a System

### Purpose

This document turns the previous documents — scientist workflow, product requirements,
user flow, UI spec, prototype plan, and data specification — into an implementable
architecture. The goal isn't a production bioinformatics platform; it's a small,
maintainable application that demonstrates the core interaction: **DGE results →
exploration → pattern → subset → gene-level evidence.**

### Architecture Goals

The prototype should be simple enough to build locally, interactive enough to demonstrate
the scientist workflow, modular enough to extend later, independent of the original DGE
analysis pipeline, transparent about where information comes from, easy to modify as the
prototype is tested, and equally legible as scientific thinking and as engineering. The
architecture prioritizes **clarity over infrastructure complexity.**

### Technology Stack

| Layer | Technology |
|---|---|
| Frontend | React |
| Language | TypeScript |
| Build tool | Vite |
| Styling | CSS / Tailwind |
| Visualization | Plotly or D3 |
| State | React state / Context |
| Data | Local JSON |
| Testing | Vitest / React Testing Library |
| Version control | Git |
| Deployment | Optional static hosting |

The prototype does not need Kubernetes, microservices, a cloud database, authentication,
message queues, serverless infrastructure, or a dedicated backend. Those can be introduced
later, if the product actually needs them.

### High-Level Architecture

```
Prepared DGE Data (JSON/CSV)
        ↓
Data Layer (validation / loading)
        ↓
Application State (filters / selections)
        ↓
     ┌─────────────────┬─────────────────┐
Experiment Context  Exploration Views  Gene Detail View
     └─────────────────┴─────────────────┘
                        ↓
              Scientist Interaction
```

### Frontend-First, Deliberately

The core product question is about **scientific interaction**, not computational
infrastructure, so the prototype should run entirely from a prepared local dataset:
`Local Dataset → React Application → Interactive Scientific Interface`. That avoids routing
every development change through a full frontend → API → backend → database → analysis
pipeline chain, and lets iteration stay fast while the interaction model is still being
validated.

### Application Layers

```
UI Layer            pages / panels / controls / charts
Interaction Layer    selection / filtering / navigation
State Layer          experiment / filters / gene sets
Data Layer           loading / validation / transformation
Dataset              DGE results
```

The **Data Layer** is specifically responsible for getting scientific data into the
application: loading the dataset, validating its structure, normalizing records, and
providing access to experiment, comparison, and gene-level data. The UI should never
directly parse raw CSV or perform data cleaning — everything flows `Raw/Prepared Data →
Data Layer → Clean Application Objects → UI`.

### Application State

State describes what the researcher is doing, not a duplicate of the dataset:

```typescript
interface ExplorationState {
  filters: FilterState;
  selectedGenes: string[];
  activeView: ViewName;
  currentSubset?: GeneSubset;
}
```

### Filtering and Derived Data

Filtering is a transformation of the underlying dataset, not a mutation of it: `All DGE
Results → Filter Conditions → Filtered Results → Visualization + Table` — e.g. `padj <
0.05` → `|log2FC| > 1` → upregulated only → current subset. The dataset itself never
changes.

Derived values (significant genes, upregulated/downregulated genes, ranked genes, filtered
genes) should be computed from the source dataset and current state — `const filteredGenes
= genes.filter(...)` — rather than stored as a second, independent copy. That's what keeps
different views from silently disagreeing with each other.

### Visualization Layer

Visualizations are reusable components that receive data as input — `<VolcanoPlot
data={filteredGenes} />` — rather than independently retrieving or transforming the
dataset themselves. Candidate components: `VolcanoPlot`, `MAPlot`, `GeneTable`, `Heatmap`,
`ExpressionDistribution`.

More importantly, visualizations need to communicate back to application state, not just
render it: a click on the volcano plot updates `selectedGenes`, which simultaneously
updates the gene detail panel and the gene table. This is one of the most important
patterns in the whole architecture — **a visualization here is an interaction surface, not
an isolated image.**

### Component Architecture and Page Structure

Components fall into a few natural categories: layout (`AppShell`, `Header`, `Sidebar`,
`MainContent`), context (`ExperimentContext`, `ComparisonSummary`, `DatasetSummary`),
exploration (`FilterPanel`, `VolcanoPlot`, `MAPlot`, `GeneTable`, `GeneSetPanel`), gene
(`GeneDetailPanel`, `GeneStatistics`, `GeneAnnotation`), and shared/generic UI (`Button`,
`Dropdown`, `Slider`, `Badge`, `Tooltip`, `Modal`, `EmptyState`). Keeping scientific
components separate from generic UI components keeps the codebase legible as it grows.

The prototype should use a single application shell with multiple views — Experiment
Overview, Global Response, Explore, Gene Detail — rather than independent pages. The
researcher should feel like they're moving through one continuous workspace, not four
unrelated webpages.

### Shell, Navigation, and Gene Detail

The shell should keep experiment, comparison, current exploration context, and selected
genes persistently visible, so orientation never gets lost:

```
┌─────────────────────────────────────────────────────┐
│ Experiment: Treatment vs Control                     │
├───────────────┬───────────────────────────────────────┤
│ Overview       │                                      │
│ Response       │            Main Workspace             │
│ Explore        │                                      │
│ Selected (12)  │                                      │
└───────────────┴───────────────────────────────────────┘
```

Navigation should reflect the reasoning process (Overview → Global Response → Explore →
Selected Genes) without forcing a strictly sequential path — a researcher should be able to
go Overview → Global Response → Gene Detail → Explore Similar Genes without losing context.
The workflow is guided, not locked.

Gene detail specifically should open as a panel or drawer rather than replacing the current
visualization outright — keeping the volcano plot visible alongside the gene detail panel
is what makes **Pattern → Gene → Pattern** actually work as a two-way interaction instead
of a one-way drill-down.

### State Flow and Ownership

A typical interaction: open experiment → initialize experiment state → calculate global
results → user changes filter → filter state updates → derived result set recalculates →
charts and table update → user selects gene → selection state updates → gene detail panel
opens. Different components should never end up holding conflicting versions of this state.

The rule of thumb for where state should live: **state lives at the lowest level that
needs to share it, but no lower.** Experiment, comparison, filters, and selected genes are
global state; tooltip visibility, temporary input values, and whether a panel is expanded
are local component state. Keeping that boundary clear avoids unnecessary complexity.

### What's Deliberately Deferred

Three things are architected for, but not built, in the MVP:

- **URL state.** Encoding exploration state into the URL (`/explore?padj=0.05&log2fc=1&
  direction=up`) would eventually enable sharing, bookmarking, and reproducibility — useful,
  but not required for V1.
- **A backend.** The prototype doesn't need one, but the architecture leaves room for a
  future `Frontend → API → {Dataset, Analysis, Annotation} Services` split without
  redesigning the interaction model.
- **AI.** Treated as a future service layered on top of the application state
  (`Scientific Data → Application State → AI Reasoning Service → Candidate Pattern →
  Researcher Review`), not embedded into individual components. AI should never silently
  modify scientific data, and any AI-generated interpretation must stay visually
  distinguishable from measured or calculated results. **No AI-generated biological
  conclusions are required for the MVP.**

### Error, Loading, and Empty States

The interface needs explicit states for dataset failure ("Unable to load experiment
data"), invalid data ("This dataset is missing required DGE fields"), missing annotation
("Gene annotation is unavailable for this gene"), and simple loading feedback ("Loading
experiment...", "Updating results..."). Simple, clear loading states are enough — the MVP
doesn't need skeleton screens.

One state deserves special handling: **an empty result ("No genes match the current
filters") is not an application failure — it's a legitimate scientific outcome**, and
should be presented that way rather than as an error.

### Performance

At the expected scale (roughly 10,000–50,000 genes), client-side filtering should be
sufficient — no need for web workers, virtualized tables, server-side filtering, or
precomputed summaries in the MVP. Worth revisiting only if the dataset size grows well
beyond that.

### Folder Structure and Core Types

```
src/
├── app/                  App.tsx, routes.ts
├── components/           layout/, experiment/, exploration/, genes/, visualizations/, shared/
├── data/                 experiment.json, dge-results.json, annotations.json
├── hooks/                useExperiment.ts, useFilters.ts, useGeneSelection.ts
├── state/                explorationContext.tsx
├── types/                experiment.ts, dge.ts, exploration.ts
├── utils/                filtering.ts, statistics.ts, formatting.ts
└── styles/
```

Core TypeScript types become the contract between the data layer and the UI:

```typescript
interface DGEResult {
  gene_id: string;
  gene_symbol: string;
  log2FC: number;
  pvalue: number;
  padj: number;
  baseMean?: number;
  annotation?: string;
}

interface Experiment {
  id: string;
  name: string;
  organism?: string;
  tissue?: string;
  cell_type?: string;
  analysis_method?: string;
}

interface Comparison {
  id: string;
  name: string;
  condition_a: string;
  condition_b: string;
  reference_condition: string;
}
```

### Scientific Logic vs. UI Logic, and Testing

Scientific/data logic (filtering by adjusted p-value, computing absolute log2FC, ranking
genes, classifying direction) should stay separate from UI logic (opening a panel,
switching a tab, showing a tooltip) — that separation is what makes both easier to reason
about and test.

Testing doesn't need to be exhaustive; it needs to protect the core interaction. Data tests
verify required fields exist, numerical values are valid, and filtering behaves correctly.
Component tests verify filters update correctly, gene selection works, and gene detail
shows correct values. Interaction tests cover the core loop: filter → select gene → open
gene detail → return to visualization.

### Reproducibility

A new developer should be able to clone the repo, install dependencies, start the dev
server, load the example dataset, and start exploring — with no hidden local files or
undocumented configuration standing in the way.

### Architecture Decisions

| Decision | Reason |
|---|---|
| Use prepared data, not raw sequencing data | The prototype is testing scientific interaction, not sequencing infrastructure |
| Frontend-first architecture | Fast iteration matters more than backend completeness at this stage |
| Keep source data separate from exploration state | A filter should change what the researcher sees, not the underlying experiment |
| Make visualizations interactive components | The core workflow depends on moving between patterns and genes, not just viewing static charts |
| Defer AI | The underlying interaction needs to prove useful before adding automated interpretation on top of it |

### Architecture Risks

| Risk | Mitigation |
|---|---|
| Overengineering — backend/DB/state complexity creeps in early | Keep the MVP local and frontend-first |
| Visualization overload | Every visualization must map to a real scientific question |
| State fragmentation across components | One central exploration state |
| Hard-coded thresholds become invisible assumptions | Represent thresholds explicitly in configuration/state |
| Prototype drifts into a generic analytics dashboard | Continuously check features against the scientist workflow in `05` |

### Architecture Principle

> **The application should be organized around the movement of scientific reasoning, not
> around the structure of the codebase.**

Everything exists to support one movement: `Experiment → Response → Pattern → Subset →
Gene → Evidence → Interpretation`. The architecture's only job is to make that movement
easy to implement, easy to understand, and easy to extend.

### Definition of Done

The architecture is sufficiently defined once implementation can begin without further
major structural decisions — i.e. it's clear what data enters the application, where it
lives, how it's represented, what state the researcher controls, how filtering works, how
visualizations receive data, how selections propagate, how gene-level inspection works,
where scientific logic lives vs. UI logic, and what's intentionally deferred.

**Next artifact:** `13_component_specification.md` — the individual components to build,
what each one does, what data it receives, what interactions it supports, and how they
connect to one another.
