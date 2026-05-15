# Literature Evidence

This folder contains the clinical literature evidence used by the agentic AI system during **Phase 1 — Framework Generation**.

The sources included here were identified using the **Tavily Deep Research Tool** and were used by the research agents to generate and consolidate the Type 2 Diabetes severity assessment framework.

## Purpose of this folder

The purpose of the `Literature_Evidence` folder is to provide transparency around the external clinical sources used during framework generation. These sources informed the identification of clinically relevant severity markers, including glycaemic control, comorbidities, cardiorenal risk, treatment escalation, and diabetes-related complications.

## Sources identified by Tavily Deep Research

### 1. IDF Clinical Practice Recommendations for Managing Type 2 Diabetes in Primary Care

Aschner, P., Adler, A., Bailey, C., Colagiuri, S., Day, C., Jose Gagliardino, J., Leiter, L. A., Nutrition, C., Han Cho, N., & Sobngwi, E. (2016). *IDF Clinical Practice Recommendations for managing Type 2 Diabetes in Primary Care-2017 Chair: Core Contributors*.  
Available at: [www.idf.org/managing-type2-diabetes](https://www.idf.org/managing-type2-diabetes)

### 2. Guideline Directed Management of Diabetes Comorbidities

*Guideline Directed Management of Diabetes Comorbidities*. (2024). *ADA Standards of Medical Care in Diabetes - Novo Nordisk*.  
DOI: [https://doi.org/10.1016/j.metabol.2024.155931](https://doi.org/10.1016/j.metabol.2024.155931)

### 3. ISPAD Clinical Practice Consensus Guidelines 2024: Type 2 Diabetes in Children and Adolescents

Shah, A. S., Barrientos-Pérez, M., Chang, N., Fu, J. F., Hannon, T. S., Kelsey, M., Peña, A. S., Pinhas-Hamiel, O., Urakami, T., Wicklow, B., Wong, J., & Mahmud, F. H. (2024). *ISPAD Clinical Practice Consensus Guidelines 2024: Type 2 Diabetes in Children and Adolescents*. *Hormone Research in Pædiatrics, 97*(6), 555.  
DOI: [https://doi.org/10.1159/000543033](https://doi.org/10.1159/000543033)

## Notes

These sources were not manually selected before the framework-generation process. Instead, they were retrieved by the Tavily-enabled agentic research workflow and then used as evidence by the LLM research agents and consolidator.

The resulting framework derived from this evidence is stored in:

- `Consolidated_Framework.md` — final consolidated Type 2 Diabetes severity framework.
- `Framework_Generation_Consolidator_Output.md` — raw CrewAI generation output from the framework-generation process.
