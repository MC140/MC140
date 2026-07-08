- 👋 Hi, I’m Manohar Chekka
- 📫 Reach me at : manoharch0698@gmail.com


<!---
MC140/MC140 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->



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

    // Read raw text, drop "Scan Data" header, strip outer quotes,
    // fix }" "{ boundaries between workspaces, wrap in [ ], parse once.
    #"Parsed Workspaces" = Table.AddColumn(
        #"Kept Needed File Columns",
        "Workspaces",
        each
            let
                raw      = Text.FromBinary([Content], 1252),
                lines    = Lines.FromText(raw),
                noHeader = Text.Combine(List.Skip(lines, 1), "#(lf)"),
                t0       = Text.Trim(noHeader),
                t1       = if Text.StartsWith(t0, """") then Text.Range(t0, 1) else t0,
                t2       = if Text.EndsWith(t1, """") then Text.Start(t1, Text.Length(t1) - 1) else t1,
                t3       = Text.Replace(t2, "}""#(cr)#(lf)""{", "},{"),
                t4       = Text.Replace(t3, "}""#(lf)""{", "},{"),
                parsed   = try Json.Document("[" & t4 & "]") otherwise null
            in
                parsed,
        type list
    ),

    #"Removed Content" = Table.RemoveColumns(#"Parsed Workspaces", {"Content"}),

    #"Added SnapshotDate" = Table.AddColumn(
        #"Removed Content", "SnapshotDate",
        each fnGetSnapshotDate([SourceFile]), type date
    ),

    #"Added LoadDate" = Table.AddColumn(
        #"Added SnapshotDate", "LoadDate",
        each Date.From([Date modified]), type date
    ),

    #"Expanded Workspace List" = Table.ExpandListColumn(#"Added LoadDate", "Workspaces"),

    // Keep only rows that parsed into records
    #"Valid Workspaces" = Table.SelectRows(
        #"Expanded Workspace List",
        each [Workspaces] is record
    ),

    #"Expanded Workspace Record" = Table.ExpandRecordColumn(
        #"Valid Workspaces",
        "Workspaces",
        {"id", "name", "type", "state", "isOnDedicatedCapacity", "capacityId",
         "defaultDatasetStorageFormat", "reports", "dashboards", "datasets",
         "dataflows", "datamarts", "users", "folders"},
        {"WorkspaceID", "WorkspaceName", "WorkspaceType", "WorkspaceState",
         "IsOnDedicatedCapacity", "CapacityId", "DefaultDatasetStorageFormat",
         "Reports", "Dashboards", "Datasets", "Dataflows", "Datamarts",
         "Users", "Folders"}
    ),

    // Normalize: if a nested field is "" or " " instead of a list, make it null
    #"Cleaned Nested Lists" = Table.TransformColumns(
        #"Expanded Workspace Record",
        {
            {"Reports",    each if _ is list then _ else null},
            {"Dashboards", each if _ is list then _ else null},
            {"Datasets",   each if _ is list then _ else null},
            {"Dataflows",  each if _ is list then _ else null},
            {"Datamarts",  each if _ is list then _ else null},
            {"Users",      each if _ is list then _ else null},
            {"Folders",    each if _ is list then _ else null}
        }
    ),

    #"Changed Final Types" = Table.TransformColumnTypes(
        #"Cleaned Nested Lists",
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


