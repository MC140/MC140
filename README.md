- 👋 Hi, I’m Manohar Chekka
- 👀 I’m Data Engineer
- 📫 Reach me at : manoharch0698@gmail.com


<!---
MC140/MC140 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->

Assessment Area (Best Practice / Guardrail / Watchout)
What to check (signals)
Overall Status (🟢/🟡/🔴)
CoE Comments / Developer Actions
Upstream-first transformations (Guardrail)
Power Query is only light shaping; heavy joins/grouping/business rules done in Synapse (AZ)
Move complex shaping to Synapse views/tables; keep M minimal.
Only approved datasource (Guardrail)
Uses only AZ Synapse (no Excel/SharePoint/local files mixed)
Remove non-approved sources or request exception via CoE process.
Star schema (Best Practice)
Facts + dimensions; dims filter facts one-direction; minimal snowflake
Redesign model to star; avoid bi-directional unless required & justified.
Fact table grain is defined (Guardrail)
One clear grain (per txn/day/customer etc.); no mixed grain columns
Split facts or create bridge tables; document grain.
Avoid many-to-many ambiguity (Watchout)
No accidental M:M; bridge tables used intentionally
Fix relationships; introduce bridge and enforce unique keys in dims.
Relationship quality (Guardrail)
Correct keys, correct cardinality, active relationships, minimal inactive
Clean keys upstream; remove inactive unless used by USERELATIONSHIP.
Disable Auto date/time (Guardrail)
Auto date/time OFF (prevents hidden date tables & bloat)
Turn off and use a proper Date dimension.
Single shared Date table (Best Practice)
One Date table marked as date table; used across model
Implement enterprise Date table; mark as Date Table.
Calculated columns used only when needed (Guardrail)
No heavy calculated columns for row-by-row logic that can be upstream
Push column logic to Synapse; keep only essential columns in model.
Measures over calculated columns (Best Practice)
Business logic mainly as measures; columns not used for aggregation logic
Convert to measures where possible.
Avoid heavy iterator patterns (Watchout)
Too much SUMX/FILTER/ADDColumns over large tables
Replace with model redesign, pre-aggregation, or simpler measures.
Avoid using FILTER in measures unnecessarily (Watchout)
FILTER used where simple CALCULATE filters could work
Use CALCULATE([Measure], Dim[Col]="X") style filters; avoid scanning big fact tables.
Measure performance meets “10-sec rule” (Guardrail)
Key pages respond under ~10 seconds (esp. common filters)
Optimize measures, reduce visuals, aggregate upstream, add proper dims.
Visual density & design (Best Practice)
Not too many visuals per page; avoids huge tables; limited custom visuals
Reduce visuals; split pages; prefer summaries with drillthrough.
Avoid “Show all data” by default (Guardrail)
Landing page isn’t rendering millions of rows; sensible default filters
Add default slicers; use bookmarks; limit table visuals.
Column pruning (Best Practice)
Only required columns loaded; unused columns removed
Remove unused columns; keep model narrow.
Data types are correct (Guardrail)
IDs are text/whole number properly; dates as date; no mixed types
Fix types upstream; avoid “Any” type in Power Query.
High-cardinality columns controlled (Watchout)
Avoid loading GUIDs/long text into fact; minimize unique strings
Remove/encode; move to dimension; avoid “random IDs” in fact.
No unnecessary bi-directional filters (Guardrail)
Cross-filter mostly single direction
Change to single; use measures/bridge tables for special cases.
RLS design (Guardrail)
RLS defined clearly; uses AD groups; tested with roles
Implement RLS via dimension table + AD group mapping; test in Service.
Refresh strategy (Best Practice)
Incremental refresh where needed; partitions make sense
Enable incremental refresh for large history; keep only required history.
Refresh duration acceptable (Guardrail)
Refresh does not routinely time out; stable refresh history
Reduce data volume; aggregate upstream; review query folding.
Query folding preserved (Guardrail)
Synapse queries fold; minimal transformations that break folding
Move transforms to SQL views; keep M steps folding-friendly.
Dataset size controlled (Guardrail)
Dataset not bloated (e.g., very large PBIX, huge model memory)
Reduce columns/rows, aggregate, remove high-cardinality fields.
Model documentation (Best Practice)
Measure naming, descriptions, folders, display tables, lineage clear
Apply naming standards; add descriptions; separate “Measures” table.
Naming standards (Best Practice)
Clear names for tables/columns/measures; no “Table1/Column2”
Rename and standardize; align to business terms.
Deployment pipeline readiness (Guardrail)
No hardcoded workspace-specific parameters; connections supported
Parameterize where needed; ensure gateway mapping is correct.
Owner/support model (Watchout)
Support contacts, refresh owners, and change process defined
Provide owner list; set expectations for changes & refresh failures.

