# MOSAIC — Phase 1: Cohort Construction

> **Disease**: Type 2 Diabetes Mellitus (T2D)  
> **Dataset**: Synthea Coherent  
> **Output**: 242 fully eligible T2D patients with observation periods, index dates, eligibility flags, and mortality status

---

## Overview

Phase 1 establishes the analytical cohort from the raw Synthea Coherent dataset. It proceeds in four sequential stages:

1. **Data Audit** — establish observation periods and death status for all patients
2. **T2D Identification** — identify T2D patients and their first diagnosis/treatment dates
3. **Index Date Assignment & Eligibility** — compute four index dates per patient and apply strict eligibility criteria
4. **Descriptive Analysis** — characterize the eligible cohort with summary statistics and figures

All scripts are designed for **Google Colab** with data stored on Google Drive. Scripts are modular and run sequentially, with each producing output CSV files that serve as inputs to the next stage.

---

## Repository Structure

```
MOSAIC/
├── phase1_cohort_construction/
│   ├── 01_observation_period_audit.py
│   ├── 02_t2d_patient_summary.py
│   ├── 03_t2d_index_date_eligibility.py
│   ├── 04_t2d_fully_eligible_cohort.py
│   ├── 05_check_dna_dicom_availability.py
│   ├── 06_t2d_cohort_descriptives.py
│   └── 07_t2d_cohort_figures.py
├── exploration/
│   ├── explore_antidiabetic_meds.py
│   ├── explore_t2d_first_mention.py
│   └── explore_admin_files.py
├── data/
│   └── [Coherent CSV files — not tracked in git]
└── STUDY_DESIGN.md
```

---

## Stage 1 — Data Audit

**Script**: `01_observation_period_audit.py`  
**Input**: All clinical CSV files from the Coherent dataset  
**Output**: `observation_period_audit.csv`, `t2d_observation_periods.csv`, `death_records_audit.csv`

### What it does

This script establishes the true observation period for every patient in the Coherent dataset by scanning all clinical CSV files and finding the earliest and latest recorded date per patient. It also reconciles death information from three independent sources.

### Files scanned for observation period

| File | Date Columns Used |
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

> **Implementation note**: All date columns are parsed with timezone stripping (`dt.tz_localize(None)`) to ensure tz-naive/tz-aware compatibility across files. The minimum date across all sources gives `OBS_START`; the maximum gives `OBS_END`.

### Death status reconciliation

Death information is sourced from three locations, reconciled in priority order:

| Priority | Source | Field | Code |
|---|---|---|---|
| 1 (highest) | `encounters.csv` | REASONDESCRIPTION | SNOMED 308646001 (Death Certification) |
| 2 | `observations.csv` | VALUE | LOINC 69453-9 (Cause of Death US Certificate) |
| 3 (fallback) | `patients.csv` | DEATHDATE | — |

The encounter source is preferred because it contains cause of death in the `REASONDESCRIPTION` field. The final reconciled fields are `FINAL_DEATH_DATE` and `FINAL_CAUSE_OF_DEATH`.

A cross-source agreement check is printed at runtime to flag any discrepancies between sources.

### Key outputs

| Column | Description |
|---|---|
| `PATIENT` | Patient UUID |
| `OBS_START` | Earliest clinical event date |
| `OBS_END` | Latest clinical event date |
| `OBS_DURATION_YEARS` | Observation duration in years |
| `FINAL_DEATH_DATE` | Reconciled death date (if applicable) |
| `FINAL_CAUSE_OF_DEATH` | Cause of death from encounter REASONDESCRIPTION |
| `DIED` | Boolean death flag |
| `IS_T2D` | Boolean T2D flag |
| `T2D_FIRST_DX_DATE` | Earliest T2D-related condition date |

---

## Stage 2 — T2D Patient Summary

**Script**: `02_t2d_patient_summary.py`  
**Input**: `t2d_observation_periods.csv`  
**Output**: `t2d_patient_summary.csv`

### What it does

Produces a clean one-row-per-patient summary table for all T2D patients, sorted by mortality status then diagnosis date. Adds age at diagnosis, age at death, and time from diagnosis to death.

### T2D identification

T2D patients are identified from `conditions.csv` using the following seven SNOMED-CT descriptions. The **earliest recorded date** across all seven is used as `T2D_FIRST_DX_DATE`:

```python
T2D_DESCRIPTIONS = [
    "Diabetes",
    "Blindness due to type 2 diabetes mellitus (disorder)",
    "Macular edema and retinopathy due to type 2 diabetes mellitus (disorder)",
    "Microalbuminuria due to type 2 diabetes mellitus (disorder)",
    "Neuropathy due to type 2 diabetes mellitus (disorder)",
    "Nonproliferative diabetic retinopathy due to type 2 diabetes mellitus (disorder)",
    "Proteinuria due to type 2 diabetes mellitus (disorder)",
]
```

> **Methodological rationale**: All T2D-related condition codes are included (not just the primary diagnosis code) because complications may be coded before the primary diagnosis in synthetic EHR data. This gives the most conservative (earliest) estimate of disease onset. This was verified empirically: conditions.csv is the earliest source for T2D across all files for all patients. Encounters and procedures only reference T2D in their REASONDESCRIPTION (as the reason for a visit/procedure), not as a primary diagnosis.

---

## Stage 3 — Index Date Assignment & Eligibility

**Script**: `03_t2d_index_date_eligibility.py`  
**Input**: `t2d_observation_periods.csv`, `conditions.csv`, `medications.csv`  
**Output**: `t2d_index_date_eligibility.csv`

### What it does

For each T2D patient, this script computes four index dates, calculates the available lookback and follow-up years for each, and applies strict eligibility criteria.

### Index date 1 — First Diagnosis

```
T2D_FIRST_DX_DATE = min(START) across all T2D condition codes per patient
```

**Eligible if**:
```
(T2D_FIRST_DX_DATE − OBS_START) ≥ 5 years   AND
(OBS_END − T2D_FIRST_DX_DATE) ≥ 5 years
```

### Index date 2 — First Treatment

First treatment is the earliest dispensing date of any antidiabetic medication. Drug descriptions are matched **exactly** against the Coherent dataset's unique descriptions catalog (confirmed via `explore_antidiabetic_meds.py`):

```python
ANTIDIABETIC_DESCRIPTIONS = [
    "24 HR Metformin hydrochloride 500 MG Extended Release Oral Tablet",
    "Insulin Lispro 100 UNT/ML Injectable Solution [Humalog]",
    "insulin human  isophane 70 UNT/ML / Regular Insulin  Human 30 UNT/ML Injectable Suspension [Humulin]",
    "3 ML liraglutide 6 MG/ML Pen Injector",
    "canagliflozin 100 MG Oral Tablet",
]
```

> **Design decision**: Exact string matching against the catalog (rather than keyword matching) ensures complete reproducibility and avoids false positives. Only 5 unique antidiabetic drugs exist in the Coherent dataset.

**Eligible if**:
```
FIRST_TREATMENT_DATE is not null   AND
(FIRST_TREATMENT_DATE − OBS_START) ≥ 5 years   AND
(OBS_END − FIRST_TREATMENT_DATE) ≥ 5 years
```

### Index date 3 — Random Date

```python
# Sampling window: T2D_FIRST_DX_DATE to (OBS_END − 5 years)
rng = np.random.default_rng(seed=42)
window_days = (OBS_END − timedelta(days=5*365.25)) − T2D_FIRST_DX_DATE
RANDOM_INDEX_DATE = T2D_FIRST_DX_DATE + timedelta(days=rng.integers(0, window_days))
```

**Eligible if**:
```
window_days > 0   AND
(RANDOM_INDEX_DATE − OBS_START) ≥ 5 years   AND
(OBS_END − RANDOM_INDEX_DATE) ≥ 5 years
```

> **Reproducibility**: NumPy's `default_rng(seed=42)` is used (not the legacy `np.random.seed`). The seed is stored in the output table. The random date is always within the patient's valid window by construction, so the follow-up requirement is guaranteed to be met.

### Index date 4 — Latest (cross-sectional)

```
LATEST_INDEX_DATE = OBS_END
```

**Eligible if**:
```
(OBS_END − OBS_START) ≥ 5 years
```

No follow-up requirement — this is a cross-sectional analysis.

### Output columns

| Column group | Columns |
|---|---|
| Identity | PATIENT_ID, GENDER, RACE, BIRTHDATE |
| Observation | OBS_START, OBS_END, OBS_DURATION_YEARS |
| First Diagnosis | T2D_FIRST_DX_DATE, AGE_AT_DX, LOOKBACK_YEARS_DX, FOLLOWUP_YEARS_DX, ELIGIBLE_FIRST_DX |
| First Treatment | FIRST_TREATMENT_DATE, FIRST_TREATMENT_DRUG, LOOKBACK_YEARS_TX, FOLLOWUP_YEARS_TX, ELIGIBLE_FIRST_TX |
| Random Date | RANDOM_INDEX_DATE, RANDOM_SEED, LOOKBACK_YEARS_RND, FOLLOWUP_YEARS_RND, ELIGIBLE_RANDOM |
| Latest | LATEST_INDEX_DATE, LOOKBACK_YEARS_LATEST, ELIGIBLE_LATEST |
| Outcome | DIED, DEATH_DATE, CAUSE_OF_DEATH, AGE_AT_DEATH |

---

## Stage 4 — Fully Eligible Cohort

**Script**: `04_t2d_fully_eligible_cohort.py`  
**Input**: `t2d_index_date_eligibility.csv`  
**Output**: `t2d_fully_eligible_cohort.csv`

### What it does

Filters to patients who are eligible for **all four** index date analyses simultaneously:

```python
fully_eligible = df[
    df["ELIGIBLE_FIRST_DX"] &
    df["ELIGIBLE_FIRST_TX"] &
    df["ELIGIBLE_RANDOM"]   &
    df["ELIGIBLE_LATEST"]
]
```

> **Rationale**: Using a single consistent cohort across all four analyses eliminates patient selection as a source of variation between index date comparisons. Any differences in severity distribution or mortality associations can be attributed to the index date definition itself, not to differences in the underlying patient population.

### Result

| Metric | Value |
|---|---|
| Total T2D patients in Coherent | ~382 |
| Fully eligible (all 4 index dates) | **242** |
| Exclusion rate | ~36.6% |

Primary reasons for exclusion: insufficient lookback before first diagnosis (patients diagnosed early in their observation window) or insufficient follow-up after the index date (patients diagnosed or treated close to their observation end).

---

## Stage 5 — Multimodal Data Availability

**Script**: `05_check_dna_dicom_availability.py`  
**Input**: `t2d_fully_eligible_cohort.csv`, DNA folder, DICOM folder  
**Output**: Updated `t2d_fully_eligible_cohort.csv` with `HAS_DNA`, `HAS_DICOM`, `HAS_BOTH`, `N_DICOM_FILES`

### What it does

The Coherent dataset uniquely provides linked genetic (DNA) and imaging (DICOM) data alongside clinical records. This script checks which of the 242 eligible T2D patients have these additional data modalities available.

### Filename parsing

Patient UUIDs are extracted from filenames using a regex pattern:

```python
UUID_PATTERN = re.compile(
    r'([0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12})',
    re.IGNORECASE
)
```

This reliably extracts the UUID regardless of patient name length or DICOM UID suffix, and operates on filenames in memory without opening any files.

- **DNA files**: `Firstname_Lastname_PATIENTID_dna.csv`
- **DICOM files**: `Firstname_Lastname_PATIENTID_DICOMUID.dcm`

---

## Stage 6 — Descriptive Statistics

**Script**: `06_t2d_cohort_descriptives.py`  
**Input**: `t2d_fully_eligible_cohort.csv`, `conditions.csv`  
**Output**: `t2d_cohort_descriptive.csv` (enriched with complication flags)

### What it does

Computes Table 1 equivalent descriptive statistics for the 242 eligible T2D patients:

- **Demographics**: sex, race/ethnicity, age at diagnosis, age groups, decade of diagnosis
- **Observation period**: duration statistics, follow-up and lookback by index date
- **T2D complications**: prevalence of each of the 6 T2D-related complication codes, number of complications per patient
- **Treatment**: first antidiabetic drug breakdown, time from diagnosis to treatment
- **Mortality**: alive vs deceased, cause of death breakdown, age at death, time from diagnosis to death
- **Data richness**: DNA and DICOM availability

---

## Stage 7 — Cohort Figures

**Script**: `07_t2d_cohort_figures.py`  
**Input**: `t2d_cohort_descriptive.csv`  
**Output**: 5 publication-quality figures saved to `/THESIS/Figures/`

| Figure | Content |
|---|---|
| `fig1_demographics.png` | Age distribution by sex, mortality by sex, race/ethnicity, diagnosis by decade |
| `fig2_complications.png` | Complication prevalence, complications per patient |
| `fig3_mortality_treatment.png` | Cause of death, age at death distribution, first treatment drug |
| `fig4_index_date_windows.png` | Follow-up and lookback boxplots across index dates |
| `fig5_data_richness.png` | DNA/DICOM availability donut and by mortality status |

All figures use the MOSAIC visual identity: teal-to-navy palette, white background, no grid, Helvetica typography.

---

## Exploration Scripts

Three additional scripts were used during the data discovery process and are retained for transparency:

### `explore_antidiabetic_meds.py`
Queries the unique descriptions catalog (`unique_descriptions_coherent_dataset.csv`) to identify all antidiabetic medication descriptions present in the Coherent dataset. Result: 5 unique antidiabetic drugs across 3 drug classes. Used to populate `ANTIDIABETIC_DESCRIPTIONS` in the eligibility script.

### `explore_t2d_first_mention.py`
Investigates whether T2D appears earlier in encounters, procedures, careplans, or observations than in `conditions.csv`. Result: T2D appears in encounters as a REASONDESCRIPTION (13,896 rows across 2,101 patients including non-T2D patients), but this reflects visits *for* T2D complications rather than diagnosis events. `conditions.csv` remains the authoritative source for T2D diagnosis dates.

### `explore_admin_files.py`
Inspects `payers.csv`, `payer_transitions.csv`, `providers.csv`, and `organizations.csv` to identify variables usable as a socioeconomic deprivation proxy in adjusted Cox models. Result: insurance type (Medicaid/Dual Eligible/NO_INSURANCE vs private insurers) from `payer_transitions.csv` will be used as a US-appropriate deprivation proxy. Implementation is planned for Phase 2.

---

## Reproducibility Notes

- All scripts tested in **Google Colab** (Python 3.12)
- Random date assignment uses `numpy.random.default_rng(seed=42)` — fully reproducible
- Exact drug description strings sourced from the dataset's own unique descriptions catalog — no keyword guessing
- All date parsing includes timezone stripping to handle mixed tz-naive/tz-aware timestamps in Synthea files
- Death reconciliation priority order is fixed and documented above

---

## Next Steps → Phase 2

Phase 2 will run the MOSAIC LLM pipeline on the 242 eligible patients using each of the four index dates as the temporal anchor. For each patient × index date combination, MOSAIC will:

1. Extract and compress clinical information from the 5-year lookback window
2. Run dual LLM assessors (Dr. A / GPT-4o and Dr. B / DeepSeek) to classify severity
3. Resolve disagreements via the Claude Sonnet consolidator
4. Output a severity tier (Mild / Moderate / Severe / Critical-Complex) per patient per index date

Phase 2 outputs feed directly into Phase 3 (Cox regression and validation against mortality outcomes).
