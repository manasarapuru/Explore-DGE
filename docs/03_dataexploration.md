# Data Exploration Findings

## Airway DGE — Dexamethasone Response

**Dataset:** Airway / Dexamethasone DGE
**Source:** Existing DESeq2 results table
**Rows:** 22,008 genes · **Columns:** 5

---

### Overview

Before designing the scientist-facing application, we characterized the existing DGE
results table to understand what it actually contains: the distribution of transcriptional
responses, response direction and magnitude, statistical evidence, expression levels,
estimate uncertainty, and how these measurements relate to one another. This is a
lightweight assessment of the existing analysis output, not a new differential expression
analysis — the goal is to let the data inform the product, not the other way around.

### Dataset Integrity

| Finding | Result |
|---|---:|
| Total genes | 22,008 |
| Positive log2FC | 11,441 |
| Negative log2FC | 10,567 |
| Zero log2FC | 0 |
| Non-missing `baseMean` | 22,008 |
| Non-missing `log2FoldChange` | 22,008 |
| Non-missing `lfcSE` | 22,008 |
| Non-missing `pvalue` | 21,957 |
| Non-missing `padj` | 18,117 |

The response is distributed in both directions — 11,441 genes increased and 10,567
decreased — and no gene shows an exactly zero log2FC. The statistical columns do contain
missing values, and those need to be handled explicitly by the application rather than
treated as evidence of a biological result.

### Response Magnitude

The distribution of log2 fold changes is heavily concentrated around zero:

| log2FC range | Number of genes |
|---|---:|
| `< -2` | 109 |
| `-2 to -1` | 281 |
| `-1 to -0.5` | 591 |
| `-0.5 to 0` | 9,586 |
| `0 to 0.5` | 10,560 |
| `0.5 to 1` | 564 |
| `1 to 2` | 243 |
| `> 2` | 74 |

About 91.5% of genes fall between -0.5 and +0.5 log2FC, with only 74 genes above +2 and 109
below -2. So the dataset contains a large background of relatively small estimated changes
alongside a much smaller number of larger responses. Practically, this means a
scientist-facing application shouldn't require researchers to inspect all 22,008 genes
individually — it should help them understand the overall response landscape first, then
progressively narrow in.

### Statistical Evidence

Adjusted p-value (`padj`) was available for 18,117 of the 22,008 genes:

| padj range | Number of genes |
|---|---:|
| `< 0.001` | 1,121 |
| `0.001–0.01` | 614 |
| `0.01–0.05` | 977 |
| `0.05–0.1` | 699 |
| `0.1–0.5` | 4,319 |
| `0.5–1` | 6,387 |
| Missing | 3,891 |

Statistical evidence varies considerably across the dataset, which means `padj` provides a
genuinely distinct exploration dimension from effect magnitude — not just a filter on top
of it. The application should let researchers explore statistical evidence separately from
response magnitude, rather than collapsing the two into one universal score.

### Effect Magnitude + Statistical Evidence

To see how magnitude and evidence relate, we grouped genes by absolute log2FC and adjusted
p-value (excluding genes with missing `padj`):

| Absolute log2FC | <0.001 | 0.001–0.01 | 0.01–0.05 | 0.05–0.1 | 0.1–1 |
|---|---:|---:|---:|---:|---:|
| <0.25 | 2 | 6 | 73 | 130 | 13,535 |
| 0.25–0.5 | 132 | 266 | 530 | 433 | 1,156 |
| 0.5–1 | 503 | 237 | 268 | 132 | 15 |
| 1–2 | 346 | 71 | 98 | 4 | 0 |
| >2 | 138 | 34 | 8 | 0 | 0 |

The dataset clearly contains several distinct response patterns. 13,535 genes have small
effects (`<0.25`) and weak evidence (`padj > 0.1`) — the uninteresting bulk. But 138 genes
combine a large effect (`>2`) with very strong evidence (`padj < 0.001`), and 503 genes
combine only a moderate effect (`0.5–1`) with that same strong evidence. Effect size and
statistical evidence clearly aren't measuring the same thing, so a single "top genes"
ranking would hide real differences in how genes are being prioritized — the application
needs to support multiple exploration lenses.

### Expression Distribution

`baseMean` is highly right-skewed:

| Statistic | baseMean |
|---|---:|
| Minimum | 1.08 |
| 10th percentile | 2.47 |
| 25th percentile | 8.15 |
| Median | 109.10 |
| 75th percentile | 627.76 |
| 90th percentile | 1,745.40 |
| 99th percentile | 12,116.52 |
| Maximum | 325,654.62 |
| Mean | 972.36 |

The mean sits well above the median, meaning a relatively small number of genes carry
extremely high expression values. If expression is visualized, a raw linear scale would
make most genes hard to distinguish — the interface should use a transformed or logarithmic
view when displaying `baseMean`.

### Estimate Uncertainty

`lfcSE` describes the standard error of the estimated log2 fold change:

| Statistic | lfcSE |
|---|---:|
| Minimum | 0.058 |
| 25th percentile | 0.136 |
| Median | 0.216 |
| 75th percentile | 0.256 |
| Maximum | 3.065 |

Most estimates carry a relatively small standard error, but some are substantially more
uncertain. `lfcSE` should stay visible whenever a scientist examines an individual gene, and
could support a dedicated view comparing effect magnitude against uncertainty.

### Extreme Responses

The genes with the largest absolute log2 fold changes:

| Gene | log2FC | lfcSE | padj |
|---|---:|---:|---:|
| ENSG00000179593 | -11.09 | 3.07 | 2.50e-16 |
| ENSG00000109906 | -7.10 | 0.50 | 7.35e-44 |
| ENSG00000132518 | -6.64 | 2.65 | 3.72e-05 |
| ENSG00000250978 | -6.16 | 0.66 | 1.34e-18 |
| ENSG00000265702 | -5.56 | 2.59 | — |
| ENSG00000127954 | -4.95 | 0.94 | 2.10e-07 |
| ENSG00000128285 | +4.91 | 1.36 | 8.36e-05 |
| ENSG00000100033 | -4.80 | 0.63 | 3.03e-13 |
| ENSG00000152583 | -4.58 | 0.21 | 1.75e-100 |
| ENSG00000168481 | -4.56 | 0.88 | 8.99e-07 |

Extreme effect estimates can carry very different levels of uncertainty — an extreme
negative log2FC doesn't necessarily come with the same precision as another extreme
response. A gene-level view should present effect magnitude together with its supporting
measurements, not fold change alone.

### Exploration Dimensions Identified

The existing data supports several legitimate exploration questions:

| Scientist question | Measurement |
|---|---|
| What changed the most? | `abs(log2FoldChange)` |
| Which genes increased? | `log2FoldChange > 0` |
| Which genes decreased? | `log2FoldChange < 0` |
| Where is statistical evidence strongest? | `padj` |
| Which genes have higher expression? | `baseMean` |
| How certain is an estimated response? | `lfcSE` |
| Where do large effects and strong evidence overlap? | `log2FoldChange` + `padj` |
| How does expression relate to response? | `baseMean` + `log2FoldChange` |
| How does uncertainty relate to response? | `lfcSE` + `log2FoldChange` |

These are exploration dimensions, not biological classifications.

### What the Data Does Not Establish

The current dataset provides gene-level differential expression measurements. On its own,
it does not establish biological importance, causal mechanisms, therapeutic efficacy,
drug-target validity, clinical relevance, safety or toxicity, pathway activation or
inhibition, or disease outcomes. Those questions need additional datasets, annotations,
literature, or pathway analysis — covered separately in `07_data_to_feature_mapping.md`.

### Data-Derived Views vs. Biological Interpretation

A central product distinction is between the information contained in the existing output
and the interpretation a scientist applies to it:

```
Existing measurements
baseMean, log2FoldChange, lfcSE, pvalue, padj
        ↓
Application-generated views
Largest observed responses, strongest statistical evidence,
upregulated/downregulated responses, expression distribution,
effect + evidence patterns, effect + uncertainty patterns
        ↓
Scientist interpretation
What biological process could explain this? Is this consistent with
existing literature? Could this be relevant to the drug mechanism?
Does this warrant further investigation?
```

The application should clearly distinguish these layers rather than blurring them together.

### Preliminary Product Requirements

Based on this analysis, the initial product requirements are:

- **R1 — Dataset Overview:** Provide an immediate overview of the 22,008-gene result set.
- **R2 — Response Landscape:** Allow researchers to understand the overall distribution of
  transcriptional responses.
- **R3 — Multiple Exploration Lenses:** Support exploration through response magnitude,
  response direction, statistical evidence, expression context, and uncertainty.
- **R4 — Preserve Underlying Measurements:** Don't replace the original measurements with
  opaque scores or categories.
- **R5 — Progressive Exploration:** Support movement from overall pattern → subset → gene →
  underlying measurements.
- **R6 — Avoid Arbitrary Importance Rankings:** Don't present a universal "most important
  genes" ranking without a scientific basis.
- **R7 — Explicit Missing Data:** Missing statistical values should stay identifiable and
  shouldn't automatically read as weak evidence.
- **R8 — Gene Search as Secondary Interaction:** Gene search is useful, but the primary
  interface should help researchers discover patterns before they know which genes they
  want to investigate.

### Next Step

The next phase is understanding the scientist workflow around this output: what a
researcher currently does when they receive a DGE result table, what decisions they need to
make, how they decide which results warrant further investigation, what information they
need alongside a gene-level result, what happens after a candidate gene or pattern is
identified, where spreadsheets and manual interpretation create friction, what the
application should surface automatically, and what should stay under scientist control.

**Next artifact:** `04_fde_data_reasoning.md`
