# Phase 2 — Patient Classification

This folder contains the code used for **Phase 2 of MOSAIC: Patient Severity Classification**.

In this phase, the consolidated Type 2 Diabetes severity framework generated in **Phase 1** is applied to patient-level Electronic Health Record (EHR) data. The goal is to classify each patient into one of four ordinal Type 2 Diabetes severity tiers using a multi-agent LLM orchestration.

---

## Purpose of this phase

Phase 2 operationalises the evidence-based severity framework generated in Phase 1.

The pipeline takes longitudinal synthetic EHR data and asks multiple LLM-based clinical agents to independently assess each patient’s Type 2 Diabetes severity. Their outputs are then reviewed by a consolidator agent, which produces the final patient-level severity classification.

The four severity tiers used throughout this phase are:

1. **Baseline T2D**
2. **Mild Complications**
3. **Moderate Complications**
4. **Advanced/Critical**

This phase was designed to test whether agentic LLM systems can generate clinically meaningful severity phenotypes from longitudinal patient records using a fixed, evidence-based framework.

---

## Folder contents

```text
Phase_2_Patient_Classification/
│
├── CLOSED_WEIGHT_FINAL_SET_UP.ipynb
├── OPEN_WEIGHT_FINAL_SET_UP.ipynb
└── README.md
```

---

## Notebooks

### `CLOSED_WEIGHT_FINAL_SET_UP.ipynb`

This notebook contains the **closed-weight model setup** for patient classification.

It uses proprietary API-based models in a multi-agent architecture:

- **Assessor 1:** gpt-4o
- **Assessor 2:** deepseek-chat
- **Consolidator:** claude-sonnet-4-6

The closed-weight setup serves as the high-performance benchmark for the MOSAIC patient classification pipeline.

The notebook includes:

- dependency installation
- API key and model configuration
- upload of the frozen consolidated severity framework
- upload and processing of patient EHR data
- patient record compression and size checks
- multi-agent severity classification
- checkpointing and recovery logic
- post-batch audit of classification results

---

### `OPEN_WEIGHT_FINAL_SET_UP.ipynb`

This notebook contains the **open-weight model setup** for patient classification.

It runs local/open-weight models through Ollama and CrewAI, using an ensemble designed to run on GPU infrastructure.

The open-weight setup uses:

- **Assessor 1:** Gemma 2 27B
- **Assessor 2:** Qwen 2.5 14B
- **Consolidator:** Llama 3.1 70B

This setup was designed as a privacy-preserving and lower-cost alternative to the closed-weight API-based pipeline.

The notebook includes:

- GPU and hardware checks
- Ollama server setup
- local model pulling and loading
- CrewAI connection to local Ollama models
- upload of the frozen consolidated severity framework
- upload and processing of patient EHR data
- patient record compression and size checks
- multi-agent severity classification
- checkpointing and recovery logic
- post-batch audit of classification results

---

## Input data

Both notebooks are designed to classify patients from a structured Excel file containing longitudinal patient-level EHR data.

The main input file used in this phase is:

```text
200_patients_LLM_final_version_T5.xlsx
```

This file contains patient records prepared for the **T5 reclassification landmark**, corresponding to the 5-year point after initial Type 2 Diabetes treatment.

The notebooks also use the frozen severity framework generated in Phase 1:

```text
consolidated_framework.txt
```

---

## Note on compression maps

The notebooks include logic for handling patient records that may be too large for the model context window.

In principle, if a patient record exceeds the available context capacity, a compression map can be used to reduce the record size while preserving clinically relevant evidence.

However, in the experimentation performed for this thesis, this issue was not encountered. The patient records used in the final Phase 2 runs were small enough to be processed without requiring a separate master compression map.

---

## Classification workflow

The patient classification pipeline follows the same general structure in both notebooks.

### 1. Load patient records

The notebook loads the patient-level Excel file and extracts the available EHR evidence for each patient.

The relevant clinical evidence may include:

- diagnoses
- laboratory observations
- medication records
- procedures
- encounters
- care activity information

---

### 2. Build patient summaries

For each patient, the notebook builds a structured patient record from the available EHR tables.

The record is formatted so that the LLM agents can reason over the relevant clinical history within the 5-year T5 covariate window.

---

### 3. Apply the frozen framework

The consolidated Type 2 Diabetes severity framework from Phase 1 is loaded into the notebook.

This framework remains fixed during classification so that all patient-level decisions are made using the same severity rules.

---

### 4. Independent assessor classification

Two assessor agents independently evaluate each patient.

The assessors are given:

- the frozen severity framework
- the patient-level EHR record
- instructions to classify the patient into one of the four severity tiers
- instructions to provide supporting clinical evidence and reasoning
- instructions to provide tier support scores

The two assessors are intentionally given different clinical roles to mimic a clinical consensus setting.

---

### 5. Consolidator classification

A consolidator agent reviews the two independent assessments.

The consolidator resolves agreement or disagreement between the assessors and produces the final severity classification.

The final output includes:

- final severity tier
- binary classification
- assessor-level tier outputs
- confidence level
- agreement level
- key evidence supporting the decision
- tier support scores
- patient-level classification metadata

---

### 6. Checkpointing and recovery

Both notebooks include checkpointing logic so that long classification runs can be resumed after interruption.

Checkpoint files are saved during the run and can be reloaded if the runtime disconnects.

This was particularly important because patient-level multi-agent classification is computationally expensive and can take a long time when running across many patients.

---

### 7. Post-batch audit

After a batch completes, the notebooks include audit logic to summarise classification outputs.

This helps check:

- number of successfully classified patients
- number of failed patients
- distribution of severity tiers
- binary classification distribution
- inter-assessor agreement
- presence of unexpected or invalid outputs

---

## Closed-weight vs open-weight comparison

A key methodological feature of Phase 2 is the comparison between closed-weight and open-weight agentic pipelines.

| Setup | Models | Main role |
|---|---|---|
| Closed-weight | GPT-4o, DeepSeek, Claude Sonnet | High-performance benchmark |
| Open-weight | Gemma 2 27B, Qwen 2.5 14B, Llama 3.1 70B | Local, privacy-preserving, lower-cost alternative |

The same general classification logic is used in both setups, allowing comparison of whether open-weight models can reproduce similar clinical severity signals to proprietary models.

---

## Agent architecture

Both setups use a three-agent architecture:

1. **Dr. A — Clinical Informatician**
   - Focuses on EHR-based phenotyping.
   - Takes a methodical and conservative approach.
   - Prioritises explicitly documented evidence.

2. **Dr. B — Consultant Endocrinologist**
   - Focuses on clinical trajectory and disease progression.
   - Takes a more holistic, outcomes-focused approach.
   - Considers medication escalation, comorbidity burden, and longitudinal evidence.

3. **Chief Medical Informatician — Final Arbiter**
   - Reviews the two independent assessments.
   - Resolves disagreement.
   - Produces the final structured classification.

This setup is intended to mimic a simplified clinical consensus panel, where multiple perspectives are reviewed before a final judgement is made.

---

## Prompting strategy

The two notebooks use different prompt styles adapted to the model setup.

The **closed-weight setup** uses longer, high-context prompts with detailed instructions, richer clinical reasoning requirements, and more verbose agent backstories.

The **open-weight setup** uses shorter and more direct prompts, designed to reduce instruction drift and make the task easier for local/open-weight models to follow reliably.

---

# Exact prompts used

## Closed-weight setup

### Closed-weight agent personas

#### Assessor 1 — GPT-4o

```python
assessor_gpt = Agent(
    role="Dr. A — Clinical Informatician",
    goal=(
        "Classify this T2D patient's severity tier at their 5-year "
        "reclassification point using the provided framework AND all available "
        "clinical evidence in the record. Be methodical, data-driven, and "
        "evidence-based. Use everything in the record — not just lab values."
    ),
    backstory=(
        "You are a clinical informatician specialising in EHR-based T2D phenotyping. "
        "You are methodical and conservative — you only count what is explicitly "
        "documented. You systematically work through the framework domains but you "
        "also look at the broader clinical picture: medication trajectory, encounter "
        "frequency, comorbidity burden, and care plan activity. "
        "You never refuse to classify — if a patient has T2D with no documented "
        "complications, that is clinically meaningful and correctly classified as "
        "Baseline T2D, not 'insufficient data'."
    ),
    verbose=True,
    allow_delegation=False,
    llm="gpt-4o"
)
```

#### Assessor 2 — DeepSeek

```python
assessor_deepseek = Agent(
    role="Dr. B — Consultant Endocrinologist",
    goal=(
        "Classify this T2D patient's severity tier at their 5-year "
        "reclassification point using the provided framework AND all available "
        "clinical evidence. Take a holistic, outcomes-focused approach."
    ),
    backstory=(
        "You are a consultant endocrinologist with 20 years of clinical experience "
        "in T2D management. You take a holistic view — you consider the full clinical "
        "trajectory over the 5-year window: what conditions appeared, what medications "
        "were started or escalated, how labs evolved over time, what specialist referrals "
        "or care plans are present. "
        "You also read between the lines — a patient with frequent encounters and "
        "escalating medications tells a different story than one with a single annual "
        "review, even if the lab values are similar. "
        "You never use 'Insufficient Data' as a classification — a patient with "
        "confirmed T2D and no complication evidence is correctly classified as "
        "Baseline T2D. Absence of complications IS a finding."
    ),
    verbose=True,
    allow_delegation=False,
    llm="deepseek/deepseek-chat"
)
```

#### Consolidator — Claude Sonnet

```python
consolidator = Agent(
    role="Chief Medical Informatician — Final Arbiter",
    goal=(
        "Review both assessors' classifications, resolve disagreements, and "
        "produce a final severity classification with full justification."
    ),
    backstory=(
        "You are the chief medical informatician responsible for the final "
        "phenotyping decision at the T5 reclassification point. "
        "You review both assessors' reasoning, identify agreement and disagreement, "
        "and produce a definitive classification. "
        "When assessors disagree, you examine the specific evidence each cited "
        "and determine which interpretation is better supported. "
        "You never classify as 'Insufficient Data' — all patients have confirmed "
        "T2D and a 5-year EHR window; the correct classification for a patient "
        "with no documented complications is Baseline T2D."
    ),
    verbose=True,
    allow_delegation=False,
    llm="anthropic/claude-sonnet-4-6"
)
```

---

### Closed-weight assessor prompt

```text
You are classifying a Type 2 Diabetes patient's severity at their
T5 reclassification point (5 years after first T2D treatment).

The clinical data in the record covers the full window from first treatment
up to and including the T5 index date. Use EVERYTHING in the record.

=== CLASSIFICATION TIERS (use these exact labels) ===
  Tier 1: Baseline T2D
  Tier 2: Mild Complications
  Tier 3: Moderate Complications
  Tier 4: Advanced/Critical

IMPORTANT — SPARSE RECORDS:
All patients in this cohort have CONFIRMED Type 2 Diabetes.
If the record contains little clinical data beyond the T2D diagnosis:
  → DO NOT classify as "Insufficient Data"
  → Absence of documented complications IS a valid clinical finding
  → Classify as "Baseline T2D" and explain that no complication evidence
    was found in the 5-year window

=== CONSOLIDATED SEVERITY PHENOTYPING FRAMEWORK ===
{CONSOLIDATED_FRAMEWORK}
=== END FRAMEWORK ===

=== PATIENT CLINICAL RECORD (T5 covariate window) ===
{raw_record}
=== END RECORD ===

INSTRUCTIONS — USE ALL AVAILABLE EVIDENCE:

1. FRAMEWORK DOMAINS (systematic):
   Work through each domain in the framework. For each, state:
   - What evidence is present in the record
   - What threshold it meets (or does not meet)
   - Which tier it triggers

2. BEYOND THE FRAMEWORK — also consider:
   - Medication trajectory: How many antidiabetic agents? Any insulin?
     Escalation over time suggests worsening control.
   - Encounter frequency and type: Frequent ED visits, hospitalisations,
     or specialist referrals indicate higher burden.
   - Comorbidity pattern: Hypertension + dyslipidemia together vs alone.
   - Care plan content: Active diabetes management plans, foot care,
     ophthalmology referrals.
   - Lab trends over time: Worsening eGFR, rising HbA1c, new proteinuria.
   - Condition onset dates: Did complications develop early or late in window?

3. FINAL TIER ASSIGNMENT:
   Apply Framework Rule 0 (highest tier triggered across any domain).
   Apply Framework Rule 0a (Advanced/Critical if ≥2 Moderate Complications
   domain criteria are met simultaneously).

REQUIRED OUTPUT FORMAT:
- Domain-by-domain assessment with specific evidence citations
- Beyond-framework clinical observations
- Summary: which criteria ARE met vs NOT met
- Final Four-Tier Classification: [Baseline T2D / Mild Complications /
  Moderate Complications / Advanced/Critical]
- Final Binary Classification:
    COMPLEX     = Moderate Complications OR Advanced/Critical
    NOT_COMPLEX = Baseline T2D OR Mild Complications
- Confidence: [High / Medium / Low]
- Key uncertainties or data gaps

TIER SUPPORT SCORES (append at end of response):
Rate how strongly the evidence supports each tier, 0-100 independently.
  0   = no evidence supports this tier
  50  = ambiguous
  100 = evidence overwhelmingly supports this tier

SCORE_BASELINE_T2D: [0-100]
SCORE_MILD_COMPLICATIONS: [0-100]
SCORE_MODERATE_COMPLICATIONS: [0-100]
SCORE_ADVANCED_CRITICAL: [0-100]
```

---

### Closed-weight assessor tasks

```python
task_assess_gpt = Task(
    description=ASSESS_PROMPT,
    expected_output=(
        "Structured T2D severity assessment with domain-by-domain evidence, "
        "beyond-framework clinical observations, four-tier and binary "
        "classification, confidence level, and four tier support scores."
    ),
    agent=assessor_gpt,
    async_execution=True
)
```

```python
task_assess_deepseek = Task(
    description=ASSESS_PROMPT,
    expected_output=(
        "Structured T2D severity assessment with domain-by-domain evidence, "
        "beyond-framework clinical observations, four-tier and binary "
        "classification, confidence level, and four tier support scores."
    ),
    agent=assessor_deepseek,
    async_execution=True
)
```

---

### Closed-weight consolidator prompt

```text
You have received T2D severity assessments from two independent clinical assessors
for the same patient at their T5 reclassification point.
Patient ID: {patient_id}

REMINDER: All patients have confirmed T2D. Sparse record = Baseline T2D.
Absence of complications IS a finding. Never use Insufficient Data.

=================================================================
CRITICAL INSTRUCTION: OUTPUT THE ===FINAL=== BLOCK FIRST.
Write it as the VERY FIRST THING in your response, before any analysis.
This is mandatory — the block must appear at the start of your output.
=================================================================

===FINAL===
PATIENT_ID: {patient_id}
FOUR_TIER: [Baseline T2D / Mild Complications / Moderate Complications / Advanced/Critical]
BINARY: [COMPLEX / NOT_COMPLEX]
ASSESSOR_A_TIER: [Dr. A four-tier]
ASSESSOR_B_TIER: [Dr. B four-tier]
ASSESSOR_A_BINARY: [Dr. A binary]
ASSESSOR_B_BINARY: [Dr. B binary]
CONFIDENCE: [High / Medium / Low]
AGREEMENT: [Full / Partial / None]
INDEX_DATE_CONTEXT: {INDEX_DATE_LABEL}
KEY_EVIDENCE: [One sentence — single most decisive piece of evidence]
SCORE_BASELINE_T2D: [0-100]
SCORE_MILD_COMPLICATIONS: [0-100]
SCORE_MODERATE_COMPLICATIONS: [0-100]
SCORE_ADVANCED_CRITICAL: [0-100]
===END===

After the block, briefly provide:
- Where assessors agreed / disagreed and your resolution
- Key evidence driving the classification
- Tier score rationale (each score 0-100 independently)
- Any Synthea data quality notes
```

---

### Closed-weight consolidator task

```python
task_consolidate = Task(
    description=f"""
You have received T2D severity assessments from two independent clinical assessors
for the same patient at their T5 reclassification point.
Patient ID: {patient_id}

REMINDER: All patients have confirmed T2D. Sparse record = Baseline T2D.
Absence of complications IS a finding. Never use Insufficient Data.

=================================================================
CRITICAL INSTRUCTION: OUTPUT THE ===FINAL=== BLOCK FIRST.
Write it as the VERY FIRST THING in your response, before any analysis.
This is mandatory — the block must appear at the start of your output.
=================================================================

===FINAL===
PATIENT_ID: {patient_id}
FOUR_TIER: [Baseline T2D / Mild Complications / Moderate Complications / Advanced/Critical]
BINARY: [COMPLEX / NOT_COMPLEX]
ASSESSOR_A_TIER: [Dr. A four-tier]
ASSESSOR_B_TIER: [Dr. B four-tier]
ASSESSOR_A_BINARY: [Dr. A binary]
ASSESSOR_B_BINARY: [Dr. B binary]
CONFIDENCE: [High / Medium / Low]
AGREEMENT: [Full / Partial / None]
INDEX_DATE_CONTEXT: {INDEX_DATE_LABEL}
KEY_EVIDENCE: [One sentence — single most decisive piece of evidence]
SCORE_BASELINE_T2D: [0-100]
SCORE_MILD_COMPLICATIONS: [0-100]
SCORE_MODERATE_COMPLICATIONS: [0-100]
SCORE_ADVANCED_CRITICAL: [0-100]
===END===

After the block, briefly provide:
- Where assessors agreed / disagreed and your resolution
- Key evidence driving the classification
- Tier score rationale (each score 0-100 independently)
- Any Synthea data quality notes
""",
    expected_output=(
        "===FINAL=== block FIRST (immediately, before any analysis), "
        "then brief agreement analysis and evidence summary."
    ),
    agent=consolidator,
    context=[task_assess_gpt, task_assess_deepseek]
)
```

---

## Open-weight setup

### Open-weight agent personas

#### Assessor 1 — Gemma 2 27B

```python
assessor_gemma = Agent(
    role="Dr. A — Clinical Informatician",
    goal="Classify T2D severity at T5 using the framework. Be methodical and evidence-based.",
    backstory="Clinical informatician specialising in EHR-based T2D phenotyping. Conservative — only count what is explicitly documented.",
    verbose=False,
    allow_delegation=False,
    llm=llm_gemma
)
```

#### Assessor 2 — Qwen 2.5 14B

```python
assessor_qwen = Agent(
    role="Dr. B — Consultant Endocrinologist",
    goal="Classify T2D severity at T5 using the framework. Take a holistic, outcomes-focused approach.",
    backstory="Consultant endocrinologist with 20 years T2D experience. Considers the full 5-year clinical trajectory.",
    verbose=False,
    allow_delegation=False,
    llm=llm_qwen
)
```

#### Consolidator — Llama 3.1 70B

```python
consolidator = Agent(
    role="Chief Medical Informatician — Final Arbiter",
    goal="Review both assessors' classifications, resolve disagreements, produce final severity classification.",
    backstory="Chief medical informatician responsible for final phenotyping decisions.",
    verbose=False,
    allow_delegation=False,
    llm=llm_llama
)
```

---

### Open-weight assessor prompt

```text
Classify this T2D patient's severity at their T5 (5-year) reclassification point.

TIERS: Tier 1=Baseline T2D | Tier 2=Mild Complications | Tier 3=Moderate Complications | Tier 4=Advanced/Critical
Rule: Classify based on ALL available evidence — medications, procedures, conditions, and lab values — not just explicit complication codes. Only default to Baseline T2D if the record is truly empty of any complication signals. Never respond with "Insufficient Data".

=== FRAMEWORK ===
{CONSOLIDATED_FRAMEWORK}

=== PATIENT RECORD ===
{raw_record}

Respond with:
1. Domain assessment (1-2 sentences per domain only). After your domain assessment, explicitly state:
"Concurrent Tier 3 domains identified: [list them]"
and "Rule 0a check: [met / not met]".
This must appear before your final tier.
2. Any notable clinical observations not covered by the framework
3. Final tier + binary classification (COMPLEX or NOT_COMPLEX) + confidence (High/Medium/Low)
4. Scores (0-100 each, must sum to approximately 100):
SCORE_BASELINE_T2D: [0-100]
SCORE_MILD_COMPLICATIONS: [0-100]
SCORE_MODERATE_COMPLICATIONS: [0-100]
SCORE_ADVANCED_CRITICAL: [0-100]
```

---

### Open-weight assessor tasks

```python
task_assess_gemma = Task(
    description=ASSESS_PROMPT,
    expected_output="Brief domain assessment, final tier, binary, confidence, and four scores.",
    agent=assessor_gemma,
    async_execution=False
)
```

```python
task_assess_qwen = Task(
    description=ASSESS_PROMPT,
    expected_output="Brief domain assessment, final tier, binary, confidence, and four scores.",
    agent=assessor_qwen,
    async_execution=False
)
```

---

### Open-weight consolidator prompt

```text
Review the two assessors' outputs for patient {patient_id} and produce the final classification.
Before finalising, perform a structured self-check grounded in the framework's own decision logic:

1. Re-read the KEY_EVIDENCE from each assessor. Count how many distinct domain criteria
   at Tier 3 level or above are mentioned across both assessments combined.

2. Apply Framework Rule 0a: if two or more concurrent Tier 3 domain criteria are present
   in the record, the framework mandates upward reclassification to Tier 4. Have both
   assessors accounted for this rule, or did either stop at Tier 3 without checking it?

3. Apply Framework Rule 0: the final tier is the highest tier triggered by any single domain —
   not an average. If one assessor found a Tier 4 signal and the other found Tier 3,
   the correct resolution is Tier 4 unless the Tier 4 signal is explicitly contradicted
   by the record.

4. Check for under-classification risk: if the record contains insulin dependence alongside
   any documented complication, confirm this has been correctly weighted per the
   pharmacotherapy proxy in Domain 7.

5. Check for over-classification risk: confirm that a Tier 4 assignment is not based
   on a single isolated finding without corroborating domain evidence.

Output the ===FINAL=== block first, then write 2-3 sentences summarising your self-check

===FINAL===
PATIENT_ID: {patient_id}
FOUR_TIER: [Baseline T2D / Mild Complications / Moderate Complications / Advanced/Critical]
BINARY: [COMPLEX / NOT_COMPLEX]
ASSESSOR_A_TIER: [Dr. A tier]
ASSESSOR_B_TIER: [Dr. B tier]
CONFIDENCE: [High / Medium / Low]
AGREEMENT: [Full / Partial / None]
INDEX_DATE_CONTEXT: {INDEX_DATE_LABEL}
KEY_EVIDENCE: [One decisive sentence]
SCORE_BASELINE_T2D: [0-100]
SCORE_MILD_COMPLICATIONS: [0-100]
SCORE_MODERATE_COMPLICATIONS: [0-100]
SCORE_ADVANCED_CRITICAL: [0-100]
===END===
```

---

### Open-weight consolidator task

```python
task_consolidate = Task(
    description=CONSOLIDATE_PROMPT,
    expected_output="===FINAL=== block followed by 1-2 sentences of resolution reasoning.",
    agent=consolidator,
    context=[task_assess_gemma, task_assess_qwen]
)
```

---

## Output format

The final structured output for each patient includes:

| Output field | Description |
|---|---|
| `PATIENT_ID` | Unique patient identifier |
| `FOUR_TIER` | Final four-tier Type 2 Diabetes severity classification |
| `BINARY` | Binary complexity label |
| `ASSESSOR_A_TIER` | Severity tier assigned by Dr. A |
| `ASSESSOR_B_TIER` | Severity tier assigned by Dr. B |
| `CONFIDENCE` | Final confidence level |
| `AGREEMENT` | Whether assessors fully, partially, or did not agree |
| `INDEX_DATE_CONTEXT` | Index date used for classification |
| `KEY_EVIDENCE` | Most decisive evidence supporting the final classification |
| `SCORE_BASELINE_T2D` | Support score for Baseline T2D |
| `SCORE_MILD_COMPLICATIONS` | Support score for Mild Complications |
| `SCORE_MODERATE_COMPLICATIONS` | Support score for Moderate Complications |
| `SCORE_ADVANCED_CRITICAL` | Support score for Advanced/Critical |

---

## Why this phase matters

Phase 2 is the core implementation step of MOSAIC.

Phase 1 generates the clinical severity framework, but Phase 2 tests whether that framework can actually be applied to individual patient records by an agentic LLM system.

This phase therefore evaluates whether multi-agent LLMs can move beyond general clinical reasoning and perform structured, reproducible, patient-level severity phenotyping.

The outputs from this phase are later used in downstream evaluation against algorithmic ground truths and clinical outcomes.

---

## Relation to the full MOSAIC pipeline

```text
Phase 1 — Framework Generation
        ↓
Phase 2 — Patient Classification
        ↓
Phase 3 — Evaluation and Statistical Analysis
```

Phase 2 takes the framework created in Phase 1 and applies it to patient records.

The resulting severity classifications are then passed into Phase 3, where they are compared against ground-truth severity definitions and evaluated for predictive validity.

---

## Notes

This folder contains the final working notebooks for the two main patient classification setups.

The notebooks were developed in a Google Colab environment and include upload cells for data, framework files, checkpoint files, and patient-level classification inputs. Some paths may need to be adjusted if running the code outside Colab.

API keys and model credentials are intentionally not included in the repository.
