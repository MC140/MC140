- 👋 Hi, I’m Manohar Chekka
- 👀 I’m Data Engineer
- 📫 Reach me at : manoharch0698@gmail.com


<!---
MC140/MC140 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->

## Power BI Performance Checklist (AZ Synapse + Large Data)

| **Area** | **What to Check** | **What to Fix** |
|---------|--------------------|------------------|
| **Source (AZ Synapse)** | - Using raw tables?<br>- Transformations in Power BI? | - Do **all transformations in Synapse** (views).<br>- Pre-aggregate, clean, filter data in source. |
| **Power Query** | - Too many steps?<br>- Query folding broken? | - Keep only: remove columns, filter rows, rename.<br>- Ensure **query folding** works. |
| **Data Model** | - Snowflake or many-to-many?<br>- Bi-directional relationships? | - Use **Star Schema**.<br>- Single-direction relationships.<br>- Use numeric surrogate keys. |
| **DAX** | - SUMX/FILTER over big fact table?<br>- Heavy calculated columns? | - Move heavy logic to Synapse.<br>- Use simple SUM + measure branching.<br>- Avoid iterators on big tables. |
| **Visuals** | - Too many visuals?<br>- Too many slicers?<br>- Large table visuals? | - Keep **<12 visuals** per page.<br>- Avoid high-cardinality slicers.<br>- Use drill-through instead of giant tables. |
| **Service** | - Slow refresh?<br>- Capacity (P1) spikes? | - Use **Incremental Refresh**.<br>- Stagger refresh times.<br>- Use Import Agg tables + DirectQuery detail. |


## Quick COE Checklist (Tick Before Escalating)

- [ ] All transformations done in **AZ Synapse**, not Power BI  
- [ ] Using Synapse **views** instead of raw tables  
- [ ] Power Query steps minimal & **query folding** works  
- [ ] Model follows **Star Schema**  
- [ ] No unnecessary bi-directional relationships  
- [ ] No heavy SUMX/FILTER on large fact tables  
- [ ] Visuals kept under **12 per page**  
- [ ] Incremental Refresh enabled  