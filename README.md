- 👋 Hi, I’m Manohar Chekka
- 👀 I’m Data Engineer
- 📫 Reach me at : manoharch0698@gmail.com


<!---
MC140/MC140 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->

Area
Issues to Check
Fix / Solution
COE Recommendation
Dataset / Source
• Unused columns/tables• Loading too many years• Wrong storage mode• Slow DirectQuery
• Remove unused columns• Filter rows early• Use Import where possible• For large DBs → Composite + Aggregations
• Keep only required grain & retention• Avoid pure DirectQuery unless justified
Power Query (M)
• Query folding broken• Heavy custom columns• Duplicate queries• Transformations happening row-by-row
• Maintain query folding (View Native Query)• Use Reference, not Duplicate• Remove columns at top• Move heavy logic to SQL if possible
• Build Staging → Final query layers• Keep M logic simple & foldable
Data Model
• Snowflake / M2M• Bidirectional relationships• High-cardinality keys• RLS on facts
• Convert to Star Schema• Set relationships single-direction• Use numeric surrogate keys• Move RLS to dimensions
• Star schema mandatory for performance• Avoid complex web of relationships
DAX / Measures
• Slow visuals in Performance Analyzer• Overuse of SUMX/FILTER• Heavy calculated columns• Too many nested CALCULATEs
• Replace iterators with simple SUM where possible• Use measure branching• Push static logic to columns/PQ• Simplify CALCULATE transitions
• Prefer reusable base measures• Keep DAX modular & simple
Visuals / UX
• Too many visuals per page• Many slicers• Custom visuals• Large tables/matrices
• Keep <12 visuals per page• Limit slicers (especially high-cardinality)• Turn off unnecessary interactions• Add Top N filters / drill-through
• Use drill-through, bookmarks• Prefer built-in visuals
Service / Capacity
• Slow refresh• Refresh overlap• Large datasets competing in P1• Too many heavy models in workspace
• Implement Incremental Refresh• Stagger refresh schedules• Keep only required models in live• Monitor with Metrics app
• Enforce dataset size guardrails• Tier 1 reports require COE performance sign-off


✔
Item
☐
Unused columns/tables removed
☐
Query folding confirmed
☐
Only required time-range loaded
☐
Star schema followed
☐
No unnecessary bidirectional relationships
☐
No heavy iterators in core measures
☐
High-cardinality slicers avoided
☐
Visuals < 12 per page
☐
Incremental refresh used if applicable
☐
Performance Analyzer checked & slow visuals identified
