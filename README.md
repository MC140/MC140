- 👋 Hi, I’m Manohar Chekka
- 👀 I’m Data Engineer
- 📫 Reach me at : manoharch0698@gmail.com


<!---
MC140/MC140 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->

 (FileName as text) as date =>
let
    // Remove the .csv extension
    CleanName = Text.BeforeDelimiter(FileName, ".csv"),

    // Detect which pattern: dimension files have "--MM-DD-YY" format
    // Activity files have "-YYYYMMDD-" format

    IsDimensionFile = Text.Contains(CleanName, "__"),

    SnapshotDate = if IsDimensionFile then
        // Pattern: ...-07-11-25__19-35
        // Extract the date part before "__"
        let
            DatePart = Text.BetweenDelimiters(CleanName, "-", "__", 
                        {Text.PositionOf(CleanName, "-", Occurrence.Last) - 8, 
                         RelativePosition.FromStart}, 
                        {0, RelativePosition.FromStart}),
            // DatePart = "07-11-25"
            Parts    = Text.Split(DatePart, "-"),
            MM       = Parts{0},
            DD       = Parts{1},
            YY       = Parts{2},
            YYYY     = "20" & YY,
            Result   = Date.FromText(YYYY & "-" & MM & "-" & DD)
        in Result
    else
        // Pattern: PBIActivityEvents-20250710-...
        let
            AfterFirstDash = Text.AfterDelimiter(CleanName, "-"),
            DateStr        = Text.Start(AfterFirstDash, 8),
            // DateStr = "20250710"
            YYYY = Text.Start(DateStr, 4),
            MM   = Text.Middle(DateStr, 4, 2),
            DD   = Text.End(DateStr, 2),
            Result = Date.FromText(YYYY & "-" & MM & "-" & DD)
        in Result

in
    SnapshotDate
