- 👋 Hi, I’m Manohar Chekka
- 📫 Reach me at : manoharch0698@gmail.com


<!---
MC140/MC140 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->

let
    Source = DimWorkspaceScanHistory,

    #"Kept Columns" = Table.SelectColumns(
        Source,
        {"WorkspaceID", "WorkspaceName", "SnapshotDate", "LoadDate",
         "Reports", "Dashboards", "Datasets", "Dataflows",
         "Datamarts", "Users", "Folders"}
    ),

    // Stack the 7 list columns into EntityType + Item pairs
    #"Unpivoted" = Table.UnpivotOtherColumns(
        #"Kept Columns",
        {"WorkspaceID", "WorkspaceName", "SnapshotDate", "LoadDate"},
        "EntityType", "Item"
    ),

    // Each Item is a list of records -> one row per record
    #"Expanded Items" = Table.ExpandListColumn(#"Unpivoted", "Item"),

    #"Valid Items" = Table.SelectRows(#"Expanded Items", each [Item] is record),

    // Pull out the union of fields across all entity types.
    // Fields that don't apply to a given entity just come back null.
    #"Expanded Item Record" = Table.ExpandRecordColumn(
        #"Valid Items", "Item",
        {"id", "name", "reportType", "datasetId",
         "createdDateTime", "modifiedDateTime", "createdBy", "modifiedBy",
         "configuredBy", "targetStorageMode", "contentProviderType", "createdDate",
         "groupUserAccessRight", "displayName", "emailAddress",
         "identifier", "principalType", "userType", "folderId"},
        {"ItemID", "ItemName", "ReportType", "DatasetId",
         "CreatedDateTime", "ModifiedDateTime", "CreatedBy", "ModifiedBy",
         "ConfiguredBy", "TargetStorageMode", "ContentProviderType", "CreatedDate",
         "AccessRight", "DisplayName", "EmailAddress",
         "Identifier", "PrincipalType", "UserType", "FolderId"}
    )
in
    #"Expanded Item Record"


