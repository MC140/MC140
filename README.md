- 👋 Hi, I’m Manohar Chekka
- 👀 I’m Data Engineer
- 📫 Reach me at : manoharch0698@gmail.com


<!---
MC140/MC140 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
    
let
    Source = Source_AllFiles,

    FilterFiles = Table.SelectRows(Source, each
        Text.Contains([Name], "WorkspacesScan") and
        not Text.Contains([Name], "MisConfig")),

    AddSnapshotDate = Table.AddColumn(FilterFiles, "SnapshotDate",
        each fnGetSnapshotDate([Name]), type date),

    AddParsedJSON = Table.AddColumn(AddSnapshotDate, "ParsedJSON", each
        let
            // Read as raw text — NOT as CSV
            // This preserves all commas inside JSON values
            RawText  = Text.FromBinary([Content], TextEncoding.Utf8),

            // Parse directly
            Parsed   = Json.Document(RawText),

            // Always return a list
            AsList   = if Value.Is(Parsed, type list)
                       then Parsed
                       else {Parsed}
        in
            AsList
    ),

    KeepCols = Table.SelectColumns(AddParsedJSON,
                   {"Name", "SnapshotDate", "ParsedJSON"})
in
    KeepCols

