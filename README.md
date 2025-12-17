- 👋 Hi, I’m Manohar Chekka
- 👀 I’m Data Engineer
- 📫 Reach me at : manoharch0698@gmail.com


<!---
MC140/MC140 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
3) Watchouts (Common Causes of Bloat + Slowness)

Use this section for “symptom → likely cause → fix”.

Watchout A — PBIX grows fast / refresh slows over time
	•	Likely causes: Auto Date/Time enabled; too many date columns; unused columns.  ￼
	•	Fix: Disable Auto Date/Time; add DimDate; remove unused columns.

Watchout B — Slicers lag / visuals spin
	•	Likely causes: Wide flat tables; bi-directional filters; many-to-many without a proper bridge; high-cardinality columns.  ￼
	•	Fix: Star schema; single direction relationships; remove GUID/timestamps; redesign M2M.

Watchout C — Capacity “chokes” during peak hours
	•	Likely causes: Query storms from slicers; too many visuals on one page; heavy DirectQuery patterns.
	•	Fix: Query reduction (Apply); reduce visuals per page; use aggregation strategy / import where possible.

Watchout D — Numbers don’t match across reports
	•	Likely causes: Implicit measures; duplicated models per report; inconsistent business rules.
	•	Fix: Explicit measures only; golden dataset pattern; shared certified definitions.

Watchout E — RLS is slow or unpredictable
	•	Likely causes: RLS filters on facts; complex LOOKUPVALUE security logic.  ￼
	•	Fix: Security on dimensions; relationship-driven propagation.




Guardrails for Data Modelling

1) Model architecture and schema

G1. Use a Star Schema by default
	•	Facts = transactional tables (Sales, Calls, Payments, Events).
	•	Dimensions = descriptive lookup tables (Date, Customer, Product, Branch, Employee).
	•	Keep dimensions wide (attributes) and facts narrow (metrics + keys).

G2. One semantic model = one clear business scope
	•	If the model serves multiple unrelated use cases, split into multiple datasets/models.
	•	Avoid “dump everything” datasets.

G3. Grain must be explicitly stated
	•	Every fact table must have a documented grain:
“1 row = 1 transaction / 1 call / 1 daily-branch-product summary”
	•	Measures and relationships must align to grain.

⸻

2) Relationship rules (this is where most issues come from)

G4. Relationships must be predictable
	•	Prefer 1-to-many from Dimension → Fact.
	•	Single direction filter (Dim → Fact) by default.
	•	Avoid bi-directional unless you can explain exactly why it’s needed and it’s tested.

G5. No many-to-many unless approved
	•	Many-to-many is allowed only with a documented reason and testing results.
	•	If you need bridging, use a proper bridge table with unique keys.

G6. One active relationship only
	•	If multiple date fields exist (CreatedDate, ClosedDate, PaidDate), keep:
	•	One active relationship to the Date table
	•	Others as inactive + USERELATIONSHIP() in measures.

G7. Keys must be clean
	•	All dimension keys must be unique (no duplicates).
	•	Fact foreign keys should be complete (track nulls/unknowns intentionally).
	•	Use a standard “Unknown” member in dimensions when needed.

⸻

3) Columns, data types, and VertiPaq hygiene (performance)

G8. Remove unused columns early
	•	If it’s not used in visuals/filters/measures/relationships → remove it.
	•	Avoid bringing long description fields, comments, JSON blobs, etc.

G9. Use correct data types
	•	Numbers as whole/decimal (not text).
	•	Dates as Date (not text).
	•	Keys ideally as integers where possible (better compression than long strings).

G10. Avoid high-cardinality text columns
	•	Don’t expose columns like TransactionID, GUIDs, or free-text fields for slicing.
	•	Keep GUID columns hidden unless required for relationships.

G11. Hide technical columns
	•	Hide surrogate keys, system fields, audit columns from report view.
	•	Expose only business-friendly fields.

⸻

4) Measures and calculation rules (semantic layer discipline)

G12. All business logic should be measures
	•	Avoid calculated columns unless necessary.
	•	Avoid doing business logic in visuals.

G13. Explicit measures only
	•	Discourage implicit measures (auto-sum) for published models.
	•	Measures should be named, formatted, and grouped.

G14. Standard measure naming & folders
	•	Example:
	•	Sales Amount
	•	Sales Amount LY
	•	Sales Amount YTD
	•	Place measures in a dedicated “Measures” table and use display folders.

G15. Avoid expensive DAX patterns
	•	Minimize iterators (SUMX, FILTER over big tables) unless required.
	•	Prefer pre-aggregation upstream for heavy logic.

⸻

5) Aggregations and shaping location (where logic should live)

G16. Aggregations belong upstream by default
	•	If users consume summary-level reporting, build summary facts in AZ (daily/weekly/monthly).
	•	Don’t depend on Power BI to compute large-scale rollups repeatedly.

G17. Duplicate summary tables are not allowed without justification
	•	If multiple teams need different rollups, standardize shared rollups upstream.

G18. Power Query is not an ETL engine
	•	Power Query should do light shaping only:
	•	Rename columns, set types, small filters
	•	Heavy joins, big transformations, and large aggregations → upstream.

⸻

6) Date/time modelling guardrails

G19. One enterprise Date table
	•	Single Date table used across the model.
	•	Mark it as Date table.
	•	Auto Date/Time must be disabled (recommended).

G20. Date relationships
	•	Prefer Date type relationship keys.
	•	Use inactive relationships + measures for multiple date roles.

⸻

7) Security and governance guardrails (for publishable models)

G21. RLS must be model-driven, not report hacks
	•	If RLS is required, document:
	•	Roles
	•	Mapping tables (user → branch/LOB/etc.)
	•	AD group strategy (if applicable)

G22. Certified/publishable dataset checklist
	•	Model has documented scope + owner + refresh SLA
	•	Measures documented (at least top KPIs)
	•	Column-level sensitivity handled as per policy (masking/classification)

⸻

8) Refresh and reliability guardrails

G23. Refresh must complete within agreed window
	•	If refresh is slow:
	•	Reduce columns
	•	Reduce row count via AZ aggregation
	•	Consider incremental refresh where applicable

G24. Incremental refresh when data is large
	•	Use partitions for large facts where it fits (e.g., date-driven fact tables).
	•	Avoid full refresh for huge historical datasets.

