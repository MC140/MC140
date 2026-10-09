- 👋 Hi, I’m Manohar Chekka
- 📫 Reach me at : manoharch0698@gmail.com


<!---
MC140/MC140 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->

I was thinking we could take a more metadata-driven approach instead of creating so many separate measures for formatting and calculations.

Basically, we can have one configuration table where we define all the metrics, their display formats, sort order, thresholds and conditional formatting rules, like whether higher or lower values are considered good.

We can still reuse our existing DAX measures for the actual calculations and use a common SWITCH logic to dynamically return Actual, Plan, Variance and Target based on the metric. Then, instead of creating separate measures for every metric’s colour or indicator, we can have centralized formatting logic that works across all metrics.

This way, we can build everything in a single Matrix visual, and whenever business adds a new metric, we mainly update the configuration and map the underlying measures rather than rebuilding the entire logic.

And this approach would also support slice and dice. Since the actual calculations still come from the underlying data model, business can filter by month, LOB, product or other available dimensions, and the values, variances and indicators will dynamically recalculate based on those selections.

I think this would be much cleaner, more scalable and easier to maintain, especially if we’re planning to extend the scorecard with more metrics later.