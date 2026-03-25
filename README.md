- 👋 Hi, I’m Manohar Chekka
- 👀 I’m Data Engineer
- 📫 Reach me at : manoharch0698@gmail.com


<!---
MC140/MC140 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
let
    // Reference our master file list
    Source = Source_AllFiles,

    // Filter to WorkspacesScan files only
    FilterFiles = Table.SelectRows(Source, each 
        Text.Contains([Name], "WorkspacesScan") and 
        not Text.Contains([Name], "MisConfig")),

    // Add SnapshotDate from filename
    AddSnapshotDate = Table.AddColumn(FilterFiles, "SnapshotDate", 
        each fnGetSnapshotDate([Name]), type date),

    // Parse each file's JSON
    AddParsedJSON = Table.AddColumn(AddSnapshotDate, "ParsedJSON", each 
        let
            // Read CSV content
            RawCSV    = Csv.Document([Content], 
                            [Delimiter=",", Encoding=65001, 
                             QuoteStyle=QuoteStyle.None]),
            Promoted  = Table.PromoteHeaders(RawCSV, 
                            [PromoteAllScalars=true]),

            // Get all values from "Scan Data" column as a list
            DataList  = Table.Column(Promoted, "Scan Data"),

            // Remove nulls and empty rows
            CleanList = List.Select(DataList, each 
                            _ <> null and _ <> ""),

            // Concatenate all lines back into one JSON string
            JSONText  = Text.Combine(CleanList, "#(lf)"),

            // Parse — if top level is array use as-is
            // If top level is a record (single workspace) wrap in list
            Parsed    = Json.Document(JSONText),
            AsList    = if Value.Is(Parsed, type list) then Parsed
                        else {Parsed}
        in
            AsList
    ),

    // Keep only what we need
    KeepCols = Table.SelectColumns(AddParsedJSON, 
                   {"Name", "SnapshotDate", "ParsedJSON"})

in
    KeepCols


