
# Phase 3 — Evaluation and Statistical Analysis

This folder contains the code, documentation, and statistical outputs used for **Phase 3 of MOSAIC: Evaluation and Statistical Analysis**.

In this phase, the patient-level severity classifications generated in **Phase 2** are evaluated against algorithmic ground truths and downstream clinical outcomes. The goal is to assess whether MOSAIC produces severity labels that are not only internally consistent, but also clinically meaningful and prognostically valid.

---

## Purpose of this phase

Phase 3 evaluates the performance and clinical validity of the MOSAIC agentic LLM pipeline.

The analysis is structured around four pre-specified hypotheses:

| Hypothesis | Framework | Main question | Primary output |
|---|---|---|---|
| **H1** | Bayesian | Does MOSAIC agree with the algorithmic ground truth? | Posterior probability that weighted κ meets the target threshold |
| **H2** | Frequentist | Do higher MOSAIC tiers predict worse mortality and complications? | Cox HR trend, RMST, and Fine–Gray competing-risks analysis |
| **H3** | Bayesian | Is the open-weight pipeline non-inferior to the closed-weight pipeline? | Posterior probability that both pipelines meet the H1 threshold |
| **H4** | Qualitative | Do disagreements reflect meaningful clinical reasoning? | Evidence-dimension analysis and patient-level case studies |

Together, these hypotheses assess agreement, predictive validity, pipeline reproducibility, and reasoning transparency.

---

## Folder contents

```text
Phase_3_Evaluation/
│
├── MOSAIC_SAP.docx
├── README.md
└── [analysis notebooks / scripts / outputs]
```

> `MOSAIC_SAP.docx` contains the full Statistical Analysis Plan used to define the evaluation strategy for this phase.

---

## Statistical Analysis Plan overview

The MOSAIC Statistical Analysis Plan uses a mixed-methods evaluation framework.

Different statistical frameworks are used depending on the nature of the research question:

- **H1 and H3** use a **Bayesian framework**, because the key question is the probability that MOSAIC meets a pre-specified performance threshold.
- **H2** uses a **frequentist survival-analysis framework**, because Cox regression and competing-risks methods are standard in epidemiology and allow direct comparison with reference studies.
- **H4** uses a **qualitative framework**, because the goal is to understand why MOSAIC disagrees with the algorithmic ground truths.

This design allows the evaluation to go beyond simple classification accuracy and assess whether MOSAIC severity labels are clinically interpretable, prognostically useful, and reproducible across model setups.

---

# H1 — Classification Agreement

## Formal statement

MOSAIC is hypothesised to assign Type 2 Diabetes patients to severity tiers consistently with the Cooper et al. algorithmic ground truth.

Agreement is evaluated at two complementary levels:

1. **Patient-level agreement**, measured using **quadratic weighted Cohen’s κ**.
2. **Population-level agreement**, measured using a **Bayesian Dirichlet-Multinomial model** comparing tier distributions.

Quadratic weighted κ is used because the four severity tiers are ordinal. A disagreement between adjacent tiers is less clinically severe than a disagreement between Baseline T2D and Advanced/Critical disease.

## Severity tiers

The four ordinal severity tiers evaluated in H1 are:

1. **Baseline T2D**
2. **Mild Complications**
3. **Moderate Complications**
4. **Advanced/Critical**

## Decision criteria

```text
Target agreement:
Pr(κw ≥ 0.80 | data) > 0.95

Minimum acceptable agreement:
Pr(κw ≥ 0.61 | data) > 0.95

Population-level bias:
Δk = p(MOSAIC,k) − p(GT,k)
95% HDI includes 0 → no systematic bias for tier k
```

## How H1 is tested

The Bayesian posterior distribution of weighted κ is estimated using MCMC.

The analysis reports:

- posterior mean weighted κ
- 95% highest density interval
- posterior probability that κw exceeds the target threshold
- posterior probability that κw exceeds the minimum acceptable threshold

A Dirichlet-Multinomial model estimates posterior tier probabilities for MOSAIC and the ground truth. The difference in posterior tier probabilities is computed for each severity tier to identify systematic over-classification or under-classification.

## H1 outputs

| Output | Type | Purpose |
|---|---|---|
| Bayesian posterior Pr(κw ≥ 0.80 \| data) | Formal confirmatory | Directly tests whether MOSAIC reaches the target agreement threshold |
| Dirichlet Δk per tier with 95% HDI | Model diagnostic | Detects systematic over- or under-classification by tier |
| Confusion matrices | Model diagnostic | Shows direction and severity of misclassification |
| Landis–Koch benchmark labels | Informal descriptive | Enables comparison with existing literature |
| Posterior density plots | Informal descriptive | Visualises uncertainty around κw |
| Per-tier precision, recall, and F1 | Informal descriptive | Characterises performance by severity category |

## Why Bayesian?

The H1 research question is probabilistic: *what is the probability that MOSAIC meets the clinically meaningful agreement threshold?*

A frequentist p-value would test a different question. The Bayesian posterior probability directly answers the decision question and provides an interpretable measure of uncertainty.

---

# H2 — Predictive Validity

## Formal statement

MOSAIC severity classifications are hypothesised to predict clinical outcomes in a monotonically increasing gradient.

Patients assigned to higher severity tiers should have:

- shorter time to all-cause mortality
- higher risk of first new diabetes complication
- higher risk of hospitalisation or increased healthcare utilisation

Expected pattern:

```text
HR Baseline < HR Mild < HR Moderate < HR Advanced/Critical
```

H2 provides outcome-based validation. Even if MOSAIC agrees with a ground truth, the severity labels are only clinically meaningful if they predict future outcomes.

---

## Study design

A longitudinal fixed-window design is used to ensure that severity classification is determined before outcome observation begins.

This prevents outcome leakage and reduces immortal time bias.

## Index dates

| Index date | Anchor | Rationale |
|---|---|---|
| **TDiag** | Initial T2D diagnosis | Severity at earliest clinical recognition |
| **TTx** | First pharmacological treatment | Severity at point of medication initiation |
| **T5** | 5 years post-treatment | Primary landmark; separates rapid and slow progressors |
| **T10** | 10 years post-treatment | Long-term disease trajectory assessment |

The primary analysis is performed at **T5**.

---

## The 5+5 rule

The Phase 3 evaluation uses a fixed-window pharmacoepidemiological design:

```text
5-year lookback window + 5-year prospective follow-up window
```

### 5-year lookback window

The lookback window is used to define MOSAIC severity.

It includes patient-level evidence before the index date, such as:

- laboratory values
- medication history
- diagnoses
- procedures
- encounters
- diabetes-related complications

### 5-year follow-up window

The follow-up window is used to observe outcomes after severity has already been assigned.

This temporal separation ensures that MOSAIC severity is treated as an exposure measured before the outcome.

---

## Primary outcome

The primary outcome is:

```text
All-cause mortality
```

The primary analysis uses:

- Kaplan–Meier survival curves
- Cox proportional hazards regression
- age- and sex-adjusted Cox regression
- restricted mean survival time at 5 years
- Nelson–Aalen cumulative hazard curves

---

## Secondary outcomes

Secondary outcomes include:

- first new diabetes complication
- hospitalisation
- emergency department visits
- outpatient care utilisation

For new complications, death is treated as a competing risk because death prevents subsequent complication occurrence.

The competing-risks analysis uses:

- cumulative incidence functions
- Aalen–Johansen estimator
- Fine–Gray subdistribution hazard model

---

## H2 outputs

| Output | Type | Purpose |
|---|---|---|
| Cox HR monotonic trend | Formal confirmatory | Tests ordered mortality risk gradient across severity tiers |
| RMST differences at 5-year horizon | Formal confirmatory | Provides PH-assumption-free estimate of event-free survival time |
| Fine–Gray subdistribution HR | Formal confirmatory | Models complications while accounting for death as a competing risk |
| Schoenfeld residuals | Model diagnostic | Tests proportional hazards assumption |
| Martingale residuals | Model diagnostic | Assesses linearity of continuous covariates such as age |
| Deviance residuals | Model diagnostic | Identifies influential observations |
| Love plots | Model diagnostic | Assesses covariate balance across severity tiers |
| Kaplan–Meier curves | Informal descriptive | Visualises tier-stratified survival |
| Nelson–Aalen cumulative hazard plots | Informal descriptive | Visualises cumulative mortality hazard |
| Cumulative incidence functions | Informal descriptive | Visualises complication incidence under competing risks |
| Multi-index-date comparison | Informal descriptive | Evaluates temporal sensitivity across TDiag, TTx, T5, and T10 |

## Why frequentist?

H2 uses frequentist survival analysis for three reasons:

1. **Methodological consistency** with the reference studies used as clinical benchmarks.
2. **Epidemiological convention**, because Cox regression is the standard approach for time-to-event outcomes.
3. **Inferential sufficiency**, because the mortality event count is large enough for standard asymptotic methods to be reliable.

Bayesian methods are reserved for H1 and H3, where the inferential question is explicitly about the posterior probability of meeting a threshold.

---

# H3 — Pipeline Equivalence

## Formal statement

The open-weight pipeline is hypothesised to produce classifications non-inferior to the closed-weight pipeline when both are evaluated against the same ground truth.

This hypothesis evaluates whether MOSAIC-style severity phenotyping can be reproduced using open-weight models rather than proprietary APIs.

---

## Pipelines compared

| Pipeline | Assessors | Consolidator | Deployment style |
|---|---|---|---|
| **Closed-weight** | GPT-4o + DeepSeek-V3 | Claude Sonnet | Proprietary API-based setup |
| **Open-weight** | Gemma 2 27B + Qwen 2.5 14B | Llama 3.1 70B | Local/open-weight setup on A100 GPU |

---

## Decision criteria

```text
Strong equivalence criterion:
Pr(κw,OW ≥ 0.80 | data) > 0.95
AND
Pr(κw,CW ≥ 0.80 | data) > 0.95
```

Both pipelines must independently satisfy the H1 agreement threshold.

---

## How H3 is tested

The same Bayesian weighted κ procedure used in H1 is applied separately to:

- open-weight MOSAIC vs ground truth
- closed-weight MOSAIC vs ground truth

The posterior performance gap is then reported:

```text
Δκ = κw,OW − κw,CW
```

A 95% HDI for Δκ describes the magnitude and direction of any difference between pipelines.

No formal non-inferiority margin is imposed, because no clinically validated benchmark exists for how much performance loss would be acceptable when replacing proprietary LLMs with open-weight equivalents.

---

## H3 outputs

| Output | Type | Purpose |
|---|---|---|
| Pr(κw,OW ≥ 0.80 \| data) | Formal confirmatory | Tests whether open-weight pipeline reaches the H1 target |
| Pr(κw,CW ≥ 0.80 \| data) | Formal confirmatory | Tests whether closed-weight pipeline reaches the H1 target |
| Posterior 95% HDI for Δκ | Model diagnostic | Describes magnitude and direction of performance gap |
| Inter-assessor κw comparison | Informal descriptive | Characterises pre-consolidation disagreement within each pipeline |
| Tier-level disagreement breakdown | Informal descriptive | Identifies whether disagreement clusters at specific severity boundaries |

## Why Bayesian?

H3 uses the same Bayesian framework as H1 for consistency.

The research question is again probabilistic: *what is the probability that each pipeline reaches the clinical agreement threshold?*

A frequentist non-inferiority test would require a pre-specified margin, which is not currently justified by the literature for this use case.

---

# H4 — Evidence Transparency and Qualitative Reasoning Analysis

## Formal statement

When MOSAIC disagrees with the Cooper or Young algorithmic ground truths, these disagreements are hypothesised to reflect clinically meaningful reasoning rather than random classification noise.

Specifically, disagreements may occur because:

1. the LLM weights a clinical dimension that the rule-based system underweights, or
2. the LLM integrates evidence across multiple dimensions in a way that single-threshold logic cannot reproduce.

H4 is exploratory and qualitative. It is not tested using a statistical decision threshold.

---

## Data used for H4

The qualitative analysis uses MOSAIC output fields such as:

- `LLM_KEY_EVIDENCE`
- `ASSESSOR_REASONING`
- final consolidator reasoning
- assessor-level severity classifications
- ground-truth classifications

The analysis focuses on disagreement cases where MOSAIC and the algorithmic ground truths assign different severity tiers.

---

## Component 1 — Evidence dimension frequency analysis

Each patient is coded across six evidence dimensions:

1. biomarker values
2. medical history and complications
3. treatment intensity
4. imaging
5. sociodemographics
6. encounter patterns

Each dimension is coded as:

- concordant
- LLM–LLM discordant
- LLM–Cooper discordant
- LLM–Young discordant
- absent

This identifies which clinical evidence domains most often drive disagreement.

---

## Component 2 — Agent reasoning comparison tables

For each selected patient, a structured comparison table is produced across:

- GPT-4o assessor
- DeepSeek assessor
- closed-weight consolidator
- Cooper ground truth
- Young ground truth

The table records:

- evidence used
- inference drawn
- consistency with final tier
- missing evidence
- disagreement source

Missingness is coded as **data absence**, not as negative evidence.

---

## Component 3 — Illustrative case studies

At least two case studies are selected:

1. one case of full concordance
2. one case of plausible LLM divergence

Each case receives a ternary judgement:

1. LLM interpretation clinically plausible
2. ground-truth interpretation more appropriate
3. genuinely ambiguous

This acknowledges that disagreement with a rule-based ground truth is not automatically an LLM error.

---

## H4 outputs

| Output | Type | Purpose |
|---|---|---|
| Evidence dimension frequency table | Informal descriptive | Identifies EHR domains driving LLM–GT divergence |
| Agent reasoning comparison tables | Informal descriptive | Compares assessor, consolidator, and ground-truth reasoning by evidence dimension |
| Illustrative case studies | Informal descriptive | Anchors disagreement patterns to concrete clinical examples |

## Why qualitative?

Agreement metrics show how often systems agree, but not why they agree or disagree.

A model may assign the correct tier for the wrong reason, or disagree with a rule-based ground truth for a clinically defensible reason. These patterns are invisible from κw or accuracy metrics alone.

H4 therefore complements the quantitative hypotheses by evaluating the reasoning quality, evidence use, and clinical plausibility of MOSAIC classifications.

---

# Summary of evaluation logic

```text
H1: Does MOSAIC agree with the algorithmic ground truth?
        ↓
H2: Do MOSAIC severity tiers predict future clinical outcomes?
        ↓
H3: Can open-weight models reproduce closed-weight performance?
        ↓
H4: When systems disagree, is the LLM reasoning clinically meaningful?
```

Together, these analyses evaluate MOSAIC from four complementary perspectives:

- **Agreement** with existing algorithmic severity definitions
- **Predictive validity** for future outcomes
- **Reproducibility** across proprietary and open-weight model setups
- **Transparency** of evidence use and clinical reasoning

---

## Relation to the full MOSAIC pipeline

```text
Phase 1 — Framework Generation
        ↓
Phase 2 — Patient Classification
        ↓
Phase 3 — Evaluation and Statistical Analysis
```

Phase 3 takes the patient-level classifications generated in Phase 2 and evaluates whether they are valid, reproducible, and clinically meaningful.

---

## Notes

This README summarises the Statistical Analysis Plan for Phase 3. The full SAP is stored in:

```text
MOSAIC_SAP.docx
```

The statistical analysis plan defines the confirmatory, diagnostic, and descriptive outputs used to evaluate MOSAIC.

API keys, raw patient-level files, and sensitive intermediate outputs are not included in this repository.
