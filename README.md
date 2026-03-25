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
            RawCSV   = Csv.Document([Content],
                           [Delimiter = ",", Encoding = 65001,
                            QuoteStyle = QuoteStyle.None]),
            Promoted = Table.PromoteHeaders(RawCSV,
                           [PromoteAllScalars = true]),

            LineList = Table.Column(Promoted, "Scan Data"),
            NonEmpty = List.Select(LineList,
                           each _ <> null and Text.Trim(_) <> ""),

            // Join all lines back into one string
            Joined = Text.Combine(NonEmpty, " "),

            // Parse directly — file is already valid JSON
            Parsed = Json.Document(Joined),

            // Normalise: whether it's a record or a list,
            // always return a list of records
            AsList = if Value.Is(Parsed, type list) 
                     then Parsed 
                     else {Parsed}
        in
            AsList
    ),

    KeepCols = Table.SelectColumns(AddParsedJSON,
                   {"Name", "SnapshotDate", "ParsedJSON"})
in
    KeepCols

