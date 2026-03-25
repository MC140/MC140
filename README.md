- 👋 Hi, I’m Manohar Chekka
- 👀 I’m Data Engineer
- 📫 Reach me at : manoharch0698@gmail.com


<!---
MC140/MC140 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
    

let
    Source = Stg_WorkspacesScan_Base,

    // Expand list into one row per workspace per snapshot
    ExpandList = Table.ExpandListColumn(Source, "ParsedJSON"),

    // Expand workspace-level fields
    ExpandFields = Table.ExpandRecordColumn(
        ExpandList, "ParsedJSON",
        {"id", "name", "type", "state",
         "isOnDedicatedCapacity", "capacityId",
         "defaultDatasetStorageFormat"},
        {"WorkspaceId", "WorkspaceName", "WorkspaceType",
         "WorkspaceState", "IsOnDedicatedCapacity",
         "CapacityId", "DefaultStorageFormat"}
    ),

    // Remove rows with no WorkspaceId
    RemoveNulls = Table.SelectRows(ExpandFields,
                      each [WorkspaceId] <> null and
                           [WorkspaceId] <> ""),

    // Sort by WorkspaceId then SnapshotDate descending
    Sorted = Table.Sort(RemoveNulls, {
                 {"WorkspaceId",   Order.Ascending},
                 {"SnapshotDate",  Order.Descending}
             }),

    // Keep only the first row per WorkspaceId
    // (which is the most recent due to sort above)
    Deduped = Table.Buffer(
                  Table.Distinct(Sorted, {"WorkspaceId"})
              ),

    // Rename SnapshotDate to LastSeenDate
    Renamed = Table.RenameColumns(Deduped, 
                  {{"SnapshotDate", "LastSeenDate"}}),

    // IsActive flag
    LatestDate = List.Max(RemoveNulls[SnapshotDate]),
    LatestIds  = List.Distinct(
                     Table.SelectRows(RemoveNulls,
                         each [SnapshotDate] = LatestDate
                     )[WorkspaceId]
                 ),

    AddIsActive = Table.AddColumn(Renamed, "IsActive",
                      each List.Contains(LatestIds, [WorkspaceId]),
                      type logical),

    // Drop the file Name column
    DropName = Table.RemoveColumns(AddIsActive, {"Name"}),

    SetTypes = Table.TransformColumnTypes(DropName, {
        {"WorkspaceId",           type text},
        {"WorkspaceName",         type text},
        {"WorkspaceType",         type text},
        {"WorkspaceState",        type text},
        {"IsOnDedicatedCapacity", type text},
        {"CapacityId",            type text},
        {"DefaultStorageFormat",  type text},
        {"LastSeenDate",          type date},
        {"IsActive",              type logical}
    })

in
    SetTypes
