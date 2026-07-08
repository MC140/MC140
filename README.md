- 👋 Hi, I’m Manohar Chekka
- 📫 Reach me at : manoharch0698@gmail.com


<!---
MC140/MC140 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->


let
    Source = DimWorkspaceScanHistory,
    #"Kept Columns" = Table.SelectColumns(Source,
        {"WorkspaceID", "WorkspaceName", "SnapshotDate", "Reports"}),
    #"Expanded List" = Table.ExpandListColumn(#"Kept Columns", "Reports"),
    #"Valid Rows" = Table.SelectRows(#"Expanded List", each [Reports] is record),
    #"Expanded Record" = Table.ExpandRecordColumn(#"Valid Rows", "Reports",
        {"reportType", "id", "name", "datasetId", "createdDateTime",
         "modifiedDateTime", "createdBy", "modifiedBy", "folderId"},
        {"ReportType", "ReportID", "ReportName", "DatasetId", "CreatedDateTime",
         "ModifiedDateTime", "CreatedBy", "ModifiedBy", "FolderId"})
in
    #"Expanded Record"

let
    Source = DimWorkspaceScanHistory,
    #"Kept Columns" = Table.SelectColumns(Source,
        {"WorkspaceID", "WorkspaceName", "SnapshotDate", "Datasets"}),
    #"Expanded List" = Table.ExpandListColumn(#"Kept Columns", "Datasets"),
    #"Valid Rows" = Table.SelectRows(#"Expanded List", each [Datasets] is record),
    #"Expanded Record" = Table.ExpandRecordColumn(#"Valid Rows", "Datasets",
        {"id", "name", "configuredBy", "targetStorageMode", "createdDate",
         "contentProviderType", "isEffectiveIdentityRequired", "refreshSchedule"},
        {"DatasetID", "DatasetName", "ConfiguredBy", "TargetStorageMode", "CreatedDate",
         "ContentProviderType", "IsEffectiveIdentityRequired", "RefreshSchedule"})
in
    #"Expanded Record"


let
    Source = DimWorkspaceScanHistory,
    #"Kept Columns" = Table.SelectColumns(Source,
        {"WorkspaceID", "WorkspaceName", "SnapshotDate", "Dashboards"}),
    #"Expanded List" = Table.ExpandListColumn(#"Kept Columns", "Dashboards"),
    #"Valid Rows" = Table.SelectRows(#"Expanded List", each [Dashboards] is record),
    #"Expanded Record" = Table.ExpandRecordColumn(#"Valid Rows", "Dashboards",
        {"id", "displayName", "isReadOnly"},
        {"DashboardID", "DashboardName", "IsReadOnly"})
in
    #"Expanded Record"



let
    Source = DimWorkspaceScanHistory,
    #"Kept Columns" = Table.SelectColumns(Source,
        {"WorkspaceID", "WorkspaceName", "SnapshotDate", "Folders"}),
    #"Expanded List" = Table.ExpandListColumn(#"Kept Columns", "Folders"),
    #"Valid Rows" = Table.SelectRows(#"Expanded List", each [Folders] is record),
    #"Expanded Record" = Table.ExpandRecordColumn(#"Valid Rows", "Folders",
        {"id", "displayName", "parentFolderId"},
        {"FolderID", "FolderName", "ParentFolderId"})
in
    #"Expanded Record"
