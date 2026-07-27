# Dataset Specifications (v1.0)

Resolves the parameter gaps in the original requirements doc into concrete,
buildable numbers. Record counts follow the "keep it small for fast review"
direction — scale these up later by editing the config values only.

All datasets draw from the `us_california` profile (10 cities listed in the
source doc). All defect injection is seeded and logged, so
`metrics_calculator` can report ground-truth counts rather than inferred
ones.

---

### Dataset 1 — Perfect Campaign Dataset
- **Records:** 100
- **Config:** No defects. Standard canonical column names, standard order.
  All fields populated including PreferredLanguage and HouseholdID.
  Households: each voter defaults to its own household unless address
  happens to coincide with another generated voter (rare at this size).
- **Expected:** 100% import success, 0 warnings, Readiness = 100%.

### Dataset 2 — County Election Export
- **Records:** 800
- **Config:** Columns renamed to the doc's abbreviated export-style headers
  (`VOTER_ID`, `FNAME`, `LNAME`, `ADDR1`, `CITY_NAME`, `ZIP5`,
  `PRECINCT_CODE`, `CELL_PHONE`, `EMAIL_ADDR`, `LANG_PREF`, ...). Column
  order randomized per run (seeded). No data-quality defects — this
  dataset isolates the mapping engine, not the QA engine.
- **Expected:** ≥95% auto-mapping, minimal manual mapping.

### Dataset 3 — Excel Spreadsheet Export
- **Records:** 600
- **Config:** Random column order; 2 fully blank columns inserted; 4 extra
  non-canonical columns appended (`Volunteer Notes`, `Call Status`,
  `Favorite Issue`, `Visited`). No other defects.
- **Expected:** Unknown columns ignored/flagged, not mapped; canonical
  fields still map normally.

### Dataset 4 — Volunteer Maintained File
- **Records:** 1,000
- **Config:**
  - 30% of name fields get randomized casing (ALL CAPS / lowercase / MiXeD)
  - 25% of address "Street" tokens use inconsistent abbreviation
    (`Street` / `STREET` / `St.` / `st`)
  - 10% of records get leading/trailing/double whitespace injected
  - 3% blank rows (all fields empty except VoterID)
- **Expected:** Whitespace normalized, capitalization corrected on import;
  Readiness score reduced proportionally to blank rows.

### Dataset 5 — Duplicate Heavy Dataset
- **Records:** 1,000 (≈700 unique underlying voters)
- **Config:**
  - 15% exact duplicate rows (byte-identical record repeated)
  - 10% near-duplicates: same person, name variant (e.g. middle initial
    added/dropped, "Jr." suffix added) but same phone AND address
  - 5% duplicates identifiable only by matching phone (name/address differ
    slightly — simulates remarriage/name change scenarios)
  - 5% duplicates identifiable only by matching email
- **Expected:** Duplicate report flags all four categories separately;
  ground-truth dup count matches `metrics_calculator` output exactly.

### Dataset 6 — Household Dataset
- **Records:** 500
- **Config:** 60% of records grouped into households of size 2–5 sharing
  Street1+City+ZIP. Of those households, 20% use inconsistent apartment
  notation across members (`Apt 101` vs `#101` vs `Unit 101`) to test that
  household grouping tolerates unit-format noise. Remaining 40% are
  single-occupant households.
- **Expected:** Household grouping recovers all constructed households
  despite unit-notation noise; average household size reported matches
  ground truth.

### Dataset 7 — Missing Data Dataset
- **Records:** 800
- **Config:** Independent random missingness applied per field:
  - Phone missing: 20%
  - Email missing: 30%
  - DOB missing: 15%
  - ZIP missing: 5%
  - PreferredLanguage missing: 40%
  - Street2 missing: 85% (realistic — most voters have no unit)
- **Expected:** Readiness score reduced in proportion to missingness on
  weighted-important fields (Phone/Email weighted higher than Street2).

### Dataset 8 — Invalid Data Dataset
- **Records:** 700
- **Config:** Every record gets exactly one deliberately invalid field,
  evenly distributed across:
  - Invalid phone (too short, alpha characters, too many digits) — the
    three example patterns from the source doc, evenly split
  - Invalid email (missing @, missing domain, double @, no TLD)
  - Invalid ZIP (alpha characters, wrong digit count)
- **Expected:** Validation engine flags every deliberately-invalid field;
  zero false negatives, zero false positives on the clean 23 other fields
  per record.

### Dataset 9 — Multilingual Dataset
- **Records:** 900
- **Config:** Names drawn from language-tagged pools (Spanish, Chinese,
  Vietnamese, Korean, Japanese, Indian, Arabic, Russian, Persian, Tagalog —
  ~90 records per language group). PreferredLanguage field populated for
  60% of records (sometimes matching the name's likely language, sometimes
  not — real voters don't always share a household's assumed language)
  and blank for 40%, to be estimated.
- **Expected:** Language-estimation engine's guesses are compared against
  the *name-pool tag* (a heuristic ground truth, not a certainty) —
  documented in the README as an estimate-vs-estimate comparison, not a
  hard pass/fail.

### Dataset 10 — Real Campaign Simulation
- **Records:** 2,000
- **Config:** Union of all defect types above at reduced individual rates
  (roughly half the intensity of each dedicated dataset), plus randomized
  column order, plus unknown extra columns, plus volunteer-note noise.
  This is the only dataset combining every defect class simultaneously.
- **Expected:** No single expected metric in isolation — this is a stress
  test. Engine should complete without crashing and produce a Readiness
  score meaningfully lower than Dataset 1 but the import should still
  succeed (no fatal parse failures).

---

## Total record count
100 + 800 + 600 + 1,000 + 1,000 + 500 + 800 + 700 + 900 + 2,000 = **8,400
records** across 10 files (small-scale pass; multiply per-dataset counts by
a constant factor later to approach the original ~100k target once the
architecture is validated).
