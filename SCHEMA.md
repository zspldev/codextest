# Vinvite Canonical Schema (v1.0)

Every generated dataset — regardless of how messy — represents data that,
once correctly mapped, conforms to this schema. This is the target schema
the Column Mapping Engine maps *into*.

| Field | Type | Nullable | Notes |
|---|---|---|---|
| VoterID | string | No | Unique per record; region-specific format via `CountryProfile.voter_id_scheme()` |
| Prefix | string | Yes | Mr., Ms., Dr., etc. |
| FirstName | string | No | |
| MiddleName | string | Yes | |
| LastName | string | No | |
| Suffix | string | Yes | Jr., Sr., III, etc. |
| Gender | enum(M,F,U) | Yes | U = unknown/unspecified |
| DOB | date (YYYY-MM-DD) | Yes | |
| Age | integer | Yes | Derived from DOB where present; independently generated where DOB is deliberately missing |
| Street1 | string | No | |
| Street2 | string | Yes | Apt/unit/suite |
| City | string | No | |
| State | string | No | 2-letter code (MVP: always "CA") |
| ZIP | string | No | 5-digit (region-specific pattern via profile) |
| County | string | No | |
| Precinct | string | Yes | |
| District | string | Yes | |
| Phone | string | Yes | Format varies by dataset (see defect rules) |
| Email | string | Yes | |
| PreferredLanguage | string | Yes | ISO-ish language name, not a locale code (matches source doc's examples) |
| Party | string | Yes | |
| HouseholdID | string | Yes | Assigned by `household_grouper`, shared across co-resident records |
| Latitude | float | Yes | Approximate, derived from City/ZIP centroid + jitter |
| Longitude | float | Yes | Same as above |
| RegistrationStatus | enum(Active,Inactive,Pending) | Yes | |
| LastUpdated | date (YYYY-MM-DD) | Yes | |
| Notes | string | Yes | Free text; used to simulate volunteer-entered noise |

**Field count: 24.** This is the field set `column_synonyms.json` provides
variant headers for.
