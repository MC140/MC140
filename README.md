- 👋 Hi, I’m Manohar Chekka
- 👀 I’m Data Engineer
- 📫 Reach me at : manoharch0698@gmail.com


<!---
MC140/MC140 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->

I’m building a Power BI Adoption & Operational report in Power BI Desktop. Source is a SharePoint folder with CSV files arriving twice weekly. Two file types: PBIActivityEvents (fact/logs) and dimension files (Workspaces, WorkspacesScan, Datasets, Datasources etc).
Phase 1 done: Parameter SharePointFolderPath, query Source_AllFiles (filters CSVs from SharePoint folder), function fnGetSnapshotDate (parses date from filename) — all working.
Phase 2 stuck: Query Stg_WorkspacesScan_Base reads WorkspacesScan files. These are single-column CSVs with JSON spread line by line. ParsedJSON column shows List, but clicking into it shows a nested List instead of Records. Tried Csv.Document, Text.FromBinary, JSON wrapping — all fail. Next step: open one WorkspacesScan CSV in Notepad and check the very first character ([ or {) to determine parsing strategy. Then fix the M code to correctly parse JSON into one row per 