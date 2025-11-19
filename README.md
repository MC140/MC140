- 👋 Hi, I’m Manohar Chekka
- 👀 I’m Data Engineer
- 📫 Reach me at : manoharch0698@gmail.com


<!---
MC140/MC140 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
# Power BI Copilot: Developer & Business Team Test Plan

**Date:** November 20, 2025  
**Subject:** Copilot in Power BI Desktop – Developer & Business Experience Testing  
**To:** Rishabh, Dhaval, Jacques, Edward  

---

## 1. Overview
We are launching Copilot in Power BI to a pilot team. This document outlines a comprehensive list of test cases to validate the tool’s effectiveness in accelerating development, generating DAX, improving insights, and creating visualizations.

**Objective:** Validate if Copilot acts as an effective “pair programmer” and “insights assistant” within our Power BI workflow.

---

## 2. Test Cases

---

## A. Developer Test Cases (Power BI Desktop)

### 1. Generate DAX Measures
- Ask Copilot: “Create a YoY Sales measure.”
- **Expected:** Valid DAX is generated.

### 2. Modify Existing DAX Measure
- Ask: “Optimize this measure for performance.”
- **Expected:** Improved version returned.

### 3. Explain DAX Logic
- Provide a complex measure and ask for explanation.
- **Expected:** Clear description in plain English.

### 4. Generate New Calculated Columns
- Ask Copilot to create a “Week of Month” column.
- **Expected:** Correct DAX output.

### 5. Suggest Data Model Improvements
- Ask: “Review and recommend improvements to my model.”
- **Expected:** Star schema guidance, naming suggestions, relationship fixes.

### 6. Convert Business Requirements to DAX
- Provide requirement → Ask Copilot to generate measure.
- **Expected:** Accurate DAX.

### 7. Create Narrative Summaries
- Ask: “Summarize insights from this page.”
- **Expected:** Clear, insight-driven text.

### 8. Create Visuals Using Copilot
- Ask Copilot to create a visual (e.g., Sales by Region).
- **Expected:** Visual auto-created on canvas.

### 9. Visual Recommendation
- Ask: “Best visual for trends vs categories?”
- **Expected:** Correct visual type recommendation.

### 10. Troubleshoot Model Issues
- Ask: “Why is my date relationship not working?”
- **Expected:** Root cause identified.

---

## B. Business User Test Cases (Power BI Service)

### 1. Natural Language Q&A
- “What is the trend of monthly revenue?”
- **Expected:** Correct insights + visual suggestion.

### 2. Rewrite Commentary
- Provide text to Copilot to rewrite professionally.
- **Expected:** Clean business summary.

### 3. Page Insight Generation
- Ask for insights from a report page.
- **Expected:** Accurate business interpretation.

### 4. Identify Anomalies
- Ask: “Any unusual spikes?”
- **Expected:** Copilot identifies outliers.

### 5. Full Report Summary
- “Summarize the entire dashboard.”
- **Expected:** Executive summary.

### 6. Stakeholder Questions
- “What questions should I ask business based on this data?”
- **Expected:** Intelligent suggestions.

### 7. PPT Narrative
- Ask Copilot to generate text for PowerPoint.
- **Expected:** Concise content.

### 8. Time Comparison
- “Compare this quarter vs last quarter.”
- **Expected:** Accurate time comparison.

---

## C. Governance & Security Test Cases

### 1. RLS Enforcement
- Validate that restricted users cannot access secured data.
- **Expected:** No RLS bypass.

### 2. Sensitivity Label Enforcement
- Apply Confidential label → test Copilot outputs.
- **Expected:** No sensitive data leakage.

### 3. Access Boundary Testing
- User without Build permission tries to create model-level assets.
- **Expected:** Copilot blocks action.

### 4. Audit Logging
- Perform Copilot actions → Check audit logs.
- **Expected:** All actions logged.

### 5. Data Leakage Check
- Ask about data not in the model.
- **Expected:** Copilot refuses.

---

## D. Performance & Usability Tests

### 1. Response Speed
- Measure average response time.
- **Expected:** 5–7 seconds or less.

### 2. Handling Poor Data Quality
- Use dataset with nulls/missing values.
- **Expected:** Copilot acknowledges gaps.

### 3. Hallucination Check
- Ask ambiguous questions.
- **Expected:** Clarifying questions instead of hallucinations.

---

## E. End-to-End Scenario Tests

### 1. Build a Mini Report End-to-End
- Use Copilot to generate DAX, visuals, and summary.
- **Expected:** Smooth end-to-end workflow.

### 2. Requirement → Model → Measures → Visuals → Narrative
- Provide business requirement → Use Copilot for each step.
- **Expected:** Coherent output across stages.

---

## F. Recommended Excel Columns (If Tracking Results)

| Test Case ID | Category | Scenario | Steps | Expected Result | Actual Result | Pass/Fail | Comments |
|--------------|----------|----------|--------|----------------|----------------|------------|----------|

---

## G. Versioning
**Pilot Version:** v1.0  
**Owner:** Rahona Power BI Center of Excellence  
**Last Updated:** November 20, 2025