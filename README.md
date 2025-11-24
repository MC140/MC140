- 👋 Hi, I’m Manohar Chekka
- 👀 I’m Data Engineer
- 📫 Reach me at : manoharch0698@gmail.com


<!---
MC140/MC140 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->

COE Standard: High-Performance Architecture for Large Datasets
1. Executive Summary
As data volumes in Analytics Zone (AZ) Synapse databases grow into hundreds of millions of rows, standard Import Mode is no longer viable due to P1 Capacity memory limits. Conversely, pure DirectQuery often results in poor user experience due to latency.
The Solution: Implement a Composite Model with User-Defined Aggregations. This architecture allows reports to load instantly (sub-second) for high-level KPIs while preserving the ability to drill down into granular transaction details without overloading the capacity.
2. The Architecture Concept
Think of this architecture as a "Traffic Control" system for data queries:
95% of Queries (High Level): Served instantly from a small, in-memory cache (Import).
5% of Queries (Detail Level): Served directly from Synapse via SQL (DirectQuery) only when necessary.
3. Implementation Guide (Step-by-Step)
Phase 1: Data Preparation (Synapse/ETL)
Do not attempt to perform heavy aggregations inside Power BI. Pre-process the data in the ETL layer.
Create the Detail Table: Ensure the raw transactional table exists in Synapse (e.g., Fact_Sales_Detail).
Create the Aggregation Table: Create a physical summary table in Synapse (e.g., Agg_Sales_Summary).
Group By: Key dimensions used in charts (e.g., Date, Store, ProductCategory).
Measures: Sum key metrics (e.g., Sum_Revenue, Sum_Units).
Target Size: Aim for < 5 Million rows for optimal performance.
Phase 2: Power BI Modeling
The secret lies in mixing Storage Modes.
Connect to Detail: Connect to Fact_Sales_Detail using DirectQuery mode.
Connect to Summary: Connect to Agg_Sales_Summary using Import mode.
Configure Dimensions: Ensure shared dimensions (Date, Geography, Product) are set to Dual storage mode.
Why? Dual mode allows the dimension to act as "Import" when talking to the Summary table (Fast) and "DirectQuery" when talking to the Detail table (Compatible).
Phase 3: The "Magic" Configuration
Tell Power BI how to link the two tables.
Go to the Model View in Power BI Desktop.
Right-click the Aggregation Table and select Manage Aggregations.
Map the columns:
Map the Summary Measures (e.g., Sum_Revenue) to the Detail Table Columns (Fact_Sales_Detail[Revenue]).
Map the Group By Columns to the respective Dimension Tables.
Apply and Hide the Aggregation table.
4. Developer Experience vs. User Experience
Perspective
What Happens
The Developer
Builds visuals using only the Detail Table fields. You act as if the Aggregation table doesn't exist.
The User
Interactions (Slicing/Filtering) are instant. They do not know two tables exist.
The Engine
Automatically intercepts the query. If the Aggregation table has the answer, it uses the cache. If not, it queries Synapse.
5. COE Best Practices & Guardrails
✅ DO: Aggregate by high-level dimensions (Date, Region, Category).
✅ DO: Use Dual Mode for all dimension tables connected to the Fact.
⛔ DON'T: Aggregate by high-cardinality columns (Transaction ID, User ID, Zip Code). This defeats the purpose of the cache.
⛔ DON'T: Use Power BI Calculated Columns on the DirectQuery table. Push this logic to Synapse/Databricks.
6. Decision Matrix
When should a team use this pattern?
Dataset Size
Storage Mode Recommendation
Small (< 1GB)
Import Mode (Simplest, Fastest).
Medium (< 10GB)
Import Mode (If P1 memory permits).
Large (> 10GB)
Composite Model (This Standard).
Real-Time Req
DirectQuery (Accepting performance trade-offs).
