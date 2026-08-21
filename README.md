- 👋 Hi, I’m Manohar Chekka
- 📫 Reach me at : manoharch0698@gmail.com


<!---
MC140/MC140 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
For the star-schema conversion, I’d suggest doing it upstream in Databricks rather than inside Power BI:

1. Define the fact grain
    * Confirm exactly what one row represents, e.g. one transaction/inter­action.
2. Identify dimensions
    * Customer, Product, Employee, Branch, Date, Channel, etc.
    * Confirm the natural/business key for each dimension.
3. Create dimension tables in Databricks
    * Select distinct business entities from the flat data.
    * Keep the original business key.
    * Generate a stable integer surrogate key for each dimension.
    * Do not regenerate keys with ROW_NUMBER() every refresh; persist the key mapping.
    * Apply SCD Type 1/2 if historical attribute changes need to be tracked.
4. Create the fact table
    * Join the flat data to each dimension using the business keys.
    * Bring the surrogate keys into the fact.
    * Keep measures and required transaction-level fields.
    * Remove descriptive fields now stored in dimensions.
5. Clean/optimize
    * Remove unused columns.
    * Review high-cardinality text, GUIDs, descriptions, timestamps, etc.
    * Use appropriate numeric/data types.
6. Publish curated tables
    * Write Fact_* and Dim_* tables from Databricks into the AZ/Synapse consumption layer.
7. Build Power BI model
    * Load Fact + Dimensions from Synapse.
    * Create 1: single-direction relationships* from Dimension → Fact.
    * Use Import + incremental refresh for the large fact where appropriate.
8. Validate against the current flat model
    * Compare model size
    * Refresh time
    * Visual/query performance
    * Maintainability

Target flow:
Flat data → Databricks → Fact + Dimensions → AZ Synapse → Power BI