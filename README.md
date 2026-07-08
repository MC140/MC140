- 👋 Hi, I’m Manohar Chekka
- 📫 Reach me at : manoharch0698@gmail.com


<!---
MC140/MC140 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->

let
    Source = SharePoint.Files(
        "https://myteam.td.com/teams/PowerBIEnhancements/",
        [ApiVersion = 15]
    ),

    #"Filtered Rows" = Table.SelectRows(
        Source,
        each [Folder Path] = "https://myteam.td.com/teams/PowerBIEnhancements/Shared Documents/Power BI Premium Capacity User Logs/Prod Capacity/Dimensions/"
            and Text.StartsWith([Name], "DAAS_PowerBIUsageReportGeneration-WorkspacesScan-")
    ),

    #"Filtered Hidden Files" = Table.SelectRows(
        #"Filtered Rows",
        each [Attributes]?[Hidden]? <> true
    ),

    #"Renamed Source File" = Table.RenameColumns(
        #"Filtered Hidden Files",
        {{"Name", "SourceFile"}}
    ),

    #"Kept Needed File Columns" = Table.SelectColumns(
        #"Renamed Source File",
        {"SourceFile", "Date modified", "Content"}
    ),

    // CSV parser strips the outer quotes and un-doubles "" inside each row.
    // Columns=1 keeps each JSON object in a single cell even though it contains commas.
    #"Added CSV Table" = Table.AddColumn(
        #"Kept Needed File Columns",
        "Data",
        each Table.PromoteHeaders(
            Csv.Document(
                [Content],
                [Delimiter = ",", Columns = 1, Encoding = 1252, QuoteStyle = QuoteStyle.Csv]
            ),
            [PromoteAllScalars = true]
        )
    ),

    #"Removed Content" = Table.RemoveColumns(#"Added CSV Table", {"Content"}),

    #"Expanded Data" = Table.ExpandTableColumn(
        #"Removed Content",
        "Data",
        {"Scan Data"},
        {"Scan Data"}
    ),

    #"Changed Type" = Table.TransformColumnTypes(
        #"Expanded Data",
        {
            {"SourceFile", type text},
            {"Date modified", type datetime},
            {"Scan Data", type text}
        }
    ),

    // Drop blank/junk rows so only real JSON objects go to the parser
    #"Filtered Valid Rows" = Table.SelectRows(
        #"Changed Type",
        each [Scan Data] <> null
            and Text.StartsWith(Text.Trim([Scan Data]), "{")
    ),

    #"Added SnapshotDate" = Table.AddColumn(
        #"Filtered Valid Rows",
        "SnapshotDate",
        each fnGetSnapshotDate([SourceFile]),
        type date
    ),

    #"Added LoadDate" = Table.AddColumn(
        #"Added SnapshotDate",
        "LoadDate",
        each Date.From([Date modified]),
        type date
    ),

    #"Parsed JSON" = Table.AddColumn(
        #"Added LoadDate",
        "Parsed",
        each try Json.Document([Scan Data]) otherwise null,
        type record
    ),

    #"Expanded Workspace Record" = Table.ExpandRecordColumn(
        #"Parsed JSON",
        "Parsed",
        {"id", "name", "type", "state", "isOnDedicatedCapacity", "capacityId", "defaultDatasetStorageFormat"},
        {"WorkspaceID", "WorkspaceName", "WorkspaceType", "WorkspaceState", "IsOnDedicatedCapacity", "CapacityId", "DefaultDatasetStorageFormat"}
    ),

    #"Removed Raw Scan Data" = Table.RemoveColumns(
        #"Expanded Workspace Record",
        {"Scan Data", "Date modified"}
    ),

    #"Changed Final Types" = Table.TransformColumnTypes(
        #"Removed Raw Scan Data",
        {
            {"WorkspaceID", type text},
            {"WorkspaceName", type text},
            {"WorkspaceType", type text},
            {"WorkspaceState", type text},
            {"IsOnDedicatedCapacity", type logical},
            {"CapacityId", type text},
            {"DefaultDatasetStorageFormat", type text},
            {"SnapshotDate", type date},
            {"LoadDate", type date}
        }
    ),

    #"Removed Duplicates" = Table.Distinct(
        #"Changed Final Types",
        {"WorkspaceID", "SnapshotDate"}
    )
in
    #"Removed Duplicates"
