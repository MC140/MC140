- 👋 Hi, I’m Manohar Chekka
- 👀 I’m Data Engineer
- 📫 Reach me at : manoharch0698@gmail.com


<!---
MC140/MC140 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
    
let
    Source = Stg_WorkspacesScan_Base,

    // Expand each file's list into individual workspace rows
    ExpandList = Table.ExpandListColumn(Source, "ParsedJSON"),

    // Expand workspace-level fields from each record
    ExpandFields = Table.ExpandRecordColumn(
        ExpandList, "ParsedJSON",
        {"id", "name", "type", "state",
         "isOnDedicatedCapacity", "capacityId",
         "defaultDatasetStorageFormat"},
        {"WorkspaceId", "WorkspaceName", "WorkspaceType",
         "WorkspaceState", "IsOnDedicatedCapacity",
         "CapacityId", "DefaultStorageFormat"}
    ),

    // Remove nulls
    RemoveNulls = Table.SelectRows(ExpandFields,
                      each [WorkspaceId] <> null and
                           [WorkspaceId] <> ""),

    // DEDUP: one row per WorkspaceId keeping most recent snapshot
    MaxDatePerWS = Table.Group(RemoveNulls, {"WorkspaceId"}, {
        {"WorkspaceName",         each List.Last(List.Sort([WorkspaceName])),         type text},
        {"WorkspaceType",         each List.Last(List.Sort([WorkspaceType])),         type text},
        {"WorkspaceState",        each List.Last(List.Sort([WorkspaceState])),        type text},
        {"IsOnDedicatedCapacity", each List.Last(List.Sort(
                                      List.Transform([IsOnDedicatedCapacity], 
                                          each Text.From(_)))),                       type text},
        {"CapacityId",            each List.Last(List.Sort([CapacityId])),            type text},
        {"DefaultStorageFormat",  each List.Last(List.Sort([DefaultStorageFormat])), type text},
        {"LastSeenDate",          each List.Max([SnapshotDate]),                      type date}
    }),

    // IsActive = appeared in the latest snapshot file
    LatestSnapshotDate = List.Max(RemoveNulls[SnapshotDate]),

    // Get WorkspaceIds in the latest snapshot
    LatestWSIds = Table.SelectRows(RemoveNulls, 
                      each [SnapshotDate] = LatestSnapshotDate)[WorkspaceId],

    AddIsActive = Table.AddColumn(MaxDatePerWS, "IsActive",
        each List.Contains(LatestWSIds, [WorkspaceId]), type logical),

    // Final types
    SetTypes = Table.TransformColumnTypes(AddIsActive, {
        {"WorkspaceId",           type text},
        {"WorkspaceName",         type text},
        {"WorkspaceType",         type text},
        {"WorkspaceState",        type text},
        {"CapacityId",            type text},
        {"DefaultStorageFormat",  type text},
        {"LastSeenDate",          type date},
        {"IsActive",              type logical}
    })

in
    SetTypes
