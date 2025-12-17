- 👋 Hi, I’m Manohar Chekka
- 👀 I’m Data Engineer
- 📫 Reach me at : manoharch0698@gmail.com


<!---
MC140/MC140 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->

To expand the "Guardrails" section, we shouldn't just repeat what is in the meeting description. Instead, we need to define the concrete technical standards (the "laws" of the model) that ensure the system stays fast and reliable.
Here is the expanded Section 1: Data Modelling Guardrails, developed as a set of actionable rules for your team.
1. Data Modelling Guardrails
These guardrails are the mandatory technical standards for all Power BI developers. They are designed to prevent "model bloat" and ensure reports remain responsive as data volume grows.
Rule 1: The "Upstream First" Transformation Law
• The Guardrail: All heavy data transformations (joins, complex grouping, and business logic) must occur in the Availability Zones (AZs) or SQL layer.
• Why it matters: Moving logic upstream means the Power BI engine (VertiPaq) only has to deal with "clean" data. This prevents long refresh times and reduces the risk of the Power BI Service timing out.
• Developer Action: If you find yourself writing 50+ lines of Power Query (M) code to clean a table, that logic should be moved to a SQL View in the AZ.
Rule 2: Star Schema Enforcement
• The Guardrail: Every model must be organized into Fact tables (the numbers/metrics) and Dimension tables (the descriptive labels like Product, Date, or Region).
• The "No-Flat-Table" Policy: Avoid importing one giant, wide table. Large "flat" tables consume significantly more memory and lead to slow slicer performance.
• Relationship Cardinality: Aim for 1-to-Many relationships. Avoid "Many-to-Many" or "Bi-directional" filters unless explicitly approved by the CoE, as they create "ambiguous paths" that lead to incorrect data.
Rule 3: Columnar Hygiene (Minimize Width)
• The Guardrail: Only import columns that are actively used in a visual or a measure.
• The "High-Cardinality" Ban: Avoid importing columns with unique values (like Transaction IDs, GUIDs, or high-precision timestamps) unless absolutely necessary.
• Why it matters: Power BI is a columnar database. One unique ID column can take up more space than 10 columns of category data. Removing these can often reduce file size by 50% or more.
Rule 4: Measure-Only Aggregations
• The Guardrail: Do not use "Implicit Measures" (dragging a numeric field directly into a visual). All calculations must be written as Explicit DAX Measures.
• Why it matters: This ensures consistency across the organization. It prevents two different users from calculating "Total Profit" in two different ways. It also allows for easier auditing and performance tuning using the Performance Analyzer.
Rule 5: Query Reduction Guardrails
• The Guardrail: For large-scale reports, developers must enable "Apply" buttons for slicers and filters.
• Why it matters: This prevents Power BI from sending a new query to the database every time a user clicks a single checkbox, which can "choke" the capacity during peak usage hours.
Would you like me to now expand the 2nd component, focusing on the "10-Second Rule" and the Assessment Rubric?

Rule 6: Data Type Precision & Memory Optimization
• The Guardrail: Avoid the "Decimal" (Floating Point) data type for currency or fixed numbers. Use "Fixed Decimal Number" (Currency) instead.
• The "Why": Floating point decimals can have many unique trailing digits (e.g., 10.000004). Fixed decimals limit this to 4 places, which allows the Power BI engine to compress the data much more efficiently, often saving significant memory.
• Developer Action: Change all price and cost columns to Fixed Decimal in the Power Query editor.
Rule 7: Separation of Concerns (Tiered Modelling)
• The Guardrail: Separate the Data Layer from the Report Layer by using "Live Connections."
• The "Why": Avoid building reports directly inside the .pbix file that contains the data model. By publishing the Semantic Model separately, multiple reports can connect to that one "Golden Dataset."
• The Benefit: This prevents "version sprawl" where 10 different reports have 10 slightly different versions of the same calculation.
Rule 8: DAX Complexity Limits
• The Guardrail: Avoid using the FILTER() function over an entire Fact table. Use CALCULATE() with simple filter predicates where possible.
• The "Why": Iterating over millions of rows in DAX is the #1 cause of "spinning wheels" for the end-user.
• Developer Action: If a calculation is too slow, move the logic further upstream into a calculated column in the AZ/SQL layer (referencing Rule 1).
Rule 9: Metadata & "Self-Documenting" Models
• The Guardrail: Every measure and complex column must have a description filled out in the "Description" property field.
• The "Why": When a user hovers over a field in the report, they see exactly what that metric means. This reduces the number of "How is this calculated?" emails the CoE receives.
Rule 10: Security & RLS (Row-Level Security) Efficiency
• The Guardrail: RLS roles must be applied to Dimension tables, never directly to Fact tables.
• The "Why": Filtering a small Dimension table (e.g., 100 regions) and letting that filter flow down to the Fact table (100 million rows) is much faster than asking the engine to check every single row in the Fact table for security permissions.
