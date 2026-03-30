- 👋 Hi, I’m Manohar Chekka
- 👀 I’m Data Engineer
- 📫 Reach me at : manoharch0698@gmail.com


<!---
MC140/MC140 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
Build me a generalized Python script for a Power BI Python Visual that mimics a Tableau-style connected scatter plot for ANY dataset.
Core behavior:
	∙	Works with any categorical dimension (companies, products, regions, people etc.)
	∙	Works with any two numeric measures on X and Y axes
	∙	Works with any two time periods (not hardcoded to 2020/2025 — auto-detects the earliest and latest period in the data)
	∙	Draws a dashed arrow/line connecting earlier period → later period for each category
	∙	Earlier period point = triangle marker (△)
	∙	Later period point = filled circle marker (●)
	∙	Each category gets a unique color applied to both its points and connecting line
	∙	Category name label displayed next to the later period point
	∙	Arrow direction shows movement from old → new
	∙	Legend showing △ = earlier period, ● = later period with actual year/period values
	∙	Clean white background, minimal gridlines, no chart borders
	∙	X and Y axis titles auto-populated from column names
	∙	Chart title auto-generated as “[Y column] vs [X column] by [Category column]”
Generalization requirements:
	∙	Script must work regardless of what the columns are named — it should auto-detect by position: column 1 = category, column 2 = time period, column 3 = X measure, column 4 = Y measure
	∙	Must handle any number of categories dynamically (not just 5 companies)
	∙	Must handle both 2 time periods and more than 2 time periods (connect all in chronological order per category)
	∙	Color palette should auto-cycle if there are more categories than preset colors
	∙	Labels must auto-avoid overlap where possible
	∙	Percentage formatting on axes if values are between 0 and 100
Power BI specifics:
	∙	Input dataframe is called dataset — this is what Power BI passes in automatically
	∙	Output is a matplotlib figure rendered inline — no plt.show() needed, Power BI handles rendering
	∙	Use only matplotlib, pandas, numpy — no other libraries
	∙	Add a comment block at the top explaining what each field well slot should contain
Output: Single production-ready .py script with inline comments. No explanations outside the script itself.