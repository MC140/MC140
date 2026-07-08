- 👋 Hi, I’m Manohar Chekka
- 📫 Reach me at : manoharch0698@gmail.com


<!---
MC140/MC140 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->

let
    Source = DimWorkspaceScanHistory,

    #"Removed List Columns" = Table.SelectColumns(
        Source,
        {"WorkspaceID", "WorkspaceName", "WorkspaceType", "WorkspaceState",
         "IsOnDedicatedCapacity", "CapacityId", "DefaultDatasetStorageFormat",
         "SnapshotDate", "LoadDate", "SourceFile"}
    ),

    LatestSnapshot =
        List.Max(#"Removed List Columns"[SnapshotDate]),

    FilterLatest =
        Table.SelectRows(
            #"Removed List Columns",
            each [SnapshotDate] = LatestSnapshot
        ),

    RemoveDuplicates =
        Table.Distinct(
            FilterLatest,
            {"WorkspaceID"}
        )
in
    RemoveDuplicates

