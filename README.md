- 👋 Hi, I’m Manohar Chekka
- 📫 Reach me at : manoharch0698@gmail.com


<!---
MC140/MC140 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
let
    Source = FactWorkspaceUsers,

    LatestSnapshot =
        List.Max(Source[SnapshotDate]),

    FilterLatest =
        Table.SelectRows(
            Source,
            each [SnapshotDate] = LatestSnapshot
        ),

    RemoveDuplicates =
        Table.Distinct(
            FilterLatest,
            {"WorkspaceID", "Identifier", "AccessRight"}
        )
in
    RemoveDuplicates

