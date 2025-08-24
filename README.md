- 👋 Hi, I’m Manohar Chekka
- 👀 I’m Data Engineer
- 📫 Reach me at : manoharch0698@gmail.com

<!---
MC140/MC140 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
// === begin: parse "Audit Data" into columns ===
#"Audit Clean" =
    Table.TransformColumns(
        #"Removed Columns",
        {{"Audit Data", each Text.Trim(Text.From(_)), type text}}
    ),

#"Add KV" =
    Table.AddColumn(
        #"Audit Clean", "KV",
        each
            let
                raw   = [#"Audit Data"],
                inner = try Text.BetweenDelimiters(raw, "@{", "}", 0, 0) otherwise raw,
                pairs = List.Transform(Text.Split(inner, ";"), each Text.Trim(_)),
                kv    =
                    List.Transform(
                        List.Select(pairs, each Text.Contains(_, "=")),
                        (p) =>
                            let
                                sp = Text.Split(p, "="),
                                k  = Text.Trim(sp{0}),
                                v  = Text.Trim(Text.Combine(List.Skip(sp, 1), "="))
                            in
                                [Key = k, Value = v]
                    ),
                tbl   = #table({"Key","Value"}, kv)
            in
                tbl,
        type table [Key = text, Value = text]
    ),

#"Pivot Fields" =
    Table.AddColumn(
        #"Add KV", "Fields",
        each Table.Pivot([KV], List.Distinct([KV][Key]), "Key", "Value"),
        type table
    ),

#"Expanded Fields" =
    Table.ExpandTableColumn(
        #"Pivot Fields", "Fields",
        // list any keys you want as columns; include more if present
        {"Id","ActivityId","ItemId","WorkspaceId","CapacityId","UserId","CreationTime"},
        {"EventId","ActivityId","ItemId","WorkspaceId","CapacityId","UserId","CreationTime"}
    ),

#"Typed Fields" =
    Table.TransformColumnTypes(
        #"Expanded Fields",
        {{"CreationTime", type datetime}, {"EventId", type text}}
    )
// === end: parse "Audit Data" into columns ===