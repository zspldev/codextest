# CODING_STANDARDS.md — Vinvite Synthetic Voter Dataset Generator

> **Note:** `ARCHITECTURE.md` deliberately stayed language-agnostic (an
> explicit pre-implementation decision, since the language/stack was an
> open question at the time). The stack below was decided by the Product
> Owner during the `AGENTS.md`/`CODING_STANDARDS.md` review pass:
> **Python**. This document is the source of truth for that decision.

## Runtime and Dependencies

- **Language/runtime:** Python 3.11 or later.
- **Package manager:** `pip` with a `pyproject.toml` (PEP 621) for project metadata and dependencies. Use a virtual environment (`venv`) for all local and CI work — never install into system Python.
- **Dependency policy:** prefer the standard library. Add a third-party runtime dependency only when it removes meaningful risk or effort (e.g. `pytest` for testing is a dev dependency, not a runtime one). Pin exact versions in `requirements.txt` / `requirements-dev.txt`, generated from `pyproject.toml` via `pip-compile` (pip-tools). No unpinned or floating (`>=` only) versions in committed lockfiles.

## Code Style

- **Formatter:** `black`, default settings (line length 88).
- **Linter:** `ruff`, configured in `pyproject.toml`. Zero warnings on merge.
- **Naming conventions:** PEP 8 — `snake_case` for functions/variables/modules, `PascalCase` for classes, `UPPER_SNAKE_CASE` for constants. Canonical schema field names (`VoterID`, `FirstName`, etc.) are the one deliberate exception, since they must match `SCHEMA.md` exactly.
- **Type hints:** required on all public function signatures (parameters and return type). Internal/private helper functions should have them where they meaningfully aid readability.
- **Docstrings:** Google-style docstrings required on every public module, class, and function. One-line summary minimum; expand with Args/Returns/Raises for anything non-trivial.
- **Separation of concerns:** generation, mutation (defect injection), mapping, export, and metrics stay in their own modules per `ARCHITECTURE.md` §3 — a function in `core/record_generator` should never directly write CSV or apply defects, and vice versa.

## Project Structure

- **Module organization:** follow `ARCHITECTURE.md` §3 exactly — `core/`, `profiles/`, `dictionaries/`, `dataset_configs/`, `pipeline/`, `tests/`. Do not introduce new top-level directories without flagging it (see `AGENTS.md` § Stop Conditions).
- **Public vs. internal modules:** a module's public surface is whatever's exported via `__all__`; everything else is internal. Prefix internal-only helper functions with a leading underscore.
- **Configuration location:** dataset configs live under `dataset_configs/*.config`; the config format spec and loader live under `dictionaries/` and `pipeline/` respectively, per `IMPLEMENTATION_PLAN.md` Epic E1.3.
- **Test location:** `tests/`, mirroring the module under test (e.g. `core/record_generator` → `tests/test_record_generator.py`).

## Error Handling

- No silent failures — never swallow an exception with a bare `except: pass`.
- Config and schema validation errors must be specific: name the field, the value received, and the rule violated (e.g. `"ZIP must be 5 digits, got '99999999'"`, not `"invalid config"`).
- Use custom exception classes for domain errors (e.g. `SchemaValidationError`, `ConfigValidationError`) rather than raising bare `Exception` or `ValueError` everywhere — this lets `AGENTS.md`'s stop-condition handling and tests distinguish error types.
- Catch exceptions only where you can meaningfully handle or re-raise them with more context; let unexpected errors propagate.

## Logging

- Use the standard library `logging` module — no `print()` statements in library/pipeline code (CLI output via `print` is fine at the `pipeline/cli` entrypoint only).
- Standard levels: `DEBUG` for per-record detail, `INFO` for per-dataset progress (e.g. "Generated 2000 records for Dataset 10"), `WARNING` for recoverable issues, `ERROR` for failures that stop a run.
- Never log PII-shaped values at any level — since all data is synthetic this is about defense-in-depth (catching a real-data leak early) as much as it is about clean logs. Log counts and field names, not full record contents, at `INFO` and above.

## Configuration

- Dataset parameters (record count, defect rates, seed, column-mapping style) live in `dataset_configs/*.config` files, loaded and validated per `IMPLEMENTATION_PLAN.md` Epic E1.3 — not hard-coded in Python.
- Random seed is always an explicit, required parameter to the top-level generation call — never a default or implicit global seed.
- Output paths are configurable (CLI argument or environment variable, e.g. `VINVITE_OUTPUT_DIR`), defaulting to a local `output/` directory.
- Country profile selection is an explicit config parameter (e.g. `profile: us_california`), read through the `CountryProfile` interface — never imported directly by name in generation logic.
- No secrets of any kind belong in source code or config files. This project has none by design (no real data, no external API calls), but the rule stands for any future integration.

## Data Handling

- All file I/O is UTF-8, explicit (`encoding="utf-8"` on every open call — never rely on platform default).
- CSV: use the standard library `csv` module with `QUOTE_MINIMAL` and standard comma delimiter; never hand-build CSV rows with string concatenation, especially given Dataset 10's embedded-comma/quoted-value noise (`T4.4.5`) — the `csv` module's quoting must be trusted to round-trip these correctly.
- JSON: `json` module, UTF-8, `ensure_ascii=False` so multilingual names/text serialize as readable Unicode rather than escape sequences.
- Dates: ISO 8601 (`YYYY-MM-DD`) everywhere, matching `SCHEMA.md`.
- Deterministic generation: every module that uses randomness must draw from the single seeded RNG wrapper (`T2.2.2`) — never `random` module calls outside that wrapper, never `datetime.now()` or other non-deterministic inputs in generation logic.
- No real voter data anywhere — this includes test fixtures, docstring examples, and code comments.
- California-specific logic (city lists, ZIP patterns, phone formats, etc.) must stay inside `profiles/us_california/` — nothing in `core/` or `pipeline/` may reference California by name, per the `CountryProfile` abstraction in `ARCHITECTURE.md` §5.

## Testing

- **Framework:** `pytest`.
- **Unit tests:** every module in `core/`, `profiles/`, and `pipeline/` has a corresponding test file with meaningful coverage of its public functions — not just a smoke test.
- **Integration tests:** full dataset generation (`run_dataset(config)`) is tested end-to-end for all 10 dataset configs, per `IMPLEMENTATION_PLAN.md` Epic E6.2 / `TASKS.md` TASK-077.
- **Deterministic tests:** any test involving generation must fix the seed explicitly and assert exact or statistically-toleranced output — never assert on a run without a fixed seed.
- **Validation of expected metrics:** metrics-engine tests assert exact ground-truth match against known injection state (per `IMPLEMENTATION_PLAN.md` Epic E5.1) — not approximate or "close enough" comparisons, except where a task's spec explicitly calls for statistical tolerance (e.g. missingness rates).
- **Performance test:** once record counts are scaled toward the ~100,000-voter target (see `DATASET_SPECS.md`'s scale-up note), add a performance test asserting full-suite generation completes within an agreed time budget. Not required at the current small-scale pass (8,400 records) but the test harness should be written so it's a config change, not new code, when that scale-up happens.

## Required Checks Before Completion

Every task must pass all of the following before it's considered done:

```bash
# Format check (fails if not already formatted)
black --check .

# Lint
ruff check .

# Type check
mypy .

# Tests
pytest --cov=core --cov=profiles --cov=pipeline --cov-report=term-missing
```

All four must exit zero. `AGENTS.md`'s Completion Report requires stating the test results explicitly — a task is not complete on "tests probably pass," only on an actual passing run reported.
