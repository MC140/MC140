- 👋 Hi, I’m Manohar Chekka
- 👀 I’m Data Engineer
- 📫 Reach me at : manoharch0698@gmail.com


<!---
MC140/MC140 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
# Power BI Copilot: Developer & Business Team Test Plan

**Date:** November 20, 2025
**Subject:** Copilot in Power BI Desktop - Developer Experience Testing
**To:** Rishabh, Dhaval, Jacques, Edward

---

## 1. Overview
We are launching Copilot in Power BI to a pilot team. This document outlines a list of test cases to validate the tool's effectiveness in accelerating development, generating DAX, and creating visualizations.

**Objective:** Validate if Copilot acts as an effective "pair programmer" for our data workflow.

---

## 2. Test Cases

### Phase 1: DAX Query & Measure Generation
*Goal: Test syntax accuracy and logic explanation capabilities.*

| Test ID | Feature | Action to Perform | Success Criteria | Pass/Fail |
| :--- | :--- | :--- | :--- | :--- |
| **DAX-01** | Basic Math | Ask Copilot: *"Create a measure to calculate Total Sales from the Sales table."* | Valid `SUM` syntax generated. | |
| **DAX-02** | Time Intel | Ask Copilot: *"Create a measure for Year-over-Year Sales growth."* | Valid `CALCULATE` / `SAMEPERIODLASTYEAR` logic generated. | |
| **DAX-03** | Filtering | Ask Copilot: *"Create a measure for Total Sales where Category is 'Accessories'."* | Correct filter context applied. | |
| **DAX-04** | Explain | Highlight a complex measure and ask: *"Explain this code."* | Clear English explanation provided. | |
| **DAX-05** | Optimize | Paste a long query and ask: *"Optimize this DAX."* | Suggestions for formatting or efficiency provided. | |

### Phase 2: Report Creation & Visualization
*Goal: Test ability to interpret data schema and suggest UI.*

| Test ID | Feature | Action to Perform | Success Criteria | Pass/Fail |
| :--- | :--- | :--- | :--- | :--- |
| **VIS-01** | Suggestion | Ask Copilot: *"Suggest a visual to show Sales trends by Region."* | Line/Area chart created with correct axes. | |
| **VIS-02** | Page Build | Ask Copilot: *"Create a page analyzing Customer Churn."* | Layout with multiple relevant visuals created. | |
| **VIS-03** | Narrative | Add Narrative visual. Ask: *"Summarize key insights."* | Accurate text summary of data trends generated. | |

### Phase 3: Semantic Model & "Hallucinations"
*Goal: Test metadata handling and error boundaries.*

| Test ID | Feature | Action to Perform | Success Criteria | Pass/Fail |
| :--- | :--- | :--- | :--- | :--- |
| **MOD-01** | Synonyms | Ask Copilot: *"Generate synonyms for 'Revenue'."* | Relevant terms (Income, Sales) suggested. | |
| **QC-01** | Ambiguity | Ask vague question: *"Show me performance."* | Copilot asks clarifying questions (e.g., "Sales or Employee performance?"). | |
| **QC-02** | Errors | Ask for a calculation on a non-existent column. | Copilot states it cannot find the column (No fake formulas). | |

---

## 3. Feedback Notes
*Please record any bugs or specific "wins" below.*

* **Tester:** ___________________
* **Notes:**
    * [ ] DAX code required manual fixing?
    * [ ] Visuals were relevant to business context?
