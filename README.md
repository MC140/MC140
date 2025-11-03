- 👋 Hi, I’m Manohar Chekka
- 👀 I’m Data Engineer
- 📫 Reach me at : manoharch0698@gmail.com

JobCategory =
DATATABLE(
  "JobCategory", STRING,
  { {"Branch"}, {"District"}, {"Region"}, {"National"}, {"PBA"} }
)

FCR by JobCategory :=
VAR jc = SELECTEDVALUE(JobCategory[JobCategory])
VAR metricFilter = KEEPFILTERS(DimMetrics[MetricID] = 755)
RETURN
SWITCH(
    jc,
    "Branch",   CALCULATE([FCR %], metricFilter, REMOVEFILTERS(DimMetrics[Pillar_DVP], DimMetrics[Pillar_SVP], DimMetrics[Pillar_PBA])),
    "District", CALCULATE([FCR %], metricFilter, REMOVEFILTERS(DimMetrics[Pillar_BM], DimMetrics[Pillar_SVP], DimMetrics[Pillar_PBA])),
    "Region",   CALCULATE([FCR %], metricFilter, REMOVEFILTERS(DimMetrics[Pillar_BM], DimMetrics[Pillar_DVP], DimMetrics[Pillar_PBA])),
    "National", CALCULATE([FCR %], metricFilter, REMOVEFILTERS(DimMetrics[Pillar_BM], DimMetrics[Pillar_DVP], DimMetrics[Pillar_SVP], DimMetrics[Pillar_PBA])),
    "PBA",      CALCULATE([FCR %], metricFilter, REMOVEFILTERS(DimMetrics[Pillar_BM], DimMetrics[Pillar_DVP], DimMetrics[Pillar_SVP]))
)
<!---
MC140/MC140 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
