# Vinvite Synthetic Voter Dataset Generator — Architecture

## 1. Purpose

Produce a reusable, deterministic-when-seeded system that generates synthetic
(no real PII) campaign voter export files for testing the Vinvite Campaign
Data Engine: Import Wizard, Column Mapping Engine, Data Quality Engine,
Household Detection, Duplicate Detection, Language Estimation, and Campaign
Readiness Report.

This document specifies architecture only — module boundaries, data flow,
and extension points. It does not contain implementation code.

## 2. Design principles

- **Country-profile abstraction first.** Nothing in the core engine should
  assume US/California. All region-specific facts (address formats, phone
  formats, ZIP/postcode patterns, name pools, language distribution, voter
  ID conventions, admin divisions like county/precinct/district) live behind
  a `CountryProfile` interface. MVP ships one profile: `us_california`.
- **Defects are parameters, not hardcoded datasets.** A single core
  "record generator + defect injector" pipeline produces all 10 datasets.
  Each dataset is a *configuration* (which defects, at what rate, on which
  fields) applied to the same underlying generator — not 10 separate
  code paths. This keeps the 10 datasets consistent with each other and
  makes adding an 11th dataset a config change, not new code.
- **Correct-by-construction expected outcomes.** Whatever the future
  code-generation phase does, it must derive "expected" metrics (dup count,
  missing count, invalid count, mapping %) from the same generation
  parameters used to build the dataset — never hand-authored — so the
  regression suite can't silently drift from the data it's supposed to
  describe.
- **Reproducibility.** Every dataset generation run takes a seed. Same seed
  + same config = byte-identical output. This matters for regression
  testing (CI must get the same fixtures every run) and for debugging
  (a failing test run must be reproducible).

## 3. Module map

```
vinvite-synth/
├── core/
│   ├── record_generator      # builds one canonical-schema voter record
│   ├── household_grouper     # clusters records into households by address
│   ├── defect_injector       # applies configured defects to a record set
│   ├── column_mapper         # renames/reorders/drops columns per dataset config
│   └── metrics_calculator    # derives expected-outcome metrics from config + data
│
├── profiles/
│   ├── country_profile.interface   # abstract contract (see §5)
│   └── us_california/
│       ├── geography.data     # cities, ZIPs, counties, precincts, districts
│       ├── names.data         # first/last name pools, tagged by likely-language
│       ├── phone_format.rules # NANP formats + malformed variants
│       └── language_dist.data # CA-realistic language distribution weights
│
├── dictionaries/
│   └── column_synonyms.json   # canonical field -> known real-world header variants
│
├── dataset_configs/
│   ├── 01_perfect.config
│   ├── 02_county_export.config
│   ├── 03_excel_spreadsheet.config
│   ├── 04_volunteer_file.config
│   ├── 05_duplicate_heavy.config
│   ├── 06_household.config
│   ├── 07_missing_data.config
│   ├── 08_invalid_data.config
│   ├── 09_multilingual.config
│   └── 10_real_world_simulation.config
│
└── pipeline/
    └── orchestrator          # for each config: generate -> group households
                               # -> inject defects -> map columns -> write CSV
                               # -> compute + write expected-outcome JSON
```

## 4. Data flow

```
CountryProfile (us_california)
        │
        ▼
record_generator ──► N canonical-schema records (clean, fully populated)
        │
        ▼
household_grouper ──► assigns HouseholdID by shared address (rate configurable)
        │
        ▼
defect_injector ──► applies dataset-specific defects:
        │             duplicates, missing fields, invalid formats,
        │             case/whitespace noise, unknown extra columns
        ▼
column_mapper ──► renames canonical headers to the dataset's real-world
        │           header variants (pulled from column_synonyms.json),
        │           reorders columns, optionally drops/adds columns
        ▼
metrics_calculator ──► computes actual dup/missing/invalid/mapping-%
        │               counts from the *known* defect-injection state
        │               (ground truth, not inference)
        ▼
   ┌────┴─────┐
   ▼          ▼
CSV file   metadata JSON (skipped in this deliverable per current scope —
(output)   schema left for a later phase)
```

## 5. `CountryProfile` interface (contract, not code)

Each profile must supply:

| Capability | Description |
|---|---|
| `geographies()` | List of (city, state/region, ZIP/postcode pattern, county, precinct/district scheme) |
| `name_pools()` | First/last name pools, each entry tagged with a likely-language label for the language-estimation test dataset |
| `phone_formats()` | Valid format(s) + a set of "plausible but invalid" formats for negative testing |
| `email_domains()` | Realistic domain pool |
| `language_distribution()` | Weighted list of preferred-language values realistic for the region |
| `voter_id_scheme()` | Format rule for VoterID (e.g. state-specific prefix/length) |
| `address_format()` | Street naming conventions, unit/apartment notation variants |

`us_california` is the only implementation required for MVP. Adding
`india`, `uk`, `canada`, `australia` later means writing a new folder under
`profiles/` that satisfies this same contract — the core pipeline and all
10 dataset configs are region-agnostic and untouched.

## 6. Extension points

- **New dataset scenario** → add one config file under `dataset_configs/`;
  no core code changes.
- **New region** → add one folder under `profiles/`; no core code changes.
- **New defect type** (e.g. transposed digits, OCR-style errors) → add one
  handler to `defect_injector`; becomes available to all dataset configs.
- **New canonical field** → update the schema (see `SCHEMA.md`) and
  `column_synonyms.json`; existing profiles need their pools extended but
  the pipeline shape is unchanged.

## 7. Out of scope for this deliverable

- Actual implementation code
- The 10 expected-outcome JSON files and their schema (explicitly descoped)
- CI/regression harness that runs the Campaign Data Engine against the
  fixtures (belongs to the Campaign Data Engine repo, not the generator)
