- 👋 Hi, I’m Manohar Chekka
- 👀 I’m Data Engineer
- 📫 Reach me at : manoharch0698@gmail.com


<!---
MC140/MC140 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->

 File Classification Final =
VAR s = LOWER(TRIM([Source]))

-- Flags: detect tokens anywhere in the string
VAR HasAzure =
    CONTAINSSTRING(s, "azure_sql_dw") || CONTAINSSTRING(s, "azure_sqldb") || CONTAINSSTRING(s, "azure_")

VAR HasFile =
    CONTAINSSTRING(s, "excel") || CONTAINSSTRING(s, "csv") || CONTAINSSTRING(s, "msaccess")
        || CONTAINSSTRING(s, "textscan") || CONTAINSSTRING(s, "textclean")
        || CONTAINSSTRING(s, "googledrive") || CONTAINSSTRING(s, "google-sheets")
        || CONTAINSSTRING(s, "hyper")

VAR HasOnPrem =
    CONTAINSSTRING(s, "sqlserver") || CONTAINSSTRING(s, "oracle") || CONTAINSSTRING(s, "db2")
        || CONTAINSSTRING(s, "teradata") || CONTAINSSTRING(s, "hadoop") || CONTAINSSTRING(s, "hadoopive")
        || CONTAINSSTRING(s, "snowflake") || CONTAINSSTRING(s, "genericodbc")
        || CONTAINSSTRING(s, "ogridirect") || CONTAINSSTRING(s, "webdata-direct")
        || CONTAINSSTRING(s, "semistructpassivestore-direct")
        || CONTAINSSTRING(s, "stat-direct")

-- Count separators to guess if it has multiple sources
VAR IsMulti =
    CONTAINSSTRING(s, ";")

RETURN
SWITCH(
    TRUE(),
    -- Catherine rule 1
    HasFile && HasOnPrem, "File+On-Prem",

    -- Catherine rule 2 (Azure combined with anything else)
    HasAzure && (HasFile || HasOnPrem) && IsMulti, "Azure+File/On-Prem",

    -- Pure Azure
    HasAzure && NOT(HasFile || HasOnPrem), "Azure",

    -- Otherwise keep broad buckets
    HasFile, "File",
    HasOnPrem, "On-Prem MAL",

    "Other"
)