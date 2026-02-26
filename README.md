- 👋 Hi, I’m Manohar Chekka
- 👀 I’m Data Engineer
- 📫 Reach me at : manoharch0698@gmail.com


<!---
MC140/MC140 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->

 File Classification 1 =
VAR s0 = LOWER ( TRIM ( 'Until Feb data'[Source] ) )
-- remove non-breaking spaces + normal spaces
VAR s1 = SUBSTITUTE ( s0, UNICHAR ( 160 ), " " )
VAR s2 = SUBSTITUTE ( s1, " ", "" )
-- pad with ; so token matching works for: ";sql;" ";azure_sql_dw;" etc.
VAR s  = ";" & s2 & ";"

-- Azure tokens
VAR HasAzure =
    CONTAINSSTRING ( s, ";azure_sql_dw;" )
        || CONTAINSSTRING ( s, ";azure_sqldb;" )
        || CONTAINSSTRING ( s, ";azure_" )

-- File / extract / flat-file style tokens (incl Tableau extracts)
VAR HasFile =
    CONTAINSSTRING ( s, ";excel-direct;" )
        || CONTAINSSTRING ( s, ";excel;" )
        || CONTAINSSTRING ( s, ";csv;" )
        || CONTAINSSTRING ( s, ";msaccess;" )
        || CONTAINSSTRING ( s, ";hyper;" )
        || CONTAINSSTRING ( s, ";textscan;" )
        || CONTAINSSTRING ( s, ";textclean;" )
        || CONTAINSSTRING ( s, ";googledrive;" )
        || CONTAINSSTRING ( s, ";google-sheets;" )
        || CONTAINSSTRING ( s, ";tableau;" )
        || CONTAINSSTRING ( s, ";tde;" )
        || CONTAINSSTRING ( s, ";union;" )

-- On-prem / server / ODBC / big-data / DB style tokens (incl SQL token)
VAR HasOnPrem =
    CONTAINSSTRING ( s, ";sql;" )
        || CONTAINSSTRING ( s, ";sqlserver;" )
        || CONTAINSSTRING ( s, ";oracle;" )
        || CONTAINSSTRING ( s, ";db2;" )
        || CONTAINSSTRING ( s, ";teradata;" )
        || CONTAINSSTRING ( s, ";hadoop;" )
        || CONTAINSSTRING ( s, ";hadoophive;" )
        || CONTAINSSTRING ( s, ";snowflake;" )
        || CONTAINSSTRING ( s, ";genericodbc;" )
        || CONTAINSSTRING ( s, ";ogridirect;" )
        || CONTAINSSTRING ( s, ";webdata-direct;" )
        || CONTAINSSTRING ( s, ";semistructpassivestore-direct;" )
        || CONTAINSSTRING ( s, ";stat-direct;" )
        || CONTAINSSTRING ( s, ";mongodb;" )

-- server db (kept separate if you want a distinct bucket)
VAR HasServerDb =
    CONTAINSSTRING ( s, ";postgres;" )

-- multi-source indicator
VAR IsMulti =
    CONTAINSSTRING ( s2, ";" )

RETURN
SWITCH (
    TRUE (),

    -- Catherine: if both File + On-Prem present
    HasFile && ( HasOnPrem || HasServerDb ), "File+On-Prem",

    -- Azure mixed with anything else
    HasAzure && ( HasFile || HasOnPrem || HasServerDb ) && IsMulti, "Azure+File/On-Prem",

    -- Pure Azure
    HasAzure && NOT ( HasFile || HasOnPrem || HasServerDb ), "Azure",

    -- Optional separate bucket
    HasServerDb, "Server DB",

    -- Otherwise
    HasFile, "File",
    HasOnPrem, "On-Prem MAL",

    "Other"
)

