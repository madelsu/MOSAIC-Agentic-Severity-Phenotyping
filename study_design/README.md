# MOSAIC — Pharmacoepidemiology Cohort Construction

> **Notebook**: `mosaic_cohort_construction.ipynb`  
> **Purpose**: Build a fully eligible patient cohort from the Synthea Coherent dataset, ready for downstream severity classification (MOSAIC pipeline) and survival analysis (Cox regression notebook).  
> **Disease shown**: Type 2 Diabetes Mellitus (T2D) — designed to be reused for any condition with minimal changes.

---

## What this notebook does

This notebook takes raw Synthea Coherent CSV files and produces a clean, analysis-ready cohort table. For each eligible patient it computes:

- Their observation period (first and last recorded clinical event)
- Their disease first diagnosis date
- Their first treatment date
- Three survival analysis index dates (first diagnosis, first treatment, random date)
- One cross-sectional index date (latest)
- Eligibility flags for each index date
- Mortality status and cause of death

The output is a single CSV file per disease that feeds directly into:
1. The **MOSAIC LLM pipeline** (severity classification per index date)
2. The **analysis notebook** (Cox proportional hazards regression)

---

## How to reuse for a new disease

Only **Cell 2** needs to change. Update:
- `DISEASE_NAME`
- `CONDITION_SEARCH_KEYWORDS`
- `TREATMENT_SEARCH_KEYWORDS`
- `EXCLUDE_KEYWORDS` (if needed)

Run all cells in order. Everything else is fully automatic.

---

## Cell-by-cell documentation

### Cell 1 — Setup & Library Imports

**What it does:**  
Installs and imports all Python libraries needed for cohort construction. Defines one global utility function used throughout the notebook.

**Libraries installed:**
- `openpyxl` — for writing Excel output files

**Libraries imported:**
- `pandas` — data loading, manipulation, and CSV processing
- `numpy` — numerical operations
- `os`, `re`, `warnings` — standard Python utilities

**Global utility defined:**

```python
def to_naive(series):
    """Parse to datetime and strip timezone if present."""
```

This function is used in every subsequent cell that handles dates. It is needed because Synthea mixes timezone-aware and timezone-naive timestamps across different CSV files — without stripping timezone info, date comparisons fail with a `TypeError`. The function parses any column to datetime and calls `.dt.tz_localize(None)` if a timezone is detected.

**When to re-run:** Every time you open the notebook. No changes needed between diseases.

---

### Cell 2 — Paths, Parameters & Disease Configuration

**What it does:**  
Three things in one cell: mounts Google Drive, auto-detects the study period by scanning all clinical CSV files, and defines all disease-specific configuration.

**Part A — Google Drive mount:**  
Connects to your Google Drive so all subsequent cells can read and write files.

**Part B — Study period auto-detection:**  
Scans all 12 clinical CSV files and finds the earliest and latest date recorded anywhere in the dataset:

```
DATASET_START_DATE → first ever recorded clinical event across all files
DATASET_END_DATE   → last ever recorded clinical event across all files
```

This scan covers: `conditions`, `observations`, `medications`, `encounters`, `procedures`, `careplans`, `immunizations`, `imaging_studies`, `devices`, `allergies`, `supplies`, `patients` (including birth and death dates).

Each file's own date range is printed as it is scanned, giving you a clear picture of the data coverage per file. The overall study period is printed at the end.

**Why `DATASET_END_DATE` matters — the follow-up eligibility fix:**  
A critical design decision. For follow-up eligibility we check:

```
DATASET_END_DATE - index_date >= 5 years
```

rather than:

```
patient OBS_END - index_date >= 5 years   ← WRONG
```

The reason: for a deceased patient, `OBS_END` equals their death date. Using `OBS_END` would exclude patients who died within 5 years of their index date — the very outcome we are trying to study. This is **survivorship bias**. Using `DATASET_END_DATE` instead asks the correct question: *did the dataset run long enough after this patient's index date to potentially observe a death?*

Time-to-event for the Cox regression (computed in the analysis notebook, not here) is:

```
duration = min(DEATH_DATE, index_date + 5 years) - index_date
event    = 1 if died within 5 years, 0 if censored at 5 years
```

Since `DATASET_END_DATE` is computed automatically from the data, this cell works correctly for any Synthea dataset (Coherent, Cheng, OMOP) without manual adjustment.

**Part C — Disease configuration:**  
The only block that changes between diseases. Defines:

| Variable | Description |
|---|---|
| `DISEASE_NAME` | Used in output filenames and print statements |
| `CONDITION_SEARCH_KEYWORDS` | Keywords to find condition codes in the catalog |
| `TREATMENT_SEARCH_KEYWORDS` | Keywords to find treatment codes in the catalog |
| `EXCLUDE_KEYWORDS` | Terms to remove from condition results (e.g. related but distinct conditions) |
| `LOOKBACK_YEARS` | Minimum years of data required before index date (default: 5) |
| `FOLLOWUP_YEARS` | Minimum years of dataset coverage required after index date (default: 5) |
| `RANDOM_SEED` | Seed for reproducible random date assignment (default: 42) |

**When to re-run:** Every time you open the notebook, and whenever you change disease. Always re-run Cell 3 after changing any keyword list.

---

### Cell 3 — Catalog Extractor

**What it does:**  
Reads the unique descriptions catalog (`unique_descriptions_coherent_dataset.csv`) and automatically builds the exact lists of condition descriptions and treatment descriptions used for patient identification in later cells.

**Why a catalog?**  
Rather than hardcoding codes manually or using broad keyword matching against millions of rows, we use a pre-built catalog of every unique description string in the dataset. This catalog was generated separately and has one row per unique description, with the source file(s) where it appears. Matching against this catalog first is efficient, transparent, and fully auditable.

**Catalog format:**

```
Description                                          Source_Files
Diabetes (code: 44054006)                            conditions
Encounter for problem (code: 185347001,              encounters
  REASONDESCRIPTION: Neuropathy due to type 2 DM)
24 HR Metformin hydrochloride 500 MG (code: 860975,  medications
  REASONDESCRIPTION: Diabetes)
```

**Part A — Catalog parsing:**  
Every catalog entry is parsed into three structured fields:

| Field | Example |
|---|---|
| `CLEAN_DESC` | `Diabetes` |
| `CODE` | `44054006` |
| `REASON_DESC` | `Neuropathy due to type 2 diabetes mellitus` |

This allows searching against both the primary description and the reason description — important because a condition often appears in encounters, procedures, and careplans as a `REASONDESCRIPTION` rather than a primary `DESCRIPTION`.

**Part B — Source file configuration:**  
Defines how to match descriptions against each source CSV and which columns to use:

| Source file | Date column | Match columns |
|---|---|---|
| `conditions.csv` | START | DESCRIPTION |
| `medications.csv` | START | DESCRIPTION |
| `encounters.csv` | START | DESCRIPTION, REASONDESCRIPTION |
| `observations.csv` | DATE | DESCRIPTION |
| `procedures.csv` | DATE | DESCRIPTION, REASONDESCRIPTION |
| `careplans.csv` | START | DESCRIPTION, REASONDESCRIPTION |

**Part C — Condition extraction:**  
Searches `CLEAN_DESC` and `REASON_DESC` against `CONDITION_SEARCH_KEYWORDS` (from Cell 2), then removes any matches containing `EXCLUDE_KEYWORDS`. Results are grouped by source file and printed for review, showing description, code, and reason description for each match.

**Part D — Treatment extraction:**  
Same process but restricted to medication descriptions only, searching against `TREATMENT_SEARCH_KEYWORDS`.

**Output variables for downstream cells:**

| Variable | Content |
|---|---|
| `CONDITION_DESCRIPTIONS` | List of exact description strings for `.isin()` matching |
| `CONDITION_CODE_LIST` | List of SNOMED/other codes |
| `CONDITION_SOURCE_MAP` | Dict mapping source file → descriptions, codes, and CSV config |
| `TREATMENT_DESCRIPTIONS` | List of exact medication description strings for `.isin()` matching |
| `TREATMENT_CODE_LIST` | List of RxNorm/other codes |

**Review step:**  
The cell ends with a checklist that prints counts and warns if nothing was found, prompting you to adjust keywords in Cell 2 before continuing. You should always read the full output of this cell carefully before running Cell 4 — it is the foundation for all patient identification downstream.

**When to re-run:** Any time you change keywords in Cell 2.

---

## Output files

| File | Description |
|---|---|
| `{DISEASE}_observation_periods.csv` | All patients with OBS_START, OBS_END, death status |
| `{DISEASE}_index_date_eligibility.csv` | All disease patients with index dates and eligibility flags |
| `{DISEASE}_fully_eligible_cohort.csv` | Final eligible cohort — input to MOSAIC pipeline and analysis notebook |

---

## Study design reference

For full pharmacoepidemiology study design documentation including all definitions (index date, lookback window, follow-up period, censoring, hazard ratio) and the SNOMED-CT / RxNorm codes used for T2D, see [`STUDY_DESIGN.md`](../STUDY_DESIGN.md).

---

## Reference

Cooper, J. et al. (2025). Defining and validating severity phenotypes for long-term conditions to support risk stratification in primary care. *Journal of Biomedical Informatics*, 166, 104831.

---

### Cell 4 — Observation Periods & Death Reconciliation

**What it does:**
Scans all clinical CSV files to establish the true observation period for every patient in the dataset, then reconciles death status from three independent sources. This cell operates on **all patients** — no disease filtering yet.

**Part A — Demographics:**
Loads `patients.csv` and standardises the patient ID column name (handles both `Id` and `ID` variants across Synthea datasets). Parses `BIRTHDATE` and `DEATHDATE`.

**Part B — Observation period computation:**
Iterates all 11 clinical CSV files and collects every valid date per patient across all date columns:

| File | Date columns scanned |
|---|---|
| `conditions.csv` | START, STOP |
| `observations.csv` | DATE |
| `medications.csv` | START, STOP |
| `encounters.csv` | START, STOP |
| `procedures.csv` | DATE |
| `careplans.csv` | START, STOP |
| `immunizations.csv` | DATE |
| `imaging_studies.csv` | DATE |
| `devices.csv` | START, STOP |
| `allergies.csv` | START, STOP |
| `supplies.csv` | DATE |

For each patient: `OBS_START` = minimum date across all files and columns; `OBS_END` = maximum date. `OBS_DURATION_YEARS` is computed as `(OBS_END - OBS_START).days / 365.25`.

Note: `patients.csv` birth and death dates are intentionally excluded from this scan — `OBS_START` and `OBS_END` reflect clinical activity, not demographic dates.

**Part C — Death reconciliation:**
Death status is ascertained from three independent sources and merged in priority order:

| Priority | Source | Code | Field used |
|---|---|---|---|
| 1 (highest) | `encounters.csv` | SNOMED `308646001` (Death Certification) | `REASONDESCRIPTION` → cause of death |
| 2 | `observations.csv` | LOINC `69453-9` (Cause of Death US Certificate) | `VALUE` → cause of death |
| 3 (fallback) | `patients.csv` | `DEATHDATE` column | date only, no cause |

The encounter source is preferred because it is the richest — it contains the cause of death in `REASONDESCRIPTION`. A cross-source overlap summary is printed to flag any discrepancies between sources.

**Final death fields:**

| Field | Description |
|---|---|
| `DIED` | Boolean — True if any source records death |
| `FINAL_DEATH_DATE` | Reconciled death date (enc > obs > patients.csv) |
| `FINAL_CAUSE_OF_DEATH` | Cause of death text (enc > obs) |

**Output variable:** `all_patients_df` — master patient table with observation periods and death info for all patients in the dataset.

**When to re-run:** Once per session. No changes needed between diseases.

---

### Cell 5 — Disease Patient Identification & First Treatment

**What it does:**
Identifies which patients have the target disease and finds their first diagnosis date and first treatment date. Uses a deliberate two-step approach to avoid inflating the patient count.

**Why two steps?**
A naive approach — searching for disease mentions across all files including encounters and procedures — massively inflates the patient count. In the T2D case this caused a jump from 382 to 2,253 patients, because `REASONDESCRIPTION` in encounters captures every visit *related to* a T2D complication, not just T2D diagnoses. The two-step approach separates WHO has the disease from WHEN their earliest evidence was recorded.

**Step 1 — WHO has the disease? (`conditions.csv` only)**
Patient identification uses `conditions.csv` exclusively. This is the canonical diagnostic source in Synthea. Matching uses both `DESCRIPTION` (via `CONDITION_DESCRIPTIONS`) and `CODE` (via `CONDITION_CODE_LIST`) from Cell 3. Exclusions from `EXCLUDE_KEYWORDS` are applied after matching.

> Example for T2D: matches `DESCRIPTION == "Diabetes"` OR `CODE == "44054006"`, then excludes any row containing `"cystic fibrosis"`.

The result is `confirmed_patients` — a set of patient UUIDs who definitively have the disease.

**Step 2 — WHEN was their first evidence? (all source files)**
For patients already in `confirmed_patients`, searches across all files in `CONDITION_SOURCE_MAP` for the earliest date of any disease-related record. The critical guard is:

```python
df = df[df[pat_col].isin(confirmed_patients)]  # filter FIRST
```

This means encounters and procedures can only contribute earlier dates for already-confirmed patients — they can never add new patients. `FIRST_DX_DATE` = minimum date across all sources. `FIRST_DX_SOURCE` records which file provided the earliest date for auditing.

**First treatment:**
Loads `medications.csv`, filters to `confirmed_patients`, and matches against `TREATMENT_DESCRIPTIONS` and `TREATMENT_CODE_LIST` from Cell 3. First treatment per patient = earliest dispensing date. Drug name stored in `FIRST_TREATMENT_DRUG`.

**Merge:**
Uses an `inner` join with `all_patients_df` from Cell 4 — automatically keeps only disease patients and drops the rest of the dataset.

**Output variable:** `disease_patients_df` — disease patients only, with observation periods, death info, first diagnosis date, and first treatment date.

**When to re-run:** Any time Cell 3 is re-run (keywords changed). No other changes needed between diseases.

---

### Cell 6 — Index Date Assignment & Eligibility

**What it does:**
Computes four index dates per patient and evaluates eligibility for each. This is the cell where the core pharmacoepidemiology study design is operationalised.

**Key design decision — using `DATASET_END_DATE` for follow-up:**
Follow-up eligibility is checked as:

```
DATASET_END_DATE - index_date >= FOLLOWUP_YEARS
```

NOT as `OBS_END - index_date >= FOLLOWUP_YEARS`. See Cell 2 for the full explanation of why this matters (survivorship bias fix). Time-to-event for Cox regression is computed in the analysis notebook as `min(DEATH_DATE, index_date + 5 years) - index_date`.

**Index Date 1 — First Diagnosis:**
Index date = `FIRST_DX_DATE` from Cell 5. Eligible if:
- Lookback ≥ 5 years before index date
- Dataset extends ≥ 5 years after index date
- Patient is alive on the index date

**Index Date 2 — First Treatment:**
Index date = `FIRST_TREATMENT_DATE` from Cell 5. Same eligibility rules. Patients without any recorded treatment are automatically ineligible.

**Index Date 3 — Random Date:**
Index date = randomly sampled from the window `[FIRST_DX_DATE, min(FINAL_DEATH_DATE, DATASET_END_DATE) - 5 years]`.

The window is capped at `FINAL_DEATH_DATE` so the random index date never falls after the patient has already died. For alive patients the upper bound is simply `DATASET_END_DATE - 5 years`. If the window is zero or negative the patient is ineligible. Reproducible via `numpy.random.default_rng(seed=42)`.

**Index Date 4 — Latest (cross-sectional):**
Index date = `OBS_END` (last recorded clinical event). No follow-up requirement — this is a cross-sectional analysis only. Requires minimum lookback of 5 years.

**Output variable:** `eligibility_df` — all disease patients with index dates, lookback/followup years, and eligibility flags for each analysis.

**When to re-run:** Any time Cell 5 is re-run.

---

### Cell 6b — Eligibility Audit

**What it does:**
Diagnostic cell that explains why each ineligible patient fails eligibility. Run after Cell 6, before Cell 7. Does not modify any variables — read-only audit.

**Ineligibility reasons per index date:**

| Reason | Description |
|---|---|
| A | No index date recorded (e.g. no treatment in dataset) |
| B | Insufficient lookback — < 5 years of data before index date |
| C | Insufficient dataset coverage — < 5 years after index date |
| D | Index date falls after death date — data quality flag |

**Reason C is further split:**
- **C1 — deceased patients**: died within the potential follow-up window. In the T2D Coherent cohort only 1 patient falls into C1 — they are correctly excluded because their index date is genuinely too close to `DATASET_END_DATE` (not because they died early).
- **C2 — alive patients**: index date too recent for the dataset to provide 5 years of coverage.

This distinction matters for your supervisor conversation: C1 patients are not excluded because they died, they are excluded because the dataset does not extend far enough after their index date. The survivorship bias fix in Cell 2 (using `DATASET_END_DATE`) already handles the general case correctly.

**When to run:** Once after Cell 6, before running Cell 7. Especially important when switching to a new disease or dataset.

---

### Cell 7 — Final Eligible Cohort & Save

**What it does:**
Filters `eligibility_df` to patients eligible for all four index date analyses simultaneously, prints a full cohort summary, builds an exclusion log, and saves three output files.

**Filtering logic:**

```python
fully_eligible = eligibility_df[
    eligibility_df["ELIGIBLE_FIRST_DX"] &
    eligibility_df["ELIGIBLE_FIRST_TX"] &
    eligibility_df["ELIGIBLE_RANDOM"]   &
    eligibility_df["ELIGIBLE_LATEST"]
]
```

Using a single consistent cohort across all four analyses ensures any differences in results between index dates are attributable to the index date definition itself, not to differences in the underlying patient population.

**Exclusion log:**
For every excluded patient, a human-readable `EXCLUSION_REASON` string is generated explaining exactly which eligibility criterion failed and by how much. Example:

```
First Dx: lookback 3.2 < 5 yrs | No first treatment date
```

**Cohort summary printed:**
Sex, race/ethnicity, age at diagnosis, observation period, index date ranges, mortality breakdown, cause of death, first treatment drug breakdown, follow-up available per index date.

**Output files:**

| File | Contents |
|---|---|
| `{DISEASE}_all_patients.csv` | All disease patients with eligibility flags — full picture |
| `{DISEASE}_eligible_cohort.csv` | Fully eligible cohort — input to MOSAIC pipeline and analysis notebook |
| `{DISEASE}_exclusion_log.csv` | One row per excluded patient with `EXCLUSION_REASON` |

**When to re-run:** Any time Cell 6 is re-run.
