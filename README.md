- 👋 Hi, I’m Manohar Chekka
- 📫 Reach me at : manoharch0698@gmail.com


<!---
MC140/MC140 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
let
    // Version 1: plain columns
    Set1 = Table.SelectRows(
        Table.SelectColumns(FactAuditEvents, {"Capacity Id", "Capacity Name"}),
        each [Capacity Id] <> null and [Capacity Name] <> null
    ),
    Set1Renamed = Table.RenameColumns(Set1, {{"Capacity Id", "CapacityId"}, {"Capacity Name", "CapacityName"}}),

    // Version 2: Audit.* columns
    Set2 = Table.SelectRows(
        Table.SelectColumns(FactAuditEvents, {"Audit.CapacityId", "Audit.CapacityName"}),
        each [Audit.CapacityId] <> null and [Audit.CapacityName] <> null
    ),
    Set2Renamed = Table.RenameColumns(Set2, {{"Audit.CapacityId", "CapacityId"}, {"Audit.CapacityName", "CapacityName"}}),

    // Combine both sources of names
    AllNames = Table.Distinct(Table.Combine({Set1Renamed, Set2Renamed})),

    // Normalize GUID casing to match the scan (scan GUIDs are UPPERCASE)
    NamesUpper = Table.TransformColumns(AllNames, {{"CapacityId", Text.Upper, type text}}),

    // Complete id list from the scan
    ScanIds = Table.SelectRows(
        Table.Distinct(Table.SelectColumns(DimWorkspaceScanCurrent, {"CapacityId"})),
        each [CapacityId] <> null
    ),
    ScanIdsUpper = Table.TransformColumns(ScanIds, {{"CapacityId", Text.Upper, type text}}),

    // Every scan id, named where activity knew the name
    Merged = Table.NestedJoin(ScanIdsUpper, {"CapacityId"}, NamesUpper, {"CapacityId"}, "N", JoinKind.LeftOuter),
    Expanded = Table.ExpandTableColumn(Merged, "N", {"CapacityName"}),
    Final = Table.TransformColumns(Expanded,
        {{"CapacityName", each if _ = null then "(Unknown - to be named)" else _, type text}}),

    // One row per id (in case two name spellings existed)
    Deduped = Table.Distinct(Final, {"CapacityId"})
in
    Deduped
