# Vinvite Synthetic Voter Dataset Generator — Implementation Plan
**Prepared by:** Chief Software Architect, Zapurzaa Systems (ZSPL)
**Audience:** AI coding agent (Codex) or human engineering team
**Status:** Pre-implementation — to be committed before any production code is written

---

# 1. Project Overview
This plan converts the existing architecture (`ARCHITECTURE.md`, `SCHEMA.md`, `DATASET_SPECS.md`, `column_synonyms.json`) into a buildable execution roadmap. It is intentionally free of source code — every task below describes *what* to build and *how it will be verified*, not implementation detail.

**Build order and rationale.** The system is built bottom-up, in strict dependency order:

1. **Foundation first** (Phase 1) — the canonical schema and config system are consumed by literally every later module, so they must be stable before anything else starts.
2. **Generation before mutation** (Phases 2–3) — you cannot inject defects into records that don't exist yet, and the `CountryProfile` abstraction must be proven with one concrete profile (`us_california`) before defect logic is layered on top.
3. **Mutation before metrics** (Phases 4–5) — metrics are defined as ground truth *derived from* injection parameters (per `ARCHITECTURE.md`'s correct-by-construction principle), so the injectors must exist first.
4. **Export and orchestration last among 'build' phases** (Phase 6) — this is where every prior module is wired together into the one command that actually produces the 10 dataset files.
5. **Validation, regression, and documentation close the loop** (Phases 7–9) — proving the system does what it claims, locking that in as an ongoing CI gate, and making the result maintainable by someone who didn't build it.

This ordering also maximizes early testability: by the end of Phase 2, there is already a working, testable (if region-less) generation core; by the end of Phase 4, every defect type is independently verifiable before they're ever combined.

---

# 2. Engineering Principles
- Build incrementally — every phase must leave the system in a working, testable state.
- Keep modules independent — a change to one injector or provider must not require touching unrelated modules.
- Every task must be testable — no task in this plan is marked done without an associated test or explicit manual-verification step.
- Every Epic should produce a working increment — not just code, but something demonstrably runnable or verifiable.
- No task should require more than one pull request — tasks are sized so a single PR can complete one task's Definition of Done.
- Prefer reusable components over duplication — e.g. one seeded RNG wrapper (T2.2.2) used everywhere, not one per module.
- Follow ARCHITECTURE.md at all times — if a task seems to require deviating from the architecture doc, stop and flag it (see §12) rather than improvising.
- Reproducibility is non-negotiable — every task touching randomness must go through the seeded RNG (T2.2.2); this is a repeated theme, not a one-time task.
- Metrics are derived, never inferred — the metrics engine (Phase 5) must compute from known injection state, never by re-detecting patterns in output data.

---

# 3. Project Phases
| Phase | Name | Goal |
|---|---|---|
| 1 | Project Foundation | Stand up the repo skeleton, canonical schema model, and config system that every later phase depends on. |
| 2 | Core Data Generation Engine | Build the CountryProfile contract and the clean-record/household generation pipeline before any defects or region-specific data exist. |
| 3 | Synthetic Data Providers | Build the concrete us_california CountryProfile and wire the column-synonym dictionary into a working column mapper. |
| 4 | Data Mutation Engine | Implement every defect category from DATASET_SPECS.md as an independent, composable injector, per ARCHITECTURE.md's 'defects are parameters' principle. |
| 5 | Metrics Engine | Compute ground-truth metrics directly from generation/injection parameters, never by re-inferring them from output data — the 'correct-by-construction' principle from ARCHITECTURE.md. |
| 6 | Export Engine | Turn generated, mutated, mapped record batches into the actual output files (CSV) for all 10 datasets in one orchestrated run. |
| 7 | Validation | Confirm the generated output actually satisfies DATASET_SPECS.md's stated expectations, not just that the code ran without error. |
| 8 | Regression Test Suite | Lock in reproducibility and correctness as an ongoing CI gate, not a one-time check. |
| 9 | Documentation | Ensure the system is maintainable and extensible by someone who wasn't part of building it. |

---

# 4–5. Epics and Engineering Tasks
Each phase below lists its Epics (purpose, business value, dependencies, deliverables, Definition of Done) followed by that Epic's full task breakdown.

## Phase 1: Project Foundation
**Goal:** Stand up the repo skeleton, canonical schema model, and config system that every later phase depends on.

### Epic E1.1: Repository & Environment Setup
- **Purpose:** Create a working, testable project skeleton before any generation logic exists.
- **Business Value:** Removes setup friction for every subsequent task; establishes conventions once instead of ad hoc.
- **Dependencies:** None (first epic)
- **Expected Deliverables:** Initialized repo layout matching ARCHITECTURE.md module map; dependency manifest; lint/format config; test runner wired to run on an empty test suite.
- **Definition of Done:** A fresh clone can install dependencies, run the (empty) test suite, and run the linter with zero errors.

#### Tasks

**T1.1.1 — Initialize project repository structure**

| Field | Detail |
|---|---|
| Description | Create the folder layout defined in ARCHITECTURE.md §3 (core/, profiles/, dictionaries/, dataset_configs/, pipeline/) as empty modules with placeholder files. |
| Inputs | ARCHITECTURE.md |
| Outputs | Folder/module skeleton committed to repo |
| Dependencies | None |
| Estimated Complexity | Small |
| Definition of Done | Folder structure matches ARCHITECTURE.md exactly; imports resolve with no code yet. |
| Required Unit Tests | N/A (structural task) — add a CI check that the expected folders exist. |
| Potential Risks | Low — risk of drifting from ARCHITECTURE.md if not reviewed against it directly. |

**T1.1.2 — Set up dependency management**

| Field | Detail |
|---|---|
| Description | Choose and configure a package manager/manifest appropriate to the implementation language; pin versions. |
| Inputs | Language/stack decision (open question, see §11) |
| Outputs | Dependency manifest + lockfile |
| Dependencies | T1.1.1 |
| Estimated Complexity | Small |
| Definition of Done | Clean install works from the manifest alone on a fresh machine. |
| Required Unit Tests | CI job that does a clean install and fails on error. |
| Potential Risks | Low — stack ambiguity could cause rework if decided late (see Open Questions). |

**T1.1.3 — Configure linting & formatting**

| Field | Detail |
|---|---|
| Description | Add a linter/formatter config enforcing one consistent style across the codebase. |
| Inputs | T1.1.2 |
| Outputs | Lint/format config files |
| Dependencies | T1.1.2 |
| Estimated Complexity | Small |
| Definition of Done | Lint passes on the empty skeleton; CI fails the build on lint errors. |
| Required Unit Tests | CI lint job. |
| Potential Risks | Low. |

**T1.1.4 — Set up test framework scaffolding**

| Field | Detail |
|---|---|
| Description | Install and configure the unit test framework and a coverage reporter. |
| Inputs | T1.1.2 |
| Outputs | Test runner config; example smoke test |
| Dependencies | T1.1.2 |
| Estimated Complexity | Small |
| Definition of Done | `run tests` command executes successfully with one passing smoke test. |
| Required Unit Tests | The smoke test itself. |
| Potential Risks | Low. |

**T1.1.5 — Set up CI pipeline skeleton**

| Field | Detail |
|---|---|
| Description | Configure CI to run lint + tests on every pull request. |
| Inputs | T1.1.3, T1.1.4 |
| Outputs | CI pipeline config |
| Dependencies | T1.1.3, T1.1.4 |
| Estimated Complexity | Small |
| Definition of Done | A PR with a deliberate lint error or failing test is blocked by CI. |
| Required Unit Tests | Manual verification with a throwaway failing PR. |
| Potential Risks | Low — CI provider access/permissions could block this task; flag if unavailable. |

### Epic E1.2: Canonical Schema Implementation
- **Purpose:** Turn SCHEMA.md's 24-field table into an enforceable data model used by every other module.
- **Business Value:** Guarantees every generated record is structurally valid by construction, not by convention.
- **Dependencies:** E1.1
- **Expected Deliverables:** Canonical record data model; schema validation utility; unit tests.
- **Definition of Done:** Every field in SCHEMA.md is represented with correct type and nullability; an intentionally malformed record fails validation.

#### Tasks

**T1.2.1 — Define canonical schema data structure**

| Field | Detail |
|---|---|
| Description | Implement the 24-field record model exactly as specified in SCHEMA.md, including types and nullability. |
| Inputs | SCHEMA.md |
| Outputs | Canonical record model (single source of truth for the schema) |
| Dependencies | T1.1.1 |
| Estimated Complexity | Medium |
| Definition of Done | All 24 fields present with correct types; non-nullable fields enforced at construction time. |
| Required Unit Tests | Construct a valid record; construct an invalid record missing a required field and assert it is rejected. |
| Potential Risks | Medium — if SCHEMA.md changes later, this task's output must be revisited (single source of truth mitigates drift). |

**T1.2.2 — Implement schema validation utility**

| Field | Detail |
|---|---|
| Description | Standalone validator that checks any record dict/object against the canonical schema, independent of how it was produced. |
| Inputs | T1.2.1 |
| Outputs | Reusable `validate_record()` utility |
| Dependencies | T1.2.1 |
| Estimated Complexity | Small |
| Definition of Done | Validator correctly accepts/rejects a table of valid and invalid test fixtures. |
| Required Unit Tests | Parametrized test covering every field's nullability and type rule. |
| Potential Risks | Low. |

**T1.2.3 — Unit tests for schema model**

| Field | Detail |
|---|---|
| Description | Comprehensive test coverage for the schema model and validator beyond the basic cases above. |
| Inputs | T1.2.1, T1.2.2 |
| Outputs | Test suite for schema module |
| Dependencies | T1.2.2 |
| Estimated Complexity | Small |
| Definition of Done | ≥ 90% line coverage on the schema module. |
| Required Unit Tests | Edge cases: empty strings vs null, boundary date formats, enum violations (Gender, RegistrationStatus). |
| Potential Risks | Low. |

### Epic E1.3: Configuration System
- **Purpose:** Build the mechanism by which each of the 10 datasets is expressed as data, not code (per ARCHITECTURE.md design principle 'defects are parameters').
- **Business Value:** Enables adding an 11th dataset later as a pure config change — the core architectural promise of this project.
- **Dependencies:** E1.1
- **Expected Deliverables:** Config file schema/format; loader; validator; unit tests.
- **Definition of Done:** A sample config file loads into a typed config object; a malformed config fails fast with a clear error, not a silent default.

#### Tasks

**T1.3.1 — Define dataset config file format**

| Field | Detail |
|---|---|
| Description | Specify the schema for a `dataset_configs/*.config` file: record count, defect toggles and rates, column-mapping style, seed. |
| Inputs | DATASET_SPECS.md (10 dataset parameter sets) |
| Outputs | Config format specification |
| Dependencies | T1.1.1 |
| Estimated Complexity | Medium |
| Definition of Done | Format covers every parameter used across all 10 dataset specs without a single one-off hack. |
| Required Unit Tests | N/A (spec task) — validated indirectly via T1.3.2/T1.3.3 tests. |
| Potential Risks | Medium — under-specifying the format now causes rework in Phase 4 when defect injection needs new parameters. |

**T1.3.2 — Implement config loader/parser**

| Field | Detail |
|---|---|
| Description | Parse a config file into the typed config object defined in T1.3.1. |
| Inputs | T1.3.1 |
| Outputs | `load_config()` utility |
| Dependencies | T1.3.1 |
| Estimated Complexity | Small |
| Definition of Done | Loader correctly parses all 10 dataset configs once authored (Phase 1 output feeds Phase 2+ directly). |
| Required Unit Tests | Round-trip test: load a config, assert every field matches the source file. |
| Potential Risks | Low. |

**T1.3.3 — Implement config validator**

| Field | Detail |
|---|---|
| Description | Fail fast on malformed configs (e.g. defect rates outside 0–100%, missing required keys) rather than producing silently wrong data. |
| Inputs | T1.3.1, T1.3.2 |
| Outputs | `validate_config()` utility |
| Dependencies | T1.3.2 |
| Estimated Complexity | Small |
| Definition of Done | A table of malformed sample configs all raise clear, specific errors. |
| Required Unit Tests | Parametrized test with at least one violation per validation rule. |
| Potential Risks | Low. |

**T1.3.4 — Author the 10 dataset config files**

| Field | Detail |
|---|---|
| Description | Translate DATASET_SPECS.md's 10 dataset definitions into actual config files in the format from T1.3.1. |
| Inputs | DATASET_SPECS.md, T1.3.1 |
| Outputs | 10 config files under dataset_configs/ |
| Dependencies | T1.3.3 |
| Estimated Complexity | Medium |
| Definition of Done | All 10 configs pass validation and each round-trips through the loader with values matching DATASET_SPECS.md. |
| Required Unit Tests | Loader test per config file (10 cases) asserting key parameters (record count, defect rates) match the spec doc. |
| Potential Risks | Medium — highest-risk task in this phase for silent transcription errors from the spec doc; recommend a manual review checklist against DATASET_SPECS.md. |

## Phase 2: Core Data Generation Engine
**Goal:** Build the CountryProfile contract and the clean-record/household generation pipeline before any defects or region-specific data exist.

### Epic E2.1: CountryProfile Interface & Contract
- **Purpose:** Formalize the abstract interface from ARCHITECTURE.md §5 as an enforceable contract, not just documentation.
- **Business Value:** Is the single mechanism that makes multi-region support (India, UK, Canada, Australia) a future config/plugin addition rather than a rewrite.
- **Dependencies:** E1.2
- **Expected Deliverables:** Abstract interface definition; a contract test suite any profile implementation must pass.
- **Definition of Done:** The interface exposes exactly the 7 capabilities listed in ARCHITECTURE.md §5; the contract test suite can run against any profile and reports pass/fail per capability.

#### Tasks

**T2.1.1 — Define CountryProfile abstract interface**

| Field | Detail |
|---|---|
| Description | Implement the interface contract: geographies(), name_pools(), phone_formats(), email_domains(), language_distribution(), voter_id_scheme(), address_format(). |
| Inputs | ARCHITECTURE.md §5 |
| Outputs | Abstract interface/base class |
| Dependencies | T1.1.1 |
| Estimated Complexity | Medium |
| Definition of Done | All 7 methods declared with documented expected return shapes. |
| Required Unit Tests | N/A directly (abstract) — covered by T2.1.2. |
| Potential Risks | Low. |

**T2.1.2 — Implement interface contract test suite**

| Field | Detail |
|---|---|
| Description | A reusable test suite that any CountryProfile implementation can be run against to confirm compliance. |
| Inputs | T2.1.1 |
| Outputs | `assert_profile_contract(profile)` reusable test utility |
| Dependencies | T2.1.1 |
| Estimated Complexity | Medium |
| Definition of Done | Running the suite against a deliberately incomplete stub profile fails with a clear per-method error. |
| Required Unit Tests | The suite itself, run against a stub good profile and a stub bad profile. |
| Potential Risks | Low. |

### Epic E2.2: Record Generator
- **Purpose:** Produce single clean, fully-populated canonical records from a given CountryProfile and seed.
- **Business Value:** This is the foundation every dataset (all 10) is ultimately built from; correctness here is load-bearing for the whole system.
- **Dependencies:** E2.1, E1.2
- **Expected Deliverables:** record_generator module; seeded RNG wrapper; VoterID generation; unit tests.
- **Definition of Done:** Given the same seed and profile, two runs produce byte-identical record sets.

#### Tasks

**T2.2.1 — Implement base record_generator**

| Field | Detail |
|---|---|
| Description | Generate N clean, fully-populated canonical records using a supplied CountryProfile's data pools. |
| Inputs | T1.2.1 (schema), T2.1.1 (profile interface) |
| Outputs | `generate_records(profile, n, seed)` function |
| Dependencies | T1.2.1, T2.1.1 |
| Estimated Complexity | Large |
| Definition of Done | Output passes schema validation (T1.2.2) for every record, for N up to 10,000, with zero failures. |
| Required Unit Tests | Schema-conformance test on generated batches of varying size; distribution sanity check (e.g. city values roughly match profile weights). |
| Potential Risks | Medium — largest single task in Phase 2; recommend splitting into per-field generation sub-tasks if complexity proves higher than estimated during implementation. |

**T2.2.2 — Implement seeded RNG wrapper**

| Field | Detail |
|---|---|
| Description | Centralize all randomness behind one seeded RNG so reproducibility (ARCHITECTURE.md design principle) holds across the entire pipeline, not just this module. |
| Inputs | None |
| Outputs | `SeededRNG` utility used by every later randomness-consuming module |
| Dependencies | T1.1.1 |
| Estimated Complexity | Small |
| Definition of Done | Same seed always produces the same sequence of draws across all consumers. |
| Required Unit Tests | Two independent runs with the same seed produce identical draw sequences. |
| Potential Risks | Medium — if any later module (Phase 3/4) bypasses this wrapper and uses ad hoc randomness, reproducibility silently breaks. Recommend a lint rule or code-review checklist item. |

**T2.2.3 — Implement VoterID generation**

| Field | Detail |
|---|---|
| Description | Generate VoterIDs per the profile's voter_id_scheme(), guaranteeing uniqueness within a generation run. |
| Inputs | T2.1.1, T2.2.2 |
| Outputs | `generate_voter_id()` utility |
| Dependencies | T2.1.1, T2.2.2 |
| Estimated Complexity | Small |
| Definition of Done | No collisions across 10,000 generated IDs in a single run. |
| Required Unit Tests | Uniqueness test at scale; format-conformance test against the profile's scheme. |
| Potential Risks | Low. |

**T2.2.4 — Reproducibility unit tests**

| Field | Detail |
|---|---|
| Description | Dedicated test suite proving the whole record_generator is seed-deterministic end to end. |
| Inputs | T2.2.1, T2.2.2, T2.2.3 |
| Outputs | Reproducibility test suite |
| Dependencies | T2.2.1, T2.2.2, T2.2.3 |
| Estimated Complexity | Small |
| Definition of Done | Two full generation runs with the same seed and config are byte-identical when serialized. |
| Required Unit Tests | The reproducibility suite itself. |
| Potential Risks | Low. |

### Epic E2.3: Household Grouper
- **Purpose:** Cluster generated records into households sharing an address, per ARCHITECTURE.md's household_grouper module.
- **Business Value:** Directly required by Dataset 6 (Household Dataset) and by HouseholdID population in every other dataset.
- **Dependencies:** E2.2
- **Expected Deliverables:** household_grouper module; configurable household size distribution; unit-notation noise injector; unit tests.
- **Definition of Done:** Given a configured household rate and size distribution, generated households match the configuration within tolerance, and HouseholdID is consistently assigned to co-resident records.

#### Tasks

**T2.3.1 — Implement address-based household clustering**

| Field | Detail |
|---|---|
| Description | Group a subset of generated records into shared-address households and assign a common HouseholdID. |
| Inputs | T2.2.1 output (record batch) |
| Outputs | `group_households(records, config)` function |
| Dependencies | T2.2.1 |
| Estimated Complexity | Medium |
| Definition of Done | All records sharing a HouseholdID also share Street1+City+ZIP. |
| Required Unit Tests | Assert household membership implies address equality, and vice versa within the configured rate. |
| Potential Risks | Low. |

**T2.3.2 — Implement configurable household size distribution**

| Field | Detail |
|---|---|
| Description | Support household sizes 2–5 per Dataset 6's spec, distributed per config. |
| Inputs | T2.3.1 |
| Outputs | Size-distribution parameter support in `group_households()` |
| Dependencies | T2.3.1 |
| Estimated Complexity | Small |
| Definition of Done | Generated household size distribution matches configured distribution within statistical tolerance at n=500 (Dataset 6 scale). |
| Required Unit Tests | Distribution test at Dataset-6 scale. |
| Potential Risks | Low. |

**T2.3.3 — Implement unit-notation noise injection**

| Field | Detail |
|---|---|
| Description | For the configured fraction of households, vary Street2 notation across members (Apt 101 / #101 / Unit 101) per DATASET_SPECS.md Dataset 6. |
| Inputs | T2.3.1, T2.3.2 |
| Outputs | Unit-notation noise applied to Street2 within noisy households |
| Dependencies | T2.3.2 |
| Estimated Complexity | Small |
| Definition of Done | Noisy households still share Street1+City+ZIP; only Street2 notation varies. |
| Required Unit Tests | Assert household integrity is preserved despite Street2 noise. |
| Potential Risks | Low. |

**T2.3.4 — Unit tests for household grouping correctness**

| Field | Detail |
|---|---|
| Description | End-to-end correctness suite for the household grouper against Dataset 6's exact spec. |
| Inputs | T2.3.1–T2.3.3 |
| Outputs | Household grouper test suite |
| Dependencies | T2.3.3 |
| Estimated Complexity | Small |
| Definition of Done | A generated Dataset-6-scale run matches all Dataset 6 expected outcomes in DATASET_SPECS.md. |
| Required Unit Tests | The suite itself, run against a Dataset-6-shaped config. |
| Potential Risks | Low. |

## Phase 3: Synthetic Data Providers
**Goal:** Build the concrete us_california CountryProfile and wire the column-synonym dictionary into a working column mapper.

### Epic E3.1: us_california CountryProfile Implementation
- **Purpose:** Provide the concrete, MVP-required data pools behind the CountryProfile interface from Phase 2.
- **Business Value:** Without this, no dataset can be generated — this is the first fully concrete, runnable region.
- **Dependencies:** E2.1
- **Expected Deliverables:** Geography, name pool, phone format, email domain, and language distribution data; profile implementation; contract-compliance tests.
- **Definition of Done:** us_california passes the T2.1.2 contract test suite with zero failures.

#### Tasks

**T3.1.1 — Build geography data**

| Field | Detail |
|---|---|
| Description | Author the 10-city California geography dataset (city, ZIP pattern, county, precinct/district scheme) referenced in DATASET_SPECS.md. |
| Inputs | Original source requirements doc (10 CA cities) |
| Outputs | geography.data |
| Dependencies | T2.1.1 |
| Estimated Complexity | Medium |
| Definition of Done | All 10 cities present with valid CA ZIP patterns and county assignments. |
| Required Unit Tests | Data-integrity test: every geography entry has all required fields populated. |
| Potential Risks | Low — mostly data-authoring effort, low logical complexity. |

**T3.1.2 — Build name pools tagged by likely-language**

| Field | Detail |
|---|---|
| Description | Author first/last name pools for the 10 language groups specified in Dataset 9 (Spanish, Chinese, Vietnamese, Korean, Japanese, Indian, Arabic, Russian, Persian, Tagalog), each tagged with its likely-language label. |
| Inputs | DATASET_SPECS.md Dataset 9 |
| Outputs | names.data |
| Dependencies | T2.1.1 |
| Estimated Complexity | Medium |
| Definition of Done | Each of the 10 language groups has a sufficiently large pool to avoid excessive repetition at Dataset 9 scale (~90 records/group). |
| Required Unit Tests | Data-integrity test: every name entry has a non-empty language tag from the approved set of 10. |
| Potential Risks | Low. |

**T3.1.3 — Build phone format rules**

| Field | Detail |
|---|---|
| Description | Define valid NANP formats plus the 'plausible but invalid' formats needed for Dataset 8's negative testing. |
| Inputs | DATASET_SPECS.md Dataset 8 |
| Outputs | phone_format.rules |
| Dependencies | T2.1.1 |
| Estimated Complexity | Small |
| Definition of Done | Rules cover all three invalid-phone patterns named in Dataset 8 (too short, alpha characters, too many digits). |
| Required Unit Tests | Format-generation test producing both valid and each invalid pattern on demand. |
| Potential Risks | Low. |

**T3.1.4 — Build email domain pool**

| Field | Detail |
|---|---|
| Description | Author a realistic pool of email domains for synthetic address generation. |
| Inputs | None |
| Outputs | Email domain pool data |
| Dependencies | T2.1.1 |
| Estimated Complexity | Small |
| Definition of Done | Pool has enough variety to avoid an unrealistic concentration on one domain. |
| Required Unit Tests | Data-integrity test. |
| Potential Risks | Low. |

**T3.1.5 — Build language distribution weights**

| Field | Detail |
|---|---|
| Description | Author CA-realistic weighting across the 10 language groups for use where PreferredLanguage is populated directly (not estimated). |
| Inputs | T3.1.2 |
| Outputs | language_dist.data |
| Dependencies | T3.1.2 |
| Estimated Complexity | Small |
| Definition of Done | Weights sum to 1.0 and cover all 10 language groups. |
| Required Unit Tests | Data-integrity + sum-to-1.0 test. |
| Potential Risks | Low. |

**T3.1.6 — Profile contract-compliance tests**

| Field | Detail |
|---|---|
| Description | Run the T2.1.2 contract suite against the completed us_california profile. |
| Inputs | T3.1.1–T3.1.5, T2.1.2 |
| Outputs | Passing contract-compliance test result |
| Dependencies | T3.1.5 |
| Estimated Complexity | Small |
| Definition of Done | Zero contract failures. |
| Required Unit Tests | The contract suite itself. |
| Potential Risks | Low. |

### Epic E3.2: Column Synonym Dictionary Integration
- **Purpose:** Wire the already-authored column_synonyms.json into a working column_mapper module.
- **Business Value:** Directly required by Datasets 2 and 3 (mapping-focused tests) and by Dataset 10's combined stress test.
- **Dependencies:** T1.2.1
- **Expected Deliverables:** Dictionary loader; column_mapper module; unit tests.
- **Definition of Done:** Given any dataset config's header style, column_mapper correctly renames canonical fields to their configured real-world variant.

#### Tasks

**T3.2.1 — Implement column_synonyms.json loader**

| Field | Detail |
|---|---|
| Description | Parse the existing dictionary file into an in-memory lookup structure (canonical field -> variant list). |
| Inputs | column_synonyms.json (already committed) |
| Outputs | `load_synonyms()` utility |
| Dependencies | T1.1.1 |
| Estimated Complexity | Small |
| Definition of Done | All 24 canonical fields load with their full variant list intact. |
| Required Unit Tests | Round-trip test against the known file contents. |
| Potential Risks | Low. |

**T3.2.2 — Implement column_mapper**

| Field | Detail |
|---|---|
| Description | Given a record batch and a target header style (e.g. 'county_export' abbreviated headers), rename/reorder columns per the dataset config, using the synonym dictionary as the source of allowed variants. |
| Inputs | T3.2.1, T1.3.1 (config format) |
| Outputs | `map_columns(records, config)` function |
| Dependencies | T3.2.1, T1.3.4 |
| Estimated Complexity | Medium |
| Definition of Done | Output header set exactly matches the dataset config's specified header style for all 10 configs. |
| Required Unit Tests | Per-dataset test asserting header names/order match DATASET_SPECS.md's documented examples (e.g. Dataset 2's VOTER_ID/FNAME/LNAME/... headers). |
| Potential Risks | Medium — depends on T1.3.4 config accuracy; errors there will surface here. |

**T3.2.3 — Unit tests for mapping correctness**

| Field | Detail |
|---|---|
| Description | Comprehensive test coverage across all 24 fields and multiple header styles. |
| Inputs | T3.2.2 |
| Outputs | column_mapper test suite |
| Dependencies | T3.2.2 |
| Estimated Complexity | Small |
| Definition of Done | ≥ 90% coverage on column_mapper; every canonical field verified mappable to at least 2 of its known variants. |
| Required Unit Tests | The suite itself. |
| Potential Risks | Low. |

## Phase 4: Data Mutation Engine
**Goal:** Implement every defect category from DATASET_SPECS.md as an independent, composable injector, per ARCHITECTURE.md's 'defects are parameters' principle.

### Epic E4.1: Duplicate Injection
- **Purpose:** Implement all four duplicate categories required by Dataset 5.
- **Business Value:** Directly required by Dataset 5 and contributes to Dataset 10's combined stress test.
- **Dependencies:** E2.2
- **Expected Deliverables:** Four duplicate-injection strategies; ground-truth tagging of injected duplicates; unit tests.
- **Definition of Done:** Given Dataset 5's config, the four duplicate categories are injected at the exact configured rates and each injected duplicate is ground-truth-tagged with its category.

#### Tasks

**T4.1.1 — Implement exact duplicate injection**

| Field | Detail |
|---|---|
| Description | Repeat a byte-identical record at the configured rate (15% for Dataset 5). |
| Inputs | T2.2.1 output |
| Outputs | Exact-duplicate injector |
| Dependencies | T2.2.1 |
| Estimated Complexity | Small |
| Definition of Done | Injected count matches configured rate exactly; duplicates are byte-identical to their source record. |
| Required Unit Tests | Assert injected count and byte-identity. |
| Potential Risks | Low. |

**T4.1.2 — Implement near-duplicate injection**

| Field | Detail |
|---|---|
| Description | Same person with a name variant (middle initial added/dropped, suffix added) but identical phone and address (10% for Dataset 5). |
| Inputs | T2.2.1 output |
| Outputs | Near-duplicate injector |
| Dependencies | T2.2.1 |
| Estimated Complexity | Medium |
| Definition of Done | Injected pairs share phone+address but differ only in the specified name variant dimensions. |
| Required Unit Tests | Assert phone/address equality and controlled name difference. |
| Potential Risks | Low. |

**T4.1.3 — Implement phone-match-only duplicate injection**

| Field | Detail |
|---|---|
| Description | Duplicates identifiable only by matching phone, with name/address differing slightly (5% for Dataset 5). |
| Inputs | T2.2.1 output |
| Outputs | Phone-match injector |
| Dependencies | T2.2.1 |
| Estimated Complexity | Medium |
| Definition of Done | Injected pairs share phone only; name and address are each meaningfully different. |
| Required Unit Tests | Assert phone equality and name/address inequality. |
| Potential Risks | Low. |

**T4.1.4 — Implement email-match-only duplicate injection**

| Field | Detail |
|---|---|
| Description | Duplicates identifiable only by matching email (5% for Dataset 5). |
| Inputs | T2.2.1 output |
| Outputs | Email-match injector |
| Dependencies | T2.2.1 |
| Estimated Complexity | Small |
| Definition of Done | Injected pairs share email only. |
| Required Unit Tests | Assert email equality and other-field inequality. |
| Potential Risks | Low. |

**T4.1.5 — Ground-truth duplicate count tests**

| Field | Detail |
|---|---|
| Description | Verify total injected duplicate counts and per-category breakdown match Dataset 5's configured rates exactly, for later comparison against metrics_calculator (Phase 5). |
| Inputs | T4.1.1–T4.1.4 |
| Outputs | Duplicate injection test suite |
| Dependencies | T4.1.4 |
| Estimated Complexity | Small |
| Definition of Done | At Dataset 5 scale (1,000 records), category counts match configured rates exactly (integer rounding documented). |
| Required Unit Tests | The suite itself. |
| Potential Risks | Low. |

### Epic E4.2: Missing Data Injection
- **Purpose:** Implement configurable per-field missingness required by Dataset 7.
- **Business Value:** Directly required by Dataset 7 and by Dataset 10's combined stress test.
- **Dependencies:** E2.2
- **Expected Deliverables:** Per-field missingness injector; unit tests.
- **Definition of Done:** Given Dataset 7's per-field rates, actual missingness matches configured rates within statistical tolerance at n=800.

#### Tasks

**T4.2.1 — Implement per-field configurable missingness**

| Field | Detail |
|---|---|
| Description | Independently null out each configured field at its configured rate (Phone 20%, Email 30%, DOB 15%, ZIP 5%, PreferredLanguage 40%, Street2 85%, per Dataset 7). |
| Inputs | T2.2.1 output |
| Outputs | Missingness injector |
| Dependencies | T2.2.1 |
| Estimated Complexity | Medium |
| Definition of Done | Actual missingness rate per field is within statistical tolerance of the configured rate at Dataset 7 scale. |
| Required Unit Tests | Per-field rate assertion at n=800. |
| Potential Risks | Low. |

**T4.2.2 — Missingness rate unit tests**

| Field | Detail |
|---|---|
| Description | Statistical tolerance tests across multiple seeds to confirm the injector isn't systematically biased. |
| Inputs | T4.2.1 |
| Outputs | Missingness test suite |
| Dependencies | T4.2.1 |
| Estimated Complexity | Small |
| Definition of Done | Rate holds within tolerance across ≥ 5 different seeds. |
| Required Unit Tests | The suite itself. |
| Potential Risks | Low. |

### Epic E4.3: Invalid Data Injection
- **Purpose:** Implement the exactly-one-invalid-field-per-record pattern required by Dataset 8.
- **Business Value:** Directly required by Dataset 8 and by Dataset 10's combined stress test.
- **Dependencies:** E3.1 (phone_format.rules)
- **Expected Deliverables:** Invalid phone/email/ZIP generators; per-record single-defect enforcement; unit tests.
- **Definition of Done:** Every record in a Dataset-8-shaped run has exactly one invalid field, evenly distributed across the three invalid categories, with the other 23 fields fully valid.

#### Tasks

**T4.3.1 — Implement invalid phone generator**

| Field | Detail |
|---|---|
| Description | Generate the three invalid-phone patterns from DATASET_SPECS.md Dataset 8 (too short, alpha characters, too many digits), evenly split. |
| Inputs | T3.1.3 |
| Outputs | Invalid-phone generator |
| Dependencies | T3.1.3 |
| Estimated Complexity | Small |
| Definition of Done | All three patterns are producible and are individually distinguishable in test assertions. |
| Required Unit Tests | One assertion per pattern. |
| Potential Risks | Low. |

**T4.3.2 — Implement invalid email generator**

| Field | Detail |
|---|---|
| Description | Generate the four invalid-email patterns (missing @, missing domain, double @, no TLD). |
| Inputs | T3.1.4 |
| Outputs | Invalid-email generator |
| Dependencies | T3.1.4 |
| Estimated Complexity | Small |
| Definition of Done | All four patterns producible and distinguishable. |
| Required Unit Tests | One assertion per pattern. |
| Potential Risks | Low. |

**T4.3.3 — Implement invalid ZIP generator**

| Field | Detail |
|---|---|
| Description | Generate invalid ZIPs (alpha characters, wrong digit count). |
| Inputs | T3.1.1 |
| Outputs | Invalid-ZIP generator |
| Dependencies | T3.1.1 |
| Estimated Complexity | Small |
| Definition of Done | Both invalid patterns producible. |
| Required Unit Tests | One assertion per pattern. |
| Potential Risks | Low. |

**T4.3.4 — Enforce exactly-one-invalid-field-per-record**

| Field | Detail |
|---|---|
| Description | Orchestrate T4.3.1–T4.3.3 so each record receives exactly one invalid field, evenly distributed across the three categories, and the remaining 23 fields pass schema validation. |
| Inputs | T4.3.1–T4.3.3, T1.2.2 |
| Outputs | Single-defect orchestration logic |
| Dependencies | T4.3.3 |
| Estimated Complexity | Medium |
| Definition of Done | At Dataset 8 scale (700 records), each record has exactly one schema-validation failure, localized to the intended field. |
| Required Unit Tests | Full-record validation test asserting exactly one field fails per record. |
| Potential Risks | Medium — the 'zero false positives on the clean 23 fields' requirement from DATASET_SPECS.md is easy to violate accidentally; needs a dedicated regression test, not just spot checks. |

### Epic E4.4: Formatting Noise Injection
- **Purpose:** Implement the casing/whitespace/abbreviation noise required by Dataset 4.
- **Business Value:** Directly required by Dataset 4 and by Dataset 10's combined stress test.
- **Dependencies:** E2.2
- **Expected Deliverables:** Casing noise, whitespace noise, and street-abbreviation noise injectors; unit tests.
- **Definition of Done:** Given Dataset 4's config, each noise type is applied at its configured rate without corrupting the underlying record identity.

#### Tasks

**T4.4.1 — Implement casing noise**

| Field | Detail |
|---|---|
| Description | Randomize name-field casing (ALL CAPS / lowercase / MiXeD) at the configured rate (30% for Dataset 4). |
| Inputs | T2.2.1 output |
| Outputs | Casing noise injector |
| Dependencies | T2.2.1 |
| Estimated Complexity | Small |
| Definition of Done | Rate matches config within tolerance; underlying value is recoverable via case-insensitive comparison. |
| Required Unit Tests | Rate + recoverability assertions. |
| Potential Risks | Low. |

**T4.4.2 — Implement whitespace noise**

| Field | Detail |
|---|---|
| Description | Inject leading/trailing/double whitespace at the configured rate (10% for Dataset 4). |
| Inputs | T2.2.1 output |
| Outputs | Whitespace noise injector |
| Dependencies | T2.2.1 |
| Estimated Complexity | Small |
| Definition of Done | Rate matches config; trimmed value equals the original. |
| Required Unit Tests | Rate + trim-equality assertions. |
| Potential Risks | Low. |

**T4.4.3 — Implement street abbreviation inconsistency**

| Field | Detail |
|---|---|
| Description | Vary 'Street' token abbreviation (Street/STREET/St./st) at the configured rate (25% for Dataset 4). |
| Inputs | T2.2.1 output |
| Outputs | Abbreviation noise injector |
| Dependencies | T2.2.1 |
| Estimated Complexity | Small |
| Definition of Done | Rate matches config; all four variant forms appear in a large enough sample. |
| Required Unit Tests | Rate assertion + variant-coverage assertion. |
| Potential Risks | Low. |

**T4.4.4 — Formatting noise unit tests**

| Field | Detail |
|---|---|
| Description | Combined test suite covering all three noise types plus the 3% blank-row case from Dataset 4. |
| Inputs | T4.4.1–T4.4.3 |
| Outputs | Formatting noise test suite |
| Dependencies | T4.4.3 |
| Estimated Complexity | Small |
| Definition of Done | All Dataset 4 defect rates verified simultaneously on one generated batch. |
| Required Unit Tests | The suite itself. |
| Potential Risks | Low. |

## Phase 5: Metrics Engine
**Goal:** Compute ground-truth metrics directly from generation/injection parameters, never by re-inferring them from output data — the 'correct-by-construction' principle from ARCHITECTURE.md.

### Epic E5.1: Ground-Truth Metrics Calculator
- **Purpose:** Implement metrics_calculator per ARCHITECTURE.md §3, deriving every reported metric from known injection state.
- **Business Value:** This is what makes the dataset suite trustworthy as a regression fixture — metrics can never silently drift from the data they describe.
- **Dependencies:** E4.1, E4.2, E4.3, E2.3, E3.2
- **Expected Deliverables:** Duplicate/missing/invalid/mapping/household metric calculators; unit tests proving exact match to injection ground truth.
- **Definition of Done:** For every one of the 10 dataset configs, calculated metrics exactly match the known injection parameters (not estimates).

#### Tasks

**T5.1.1 — Implement duplicate count calculator**

| Field | Detail |
|---|---|
| Description | Report per-category duplicate counts (exact, near-dup, phone-match, email-match) directly from the tags applied in Phase 4, not by re-detecting duplicates. |
| Inputs | T4.1.1–T4.1.4 tags |
| Outputs | Duplicate metrics function |
| Dependencies | T4.1.5 |
| Estimated Complexity | Small |
| Definition of Done | Exact match to injection counts on Dataset 5. |
| Required Unit Tests | Assert calculator output equals injector's own record of what it injected. |
| Potential Risks | Low. |

**T5.1.2 — Implement missing-field count calculator**

| Field | Detail |
|---|---|
| Description | Report per-field missingness counts directly from T4.2.1's injection state. |
| Inputs | T4.2.1 |
| Outputs | Missing-field metrics function |
| Dependencies | T4.2.2 |
| Estimated Complexity | Small |
| Definition of Done | Exact match to injection counts on Dataset 7. |
| Required Unit Tests | Assert calculator output equals injector's ground truth. |
| Potential Risks | Low. |

**T5.1.3 — Implement invalid-field count calculator**

| Field | Detail |
|---|---|
| Description | Report invalid-field counts and category breakdown directly from T4.3.4's injection state. |
| Inputs | T4.3.4 |
| Outputs | Invalid-field metrics function |
| Dependencies | T4.3.4 |
| Estimated Complexity | Small |
| Definition of Done | Exact match to injection counts on Dataset 8. |
| Required Unit Tests | Assert calculator output equals injector's ground truth. |
| Potential Risks | Low. |

**T5.1.4 — Implement mapping-rate calculator**

| Field | Detail |
|---|---|
| Description | Report the percentage of columns successfully mapped by column_mapper for a given dataset run. |
| Inputs | T3.2.2 |
| Outputs | Mapping-rate metrics function |
| Dependencies | T3.2.3 |
| Estimated Complexity | Small |
| Definition of Done | Reported rate matches Dataset 2's ≥95% target when run against Dataset 2's config. |
| Required Unit Tests | Assert rate calculation against a known-mapping-outcome fixture. |
| Potential Risks | Low. |

**T5.1.5 — Implement household metrics**

| Field | Detail |
|---|---|
| Description | Report household count and average household size directly from T2.3.1's grouping state. |
| Inputs | T2.3.1–T2.3.4 |
| Outputs | Household metrics function |
| Dependencies | T2.3.4 |
| Estimated Complexity | Small |
| Definition of Done | Exact match to grouping ground truth on Dataset 6. |
| Required Unit Tests | Assert calculator output equals grouper's own record of constructed households. |
| Potential Risks | Low. |

**T5.1.6 — Metrics-calculator ground-truth unit tests**

| Field | Detail |
|---|---|
| Description | Cross-cutting test suite running the full metrics_calculator against all 10 dataset configs and asserting exact ground-truth match. |
| Inputs | T5.1.1–T5.1.5 |
| Outputs | Metrics engine test suite |
| Dependencies | T5.1.5 |
| Estimated Complexity | Medium |
| Definition of Done | Zero discrepancies between reported metrics and injection ground truth across all 10 datasets. |
| Required Unit Tests | The suite itself — this is effectively the project's core correctness gate. |
| Potential Risks | Medium — by design, this task will surface any drift accumulated in Phases 2–4; budget time for fixing upstream issues discovered here, not just this task's own code. |

## Phase 6: Export Engine
**Goal:** Turn generated, mutated, mapped record batches into the actual output files (CSV) for all 10 datasets in one orchestrated run.

### Epic E6.1: CSV Writer
- **Purpose:** Write mapped record batches to disk as CSV, matching each dataset's configured header style exactly.
- **Business Value:** The literal deliverable format the Campaign Data Engine's Import Wizard consumes.
- **Dependencies:** E3.2
- **Expected Deliverables:** CSV writer; unknown/extra-column support; round-trip tests.
- **Definition of Done:** Written CSV files, when read back, reproduce the exact record set and header style that was written.

#### Tasks

**T6.1.1 — Implement column_mapper-aware CSV writer**

| Field | Detail |
|---|---|
| Description | Write a record batch to CSV using the header names/order produced by column_mapper for the given dataset config. |
| Inputs | T3.2.2 |
| Outputs | `write_csv(records, config)` function |
| Dependencies | T3.2.2 |
| Estimated Complexity | Medium |
| Definition of Done | Output header row exactly matches the dataset config's specified style for all 10 configs. |
| Required Unit Tests | Per-dataset header assertion. |
| Potential Risks | Low. |

**T6.1.2 — Implement unknown/extra column support**

| Field | Detail |
|---|---|
| Description | Support Dataset 3's requirement for 2 blank columns and 4 extra non-canonical columns (Volunteer Notes, Call Status, Favorite Issue, Visited) appended to output. |
| Inputs | T6.1.1 |
| Outputs | Extra-column support in CSV writer |
| Dependencies | T6.1.1 |
| Estimated Complexity | Small |
| Definition of Done | Dataset 3 output contains exactly 2 blank and 4 extra columns as specified. |
| Required Unit Tests | Column-count and column-name assertions on Dataset 3 output. |
| Potential Risks | Low. |

**T6.1.3 — Round-trip CSV tests**

| Field | Detail |
|---|---|
| Description | Read back every written CSV and confirm the parsed records match the generated records exactly (accounting for intentional defects). |
| Inputs | T6.1.1, T6.1.2 |
| Outputs | Round-trip test suite |
| Dependencies | T6.1.2 |
| Estimated Complexity | Small |
| Definition of Done | Zero data loss or corruption on round-trip for all 10 dataset shapes. |
| Required Unit Tests | The suite itself. |
| Potential Risks | Low. |

### Epic E6.2: Dataset Orchestrator
- **Purpose:** Implement ARCHITECTURE.md's pipeline/orchestrator: the single entrypoint that runs generate → group → inject → map → write for a given config.
- **Business Value:** This is the module a developer or CI job actually invokes; it ties every prior phase together into one runnable system.
- **Dependencies:** E5.1, E6.1
- **Expected Deliverables:** Per-config orchestration function; CLI/entrypoint running all 10 configs; unit tests.
- **Definition of Done:** Running the orchestrator against all 10 configs produces 10 correct CSV files and 10 correct metrics reports in one command.

#### Tasks

**T6.2.1 — Implement per-dataset-config orchestration**

| Field | Detail |
|---|---|
| Description | Wire the full pipeline (generate → group households → inject defects → map columns → write CSV → compute metrics) for a single config. |
| Inputs | All prior phase outputs |
| Outputs | `run_dataset(config)` function |
| Dependencies | T6.1.2, T5.1.6 |
| Estimated Complexity | Large |
| Definition of Done | Running against each of the 10 configs individually produces output matching that dataset's DATASET_SPECS.md entry. |
| Required Unit Tests | One integration test per dataset (10 total), each asserting record count, defect rates, and metrics match spec. |
| Potential Risks | Medium — integration point where upstream bugs from any prior phase will surface; treat failures here as diagnostic, not just this task's fault. |

**T6.2.2 — Implement CLI/entrypoint for all 10 configs**

| Field | Detail |
|---|---|
| Description | Single command that runs T6.2.1 against every config in dataset_configs/ and writes all outputs. |
| Inputs | T6.2.1 |
| Outputs | CLI entrypoint |
| Dependencies | T6.2.1 |
| Estimated Complexity | Small |
| Definition of Done | One command produces all 10 CSVs (and metrics reports) from a clean checkout. |
| Required Unit Tests | End-to-end smoke test invoking the CLI. |
| Potential Risks | Low. |

**T6.2.3 — Orchestrator record-count unit tests**

| Field | Detail |
|---|---|
| Description | Assert final record counts for all 10 datasets match DATASET_SPECS.md exactly (100/800/600/1,000/1,000/500/800/700/900/2,000). |
| Inputs | T6.2.2 |
| Outputs | Record-count test suite |
| Dependencies | T6.2.2 |
| Estimated Complexity | Small |
| Definition of Done | Exact match for all 10 datasets. |
| Required Unit Tests | The suite itself. |
| Potential Risks | Low. |

## Phase 7: Validation
**Goal:** Confirm the generated output actually satisfies DATASET_SPECS.md's stated expectations, not just that the code ran without error.

### Epic E7.1: Dataset-Level Validation
- **Purpose:** Validate generated output against both the canonical schema and each dataset's documented expected outcomes.
- **Business Value:** Closes the loop between 'the code ran' and 'the code produced what DATASET_SPECS.md promised.'
- **Dependencies:** E6.2
- **Expected Deliverables:** Schema-conformance validator over full output; per-dataset expected-outcome checker; manual review checklist.
- **Definition of Done:** All 10 generated datasets pass schema conformance and match their DATASET_SPECS.md expected-outcome targets.

#### Tasks

**T7.1.1 — Implement schema-conformance validator over generated output**

| Field | Detail |
|---|---|
| Description | Run T1.2.2's validator against every record in every generated CSV, accounting for intentional invalid-field defects (Dataset 8) so those aren't false-flagged as bugs. |
| Inputs | T6.2.2 output, T1.2.2 |
| Outputs | Full-output validation report |
| Dependencies | T6.2.2 |
| Estimated Complexity | Medium |
| Definition of Done | Zero unintentional schema violations across all 10 datasets. |
| Required Unit Tests | The validator run itself, treated as a CI gate. |
| Potential Risks | Low. |

**T7.1.2 — Implement per-dataset expected-outcome checker**

| Field | Detail |
|---|---|
| Description | Automated check comparing each dataset's metrics_calculator output against the target values stated in DATASET_SPECS.md (e.g. Dataset 2's ≥95% mapping rate, Dataset 8's zero-false-positive requirement). |
| Inputs | T5.1.6, DATASET_SPECS.md |
| Outputs | Expected-outcome checker |
| Dependencies | T5.1.6 |
| Estimated Complexity | Medium |
| Definition of Done | All 10 datasets pass their documented targets. |
| Required Unit Tests | The checker itself, run per dataset. |
| Potential Risks | Low. |

**T7.1.3 — Manual review checklist for generated CSV samples**

| Field | Detail |
|---|---|
| Description | Human-reviewable checklist for spot-checking a sample of each dataset's output for plausibility (e.g. do multilingual names look reasonable, do addresses look real). |
| Inputs | T6.2.2 output |
| Outputs | Review checklist document + completed review notes |
| Dependencies | T6.2.2 |
| Estimated Complexity | Small |
| Definition of Done | Checklist completed and signed off for all 10 datasets. |
| Required Unit Tests | N/A (manual task). |
| Potential Risks | Low — subjective judgment call; document reviewer name/date for traceability. |

## Phase 8: Regression Test Suite
**Goal:** Lock in reproducibility and correctness as an ongoing CI gate, not a one-time check.

### Epic E8.1: End-to-End Regression Harness
- **Purpose:** Wire the full generation pipeline into CI as a repeatable regression suite.
- **Business Value:** Protects the fixture suite from silent regressions as the Campaign Data Engine (the system under test) evolves alongside this generator.
- **Dependencies:** E7.1
- **Expected Deliverables:** Full-suite regression run script; diff-based reproducibility check; CI wiring.
- **Definition of Done:** CI fails if any dataset's output changes for a fixed seed, or if any expected-outcome target regresses.

#### Tasks

**T8.1.1 — Implement full-suite regression run script**

| Field | Detail |
|---|---|
| Description | Single script running all 10 dataset configs plus all validation/expected-outcome checks from Phase 7. |
| Inputs | T6.2.2, T7.1.1, T7.1.2 |
| Outputs | Regression run script |
| Dependencies | T7.1.2 |
| Estimated Complexity | Medium |
| Definition of Done | One command runs generation + validation for all 10 datasets and produces a pass/fail summary. |
| Required Unit Tests | The script itself, run manually once as acceptance. |
| Potential Risks | Low. |

**T8.1.2 — Implement diff-based reproducibility check**

| Field | Detail |
|---|---|
| Description | Compare freshly generated output against a committed golden-output snapshot (same seed) and fail on any byte difference. |
| Inputs | T8.1.1 |
| Outputs | Golden-snapshot diff checker |
| Dependencies | T8.1.1 |
| Estimated Complexity | Medium |
| Definition of Done | An intentional code change that alters output is caught by the diff check. |
| Required Unit Tests | Negative test: introduce a deliberate change and confirm the check fails. |
| Potential Risks | Medium — golden snapshots need a clear, documented process for intentional updates (vs. regressions) to avoid the check becoming a rubber stamp. |

**T8.1.3 — Wire regression suite into CI**

| Field | Detail |
|---|---|
| Description | Run T8.1.1/T8.1.2 on every pull request. |
| Inputs | T8.1.2, T1.1.5 |
| Outputs | CI job |
| Dependencies | T8.1.2 |
| Estimated Complexity | Small |
| Definition of Done | A PR that breaks reproducibility or an expected-outcome target is blocked by CI. |
| Required Unit Tests | Manual verification with a throwaway failing PR. |
| Potential Risks | Low. |

## Phase 9: Documentation
**Goal:** Ensure the system is maintainable and extensible by someone who wasn't part of building it.

### Epic E9.1: Developer Documentation
- **Purpose:** Document setup, extension points, and sign off on the full documentation set before release.
- **Business Value:** Directly enables the ARCHITECTURE.md extension points (new dataset, new region, new defect type) to actually be used by future contributors.
- **Dependencies:** E8.1
- **Expected Deliverables:** Setup guide; config format guide; CountryProfile extension guide; final documentation review.
- **Definition of Done:** A new contributor can, using only the docs, add a new dataset config and a stub new-region profile without reading source code.

#### Tasks

**T9.1.1 — Write developer setup guide**

| Field | Detail |
|---|---|
| Description | README addendum covering install, running the generator, and running the regression suite. |
| Inputs | T1.1.2, T1.1.4, T8.1.1 |
| Outputs | Setup guide section |
| Dependencies | T8.1.1 |
| Estimated Complexity | Small |
| Definition of Done | A developer unfamiliar with the project can get a working local run using only this guide. |
| Required Unit Tests | N/A — validate via a dry-run walkthrough with someone not on the implementation team. |
| Potential Risks | Low. |

**T9.1.2 — Document config file format**

| Field | Detail |
|---|---|
| Description | Reference documentation for the dataset_configs/*.config format from T1.3.1, with a worked example of adding an 11th dataset. |
| Inputs | T1.3.1, T1.3.4 |
| Outputs | Config format reference doc |
| Dependencies | T1.3.4 |
| Estimated Complexity | Small |
| Definition of Done | Worked example is copy-pasteable and produces a valid new config. |
| Required Unit Tests | N/A — validate the worked example actually loads and validates. |
| Potential Risks | Low. |

**T9.1.3 — Document CountryProfile extension guide**

| Field | Detail |
|---|---|
| Description | Step-by-step guide for implementing a new region (e.g. india) against the T2.1.1 interface, referencing the T2.1.2 contract suite. |
| Inputs | T2.1.1, T2.1.2, T3.1.6 |
| Outputs | CountryProfile extension guide |
| Dependencies | T3.1.6 |
| Estimated Complexity | Small |
| Definition of Done | Guide references every one of the 7 required interface capabilities with a concrete example. |
| Required Unit Tests | N/A — validate by having someone stub a second profile using only this guide. |
| Potential Risks | Low. |

**T9.1.4 — Final review and sign-off of all documentation**

| Field | Detail |
|---|---|
| Description | Review README.md, ARCHITECTURE.md, SCHEMA.md, DATASET_SPECS.md, column_synonyms.json, and the three new guides above for consistency with the as-built system. |
| Inputs | All prior documentation |
| Outputs | Sign-off record |
| Dependencies | T9.1.3 |
| Estimated Complexity | Small |
| Definition of Done | No contradictions between documentation and as-built behavior. |
| Required Unit Tests | N/A (review task). |
| Potential Risks | Low — but skipping this task is the most common source of stale docs; treat as mandatory, not optional. |

---

# 6. Parallel Development Opportunities
The dependency graph above allows meaningful parallelism once Phase 1 (Foundation) is complete. Recommended parallel tracks for multiple AI agents or developers:

**Track A — Core Engine (sequential, single owner recommended):**
E2.1 → E2.2 → E2.3. This is the critical path; other tracks depend on its output and should not start their integration-dependent tasks until it lands.

**Track B — Regional Data (parallel with Track A once E2.1 lands):**
All of E3.1's tasks (T3.1.1–T3.1.5) are pure data-authoring tasks with no code dependency on E2.2/E2.3 — only on the E2.1 interface shape. These can be split across multiple contributors simultaneously (e.g. one per data pool).

**Track C — Defect Injectors (parallel with each other once E2.2 lands):**
E4.1 (duplicates), E4.2 (missing data), and E4.4 (formatting noise) have no dependencies on each other — only on E2.2's record generator. These three Epics can be built simultaneously by three different agents/developers. E4.3 (invalid data) additionally depends on E3.1's phone/email/ZIP data and should start after Track B's relevant tasks land.

**Track D — Mapping (parallel with Track C):**
E3.2 (column synonym integration) depends only on the already-committed `column_synonyms.json` and T1.2.1 — it can proceed independently of the entire Data Mutation Engine phase.

**Serialization points (do not parallelize across these):**
- Phase 5 (Metrics Engine) requires Phase 4 substantially complete — it measures what Phase 4 injects.
- Phase 6's orchestrator (E6.2) requires Phase 5 complete — it reports metrics as part of each run.
- Phase 7–9 are inherently sequential validation/lock-in/documentation passes over a complete system.

**Task-level parallelism within Epics:** within E1.1, T1.1.3 and T1.1.4 have no dependency on each other (both only depend on T1.1.2) and can be done in parallel. Within E4.1, all four injector tasks (T4.1.1–T4.1.4) are independent of each other and can be split across agents, with T4.1.5 as the integration/verification task that must wait for all four.

---

# 7. Testing Milestones
- **Unit Testing:** continuous, per-task — every task in §5 has an explicit Required Unit Tests entry and is not considered done without it. No milestone gate needed; this is the default mode of work.
- **Integration Testing:** at the end of Phase 6 (Export Engine), once T6.2.1's 10 per-dataset integration tests exist — this is the first point the full pipeline runs end-to-end for every dataset.
- **Regression Testing:** established at Phase 8 (T8.1.1–T8.1.3) and run on every PR from that point forward, including all PRs during Phase 9.
- **Performance Testing:** at the end of Phase 6, once the orchestrator (T6.2.2) can run all 10 configs — measure total generation time at current (small) scale, and specifically benchmark Dataset 10 (largest, most defect-dense) as the stress-test proxy for the eventual ~100k-record target scale.
- **Synthetic Dataset Validation:** Phase 7 in full (E7.1) — this phase exists specifically to validate the generated datasets against `DATASET_SPECS.md`'s stated expected outcomes, distinct from and after integration testing confirms the pipeline runs correctly.

---

# 8. Release Milestones
**Alpha** (end of Phase 3): `us_california` profile complete and passing its contract suite; clean records can be generated and correctly column-mapped for at least the two mapping-focused datasets (2 and 3). No defect injection yet. Internal use only — not yet representative of the full suite.

**Beta** (end of Phase 6): All 10 dataset configs run end-to-end via the orchestrator and produce CSV output with correctly computed metrics. Not yet independently validated against `DATASET_SPECS.md` targets (that's Phase 7) and no regression lock-in yet (Phase 8). Suitable for early manual testing against the Campaign Data Engine, with the caveat that dataset correctness is not yet formally verified.

**Release Candidate** (end of Phase 8): All 10 datasets pass Phase 7 validation against `DATASET_SPECS.md` targets, and the regression suite (Phase 8) is wired into CI with a committed golden-snapshot baseline. Functionally complete; only documentation remains.

**Version 1.0** (end of Phase 9): All documentation (README, ARCHITECTURE, SCHEMA, DATASET_SPECS, this plan, and the three new developer guides from T9.1.1–T9.1.3) is reviewed and signed off (T9.1.4) as consistent with the as-built system. Ready to be used as the standing regression-fixture generator for the Campaign Data Engine, and ready for a new contributor to extend with a new region or dataset using only the docs.

---

# 9. Risks
| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Config-to-spec transcription errors (T1.3.4) | Medium | High — every downstream dataset inherits any error made translating DATASET_SPECS.md into config files. | Dedicated manual review checklist cross-referencing each config field against its DATASET_SPECS.md source line; caught early by T5.1.6's ground-truth tests. |
| Randomness bypassing the seeded RNG wrapper (T2.2.2) | Medium | High — silently breaks reproducibility, undermining the entire regression-fixture value proposition. | Code-review checklist item; consider a lint rule flagging direct use of unseeded random functions outside T2.2.2. |
| record_generator (T2.2.1) complexity underestimated | Medium | Medium — largest single task in the plan; could stall Phase 2 if it grows beyond one PR's worth of work. | Explicitly flagged as splittable into per-field sub-tasks if needed during implementation; don't force it into one PR if DoD can't be met. |
| Dataset 8's 'zero false positives on clean fields' requirement violated silently (T4.3.4) | Medium | Medium — would undermine the specific test purpose of the Invalid Data Dataset. | Dedicated full-record validation test, not just spot checks; treat as a hard gate before Phase 5 begins. |
| Metrics Engine (Phase 5) surfaces upstream bugs from Phases 2–4 | High | Medium — expected and even desirable, but could be mistaken for a Phase 5 defect and misallocate debugging time. | Document in task T5.1.6 that failures here are diagnostic for earlier phases; budget fix time against the originating phase, not Phase 5. |
| Golden-snapshot regression check (T8.1.2) becomes a rubber stamp | Low | Medium — if updating the snapshot becomes routine/unreviewed, the check stops catching real regressions. | Require an explicit, reviewed justification in the PR description any time a golden snapshot is updated. |
| Stack/language decision (see §11 Open Questions) made late | Medium | Low–Medium — mostly affects Phase 1 setup tasks; low blast radius if decided before Phase 1 starts. | Resolve as the first Open Question before any Phase 1 task begins. |
| CI provider access/permissions unavailable to the implementing agent (T1.1.5, T8.1.3) | Low | Medium — blocks automated gating, though local test/lint runs remain possible. | Flag immediately if CI access is unavailable; fall back to a documented manual pre-merge checklist. |

---

# 10. Assumptions
- The 5 existing artifacts (`README.md`, `ARCHITECTURE.md`, `SCHEMA.md`, `DATASET_SPECS.md`, `column_synonyms.json`) are the complete and final architecture baseline for this implementation — this plan does not re-derive or re-litigate architecture decisions.
- Record counts remain at the small-scale pass defined in `DATASET_SPECS.md` (8,400 records total across 10 datasets) for this implementation cycle; scaling toward the original ~100,000-voter target is a follow-on config change, not part of this plan's scope.
- The 10 expected-outcome metadata JSON files (one per dataset) remain explicitly out of scope, consistent with the prior architecture-phase decision — Phase 5's metrics_calculator computes these values in-memory/at test time rather than persisting them as committed fixtures, unless a future plan revision adds that back in.
- Only the `us_california` CountryProfile is required for this implementation cycle; other regions (India, UK, Canada, Australia) are extension points, not delivered work.
- "Codex" in this context refers to an AI coding agent capable of executing discrete, well-specified engineering tasks against this repository — the task granularity in §5 is sized accordingly (one task ≈ one PR).
- A CI provider is available and can be configured by the implementing team/agent; if not, T1.1.5 and T8.1.3 will need to be re-scoped to manual gating.
- Reproducibility (seeded, deterministic output) is treated as a hard requirement throughout, per `ARCHITECTURE.md` §2, not a nice-to-have.

---

# 11. Open Questions
- **Implementation language/stack.** Not yet decided (flagged as open during the architecture phase). This affects T1.1.2 onward and should be resolved before Phase 1 begins.
- **CI provider.** Which CI system will host T1.1.5/T8.1.3? Needs to be confirmed or the plan re-scoped to manual gating.
- **Golden-snapshot storage.** Where do T8.1.2's committed golden-output snapshots live — in-repo (adds repo size) or an external artifact store? Needs a decision before Phase 8.
- **Scale-up trigger.** What signals that it's time to multiply the small-scale record counts toward the original ~100k target — is that a separate future plan, or should Phase 6+ budget for a scale-up pass within this cycle?
- **Ownership model for parallel tracks (§6).** If multiple AI agents work in parallel, is there a designated integration owner who resolves conflicts between tracks, or is that left to whoever lands second?
- **Manual review checklist reviewer (T7.1.3).** Who signs off on the subjective plausibility review — does this require a specific named role (e.g. the original Chief Software Architect, or the eventual Campaign Data Engine team)?

---

# 12. Codex Handoff

## Instructions for Engineering Team
This section governs how an AI coding agent (such as Codex) or a human engineer should execute this plan.

- Implement only one task at a time, identified by its Task ID (e.g. T2.2.1). Do not begin a second task before the first meets its Definition of Done.
- Do not skip dependencies. Check each task's Dependencies field before starting; if a dependency isn't yet merged, stop and flag it rather than working around it.
- Run tests before completing each task. A task is not done until its Required Unit Tests pass, not just until the code compiles or runs once manually.
- Update documentation whenever implementation changes. If implementation reveals that ARCHITECTURE.md, SCHEMA.md, or this plan is inaccurate, update the doc in the same PR — don't let docs drift silently.
- Never modify architecture without approval. If a task seems to require a design change beyond what's described here, stop and raise it as a question rather than improvising a solution.
- Report assumptions rather than invent missing requirements. If a task's inputs are ambiguous, state the assumption made and proceed, or ask — do not silently guess at requirements not present in the source documents.
- Stop after completing the assigned task and wait for review. Do not chain into the next task automatically, even if it seems obvious what comes next.
- Treat the Risks column (§9) as active guidance, not background reading — several tasks (T1.3.4, T2.2.2, T4.3.4, T5.1.6) carry specific known failure modes worth re-reading before starting that task.

This document is **IMPLEMENTATION_PLAN.md** and will be committed to the repository before any production code is written. It reflects the current, final architecture as of this writing; any future architecture change requires a corresponding revision to this plan before implementation resumes on the affected phase.
