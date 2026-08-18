# DGE Exploration — FDE Case Study

This repo/folder restructures a 150-page planning brainstorm into a clean, numbered case
study, plus a portfolio-ready narrative on top.

## Start here

- **[`PORTFOLIO_CASE_STUDY.md`](PORTFOLIO_CASE_STUDY.md)** — the recruiter/hiring-manager
  facing narrative. Read this first if you're seeing this project for the first time.

## The underlying case study

- **[`docs/`](docs/)** — the 9 working documents, cleaned up and renumbered 00–08 (the
  original brainstorm had a gap between doc 2 and doc 4; that's fixed here):

  1. `00_fde_mental_model.md`
  2. `01_project_brief.md`
  3. `02_data_exploration_findings.md`
  4. `03_fde_data_reasoning.md`
  5. `04_scientist_workflow.md`
  6. `05_product_requirements.md`
  7. `06_data_to_feature_mapping.md`
  8. `07_application_user_flow.md`
  9. `08_ui_prototype_specification.md`

Each ends with a pointer to the next one, so they read in order — this is meant to double
as a legible artifact on its own if someone wants to see the full reasoning chain, not just
the summary.

## What changed from your original PDF

- Split into one file per document instead of one long PDF.
- Stripped the collaborative back-and-forth / meta-commentary ("yes, I think we should
  write it as markdown...") so each file reads as a clean deliverable, not a chat log.
- Renumbered sequentially (00–08) and fixed inconsistent heading styles.
- Normalized tables, diagrams, and section headers to consistent Markdown.
- Added `PORTFOLIO_CASE_STUDY.md` — this didn't exist in your draft. It's the piece
  written specifically to be skimmed by a recruiter or hiring manager in ~90 seconds, with
  the technical depth still one click away in `docs/`.

## Suggested next move

The project is currently a very strong *spec* — it stops right before implementation. For
a portfolio piece aimed at FDE/Solutions Architect roles, the single highest-leverage next
step is building the V1 prototype described in `08_ui_prototype_specification.md` and
linking a live demo (or screen recording) from the top of `PORTFOLIO_CASE_STUDY.md`. See
that file's "Next steps" section for a short prioritized list.
