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

            // Get all lines, remove nulls and blanks
            LineList  = Table.Column(Promoted, "Scan Data"),
            NonEmpty  = List.Select(LineList, 
                            each _ <> null and Text.Trim(_) <> ""),

            // Join with a unique pipe separator
            Joined = Text.Combine(NonEmpty, "|~|"),

            // Every top-level workspace ends with a lone "}" 
            // followed by a lone "{" for the next one.
            // Pattern in joined text:  }|~|{
            // Replace that boundary with a unique SPLIT marker
            Marked  = Text.Replace(Joined, "}|~|{", "}|||SPLIT|||{"),

            // Split into individual workspace text blocks
            Blocks  = Text.Split(Marked, "|||SPLIT|||"),

            // Each block is now a valid JSON object — parse individually
            Parsed  = List.Transform(Blocks, each 
                        let
                            // Restore original spacing inside each block
                            Clean = Text.Replace(_, "|~|", " ")
                        in
                            try Json.Document(Clean) 
                            otherwise null
                      ),

            // Remove any nulls from failed parses
            Final = List.Select(Parsed, each _ <> null)
        in
            Final
    ),

    KeepCols = Table.SelectColumns(AddParsedJSON,
                   {"Name", "SnapshotDate", "ParsedJSON"})
in
    KeepCols

