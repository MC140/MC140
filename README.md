- 👋 Hi, I’m Manohar Chekka
- 👀 I’m Data Engineer
- 📫 Reach me at : manoharch0698@gmail.com


<!---
MC140/MC140 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
# Power BI Center of Excellence – Semantic Model Assessment

This checklist is used by the Power BI Center of Excellence (CoE) to assess semantic models
before enabling refresh, performance tuning, or production deployment.

The objective is to ensure models are scalable, performant, secure, and aligned with
enterprise Power BI and Synapse-first guardrails.

---

## Status Legend

| Status | Meaning |
|------|--------|
| Green | Meets CoE standard; no action required |
| Yellow | Acceptable but improvement recommended |
| Red | Guardrail violation; must be fixed before approval |

---

## Semantic Model Assessment Checklist

| Assessment Area | What Is Checked | Status (Green / Yellow / Red) | CoE Comments / Required Action |
|----------------|----------------|-------------------------------|--------------------------------|
| Upstream-first transformations | Heavy joins, aggregations, and business logic handled in Synapse (AZ), not in Power BI |  |  |
| Approved data source only | Dataset uses only AZ Synapse as the data source (no Excel, SharePoint, CSV, or local files) |  |  |
| Star schema design | Model follows fact and dimension structure with single-direction relationships |  |  |
| Fact table grain defined | Fact table has one clear and documented grain |  |  |
| Mixed grain avoided | Fact tables do not contain columns at different levels of granularity |  |  |
| Many-to-many relationships controlled | No accidental many-to-many relationships; bridge tables used intentionally |  |  |
| Relationship cardinality correct | Relationships use correct cardinality and active relationships only |  |  |
| Bi-directional filters avoided | Cross-filter direction is single unless explicitly justified |  |  |
| Auto Date/Time disabled | Auto date/time feature is disabled in Power BI |  |  |
| Single enterprise Date table | One shared Date table is used and marked as Date table |  |  |
| Calculated columns minimized | Calculated columns used only when unavoidable |  |  |
| Measures preferred over columns | Business logic implemented using DAX measures, not calculated columns |  |  |
| Iterator functions controlled | Limited use of SUMX, FILTER, ADDCOLUMNS on large fact tables |  |  |
| FILTER usage optimized | FILTER is not used where CALCULATE filters would suffice |  |  |
| ATTR and COUNTD avoided | ATTR and COUNTD are not used unless strictly required |  |  |
| 10-second performance rule | Key report pages respond within approximately 10 seconds |  |  |
| Visual density optimized | Dashboards do not contain excessive visuals or large tables |  |  |
| Default landing state optimized | Report landing page does not load full dataset by default |  |  |
| Column pruning applied | Only required columns are loaded into the semantic model |  |  |
| Correct data types | Columns use appropriate data types (dates, numerics, IDs) |  |  |
| High-cardinality columns controlled | GUIDs and long text columns are not stored in fact tables |  |  |
| Incremental refresh strategy | Incremental refresh is configured where historical data exists |  |  |
| Refresh duration acceptable | Dataset refresh completes consistently without timeouts |  |  |
| Query folding preserved | Power Query steps fold back to Synapse |  |  |
| Dataset size controlled | Dataset memory size is within acceptable enterprise limits |  |  |
| Row-Level Security (RLS) implemented | RLS uses AD groups and is tested in Power BI Service |  |  |
| RLS performance validated | RLS does not cause significant performance degradation |  |  |
| Model documentation present | Measures, tables, and columns include descriptions |  |  |
| Naming standards followed | Business-friendly and consistent naming conventions applied |  |  |
| Measures organization | Measures are grouped in dedicated tables or folders |  |  |
| Deployment pipeline readiness | Dataset contains no hardcoded environment-specific values |  |  |
| Gateway and connectivity validated | Dataset connects successfully through approved gateway |  |  |
| Ownership and support defined | Dataset owner and support contact are documented |  |  |

---

## Assessment Summary

| Metric | Value |
|------|------|
| Total Rows Assessed |  |
| Green |  |
| Yellow |  |
| Red |  |

---

## CoE Decision

| Decision | Notes |
|--------|------|
| Approve |  |
| Approve with Fixes |  |
| Block |  |

---

## Notes and Expectations

- All Red items must be resolved before production refresh is enabled.
- Yellow items should be addressed in the next development iteration.
- This assessment aligns with enterprise Power BI best practices and Synapse-first architecture.
- Power BI is not an ETL tool; all heavy data processing must occur upstream.


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

