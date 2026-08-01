# AGENTS.md — Vinvite Synthetic Voter Dataset Generator

## Project Purpose

A synthetic voter dataset generator that produces realistic, entirely
fake campaign voter export files for testing the Vinvite Campaign Data
Engine (Import Wizard, Column Mapping Engine, Data Quality Engine,
Household Detection, Duplicate Detection, Language Estimation, Campaign
Readiness Report). It generates 10 distinct datasets, each modeling a
different real-world data-quality scenario, plus a reusable column
synonym dictionary. No real voter data is ever used or produced.

## Authoritative Documents

Read these in order before implementing anything:

1. `README.md`
2. `ARCHITECTURE.md`
3. `SCHEMA.md`
4. `DATASET_SPECS.md`
5. `IMPLEMENTATION_PLAN.md`
6. `TASKS.md`
7. `CODING_STANDARDS.md`

## Scope Rules

- Implement only the assigned task.
- Do not implement future tasks, even if they seem obvious or trivial to include.
- Do not change architecture or schema without approval.
- Do not invent missing requirements — if something is unspecified, ask or report it rather than guessing.
- Report conflicts or ambiguity before proceeding.
- Avoid unrelated refactoring, even in files you're already touching.

## Task Execution Rules

For each assigned task, do the following, in order:

1. Read the assigned task in `TASKS.md` and confirm every task listed in its Dependencies field is already complete (merged).
2. Inspect the relevant sections of `ARCHITECTURE.md`, `SCHEMA.md`, and `DATASET_SPECS.md` that the task touches.
3. Implement only the assigned task — nothing from adjacent or future tasks.
4. Add or update tests per the task's "Unit Tests Required" field.
5. Run all required checks (see `CODING_STANDARDS.md` § Required Checks Before Completion).
6. Update documentation only where the task explicitly requires it.
7. Summarize changes and stop for review. Do not chain into the next task automatically.

## Testing Rules

- No task is complete with failing tests.
- New functionality must have tests.
- Generated datasets must be deterministic when a seed is provided — the same seed and config must always produce byte-identical output.
- Tests must not use real voter data. All test fixtures are synthetic.

## File and Data Rules

- Do not commit secrets (API keys, credentials, tokens).
- Do not include real voter information anywhere — code, comments, test fixtures, or examples.
- Do not commit generated datasets unless the task explicitly requires test fixtures (e.g. golden-snapshot regression fixtures).
- Preserve the canonical schema defined in `SCHEMA.md`; do not add, remove, or rename fields without approval.
- Use `column_synonyms.json` rather than hard-coding mapping synonyms anywhere in the codebase.

## Completion Report

At the end of every task, report:

- Task completed (Task ID and title)
- Files created or modified
- Tests added
- Test results (pass/fail, coverage where applicable)
- Assumptions made, if any
- Unresolved issues, if any
- Confirmation that no future task was implemented

## Stop Conditions

Stop and ask before proceeding if:

- Documents conflict with each other.
- A dependency the task relies on is incomplete.
- The task appears to require an architecture or schema change.
- Acceptance criteria are unclear or ambiguous.
- The task requires a product decision not documented anywhere in the repository.
