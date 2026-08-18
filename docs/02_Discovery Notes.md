# 02 — Discovery Notes

## What to Ask Before (and Alongside) the Data

### Why this document exists



This document is organized around the same six-stage sequence `01` names as the project's
FDE workflow. Each stage lists the questions that would normally get asked *at* that
stage — before assuming, inferring, or defaulting.

```
Client Context → Data → User Workflow → Requirements → Prototype → Evaluation
     ↓               ↓          ↓              ↓              ↓            ↓
   below           below      below          below          below       below
```

---

### 1. Client Context

*Sits underneath `01`. Gets the scope right before anything else starts.*

| Question | What it de-risks |
|---|---|
| What does success look like to you, concretely? | Not "a better dashboard" — what would make a scientist say this saved them time this week. Unasked, you build the technically impressive thing instead of the used thing. |
| Walk me through the last time someone used a DGE result to make a decision. | A real recent example beats an abstract workflow description — surfaces actual friction, not assumed friction. |
| What have you already tried? | Existing spreadsheet templates, R Shiny apps, prior vendor tools. Skipping this risks rebuilding something already rejected. |
| Who is this for, specifically? | A bench scientist, a computational biologist, a PI, and a regulatory team all want different things from the same table. "Researchers" is not one persona. |

### 2. Data

*Sits underneath `03`. Determines whether the numbers mean what the analysis assumes.*

| Question | What it de-risks |
|---|---|
| Where does the authoritative experimental metadata live, and how is it attached to each result? | If metadata isn't reliably linked, the "experiment overview" screen in `09` is built on sand. |
| Is this the only comparison, or one of several? | If there are multiple treatments/doses/timepoints, the single-comparison model in `08` needs to become a comparison model much sooner. |
| How was this table produced, and is that pipeline still the source of truth? | Determines whether the app owns a frozen snapshot or needs to stay pointed at a live pipeline. |
| Are there QC flags, replicate counts, or batch effects to know about? | `03` currently treats all 22,008 rows as equally trustworthy — a real conversation would confirm or correct that. |

### 3. User Workflow

*Sits underneath `05`. Confirms the workflow was observed, not assumed.*

| Question | What it de-risks |
|---|---|
| What annotation sources do your scientists already trust and use? | Picking an unfamiliar source, even a better one, adds adoption friction. |
| Are there internal, non-public knowledge bases that should factor in? | Changes both scope and data-governance requirements substantially. |
| Should scientists jump from a gene straight into literature, or do they already have a tool for that? | Avoids building a redundant literature search. |
| What's the next action after someone flags a gene as interesting? | Determines whether "export/share" (`01`, V1 scope) is enough, or whether the tool needs to integrate with wherever that next action lives. |

### 4. Requirements

*Sits underneath `06`. Turns workflow observations into a scoped, defensible set of
requirements instead of a guessed one.*

| Question | What it de-risks |
|---|---|
| Are scientists trying to identify drug-responsive genes, compare compounds, validate targets, or explore mechanism? | These are different jobs wearing the same UI — the "which lens matters most" call in `06` should be informed, not guessed. |
| Will this need to compare across other experiments eventually? | Reshapes the data model from day one, even though multi-experiment comparison is out of scope for `09`'s V1. |
| How much do scientists need to trust a number before acting on it — what would break that trust? | Calibrates how strict the traceability requirement in `06` needs to actually be. |
| If this tool is wrong or misleading once, what's the cost? | Sets how conservative the "don't imply importance" boundary in `06`/`07` needs to be. |

### 5. Prototype

*Sits underneath `08` and `09`. Determines what's actually buildable and where.*

| Question | What it de-risks |
|---|---|
| Is there a regulatory or compliance reason this data can't leave a certain environment? | Directly affects deployment architecture (`01`'s V8) and whether a hosted demo is even viable. |
| Which platform constraints are real vs. assumed — internal tooling, browser restrictions, existing auth? | Avoids designing screens in `09` around infrastructure that doesn't exist. |
| Who needs to review or approve the interface before scientists see it? | Determines whether the prototype needs a review cycle built into the timeline. |

### 6. Evaluation

*Feeds back into the loop — what "working" means once something ships.*

| Question | What it de-risks |
|---|---|
| How would you know if this tool actually changed how someone works? | Without a metric here, "success" in `01` stays untestable after launch. |
| What would make a scientist stop using this after week one? | Surfaces the failure mode worth designing against now, not after churn. |
| What's the next capability you'd want, if this V1 works? | Validates the V2–V9 roadmap in `01` against real appetite instead of an assumed one. |

---

### What happens if these go unasked

Every document from `03` onward in this case study is a reasonable substitute for these
conversations, not a replacement for them. The inferences hold up because the dataset was
well-behaved and every assumption was stated explicitly — but in a real engagement, several
would likely be wrong. A client conversation at the "Client Context" stage alone would
probably surface, within the first hour, several things that took multiple passes of data
analysis in `03` to approximate.

That's the point of this document: data analysis builds confidence in an answer you don't
have a client to give you. It isn't a substitute for asking. The skill on display across
this case study isn't "can extract patterns from a CSV" — it's knowing which questions
would have made most of that extraction unnecessary.
