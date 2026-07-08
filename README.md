- 👋 Hi, I’m Manohar Chekka
- 📫 Reach me at : manoharch0698@gmail.com


<!---
MC140/MC140 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
let
    Set1 = Table.SelectRows(
        Table.SelectColumns(FactAuditEvents, {"Capacity Id", "Capacity Name"}),
        each [Capacity Id] <> null and [Capacity Name] <> null
    ),
    Set1Renamed = Table.RenameColumns(Set1, {{"Capacity Id", "CapacityId"}, {"Capacity Name", "CapacityName"}}),

    Set2 = Table.SelectRows(
        Table.SelectColumns(FactAuditEvents, {"Audit.CapacityId", "Audit.CapacityName"}),
        each [Audit.CapacityId] <> null and [Audit.CapacityName] <> null
    ),
    Set2Renamed = Table.RenameColumns(Set2, {{"Audit.CapacityId", "CapacityId"}, {"Audit.CapacityName", "CapacityName"}}),

    AllNames = Table.Distinct(Table.Combine({Set1Renamed, Set2Renamed})),

    NamesUpper = Table.TransformColumns(AllNames, {{"CapacityId", Text.Upper, type text}}),

    ScanIds = Table.SelectRows(
        Table.Distinct(Table.SelectColumns(DimWorkspaceScanCurrent, {"CapacityId"})),
        each [CapacityId] <> null
    ),
    ScanIdsUpper = Table.TransformColumns(ScanIds, {{"CapacityId", Text.Upper, type text}}),

    Merged = Table.NestedJoin(ScanIdsUpper, {"CapacityId"}, NamesUpper, {"CapacityId"}, "N", JoinKind.LeftOuter),
    Expanded = Table.ExpandTableColumn(Merged, "N", {"CapacityName"}),
    Final = Table.TransformColumns(Expanded,
        {{"CapacityName", each if _ = null then "(Unknown - to be named)" else _, type text}}),

    Deduped = Table.Distinct(Final, {"CapacityId"})
in
    Deduped

