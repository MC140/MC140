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
        {"WorkspaceID", "WorkspaceName", "SnapshotDate", "Users"}
    ),

    #"Expanded Users" = Table.ExpandListColumn(#"Kept Columns", "Users"),

    #"Valid Users" = Table.SelectRows(#"Expanded Users", each [Users] is record),

    #"Expanded User Record" = Table.ExpandRecordColumn(
        #"Expanded Users" , "Users",
        {"groupUserAccessRight", "emailAddress", "displayName",
         "identifier", "graphId", "principalType", "userType"},
        {"AccessRight", "EmailAddress", "DisplayName",
         "Identifier", "GraphId", "PrincipalType", "UserType"}
    )
in
    #"Expanded User Record"

