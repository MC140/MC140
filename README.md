- 👋 Hi, I’m Manohar Chekka
- 👀 I’m Data Engineer
- 📫 Reach me at : manoharch0698@gmail.com


<!---
MC140/MC140 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->

(FileName as text) as date =>
let
    CleanName = Text.BeforeDelimiter(FileName, ".csv"),

    IsDimensionFile = Text.Contains(CleanName, "__"),

    SnapshotDate = if IsDimensionFile then
        // Pattern: ...-Workspaces-07-11-25__19-35
        // Everything before __ then take last 8 chars = "07-11-25"
        let
            BeforeUnderscore = Text.BeforeDelimiter(CleanName, "__"),
            DatePart = Text.End(BeforeUnderscore, 8),
            Parts = Text.Split(DatePart, "-"),
            MM   = Parts{0},
            DD   = Parts{1},
            YY   = Parts{2},
            Result = Date.FromText("20" & YY & "-" & MM & "-" & DD)
        in Result
    else
        // Pattern: PBIActivityEvents-20250710-202507112339
        let
            Parts  = Text.Split(CleanName, "-"),
            DateStr = Parts{1},
            YYYY = Text.Start(DateStr, 4),
            MM   = Text.Middle(DateStr, 4, 2),
            DD   = Text.End(DateStr, 2),
            Result = Date.FromText(YYYY & "-" & MM & "-" & DD)
        in Result

in
    SnapshotDate
