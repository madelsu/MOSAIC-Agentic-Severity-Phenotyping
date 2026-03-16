# MOSAIC — Pharmacoepidemiology Study Design

> **MOSAIC** (Multi-LLM Orchestrated Severity Assessment In Clinical Records)  
> MSc Bioinformatics · University of Copenhagen · 2026  
> Principal Investigator: Manu | Supervisors: Maurizio Sessa (clinical), Thomas Hamelryck (internal)

---

## Overview

MOSAIC is a multi-agent LLM pipeline that classifies patient disease severity from electronic health records (EHR). The pharmacoepidemiology study described here serves as the **validation framework** for MOSAIC: we assess whether LLM-assigned severity phenotypes meaningfully predict five-year mortality, using Cox proportional hazards regression as the primary analytical tool — mirroring the methodology of Cooper et al. (2025, JBI 166:104831).

The study is designed around **Type 2 Diabetes Mellitus (T2D)** as the index condition, using the Synthea Coherent dataset (N=3,539 total patients; N=242 fully eligible T2D patients after strict eligibility criteria). The design is intentionally **generalizable**: the same pipeline will subsequently be applied to Hypertension, Ischaemic Heart Disease, Heart Failure, and Chronic Kidney Disease.

---

## Pharmacoepidemiology: Key Definitions

This section defines all core pharmacoepidemiological concepts used in this study. These definitions are standard in the field and apply consistently across all disease analyses within MOSAIC.

### Study Period
The **study period** is the total calendar time span of the dataset from which patients can be drawn. For the Coherent dataset, this spans from the earliest recorded clinical event (~1913) to the latest recorded date (~2021). The study period defines the outer boundaries within which all patient observation windows must fall.

### Observation Period (Patient-Level)
The **observation period** is the patient-specific time window during which we have reliable clinical data. It is defined as:

- **Observation Start (`OBS_START`)**: the earliest date of any recorded clinical event across all clinical files (conditions, medications, encounters, observations, procedures, careplans, immunizations, imaging studies, devices, allergies, supplies)
- **Observation End (`OBS_END`)**: the latest date of any recorded clinical event across the same files

This is computed empirically by scanning all clinical CSV files rather than relying on any single source, to capture the true extent of available data per patient.

### Index Date
The **index date** is the anchor point in time at which a patient enters a specific analysis. All covariate assessment (exposure: severity) is measured in the window *before* the index date, and all outcome assessment (mortality) is measured in the window *after* it. The choice of index date is a fundamental study design decision that directly affects which information is used to classify severity and what follow-up time is available.

MOSAIC implements **four index date definitions** per patient, enabling comparison across analytical approaches:

| Index Date | Definition | Analysis Type |
|---|---|---|
| **First Diagnosis** | Date of first T2D-related condition code in the record | Survival (Cox regression) |
| **First Treatment** | Date of first antidiabetic medication dispensed | Survival (Cox regression) |
| **Random Date** | Randomly sampled date between first diagnosis and (OBS_END − 5 years) | Survival (Cox regression) |
| **Latest** | OBS_END — the most recent date of any clinical record | Cross-sectional (descriptive) |

The **Latest** index date corresponds to the approach used in the original MOSAIC pilot (100-patient analysis), where all available patient information was used without temporal constraints. It serves as a comparator to the three methodologically rigorous survival analyses.

### Covariate Assessment Window (Lookback Period)
The **covariate assessment window** (also called the lookback period) is the time interval *before* the index date used to assess patient characteristics and the primary exposure (severity). In this study, a minimum of **5 years of lookback** is required for a patient to be eligible for any survival analysis. This ensures that sufficient clinical history is available to meaningfully characterize severity.

For the Latest index date (cross-sectional), the entire observation period serves as the lookback window.

### Follow-up Period
The **follow-up period** is the time interval *after* the index date during which the outcome (mortality) is observed. This study uses a **five-year follow-up** window, consistent with Cooper et al. (2025). A patient is censored at five years if they have not died by the end of the follow-up window.

For a patient to be eligible for a survival analysis at a given index date, at least **5 years of follow-up** must be available after that index date (i.e., `OBS_END − index date ≥ 5 years`).

### Enrollment Period
The **enrollment period** is the range of index dates that satisfy both the lookback and follow-up eligibility criteria. It is patient-specific: each patient has a different enrollment window depending on when their index date falls relative to their observation start and end.

### Eligibility Criteria (Strict)
A patient is **eligible** for a given index date analysis if and only if:

```
(index_date − OBS_START) ≥ 5 years    [lookback requirement]
AND
(OBS_END − index_date) ≥ 5 years      [follow-up requirement]
```

For the **Latest** index date (cross-sectional only):
```
(OBS_END − OBS_START) ≥ 5 years       [minimum observation duration]
```

### Exposure: Severity Phenotype
The **exposure** in this study is the MOSAIC-assigned disease severity tier, assessed using clinical information recorded within the covariate assessment window (5 years before the index date). Severity is classified on a four-tier ordinal scale: **Mild / Moderate / Severe / Critical-Complex**, following the framework derived from Cooper et al. (2025).

For the Latest index date, severity is assessed using all available clinical information.

### Outcome: Five-Year Mortality
The **primary outcome** is all-cause mortality within five years of the index date. Death status is ascertained from three sources in the Coherent dataset (reconciled by priority):

1. `encounters.csv` — Death Certification encounter (SNOMED code 308646001), with cause of death from `REASONDESCRIPTION`
2. `observations.csv` — Cause of Death [US Standard Certificate of Death] (LOINC code 69453-9)
3. `patients.csv` — `DEATHDATE` column

### Censoring
Patients who do not experience the outcome (death) within the five-year follow-up window are **censored** at five years. Time-to-event is defined as:
- **If died within 5 years**: days from index date to death date
- **If alive at 5 years**: 5 × 365.25 days (censored)

### Hazard Ratio (HR)
A **hazard ratio** is the ratio of the hazard rate (instantaneous risk of death at any given time) between two groups. In this study, HRs compare each severity tier to the Mild reference category. An HR > 1 indicates higher mortality risk; HR < 1 indicates lower risk. Both crude (unadjusted) and adjusted HRs are reported.

### Cox Proportional Hazards Regression
**Cox proportional hazards regression** is a semi-parametric survival analysis model that estimates HRs while accounting for censoring and the time-varying nature of mortality risk. It assumes that the hazard ratio between groups is constant over time (proportional hazards assumption), which will be formally tested.

---

## Study Design

### Design Type
Retrospective cohort study using a synthetic EHR dataset (Synthea Coherent). Four parallel analyses using different index date definitions, three of which are survival analyses and one cross-sectional.

### Dataset
**Synthea Coherent Dataset**
- Total patients: 3,539
- Data format: CSV (clinical tables) + DNA files + DICOM imaging files + FHIR bundles
- Geographic scope: Massachusetts, USA
- Calendar period: ~1913–2021
- Condition coding: SNOMED-CT
- Medication coding: RxNorm
- Observation coding: LOINC

### Index Condition: Type 2 Diabetes Mellitus
T2D is identified using the following SNOMED-CT condition codes, all recorded in `conditions.csv`. The **earliest recorded date** across all seven descriptions is used as the first T2D diagnosis date:

| Description | SNOMED Code |
|---|---|
| Diabetes (T2D primary code) | 44054006 |
| Neuropathy due to type 2 diabetes mellitus | 368581000119106 |
| Nonproliferative diabetic retinopathy due to type 2 diabetes mellitus | 1551000119108 |
| Macular edema and retinopathy due to type 2 diabetes mellitus | 97331000119101 |
| Microalbuminuria due to type 2 diabetes mellitus | 90781000119102 |
| Proteinuria due to type 2 diabetes mellitus | 157141000119108 |
| Blindness due to type 2 diabetes mellitus | 60951000119105 |

### First Antidiabetic Treatment
First treatment date is defined as the earliest dispensing date of any of the following medications (exact description strings confirmed from the Coherent dataset unique descriptions catalog):

| Drug | Class |
|---|---|
| 24 HR Metformin hydrochloride 500 MG Extended Release Oral Tablet | Biguanide |
| Insulin Lispro 100 UNT/ML Injectable Solution [Humalog] | Insulin |
| insulin human isophane 70 UNT/ML / Regular Insulin Human 30 UNT/ML Injectable Suspension [Humulin] | Insulin |
| 3 ML liraglutide 6 MG/ML Pen Injector | GLP-1 agonist |
| canagliflozin 100 MG Oral Tablet | SGLT2 inhibitor |

### Random Date Assignment
For each patient, a random index date is sampled uniformly from the interval `[T2D_FIRST_DX_DATE, OBS_END − 5 years]` using NumPy's `default_rng` with seed 42, ensuring full reproducibility. Patients for whom this interval is zero or negative are ineligible for the random date analysis.

### Covariates for Adjusted Cox Models
| Covariate | Source | Notes |
|---|---|---|
| Age at index date | Derived from `BIRTHDATE` | Continuous, years |
| Sex | `patients.csv` | Binary (Male/Female) |
| Race/Ethnicity | `patients.csv` | Categorical |
| Insurance type at index date | `payer_transitions.csv` + `payers.csv` | Proxy for socioeconomic deprivation (Medicaid/Dual Eligible/Uninsured vs private) — *planned* |

> **Note on deprivation**: Cooper et al. (2025) adjust for IMD (Index of Multiple Deprivation), a UK-specific area-level deprivation measure. As the Coherent dataset is US-based synthetic data, we use insurance type as a US-appropriate socioeconomic proxy, following established US pharmacoepidemiology practice. This variable is currently planned and will be incorporated in the adjusted models.

### Statistical Analysis
For each of the three survival index dates:

1. **Binary Cox regression** — Severe/Critical-Complex vs Mild/Moderate
   - Crude HR (severity alone)
   - Adjusted HR (severity + age + sex + race/ethnicity)

2. **Four-tier Cox regression** — Mild (reference) vs Moderate, Severe, Critical-Complex
   - Crude HRs for each tier
   - Adjusted HRs for each tier
   - Dose-response assessment: monotonic increase in HR across tiers constitutes strong predictive validity evidence

3. **Proportional hazards assumption** tested via Schoenfeld residuals

4. **Stratified analyses** by age group (<50 / 50–65 / >65), sex, and race/ethnicity

For the Latest (cross-sectional) index date:
- Severity distribution summary
- Comparison of severity distributions across all four index dates

### Planned Extension to Additional Diseases
The identical pipeline will be applied to:
- **Hypertension**
- **Ischaemic Heart Disease (IHD)**
- **Heart Failure**
- **Chronic Kidney Disease (CKD)**

Each disease will use its own condition-specific SNOMED codes, treatment definitions, and severity framework (derived from Cooper et al. 2025 green-rated phenotypes). Eligibility criteria (5+5 year strict rule) and Cox regression approach remain identical across all conditions, enabling cross-disease comparison.

---

## Reference
Cooper, J. et al. (2025). Defining and validating severity phenotypes for long-term conditions to support risk stratification in primary care: A cross-sectional study. *Journal of Biomedical Informatics*, 166, 104831. https://doi.org/10.1016/j.jbi.2025.104831
