
# Ground Truths — Clinical Benchmark Generation

This folder contains the notebook used to generate the **algorithmic ground-truth severity labels** used in the MOSAIC evaluation.

The ground-truth labels are used to compare the MOSAIC agentic LLM classifications against two independent clinical benchmark systems:

1. **Young GT / DCSI** — based on the Diabetes Complications Severity Index.
2. **Cooper GT** — based on the 7-dimensional feasibility framework adapted from Cooper et al. (2025).

These benchmarks are used in **Phase 3 — Evaluation and Statistical Analysis** to assess whether MOSAIC produces severity classifications that are clinically meaningful, reproducible, and aligned with validated clinical definitions.

---

## Folder contents

```text
Ground_Truths/
│
├── Ground_Truth_FINAL_v03_05_2026.ipynb
└── README.md
```

---

## Purpose of this notebook

The notebook generates two sets of algorithmic ground-truth labels for each patient:

- `young_tier` — severity tier derived from the Diabetes Complications Severity Index.
- `cooper_tier` — severity tier derived from the Cooper et al. severity phenotype framework.

Both ground truths convert structured EHR evidence into the same four-tier severity scale used by MOSAIC:

```text
Baseline T2D
Mild Complications
Moderate Complications
Advanced/Critical
```

This allows the LLM-generated MOSAIC classifications to be compared against rule-based clinical benchmarks using agreement statistics, survival analysis, and qualitative disagreement analysis.

---

## How this notebook should be run

This notebook is designed to be run together with the main **Phase 3 full analysis notebook**.

The ground-truth notebook assumes that the following objects have already been created by the upstream preprocessing and cohort-building code:

```text
cohort
con_all
obs_all
med_all
```

These objects are expected to contain the cohort definition and the structured EHR tables needed for ground-truth scoring.

### Recommended use

The recommended use is to run this notebook after the relevant preprocessing cells in the full Phase 3 analysis notebook have already been executed.

In the original MOSAIC workflow, this notebook corresponds to the ground-truth scoring section of the full analysis pipeline.

### Alternative use with another dataset

This notebook can also be adapted to run on a different dataset.

However, if using another dataset, the user must manually adjust the relevant variables, column names, concept codes, and time-window definitions as needed.

At minimum, the following elements may need to be modified:

```text
PATIENT identifier column
Condition code column
Observation code column
Medication code column
Date columns
Laboratory value column
T2D diagnosis/treatment/index-date columns
Covariate window start and end columns
SNOMED concept codes
LOINC laboratory codes
Medication concept codes
```

The notebook was written for the MOSAIC Synthea-based structured EHR dataset, so direct reuse on another dataset may require dataset-specific mapping.

---

## Required input objects

The notebook assumes the following input objects are already available in memory.

| Object | Description |
|---|---|
| `cohort` | Patient-level cohort dataframe containing index dates, eligibility flags, and covariate window boundaries |
| `con_all` | Conditions table containing patient-level diagnosis records |
| `obs_all` | Observations table containing laboratory and clinical measurements |
| `med_all` | Medications table containing medication exposure records |

The notebook also assumes that relevant date columns have already been parsed into datetime format and that the cohort contains patient-specific covariate assessment windows.

---

## Time windows scored

The notebook scores ground-truth severity across four clinically meaningful time windows.

| Prefix | Time window | Interpretation |
|---|---|---|
| `td` | At diagnosis | Severity at initial Type 2 Diabetes diagnosis |
| `t0` | At first treatment | Severity at first pharmacological treatment initiation |
| `t5` | 5 years post-treatment | Primary reclassification landmark used for validation |
| `t10` | 10 years post-treatment | Long-term disease trajectory assessment |

In the original MOSAIC full cohort, the notebook was configured to score:

```text
td  — At diagnosis: all eligible patients
t0  — At first treatment: all eligible patients
t5  — 5-year reclassification: patients eligible at 5 years
t10 — 10-year reclassification: patients eligible at 10 years
```

The `t5` window is the primary landmark used for the main MOSAIC evaluation.

---

## Notebook structure

The notebook contains two main sections.

### Cell 5 — Define ground-truth scoring functions

This cell defines the reusable functions and concept dictionaries used to compute both ground truths.

It defines:

```python
slice_window()
compute_dcsi()
extract_cooper_dimensions()
```

### Cell 6 — Score all four time windows

This cell applies the scoring functions to all four predefined time windows:

```text
td
t0
t5
t10
```

For each time window, the notebook:

1. Slices conditions, observations, and medications to the relevant covariate window.
2. Computes the Young / DCSI ground truth.
3. Computes the Cooper ground truth.
4. Merges all resulting ground-truth labels and evidence columns back into the `cohort` dataframe.

---

# Gold Standard Definitions

To evaluate the performance and validity of the MOSAIC agentic orchestrations, two distinct validated clinical frameworks were used as gold standards.

In this context, a **gold-standard label** refers to the most reliable available classification of a patient’s phenotype status, typically based on expert-defined criteria or detailed clinical review.

In this project, the two frameworks are used as **algorithmic gold standards** because they translate structured Electronic Health Record data into clinically meaningful severity categories. This enables systematic comparison between MOSAIC model-generated classifications and established clinical definitions.

---

## 1. Young GT — Diabetes Complications Severity Index

The first ground truth is based on the **Diabetes Complications Severity Index (DCSI)** developed by Young et al.

Young et al. developed and validated the DCSI as a structured scoring system for quantifying cumulative diabetic complication burden using automated clinical data. In a prospective cohort study of 4,229 primary care patients followed over four years, the authors demonstrated that the DCSI — derived from ICD-9 diagnostic codes, pharmacy records, and laboratory results — was a stronger predictor of mortality and hospitalisation than a simple count of complications.

The DCSI evaluates seven physiological domains:

1. Cardiovascular disease
2. Nephropathy
3. Retinopathy
4. Peripheral vascular disease
5. Stroke / cerebrovascular disease
6. Neuropathy
7. Metabolic complications

Each domain is scored as:

```text
0 = no abnormality
1 = mild/moderate abnormality
2 = severe abnormality
```

Neuropathy is scored as a binary domain:

```text
0 = absent
1 = present
```

The total DCSI score has a maximum possible value of 13.

Notably, a DCSI score of 1 alone did not significantly increase mortality risk in the original study. Risk escalated meaningfully from a score of 3 upwards, with each additional point associated with a 1.34-fold increase in mortality risk.

---

## Young GT implementation in this notebook

The notebook implements the Young GT using structured diagnosis and laboratory evidence available in the MOSAIC dataset.

The function used is:

```python
compute_dcsi()
```

This function scores each patient across the seven DCSI domains using diagnosis codes and selected laboratory values.

### DCSI domains implemented

| Domain | Evidence used in notebook |
|---|---|
| Retinopathy | SNOMED condition codes for background retinopathy, non-proliferative diabetic retinopathy, proliferative retinopathy, and macular oedema |
| Nephropathy | SNOMED condition codes, serum creatinine, and UACR |
| Neuropathy | SNOMED condition codes for diabetic polyneuropathy and peripheral neuropathy |
| Cerebrovascular disease | SNOMED condition codes for stroke and transient ischaemic attack |
| Cardiovascular disease | SNOMED condition codes for myocardial infarction, heart attack, and angina |
| Peripheral vascular disease | SNOMED condition codes for peripheral arterial disease |
| Metabolic complications | SNOMED condition codes for diabetic ketoacidosis and hypoglycaemia |

### Laboratory values used

| Laboratory marker | Code used | Interpretation |
|---|---|---|
| Serum creatinine | `2160-0` | Used as part of nephropathy severity scoring |
| UACR / microalbumin-creatinine ratio | `14959-1` | Used as part of nephropathy severity scoring |
| HbA1c | `4548-4` | Extracted for evidence reporting |

---

## Young GT tier mapping

For comparability with the four-tier MOSAIC framework, continuous DCSI scores were mapped into ordinal severity tiers.

| DCSI score | MOSAIC-compatible tier |
|---|---|
| `0` | Baseline T2D |
| `1` | Mild Complications |
| `2–3` | Moderate Complications |
| `≥4` | Advanced/Critical |

This binning is an adaptation made for this study and does not correspond to severity tiers defined in the original Young et al. publication.

---

## 2. Cooper GT — 7-Dimensional Feasibility Framework

The second ground truth is based on the severity phenotype framework developed by Cooper et al.

Cooper et al. used a mixed-methods approach combining systematic literature review, primary care database analysis, and expert consensus via the **Nominal Group Technique (NGT)** to identify severity phenotypes for nine long-term conditions in primary care EHR.

Eighteen clinical academics rated candidate phenotypes on two dimensions:

1. **Clinical importance** — does this meaningfully distinguish mild from severe disease?
2. **Feasibility** — is it reliably recorded in routine EHR?

Each dimension was rated on a 1–5 Likert scale.

The combined clinical importance and feasibility score was interpreted as:

```text
≥8/10 = green / recommended
6–8   = amber / potential
<6    = red / not recommended
```

The candidate phenotypes were further validated by testing whether patients classified as severe had significantly higher five-year mortality in more than nine million CPRD Aurum records.

For Type 2 Diabetes, five green-rated phenotypes were identified:

| Phenotype | Score |
|---|---|
| Microvascular complications | 8.75/10 |
| Proteinuria | 8.33/10 |
| Retinopathy staging | 8.25/10 |
| Diabetes medications | 8.17/10 |
| Diabetic foot ulcer risk score | 8.00/10 |

These phenotypes do not individually define severity tiers in the original publication. Instead, they identify the most clinically important and reliably recorded severity markers for Type 2 Diabetes.

---

## Cooper GT implementation in this notebook

For this study, the Cooper et al. green-rated markers were operationalised into a four-tier algorithmic ground truth across seven dimensions.

The function used is:

```python
extract_cooper_dimensions()
```

This function extracts clinical dimensions from conditions, observations, and medications, and then assigns each patient to one of the four MOSAIC-compatible severity tiers.

### Cooper dimensions implemented

| Dimension | Evidence used in notebook |
|---|---|
| Microvascular complication count | Nephropathy/CKD, retinopathy, neuropathy, foot ulcer/amputation |
| Macrovascular disease | Stroke, coronary disease/MI, heart failure, peripheral vascular disease |
| UACR category | Normal, microalbuminuria, macroalbuminuria |
| Retinopathy staging | None, background, NPDR, proliferative/oedema |
| Medication intensity | Number of glucose-lowering drug classes |
| Insulin use | Insulin medication exposure |
| Renal function | Minimum eGFR |
| Glycaemic control | Maximum HbA1c |

Although HbA1c was not one of the green-rated Cooper markers, it was included in this implementation with acknowledged caveats because it is clinically relevant and available in the structured EHR data. This should be interpreted as a study-specific adaptation.

---

## Cooper GT tier mapping

The Cooper GT assigns patients to four severity tiers using the following operational rules.

| Tier | Operational definition |
|---|---|
| Baseline T2D | No complications, no macrovascular or microvascular flags, and fewer than 2 glucose-lowering medication classes |
| Mild Complications | HbA1c >8.0% or more than 1 glucose-lowering drug class |
| Moderate Complications | Any macrovascular or microvascular flag not meeting critical thresholds |
| Advanced/Critical | eGFR <30, proliferative retinopathy, foot ulcer/amputation, or high complication burden |

In the notebook implementation, a patient is assigned to **Advanced/Critical** if any of the following are present:

```text
complication count ≥3
foot ulcer or amputation
proliferative retinopathy / macular oedema
eGFR <30
```

A patient is assigned to **Moderate Complications** if they have macrovascular or microvascular disease that does not meet Advanced/Critical criteria.

A patient is assigned to **Mild Complications** if they have elevated HbA1c or medication intensification but no documented complication flags.

A patient is assigned to **Baseline T2D** if no severity signals are identified.

This four-tier operationalisation is this study’s adaptation of Cooper et al. and should be distinguished from the original publication, which provides ranked phenotype recommendations rather than direct tier definitions.

---

## Output columns

After scoring, the notebook produces Young GT and Cooper GT outputs for each time window.

For example, for the `t5` window, output columns include:

### Young GT columns

```text
t5_ret
t5_neph
t5_neu
t5_cbv
t5_cvd
t5_pvd
t5_met
t5_dcsi
t5_creat
t5_uacr
t5_hba1c
t5_young_tier
t5_young_evidence
```

### Cooper GT columns

```text
t5_comp_count
t5_uacr_cat
t5_ret_stage
t5_n_meds
t5_insulin
t5_has_foot
t5_egfr_min
t5_hba1c_max
t5_macro_flag
t5_micro_flag
t5_cooper_tier
t5_cooper_evidence
```

Equivalent columns are generated for:

```text
td
t0
t5
t10
```

---

## How outputs are used in Phase 3

The generated ground-truth labels are merged back into the main `cohort` dataframe and used in the Phase 3 evaluation.

They support:

- classification agreement analysis
- confusion matrices
- weighted Cohen’s kappa
- tier distribution comparison
- survival analysis by severity tier
- comparison of MOSAIC against Young GT and Cooper GT
- qualitative review of MOSAIC disagreements with algorithmic ground truths

The main comparison labels used downstream are typically:

```text
t5_young_tier
t5_cooper_tier
```

because the `t5` landmark is the primary evaluation point in the MOSAIC study design.

---

## Important methodological notes

### Algorithmic ground truths

Both Young GT and Cooper GT are algorithmic adaptations of published clinical frameworks.

They are not manual expert adjudications of each individual patient record.

This distinction is important because disagreement between MOSAIC and a ground truth does not automatically imply that MOSAIC is wrong. Instead, disagreement may reflect differences in how each system weights available clinical evidence.

### Study-specific tier adaptation

The four-tier severity labels used in this repository were created to enable comparison with the MOSAIC severity framework.

The original Young et al. DCSI publication provides a continuous complication severity score.

The original Cooper et al. publication provides ranked phenotype recommendations based on clinical importance and EHR feasibility.

Therefore, the tier mappings used here should be interpreted as study-specific operationalisations.

### Structured EHR limitation

The ground-truth algorithms rely on structured EHR fields such as diagnosis codes, laboratory values, and medications.

They may miss severity signals that are only visible in narrative text, clinical context, or longitudinal reasoning patterns.

This limitation is one of the motivations for evaluating MOSAIC, which is designed to reason across broader patient-level evidence.

---

## References

Cooper, J., Jackson, T., Haroon, S., Crowe, F. L., Hathaway, E., Fitzsimmons, L., & Nirantharakumar, K. (2025). Defining phenotypes of disease severity for long-term cardiovascular, renal, metabolic, and mental health conditions in primary care electronic health records: A mixed-methods study using the nominal group technique. *Journal of Biomedical Informatics, 166*, 104831. https://doi.org/10.1016/J.JBI.2025.104831

Young, B. A., Lin, E., von Korff, M., Simon, G., Ciechanowski, P., Ludman, E. J., Everson-Stewart, S., Kinder, L., Oliver, M., Boyko, E. J., & Katon, W. J. (2008). Diabetes Complications Severity Index and Risk of Mortality, Hospitalization, and Healthcare Utilization. *The American Journal of Managed Care, 14*(1), 15. https://pmc.ncbi.nlm.nih.gov/articles/PMC3810070/
