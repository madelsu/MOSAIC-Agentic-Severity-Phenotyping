<p align="center">
  <img src="assets/mosaic_logo.png" alt="MOSAIC Logo" width="600"/>
</p>

<h1 align="center">🧩 MOSAIC</h1>

<h3 align="center"><b>M</b>ulti-LLM <b>O</b>rchestrated <b>S</b>everity <b>A</b>ssessment <b>I</b>n <b>C</b>linical Records</h3>

<p align="center"><i>Agentic Severity Phenotyping in Electronic Health Records</i></p>

<p align="center">
  <img src="https://img.shields.io/badge/python-3.12-blue?logo=python&logoColor=white" alt="Python"/>
  <img src="https://img.shields.io/badge/CrewAI-orchestration-orange" alt="CrewAI"/>
  <img src="https://img.shields.io/badge/LLMs-GPT--4o%20|%20DeepSeek--V3%20|%20Claude%203.5%20Sonnet%20|%20Gemma%202%20|%20Qwen%202.5%20|%20Llama%203.1-purple" alt="LLMs"/>
  <img src="https://img.shields.io/badge/data-SyntheticMass%20V2-green" alt="Data"/>
  <img src="https://img.shields.io/badge/focus-T2D%20Severity%20Phenotyping-teal" alt="Focus"/>
</p>

---

## 📖 What is MOSAIC?

**MOSAIC** is a multi-agent AI framework for deriving and applying clinically meaningful disease severity phenotypes from longitudinal Electronic Health Record (EHR) data.

Rather than relying on a single model or a fixed rule-based classifier, MOSAIC orchestrates multiple Large Language Model (LLM) agents that independently assess patient evidence and then contribute to a consolidated final severity classification. The system is designed to mimic a simplified clinical consensus panel: independent interpretation first, structured consolidation second.

The current thesis implementation focuses on **Type 2 Diabetes Mellitus (T2D)** as a case study, while keeping the overall framework generalisable to other chronic conditions where severity is multidimensional, longitudinal, and clinically important.

---

## 🤔 Why MOSAIC?

Clinical phenotyping is the process of transforming raw EHR data into structured, computable representations of patient characteristics. It is essential for observational research, pharmacoepidemiology, clinical decision support, precision medicine, and real-world evidence generation.

MOSAIC was developed because severity phenotyping remains especially difficult:

- **EHR data are rich but messy.** Longitudinal records contain diagnoses, medications, laboratory values, procedures, and clinical observations, but these data are often fragmented, irregularly recorded, and originally collected for care or billing rather than research.
- **Traditional phenotyping is hard to scale.** Rule-based algorithms are interpretable but can require extensive manual design, expert input, local adaptation, and validation. Machine learning approaches can improve scalability, but often depend on labelled data that are expensive and difficult to obtain.
- **Severity is not a single variable.** Disease severity emerges from the interaction of glycaemic control, complication burden, treatment intensity, renal function, cardiovascular involvement, and longitudinal progression.
- **Binary phenotyping is not enough.** Most phenotyping studies focus on whether a condition is present or absent. For chronic diseases such as T2D, clinically meaningful risk depends on *how severe* the disease is and how it evolves over time.
- **Pharmacoepidemiology needs better severity capture.** Poorly measured severity can bias comparisons between patient groups, obscure treatment heterogeneity, and weaken outcome analyses. More robust severity phenotyping can support risk stratification, cohort enrichment, confounding control, and interpretation of treatment response.
- **LLMs offer a new reasoning interface.** Recent LLMs can integrate heterogeneous clinical information through natural language prompts, making them promising tools for structured reasoning over patient trajectories.
- **Agentic systems add transparency and disagreement analysis.** By comparing independent assessors and a consolidator, MOSAIC can expose agreement patterns, uncertainty, and failure modes rather than producing only a black-box label.
- **Open-weight models matter.** A central part of the thesis is testing whether privacy-preserving, locally runnable open-weight pipelines can approximate the behaviour and clinical validity of closed-weight API-based systems.

This work builds on EHR phenotyping literature, disease severity frameworks, LLM-based clinical phenotyping studies, and multi-agent medical reasoning systems, including Cooper et al. (2025), Young et al. (2008), Yang et al. (2022), Banda et al. (2018), Neves et al. (2025), Yan et al. (2024), and Berger et al. (2025).

---

## 🎯 Research Aim

This thesis investigates whether agentic LLM systems can be used to derive and apply clinically meaningful severity phenotypes from EHR data, and whether such phenotypes are valid in terms of both agreement with established clinical frameworks and association with downstream health outcomes.

The study focuses on **Type 2 Diabetes** as a case study, while maintaining a framework that can be extended to other chronic conditions.

---

## 🔬 Research Questions

| # | Research Question |
|---|---|
| **RQ1 — Framework Generation** | To what extent can agentic LLM systems derive clinically coherent and operationalisable severity phenotyping frameworks from published literature and guidelines from the web, and how do these compare to established expert-defined frameworks? |
| **RQ2 — Classification Performance** | When applied to longitudinal EHR data, how accurately and consistently can agentic LLM systems classify disease severity compared to validated clinical reference standards? |
| **RQ3 — Clinical Validity and Utility** | Do severity classifications generated by agentic LLM systems demonstrate clinical validity, as reflected in their association with downstream health outcomes such as mortality and healthcare utilisation? |
| **RQ4 — Agreement, Transparency, and Failure Modes** | What are the agreement patterns, reasoning characteristics, and failure modes of multi-agent LLM systems in severity phenotyping, and how do these vary across different model configurations? |

---

## 🏗️ The MOSAIC Framework

MOSAIC is organised as a three-phase pipeline.

<p align="center">
  <img src="assets/mosaic_framework.png" alt="MOSAIC Framework Diagram" width="800"/>
</p>

### Phase 1 — 📝 Severity Framework Generation

A disease-specific severity assessment framework is generated through automated synthesis of clinical literature and guidelines.

| Step | Description |
|---|---|
| 🔍 Independent Research | Parallel LLM research agents search and summarise clinical literature and guideline evidence. |
| 🧠 Evidence Synthesis | Agents identify clinically relevant severity markers, thresholds, and longitudinal signals. |
| 🧊 Consolidation | A consolidator model reconciles the findings into a frozen operational severity rubric. |
| 📌 Output | A four-tier T2D severity framework with explicit computable triggers and evidence requirements. |

The frozen framework is used unchanged during patient classification so that performance differences can be attributed to model reasoning rather than shifting rules.

### Phase 2 — 🏥 Patient Classification

The MOSAIC classification system uses a multi-agent orchestration architecture designed to mimic a clinical consensus panel. Two assessor agents reason independently before a consolidator issues the final severity verdict.

| Step | Description |
|---|---|
| 🩺 Assessor A | Independently reviews longitudinal patient evidence and assigns a severity tier. |
| 🧬 Assessor B | Independently reviews the same patient evidence from a different clinical/informatics perspective. |
| 🤝 Consolidation | A senior consolidator compares both assessments, resolves disagreements, and produces the final MOSAIC label. |
| 📝 Audit Trail | Agent reasoning, key evidence, confidence, and disagreement patterns are retained for qualitative analysis. |

### Phase 3 — 📊 Evaluation

## 🧪 Four Hypotheses at a Glance

| Hypothesis | Focus | Summary | Primary Output |
|---|---|---|---|
| **H1 — Bayesian Classification Agreement** | Agreement with algorithmic ground truth | MOSAIC severity-tier distributions are expected to be consistent with validated ground-truth frameworks at the population level. | Bayesian Dirichlet-Multinomial tier probabilities; posterior differences; 95% HDI. |
| **H2 — Frequentist Predictive Validity** | Clinical validity | Higher MOSAIC severity tiers are expected to predict worse downstream outcomes, including mortality, new complications, and healthcare utilisation. | Kaplan-Meier curves; Cox HR trend; RMST; Fine-Gray competing risks. |
| **H3 — Bayesian Pipeline Equivalence** | Closed-weight vs open-weight comparison | The open-weight MOSAIC pipeline is expected to be non-inferior or broadly equivalent to the closed-weight benchmark in severity classification behaviour. | Posterior comparison of CW and OW tier distributions; threshold-based equivalence assessment. |
| **H4 — Qualitative Evidence Transparency** | Reasoning and failure modes | MOSAIC disagreements with ground truth are expected to reveal clinically meaningful reasoning patterns rather than only random classification noise. | Qualitative taxonomy of `LLM_KEY_EVIDENCE` and `ASSESSOR_REASONING`; patient-level case studies. |

A fuller statistical analysis plan is documented separately in the methods/results materials.

---

## 🤖 LLMs & Tools

### Phase 1 — Framework Generation

| Role | Model | Purpose |
|---|---|---|
| **Research Agent 1** | OpenAI GPT-4o | Literature and guideline search; extraction of candidate severity markers. |
| **Research Agent 2** | DeepSeek-V3 / `deepseek-chat` | Independent literature and guideline search; alternative clinical synthesis. |
| **Consolidator** | Anthropic Claude 3.5 Sonnet | Reconciles outputs into the final frozen operational framework. |

### Phase 2 — Experimental Setup: Agentic Orchestrations

#### Setup A — Closed-Weight Pipeline (CW, API-Based)

This setup serves as the high-performance benchmark using proprietary models.

| Role | Model | Version / Identifier | Function |
|---|---|---|---|
| **Assessor 1** | GPT-4o | `gpt-4o` / GPT-4o API model | Clinical Informatician assessor. |
| **Assessor 2** | DeepSeek-V3 | `deepseek-chat` | Consultant Endocrinologist assessor. |
| **Consolidator** | Claude 3.5 Sonnet | Claude 3.5 Sonnet API model | Final decision maker and disagreement resolver. |

**Prompting strategy:** High-context prompts with detailed backstories, clinical reasoning requirements, structured evidence extraction, and explicit final-output schemas. This configuration leverages the stronger instruction-following and long-context stability of large proprietary APIs.

#### Setup B — Open-Weight Pipeline (OW, Local/Inference-Based)

This setup is the privacy-preserving and lower-cost alternative, running locally/in-notebook on an **NVIDIA A100 80 GB GPU**.

| Role | Model | Version / Identifier | Function |
|---|---|---|---|
| **Assessor 1** | Gemma 2 27B | `gemma-2-27b-it` | Clinical reasoning assessor. |
| **Assessor 2** | Qwen 2.5 14B | `Qwen2.5-14B-Instruct` | Structured-data parsing and severity logic assessor. |
| **Consolidator** | Llama 3.1 70B | `Meta-Llama-3.1-70B-Instruct` | Final consensus and rule-checking model. |

**Prompting strategy:** Condensed logic prompts with shorter instructions, strict hierarchical rules, and reduced narrative framing to minimise instruction drift in smaller context windows.

### Tools

| Tool | Purpose |
|---|---|
| **CrewAI** | Multi-agent orchestration using agents, tasks, and crews. |
| **LiteLLM** | Unified routing across model providers and local/open-weight endpoints. |
| **Tavily API** | Literature and guideline retrieval during Phase 1 framework generation. |
| **Google Colab Pro+** | Main runtime environment for notebook execution and GPU-backed experiments. |
| **Python / pandas / openpyxl** | EHR table processing, cohort construction, ground-truth generation, and result exports. |

---

## 📁 Repository Structure

The repository is organised around the main methodological components of the thesis.

```text
MOSAIC/
│
├── README.md
│
├── assets/
│   ├── mosaic_logo.png
│   └── mosaic_framework.png
│
├── Datasets/
│   └── README.md
│
├── Ground_Truths/
│   ├── Cooper_GT/
│   ├── Young_GT_DCSI/
│   └── ground_truth_generation_scripts/
│
├── Phase_1_Framework_Generation/
│   ├── prompts/
│   ├── agent_outputs/
│   ├── tavily_search_logs/
│   └── frozen_frameworks/
│
├── Phase_2_Patient_Classification/
│   ├── closed_weight_pipeline/
│   ├── open_weight_pipeline/
│   ├── prompts/
│   ├── orchestration_scripts/
│   └── patient_level_outputs/
│
└── Results/
    ├── statistical_analysis_plan/
    ├── tier_distribution_analysis/
    ├── agreement_analysis/
    ├── survival_analysis/
    ├── healthcare_utilisation/
    ├── qualitative_disagreement_analysis/
    └── figures/
```

> Raw patient-level datasets are **not** included in this repository. The `Datasets/` folder documents dataset provenance, expected file structure, and access information.

---

## 📊 Dataset

### Data Source: Synthea Synthetic EHR Ecosystem

MOSAIC uses synthetic EHR data generated by **Synthea**, an open-source simulator of complete patient lifespans and longitudinal health records. Synthea uses a modular state-transition architecture where clinical conditions are represented as state machines. Transitions between states, such as from pre-diabetes to Type 2 Diabetes, are governed by probabilities derived from epidemiological sources including the Global Burden of Disease and NHANES.

Synthea can export interoperable health data formats including **HL7 FHIR**, **C-CDA**, and **CSV**. This thesis uses the CSV representation.

### Dataset: SyntheticMass Version 2

The main dataset is **SyntheticMass Version 2**, released on **24 May 2017**. This version contains approximately **one million synthetic patient records** and is distributed as a roughly **21 GB CSV archive**.

### Rationale for Synthetic Data

Synthetic EHR data provide a privacy-by-design environment for developing and testing clinical AI pipelines. This is important because real-world EHR data are sensitive, access-restricted, and governed by GDPR/HIPAA and institutional approval processes.

Using Synthea enables:

- large-scale testing without exposing real patient data;
- iterative development of agentic AI workflows;
- reproducible experiments in a shared synthetic environment;
- preservation of realistic EHR structure, including longitudinal diagnoses, observations, medications, encounters, and procedures;
- reduced legal, security, and intellectual-property risk during early-stage method development.

---

## 🔒 Ground Truth Definitions

Two validated clinical frameworks are used as algorithmic gold standards. In this thesis, a gold-standard label refers to the most reliable available classification of a patient's phenotype status based on expert-defined criteria, validated clinical indexes, or structured clinical review logic.

### Young GT — Diabetes Complications Severity Index (DCSI)

The **Diabetes Complications Severity Index (DCSI)**, developed by Young et al. (2008), is a structured scoring system for quantifying cumulative diabetic complication burden using automated clinical data.

The DCSI evaluates seven physiological domains:

1. cardiovascular disease;
2. nephropathy;
3. retinopathy;
4. peripheral vascular disease;
5. stroke;
6. neuropathy;
7. metabolic complications.

Most domains are scored from 0 to 2, while neuropathy is binary, yielding a maximum total score of 13. The original study showed that DCSI was a stronger predictor of mortality and hospitalisation than a simple count of complications.

For comparability with MOSAIC's four-tier framework, continuous DCSI scores are mapped as:

| DCSI Score | Thesis Severity Tier |
|---|---|
| 0 | Baseline T2D |
| 1 | Mild |
| 2–3 | Moderate |
| ≥4 | Advanced/Critical |

> This binning is an adaptation for this thesis and does not correspond to severity tiers defined in the original DCSI publication.

### Cooper GT — 7-Dimensional Feasibility Framework

The **Cooper GT** is based on Cooper et al. (2025), who used a mixed-methods approach combining literature review, primary care database analysis, and expert consensus using the Nominal Group Technique to identify severity phenotypes for nine long-term conditions in primary care EHR.

For Type 2 Diabetes, Cooper et al. identified five green-rated severity phenotypes with high clinical importance and feasibility:

- microvascular complications;
- proteinuria;
- retinopathy staging;
- diabetes medications;
- diabetic foot ulcer risk score.

These markers do not directly define four severity tiers in the original publication. In this thesis, they are operationalised into a four-tier ground truth across seven EHR-derived dimensions:

1. microvascular complication count;
2. UACR;
3. retinopathy staging;
4. medication intensity / insulin use;
5. diabetic foot ulcer or amputation;
6. minimum eGFR;
7. maximum HbA1c.

The adapted tier boundaries are:

| Tier | Operational Definition |
|---|---|
| **Baseline T2D** | No complications, no macrovascular/microvascular flags, and fewer than two glucose-lowering medication classes. |
| **Mild** | HbA1c >8.0% or use of more than one glucose-lowering drug class. |
| **Moderate** | Any macrovascular or microvascular flag not meeting advanced/critical thresholds. |
| **Advanced/Critical** | eGFR <30, proliferative retinopathy, or diabetic foot ulcer/amputation. |

> This four-tier operationalisation is specific to this thesis and should be distinguished from Cooper et al., which provides ranked phenotype recommendations rather than explicit tier definitions.

---

## 🧭 Pharmacoepidemiological Study Design

To evaluate the predictive validity of MOSAIC severity classifications, the study uses a **longitudinal fixed-window pharmacoepidemiological design**. The key principle is that the exposure — the MOSAIC severity tier — is determined entirely from clinical information observed **before** the outcome window begins. This temporal separation is essential in observational pharmacoepidemiology, because mixing exposure assessment and outcome observation can introduce systematic bias.

Because Type 2 Diabetes is a chronic and progressive disease, a single snapshot of severity may not fully capture differences in patient risk over time. MOSAIC therefore evaluates severity at multiple clinically meaningful index dates across the disease trajectory.

### The 5+5 Observation Rule

Each index date defines two observation windows:

- **5-year lookback window:**  
  MOSAIC uses the previous 60 months of EHR data to assign a severity tier. This includes historical laboratory values, medication history, diagnoses, complications, and clinical encounters.

- **5-year follow-up window:**  
  After the index date, patients are followed prospectively for outcomes including all-cause mortality, new diabetes-related complications, and healthcare utilisation.

This structure ensures that severity is treated as a baseline exposure and that outcomes occur after the severity classification has been assigned.

### Index Dates

| Index Date | Anchor | Rationale |
|---|---|---|
| **T_Diag** | Initial Type 2 Diabetes diagnosis | Captures severity at earliest clinical recognition |
| **T_Tx** | First pharmacological treatment | Captures severity at the point where medication becomes necessary |
| **T₅** ★ **Primary** | Five years after first treatment | Primary validation landmark; separates early aggressive disease from slower progression and reduces immortal time bias |
| **T₁₀** | Ten years after first treatment | Captures long-term disease trajectory and later-stage progression |

### Primary Landmark: T₅

The primary validation analysis is performed at **T₅**, five years after first pharmacological treatment.

This landmark was chosen for two main reasons:

- **Sufficient clinical history:**  
  By T₅, patients have accumulated enough longitudinal EHR data for MOSAIC to assess disease trajectory, including treatment escalation, renal decline, glycaemic control, and complication development.

- **Immortal time bias mitigation:**  
  If patients were classified from diagnosis using information recorded several years later, patients assigned to higher severity tiers would necessarily have survived long enough to accumulate that evidence. This would create a misleading survival advantage for severe patients. By setting the landmark at T₅ and observing outcomes only after T₅, the analysis avoids counting this pre-landmark period as follow-up time.

The T₅ analysis therefore estimates the association between severity and future outcomes among patients who are alive and observable at five years after treatment initiation.

### Secondary Landmark Analyses

Although T₅ is the primary analysis point, additional analyses are performed at **T_Diag**, **T_Tx**, and **T₁₀**. These secondary landmarks are used to assess whether the relationship between MOSAIC severity tiers and downstream outcomes remains stable across different stages of the Type 2 Diabetes disease course.

Together, this design allows MOSAIC to be evaluated not only as a classification system, but as a clinically meaningful risk stratification tool for longitudinal pharmacoepidemiological research.

---
## 📈 Evaluation and Statistical Plan

The evaluation strategy combines agreement analysis, Bayesian distributional modelling, time-to-event analysis, and qualitative review of reasoning traces.

### H1 — Bayesian Classification Agreement

MOSAIC tier distributions are compared with ground-truth tier distributions using a Bayesian Dirichlet-Multinomial model. Posterior tier probabilities and 95% Highest Density Intervals (HDIs) are estimated for each classifier. Posterior differences are used to assess whether MOSAIC over- or under-assigns specific severity tiers relative to the ground truths.

### H2 — Predictive Validity

Clinical validity is evaluated using downstream outcomes after the index date.

Planned analyses include:

- Kaplan-Meier survival curves for all-cause mortality;
- Cox proportional hazards models, both crude and adjusted for age and sex;
- Restricted Mean Survival Time (RMST) over the fixed follow-up horizon;
- Fine-Gray competing risks models for new diabetes complications, treating death as a competing event;
- healthcare utilisation analyses including emergency department visits and outpatient encounters.

### H3 — Pipeline Equivalence

Closed-weight and open-weight MOSAIC pipelines are compared to assess whether open-weight models preserve the same clinical signal and tier distribution patterns as the proprietary benchmark. The main analysis uses Bayesian posterior comparisons of tier probabilities and equivalence-style interpretation of posterior differences.

### H4 — Evidence Transparency and Failure Modes

Disagreement cases are reviewed using the `LLM_KEY_EVIDENCE` and `ASSESSOR_REASONING` fields. The purpose is to distinguish clinically meaningful disagreement from model failure.

A qualitative taxonomy is used to identify patterns such as:

- **Clinical nuance:** the LLM identifies longitudinal risk before a hard rule threshold is crossed;
- **Medication sensitivity:** a ground truth up-tiers based on medication count while the LLM judges the evidence as insufficient;
- **Logic drift:** the LLM misapplies a rule, hallucinates a threshold, or ignores key evidence;
- **multi-domain synthesis:** the LLM escalates severity because several moderate signals jointly imply higher clinical risk.

---

## ⚙️ Environment & Reproducibility

| Component | Details |
|---|---|
| **Runtime** | Google Colab Pro+ |
| **Hardware** | High-RAM runtime; NVIDIA A100 High-RAM GPU when available |
| **Language** | Python 3.12 |
| **Core Libraries** | pandas, numpy, openpyxl, scipy, scikit-learn |
| **Orchestration** | CrewAI |
| **Model Routing** | LiteLLM |
| **Closed-Weight APIs** | OpenAI, Anthropic, DeepSeek |
| **Open-Weight Inference** | Local / notebook-based inference on A100 80 GB |
| **Search API** | Tavily |
| **Data Storage** | Google Drive mounted in Colab |

---

## 👥 Authors & Affiliation

Developed as a **Master's Thesis in Bioinformatics** at the **University of Copenhagen**.

| Role | Name | Affiliation |
|---|---|---|
| 🎓 **Thesis Student** | Manuela Del Castillo | MSc Bioinformatics, University of Copenhagen |
| 🧠 **Supervisor** | Maurizio Sessa, MPharm, PhD, Associate Professor | Drug Safety Group, Department of Drug Design and Pharmacology, University of Copenhagen |
| 🧠 **Supervisor** | Thomas Hamelryck, PhD, Professor in Machine Learning | Department of Computer Science & Department of Biology, University of Copenhagen |

---

## 📚 Key References

1. Yang S, Varghese P, Stephenson E, Tu K, Gronsbell J. Machine learning approaches for electronic health records phenotyping: a methodical review. *Journal of the American Medical Informatics Association*. 2022;30(2):367. doi:10.1093/JAMIA/OCAC216.
2. Neves B, et al. Zero-shot learning for clinical phenotyping: Comparing LLMs and rule-based methods. *Computers in Biology and Medicine*. 2025;192:110181. doi:10.1016/J.COMPBIOMED.2025.110181.
3. Pendergrass SA, Crawford DC. Using Electronic Health Records to Generate Phenotypes for Research. *Current Protocols in Human Genetics*. 2018;100(1):e80. doi:10.1002/CPHG.80.
4. Banda JM, Seneviratne M, Hernandez-Boussard T, Shah NH. Advances in Electronic Phenotyping: From Rule-Based Definitions to Machine Learning Models. *Annual Review of Biomedical Data Science*. 2018;1:53. doi:10.1146/ANNUREV-BIODATASCI-080917-013315.
5. Shivade C, et al. A review of approaches to identifying patient phenotype cohorts using electronic health records. *Journal of the American Medical Informatics Association*. 2013;21(2):221. doi:10.1136/AMIAJNL-2013-001935.
6. Cooper J, et al. Defining phenotypes of disease severity for long-term cardiovascular, renal, metabolic, and mental health conditions in primary care electronic health records: A mixed-methods study using the nominal group technique. *Journal of Biomedical Informatics*. 2025;166:104831. doi:10.1016/J.JBI.2025.104831.
7. Young BA, et al. Diabetes Complications Severity Index and risk of mortality, hospitalization, and healthcare utilization. *American Journal of Managed Care*. 2008.
8. Nair ATN, et al. Heterogeneity in phenotype, disease progression and drug response in type 2 diabetes. *Nature Medicine*. 2022;28(5):982–988. doi:10.1038/s41591-022-01790-7.
9. Reddy K, et al. Subphenotypes in critical care: translation into clinical practice. *The Lancet Respiratory Medicine*. 2020;8(6):631–643. doi:10.1016/S2213-2600(20)30124-7.
10. Gordon AC, et al. From ICU Syndromes to ICU Subphenotypes: Consensus Report and Recommendations for Developing Precision Medicine in the ICU. *American Journal of Respiratory and Critical Care Medicine*. 2024;210(2):155–166. doi:10.1164/RCCM.202311-2086SO.
11. Wang J, et al. Prompt engineering for healthcare: Methodologies and applications. *Meta-Radiology*. 2026;4(1):100190. doi:10.1016/J.METRAD.2025.100190.
12. Berger A, Khanna S, Sparrenberg L, Deußer T, Berghaus D, Sifa R. Reasoning LLMs in the Medical Domain: A Literature Survey. 2025. doi:10.1109/DSAA65442.2025.11247922.
13. Walonoski J, et al. Synthea: An approach, method, and software mechanism for generating synthetic patients and the synthetic electronic health care record. *Journal of the American Medical Informatics Association*. 2018.

---

## 📄 License

To be added.

---

## 🙏 Acknowledgements

To be added — including dataset creators, API providers, open-weight model developers, and supporting software frameworks.

---

<p align="center">
  <i>🧩 MOSAIC — Because understanding severity requires more than one perspective.</i>
</p>
