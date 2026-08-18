# 04 — Data Reasoning

## From Existing Bioinformatics Output to Product Understanding

### Purpose

This document captures the reasoning process used to move from an existing
bioinformatics output toward a scientist-facing product concept.

The starting point is an existing Differential Gene Expression (DGE) result from an
RNA-seq experiment comparing human airway smooth muscle cells treated with
dexamethasone against untreated cells. The result contains approximately 22,000 genes and
five measurements per gene: `baseMean`, `log2FoldChange`, `lfcSE`, `pvalue`, `padj`.

The objective at this stage is **not** to generate a new biological finding. The objective
is to understand:

1. What information the existing analysis provides
2. What that information means
3. How that information is distributed in this dataset
4. What a scientist might need to do with it
5. Where the existing output creates friction
6. What those observations imply for a potential product

Python and pandas were used to inspect the data and answer these questions. The analysis
therefore acts as a way to investigate the client's data and workflow, rather than as the
end product itself.

---

## 1. Start With the Existing Scientific Output

The client already has a differential-expression analysis: a table of ~22,000 genes, where
each row is a gene and the columns describe its response to dexamethasone.

Before deciding what the application should do with the table, the guiding question is:

> If a pharmaceutical research team handed us this output, what would we need to
> understand before deciding how scientists should interact with it?

This prevents jumping immediately to a UI such as a searchable table, filter panel, or
dashboard.

## 2. Understand What Each Measurement Represents

| Measurement | What it tells us |
|---|---|
| `baseMean` | How much expression signal is present |
| `log2FoldChange` | Direction and magnitude of the estimated response |
| `lfcSE` | Uncertainty around the estimated response |
| `pvalue` | Statistical evidence against no difference |
| `padj` | Statistical evidence after accounting for multiple testing |

These measurements are not interchangeable — a scientist may want to consider several of
them at once.

### 2.1 `baseMean` — how much expression signal is present?

`baseMean` is the average normalized expression level for a gene across samples. A gene
with `baseMean = 10` and a gene with `baseMean = 100,000` have very different expression
levels — but that does not mean the second is biologically more important, only that it
has much more expression signal in this experiment. A highly expressed gene may show
almost no change after treatment; a lower-expressed gene may show a large estimated
response. **`baseMean` describes the amount of expression signal, not biological
importance.**

### 3. `log2FoldChange` — how much did expression change?

The sign indicates direction (positive = higher expression under dexamethasone, negative =
lower). The magnitude is on a log2 scale: log2FC = +1 ≈ 2-fold increase, +2 ≈ 4-fold
increase, -1 ≈ 2-fold decrease, -2 ≈ 4-fold decrease. It tells us direction and magnitude,
but not precision or statistical strength — **a large fold change alone does not
characterize a result.**

### 4. `lfcSE` — how precise is the estimated change?

The standard error of the estimated log2 fold change. Two genes can share `log2FC = 2.0`
while one has `lfcSE = 0.1` (precise) and the other `lfcSE = 1.5` (imprecise). **Effect
size and uncertainty are different dimensions of a result** — ranking by absolute fold
change alone can surface large estimates without showing how confidently they were
estimated.

### 5. `pvalue` — how strong is the statistical evidence?

Answers: if there were actually no difference between conditions, how surprising would a
result this extreme be? A smaller p-value is stronger evidence against "no difference," but
it does not tell us the size of the biological effect, whether the gene is biologically
important, whether the result is clinically meaningful, or whether dexamethasone directly
caused the change. **Statistical strength and biological effect size are not the same
thing.**

### 6. `padj` — why statistical evidence must account for thousands of genes

With ~22,000 genes tested simultaneously, using `p < 0.05` on every gene independently
would produce apparently significant results simply by chance. `padj` adjusts for this
multiple-testing problem:

- `pvalue` → how surprising is this result under the no-difference assumption?
- `padj` → how strong is that evidence after accounting for the number of genes tested?

For genome-scale DGE analysis, the adjusted p-value is the more decision-relevant
statistic.

### 7. Why no single measurement defines an "important" gene

There is no column that says `important = TRUE`. Each measurement describes a different
aspect of the result — expression signal, effect size and direction, precision, and
statistical evidence (raw and adjusted). A scientist may care about combinations, e.g.
"large effect + strong evidence" is a different observation from "large effect + high
uncertainty" or "small effect + extremely strong evidence." **The application should be
careful about creating a single unexplained score (e.g. `Importance = 94`) unless there is
a scientifically justified reason.**

## 8. Determine the Scale of the Output

The dataset contains 22,008 genes. A scientist could technically sort/filter the table, but
manually reviewing 22,000 rows is not a practical primary exploration strategy. First
product question: **how can a scientist orient themselves within a result set of this
size?** Before assuming the answer is "a better search box," we need to understand the
structure of the response.

## 9. Examine the Distribution of Treatment Responses

| log2FC range | Number of genes |
|---|---:|
| < -2 | 109 |
| -2 to -1 | 281 |
| -1 to -0.5 | 591 |
| -0.5 to 0 | 9,586 |
| 0 to 0.5 | 10,560 |
| 0.5 to 1 | 564 |
| 1 to 2 | 243 |
| > 2 | 74 |

Most genes fall between roughly -0.5 and +0.5; a much smaller group has larger changes.
This gives a first picture of the response landscape — an application could help the
scientist understand the overall distribution before inspecting individual genes. It still
doesn't tell us which genes deserve attention; that requires looking at effect size *and*
statistical evidence together.

## 10. Compare Effect Size With Statistical Evidence

A gene can have: a large effect and strong evidence; a moderate effect and strong evidence;
a large effect with substantial uncertainty; or a relatively small effect with very strong
evidence. The biological interpretation differs across these cases — a very large estimate
with substantial uncertainty should be investigated differently from a large, precisely
estimated effect.

**Product principle:** the application should expose relationships between statistical
measurements rather than automatically treating one measurement as the definition of
importance.

## 11. Examine Expression Level

| Statistic | baseMean |
|---|---:|
| Minimum | 1.08 |
| 25th percentile | 8.15 |
| Median | 109.10 |
| 75th percentile | 627.76 |
| 90th percentile | 1,745.40 |
| 99th percentile | 12,116.52 |
| Maximum | 325,654.62 |

Mean (~972) >> median (~109): a small number of highly expressed genes pull the mean up.
On a linear scale, extreme values would dominate any visualization. **Product question:**
how should expression be represented so the scientist sees the overall distribution without
a handful of extreme values dominating the interface?

## 12. Examine Uncertainty

`lfcSE` ranges from ~0.058 to ~3.065, median ~0.216 — estimated fold changes have varying
precision. A result like `log2FC = -5` should not be presented without the ability to
inspect its uncertainty. A gene-level view should preserve the relationship between effect,
uncertainty, statistical evidence, and expression context — not display fold change as an
isolated number.

## 13. Examine Missing Values

22,008 total rows, 18,117 non-missing `padj` values — not every row has a complete set of
statistical measurements. A missing value should **not** automatically be interpreted as
"not significant" or "no biological response." The interface should preserve the
distinction between *measurement unavailable* and *measurement indicates weak evidence* —
important for scientific trust.

## 14. Look at the Most Extreme Results

`ENSG00000179593` (log2FC = -11.09, lfcSE = 3.07) has an extremely large estimated effect
but also a relatively large standard error. `ENSG00000152583` (log2FC = -4.58, lfcSE =
0.21) has a smaller estimated effect but a much smaller standard error. Both are
interesting, but tell us different things — the application should let a scientist see
these distinctions rather than reducing both to "more important" / "less important" without
context.

## 15. Reframe the Scientist's Problem

Initial framing: *"Scientists have a large DGE table and need to find interesting genes."*
More precise framing after the data assessment: the scientist is navigating multiple
dimensions at once — expression (`baseMean`), effect and direction (`log2FoldChange`),
uncertainty (`lfcSE`), and statistical evidence (`pvalue`/`padj`). The challenge is not
simply finding a row; it's understanding which regions of this multidimensional result set
deserve further exploration.

## 16. From Search to Exploration

"Build a search tool for 22,000 genes" assumes the scientist already knows what they're
looking for. Instead, a scientist may begin with *"What happened in this experiment?"*,
progress to *"What stands out?"*, and eventually *"What should I investigate?"* — an
interaction model of:

```
Entire DGE result → Overall response landscape → Surface informative patterns →
Scientist chooses a direction → Explore a subset → Inspect individual genes →
Add biological context → Determine follow-up
```

Search remains useful, but becomes one part of a larger exploration workflow.

## 17. What Should the Application Surface?

- **Response magnitude:** strongest positive/negative responses, overall effect-size
  distribution
- **Statistical evidence:** strongest adjusted signals, relationship between effect and
  evidence
- **Expression context:** distribution of expression, unusually high/low expressed genes
- **Uncertainty:** large effects with low uncertainty, large effects with high uncertainty

These are exploration entry points, not scientific conclusions. *"These genes have the
largest observed responses"* is directly supported by the data; *"These are the most
biologically important genes"* requires additional scientific context.

## 18. What Should Remain Under Scientist Control?

The dataset alone cannot determine which genes are biologically important, which are
therapeutic targets, which pathways are relevant, whether a change is clinically
meaningful, whether dexamethasone directly caused the change, or whether a finding should
influence a drug-development decision. **The application should help scientists navigate
evidence, not replace scientific judgment.**

## 19. What We Have Learned About the Product

```
Initial interpretation: 22,000 genes → need search/filter

Current interpretation: 22,000 genes → multiple statistical dimensions →
different possible ways to define an interesting result → scientist may not
know what to search for → need orientation and pattern discovery →
allow progressive exploration → support deeper investigation
```

The product opportunity is broader than a more convenient DGE table: help scientists move
from large analytical output → informative observations → results worth investigating
further.

## 20. The FDE Reasoning Pattern

A recurring pattern ran through the analysis:

```
Question → Why do we need to know this? → Inspect the data →
What did we learn? → Why does that matter to the scientist? →
What product question does it create?
```

Examples:

- *How large is the result set?* → 22,008 genes → manual inspection is impractical → how
  can scientists orient themselves? → consider overview-level exploration.
- *Are the largest fold changes automatically the most useful results?* → compare `log2FC`
  with `lfcSE` and `padj` → effect size, uncertainty, and evidence describe different
  dimensions → a single ranking could hide important context → expose multiple dimensions
  for exploration.
- *Are missing `padj` values equivalent to non-significant results?* → inspect missingness
  → some results lack adjusted p-values → missing information must remain distinguishable
  from negative evidence → preserve data provenance and uncertainty in the interface.

The Python analysis matters because it lets us answer these questions with evidence — but
the goal isn't to perform data analysis for its own sake. The goal is to progressively
understand: **what problem is the scientist actually experiencing, and what should we build
to address it?**

## 21. Current Product Hypothesis

> Scientists would benefit from an interactive experience that helps them understand the
> overall transcriptional response, surfaces informative patterns across multiple
> statistical dimensions, and allows them to progressively explore from patterns to
> subsets to individual genes.

```
Global response → Pattern → Subset → Gene → Biological context → Follow-up question
```

The interface should preserve access to the underlying measurements throughout. The
application should not replace the scientist's interpretation — it should reduce the effort
required to get from *"I have 22,000 results"* to *"This is something I want to
investigate."*

## 22. Questions for the Next Stage

**Workflow**
- What does a computational biologist or researcher typically do after receiving a DGE
  result?
- Do they begin with a predefined hypothesis or explore the results broadly?
- What makes them decide that a result deserves attention?

**Exploration**
- What patterns would be useful to surface automatically?
- Which combinations of measurements are most informative?
- How should scientists move from a pattern to the underlying genes?

**Downstream interpretation**
- What information do scientists need after identifying a candidate gene? Pathway
  databases? Gene annotations? Literature? Drug-target databases? Internal experimental
  data? Other omics datasets?

**Product boundaries**
- What can be derived directly from the current dataset? What requires external biological
  knowledge? Where could machine learning help? Where could AI introduce ambiguity? What
  evidence needs to remain visible for the scientist to trust the result?

These questions define the next stage: **how should a scientist actually move through this
data?**

**Next artifact:** `05_scientist_workflow.md`
