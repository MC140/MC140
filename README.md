- 👋 Hi, I’m Manohar Chekka
- 👀 I’m Data Engineer
- 📫 Reach me at : manoharch0698@gmail.com


<!---
MC140/MC140 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
let
    // Helper: splits raw line list into individual workspace JSON strings
    // by counting opening/closing braces — works regardless of line endings
    SplitIntoObjects = (lineList as list) as list =>
        let
            NonEmpty = List.Select(lineList, 
                           each Text.Trim(_) <> ""),
            Result = List.Accumulate(
                NonEmpty,
                [objs = {}, cur = "", depth = 0],
                (state, line) =>
                let
                    T      = Text.Trim(line),
                    Opens  = Text.Length(T) - 
                             Text.Length(Text.Remove(T, "{")),
                    Closes = Text.Length(T) - 
                             Text.Length(Text.Remove(T, "}")),
                    NewDepth  = state[depth] + Opens - Closes,
                    NewCur    = if state[cur] = "" then T
                                else state[cur] & " " & T,
                    IsDone    = NewDepth = 0 and 
                                Text.Length(Text.Trim(NewCur)) > 2,
                    NewObjs   = if IsDone 
                                then state[objs] & {NewCur}
                                else state[objs],
                    FinalCur  = if IsDone then "" else NewCur
                in
                    [objs  = NewObjs, 
                     cur   = FinalCur, 
                     depth = NewDepth]
            )
        in
            Result[objs],

    // Main query
    Source = Source_AllFiles,

    FilterFiles = Table.SelectRows(Source, each
        Text.Contains([Name], "WorkspacesScan") and
        not Text.Contains([Name], "MisConfig")),

    AddSnapshotDate = Table.AddColumn(FilterFiles, "SnapshotDate",
        each fnGetSnapshotDate([Name]), type date),

    AddParsedJSON = Table.AddColumn(AddSnapshotDate, "ParsedJSON", each
        let
            RawCSV   = Csv.Document([Content],
                           [Delimiter = ",", Encoding = 65001,
                            QuoteStyle = QuoteStyle.None]),
            Promoted = Table.PromoteHeaders(RawCSV,
                           [PromoteAllScalars = true]),

            // Get all lines from the Scan Data column
            LineList = Table.Column(Promoted, "Scan Data"),

            // Split into individual workspace JSON strings
            // by counting braces
            ObjectStrings = SplitIntoObjects(LineList),

            // Parse each string individually and collect into list
            ParsedList = List.Transform(ObjectStrings,
                             each Json.Document(_))
        in
            ParsedList
    ),

    KeepCols = Table.SelectColumns(AddParsedJSON,
                   {"Name", "SnapshotDate", "ParsedJSON"})
in
    KeepCols

