# Vinvite Synthetic Voter Dataset Generator — Execution Backlog (TASKS.md)
**Prepared by:** Chief Software Architect, Zapurzaa Systems (ZSPL)
**Audience:** Implementation engineer (Codex)
**Source:** Derived from `IMPLEMENTATION_PLAN.md`'s 68 tasks across 9 phases / 18 epics
**Status:** Execution backlog — no code, no architecture changes

---

## How this document was built
Every task in `IMPLEMENTATION_PLAN.md` is broken into one or more atomic engineering tasks below, each independently implementable, sized to fit a single pull request, and scoped to roughly 2–4 hours of engineering effort.

**Documented assumption:** `IMPLEMENTATION_PLAN.md` sized its 68 tasks at Small/Medium/Large complexity, where Small tasks already fit a 2–4 hour PR but Medium and Large tasks do not. To satisfy this document's 2–4 hour requirement, every Medium task from the plan is split into 2 atomic tasks here, and every Large task (`T2.2.1`, `T6.2.1`) is split into 4. This is a decomposition of existing scope, not a redesign — no task's underlying engineering content changes, only its granularity. Per the plan's own instruction to report assumptions rather than invent requirements, this is flagged here rather than silently assumed.

Each original `IMPLEMENTATION_PLAN.md` task ID (e.g. `T2.2.1`) is referenced in parentheses after its resulting TASK-IDs below, for traceability back to the Epic and Definition of Done in that document.

**File paths:** shown as language-agnostic module paths matching `ARCHITECTURE.md`'s naming (e.g. `core/record_generator`), since implementation language/stack remains an open question per `IMPLEMENTATION_PLAN.md` §11. Once a stack is chosen, apply the appropriate file extension.

---

## Summary
**Total atomic tasks: 93**, across 9 phases.

| Phase | Name | Task Count | Task ID Range |
|---|---|---|---|
| 1 | Project Foundation | 15 | TASK-001–TASK-015 |
| 2 | Core Data Generation Engine | 16 | TASK-016–TASK-031 |
| 3 | Synthetic Data Providers | 12 | TASK-032–TASK-043 |
| 4 | Data Mutation Engine | 19 | TASK-044–TASK-062 |
| 5 | Metrics Engine | 7 | TASK-063–TASK-069 |
| 6 | Export Engine | 10 | TASK-070–TASK-079 |
| 7 | Validation | 5 | TASK-080–TASK-084 |
| 8 | Regression Test Suite | 5 | TASK-085–TASK-089 |
| 9 | Documentation | 4 | TASK-090–TASK-093 |

---

## Sequential Execution Order
Tasks are listed below in the exact order they should be executed: phase by phase, epic by epic, in dependency order. A task's Dependencies field lists the TASK-IDs that must be complete (merged) before it can start. Within a phase, tasks with no dependency on each other are candidates for parallel execution — see the **Parallel Execution Groups** section at the end of this document.

## Phase 1: Project Foundation

### Epic E1.1

#### TASK-001 — Initialize project repository structure (T1.1.1)
| Field | Detail |
|---|---|
| Description | Create the folder layout from ARCHITECTURE.md §3 (core/, profiles/, dictionaries/, dataset_configs/, pipeline/, tests/) as empty modules with placeholder files. |
| Inputs | ARCHITECTURE.md |
| Outputs | Folder/module skeleton committed to repo |
| Dependencies | None |
| Files Expected to Change | core/__init__, profiles/__init__, dictionaries/, dataset_configs/, pipeline/__init__, tests/__init__ |
| Unit Tests Required | CI check that expected folders exist (structural, no logic tests). |
| Acceptance Criteria | A fresh clone shows the exact folder layout specified in ARCHITECTURE.md §3 with no missing or extra top-level module directories. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

#### TASK-002 — Set up dependency management (T1.1.2)
| Field | Detail |
|---|---|
| Description | Choose and configure a package manager/manifest for the implementation language; pin versions. |
| Inputs | Language/stack decision (Open Question) |
| Outputs | Dependency manifest + lockfile |
| Dependencies | TASK-001 |
| Files Expected to Change | dependency manifest file, lockfile |
| Unit Tests Required | CI job performing a clean install, failing on any error. |
| Acceptance Criteria | A clean install from the manifest alone succeeds on a fresh machine with zero manual steps. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

#### TASK-003 — Configure linting & formatting (T1.1.3)
| Field | Detail |
|---|---|
| Description | Add a linter/formatter config enforcing one consistent style across the codebase. |
| Inputs | T1.1.2 manifest |
| Outputs | Lint/format config files |
| Dependencies | TASK-002 |
| Files Expected to Change | lint config, format config |
| Unit Tests Required | CI lint job. |
| Acceptance Criteria | Lint passes with zero errors on the empty skeleton; CI fails the build on any lint violation. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

#### TASK-004 — Set up test framework scaffolding (T1.1.4)
| Field | Detail |
|---|---|
| Description | Install and configure the unit test framework and a coverage reporter. |
| Inputs | T1.1.2 manifest |
| Outputs | Test runner config; example smoke test |
| Dependencies | TASK-002 |
| Files Expected to Change | test runner config, tests/test_smoke |
| Unit Tests Required | The smoke test itself. |
| Acceptance Criteria | `run tests` executes successfully with one passing smoke test and a coverage report is produced. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

#### TASK-005 — Set up CI pipeline skeleton (T1.1.5)
| Field | Detail |
|---|---|
| Description | Configure CI to run lint + tests on every pull request. |
| Inputs | T1.1.3, T1.1.4 |
| Outputs | CI pipeline config |
| Dependencies | TASK-003, TASK-004 |
| Files Expected to Change | CI pipeline config file |
| Unit Tests Required | Manual verification with a throwaway failing PR. |
| Acceptance Criteria | A PR with a deliberate lint error or failing test is blocked by CI; a clean PR passes. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

### Epic E1.2

#### TASK-006 — Define canonical schema data structure (part 1/2 of T1.2.1)
| Field | Detail |
|---|---|
| Description | Implement the 24-field record model from SCHEMA.md with correct types. |
| Inputs | SCHEMA.md |
| Outputs | Canonical record model (single source of truth) |
| Dependencies | TASK-001 |
| Files Expected to Change | core/schema |
| Unit Tests Required | Construct a record with all fields populated and assert types. |
| Acceptance Criteria | All 24 fields present with correct types matching SCHEMA.md exactly. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

#### TASK-007 — Enforce nullability rules at construction (part 2/2 of T1.2.1)
| Field | Detail |
|---|---|
| Description | Add construction-time enforcement so non-nullable fields (VoterID, FirstName, LastName, Street1, City, State, ZIP, County) cannot be omitted. |
| Inputs | Prior subtask's schema model |
| Outputs | Nullability-enforcing constructor/validator hook |
| Dependencies | TASK-006 |
| Files Expected to Change | core/schema |
| Unit Tests Required | Construct a record missing a required field and assert rejection; construct one missing only a nullable field and assert success. |
| Acceptance Criteria | Every non-nullable field in SCHEMA.md is enforced; every nullable field can be legitimately omitted. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

#### TASK-008 — Implement schema validation utility (T1.2.2)
| Field | Detail |
|---|---|
| Description | Standalone validator checking any record dict/object against the canonical schema, independent of how it was produced. |
| Inputs | T1.2.1 |
| Outputs | Reusable `validate_record()` utility |
| Dependencies | TASK-007 |
| Files Expected to Change | core/schema_validator |
| Unit Tests Required | Parametrized test covering every field's nullability and type rule. |
| Acceptance Criteria | Validator correctly accepts/rejects a table of valid and invalid test fixtures with zero misclassifications. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

#### TASK-009 — Unit tests for schema model edge cases (T1.2.3)
| Field | Detail |
|---|---|
| Description | Additional coverage beyond basic cases: empty strings vs null, boundary date formats, enum violations (Gender, RegistrationStatus). |
| Inputs | T1.2.1, T1.2.2 |
| Outputs | Extended schema test suite |
| Dependencies | TASK-008 |
| Files Expected to Change | tests/test_schema |
| Unit Tests Required | The edge-case suite itself. |
| Acceptance Criteria | ≥ 90% line coverage on the schema module; all edge cases pass. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

### Epic E1.3

#### TASK-010 — Define dataset config file format (part 1/2 of T1.3.1)
| Field | Detail |
|---|---|
| Description | Specify the schema for a dataset_configs/*.config file: record count, defect toggles/rates, column-mapping style, seed. |
| Inputs | DATASET_SPECS.md (10 dataset parameter sets) |
| Outputs | Config format specification |
| Dependencies | TASK-001 |
| Files Expected to Change | dictionaries/config_format_spec |
| Unit Tests Required | N/A (spec task) — validated indirectly by T1.3.2/T1.3.3 tasks. |
| Acceptance Criteria | Format covers every parameter used across all 10 dataset specs without a one-off hack. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

#### TASK-011 — Document config format with a worked example (part 2/2 of T1.3.1)
| Field | Detail |
|---|---|
| Description | Write reference documentation for the config format with one fully worked example config. |
| Inputs | Prior subtask's format spec |
| Outputs | Config format reference doc + example file |
| Dependencies | TASK-010 |
| Files Expected to Change | docs/config_format.md, dataset_configs/_example.config |
| Unit Tests Required | N/A — validated by confirming the example loads once T1.3.2 exists. |
| Acceptance Criteria | A developer unfamiliar with the format can author a valid config using only this doc. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

#### TASK-012 — Implement config loader/parser (T1.3.2)
| Field | Detail |
|---|---|
| Description | Parse a config file into the typed config object defined in T1.3.1. |
| Inputs | T1.3.1 |
| Outputs | `load_config()` utility |
| Dependencies | TASK-011 |
| Files Expected to Change | pipeline/config_loader |
| Unit Tests Required | Round-trip test: load a config, assert every field matches the source file. |
| Acceptance Criteria | Loader correctly parses the example config from T1.3.1 with all fields matching. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

#### TASK-013 — Implement config validator (T1.3.3)
| Field | Detail |
|---|---|
| Description | Fail fast on malformed configs (e.g. defect rates outside 0–100%, missing required keys). |
| Inputs | T1.3.1, T1.3.2 |
| Outputs | `validate_config()` utility |
| Dependencies | TASK-012 |
| Files Expected to Change | pipeline/config_validator |
| Unit Tests Required | Parametrized test with at least one violation per validation rule. |
| Acceptance Criteria | A table of malformed sample configs all raise clear, specific errors; the valid example config passes. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

#### TASK-014 — Author config files for Datasets 1–5 (part 1/2 of T1.3.4)
| Field | Detail |
|---|---|
| Description | Translate DATASET_SPECS.md's Datasets 1–5 (Perfect, County Export, Excel, Volunteer, Duplicate Heavy) into config files. |
| Inputs | DATASET_SPECS.md, T1.3.1 |
| Outputs | 5 config files under dataset_configs/ |
| Dependencies | TASK-013 |
| Files Expected to Change | dataset_configs/01_perfect.config ... 05_duplicate_heavy.config |
| Unit Tests Required | Loader test per config file (5 cases) asserting record count and defect rates match DATASET_SPECS.md. |
| Acceptance Criteria | All 5 configs pass validation and round-trip through the loader with values matching DATASET_SPECS.md exactly. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

#### TASK-015 — Author config files for Datasets 6–10 (part 2/2 of T1.3.4)
| Field | Detail |
|---|---|
| Description | Translate DATASET_SPECS.md's Datasets 6–10 (Household, Missing Data, Invalid Data, Multilingual, Real Simulation) into config files. |
| Inputs | DATASET_SPECS.md, T1.3.1 |
| Outputs | 5 config files under dataset_configs/ |
| Dependencies | TASK-014 |
| Files Expected to Change | dataset_configs/06_household.config ... 10_real_world_simulation.config |
| Unit Tests Required | Loader test per config file (5 cases) asserting record count and defect rates match DATASET_SPECS.md. |
| Acceptance Criteria | All 5 configs pass validation and round-trip through the loader with values matching DATASET_SPECS.md exactly. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

## Phase 2: Core Data Generation Engine

### Epic E2.1

#### TASK-016 — Define CountryProfile interface method signatures (part 1/2 of T2.1.1)
| Field | Detail |
|---|---|
| Description | Declare the 7-method interface: geographies(), name_pools(), phone_formats(), email_domains(), language_distribution(), voter_id_scheme(), address_format(). |
| Inputs | ARCHITECTURE.md §5 |
| Outputs | Abstract interface/base class |
| Dependencies | TASK-007 |
| Files Expected to Change | profiles/country_profile |
| Unit Tests Required | N/A directly (abstract) — covered by the contract test suite task. |
| Acceptance Criteria | All 7 methods declared with documented expected return shapes. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

#### TASK-017 — Document CountryProfile interface contract (part 2/2 of T2.1.1)
| Field | Detail |
|---|---|
| Description | Docstring/spec each of the 7 methods with expected return types and example values. |
| Inputs | Prior subtask's interface |
| Outputs | Documented interface reference |
| Dependencies | TASK-016 |
| Files Expected to Change | profiles/country_profile |
| Unit Tests Required | N/A (documentation task). |
| Acceptance Criteria | Every method has a clear, unambiguous documented contract a new profile author can follow without reading other profiles. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

#### TASK-018 — Implement contract test suite — happy path (part 1/2 of T2.1.2)
| Field | Detail |
|---|---|
| Description | Reusable test suite that runs against a complete, correct profile and confirms it passes. |
| Inputs | T2.1.1 |
| Outputs | `assert_profile_contract(profile)` utility (happy-path checks) |
| Dependencies | TASK-017 |
| Files Expected to Change | tests/profile_contract |
| Unit Tests Required | Run the suite against a stub fully-correct profile. |
| Acceptance Criteria | Suite passes cleanly against a deliberately correct stub profile. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

#### TASK-019 — Implement contract test suite — failure path (part 2/2 of T2.1.2)
| Field | Detail |
|---|---|
| Description | Extend the suite to fail with a clear, per-method error when a profile is incomplete or malformed. |
| Inputs | Prior subtask's suite |
| Outputs | Failure-path checks added to `assert_profile_contract()` |
| Dependencies | TASK-018 |
| Files Expected to Change | tests/profile_contract |
| Unit Tests Required | Run the suite against a stub profile missing one or more methods/fields. |
| Acceptance Criteria | Suite fails with a clear, specific error identifying exactly which capability is missing or malformed. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

### Epic E2.2

#### TASK-020 — Implement per-field synthetic value generators (part 1/4 of T2.2.1)
| Field | Detail |
|---|---|
| Description | Standalone generator functions for each schema field (name, address, phone, email, etc.) drawing from a supplied CountryProfile's pools. |
| Inputs | T1.2.1 (schema), T2.1.1 (profile interface) |
| Outputs | Per-field generator functions |
| Dependencies | TASK-007, TASK-017 |
| Files Expected to Change | core/record_generator |
| Unit Tests Required | Unit test per field generator asserting output matches the field's schema type/constraints. |
| Acceptance Criteria | Every schema field has a corresponding generator function producing schema-conformant values. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

#### TASK-021 — Implement record assembly (part 2/4 of T2.2.1)
| Field | Detail |
|---|---|
| Description | Compose the per-field generators into one function producing a single complete canonical record. |
| Inputs | Prior subtask's field generators |
| Outputs | `generate_one_record(profile, rng)` function |
| Dependencies | TASK-020 |
| Files Expected to Change | core/record_generator |
| Unit Tests Required | Assert a single generated record passes full schema validation (T1.2.2). |
| Acceptance Criteria | A single call produces one fully schema-valid record with no missing required fields. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

#### TASK-022 — Implement batch generation loop (part 3/4 of T2.2.1)
| Field | Detail |
|---|---|
| Description | Generate N records by repeatedly calling record assembly, ensuring performance is acceptable at Dataset-10 scale (2,000 records). |
| Inputs | Prior subtask's record assembly |
| Outputs | `generate_records(profile, n, seed)` function |
| Dependencies | TASK-021 |
| Files Expected to Change | core/record_generator |
| Unit Tests Required | Generate a 2,000-record batch and assert count and completion time are within acceptable bounds. |
| Acceptance Criteria | Function generates exactly N valid records for N up to 10,000 without error. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

#### TASK-023 — Distribution and schema-conformance tests for generated batches (part 4/4 of T2.2.1)
| Field | Detail |
|---|---|
| Description | Verify every record in a generated batch passes schema validation and that field distributions (e.g. city) roughly match profile weights. |
| Inputs | Prior subtask's batch generator |
| Outputs | Batch-level test suite |
| Dependencies | TASK-022 |
| Files Expected to Change | tests/test_record_generator |
| Unit Tests Required | Schema-conformance test across a full batch; distribution sanity check against profile weights. |
| Acceptance Criteria | Zero schema-validation failures across a 10,000-record batch; distributions are not obviously skewed relative to profile weights. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

#### TASK-024 — Implement seeded RNG wrapper (T2.2.2)
| Field | Detail |
|---|---|
| Description | Centralize all randomness behind one seeded RNG so reproducibility holds across the entire pipeline. |
| Inputs | None |
| Outputs | `SeededRNG` utility used by every later randomness-consuming module |
| Dependencies | TASK-001 |
| Files Expected to Change | core/seeded_rng |
| Unit Tests Required | Two independent runs with the same seed produce identical draw sequences. |
| Acceptance Criteria | Same seed always produces the same sequence of draws across all consumers, verified across ≥ 3 consumer modules. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

#### TASK-025 — Implement VoterID generation (T2.2.3)
| Field | Detail |
|---|---|
| Description | Generate VoterIDs per the profile's voter_id_scheme(), guaranteeing uniqueness within a run. |
| Inputs | T2.1.1, T2.2.2 |
| Outputs | `generate_voter_id()` utility |
| Dependencies | TASK-017, TASK-024 |
| Files Expected to Change | core/record_generator |
| Unit Tests Required | Uniqueness test at scale (10,000 IDs); format-conformance test against the profile's scheme. |
| Acceptance Criteria | Zero collisions across 10,000 generated IDs in a single run; format matches the profile's scheme. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

#### TASK-026 — Reproducibility unit tests for record_generator (T2.2.4)
| Field | Detail |
|---|---|
| Description | Dedicated test suite proving the whole record_generator is seed-deterministic end to end. |
| Inputs | T2.2.1, T2.2.2, T2.2.3 |
| Outputs | Reproducibility test suite |
| Dependencies | TASK-023, TASK-024, TASK-025 |
| Files Expected to Change | tests/test_reproducibility |
| Unit Tests Required | The reproducibility suite itself, run at both small and Dataset-10 scale. |
| Acceptance Criteria | Two full generation runs with the same seed and config are byte-identical when serialized. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

### Epic E2.3

#### TASK-027 — Implement address-based household clustering (part 1/2 of T2.3.1)
| Field | Detail |
|---|---|
| Description | Group a configured subset of generated records into shared-address households. |
| Inputs | T2.2.1 output (record batch) |
| Outputs | `group_households(records, config)` function (clustering only) |
| Dependencies | TASK-023 |
| Files Expected to Change | core/household_grouper |
| Unit Tests Required | Assert household membership implies address equality within the configured rate. |
| Acceptance Criteria | Clustering groups the configured proportion of records by shared Street1+City+ZIP. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

#### TASK-028 — Assign HouseholdID and verify membership integrity (part 2/2 of T2.3.1)
| Field | Detail |
|---|---|
| Description | Assign a common HouseholdID to each clustered group and verify no cross-household leakage. |
| Inputs | Prior subtask's clustering |
| Outputs | HouseholdID assignment logic |
| Dependencies | TASK-027 |
| Files Expected to Change | core/household_grouper |
| Unit Tests Required | Assert every record sharing a HouseholdID also shares Street1+City+ZIP, and vice versa. |
| Acceptance Criteria | All records sharing a HouseholdID share an address; no record has more than one HouseholdID. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

#### TASK-029 — Implement configurable household size distribution (T2.3.2)
| Field | Detail |
|---|---|
| Description | Support household sizes 2–5 per Dataset 6's spec, distributed per config. |
| Inputs | T2.3.1 |
| Outputs | Size-distribution parameter support in `group_households()` |
| Dependencies | TASK-028 |
| Files Expected to Change | core/household_grouper |
| Unit Tests Required | Distribution test at Dataset-6 scale (n=500). |
| Acceptance Criteria | Generated household size distribution matches configured distribution within statistical tolerance at n=500. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

#### TASK-030 — Implement unit-notation noise injection for households (T2.3.3)
| Field | Detail |
|---|---|
| Description | For a configured fraction of households, vary Street2 notation across members (Apt 101 / #101 / Unit 101). |
| Inputs | T2.3.1, T2.3.2 |
| Outputs | Unit-notation noise applied to Street2 within noisy households |
| Dependencies | TASK-029 |
| Files Expected to Change | core/household_grouper |
| Unit Tests Required | Assert household integrity (shared Street1+City+ZIP) is preserved despite Street2 noise. |
| Acceptance Criteria | Noisy households still share Street1+City+ZIP; only Street2 notation varies across members. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

#### TASK-031 — Unit tests for household grouping correctness (Dataset 6 spec) (T2.3.4)
| Field | Detail |
|---|---|
| Description | End-to-end correctness suite for the household grouper against Dataset 6's exact spec. |
| Inputs | T2.3.1–T2.3.3 |
| Outputs | Household grouper test suite |
| Dependencies | TASK-028, TASK-029, TASK-030 |
| Files Expected to Change | tests/test_household_grouper |
| Unit Tests Required | The suite itself, run against the Dataset-6 config. |
| Acceptance Criteria | A generated Dataset-6-scale run matches all Dataset 6 expected outcomes in DATASET_SPECS.md. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

## Phase 3: Synthetic Data Providers

### Epic E3.1

#### TASK-032 — Author geography data for 5 CA cities (part 1/2 of T3.1.1)
| Field | Detail |
|---|---|
| Description | Author city, ZIP pattern, county, precinct/district scheme for the first 5 of the 10 required CA cities. |
| Inputs | Original source doc (10 CA cities) |
| Outputs | Partial geography.data (5 cities) |
| Dependencies | TASK-017 |
| Files Expected to Change | profiles/us_california/geography.data |
| Unit Tests Required | Data-integrity test: every entry has all required fields populated. |
| Acceptance Criteria | 5 cities present with valid CA ZIP patterns and county assignments. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

#### TASK-033 — Author geography data for remaining 5 CA cities (part 2/2 of T3.1.1)
| Field | Detail |
|---|---|
| Description | Complete geography.data with the remaining 5 cities plus finalize the precinct/district scheme. |
| Inputs | Prior subtask's partial data |
| Outputs | Complete geography.data (10 cities) |
| Dependencies | TASK-032 |
| Files Expected to Change | profiles/us_california/geography.data |
| Unit Tests Required | Data-integrity test across all 10 entries. |
| Acceptance Criteria | All 10 cities present with valid CA ZIP patterns, county, and precinct/district assignments. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

#### TASK-034 — Author name pools for 5 language groups (part 1/2 of T3.1.2)
| Field | Detail |
|---|---|
| Description | Author first/last name pools tagged by likely-language for Spanish, Chinese, Vietnamese, Korean, and Japanese. |
| Inputs | DATASET_SPECS.md Dataset 9 |
| Outputs | Partial names.data (5 language groups) |
| Dependencies | TASK-017 |
| Files Expected to Change | profiles/us_california/names.data |
| Unit Tests Required | Data-integrity test: every entry has a non-empty language tag. |
| Acceptance Criteria | 5 language groups populated with pools large enough to avoid excessive repetition at ~90 records/group. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

#### TASK-035 — Author name pools for remaining 5 language groups (part 2/2 of T3.1.2)
| Field | Detail |
|---|---|
| Description | Complete names.data with Indian, Arabic, Russian, Persian, and Tagalog name pools. |
| Inputs | Prior subtask's partial data |
| Outputs | Complete names.data (10 language groups) |
| Dependencies | TASK-034 |
| Files Expected to Change | profiles/us_california/names.data |
| Unit Tests Required | Data-integrity test across all 10 groups. |
| Acceptance Criteria | All 10 language groups populated with a non-empty language tag on every entry. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

#### TASK-036 — Build phone format rules (T3.1.3)
| Field | Detail |
|---|---|
| Description | Define valid NANP formats plus the three invalid-phone patterns needed for Dataset 8 (too short, alpha characters, too many digits). |
| Inputs | DATASET_SPECS.md Dataset 8 |
| Outputs | phone_format.rules |
| Dependencies | TASK-017 |
| Files Expected to Change | profiles/us_california/phone_format.rules |
| Unit Tests Required | Format-generation test producing both the valid format and each invalid pattern on demand. |
| Acceptance Criteria | All three named invalid-phone patterns are producible and distinguishable. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

#### TASK-037 — Build email domain pool (T3.1.4)
| Field | Detail |
|---|---|
| Description | Author a realistic pool of email domains for synthetic address generation. |
| Inputs | None |
| Outputs | Email domain pool data |
| Dependencies | TASK-017 |
| Files Expected to Change | profiles/us_california/email_domains.data |
| Unit Tests Required | Data-integrity test. |
| Acceptance Criteria | Pool has enough variety to avoid unrealistic concentration on one domain. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

#### TASK-038 — Build language distribution weights (T3.1.5)
| Field | Detail |
|---|---|
| Description | Author CA-realistic weighting across the 10 language groups. |
| Inputs | T3.1.2 |
| Outputs | language_dist.data |
| Dependencies | TASK-035 |
| Files Expected to Change | profiles/us_california/language_dist.data |
| Unit Tests Required | Data-integrity + sum-to-1.0 test. |
| Acceptance Criteria | Weights sum to 1.0 and cover all 10 language groups. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

#### TASK-039 — Profile contract-compliance tests for us_california (T3.1.6)
| Field | Detail |
|---|---|
| Description | Run the contract test suite against the completed us_california profile. |
| Inputs | T3.1.1–T3.1.5, T2.1.2 |
| Outputs | Passing contract-compliance test result |
| Dependencies | TASK-033, TASK-035, TASK-036, TASK-037, TASK-038, TASK-019 |
| Files Expected to Change | profiles/us_california/profile |
| Unit Tests Required | The contract suite itself. |
| Acceptance Criteria | Zero contract failures against us_california. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

### Epic E3.2

#### TASK-040 — Implement column_synonyms.json loader (T3.2.1)
| Field | Detail |
|---|---|
| Description | Parse the existing dictionary file into an in-memory lookup (canonical field -> variant list). |
| Inputs | column_synonyms.json (already committed) |
| Outputs | `load_synonyms()` utility |
| Dependencies | TASK-007 |
| Files Expected to Change | core/column_mapper |
| Unit Tests Required | Round-trip test against the known file contents. |
| Acceptance Criteria | All 24 canonical fields load with their full variant list intact. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

#### TASK-041 — Implement header renaming logic per config style (part 1/2 of T3.2.2)
| Field | Detail |
|---|---|
| Description | Rename canonical field names to the dataset config's specified real-world header style, using the synonym dictionary. |
| Inputs | T3.2.1, config format |
| Outputs | Header-renaming component of `map_columns()` |
| Dependencies | TASK-040, TASK-015 |
| Files Expected to Change | core/column_mapper |
| Unit Tests Required | Per-dataset test asserting header names match DATASET_SPECS.md's documented examples (e.g. Dataset 2). |
| Acceptance Criteria | Renamed headers exactly match the target style for at least Datasets 1–5. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

#### TASK-042 — Implement column reorder and extra-column logic (part 2/2 of T3.2.2)
| Field | Detail |
|---|---|
| Description | Support column reordering and appending unknown/extra columns per config (needed for Dataset 3). |
| Inputs | Prior subtask's renaming logic |
| Outputs | Complete `map_columns(records, config)` function |
| Dependencies | TASK-041 |
| Files Expected to Change | core/column_mapper |
| Unit Tests Required | Per-dataset test asserting header order and extra-column presence for Datasets 3 and 10. |
| Acceptance Criteria | Output header set exactly matches the dataset config's specified style, order, and extra columns for all 10 configs. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

#### TASK-043 — Unit tests for column mapping correctness (T3.2.3)
| Field | Detail |
|---|---|
| Description | Comprehensive test coverage across all 24 fields and multiple header styles. |
| Inputs | T3.2.2 |
| Outputs | column_mapper test suite |
| Dependencies | TASK-042 |
| Files Expected to Change | tests/test_column_mapper |
| Unit Tests Required | The suite itself. |
| Acceptance Criteria | ≥ 90% coverage on column_mapper; every canonical field verified mappable to ≥ 2 known variants. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

## Phase 4: Data Mutation Engine

### Epic E4.1

#### TASK-044 — Implement exact duplicate injection (T4.1.1)
| Field | Detail |
|---|---|
| Description | Repeat a byte-identical record at the configured rate (15% for Dataset 5). |
| Inputs | T2.2.1 output |
| Outputs | Exact-duplicate injector |
| Dependencies | TASK-023 |
| Files Expected to Change | core/defect_injector |
| Unit Tests Required | Assert injected count and byte-identity. |
| Acceptance Criteria | Injected count matches configured rate exactly; duplicates are byte-identical to source. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

#### TASK-045 — Implement name-variant generator (part 1/2 of T4.1.2)
| Field | Detail |
|---|---|
| Description | Generate a controlled name variant (middle initial added/dropped, suffix added) for a given source record. |
| Inputs | T2.2.1 output |
| Outputs | Name-variant generator function |
| Dependencies | TASK-023 |
| Files Expected to Change | core/defect_injector |
| Unit Tests Required | Assert variant differs from source only in the specified name dimensions. |
| Acceptance Criteria | Variant generator produces exactly one of the two controlled variant types per call. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

#### TASK-046 — Wire near-duplicate injector using name-variant + shared phone/address (part 2/2 of T4.1.2)
| Field | Detail |
|---|---|
| Description | Combine the name-variant generator with shared phone/address to inject near-duplicates at the configured rate (10% for Dataset 5). |
| Inputs | Prior subtask's name-variant generator |
| Outputs | Near-duplicate injector |
| Dependencies | TASK-045 |
| Files Expected to Change | core/defect_injector |
| Unit Tests Required | Assert phone/address equality and controlled name difference across injected pairs. |
| Acceptance Criteria | Injected pairs share phone+address but differ only in the specified name variant dimensions, at the configured rate. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

#### TASK-047 — Implement phone-sharing pairing logic (part 1/2 of T4.1.3)
| Field | Detail |
|---|---|
| Description | Pair two otherwise-independent records to share the same phone number. |
| Inputs | T2.2.1 output |
| Outputs | Phone-sharing pairing function |
| Dependencies | TASK-023 |
| Files Expected to Change | core/defect_injector |
| Unit Tests Required | Assert phone equality between paired records. |
| Acceptance Criteria | Phone-sharing pairing produces exactly one shared phone per pair, with all other fields independently generated. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

#### TASK-048 — Wire phone-match-only duplicate injector (part 2/2 of T4.1.3)
| Field | Detail |
|---|---|
| Description | Apply phone-sharing pairing at the configured rate (5% for Dataset 5), ensuring name/address remain meaningfully different. |
| Inputs | Prior subtask's pairing logic |
| Outputs | Phone-match-only duplicate injector |
| Dependencies | TASK-047 |
| Files Expected to Change | core/defect_injector |
| Unit Tests Required | Assert phone equality and name/address inequality across injected pairs. |
| Acceptance Criteria | Injected pairs share phone only, at the configured rate, with name and address each meaningfully different. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

#### TASK-049 — Implement email-match-only duplicate injection (T4.1.4)
| Field | Detail |
|---|---|
| Description | Duplicates identifiable only by matching email, at the configured rate (5% for Dataset 5). |
| Inputs | T2.2.1 output |
| Outputs | Email-match injector |
| Dependencies | TASK-023 |
| Files Expected to Change | core/defect_injector |
| Unit Tests Required | Assert email equality and other-field inequality. |
| Acceptance Criteria | Injected pairs share email only, at the configured rate. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

#### TASK-050 — Ground-truth duplicate count tests (T4.1.5)
| Field | Detail |
|---|---|
| Description | Verify total injected duplicate counts and per-category breakdown match Dataset 5's configured rates exactly. |
| Inputs | T4.1.1–T4.1.4 |
| Outputs | Duplicate injection test suite |
| Dependencies | TASK-044, TASK-046, TASK-048, TASK-049 |
| Files Expected to Change | tests/test_defect_injector_duplicates |
| Unit Tests Required | The suite itself, run at Dataset 5 scale (1,000 records). |
| Acceptance Criteria | Category counts match configured rates exactly (integer rounding documented). |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

### Epic E4.2

#### TASK-051 — Implement per-field missingness for Phone/Email/DOB (part 1/2 of T4.2.1)
| Field | Detail |
|---|---|
| Description | Independently null Phone (20%), Email (30%), and DOB (15%) at their configured rates. |
| Inputs | T2.2.1 output |
| Outputs | Missingness injector (partial: 3 fields) |
| Dependencies | TASK-023 |
| Files Expected to Change | core/defect_injector |
| Unit Tests Required | Per-field rate assertion at n=800 for these 3 fields. |
| Acceptance Criteria | Actual missingness for Phone/Email/DOB is within statistical tolerance of configured rates at n=800. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

#### TASK-052 — Implement per-field missingness for ZIP/PreferredLanguage/Street2 (part 2/2 of T4.2.1)
| Field | Detail |
|---|---|
| Description | Extend the missingness injector to cover ZIP (5%), PreferredLanguage (40%), and Street2 (85%). |
| Inputs | Prior subtask's injector |
| Outputs | Complete missingness injector (6 fields) |
| Dependencies | TASK-051 |
| Files Expected to Change | core/defect_injector |
| Unit Tests Required | Per-field rate assertion at n=800 for these 3 additional fields. |
| Acceptance Criteria | Actual missingness for all 6 configured fields is within statistical tolerance of configured rates at n=800. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

#### TASK-053 — Missingness rate unit tests across seeds (T4.2.2)
| Field | Detail |
|---|---|
| Description | Statistical tolerance tests across multiple seeds to confirm the injector isn't systematically biased. |
| Inputs | T4.2.1 |
| Outputs | Missingness test suite |
| Dependencies | TASK-052 |
| Files Expected to Change | tests/test_defect_injector_missing |
| Unit Tests Required | The suite itself. |
| Acceptance Criteria | Rate holds within tolerance across ≥ 5 different seeds for all 6 fields. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

### Epic E4.3

#### TASK-054 — Implement invalid phone generator (T4.3.1)
| Field | Detail |
|---|---|
| Description | Generate the three invalid-phone patterns from Dataset 8 (too short, alpha characters, too many digits), evenly split. |
| Inputs | T3.1.3 |
| Outputs | Invalid-phone generator |
| Dependencies | TASK-036 |
| Files Expected to Change | core/defect_injector |
| Unit Tests Required | One assertion per pattern. |
| Acceptance Criteria | All three patterns are producible and individually distinguishable. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

#### TASK-055 — Implement invalid email generator (T4.3.2)
| Field | Detail |
|---|---|
| Description | Generate the four invalid-email patterns (missing @, missing domain, double @, no TLD). |
| Inputs | T3.1.4 |
| Outputs | Invalid-email generator |
| Dependencies | TASK-037 |
| Files Expected to Change | core/defect_injector |
| Unit Tests Required | One assertion per pattern. |
| Acceptance Criteria | All four patterns are producible and distinguishable. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

#### TASK-056 — Implement invalid ZIP generator (T4.3.3)
| Field | Detail |
|---|---|
| Description | Generate invalid ZIPs (alpha characters, wrong digit count). |
| Inputs | T3.1.1 |
| Outputs | Invalid-ZIP generator |
| Dependencies | TASK-033 |
| Files Expected to Change | core/defect_injector |
| Unit Tests Required | One assertion per pattern. |
| Acceptance Criteria | Both invalid patterns are producible. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

#### TASK-057 — Implement single-defect-per-record selection logic (part 1/2 of T4.3.4)
| Field | Detail |
|---|---|
| Description | Orchestrate the three invalid-field generators so each record receives exactly one invalid field, evenly distributed across categories. |
| Inputs | T4.3.1–T4.3.3 |
| Outputs | Single-defect selection/orchestration logic |
| Dependencies | TASK-054, TASK-055, TASK-056, TASK-008 |
| Files Expected to Change | core/defect_injector |
| Unit Tests Required | Assert exactly one field per record is targeted for invalidity, evenly distributed at Dataset 8 scale. |
| Acceptance Criteria | Category distribution is even across 700 records within statistical tolerance. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

#### TASK-058 — Guarantee clean-field validity and full-record regression test (part 2/2 of T4.3.4)
| Field | Detail |
|---|---|
| Description | Ensure the remaining 23 fields on each record pass schema validation and add a dedicated full-record regression test. |
| Inputs | Prior subtask's selection logic, T1.2.2 |
| Outputs | Clean-field guarantee + regression test |
| Dependencies | TASK-057 |
| Files Expected to Change | core/defect_injector, tests/test_defect_injector_invalid |
| Unit Tests Required | Full-record validation test asserting exactly one field fails schema validation per record. |
| Acceptance Criteria | At Dataset 8 scale (700 records), each record has exactly one schema-validation failure, localized to the intended field, with zero false positives on the other 23. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

### Epic E4.4

#### TASK-059 — Implement casing noise (T4.4.1)
| Field | Detail |
|---|---|
| Description | Randomize name-field casing (ALL CAPS / lowercase / MiXeD) at the configured rate (30% for Dataset 4). |
| Inputs | T2.2.1 output |
| Outputs | Casing noise injector |
| Dependencies | TASK-023 |
| Files Expected to Change | core/defect_injector |
| Unit Tests Required | Rate + recoverability (case-insensitive) assertions. |
| Acceptance Criteria | Rate matches config within tolerance; original value recoverable via case-insensitive comparison. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

#### TASK-060 — Implement whitespace noise (T4.4.2)
| Field | Detail |
|---|---|
| Description | Inject leading/trailing/double whitespace at the configured rate (10% for Dataset 4). |
| Inputs | T2.2.1 output |
| Outputs | Whitespace noise injector |
| Dependencies | TASK-023 |
| Files Expected to Change | core/defect_injector |
| Unit Tests Required | Rate + trim-equality assertions. |
| Acceptance Criteria | Rate matches config; trimmed value equals the original. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

#### TASK-061 — Implement street abbreviation inconsistency (T4.4.3)
| Field | Detail |
|---|---|
| Description | Vary 'Street' token abbreviation (Street/STREET/St./st) at the configured rate (25% for Dataset 4). |
| Inputs | T2.2.1 output |
| Outputs | Abbreviation noise injector |
| Dependencies | TASK-023 |
| Files Expected to Change | core/defect_injector |
| Unit Tests Required | Rate assertion + variant-coverage assertion. |
| Acceptance Criteria | Rate matches config; all four variant forms appear in a large enough sample. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

#### TASK-062 — Formatting noise unit tests (combined + blank rows) (T4.4.4)
| Field | Detail |
|---|---|
| Description | Combined test suite covering all three noise types plus Dataset 4's 3% blank-row case. |
| Inputs | T4.4.1–T4.4.3 |
| Outputs | Formatting noise test suite |
| Dependencies | TASK-059, TASK-060, TASK-061 |
| Files Expected to Change | tests/test_defect_injector_formatting |
| Unit Tests Required | The suite itself. |
| Acceptance Criteria | All Dataset 4 defect rates (including 3% blank rows) verified simultaneously on one generated batch. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

## Phase 5: Metrics Engine

### Epic E5.1

#### TASK-063 — Implement duplicate count calculator (T5.1.1)
| Field | Detail |
|---|---|
| Description | Report per-category duplicate counts directly from Phase 4's injection tags, not by re-detecting duplicates. |
| Inputs | T4.1.1–T4.1.4 tags |
| Outputs | Duplicate metrics function |
| Dependencies | TASK-050 |
| Files Expected to Change | core/metrics_calculator |
| Unit Tests Required | Assert calculator output equals injector's own record of what it injected. |
| Acceptance Criteria | Exact match to injection counts on Dataset 5. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

#### TASK-064 — Implement missing-field count calculator (T5.1.2)
| Field | Detail |
|---|---|
| Description | Report per-field missingness counts directly from the missingness injector's state. |
| Inputs | T4.2.1 state |
| Outputs | Missing-field metrics function |
| Dependencies | TASK-053 |
| Files Expected to Change | core/metrics_calculator |
| Unit Tests Required | Assert calculator output equals injector's ground truth. |
| Acceptance Criteria | Exact match to injection counts on Dataset 7. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

#### TASK-065 — Implement invalid-field count calculator (T5.1.3)
| Field | Detail |
|---|---|
| Description | Report invalid-field counts and category breakdown directly from the invalid-data injector's state. |
| Inputs | T4.3.4 state |
| Outputs | Invalid-field metrics function |
| Dependencies | TASK-058 |
| Files Expected to Change | core/metrics_calculator |
| Unit Tests Required | Assert calculator output equals injector's ground truth. |
| Acceptance Criteria | Exact match to injection counts on Dataset 8. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

#### TASK-066 — Implement mapping-rate calculator (T5.1.4)
| Field | Detail |
|---|---|
| Description | Report the percentage of columns successfully mapped by column_mapper for a given dataset run. |
| Inputs | T3.2.2 output |
| Outputs | Mapping-rate metrics function |
| Dependencies | TASK-043 |
| Files Expected to Change | core/metrics_calculator |
| Unit Tests Required | Assert rate calculation against a known-mapping-outcome fixture. |
| Acceptance Criteria | Reported rate matches Dataset 2's ≥95% target when run against Dataset 2's config. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

#### TASK-067 — Implement household metrics (T5.1.5)
| Field | Detail |
|---|---|
| Description | Report household count and average household size directly from the household grouper's state. |
| Inputs | T2.3.1–T2.3.4 state |
| Outputs | Household metrics function |
| Dependencies | TASK-031 |
| Files Expected to Change | core/metrics_calculator |
| Unit Tests Required | Assert calculator output equals grouper's own record of constructed households. |
| Acceptance Criteria | Exact match to grouping ground truth on Dataset 6. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

#### TASK-068 — Set up cross-dataset metrics ground-truth test harness (part 1/2 of T5.1.6)
| Field | Detail |
|---|---|
| Description | Build the harness that runs metrics_calculator against all 10 dataset configs and compares to injection ground truth. |
| Inputs | T5.1.1–T5.1.5 |
| Outputs | Ground-truth test harness (scaffolding) |
| Dependencies | TASK-063, TASK-064, TASK-065, TASK-066, TASK-067 |
| Files Expected to Change | tests/test_metrics_calculator |
| Unit Tests Required | Harness runs against 1 dataset as a smoke check. |
| Acceptance Criteria | Harness successfully runs metrics_calculator against a single dataset config and reports pass/fail. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

#### TASK-069 — Run harness against all 10 configs and close ground-truth gaps (part 2/2 of T5.1.6)
| Field | Detail |
|---|---|
| Description | Execute the harness across all 10 dataset configs; fix any discrepancies surfaced against upstream Phase 2–4 modules. |
| Inputs | Prior subtask's harness |
| Outputs | Zero-discrepancy metrics engine |
| Dependencies | TASK-068 |
| Files Expected to Change | tests/test_metrics_calculator, core/metrics_calculator (fixes as needed) |
| Unit Tests Required | The full 10-dataset harness run. |
| Acceptance Criteria | Zero discrepancies between reported metrics and injection ground truth across all 10 datasets. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

## Phase 6: Export Engine

### Epic E6.1

#### TASK-070 — Implement CSV header-row writer per config style (part 1/2 of T6.1.1)
| Field | Detail |
|---|---|
| Description | Write the header row for a record batch to CSV using the header names/order from column_mapper. |
| Inputs | T3.2.2 |
| Outputs | Header-row writer component |
| Dependencies | TASK-043 |
| Files Expected to Change | core/csv_writer |
| Unit Tests Required | Per-dataset header assertion for at least Datasets 1–5. |
| Acceptance Criteria | Header row exactly matches the dataset config's specified style for Datasets 1–5. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

#### TASK-071 — Implement CSV row-writer for record batch (part 2/2 of T6.1.1)
| Field | Detail |
|---|---|
| Description | Write the data rows for a full record batch to CSV, matching the header's field order. |
| Inputs | Prior subtask's header writer |
| Outputs | Complete `write_csv(records, config)` function |
| Dependencies | TASK-070 |
| Files Expected to Change | core/csv_writer |
| Unit Tests Required | Per-dataset header + row-count assertion for all 10 datasets. |
| Acceptance Criteria | Output header row and all data rows exactly match the dataset config's specified style for all 10 configs. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

#### TASK-072 — Implement unknown/extra column support (T6.1.2)
| Field | Detail |
|---|---|
| Description | Support Dataset 3's 2 blank columns and 4 extra non-canonical columns (Volunteer Notes, Call Status, Favorite Issue, Visited). |
| Inputs | T6.1.1 |
| Outputs | Extra-column support in CSV writer |
| Dependencies | TASK-071 |
| Files Expected to Change | core/csv_writer |
| Unit Tests Required | Column-count and column-name assertions on Dataset 3 output. |
| Acceptance Criteria | Dataset 3 output contains exactly 2 blank and 4 named extra columns. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

#### TASK-073 — Round-trip CSV tests (T6.1.3)
| Field | Detail |
|---|---|
| Description | Read back every written CSV and confirm parsed records match generated records exactly (accounting for intentional defects). |
| Inputs | T6.1.1, T6.1.2 |
| Outputs | Round-trip test suite |
| Dependencies | TASK-072 |
| Files Expected to Change | tests/test_csv_writer |
| Unit Tests Required | The suite itself, run against all 10 dataset shapes. |
| Acceptance Criteria | Zero data loss or corruption on round-trip for all 10 dataset shapes. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

### Epic E6.2

#### TASK-074 — Wire generate + group-households steps into orchestration (part 1/4 of T6.2.1)
| Field | Detail |
|---|---|
| Description | Chain record generation and household grouping for a single dataset config into one callable step. |
| Inputs | T2.2.1, T2.3.1, config |
| Outputs | Partial `run_dataset(config)` (generate+group) |
| Dependencies | TASK-072, TASK-069 |
| Files Expected to Change | pipeline/orchestrator |
| Unit Tests Required | Assert output record count and household assignment match config for one dataset. |
| Acceptance Criteria | Generate+group step produces the configured record count with households correctly assigned. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

#### TASK-075 — Wire defect-injection + column-mapping steps into orchestration (part 2/4 of T6.2.1)
| Field | Detail |
|---|---|
| Description | Extend orchestration to apply configured defect injectors and column mapping to the generated/grouped batch. |
| Inputs | Prior subtask, T4.x injectors, T3.2.2 |
| Outputs | Partial `run_dataset(config)` (+inject+map) |
| Dependencies | TASK-074 |
| Files Expected to Change | pipeline/orchestrator |
| Unit Tests Required | Assert injected defect rates and mapped headers match config for one dataset. |
| Acceptance Criteria | Inject+map step produces correctly defected and correctly header-mapped output for one dataset. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

#### TASK-076 — Wire CSV-write + metrics-computation steps into orchestration (part 3/4 of T6.2.1)
| Field | Detail |
|---|---|
| Description | Complete orchestration by writing CSV output and computing metrics for the dataset run. |
| Inputs | Prior subtask, T6.1.2, T5.1.6 |
| Outputs | Complete `run_dataset(config)` function |
| Dependencies | TASK-075 |
| Files Expected to Change | pipeline/orchestrator |
| Unit Tests Required | Assert CSV file and metrics report are produced and consistent for one dataset. |
| Acceptance Criteria | run_dataset(config) produces a correct CSV file and a correct metrics report for one dataset end to end. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

#### TASK-077 — Integration tests for run_dataset across all 10 configs (part 4/4 of T6.2.1)
| Field | Detail |
|---|---|
| Description | One integration test per dataset (10 total) asserting record count, defect rates, and metrics match DATASET_SPECS.md. |
| Inputs | Prior subtask's complete run_dataset |
| Outputs | 10 integration tests |
| Dependencies | TASK-076 |
| Files Expected to Change | tests/test_orchestrator_integration |
| Unit Tests Required | The 10 integration tests themselves. |
| Acceptance Criteria | All 10 datasets pass their individual integration test with output matching their DATASET_SPECS.md entry. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

#### TASK-078 — Implement CLI/entrypoint for all 10 configs (T6.2.2)
| Field | Detail |
|---|---|
| Description | Single command that runs run_dataset against every config in dataset_configs/ and writes all outputs. |
| Inputs | T6.2.1 |
| Outputs | CLI entrypoint |
| Dependencies | TASK-077 |
| Files Expected to Change | pipeline/cli |
| Unit Tests Required | End-to-end smoke test invoking the CLI. |
| Acceptance Criteria | One command produces all 10 CSVs and metrics reports from a clean checkout. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

#### TASK-079 — Orchestrator record-count unit tests (T6.2.3)
| Field | Detail |
|---|---|
| Description | Assert final record counts for all 10 datasets match DATASET_SPECS.md exactly (100/800/600/1,000/1,000/500/800/700/900/2,000). |
| Inputs | T6.2.2 |
| Outputs | Record-count test suite |
| Dependencies | TASK-078 |
| Files Expected to Change | tests/test_orchestrator_counts |
| Unit Tests Required | The suite itself. |
| Acceptance Criteria | Exact record-count match for all 10 datasets. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

## Phase 7: Validation

### Epic E7.1

#### TASK-080 — Implement schema-conformance validator over generated output (part 1/2 of T7.1.1)
| Field | Detail |
|---|---|
| Description | Run the schema validator against every record in every generated CSV. |
| Inputs | T6.2.2 output, T1.2.2 |
| Outputs | Full-output validation report (base) |
| Dependencies | TASK-078 |
| Files Expected to Change | pipeline/output_validator |
| Unit Tests Required | Validator run against Dataset 1 (should be 100% clean). |
| Acceptance Criteria | Zero unintentional schema violations on Dataset 1. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

#### TASK-081 — Exclude intentional Dataset 8 defects from false-positive flags (part 2/2 of T7.1.1)
| Field | Detail |
|---|---|
| Description | Extend the validator to recognize and exclude Dataset 8's intentionally invalid fields from being flagged as bugs. |
| Inputs | Prior subtask's validator, T4.3.4 ground truth |
| Outputs | Defect-aware validation report |
| Dependencies | TASK-080 |
| Files Expected to Change | pipeline/output_validator |
| Unit Tests Required | Validator run against Dataset 8, asserting only the intended field per record is flagged. |
| Acceptance Criteria | Zero unintentional schema violations across all 10 datasets, with Dataset 8's intentional defects correctly excluded. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

#### TASK-082 — Implement per-dataset expected-outcome comparison logic (part 1/2 of T7.1.2)
| Field | Detail |
|---|---|
| Description | Compare one dataset's metrics_calculator output against its DATASET_SPECS.md target values. |
| Inputs | T5.1.6, DATASET_SPECS.md |
| Outputs | Expected-outcome checker (single-dataset) |
| Dependencies | TASK-069 |
| Files Expected to Change | pipeline/expected_outcome_checker |
| Unit Tests Required | Run against Dataset 2, asserting the ≥95% mapping-rate target. |
| Acceptance Criteria | Checker correctly passes/fails a single dataset against its documented target. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

#### TASK-083 — Wire expected-outcome checker against all 10 datasets (part 2/2 of T7.1.2)
| Field | Detail |
|---|---|
| Description | Run the checker across all 10 datasets and produce a consolidated pass/fail report. |
| Inputs | Prior subtask's checker |
| Outputs | Consolidated expected-outcome report |
| Dependencies | TASK-082 |
| Files Expected to Change | pipeline/expected_outcome_checker |
| Unit Tests Required | The checker run across all 10 datasets. |
| Acceptance Criteria | All 10 datasets pass their documented DATASET_SPECS.md targets. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

#### TASK-084 — Manual review checklist for generated CSV samples (T7.1.3)
| Field | Detail |
|---|---|
| Description | Human-reviewable checklist for spot-checking a sample of each dataset's output for plausibility. |
| Inputs | T6.2.2 output |
| Outputs | Review checklist document + completed review notes |
| Dependencies | TASK-078 |
| Files Expected to Change | docs/manual_review_checklist.md |
| Unit Tests Required | N/A (manual task). |
| Acceptance Criteria | Checklist completed and signed off for all 10 datasets, with reviewer name/date recorded. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

## Phase 8: Regression Test Suite

### Epic E8.1

#### TASK-085 — Implement full-suite regression run script (generation phase) (part 1/2 of T8.1.1)
| Field | Detail |
|---|---|
| Description | Script that runs all 10 dataset configs through the orchestrator in one pass. |
| Inputs | T6.2.2 |
| Outputs | Regression run script (generation only) |
| Dependencies | TASK-083 |
| Files Expected to Change | pipeline/regression_runner |
| Unit Tests Required | The script itself, run manually once. |
| Acceptance Criteria | One command generates output for all 10 datasets successfully. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

#### TASK-086 — Wire validation and expected-outcome checks into regression script (part 2/2 of T8.1.1)
| Field | Detail |
|---|---|
| Description | Extend the regression script to run Phase 7's validator and expected-outcome checker after generation, producing a pass/fail summary. |
| Inputs | Prior subtask, T7.1.1, T7.1.2 |
| Outputs | Complete regression run script |
| Dependencies | TASK-085 |
| Files Expected to Change | pipeline/regression_runner |
| Unit Tests Required | The script itself, run manually once as acceptance. |
| Acceptance Criteria | One command runs generation + validation for all 10 datasets and produces a pass/fail summary. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

#### TASK-087 — Implement golden-snapshot storage and comparison logic (part 1/2 of T8.1.2)
| Field | Detail |
|---|---|
| Description | Store a committed golden-output snapshot (same seed) and implement byte-level comparison against fresh output. |
| Inputs | T8.1.1 |
| Outputs | Golden-snapshot diff checker (comparison logic) |
| Dependencies | TASK-086 |
| Files Expected to Change | pipeline/regression_runner, tests/golden_snapshots/ |
| Unit Tests Required | Comparison run against the current (matching) output, asserting no diff. |
| Acceptance Criteria | Diff checker correctly reports zero differences when output matches the golden snapshot. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

#### TASK-088 — Negative test and snapshot-update process documentation (part 2/2 of T8.1.2)
| Field | Detail |
|---|---|
| Description | Add a negative test proving a deliberate change is caught, and document the process for intentionally updating a snapshot. |
| Inputs | Prior subtask's diff checker |
| Outputs | Negative test + snapshot update process doc |
| Dependencies | TASK-087 |
| Files Expected to Change | tests/test_golden_snapshot, docs/snapshot_update_process.md |
| Unit Tests Required | Negative test: introduce a deliberate change and confirm the check fails. |
| Acceptance Criteria | An intentional code change that alters output is caught by the diff check; the update process is documented and requires an explicit, reviewed justification. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

#### TASK-089 — Wire regression suite into CI (T8.1.3)
| Field | Detail |
|---|---|
| Description | Run the full regression script (generation + validation + golden-snapshot diff) on every pull request. |
| Inputs | T8.1.2, T1.1.5 |
| Outputs | CI job |
| Dependencies | TASK-088, TASK-005 |
| Files Expected to Change | CI pipeline config |
| Unit Tests Required | Manual verification with a throwaway failing PR. |
| Acceptance Criteria | A PR that breaks reproducibility or an expected-outcome target is blocked by CI. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

## Phase 9: Documentation

### Epic E9.1

#### TASK-090 — Write developer setup guide (T9.1.1)
| Field | Detail |
|---|---|
| Description | README addendum covering install, running the generator, and running the regression suite. |
| Inputs | T1.1.2, T1.1.4, T8.1.1 |
| Outputs | Setup guide section |
| Dependencies | TASK-086 |
| Files Expected to Change | README.md |
| Unit Tests Required | N/A — validate via a dry-run walkthrough with someone not on the implementation team. |
| Acceptance Criteria | A developer unfamiliar with the project can get a working local run using only this guide. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

#### TASK-091 — Document config file format with worked 11th-dataset example (T9.1.2)
| Field | Detail |
|---|---|
| Description | Reference documentation for the config format, with a worked example of adding an 11th dataset. |
| Inputs | T1.3.1, T1.3.4 |
| Outputs | Config format reference doc |
| Dependencies | TASK-015 |
| Files Expected to Change | docs/config_format.md |
| Unit Tests Required | N/A — validate the worked example actually loads and validates. |
| Acceptance Criteria | Worked example is copy-pasteable and produces a valid new config. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

#### TASK-092 — Document CountryProfile extension guide (T9.1.3)
| Field | Detail |
|---|---|
| Description | Step-by-step guide for implementing a new region (e.g. india) against the CountryProfile interface. |
| Inputs | T2.1.1, T2.1.2, T3.1.6 |
| Outputs | CountryProfile extension guide |
| Dependencies | TASK-039 |
| Files Expected to Change | docs/country_profile_extension.md |
| Unit Tests Required | N/A — validate by having someone stub a second profile using only this guide. |
| Acceptance Criteria | Guide references every one of the 7 required interface capabilities with a concrete example. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

#### TASK-093 — Final review and sign-off of all documentation (T9.1.4)
| Field | Detail |
|---|---|
| Description | Review README, ARCHITECTURE, SCHEMA, DATASET_SPECS, column_synonyms.json, and the three new guides for consistency with the as-built system. |
| Inputs | All prior documentation |
| Outputs | Sign-off record |
| Dependencies | TASK-092 |
| Files Expected to Change | docs/signoff.md |
| Unit Tests Required | N/A (review task). |
| Acceptance Criteria | No contradictions between documentation and as-built behavior. |
| Estimated Complexity | Small (2–4 hours) |
| Definition of Done | Acceptance criteria met, required unit tests pass, PR merged with no unresolved review comments. |

---

## Parallel Execution Groups
The groups below list tasks that have no dependency relationship on each other and can be assigned to different engineers/agents simultaneously, provided their prerequisite groups have already landed.

**Phase 1 — Track A:** Linting/formatting config and test framework scaffolding both depend only on dependency management (TASK-002) and not on each other.

**Phase 1 — Track B:** Schema implementation (E1.2) and Configuration System (E1.3) both depend only on TASK-001 (repo structure) and can proceed in parallel with each other once it lands.

**Phase 2 — Track A:** Once T2.1.1's tasks land, us_california data-authoring tasks in Phase 3 (Track B below) can start in parallel with Phase 2's remaining Core Engine tasks — they only need the interface shape, not the record generator itself.

**Phase 3 — Track B (Regional Data):** Geography data, name pools, phone rules, email domains are independent data-authoring tasks with no dependency on each other beyond the CountryProfile interface — assign to different contributors.

**Phase 3 — Track D (Mapping):** Column synonym integration (E3.2) depends only on the already-committed column_synonyms.json and the schema task — proceeds independently of all of Phase 4.

**Phase 4 — Track C (Defect Injectors):** Duplicate injection (E4.1), missing data injection (E4.2), and formatting noise injection (E4.4) each depend only on the record generator (Phase 2) and have no dependency on each other. Invalid data injection (E4.3) additionally needs Phase 3's phone/email/ZIP data.

**Phase 9 — Documentation:** Setup guide, config format guide, and CountryProfile extension guide each depend on different upstream tasks and can be written in parallel; final sign-off is the only serialization point.

**Do not parallelize across these boundaries** (same serialization points as `IMPLEMENTATION_PLAN.md` §6): Phase 5 (Metrics Engine) requires Phase 4 substantially complete; Phase 6's orchestrator requires Phase 5 complete; Phases 7–9 are inherently sequential passes over a complete system.

---

## Traceability Index (TASK-ID → original IMPLEMENTATION_PLAN.md task)
| Original Task | TASK-IDs |
|---|---|
| T1.1.1 | TASK-001 |
| T1.1.2 | TASK-002 |
| T1.1.3 | TASK-003 |
| T1.1.4 | TASK-004 |
| T1.1.5 | TASK-005 |
| T1.2.1 | TASK-006, TASK-007 |
| T1.2.2 | TASK-008 |
| T1.2.3 | TASK-009 |
| T1.3.1 | TASK-010, TASK-011 |
| T1.3.2 | TASK-012 |
| T1.3.3 | TASK-013 |
| T1.3.4 | TASK-014, TASK-015 |
| T2.1.1 | TASK-016, TASK-017 |
| T2.1.2 | TASK-018, TASK-019 |
| T2.2.1 | TASK-020, TASK-021, TASK-022, TASK-023 |
| T2.2.2 | TASK-024 |
| T2.2.3 | TASK-025 |
| T2.2.4 | TASK-026 |
| T2.3.1 | TASK-027, TASK-028 |
| T2.3.2 | TASK-029 |
| T2.3.3 | TASK-030 |
| T2.3.4 | TASK-031 |
| T3.1.1 | TASK-032, TASK-033 |
| T3.1.2 | TASK-034, TASK-035 |
| T3.1.3 | TASK-036 |
| T3.1.4 | TASK-037 |
| T3.1.5 | TASK-038 |
| T3.1.6 | TASK-039 |
| T3.2.1 | TASK-040 |
| T3.2.2 | TASK-041, TASK-042 |
| T3.2.3 | TASK-043 |
| T4.1.1 | TASK-044 |
| T4.1.2 | TASK-045, TASK-046 |
| T4.1.3 | TASK-047, TASK-048 |
| T4.1.4 | TASK-049 |
| T4.1.5 | TASK-050 |
| T4.2.1 | TASK-051, TASK-052 |
| T4.2.2 | TASK-053 |
| T4.3.1 | TASK-054 |
| T4.3.2 | TASK-055 |
| T4.3.3 | TASK-056 |
| T4.3.4 | TASK-057, TASK-058 |
| T4.4.1 | TASK-059 |
| T4.4.2 | TASK-060 |
| T4.4.3 | TASK-061 |
| T4.4.4 | TASK-062 |
| T5.1.1 | TASK-063 |
| T5.1.2 | TASK-064 |
| T5.1.3 | TASK-065 |
| T5.1.4 | TASK-066 |
| T5.1.5 | TASK-067 |
| T5.1.6 | TASK-068, TASK-069 |
| T6.1.1 | TASK-070, TASK-071 |
| T6.1.2 | TASK-072 |
| T6.1.3 | TASK-073 |
| T6.2.1 | TASK-074, TASK-075, TASK-076, TASK-077 |
| T6.2.2 | TASK-078 |
| T6.2.3 | TASK-079 |
| T7.1.1 | TASK-080, TASK-081 |
| T7.1.2 | TASK-082, TASK-083 |
| T7.1.3 | TASK-084 |
| T8.1.1 | TASK-085, TASK-086 |
| T8.1.2 | TASK-087, TASK-088 |
| T8.1.3 | TASK-089 |
| T9.1.1 | TASK-090 |
| T9.1.2 | TASK-091 |
| T9.1.3 | TASK-092 |
| T9.1.4 | TASK-093 |
