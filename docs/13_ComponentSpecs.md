# 13 — Component Specification

## From Architecture to Buildable Components

### Purpose

The architecture in `12` established how the application is organized. This document
defines what actually needs to get built: `Scientist Workflow → Application Views → React
Components → Data + State + Interactions`. The prototype should start with the smallest set
of components that supports the scientist's workflow, not a large component library.

### Component Design Principles

1. **One primary responsibility per component** — a component should have a clear reason
   to exist.
2. **Scientific components understand scientific data; shared components don't** — a
   `VolcanoPlot` should understand DGE results; a `Button` should not.
3. **Components communicate through state and props**, not independent copies of the
   experiment data.
4. **Build for the prototype, not the hypothetical platform** — create components to solve
   a current problem, generalize later if the application actually grows.

### Component Hierarchy

```
App
└── AppShell
    ├── Header
    ├── Sidebar
    └── MainContent
        ├── OverviewView
        ├── ResponseView       → DatasetSummary, FilterPanel, VolcanoPlot, GeneTable
        ├── ExploreView        → FilterPanel, VisualizationToolbar, MAPlot, GeneTable, SelectedGenePanel
        └── GeneDetailPanel    → GeneHeader, GeneStatistics, GeneAnnotation, GeneContext
```

### Shell Components — `App`, `AppShell`, `Header`, `Sidebar`

`App` is the root component — it loads application data, initializes state, and renders
`AppShell`. It should stay small and never contain the implementation of individual
scientific views.

`AppShell` provides the persistent structure (header, sidebar, main content, and the gene
detail panel when active) and should stay mounted while the researcher changes views, so
orientation is never lost.

`Header` provides persistent experiment context — experiment name, comparison, and
optional organism/tissue info (e.g. "Treatment Response in Neuronal Cells / Treatment vs
Control"). It doesn't need significant interactivity in the MVP; its job is context, not
action.

`Sidebar` provides navigation between the major views (Overview, Global Response, Explore,
Selected Genes) and communicates the researcher's navigation choice to application state —
it should never own the scientific data itself.

### Experiment Context — `OverviewView`, `ComparisonSummary`, `DatasetSummary`

`OverviewView` orients the researcher to the experiment — metadata, comparison, analysis
method, dataset summary — and exists to answer *"what am I looking at?"*

`ComparisonSummary` must never leave the direction of the comparison ambiguous (Treatment
vs Control, explicitly, not just "Treatment/Control") — this matters specifically because
log2 fold change is meaningless without knowing which direction the comparison runs.

`DatasetSummary` shows high-level DGE statistics — genes tested, significant, upregulated,
downregulated — derived from the dataset and current thresholds, and exists to answer
*"what happened overall?"*

### Response View and Filtering — `ResponseView`, `FilterPanel`

`ResponseView` is the first major analytical view (`DatasetSummary` + `FilterPanel` +
`VolcanoPlot` + `GeneTable`), moving the researcher from *"what is this experiment?"* to
*"what does the response look like?"*

`FilterPanel` lets the researcher adjust exploration criteria — adjusted p-value, absolute
log2 fold change, and direction for the MVP; base expression, gene type, annotation, and
pathway are reasonable future additions but not required now. When a filter changes, it
flows `FilterPanel → Exploration State → Filtered Dataset → all dependent components
(dataset summary, volcano plot, gene table, selected subset)`. The filter transforms the
view, never the underlying dataset.

### Visualizations — `VolcanoPlot`, `MAPlot`, `GeneTable`, `ExploreView`, `VisualizationToolbar`

| Component | Axes / structure | Purpose |
|---|---|---|
| `VolcanoPlot` | X: log2FC · Y: -log10(padj) | Global view of differential expression; each point is a gene |
| `MAPlot` | X: average expression · Y: log2FC | Effect size relative to expression level, including low-expression behavior |
| `GeneTable` | Sortable columns: gene, log2FC, padj, baseMean | Numerical precision the plots can't provide |

Both plots share the same click behavior: `gene clicked → gene selected → GeneDetailPanel
opens`, keeping selection consistent across visualizations rather than each plot handling
it independently. Hovering a point on the volcano plot should show a concise tooltip (gene,
log2FC, padj, baseMean) — full detail belongs in the gene detail panel, not the tooltip.

`ExploreView` is where deeper exploration happens once a pattern is noticed
(`FilterPanel` + `VisualizationToolbar` + a primary visualization + `GeneTable` +
`SelectedGenePanel`). `VisualizationToolbar` lets the researcher switch representations
(`Volcano / MA Plot / Table` for the MVP; `Heatmap / Expression / Pathways` later) — it
should only expose visualizations the available data actually supports.

### Selection and Gene Detail

`SelectedGenePanel` makes the researcher's current gene selection visible and actionable —
remove a gene, open a gene, clear the selection, create a subset.

`GeneDetailPanel` shows detail for a selected gene without forcing the researcher out of
the current exploration context. It should be dismissible, returning immediately to the
visualization — this is what makes **Pattern → Gene → Pattern** work as a real loop instead
of a dead end. It's composed of four focused sub-components:

| Sub-component | Shows |
|---|---|
| `GeneHeader` | Gene symbol, stable ID, and direction (e.g. "Upregulated") — should make it unambiguous which gene is being inspected |
| `GeneStatistics` | log2FC, p-value, padj, baseMean — preserved exactly as reported, never generating new statistical claims |
| `GeneAnnotation` | Description, gene type, pathways, functional categories where available; shows "Annotation unavailable" rather than implying missing annotation means missing biological relevance |
| `GeneContext` | Places the gene back in the broader experiment — e.g. rank among differential genes. Future versions could add similar genes, pathway membership, or expression across samples; MVP keeps this minimal |

### Shared Components and State Handling

Generic UI components (`Button`, `Badge`, `Dropdown`, `Slider`, `Tooltip`, `Tabs`, `Panel`,
`Card`, `Modal`) should stay scientifically agnostic — `<Button>Clear Selection</Button>`,
not `<ScientificClearGeneSelectionButton />`, unless there's a real need for specialization.

Every major exploratory component needs to handle an **empty state** as an actionable
outcome, not a dead end — e.g. "No genes match the current filters. Try: increasing the
adjusted p-value threshold, reducing the fold-change threshold, removing the direction
filter." That's distinct from an **error state**: "no results" is a valid scientific
outcome, while "unable to load DGE results — check the dataset configuration" is a data or
software problem. Components should never blur those two together.

### Component Communication

```
Application State
    ├── Visualization
    └── Gene Table
           ↓
      Gene Selection
           ↓
    Gene Detail Panel
```

Components should update or consume shared application state rather than communicate
through hidden direct dependencies on each other.

A representative end-to-end interaction: the researcher opens Global Response,
`DatasetSummary` and `VolcanoPlot` render, they notice a cluster of strongly upregulated
genes, adjust a filter, watch the filtered results update, click a gene, review its log2FC
and adjusted p-value in the detail panel, add it to Selected Genes, and continue exploring
the volcano plot. This is the interaction the whole component set exists to support.

### Component Build Order

| Phase | Components |
|---|---|
| 1 — Application shell | `App`, `AppShell`, `Header`, `Sidebar` |
| 2 — Experiment context | `OverviewView`, `ComparisonSummary`, `DatasetSummary` |
| 3 — Core analysis | `ResponseView`, `FilterPanel`, `VolcanoPlot`, `GeneTable` |
| 4 — Exploration | `ExploreView`, `VisualizationToolbar`, `MAPlot`, `SelectedGenePanel` |
| 5 — Gene inspection | `GeneDetailPanel`, `GeneHeader`, `GeneStatistics`, `GeneAnnotation`, `GeneContext` |
| 6 — Refinement | Empty states, error states, loading states, responsive behavior, visual polish |

### What We Should Not Build Yet

An AI chat interface, pathway browser, automated biological interpretation,
multi-experiment comparison, user accounts, project management, cloud data management, a
workflow execution interface, a raw sequencing viewer, and advanced statistical controls
are all tempting but would pull the prototype away from its central purpose. They stay
explicitly out of scope for now.

### Definition of Done

The specification is complete once every major part of the scientist workflow maps to a
concrete implementation target:

| Scientist action | Component |
|---|---|
| Understand experiment | `OverviewView` |
| Understand comparison | `ComparisonSummary` |
| Understand global response | `DatasetSummary` |
| Notice a pattern | `VolcanoPlot` |
| Narrow results | `FilterPanel` |
| Inspect precise values | `GeneTable` |
| Explore another representation | `VisualizationToolbar` / `MAPlot` |
| Maintain a gene subset | `SelectedGenePanel` |
| Inspect a gene | `GeneDetailPanel` |
| Understand gene evidence | `GeneStatistics` |
| Add biological context | `GeneAnnotation` |
| Return to broader context | Persistent exploration view |

**Next artifact:** `14_implementation_plan.md` — the order in which the application actually
gets built, what ships in each development milestone, and what should be working by the
end of each one.
