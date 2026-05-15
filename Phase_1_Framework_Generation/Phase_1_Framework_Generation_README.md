# Phase 1 — Framework Generation

> **MOSAIC: Multi-LLM Orchestrated Severity Assessment In Clinical Records**  
> Phase 1 generates the evidence-based Type 2 Diabetes severity assessment framework used by the MOSAIC pipeline.

---

## Overview

This folder contains the code and outputs for **Phase 1: Framework Generation** of the MOSAIC thesis project.  
The goal of this phase is to use an **agentic AI workflow** to search clinical literature, extract severity-relevant evidence, and consolidate the findings into a machine-actionable framework for Type 2 Diabetes (T2D) severity phenotyping.

The generated framework is later used as the fixed clinical rubric for patient-level severity classification in the downstream MOSAIC experiments.

---

## What This Phase Does

Phase 1 uses a **CrewAI multi-agent setup** to generate a severity assessment framework from clinical literature.

The workflow is designed around three main steps:

1. **Literature search**  
   A custom Tavily-based search tool retrieves clinical guideline and medical literature content relevant to T2D severity criteria.

2. **Parallel agent assessment**  
   Two independent clinical researcher agents search for severity markers and operational thresholds:
   - `GPT-4o` as a Senior Clinical Researcher
   - `DeepSeek` as a Senior Clinical Researcher

3. **Framework consolidation**  
   A Claude-based consolidator reviews both researcher outputs and synthesizes them into a single unified operational framework.

This design allows the framework to be generated through independent evidence retrieval followed by structured consensus synthesis, rather than relying on a single model output.

---

## Folder Structure

```text
Phase_1_Framework_Generation/
│
├── Copy_of_FRAMEWORK_GENERATION_PHASE_1.ipynb
│   └── Main notebook used to run the CrewAI framework-generation workflow.
│
├── README.md
│   └── This file.
│
├── Literature_Evidence/
│   └── Evidence and source material used by the agentic AI system to generate the framework.
│
├── Consolidated_Framework.md
│   └── Final consolidated T2D severity framework generated from the Phase 1 workflow.
│
└── Framework_Generation_Consolidator_Output.md
    └── Raw CrewAI generation output from the framework-generation run, including the consolidator answer and cited sources.
```

---

## Main Notebook

### `Copy_of_FRAMEWORK_GENERATION_PHASE_1.ipynb`

This notebook contains the full Phase 1 workflow.

It includes:

- installation of the required dependencies;
- configuration of API keys and model names;
- connection tests for OpenAI and DeepSeek models;
- definition of a custom Tavily literature search tool;
- creation of CrewAI agents and tasks;
- asynchronous execution of the two researcher agents;
- sequential consolidation of both researcher outputs;
- saving of the final framework and execution logs.

---

## Agentic AI Setup

The notebook defines three agents:

| Agent | Model | Role |
|---|---|---|
| Senior Clinical Researcher | `gpt-4o` | Searches for precise clinical criteria for classifying T2D severity. |
| Senior Clinical Researcher | `deepseek/deepseek-chat` | Independently searches for T2D severity criteria and operational thresholds. |
| Chief Medical Informatician | `anthropic/claude-sonnet-4-6` | Consolidates both researcher outputs into one operational EHR phenotyping framework. |

The two researcher agents are executed asynchronously, allowing them to independently retrieve and reason over evidence before the consolidator receives both outputs.

---

## Literature Search Tool

The notebook defines a custom CrewAI tool called:

```python
Deep Medical Literature Search
```

This tool uses the **Tavily API** to perform advanced literature searches and retrieve raw page content from the top results.

The search output includes:

- page title;
- source URL;
- raw content snippets;
- clinical guideline or literature evidence relevant to T2D severity.

This evidence is then passed to the researcher agents for interpretation and synthesis.

---

## Outputs

### `Literature_Evidence/`

This folder contains the evidence used by the agentic AI system during framework generation.  
It is included to make the framework-generation process more transparent and auditable.

### `Consolidated_Framework.md`

This is the final consolidated severity framework produced from the Phase 1 process.  
It represents the operational rubric used later in the MOSAIC classification pipeline.

### `Framework_Generation_Consolidator_Output.md`

This file contains the raw final CrewAI generation output from the consolidator.  
It documents the criteria returned by the agentic system and the sources cited during the framework-generation run.

---

## Why This Phase Matters

Severity phenotyping requires more than identifying whether a patient has Type 2 Diabetes.  
It requires the integration of multiple clinical dimensions, including glycemic control, comorbidities, cardiorenal risk, treatment intensity, insulin resistance, and social determinants of health.

Phase 1 provides the evidence-based clinical foundation for MOSAIC by transforming guideline-level information into a structured framework that can be applied to longitudinal EHR data.

By separating framework generation from patient classification, MOSAIC ensures that the downstream evaluation uses a fixed rubric rather than allowing the classification logic to change patient by patient.

---

## Reproducibility Notes

To run the notebook, the following API keys are required:

```python
OPENAI_API_KEY
ANTHROPIC_API_KEY
DEEPSEEK_API_KEY
TAVILY_API_KEY
```

The notebook was originally written for a Google Colab-style environment and includes output-saving and download logic using:

```python
from google.colab import files
```

If running locally, this download section may need to be adapted to save outputs directly to the local project directory.

---

## Dependencies

The main dependencies installed in the notebook are:

```bash
crewai[tools]
litellm
anthropic
duckduckgo-search
openpyxl
tavily-python
openai>=1.83.0,<2.0.0
```

---

## Phase 1 in the MOSAIC Pipeline

```text
Clinical Literature + Guidelines
            │
            ▼
Tavily Literature Search Tool
            │
            ▼
Parallel Researcher Agents
 GPT-4o                  DeepSeek
            │             │
            └──────┬──────┘
                   ▼
        Claude Consolidator
                   │
                   ▼
Final Operational T2D Severity Framework
                   │
                   ▼
Used in Phase 2 Patient-Level Severity Classification
```

---

## Repository Context

This folder belongs to the broader MOSAIC thesis project:

**MOSAIC — Multi-LLM Orchestrated Severity Assessment In Clinical Records**

The broader project evaluates whether agentic LLM systems can generate clinically meaningful severity phenotypes from longitudinal EHR data and whether those severity labels predict downstream clinical outcomes.

