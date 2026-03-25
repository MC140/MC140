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
                           [Delimiter=",", Encoding=65001, 
                            QuoteStyle=QuoteStyle.None]),
            Promoted = Table.PromoteHeaders(RawCSV, 
                           [PromoteAllScalars=true]),

            DataList  = Table.Column(Promoted, "Scan Data"),
            CleanList = List.Select(DataList, each 
                            _ <> null and Text.Trim(_) <> ""),

            // Join all lines back into one text block
            JSONText = Text.Combine(CleanList, "#(lf)"),

            // THE FIX: multiple JSON objects sit back to back
            // Replace }newline{ boundary with },{ 
            // then wrap entire thing in [ ] to make valid array
            Step1 = Text.Replace(JSONText, 
                        "}" & "#(cr)#(lf)" & "{", "},{"),
            Step2 = Text.Replace(Step1, 
                        "}" & "#(lf)" & "{", "},{"),
            WrappedJSON = "[" & Step2 & "]",

            Parsed = Json.Document(WrappedJSON)
        in
            Parsed
    ),

    KeepCols = Table.SelectColumns(AddParsedJSON, 
                   {"Name", "SnapshotDate", "ParsedJSON"})
in
    KeepCols
