# Slide 1 — Title

# MOSAIC
## Multi-LLM Orchestrated Severity Assessment In Clinical Records

**Agentic Severity Phenotyping**

Master Thesis Student  
**Manuela Del Castillo**

External Supervisor  
**Maurizio Sessa**  
Associate Professor  
Department of Drug Design and Pharmacology  
University of Copenhagen

Internal Supervisor  
**Thomas Hamelryck**  
Professor in Machine Learning  
Department of Computer Science & Department of Biology  
University of Copenhagen

---

*MOSAIC — Because understanding severity requires more than one perspective.*

---

# Slide 2 — Agenda

1. Important context
2. Rationale behind the project
3. Dataset (Synthetic EHR — Synthea)
4. MOSAIC methodology
5. Phase 1 — Severity framework generation
6. Phase 2 — Patient severity assessment
7. Preliminary examples
8. Discussion & feedback

---

# IMPORTANT CONTEXT
# Slide 3 — Severity

Severity is a **multidimensional concept** reflecting the clinical intensity and impact of a disease.

Severity may include domains such as:

- Physiological measures
- Organ dysfunction
- Symptoms and functional impact
- Treatment intensity
- Healthcare utilization
- Mortality risk

*(Cooper et al., 2025)*

---

# IMPORTANT CONTEXT
# Slide 4 — Phenotyping

> "Phenotyping, the process of identifying patients with observable traits or characteristics of interest, involves more than just assigning a list of codes to a patient record. It requires extracting clinically relevant features from patient data to create cohorts with specific characteristics."

— **Neves et al., 2025**

Phenotyping therefore requires combining multiple signals from EHR data:

- Diagnoses
- Laboratory results
- Medications
- Procedures
- Clinical documentation

*(Shivade et al., 2014; Neves et al., 2025)*

---

# IMPORTANT CONTEXT
# Slide 5 — Approaches to EHR Phenotyping

### *A review of approaches to identifying patient phenotype cohorts using electronic health records*  
*(Shivade et al., 2014)*

| Approach | What it does | Strengths | Limitations |
|---|---|---|---|
| Rule-based | Apply logical constraints to EHR values | Transparent, interpretable | Rigid, difficult to scale |
| NLP | Extract signals from clinical text | Captures information in notes | Text messy and institution-specific |
| Machine Learning | Learns patterns from labeled data | Captures complex patterns | Often lacks interpretability |

---

# IMPORTANT CONTEXT
# Slide 6 — Why Severity Phenotyping?

### *Defining phenotypes of disease severity for long-term cardiovascular, renal, metabolic, and mental health conditions in primary care electronic health records*  
*(Cooper et al., 2025)*

Patients with the same diagnosis can experience **very different levels of disease severity**.

> Severe disease can have markedly different impacts on quality of life, treatment burden, and comorbidity progression.

Reliable methods are therefore needed to identify **stages of disease severity within EHR data**.

However:

- Severity indicators are **variably recorded**
- Multiple possible measures exist
- **No consensus currently exists on optimal severity phenotypes**

---

# IMPORTANT CONTEXT
# Slide 7 — Severity Phenotyping Framework

### Cooper et al., 2025

Optimal severity phenotypes were identified using:

- Literature review
- Feasibility analysis
- Expert consensus (Nominal Group Technique)
- Validation using CPRD Aurum EHR data

| Condition | Optimal Severity Phenotype | Severity Categories |
|---|---|---|
| Type 1 Diabetes | Microvascular complications | 0 → 1 → 2 → ≥3 complications |
| Type 2 Diabetes | Microvascular complications | 0 → 1 → 2 → ≥3 complications |
| Ischaemic Heart Disease | History of myocardial infarction | No MI / MI |
| Chronic Kidney Disease | eGFR categories | G3a → G3b → G4 → G5 |
| Peripheral Vascular Disease | Interventions or amputation | Claudication → Intervention → Amputation |
| Heart Failure | Loop diuretic prescriptions | None → Moderate → Severe |
| Hypertension | Antihypertensive classes | None → 1 → ≥4 |
| Aortic Aneurysm | Intervention or rupture | None → Intervention → Rupture |
| Depression | Secondary care involvement | Primary care → Referral → Hospitalisation |

---

# IMPORTANT CONTEXT
# Slide 8 — Type 2 Diabetes Severity Indicators

### Cooper et al., 2025

Severity indicators ranked by **clinical importance + feasibility**.

| Rank | Severity Indicator | EHR Feature | Combined Score |
|---|---|---|---|
| 1 | Microvascular complications | Clinical codes | 8.75 |
| 2 | Proteinuria | Lab measurement | 8.33 |
| 3 | Retinopathy | Clinical codes | 8.25 |
| 4 | Diabetes medications | Prescriptions | 8.17 |
| 5 | Diabetic foot ulcer risk score | Clinical codes | 8.00 |
| 6 | Microvascular complications OR proteinuria | Combined | 7.67 |
| 7 | HbA1c | Lab measurement | 7.67 |
| 8 | Medication + HbA1c | Combined | 7.42 |

Top ranked severity phenotype: **microvascular complications**

---

# IMPORTANT CONTEXT
# Slide 9 — Remaining Challenges

Despite advances in EHR phenotyping, several challenges remain.

### Limited generalizability
Models often rely on **small or non-representative cohorts**.

### Incomplete EHR data
Severity information is often **fragmented or missing**.

### Transparency vs performance trade-off

| Approach | Strength | Limitation |
|---|---|---|
| Rule-based | Interpretable | Hard to scale |
| ML | Flexible | Black-box reasoning |

### Lack of standardized severity phenotypes

Few validated and portable severity phenotyping algorithms exist.

---

# OUR PROPOSAL
# Slide 10 — MOSAIC

**MOSAIC**  
Multi-LLM Orchestrated Severity Assessment In Clinical Records

A **multi-agent AI pipeline** that infers disease severity phenotypes from EHR data.

Multiple LLM agents:

- Independently analyze patient records
- Identify clinical evidence
- Classify disease severity
- Consolidate their reasoning into a final decision

---

# OUR PROPOSAL
# Slide 11 — What MOSAIC Adds

### *Zero-shot learning for clinical phenotyping: Comparing LLMs and rule-based methods*  
*(Neves et al., 2025)*

| Traditional LLM phenotyping | MOSAIC |
|---|---|
Binary disease detection | Severity phenotyping |
Single model | Multi-agent reasoning |
Limited traceability | Evidence-based explanations |
Implicit knowledge | Explicit clinical framework |

---

# OUR PROPOSAL
# Slide 12 — MOSAIC Architecture

| Phase | What it does | Output |
|---|---|---|
| Phase 1 | Multiple LLMs derive severity criteria from literature | Frozen severity framework |
| Phase 2 | LLM agents assess each patient using the framework | Patient severity classification |
| Phase 3 | Agent predictions evaluated against sealed ground truth | Performance metrics |

---

# DATA
# Slide 13 — Datasets Used

| Dataset | Format | Patients | Used in |
|---|---|---|---|
| Synthea (Cheng) | Synthea CSV | ~32,000 | Workstream A |
| Coherent | Synthea CSV | ~8,000 | Workstream A + B |
| OMOP 1K | OMOP CDM | 1,130 | Workstream A |
| OMOP 100K | OMOP CDM | ~100,000 | Workstream A |

Current focus: **Coherent Synthea dataset (~8K patients)**.

---

# DATA
# Slide 14 — What is Synthetic Data?

Synthetic data is **artificially generated data** rather than data collected from real individuals.

| Type | Description | Privacy risk |
|---|---|---|
| Deidentified | Real data with identifiers removed | Possible |
| Anonymized | Identifiers removed | Small risk |
| Pseudonymized | IDs replaced | Re-identification possible |
| Synthetic | Artificial patients | No privacy risk |

Synthetic datasets contain **no real individuals** and can therefore be freely shared.

---

# DATA
# Slide 15 — What is Synthea?

**Synthea** is an open-source synthetic patient generator.

Key features:

- Simulates **full patient lifetimes**
- Generates realistic electronic health records
- Uses clinical guidelines and epidemiological statistics
- Produces datasets in formats such as CSV and FHIR

---

# DATA
# Slide 16 — How Synthea Generates Data

1. **Generate synthetic population**
   - Age, sex, demographics

2. **Simulate patient lifetimes**
   - Encounters, diagnoses, treatments

3. **Disease modules**
   - Simulate disease progression and treatment pathways

These modules function as **probabilistic state machines** generating realistic clinical histories.

---

# DATA
# Slide 17 — Cohort Selection (T2D Severity Experiment)

Dataset used: **Coherent Synthea (~8,000 patients)**

Final experimental cohort:

| Severity Group | Patients |
|---|---|
| Severe | 50 |
| Not severe | 50 |

---

## Ground Truth Severity Criteria

| Dimension | Clinical Signal | Operationalization |
|---|---|---|
| Complication burden | Microvascular complications | ≥3 complication types |
| Renal severity | Kidney dysfunction | Minimum eGFR < 60 |
| Treatment intensity | Insulin therapy | Insulin prescription present |

---

## Severe Classification Rule

Patient classified as **SEVERE if ≥2 of 3 dimensions present**

---

<sub>
Data considerations: HbA1c excluded due to non-physiological values in the dataset. Healthcare utilization excluded due to Synthea encounter inflation artifacts. Ground truth based on Cooper et al., 2025.
</sub>

# MOSAIC PIPELINE
# Slide — Experimental Setup

## Models Used in the Pipeline

| Variable | Model | Role |
|---|---|---|
| ASSESSOR_1_MODEL | deepseek-chat | Independent severity assessor |
| ASSESSOR_2_MODEL | gpt-4o | Independent severity assessor |
| CONSOLIDATOR_MODEL | claude-sonnet-4-6 | Framework consolidation & disagreement resolution |
| EXTRACTOR_MODEL | claude-sonnet-4-6 | EHR data extraction |

---

## Technical Environment

| Component | Role |
|---|---|
| CrewAI | Multi-agent orchestration (Agents, Tasks, Crews) |
| LiteLLM | Unified routing to multiple LLM providers |
| OpenAI API | Access to GPT-4o and DeepSeek-compatible endpoint |
| Anthropic API | Access to Claude Sonnet |
| Tavily | Literature retrieval for Phase 1 |
| Google Colab Pro+ | Execution environment |


---

# MOSAIC PIPELINE
# Slide — Phase 1: Framework Generation

## Architecture

Two independent researcher agents analyze clinical literature in parallel.
# MOSAIC PIPELINE
# Slide — Experimental Setup

## Models Used in the Pipeline

| Variable | Model | Role |
|---|---|---|
| ASSESSOR_1_MODEL | deepseek-chat | Independent severity assessor |
| ASSESSOR_2_MODEL | gpt-4o | Independent severity assessor |
| CONSOLIDATOR_MODEL | claude-sonnet-4-6 | Framework consolidation & disagreement resolution |
| EXTRACTOR_MODEL | claude-sonnet-4-6 | EHR data extraction |

---

## Technical Environment

| Component | Role |
|---|---|
| CrewAI | Multi-agent orchestration (Agents, Tasks, Crews) |
| LiteLLM | Unified routing to multiple LLM providers |
| OpenAI API | Access to GPT-4o and DeepSeek-compatible endpoint |
| Anthropic API | Access to Claude Sonnet |
| Tavily | Literature retrieval for Phase 1 |
| Google Colab Pro+ | Execution environment |


---

# MOSAIC PIPELINE
# Slide — Phase 1: Framework Generation

## Architecture

Two independent researcher agents analyze clinical literature in parallel.
┌─────────────────┐ ┌─────────────────┐
│ GPT-4o │ │ DeepSeek │
│ Research Agent │ │ Research Agent │
│ async execution │ │ async execution │
└────────┬────────┘ └────────┬────────┘
│ │
▼ ▼
┌────────────────────────────────┐
│ Claude Sonnet Consolidator │
│ Receives both outputs │
│ Resolves disagreements │
└────────────────────────────────┘

The result is a **frozen severity framework** used by all later stages of the pipeline.


---

# MOSAIC PIPELINE
# Slide — CrewAI Agent Architecture

Three agents are defined using **CrewAI with LiteLLM routing**.

| Agent | LLM | LiteLLM String | Role |
|---|---|---|---|
| researcher_gpt | GPT-4o | "gpt-4o" | Searches literature and proposes severity criteria |
| researcher_deepseek | DeepSeek | "deepseek/deepseek-chat" | Performs the same task independently |
| consolidator_claude | Claude Sonnet | "anthropic/claude-sonnet-4-6" | Synthesizes both outputs into a unified framework |

Execution pattern:

1. GPT-4o researcher runs  
2. DeepSeek researcher runs **in parallel**  
3. Claude consolidates both outputs  

This mirrors **expert consensus processes used in clinical framework design**.


---

# MOSAIC PIPELINE
# Slide — CrewAI Prompt Example

Example of agent initialization in the pipeline.

```python
researcher_gpt = Agent(
    role='Senior Clinical Researcher',
    goal='Find precise clinical criteria for classifying T2D severity.',
    tools=[medical_literature_search],
    llm="gpt-4o"
)

researcher_deepseek = Agent(
    role='Senior Clinical Researcher',
    goal='Find precise clinical criteria for classifying T2D severity.',
    tools=[medical_literature_search],
    llm="deepseek/deepseek-chat"
)

consolidator_claude = Agent(
    role='Chief Medical Informatician',
    goal='Synthesize research into a unified EHR severity framework.',
    llm="anthropic/claude-sonnet-4-6"
)
```

# MOSAIC PIPELINE
# Slide — Phase 1 Output (Framework Excerpt)

### Consolidated Type 2 Diabetes Severity Framework

---
......
## Section 1: Severity Classification Schema

T2D severity is classified into three tiers:

| Tier | Label | Clinical Interpretation |
|------|-------|------------------------|
| 1 | **Mild** | Glycemic parameters near diagnostic threshold; no significant end-organ involvement; manageable with lifestyle or single-agent therapy |
| 2 | **Moderate** | Suboptimal glycemic control; early or emerging comorbidities; requires intensification of pharmacotherapy |
| 3 | **Severe** | Poor glycemic control; established end-organ damage; complex multi-drug regimen or insulin dependence likely required |

> **Framework Rule 0 (Tier Assignment Principle):** A patient's overall severity tier is determined by the **highest tier triggered** across any single domain below. No averaging of domain scores is performed. Upward reclassification is mandatory when any domain criterion for a higher tier is met.

---

## Section 2: Domain-Specific Operational Criteria

### Domain 1: Glycated Hemoglobin (HbA1c)

**Source:** ISPAD Clinical Practice Consensus Guidelines 2024
**EHR Field Mapping:** Lab result — LOINC Code 4548-4 (HbA1c/Hemoglobin.total in Blood)
**Measurement Requirements:** Most recent result within the prior 6 months; if unavailable, use most recent result within 12 months with a flag for data staleness.........

# MOSAIC PIPELINE
# Slide — Phase 2 
**Architecture (Approach B):**
```
┌──────────────────────────────┐
│  compress_patient_data()     │  ← Python function (not LLM)
│  22 keywords + last-10 limit │
└──────────────┬───────────────┘
               ▼
┌──────────────────────────────┐
│  GPT-4o (Extractor)          │  ← Framework-guided extraction
│  128K context, high rate limit│
└──────────────┬───────────────┘
               ▼
    ┌──────────┴──────────┐
    ▼                     ▼
┌────────────┐    ┌────────────┐
│ GPT-4o     │    │ DeepSeek   │
│ Dr. A      │    │ Dr. B      │
│ (async)    │    │ (async)    │
└─────┬──────┘    └──────┬─────┘
      └──────────────────┘
               ▼
┌──────────────────────────────┐
│  Claude Sonnet (Consolidator)│ 
│  Receives ~5K tokens         │
└──────────────────────────────┘
```
# MOSAIC PIPELINE
# Slide — Phase 2 (EXAMPLE)
# MOSAIC PIPELINE
....This is the most fundamental and consequential disagreement. The assessors reached opposite binary conclusions — a divergence that demands careful resolution.

**Assessor A** classified the patient as **Moderate / NOT_SEVERE**, reasoning that:
- The A1c of 3.8% is "unusually low" and likely represents a "data entry error or anomaly" requiring verification
- Without a reliable A1c, glycemic control cannot be definitively characterized
- CKD is present but characterized as Stage 1 with eGFR around 62.5 mL/min
- No macrovascular complications are documented
- No "severe or critical complications such as substantial renal dysfunction" are present

**Assessor B** classified the patient as **Critical-Complex / SEVERE**, reasoning that:
- The A1c of 3.8% is **physiologically real and medically catastrophic** — representing severe, recurrent hypoglycemia, not a data artifact
- The discordance between A1c 3.8% and random glucose readings of 109–124 mg/dL is explained by the Somogyi effect (post-hypoglycemic rebound hyperglycemia), creating a coherent clinical narrative
- The patient is on insulin (70/30 mix) — the mechanism for the hypoglycemia is established
- eGFR has declined from ~120 mL/min to 62.5–71.0 mL/min over approximately 3 years — indicating Stage 3a CKD with alarming rate of progression, not Stage 1
- Severe untreated dyslipidemia (LDL 151 mg/dL) with no statin, borderline hypertension with no antihypertensive, and no SGLT2i/GLP-1 RA despite diabetic nephropathy represent a systemic failure of guideline-directed care
- Patient is 48–49 years old — decades of life at risk.....
- 
# Slide — Phase 2 (EXAMPLE)
===FINAL===
PATIENT_ID: be37f515-5d5b-3774-3b01-bee76bbd8bcb
FOUR_TIER: Critical-Complex
BINARY: SEVERE
ASSESSOR_A_TIER: Moderate
ASSESSOR_B_TIER: Critical-Complex
ASSESSOR_A_BINARY: NOT_SEVERE
ASSESSOR_B_BINARY: SEVERE
CONFIDENCE: High
AGREEMENT: None
KEY_EVIDENCE: A 49-year-old male on insulin 70/30 mix demonstrates A1c of 3.8% across five consecutive readings (2019-2021) indicating life-threatening recurrent hypoglycemia, combined with rapidly progressive diabetic nephropathy (eGFR declining from ~120 to 62.5 mL/min in three years), severe untreated dyslipidemia (LDL 151 mg/dL without statin), and complete absence of guideline-directed cardiorenal protective therapy — constituting Critical-Complex severity through immediate physiologic harm and comprehensive treatment failure.
===END===
