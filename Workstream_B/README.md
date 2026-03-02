<h1 align="center">🏥 Workstream B (version A)</h1>

<p align="center"><i>MOSAIC Workstream B: Multi-LLM Agentic Severity Phenotyping Pipeline</i></p>

<p align="center">
  <img src="https://img.shields.io/badge/notebook-FRAMEWORK__MOSAIC__T2D-blue" alt="Notebook"/>
  <img src="https://img.shields.io/badge/cells-13-green" alt="Cells"/>
  <img src="https://img.shields.io/badge/runtime-Google%20Colab-orange" alt="Runtime"/>
  <img src="https://img.shields.io/badge/disease-Type%202%20Diabetes-red" alt="Disease"/>
</p>

---

## 📖 Table of Contents

- [Overview](#-overview)
- [Where This Fits in MOSAIC](#-where-this-fits-in-mosaic)
- [Architecture Evolution](#-architecture-evolution)
- [Prerequisites](#-prerequisites)
- [Inputs & Outputs](#-inputs--outputs)
- [Cell-by-Cell Walkthrough](#-cell-by-cell-walkthrough)
  - [Cell 1 — Dependency Installation](#-cell-1--dependency-installation)
  - [Cell 2 — API Keys & Model Configuration](#-cell-2--api-keys--model-configuration)
  - [Cell 3 — Data Upload & Pilot Selection](#-cell-3--data-upload--pilot-selection)
  - [Cell 4 — Phase 1: Severity Definition (CrewAI)](#-cell-4--phase-1-severity-definition-crewai)
  - [Cell 5 — Phase 2 V1: Raw Extraction (Pilot)](#-cell-5--phase-2-v1-raw-extraction-pilot)
  - [Cell 6 — Keyword Extraction V1 (324 Keywords)](#-cell-6--keyword-extraction-v1-324-keywords)
  - [Cell 7 — Keyword Extraction V2 (22 Keywords)](#-cell-7--keyword-extraction-v2-22-keywords)
  - [Cell 8 — Phase 2 V2: Approach B (Pilot)](#-cell-8--phase-2-v2-approach-b-pilot)
  - [Cell 9 — Phase 2 V2: 20-Patient Test Run](#-cell-9--phase-2-v2-20-patient-test-run)
  - [Cell 10 — Results Overview](#-cell-10--results-overview)
  - [Cell 11 — Single Patient Audit Trail](#-cell-11--single-patient-audit-trail)
  - [Cell 12 — Ground Truth Upload](#-cell-12--ground-truth-upload)
  - [Cell 13 — Phase 3: Evaluation vs Ground Truth](#-cell-13--phase-3-evaluation-vs-ground-truth)
- [Output Interpretation](#-output-interpretation)
- [Example Outputs](#-example-outputs)
- [Known Issues & Data Quality Notes](#-known-issues--data-quality-notes)
- [Cost Estimates](#-cost-estimates)

---

## 🔭 Overview

This notebook (`FRAMEWORK_MOSAIC_T2D`) implements the core MOSAIC pipeline: a multi-LLM agentic system that classifies **Type 2 Diabetes (T2D) severity** from synthetic Electronic Health Records. The notebook covers three phases:

| Phase | What It Does | Status |
|---|---|---|
| **Phase 1** | Multiple LLMs independently define T2D severity criteria from clinical literature, then a consolidator produces a frozen framework | ✅ Complete |
| **Phase 2** | LLM agents extract, assess, and classify each patient's severity using the frozen framework | 🔄 In Progress |
| **Phase 3** | Pipeline classifications are evaluated against a sealed ground truth | 🔄 Preliminary |

The notebook also documents the **iterative evolution** of the pipeline — from a naive raw-extraction approach that crashed on large patients, through keyword-based compression experiments, to the final working architecture.

---

## 🧩 Where This Fits in MOSAIC

```
MOSAIC Pipeline
│
├── 📝 Phase 1 — Severity Definition    ← Cell 4 (this notebook)
│   └── LLMs define what "severity" means
│
├── 🏥 Phase 2 — Patient Classification  ← Cells 5–11 (this notebook)
│   └── LLMs classify each patient
│
└── 📊 Phase 3 — Evaluation              ← Cells 12–13 (this notebook)
    └── Compare with sealed ground truth
```

---

## 🔄 Architecture Evolution

A key part of the MOSAIC contribution is the **iterative refinement** of the pipeline architecture. The notebook documents three versions, showing how practical constraints (context windows, rate limits, API costs) shaped the final design.

### ❌ Version 1 — Raw Extraction (Cell 5)

```
Claude Sonnet (Extractor) → GPT-4o + DeepSeek (Assessors) → Claude Sonnet (Consolidator)
```

- **Approach:** Send the full raw patient record to Claude Sonnet for extraction
- **Problem:** Patients with extensive medical histories exceeded context window limits. Claude also has stricter rate limits (tokens per minute) compared to OpenAI, causing throttling and crashes on larger records
- **Result:** Worked for the single pilot patient but failed on others with longer histories

### ⚠️ Compression Experiments (Cells 6–7)

To reduce patient record sizes, keyword-based filtering was tested:

| Version | Keywords | Result |
|---|---|---|
| V1 (Cell 6) | 324 (LLM-generated) | Still too broad — kept too much irrelevant data |
| V2 (Cell 7) | 22 (manually curated) | ✅ Aggressive but effective — focused on clinically relevant observations |

The 22 keywords cover: glycemic markers (HbA1c, glucose, fructosamine), renal function (eGFR, creatinine, microalbumin), lipids (cholesterol, LDL, HDL, triglycerides), vitals (blood pressure, BMI), and kidney extras (urea nitrogen).

### ✅ Version 2 — Approach B (Cells 8–9)

```
GPT-4o (Extractor) → GPT-4o + DeepSeek (Assessors) → Claude Sonnet (Consolidator)
```

- **Key change:** Switched extractor from Claude Sonnet → **GPT-4o** (128K context window, higher rate limits)
- **Compression:** 22 focused keywords + keep last 10 values per relevant observation type + only latest for non-relevant observations
- **Emergency fallback:** If a patient still exceeds 400K chars after compression, reduce to last 3 values per observation type
- **Claude usage optimized:** Claude Sonnet only called once per patient (consolidation step), receiving ~5K tokens of assessor summaries instead of raw data
- **Result:** 14/20 patients classified successfully in the test run

---

## 📋 Prerequisites

### 🔑 API Keys Required

You need active API keys for **four services**. Set them as environment variables in Cell 2:

| Service | Environment Variable | Used For |
|---|---|---|
| **OpenAI** | `OPENAI_API_KEY` | GPT-4o (extraction + assessment) |
| **DeepSeek** | `DEEPSEEK_API_KEY` | DeepSeek-chat (assessment) |
| **Anthropic** | `ANTHROPIC_API_KEY` | Claude Sonnet (consolidation) + Claude Opus (if needed) |
| **Serper** | `SERPER_API_KEY` | Web search for Phase 1 literature discovery |
| **Tavily** | `TAVILY_API_KEY` | Full-page content extraction for Phase 1 (Cell 4) |

### 📦 Python Dependencies (Cell 1)

| Package | Purpose | Version Notes |
|---|---|---|
| `crewai[tools]` | Multi-agent orchestration framework | Latest stable |
| `litellm` | Unified LLM API routing (routes to OpenAI, DeepSeek, Anthropic) | Latest stable |
| `anthropic` | Anthropic Python SDK for Claude | Latest stable |
| `openai` | OpenAI Python SDK | `>=1.83.0, <2.0.0` (pinned for CrewAI compatibility) |
| `duckduckgo-search` | Fallback search tool for CrewAI | Latest stable |
| `openpyxl` | Reading/writing Excel files | Latest stable |
| `tavily-python` | Tavily search API for full-page content | Latest stable |
| `pandas` | Data manipulation (pre-installed in Colab) | Pre-installed |

### 📂 Required Files

| File | Description | Provided By |
|---|---|---|
| `t2d_100_patients_pipeline.xlsx` | 100 T2D patient records (no labels) | Uploaded in Cell 3 |
| `t2d_100_patients_groundtruth_SEALED.xlsx` | Sealed ground truth labels | Uploaded in Cell 12 (Phase 3 only) |

### 💻 Runtime Environment

- **Google Colab** (free tier ~12GB RAM or Pro plan)
- Python 3.12
- Internet access required for API calls

---

## 📥 Inputs & Outputs

### Inputs

#### 📊 Patient Data File (`t2d_100_patients_pipeline.xlsx`)

An Excel workbook containing 11 sheets of synthetic EHR data for 100 T2D patients:

| Sheet | Description | Key Columns |
|---|---|---|
| `patient_list` | Master list of 100 patient IDs | `PATIENT_ID` |
| `patients` | Demographics (age, gender, race, etc.) | `Id`, `BIRTHDATE`, `GENDER`, `RACE`, `ETHNICITY` |
| `conditions` | Diagnoses (active and resolved) | `PATIENT`, `START`, `STOP`, `CODE`, `DESCRIPTION` |
| `observations` | Lab results, vitals, measurements | `PATIENT`, `DATE`, `CODE`, `DESCRIPTION`, `VALUE`, `UNITS` |
| `medications` | Prescriptions (active and historical) | `PATIENT`, `START`, `STOP`, `CODE`, `DESCRIPTION`, `REASONDESCRIPTION` |
| `encounters` | Hospital/clinic visits | `PATIENT`, `ENCOUNTERCLASS`, `START`, `STOP`, `REASONDESCRIPTION` |
| `careplans` | Care plans | `PATIENT`, `START`, `STOP`, `DESCRIPTION`, `REASONDESCRIPTION` |
| `allergies` | Known allergies | `PATIENT`, `START`, `STOP`, `CODE`, `DESCRIPTION` |
| `procedures` | Medical procedures | `PATIENT`, `DATE`, `CODE`, `DESCRIPTION`, `REASONDESCRIPTION` |
| `devices` | Medical devices | `PATIENT`, `START`, `STOP`, `CODE`, `DESCRIPTION` |
| `immunizations` | Vaccination records (omitted in processing — not relevant for T2D) | `PATIENT`, `DATE`, `CODE`, `DESCRIPTION` |

> ⚠️ **Important:** This file must NOT contain any ground truth columns (`RULE_LABEL`, `SEVERITY_SCORE`, `SELECTION_REASON`). Cell 3 automatically checks for this.

#### 🔒 Ground Truth File (`t2d_100_patients_groundtruth_SEALED.xlsx`)

Uploaded only at evaluation time (Cell 12). Contains the sealed severity labels:

| Column | Description |
|---|---|
| `PATIENT_ID` | Full patient UUID |
| `RULE_LABEL` | `SEVERE` or `NOT_SEVERE` |

Ground truth built using Rodgers et al. (JBI, 2025):
- **Dimension A:** ≥3 distinct complication types (nephropathy, retinopathy, neuropathy, stroke, CHD, MI)
- **Dimension B:** Minimum eGFR < 60
- **Dimension C:** Insulin use
- **SEVERE** = meets ≥2 of 3 dimensions

### Outputs

All outputs are saved to `/content/pipeline_outputs/` with the following structure:

```
pipeline_outputs/
├── phase1/
│   ├── openai/final_output.txt              ← GPT-4o severity definition
│   ├── deepseek/final_output.txt            ← DeepSeek severity definition
│   ├── consolidated_framework/
│   │   └── consolidated_framework.txt       ← 🧊 FROZEN framework (used in Phase 2)
│   └── full_execution_log.txt               ← Complete CrewAI verbose log
│
├── phase2/
│   ├── summaries/
│   │   ├── patient_XXXXXXXX_input.txt       ← Compressed patient data sent to extractor
│   │   └── extraction_XXXXXXXX.txt          ← Extractor's structured output
│   ├── openai/
│   │   └── assessment_XXXXXXXX.txt          ← Dr. A (GPT-4o) classification
│   ├── deepseek/
│   │   └── assessment_XXXXXXXX.txt          ← Dr. B (DeepSeek) classification
│   ├── consolidated/
│   │   └── classification_XXXXXXXX.txt      ← Final consolidated classification
│   ├── results_summary_20.csv               ← Summary table of all classifications
│   ├── framework_keywords.json              ← Keywords used for compression
│   └── errors.json                          ← Any patients that failed processing
│
├── evaluation/
│   ├── evaluation_results.csv               ← Full patient-by-patient comparison
│   └── evaluation_summary.json              ← Metrics summary (accuracy, kappa, etc.)
│
└── logs/
```

> 📝 Each patient output is saved in **both `.json` and `.txt`** formats for easy inspection via the `save_output()` utility function (defined in Cell 3).

---

## 🔬 Cell-by-Cell Walkthrough

---

### 📦 Cell 1 — Dependency Installation

**Purpose:** Install all required Python packages in a single command to avoid version conflicts.

**What it does:**
1. Installs `crewai[tools]` (includes CrewAI core + tool extensions), `litellm`, `anthropic`, `duckduckgo-search`, `openpyxl`, and `tavily-python`
2. Pins `openai>=1.83.0,<2.0.0` — this specific version range is required for compatibility between CrewAI's internal dependencies and the OpenAI SDK
3. Reinstalls `crewai` and `tavily-python` to ensure clean installation

**Key functions/packages:**
| Package | Role in Pipeline |
|---|---|
| `crewai[tools]` | Multi-agent orchestration — defines Agents, Tasks, and Crews |
| `litellm` | Routes API calls to different LLM providers using unified model strings (e.g., `deepseek/deepseek-chat`, `anthropic/claude-sonnet-4-6`) |
| `anthropic` | Direct Anthropic API access for testing |
| `openai` | Direct OpenAI API access + DeepSeek (OpenAI-compatible endpoint) |
| `tavily-python` | Full-page web content extraction for Phase 1 literature search |
| `openpyxl` | Reading Excel workbooks (`.xlsx`) with pandas |

**Why the version pinning?** CrewAI internally depends on specific OpenAI SDK versions. Without pinning, pip may resolve to an incompatible version that breaks agent execution with cryptic errors.

> ⏱️ Runtime: ~30–60 seconds

---

### 🔑 Cell 2 — API Keys & Model Configuration

**Purpose:** Configure all API keys, define model assignments, and verify connectivity to all services.

**What it does:**
1. Sets environment variables for four API services (OpenAI, Anthropic, DeepSeek, Serper)
2. Defines the **model roster** — which LLM plays which role:

| Variable | Model | Role |
|---|---|---|
| `ASSESSOR_1_MODEL` | `deepseek-chat` | Independent severity assessor |
| `ASSESSOR_2_MODEL` | `gpt-4o` | Independent severity assessor |
| `CONSOLIDATOR_MODEL` | `claude-opus-4-6` | Framework consolidation & disagreement resolution |
| `EXTRACTOR_MODEL` | `claude-sonnet-4-6` | EHR data extraction (V1) / consolidation (V2) |

3. **Connectivity tests** — sends a minimal request ("Hi") to each service to confirm API keys work and routing is correct:
   - OpenAI: Direct `OpenAI()` client
   - DeepSeek: `OpenAI()` client with `base_url="https://api.deepseek.com"` (DeepSeek uses an OpenAI-compatible API)
   - Claude: `Anthropic()` client
   - Serper: CrewAI's `SerperDevTool` with a test search query

**Key functions:**
| Function/Class | From | Purpose |
|---|---|---|
| `OpenAI()` | `openai` | Creates OpenAI/DeepSeek API client |
| `Anthropic()` | `anthropic` | Creates Anthropic API client |
| `SerperDevTool()` | `crewai_tools` | CrewAI-native web search tool |

**Expected output:**
```
✅ Configuration Loaded:
 📌 Assessor 1: deepseek-chat
 📌 Assessor 2: gpt-4o

--- 🧪 Testing OpenAI (Assessor 2) ---
✅ OpenAI works! → Hello! How can...
--- 🧪 Testing DeepSeek (Assessor 1) ---
✅ DeepSeek works! → Hi there!...
--- 🧪 Testing Claude (Consolidator) ---
✅ Claude works! → Hello!...
--- 🧪 Testing Serper Web Search ---
✅ Serper works! Preview: ...

🚀 All APIs tested and routing confirmed!
```

> ⚠️ **Security:** Never commit API keys to GitHub. Use Colab's Secrets manager or environment variables.

---

### 📤 Cell 3 — Data Upload & Pilot Selection

**Purpose:** Upload the patient data file, load all 11 sheets, verify no ground truth leakage, create output directories, and select a single patient for pilot testing.

**What it does:**

1. **Upload:** Uses Colab's `files.upload()` to interactively upload the Excel workbook
2. **Load all 11 sheets** into pandas DataFrames, printing column names and an example row for each
3. **Pilot mode:** Selects only the **first patient** from `patient_list` for initial testing:
   ```python
   PILOT_PATIENT_ID = ALL_PATIENT_IDS[0]
   PATIENT_IDS = [PILOT_PATIENT_ID]
   ```
4. **Ground truth leakage check:** Scans ALL sheets for forbidden columns (`RULE_LABEL`, `SEVERITY_SCORE`, `SELECTION_REASON`). This is critical — if labels leaked into the pipeline data, the entire experiment would have circular reasoning
5. **Output directory creation:** Creates the full directory tree under `/content/pipeline_outputs/`
6. **Defines `save_output()`** — a utility function used throughout the notebook:

**Key function — `save_output(phase_subdir, filename, content)`:**
| Parameter | Description |
|---|---|
| `phase_subdir` | Subfolder path (e.g., `"phase2/summaries"`) |
| `filename` | Base filename without extension |
| `content` | String, dict, or list to save |

Saves content in **both formats**: `.json` (structured) and `.txt` (human-readable). This dual-format approach ensures outputs are both machine-parseable and easy to inspect manually.

---

### 🔍 Cell 4 — Phase 1: Severity Definition (CrewAI)

**Purpose:** Multiple LLMs independently search clinical literature to define T2D severity criteria, then a consolidator produces a **frozen framework** used in all subsequent classification.

**Architecture:**
```
┌─────────────────┐    ┌─────────────────┐
│  GPT-4o          │    │  DeepSeek        │
│  (Researcher)    │    │  (Researcher)    │
│  async_execution │    │  async_execution │
└────────┬────────┘    └────────┬────────┘
         │                      │
         ▼                      ▼
    ┌────────────────────────────────┐
    │  Claude Sonnet (Consolidator)  │
    │  context=[task_gpt, task_ds]   │
    └────────────────────────────────┘
```

**What it does:**

1. **Custom Tavily tool** — `medical_literature_search()`:
   - Uses Tavily API with `search_depth="advanced"` and `include_raw_content=True`
   - Returns full page content (up to 15K chars per result, top 3 results)
   - This gives LLMs access to complete clinical guidelines, not just snippets

2. **Three CrewAI agents:**
   | Agent | LLM | LiteLLM String | Role |
   |---|---|---|---|
   | `researcher_gpt` | GPT-4o | `"gpt-4o"` | Searches literature, proposes severity criteria |
   | `researcher_deepseek` | DeepSeek | `"deepseek/deepseek-chat"` | Same task independently |
   | `consolidator_claude` | Claude Sonnet | `"anthropic/claude-sonnet-4-6"` | Synthesizes both into one framework |

3. **Async parallel execution** — Both researchers run simultaneously (`async_execution=True`), then the consolidator receives both outputs via `context=[task_gpt, task_deepseek]`

4. **Log capture** — Uses a custom `TeeOutput` class to capture ALL verbose CrewAI output while still printing to console. The full log is saved for audit purposes.

5. **Outputs saved & zipped:**
   - Individual agent outputs → `phase1/openai/`, `phase1/deepseek/`
   - Consolidated framework → `phase1/consolidated_framework/`
   - Full execution log → `phase1/full_execution_log.txt`
   - Everything zipped and downloaded as `phase1_outputs.zip`

**Key CrewAI concepts used:**
| Concept | Explanation |
|---|---|
| `Agent(llm="deepseek/deepseek-chat")` | LiteLLM string routing — the `deepseek/` prefix tells LiteLLM to call the DeepSeek API |
| `async_execution=True` | Tasks run in parallel instead of sequentially |
| `context=[task1, task2]` | Feeds the output of previous tasks as input to this task |
| `Crew(...).kickoff()` | Executes the full agent pipeline |

**Output:** A **frozen consolidated framework** — this framework is stored in the `result` variable and used by ALL subsequent cells. It defines the four severity tiers (Mild / Moderate / Severe / Critical-Complex) with specific thresholds for glycemic control, treatment intensity, complications, comorbidities, and functional status.

> ⏱️ Runtime: ~3–5 minutes (depends on API response times)

---

### 🧪 Cell 5 — Phase 2 V1: Raw Extraction (Pilot)

> ⚠️ **This version was superseded by Approach B (Cell 8).** Included for documentation of the iterative development process.

**Purpose:** First attempt at patient classification — send raw (uncompressed) patient data through a 4-agent pipeline.

**Architecture:**
```
Claude Sonnet (Extractor) → GPT-4o + DeepSeek (Assessors, parallel) → Claude Sonnet (Consolidator)
```

**What it does:**

1. **`extract_raw_patient_data(patient_id)`** — Extracts data from all clinical sheets with minimal cleanup:
   - Drops only non-clinical columns (SSN, addresses, costs, payer info, UDI)
   - Keeps everything else as-is — no compression, no filtering
   - Omits immunizations (not relevant for T2D)

2. **4 agents, 4 tasks:**
   - **Extractor (Claude Sonnet):** Reads the full raw record and organizes by framework domains (glycemic data, medications, complications, renal function, etc.)
   - **Dr. A (GPT-4o):** Classifies severity — methodical, conservative, data-driven persona
   - **Dr. B (DeepSeek):** Classifies severity — holistic, patient-outcomes focused persona
   - **Consolidator (Claude Sonnet):** Resolves disagreements, produces final tier

**Why it failed:** For patients with long medical histories, the raw record exceeded LLM context window limits. Claude Sonnet also has stricter tokens-per-minute rate limits compared to OpenAI, causing throttling when processing large records as input.

**Lesson learned:** Raw extraction is not scalable — compression is required to handle the full patient population.

---

### 🏷️ Cell 6 — Keyword Extraction V1 (324 Keywords)

> ⚠️ **This version produced too many keywords.** Superseded by Cell 7.

**Purpose:** Automatically extract clinical keywords from the consolidated framework to use for observation filtering/compression.

**What it does:**

1. Creates a **keyword extraction agent** (Claude Sonnet) that reads the frozen framework and identifies every lab test, vital sign, biomarker, and clinical measurement mentioned
2. Outputs a categorized JSON with keywords grouped by domain (glycemic, renal, lipids, vitals, other)
3. Includes fallback parsing — if JSON extraction fails, uses regex to find quoted strings

**Result:** ~324 keywords extracted. While comprehensive, this was **too permissive** — it kept too much irrelevant observation data, resulting in patient records that were still too large for the LLM context window.

---

### 🎯 Cell 7 — Keyword Extraction V2 (22 Keywords)

**Purpose:** Replace the LLM-generated keyword list with a manually curated, aggressive set of 22 keywords focused on T2D-relevant observations.

**The 22 keywords:**

| Category | Keywords |
|---|---|
| 🩸 Glycemic | `hba1c`, `hemoglobin a1c`, `glycated hemoglobin`, `glycohemoglobin`, `a1c`, `glucose`, `fructosamine` |
| 🫘 Renal | `egfr`, `glomerular filtration`, `creatinine`, `microalbumin`, `albumin creatinine ratio`, `uacr`, `urine albumin` |
| 🫀 Lipids | `cholesterol`, `ldl`, `hdl`, `triglyceride` |
| 💉 Vitals | `systolic blood pressure`, `diastolic blood pressure` |
| ⚖️ BMI | `body mass index` |
| 🫘 Kidney Extra | `urea nitrogen` |

**Verification:** Tests against the "worst case" patient (most observations) to confirm:
- How many observation rows match the keywords
- What the estimated row count would be with the "last 10 per type" limit
- How many unique non-matching descriptions remain (kept as latest-only)

**Why manual curation?** The LLM-generated list included tangentially related terms (e.g., vision tests, dental records, immunization markers) that inflated observation counts without adding clinical value for T2D severity assessment.

---

### 🏥 Cell 8 — Phase 2 V2: Approach B (Pilot)

**Purpose:** Refined architecture that addresses the context window and rate limit issues from V1. This is the **working version** used for all subsequent classification.

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
│  Claude Sonnet (Consolidator)│  ← Only Claude call per patient
│  Receives ~5K tokens         │
└──────────────────────────────┘
```

**Key changes from V1:**

| Aspect | V1 (Cell 5) | V2 / Approach B (Cell 8) |
|---|---|---|
| Extractor LLM | Claude Sonnet | **GPT-4o** (higher rate limits, 128K context) |
| Data sent to extractor | Raw (uncompressed) | **Compressed** (22 keywords + last-10 limit) |
| Claude calls per patient | 2 (extractor + consolidator) | **1** (consolidator only, ~5K tokens) |
| Classification output | Four-tier only | **Four-tier + Binary** (SEVERE / NOT_SEVERE) |
| Structured output | Free text | **`===FINAL===` block** for machine parsing |

**Compression function — `compress_patient_data(patient_id)`:**

For each patient, processes 10 clinical sheets:

| Sheet | Compression Strategy |
|---|---|
| Demographics | Drop PII and financial columns |
| Conditions | Drop encounter/patient IDs only — keep all diagnoses |
| **Observations** | 🔑 **Key compression:** Keyword-match → keep last 10 values; non-match → keep only latest value |
| Medications | Drop cost/payer columns, deduplicate by description+start+stop |
| Encounters | Summarize by class (count per type) + keep unique clinical reasons |
| Careplans | Drop IDs only |
| Procedures | Drop costs, deduplicate (keep most recent per description) |
| Allergies | Drop IDs only |
| Devices | Drop IDs and UDI |
| Immunizations | Omitted entirely (not relevant for T2D) |

**Structured output parsing — `===FINAL===` block:**

The consolidator is prompted to end with a machine-parseable block:

```
===FINAL===
PATIENT_ID: <full UUID>
FOUR_TIER: Mild/Moderate/Severe/Critical-Complex
BINARY: SEVERE/NOT_SEVERE
ASSESSOR_A_TIER: <Dr. A's tier>
ASSESSOR_B_TIER: <Dr. B's tier>
ASSESSOR_A_BINARY: <Dr. A's binary>
ASSESSOR_B_BINARY: <Dr. B's binary>
CONFIDENCE: High/Medium/Low
AGREEMENT: Full/Partial/None
===END===
```

This enables automated aggregation across patients while preserving full reasoning in the text before the block.

> ⏱️ Runtime: ~2–4 minutes per patient

---

### 🚀 Cell 9 — Phase 2 V2: 20-Patient Test Run

**Purpose:** Scale from 1 patient to 20 using Approach B, with production-grade robustness features.

**What it does:**

Everything from Cell 8, plus:

1. **Checkpoint/resume system** — `get_completed_patients()` scans the output directory for completed classifications. If the notebook crashes mid-run, re-running skips already-finished patients. This was essential because Colab sessions can disconnect unexpectedly.

2. **Pre-flight size check** — Before running any LLM calls, compresses all 20 patients and reports their sizes. Flags any that may exceed GPT-4o's context window (~512K chars / 128K tokens).

3. **Emergency compression** — If a patient exceeds 400K chars after standard compression, automatically reduces to **last 3 observations per type** (instead of 10). If still too large, the patient is skipped and logged as an error.

4. **Cooldown timer** — 30-second pause between patients to avoid rate limit throttling. This is conservative but prevents cascading failures.

5. **Error tracking** — Failed patients are logged to `errors.json` with the patient ID and error message. This enables post-run diagnosis and targeted retries.

6. **Progress reporting** — After each patient: elapsed time, count of completed/remaining, ETA for full run.

7. **Results summary** — At the end: distribution of four-tier classifications, binary labels, agreement levels, and confidence. Saves to `results_summary_20.csv`.

**Result:** 14/20 patients classified successfully. 6 patients exceeded context limits even with emergency compression (these patients had exceptionally long medical histories in the synthetic data).

> ⏱️ Runtime: ~60–90 minutes for 20 patients (with 30s cooldowns)

---

### 📊 Cell 10 — Results Overview

**Purpose:** Parse all completed classification outputs and display aggregate statistics and comparisons.

**What it does:**

1. **Loads all `.txt` files** from the consolidated output directory
2. **Parses `===FINAL===` blocks** to extract structured data
3. **Displays distributions** with visual bar charts (using Unicode block characters):
   - Four-tier distribution (Mild / Moderate / Severe / Critical-Complex)
   - Binary distribution (SEVERE / NOT_SEVERE)
   - Agreement levels (Full / Partial / None)
   - Confidence levels (High / Medium / Low)
4. **Assessor comparison table** — Side-by-side: Dr. A's tier, Dr. B's tier, final tier, agreement
5. **Binary comparison table** — Same for binary labels
6. **Evidence per patient** — Shows truncated reasoning (up to 1500 chars) from each consolidated classification

This cell is purely analytical — it reads saved outputs and does not make any API calls.

---

### 🔎 Cell 11 — Single Patient Audit Trail

**Purpose:** Deep-dive into a single patient's full classification pipeline — from raw input to final decision.

**What it does:**

Set `PATIENT_TO_VIEW` to any 8-character short ID, then the cell loads and displays the complete audit trail:

| Step | File Loaded | Contents |
|---|---|---|
| 📥 Raw Input | `patient_XXXXXXXX_input.txt` | Compressed patient data sent to the extractor |
| 🔍 Extraction | `extraction_XXXXXXXX.txt` | GPT-4o's structured clinical extraction |
| 🅰️ Dr. A | `assessment_XXXXXXXX.txt` | GPT-4o's domain-by-domain severity assessment |
| 🅱️ Dr. B | `assessment_XXXXXXXX.txt` | DeepSeek's domain-by-domain severity assessment |
| ⚖️ Consolidated | `classification_XXXXXXXX.txt` | Claude's final classification with agreement analysis |

**Why this matters:** Full traceability from raw data → extraction → independent assessments → final decision is critical for transparency and is a key thesis metric (RQ3). Every classification decision can be traced back to specific EHR data points.

---

### 📤 Cell 12 — Ground Truth Upload

**Purpose:** Upload the sealed ground truth file for evaluation.

**What it does:**

1. Uses Colab's `files.upload()` to upload the sealed Excel file
2. Displays column names, label distribution, and first 5 rows for verification

> ⚠️ **Circular reasoning protection:** This file is uploaded ONLY after all pipeline classifications are complete. The ground truth was sealed before any pipeline runs, and the pipeline data file (Cell 3) was verified to contain no ground truth columns.

---

### 📈 Cell 13 — Phase 3: Evaluation vs Ground Truth

**Purpose:** Unseal the ground truth and evaluate pipeline performance with comprehensive metrics.

**What it does:**

1. **Loads pipeline results** — Parses all `===FINAL===` blocks from consolidated outputs
2. **Matches with ground truth** — Joins on 8-character short patient IDs
3. **Confusion matrix:**
   ```
                         Predicted SEVERE    Predicted NOT_SEVERE
   Actual SEVERE              TP                    FN
   Actual NOT_SEVERE          FP                    TN
   ```

4. **Classification metrics:**

   | Metric | Formula | What It Measures |
   |---|---|---|
   | Accuracy | (TP + TN) / N | Overall correctness |
   | Sensitivity (Recall) | TP / (TP + FN) | Ability to detect SEVERE patients |
   | Specificity | TN / (TN + FP) | Ability to detect NOT_SEVERE patients |
   | Precision (PPV) | TP / (TP + FP) | When pipeline says SEVERE, is it right? |
   | F1 Score | 2 × (Precision × Recall) / (Precision + Recall) | Harmonic mean of precision and recall |
   | Cohen's Kappa | (P_observed − P_expected) / (1 − P_expected) | Agreement beyond chance |

5. **Kappa interpretation scale:**

   | Kappa Range | Interpretation |
   |---|---|
   | < 0.00 | Less than chance |
   | 0.00 – 0.20 | Slight agreement |
   | 0.21 – 0.40 | Fair agreement |
   | 0.41 – 0.60 | Moderate agreement |
   | 0.61 – 0.80 | Substantial agreement |
   | 0.81 – 1.00 | Almost perfect agreement |

6. **Individual assessor accuracy** — Each assessor (Dr. A / Dr. B) evaluated independently against ground truth with their own accuracy and Cohen's Kappa

7. **Inter-assessor agreement** — Dr. A vs Dr. B agreement rate and Kappa, plus detailed listing of every disagreement case showing which assessor was closer to ground truth

8. **Error analysis** — Lists every misclassified patient with pipeline prediction, ground truth, both assessors' predictions, agreement level, and confidence

9. **Saves all results:**
   - `evaluation/evaluation_results.csv` — Full patient-by-patient comparison
   - `evaluation/evaluation_summary.json` — Aggregated metrics

> ⚠️ Results are preliminary until all 100 patients have been classified.

---

## 📖 Output Interpretation

### Understanding the `===FINAL===` Block

Each patient classification ends with a structured block like:

```
===FINAL===
PATIENT_ID: 5a1a39df-xxxx-xxxx-xxxx-xxxxxxxxxxxx
FOUR_TIER: Severe
BINARY: SEVERE
ASSESSOR_A_TIER: Severe
ASSESSOR_B_TIER: Moderate
ASSESSOR_A_BINARY: SEVERE
ASSESSOR_B_BINARY: NOT_SEVERE
CONFIDENCE: Medium
AGREEMENT: Partial
===END===
```

**How to read this:**

| Field | Meaning |
|---|---|
| `FOUR_TIER` | Final consolidated classification (4 tiers) |
| `BINARY` | Final binary label for evaluation (Severe + Critical = SEVERE) |
| `ASSESSOR_A_TIER` / `ASSESSOR_B_TIER` | What each independent assessor concluded |
| `CONFIDENCE` | Consolidator's confidence in the final decision |
| `AGREEMENT` | Whether assessors agreed (Full), partially agreed (Partial), or completely disagreed (None) |

### Understanding the Evaluation Metrics

- **High accuracy + low kappa** = possibly just guessing the majority class
- **High sensitivity + low specificity** = catches severe cases but over-classifies
- **High specificity + low sensitivity** = conservative — misses some severe cases
- **Full agreement between assessors** = both LLMs independently reached the same conclusion — higher confidence
- **Partial/None agreement** = the consolidator had to resolve a dispute — check the reasoning

---

## ⚠️ Known Issues & Data Quality Notes

### Synthetic Data Artefacts (Synthea)

| Issue | Impact | Mitigation |
|---|---|---|
| 🩸 **HbA1c values are static** | Median 3.9%, often identical across time — unrealistic | Excluded from ground truth; noted as data quality concern |
| 🫘 **eGFR oscillates wildly** | Standard deviation ~31.6 within patients — Synthea artefact | Used as per-patient **minimum** only (not latest or average) |
| 🏥 **Healthcare utilization inflated** | 91% have ≥1 ED visit — non-discriminating | Excluded from ground truth |
| 💊 **Insulin prevalence high** | ~61% vs ~30% real-world | Noted as limitation; still used as severity marker |

---

## 💰 Cost Estimates

| Component | Cost Per Patient | Notes |
|---|---|---|
| GPT-4o (Extractor + Dr. A) | ~$0.05–0.10 | Two calls, largest input is compressed patient data |
| DeepSeek (Dr. B) | ~$0.002 | Very cheap per-token pricing |
| Claude Sonnet (Consolidator) | ~$0.07 | Only receives ~5K tokens of assessor summaries |
| **Total per patient** | **~$0.12–0.17** | |
| **100 patients** | **~$12–17** | |

> 💡 Approach B significantly reduced costs vs V1 by limiting Claude to one call per patient with minimal input.

---

<p align="center">
  <i>🧩 MOSAIC — Because understanding severity requires more than one perspective.</i>
</p>
