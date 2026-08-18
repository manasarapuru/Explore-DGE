# 01: Purpose Definition

## Scientist-Facing Exploration of Differential Gene Expression

**Case:** A **HYPOTHETICAL** Applied FDE Case Study

**Client:** Pharmaceutical respiratory research team

**Document type:** Project brief

**Version:** 1.0

---

### Abstract

An existing differential gene expression (DGE) analysis comparing dexamethasone-treated and
untreated human airway smooth muscle cells produced a results table of approximately 22,000
genes ([Kaggle Link](https://www.kaggle.com/datasets/mannekuntanagendra/deseq2-dge-analysis-airway-dataset?resource=download&select=DESeq2_results_airway.csv)). Each gene's expression characterized by five statistical measurements (`baseMean`, `log2FoldChange`,
`lfcSE`, `pvalue`, `padj`). This brief defines the scope of a case study investigating
whether, and how, an interactive scientist-facing application can reduce the effort required
to move from this raw analytical output to specific, evidence-backed candidates for further
investigation. The proposed approach follows a:
  
  Surface → Orient → Explore → Investigate

exploration model and specifies a five-stage capability roadmap (V1–V9) beginning with an
interactive exploration tool built on the existing dataset alone. This document establishes
the research context, client need, exploration model, and V1 scope; it does not report
experimental results or biological findings.

---

### 1. Background

Differential gene expression analysis is a standard output of RNA-seq experiments and is
routinely delivered to research teams as a static results table. While such tables contain
the complete statistical record of an experiment, they do not inherently communicate the
structure of the result. Some examples are the overall response landscape, the relationship between effect
size and statistical evidence or which regions of the result merit closer inspection. As a
consequence, researchers commonly rely on ad hoc combinations of spreadsheets, static plots,
and manual scripting to interpret DGE output, a process that scales poorly with dataset size
and is difficult to reproduce or audit.

This case study treats an existing DGE output as the starting artifact and investigates the
product and engineering work required to convert it into a navigable, evidence-preserving
exploration environment for domain experts who are not primarily software users.

### 2. Research Context

The reference dataset used throughout this case study is a DESeq2-based DGE analysis of
human airway smooth muscle cells, comparing cells treated with dexamethasone against
untreated controls. This is a well-characterized, publicly available dataset, selected for
its realistic scale and statistical structure rather than for any novel biological finding.

### 3. Research Questions

The case study addresses the following questions, framed from the perspective of the
research team using the output:

1. Which genes show the strongest responses to treatment?
2. Which responses are statistically supported, and to what degree?
3. What structural patterns exist across the full gene-level result set?
4. Which groups of genes warrant closer investigation, and on what basis?
5. What biological context is associated with observed statistical patterns?

### 4. Materials

**Dataset:** Airway / Dexamethasone DGE result.

**Composition:** Approximately 22,000 gene-level observations, each comprising:

| Field | Measurement |
|---|---|
| Gene ID | Unique gene identifier |
| `baseMean` | Average normalized expression level |
| `log2FoldChange` | Estimated magnitude and direction of expression change |
| `lfcSE` | Standard error of the estimated log2 fold change |
| `pvalue` | Statistical evidence against the null hypothesis of no difference |
| `padj` | `pvalue` adjusted for multiple testing |

**Analytical provenance:** The dataset was produced by an existing DESeq2-based
differential expression workflow. No new sequencing, alignment, or normalization was
performed as part of this case study; the results table is treated as a fixed input.

### 5. Client Need

The client requires an interactive environment that allows researchers, without
prior knowledge of specific genes or patterns, to:

1. Understand the overall response to treatment.
2. Surface informative patterns within the result set.
3. Explore the data through multiple prioritization lenses.
4. Investigate groups of genes sharing a pattern of interest.
5. Drill into individual genes.
6. Access the underlying statistical evidence for any observation.
7. Connect statistical observations to biological context.

### 6. Core Product Question

> How should researchers navigate a large DGE result set when they do not know in advance
> which genes or patterns are worth investigating?

### 7. Methods: Exploration Model

The proposed interaction model follows four stages:

**Surface → Orient → Explore → Investigate**

| Stage | Product behavior |
|---|---|
| Surface | Show major characteristics of the dataset |
| Orient | Present multiple ways to explore the results |
| Explore | Investigate selected patterns or groups |
| Investigate | Drill into genes, measurements, and context |

### 8. Exploration Lenses

Consistent with the exploration model, the application is required to support the
following analytical lenses over the result set:

- Largest observed effects
- Strongest statistical evidence
- Combined effect and statistical evidence
- Upregulated responses
- Downregulated responses
- Response patterns (relationships between measurements)
- Biological context

The system is required to surface evidence and patterns as descriptive observations; it
is explicitly not required, and should not attempt, to classify genes as universally
"important."

### 9. V1 Scope

| Component | Specification |
|---|---|
| Input | DGE results CSV |
| Processing | Python, pandas |
| Output | Interactive scientist-facing application |

**Core capabilities:**

- Dataset overview
- Response distributions
- Interactive filtering
- Prioritization views
- Gene/group exploration
- Gene-level detail views
- Access to underlying statistical values
- Export / share functionality

### 10. Planned Iterations Beyond V1

| Iteration | Capability | Technology |
|---|---|---|
| V1 | Interactive exploration | Python, pandas, visualization |
| V2 | Pattern discovery | scikit-learn |
| V3 | Learned representations | PyTorch |
| V4 | Similarity search | Embeddings + vector search |
| V5 | Scientific evidence retrieval | RAG |
| V6 | Multi-step investigation | Tool calling / agents |
| V7 | Biological relationships | Knowledge graphs |
| V8 | Production deployment | APIs, Docker, AWS |
| V9 | Reliability | Evaluation + observability |

Iterations V2 onward are outside the scope of this brief and are recorded here to
establish the intended trajectory of the work, not as committed deliverables.

### 11. Success Criteria

The V1 prototype will be considered successful if it demonstrates that a researcher,
using the application alone, can:

1. Quickly understand the composition and scale of the dataset ✅
2. Identify potential areas of interest without prior knowledge of specific genes ✅
3. Move from broad, dataset-level patterns to individual gene-level results ✅
4. Investigate the same result set from multiple analytical perspectives ✅
5. Access the statistical evidence underlying any surfaced observation ✅
6. Distinguish computational observations (derived directly from the data) from
   biological interpretation (requiring external context) ✅

### 12. Procedure (FDE Workflow)

The case study follows the standard engagement sequence used throughout this project:

> Client context → Data → User workflow → Requirements → Prototype → Evaluation →
> Next requirement → Technical expansion

### 13. Related Artifacts

This brief stays intentionally short. Everything that needs more explanation lives in its own document:

02 — Discovery Notes
03 — Data Exploration Findings
04 — FDE Data Reasoning
05 — Scientist Workflow
06 — Product Requirements
07 — Data-to-Feature Mapping
08 — Application User Flow
09 — UI Prototype Specification
