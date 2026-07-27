# Vinvite Synthetic Voter Dataset Generator — Architecture Deliverable

This deliverable defines the **architecture and specifications** for a
synthetic voter dataset generator used to test the Vinvite Campaign Data
Engine (Import Wizard, Column Mapping Engine, Data Quality Engine,
Household Detection, Duplicate Detection, Language Estimation, Campaign
Readiness Report). It does not include implementation code — a developer
(or Replit/Codex) can build directly from these documents.

All data described is entirely synthetic. No real voter records, real
people, or real PII are used or referenced anywhere in this suite.

## Files in this deliverable

| File | Purpose |
|---|---|
| `ARCHITECTURE.md` | Module map, data flow, country-profile plugin interface, extension points |
| `SCHEMA.md` | The 24-field canonical Vinvite schema every dataset ultimately maps into |
| `DATASET_SPECS.md` | Concrete, buildable parameters for all 10 datasets (record counts, defect rates, expected outcomes) |
| `column_synonyms.json` | Data artifact: canonical field → real-world header variants, for the mapping engine |
| `README.md` | This file |

## Dataset suite overview

| # | Name | Purpose | Records (initial scale) |
|---|---|---|---|
| 1 | Perfect Campaign Dataset | Validate ideal import path | 100 |
| 2 | County Election Export | Test column mapping via abbreviated headers | 800 |
| 3 | Excel Spreadsheet | Test blank/extra/unknown columns | 600 |
| 4 | Volunteer Maintained File | Test whitespace/casing normalization | 1,000 |
| 5 | Duplicate Heavy Dataset | Test exact + near-duplicate detection | 1,000 |
| 6 | Household Dataset | Test household grouping incl. unit-notation noise | 500 |
| 7 | Missing Data Dataset | Test missing-value handling & readiness scoring | 800 |
| 8 | Invalid Data Dataset | Test field-level validation | 700 |
| 9 | Multilingual Dataset | Test language estimation | 900 |
| 10 | Real Campaign Simulation | Combined stress test | 2,000 |

Record counts are intentionally kept small in this first pass for fast
review. Each count is a single config value in `DATASET_SPECS.md` — scale
toward the original ~100,000-voter target by multiplying once the
architecture is validated against the real engine.

## Pass/fail criteria, by category

- **Mapping tests (Datasets 2, 3):** pass if auto-mapping rate meets or
  exceeds the documented target and no canonical field is mis-mapped.
- **Normalization tests (Dataset 4):** pass if casing/whitespace defects
  are corrected without altering the underlying identity of the record.
- **Duplicate detection (Dataset 5):** pass if all four duplicate
  categories (exact, near-dup by name, phone-match, email-match) are
  flagged, with zero false positives among the ~30% of records that are
  intentionally unique.
- **Household detection (Dataset 6):** pass if constructed households are
  fully recovered despite apartment-notation inconsistency, and reported
  average household size matches ground truth.
- **Missing data (Dataset 7):** pass if the Readiness score decreases
  monotonically with missingness and weights higher-value fields (Phone,
  Email) more heavily than lower-value ones (Street2).
- **Invalid data (Dataset 8):** pass if every deliberately invalid field
  is flagged and no clean field is falsely flagged.
- **Language estimation (Dataset 9):** this is inherently a heuristic
  comparison, not a hard ground truth — a person's name is a signal for
  likely language, not proof of it. Treat the name-pool tag as the
  estimate to compare against, and evaluate by rate of reasonable
  agreement rather than 100%-match.
- **Stress test (Dataset 10):** pass if the import pipeline completes
  without fatal errors across the combined defect load, and the reported
  Readiness score is meaningfully lower than Dataset 1's.

## Explicitly out of scope for this deliverable

- Generator implementation code
- The 10 expected-outcome metadata JSON files and their schema (descoped
  by request — can be added as a follow-on once code exists to compute
  ground-truth metrics from generation parameters)
- CI/regression harness wiring against the live Campaign Data Engine

## Future expansion: country profiles

The architecture isolates all California/US-specific assumptions behind a
`CountryProfile` interface (see `ARCHITECTURE.md` §5). Adding a new market
(India, UK, Canada, Australia, etc.) means implementing that interface for
the new region — no changes to the core pipeline or the 10 dataset configs
are required.
