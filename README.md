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