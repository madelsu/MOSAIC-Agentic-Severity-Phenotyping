# Definitive Unified Framework for Type 2 Diabetes (T2D) Severity Phenotyping
## A Consolidated Operational EHR Criteria Guide
## Version 3 — Tier Labels Aligned with DCSI Ground Truth (T5 Classification)

---

## Preamble

This framework synthesizes findings from multiple clinical research sources — including ISPAD Clinical Practice Consensus Guidelines 2024, IDF Clinical Practice Recommendations for Managing Type 2 Diabetes, and Novo Nordisk Guideline Directed Management — into a single, operational EHR phenotyping ruleset. It is intended for use by clinical informaticians, data scientists, and EHR implementation teams to enable consistent, reproducible classification of T2D severity at the point of care and in retrospective cohort studies.

The framework resolves discrepancies between sources, establishes hierarchical scoring logic, and defines computable, threshold-based criteria suitable for integration into clinical decision support systems, quality registries, and population health dashboards.

---

## Section 1: Severity Classification Schema

T2D severity is classified into four tiers aligned with DCSI-based ground truth classification: [REVISION v1→v2: changed "four tiers" to "four tiers"]

| Tier | Label | Clinical Interpretation |
|------|-------|------------------------|
| 1 | **Baseline T2D** | Glycemic parameters near diagnostic threshold; no significant end-organ involvement; manageable with lifestyle or single-agent therapy |
| 2 | **Mild Complications** | Suboptimal glycemic control; early or emerging comorbidities; requires intensification of pharmacotherapy |
| 3 | **Moderate Complications** | Poor glycemic control; established end-organ damage; complex multi-drug regimen or insulin dependence likely required; single-system predominant involvement |
| 4 | **Advanced/Critical** | End-stage or multisystem disease; multiple concurrent severe complications; advanced renal failure (eGFR <15 mL/min/1.73 m² or on dialysis), insulin dependence combined with established macrovascular disease, or multisystem involvement requiring specialist-led, coordinated care |

[REVISION v1→v2: Added Tier 4 Advanced/Critical row to the tier classification table; added clarifying phrase "single-system predominant involvement" to Tier 3 Moderate Complications to distinguish it from Tier 4]

> **Framework Rule 0 (Tier Assignment Principle):** A patient's overall severity tier is determined by the **highest tier triggered** across any single domain below. No averaging of domain scores is performed. Upward reclassification is mandatory when any domain criterion for a higher tier is met.

> **Framework Rule 0a (Critical-Complex Escalation Principle):** Tier 4 (Advanced/Critical) is triggered when a patient meets **two or more concurrent Tier 3 (Moderate Complications) domain criteria**, OR meets any single domain criterion explicitly designated as a Critical-Complex threshold (e.g., eGFR <15 mL/min/1.73 m² or dialysis dependence, or insulin dependence co-occurring with established macrovascular disease across ≥2 vascular territories). A mandatory specialist referral flag must accompany any Tier 4 assignment. [REVISION v1→v2: New rule added to operationalize the Critical-Complex tier escalation logic, consistent with existing Framework Rule 0 hierarchy]

---

## Section 2: Domain-Specific Operational Criteria

### Domain 1: Glycated Hemoglobin (HbA1c)

**Source:** ISPAD Clinical Practice Consensus Guidelines 2024
**EHR Field Mapping:** Lab result — LOINC Code 4548-4 (HbA1c/Hemoglobin.total in Blood)
**Measurement Requirements:** Most recent result within the prior 6 months; if unavailable, use most recent result within 12 months with a flag for data staleness.

| Severity Tier | HbA1c Threshold |
|---------------|----------------|
| Baseline T2D (Tier 1) | 6.5% – <7.0% |
| Mild Complications (Tier 2) | 7.0% – <8.0% |
| Moderate Complications (Tier 3) | ≥8.0% – <11.0% |
| Advanced/Critical (Tier 4) | ≥11.0% in conjunction with ≥1 additional Tier 3 or Tier 4 domain criterion |

[REVISION v1→v2: Split original Tier 3 (≥8.0%) into Tier 3 (≥8.0% – <11.0%) and Tier 4 (≥11.0% with concurrent severe domain criterion); the ≥8.0% lower boundary is preserved unchanged; the upper sub-threshold of 11.0% reflects a clinically meaningful marker of severely uncontrolled glycemia in the context of multisystem disease, consistent with existing domain logic]

**Operational Notes:**
- HbA1c is the **primary glycemic anchor domain**. All other glycemic domains (FPG, PPG) are confirmatory or supplementary.
- In patients with hemoglobinopathies (e.g., sickle cell trait, HbS/C variants), HbA1c is unreliable. FPG or PPG must serve as the primary glycemic domain in these cases. Flag condition codes: ICD-10 D57.x (Sickle-cell disorders), D58.x (Other hereditary hemolytic anemias).
- Units: If reported in mmol/mol (IFCC units), apply conversion: HbA1c (%) = [mmol/mol ÷ 10.929] + 2.15.
- An HbA1c ≥11.0% as a standalone finding does **not** independently trigger Tier 4; it requires co-occurrence with at least one additional Tier 3 or Tier 4 domain criterion per Framework Rule 0a. [REVISION v1→v2: Operational note added to clarify that HbA1c ≥11.0% alone is not a standalone Advanced/Critical trigger]

---

### Domain 2: Fasting Plasma Glucose (FPG)

**Source:** IDF Clinical Practice Recommendations for Managing Type 2 Diabetes
**EHR Field Mapping:** Lab result — LOINC Code 1558-6 (Fasting glucose [Mass/volume] in Serum or Plasma)
**Measurement Requirements:** Confirmed fasting state (≥8 hours documented); most recent value within 6 months.

| Severity Tier | FPG Threshold |
|---------------|--------------|
| Baseline T2D (Tier 1) | 100–125 mg/dL (5.6–6.9 mmol/L) |
| Mild Complications (Tier 2) | 126–154 mg/dL (7.0–8.6 mmol/L) |
| Moderate Complications (Tier 3) | >154 mg/dL (>8.6 mmol/L) |
| Advanced/Critical (Tier 4) | >154 mg/dL (>8.6 mmol/L) in conjunction with ≥1 additional Tier 3 or Tier 4 domain criterion per Framework Rule 0a |

[REVISION v1→v2: Added Advanced/Critical (Tier 4) row; the Tier 3 FPG threshold (>154 mg/dL) is preserved identically; Tier 4 is triggered only when this threshold co-occurs with additional severe domain criteria, consistent with Rule 0a]

**Operational Notes:**
- FPG operates as a **secondary glycemic domain**, active when HbA1c is unavailable, flagged as unreliable, or when acute hyperglycemia monitoring is clinically indicated.
- A single FPG value is sufficient for tier assignment; however, for EHR-based phenotyping without clinical context, use the **median of the last two available fasting glucose values** to reduce artifact from transient illness or dietary non-compliance.
- Conflicting FPG and HbA1c tiers: Prioritize the higher tier per Framework Rule 0.

---

### Domain 3: Postprandial Plasma Glucose (PPG)

**Source:** Novo Nordisk Guideline Directed Management
**EHR Field Mapping:** Lab result or glucometer reading — LOINC Code 1521-4 (Glucose [Mass/volume] in Serum or Plasma —2 hours post dose glucose)
**Measurement Requirements:** Measured 1.5–2.5 hours after the start of a standardized or documented meal; most recent value within 6 months.

| Severity Tier | PPG Threshold |
|---------------|--------------|
| Baseline T2D (Tier 1) | 140–199 mg/dL (7.8–11.0 mmol/L) |
| Mild Complications (Tier 2) | 200–299 mg/dL (11.1–16.6 mmol/L) |
| Moderate Complications (Tier 3) | ≥300 mg/dL (≥16.7 mmol/L) |
| Advanced/Critical (Tier 4) | ≥300 mg/dL (≥16.7 mmol/L) in conjunction with ≥1 additional Tier 3 or Tier 4 domain criterion per Framework Rule 0a |

[REVISION v1→v2: Added Advanced/Critical (Tier 4) row; the Tier 3 PPG threshold (≥300 mg/dL / ≥16.7 mmol/L) is preserved identically; Tier 4 requires co-occurrence with additional severe domain criteria per Rule 0a]

**Operational Notes:**
- PPG is a **tertiary glycemic domain**, used primarily when both HbA1c and FPG are unavailable or flagged as unreliable, or as a supplementary signal in Continuous Glucose Monitoring (CGM)-integrated EHR environments.
- In CGM-equipped patients, use **Time Above Range (TAR >180 mg/dL)** as a proxy for PPG burden: TAR >25% aligns with Mild Complications; TAR >50% aligns with Moderate Complications.
- PPG alone should not downgrade a severity tier established by HbA1c or FPG.

---

### Domain 4: Body Mass Index (BMI) — Pediatric/Youth Populations

**Source:** ISPAD Clinical Practice Consensus Guidelines 2024
**EHR Field Mapping:** Vital signs — BMI-for-age percentile (LOINC 39156-5; percentile computed against CDC or WHO growth charts)
**Applicable Population:** Patients aged <18 years at time of T2D diagnosis or phenotyping.

| Severity Tier | BMI-for-Age Percentile |
|---------------|----------------------|
| Baseline T2D (Tier 1) | ≥85th percentile (Overweight) |
| Mild Complications (Tier 2) | ≥95th percentile (Obese) |
| Moderate Complications (Tier 3) | >99th percentile (Severely Obese) |
| Advanced/Critical (Tier 4) | >99th percentile AND concurrent Tier 3 or Tier 4 criterion in ≥1 additional domain (e.g., eGFR <15, established CVD with insulin dependence) per Framework Rule 0a |

[REVISION v1→v2: Added Advanced/Critical (Tier 4) row; the Tier 3 BMI threshold (>99th percentile) is preserved identically; Tier 4 requires co-occurrence with another severe domain criterion; BMI alone does not independently trigger Tier 4]

**Operational Notes:**
- For **adult populations (≥18 years)**, use absolute BMI thresholds as a contextual modifier (not a standalone tier determinant):
  - BMI 25.0–29.9 kg/m² (Overweight): No independent tier escalation.
  - BMI 30.0–34.9 kg/m² (Obese Class I): Mild Complications contextual flag.
  - BMI ≥35.0 kg/m² (Obese Class II/III): Moderate Complications contextual flag; triggers mandatory comorbidity domain review.
- Adult BMI contextual flags do **not** independently assign a severity tier but must prompt evaluation of the Cardiorenal and Comorbidity domains.
- Percentile computation must reference the patient's age in months and biological sex for accuracy. If growth chart data is unavailable in the EHR, absolute BMI ≥30 kg/m² may be used as an approximation for adolescents aged 16–17 years pending clinical review.

---

### Domain 5: Comorbid Conditions

**Source:** IDF Clinical Practice Recommendations for Managing Type 2 Diabetes
**EHR Field Mapping:** Active problem list (ICD-10-CM codes) and encounter diagnoses; medication lists (RxNorm codes) as corroborating evidence.

| Severity Tier | Comorbidity Criteria |
|---------------|---------------------|
| Baseline T2D (Tier 1) | No active hypertension, dyslipidemia, or cardiovascular disease |
| Mild Complications (Tier 2) | Presence of **hypertension** (ICD-10: I10) **OR dyslipidemia** (ICD-10: E78.x) on active problem list |
| Moderate Complications (Tier 3) | Presence of **cardiovascular disease** (ICD-10: I20-I25 [CAD/IHD], I50.x [Heart failure], I63-I69 [Stroke/CVA], I73.9 [PVD]) **OR** hypertension **AND** dyslipidemia co-occurring |
| Advanced/Critical (Tier 4) | Established macrovascular disease across **≥2 distinct vascular territories** (e.g., concurrent CAD [I20-I25] AND stroke history [I63-I69]; or CAD AND PVD [I73.9]) **AND** concurrent insulin dependence documented in the active medication list |

[REVISION v1→v2: Added Advanced/Critical (Tier 4) row; Tier 3 comorbidity criteria are preserved unchanged; Tier 4 requires macrovascular disease in ≥2 vascular territories co-occurring with insulin dependence, operationalizing the "combination of insulin dependency + established macrovascular disease" definition in the framework revision mandate]

**Operational Notes:**
- Comorbidities must appear on the **active problem list** with a date of onset or confirmation within the record. Historical, resolved, or rule-out diagnoses are excluded unless explicitly re-listed as active.
- Corroborating medication evidence (e.g., antihypertensive medications [RxNorm: ACE inhibitors, ARBs, calcium channel blockers], statins [RxNorm: atorvastatin, rosuvastatin]) present in the active medication list strengthens comorbidity confirmation if ICD-10 coding is absent or incomplete.
- **Discrepancy Resolution:** The original source (IDF) states that hypertension, dyslipidemia, or CVD "indicates moderate to severe severity" without sub-stratifying. This framework resolves the ambiguity: hypertension or dyslipidemia alone = Mild Complications (Tier 2); CVD or dual comorbidity combination = Moderate Complications (Tier 3), aligned with cardiorenal risk stratification logic from Novo Nordisk source.
- For Tier 4 assignment in this domain, insulin dependence is confirmed by active prescription of any insulin formulation (RxNorm: insulin glargine, insulin detemir, insulin aspart, insulin lispro, NPH insulin, or premixed insulin formulations) with ≥2 active fills within the prior 6 months. [REVISION v1→v2: Operational note added to define computable confirmation criteria for insulin dependence in the context of Tier 4 Domain 5 assignment]

---

### Domain 6: Cardiorenal Risk Factors

**Source:** Novo Nordisk Guideline Directed Management
**EHR Field Mapping:** Lab results (eGFR — LOINC 62238-1), echocardiography reports, cardiology encounter diagnoses, nephrology notes, and ICD-10 active problem list.

| Severity Tier | Cardiorenal Criteria |
|---------------|---------------------|
| Baseline T2D (Tier 1) | eGFR ≥60 mL/min/1.73 m² AND no documented heart failure or ASCVD |
| Mild Complications (Tier 2) | CKD with eGFR **30–59 mL/min/1.73 m²** (CKD Stage 3, ICD-10: N18.3) **OR** mild-moderate heart failure (HFpEF or HFrEF, NYHA Class I-II, ICD-10: I50.x) without established ASCVD |
| Moderate Complications (Tier 3) | eGFR **15–29 mL/min/1.73 m²** (CKD Stage 4, ICD-10: N18.4) **OR** established ASCVD (ICD-10: I20-I25, I63-I69) **WITH** heart failure (ICD-10: I50.x) **OR** UACR >300 mg/g (macroalbuminuria, LOINC: 9318-7) |
| Advanced/Critical (Tier 4) | eGFR **<15 mL/min/1.73 m²** (CKD Stage 5, ICD-10: N18.5) **OR** end-stage renal disease on dialysis (ICD-10: N18.6, Z99.2) **OR** eGFR **15–29 mL/min/1.73 m²** (CKD Stage 4) co-occurring with established ASCVD AND heart failure (triple cardiorenal–cardiovascular complexity) |

[REVISION v1→v2: Tier 3 cardiorenal criteria revised to specify CKD Stage 4 (eGFR 15–29) explicitly and exclude CKD Stage 5 / dialysis, which are now designated Tier 4; original Tier 3 covered eGFR <30 (Stages 4–5) as a combined threshold — this is now split at eGFR 15 to create a clinically meaningful boundary; all other Tier 3 criteria (ASCVD + HF, UACR >300) are preserved; Tier 4 row added with eGFR <15 / dialysis as primary triggers and triple cardiorenal–cardiovascular complexity as a secondary trigger]

**Operational Notes:**
- eGFR must be calculated using the **CKD-EPI 2021 creatinine equation** (race-free); most recent value within 90 days for active monitoring; within 12 months for baseline phenotyping.
- Two eGFR values separated by ≥90 days, both below threshold, are required to confirm CKD staging per KDIGO guidelines. If only one value is available, apply a "Probable CKD" flag pending confirmation.
- UACR (Urine Albumin-to-Creatinine Ratio) is a standalone Severe trigger even in the absence of reduced eGFR: UACR >300 mg/g = Moderate Complications (Tier 3); UACR 30–299 mg/g = Mild Complications (Tier 2) contextual flag.
- **Dialysis confirmation:** ICD-10 code Z99.2 (Dependence on renal dialysis) or active procedure code for hemodialysis (CPT 90935, 90937) or peritoneal dialysis (CPT 90945, 90947) in the last 90 days confirms dialysis dependence for Tier 4 assignment. [REVISION v1→v2: Operational note added to define computable EHR criteria for dialysis dependence as a Tier 4 trigger]
- **Discrepancy Resolution:** The original source specifies eGFR <60 as the Moderate threshold. This framework refines the lower boundary: eGFR 30–59 = Moderate; eGFR 15–29 = Moderate Complications (Tier 3); eGFR <15 or dialysis = Advanced/Critical (Tier 4), consistent with KDIGO CKD staging and standard nephrology practice. [REVISION v1→v2: Discrepancy resolution note updated to reflect three-way eGFR stratification (Baseline T2D / Mild Complications / Moderate Complications / Advanced/Critical) replacing the original two-way split (Mild Complications / Moderate Complications)]

---

### Domain 7: Insulin Resistance and Beta-Cell Function

**Source:** ISPAD Clinical Practice Consensus Guidelines 2024
**EHR Field Mapping:** Lab results (fasting insulin — LOINC 20448-7; C-peptide — LOINC 1986-9), clinical documentation, endocrinology notes.

| Severity Tier | Insulin Resistance & Beta-Cell Function Criteria |
|---------------|------------------------------------------------|
| Baseline T2D (Tier 1) | HOMA-IR <2.5 (calculated) AND C-peptide within normal range (0.5–2.0 nmol/L or 1.5–6.0 ng/mL); responsive to oral agents |
| Mild Complications (Tier 2) | HOMA-IR ≥2.5 (significant insulin resistance) AND mildly reduced C-peptide (0.2–0.5 nmol/L or 0.6–1.5 ng/mL); partial beta-cell dysfunction; suboptimal response to oral agents documented |
| Moderate Complications (Tier 3) | C-peptide <0.2 nmol/L (<0.6 ng/mL) OR documented failure of ≥2 oral antidiabetic agent classes AND clinical evidence of rapid beta-cell decline (progressive HbA1c rise despite adherence); insulin initiation or dependence |
| Advanced/Critical (Tier 4) | C-peptide <0.2 nmol/L (<0.6 ng/mL) AND insulin dependence AND ≥1 additional Tier 3 or Tier 4 domain criterion (e.g., eGFR <15, macrovascular disease ≥2 territories); representing advanced beta-cell failure within a multisystem disease context |

[REVISION v1→v2: Added Advanced/Critical (Tier 4) row; Tier 3 criteria are preserved identically; Tier 4 requires the conjunction of near-absent C-peptide, insulin dependence, AND concurrent severe domain involvement, ensuring that beta-cell failure alone does not independently trigger Tier 4 without accompanying multisystem complexity]

**HOMA-IR Calculation:**
```
HOMA-IR = [Fasting Insulin (μIU/mL) × Fasting Glucose (mmol/L)] ÷ 22.5
```

**Operational Notes:**
- Fasting insulin and C-peptide are not universally collected in routine EHR workflows. This domain is **active only when these values are present** in the structured lab record.
- When lab values are absent, this domain defaults to a **pharmacotherapy proxy**:
  - Tier 1 proxy: Controlled on metformin monotherapy.
  - Tier 2 proxy: Requires ≥2 oral antidiabetic agents (e.g., metformin + SGLT2i, or metformin + GLP-1RA).
  - Tier 3 proxy: Requires insulin therapy (any formulation) OR ≥3 antidiabetic agents with documented treatment failure.
  - Tier 4 proxy: Requires insulin therapy AND documented treatment failure of ≥3 antidiabetic agent classes AND concurrent Tier 3 or Tier 4 classification in ≥1 additional domain. [REVISION v1→v2: Tier 4 pharmacotherapy proxy added; all pre-existing Tier 1–3 proxy definitions preserved unchanged]
- The pharmacotherapy proxy is a **weak signal** and must be corroborated by glycemic domain criteria before influencing tier assignment.
- **Discrepancy Resolution:** The original source provides qualitative descriptions only ("significant insulin resistance," "rapid decline"). This framework operationalizes these descriptions with HOMA-IR and C-peptide thresholds drawn from ADA/EASD consensus and endocrinology reference standards.

---

### Domain 8: Lifestyle and Social Determinants of Health (SDOH)

**Source:** Novo Nordisk Guideline Directed Management
**EHR Field Mapping:** SDOH screening tools (LOINC Panel 96777-8 — Accountable Health Communities screening), social history section, nutrition and dietetics notes, physical activity documentation.

| Severity Modifier | SDOH Criteria |
|------------------|--------------|
| No Escalation | Adequate diet documentation; moderate or higher physical activity; stable housing and food security; adequate health literacy and medication access |
| **+1 Tier Escalation Risk Flag** | ≥2 of the following: food insecurity (ICD-10: Z59.4), housing instability (Z59.0–Z59.1), low health literacy (Z55.x), sedentary lifestyle (Z72.3), poor dietary habits (documented by dietitian or PCP), inability to afford medications (Z87.898 or pharmacy claim gaps) |
| **Mandatory Clinical Review Flag** | Presence of documented food insecurity AND medication non-adherence (≥2 missed prescription fills within 6 months per pharmacy claims) regardless of |

> **SDOH and Critical-Complex Interaction Note:** When a patient already meets Tier 3 (Moderate Complications) classification and also triggers the SDOH Mandatory Clinical Review Flag, this combination must be flagged for Tier 4 (Advanced/Critical) clinical review. The SDOH domain does not independently assign Tier 4 but acts as a mandatory escalation prompt when superimposed on Tier 3 multisystem severity. [REVISION v1→v2: Added SDOH–Critical-Complex interaction note to specify how the SDOH modifier interacts with the new Tier 4; the existing SDOH table content is preserved verbatim and unchanged]

---

## Section 3: Tier Classification Decision Logic

[REVISION v1→v2: New section added to consolidate the four-tier classification rules in a single computable reference; this section did not exist in v1 and is added to provide a complete operational decision framework consistent with the expanded tier structure]

The following rules govern final tier assignment after all applicable domains have been evaluated:

**Step 1 — Domain Evaluation:** Evaluate all available domains (Domains 1–8) against their respective tier thresholds. Record the highest tier triggered by each active domain.

**Step 2 — Tier 4 Screening (Framework Rule 0a):** Before assigning any tier, apply the Advanced/Critical screening check:
- Does the patient meet **two or more concurrent Tier 3 domain criteria**? → Assign **Tier 4 (Advanced/Critical)**
- Does the patient have **eGFR <15 mL/min/1.73 m² or dialysis dependence**? → Assign **Tier 4 (Advanced/Critical)**
- Does the patient have **insulin dependence AND macrovascular disease across ≥2 vascular territories**? → Assign **Tier 4 (Advanced/Critical)**
- Does the patient have a **Mandatory Clinical Review SDOH Flag superimposed on Tier 3 classification**? → Flag for **Tier 4 (Advanced/Critical) clinical review** (human adjudication required; not an automated Tier 4 assignment)

**Step 3 — Standard Tier Assignment (Framework Rule 0):** If Tier 4 criteria are not met, assign the highest tier triggered by any single domain.

**Step 4 — Flagging and Output:** All Tier 4 assignments must generate:
- A mandatory specialist referral flag (endocrinology, nephrology, or cardiology as applicable)
- A care complexity alert for the treating team
- A data staleness check: if eGFR or HbA1c values are >90 days old at the time of Tier 4 assignment, a "Pending Confirmation" sub-status must be applied

---

## Appendix: Consolidated Tier Summary Table

[REVISION v1→v2: Tier summary table updated from four tiers to four tiers; Advanced/Critical row added across all dimensions; all pre-existing Tier 1–3 content preserved]

| Domain | Baseline T2D (Tier 1) | Mild Complications (Tier 2) | Moderate Complications (Tier 3) | Advanced/Critical (Tier 4) |
|--------|--------------|-------------------|-----------------|--------------------------|
| HbA1c | 6.5–<7.0% | 7.0–<8.0% | ≥8.0–<11.0% | ≥11.0% + ≥1 Tier 3/4 domain |
| FPG | 100–125 mg/dL | 126–154 mg/dL | >154 mg/dL | >154 mg/dL + ≥1 Tier 3/4 domain |
| PPG | 140–199 mg/dL | 200–299 mg/dL | ≥300 mg/dL | ≥300 mg/dL + ≥1 Tier 3/4 domain |
| BMI (Pediatric) | ≥85th %ile | ≥95th %ile | >99th %ile | >99th %ile + ≥1 Tier 3/4 domain |
| Comorbidities | None significant | HTN or dyslipidemia | CVD or HTN+dyslipidemia | Macrovascular ≥2 territories + insulin dependence |
| Cardiorenal | eGFR ≥60, no HF/ASCVD | eGFR 30–59 or NYHA I-II HF | eGFR 15–29 or ASCVD+HF or UACR >300 | eGFR <15 or dialysis |
| Beta-Cell Function | HOMA-IR <2.5, C-pep normal | HOMA-IR ≥2.5, C-pep reduced | C-pep <0.2 nmol/L or ≥2 OAD failure | C-pep <0.2 + insulin dependent + multisystem |
| SDOH | No escalation | +1 flag (≥2 risk factors) | Mandatory review flag | Tier 3 + Mandatory review flag (human adjudication) |

---

===

##
