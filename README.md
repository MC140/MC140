- 👋 Hi, I’m Manohar Chekka
- 👀 I’m Data Engineer
- 📫 Reach me at : manoharch0698@gmail.com

<!---
MC140/MC140 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
let
  Source = #"Previous Step",
  Clean  = Table.TransformColumnTypes(
             Table.TransformColumns(Source, {{"Audit Data", each Text.Trim(_), type text}}),
             {{"Time of Event", type datetime}}
           ),

  // Build a per-row table of [Key, Value] pairs from "@{Key=Value; Key2=Value2}"
  ToKV = Table.AddColumn(
           Clean, "KV",
           each
             let
               raw      = [Audit Data],
               inner    = try Text.BetweenDelimiters(raw, "@{", "}", 0, 0) otherwise raw,
               // split into "Key=Value" strings
               pairs    = List.Transform(Text.Split(inner, ";"), each Text.Trim(_)),
               // keep only entries with "=" and split once (allow "=" inside values)
               kvRows   = List.Transform(
                            List.Select(pairs, each Text.Contains(_, "=")),
                            (p) =>
                              let
                                sp = Text.Split(p, "="),
                                k  = Text.Trim(sp{0}),
                                v  = Text.Trim(Text.Combine(List.Skip(sp, 1), "="))
                              in [Key = k, Value = v]
                          ),
               kvTable  = #table({"Key","Value"}, kvRows)
             in
               kvTable,
           type table [Key=text, Value=text]
         ),

  // Pivot each row's KV table to columns, then expand to the main row
  Pivoted = Table.AddColumn(
              ToKV, "Fields",
              each Table.Pivot([KV], List.Distinct([KV][Key]), "Key", "Value"),
              type table
            ),
  Expanded = Table.ExpandTableColumn(Pivoted, "Fields", null, null),

  // Optional: set useful types if present
  Typed =
    let
      cols = List.Intersect({{"Id"},{"ActivityId"},{"ItemId"},{"UserId"},{"WorkspaceId"},{"CapacityId"},{"CreationTime"}} & {Table.ColumnNames(Expanded)}),
      t1 = if List.Contains(Table.ColumnNames(Expanded), "CreationTime")
           then Table.TransformColumnTypes(Expanded, {{"CreationTime", type datetime}})
           else Expanded
    in
      t1
in
  Typed