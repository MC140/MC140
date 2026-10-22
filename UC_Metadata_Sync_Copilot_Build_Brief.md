# BUILD BRIEF: UC Metadata Sync — Power BI COE Edition

> **Instructions to GitHub Copilot Agent / organizational coding assistant:** Build the working application described in this document. Treat this as a product and engineering specification, not as a request for an explanation. Implement the MVP end-to-end in the opened VS Code workspace, create tests and sample fixtures, verify functionality, and leave a clear README. Do not stop after producing pseudocode or a mock UI. If an integration is not permitted, implement the offline/manual handoff specified below instead of bypassing restrictions.

## 1. Background and the actual problem

We work in a tightly governed enterprise Power BI Center of Excellence (COE), using Azure Databricks Unity Catalog (UC) as a data source and Power BI Desktop for development. Business users add table and column comments/descriptions in UC. When they connect normally via Power BI Desktop's Azure Databricks connector, those comments generally do not populate the **Description** properties of the Power BI semantic model's tables and columns. We want to preserve a shared business glossary without manually copying descriptions to dozens/hundreds of fields.

**This behavior has already been tested:** On October 22, 2026, we successfully ran a SQL query from Power BI Desktop over Databricks Unity Catalog information schema and loaded the returned metadata rows into Power BI. The result contains `table_catalog`, `table_schema`, `table_name`, `table_description`, `column_name`, `data_type`, and `column_description`. This proves that an authorized SQL connection can retrieve the comments in this environment, without any Databricks REST API access.

Real POC example (identifiers are examples; they must never be hardcoded as default organizational data):

- Catalog: `d3002-eastus2-aaaz-az4`
- Schema: `powerbi_training`
- Table: `sales_product_summary`
- UC table comment: `This is a sample Product Sales data.`
- UC column comment: `partition_date` -> `Partition Date`
- UC column comment: `date` -> `this is Date Column`
- UC column comment: `date_key` -> `Date Key`
- Other columns may have `NULL` comments, and their descriptions must not be erased in Power BI.

**MVP success criterion:** Given metadata from this SQL result and a Power BI TMDL script for the underlying `sales_product_summary` model table, produce a *minimal, audited, description-only* revised TMDL script. After applying it in Power BI Desktop's TMDL view, the original model table and matching columns display the UC descriptions. No model data, measures, relationships, RLS, partitions, or other metadata should change.

## 2. Non-negotiable enterprise constraints

1. **No Databricks REST APIs** (forbidden by organizational policy).
2. **No direct publishing from Databricks to Power BI**, no workspace overwrite, no Power BI Service write operations.
3. **No Power BI REST APIs, XMLA write, TOM/AMO live edits, MCP bridges, custom authentication, or unsupported network access** in the MVP. Do not implement these as hidden dependencies.
4. The user retrieves UC metadata **only via the already-approved SQL connection in Power BI Desktop** (or exports a file from an authorized Databricks SQL environment if permitted).
5. The app **does not connect to UC itself**. It generates SQL as text; a human copies/runs that SQL via their existing approved workflow, then pastes/imports the result into the app.
6. The app must work **locally/offline**: no CDN, cloud hosting, tracking, telemetry, external assets, automatic updates, remote AI calls, credential requests, or uploads.
7. A user must review before applying TMDL in Desktop. The tool must never directly modify a production PBIX, workspace, or repo.
8. Target **table and column descriptions only** for the MVP. Do not touch measures, calculations, relationships, security roles, incremental refresh, partition definitions, DAX, Power Query/M expressions, translations, or formatting.
9. Existing nonempty Power BI descriptions must be preserved by default; overwrite is an explicit per-item opt-in or separate guarded setting.
10. If matching or parsing is ambiguous, **skip safely** and explain. Never guess mappings or regenerate the semantic model from scratch.
11. Users may be handling confidential enterprise metadata: no persistence by default, no debug logging of pasted content, no data sent outside the browser. Use safe DOM APIs (no untrusted HTML interpolation), clear data button, and optional local downloads only by user action.

## 3. Intended user experience — one application, four-step wizard

App name: **UC Metadata Sync**. Professional enterprise UI, clean typography, responsive layout, runs in Microsoft Edge on a Windows VDI. Give the user a clear progress bar: **1. Generate SQL → 2. Import metadata → 3. Match model → 4. Generate TMDL**. Also provide a short 'How it works' / privacy and constraints section.

### STEP 1: SQL Query Generator (VERY IMPORTANT NEW REQUIREMENT)

UI controls:
- Scope: radio or segmented control **Catalog**, **Schema**, **Table**.
- `Catalog name` (required for all modes).
- `Schema name` (required for Schema/Table modes; not displayed/used for Catalog mode).
- `Table name` (required for Table mode only).
- Toggle **Include columns with missing descriptions** — **ON by default**. This is required to inventory all columns and leave missing comments untouched. If OFF, filter rows having neither table nor column description, but avoid accidentally losing table-level description records when column comments are null.
- Optional `Include views` toggle, default ON; explain views are also represented in `information_schema.tables` / `columns`.
- Button `Generate SQL`. Render clean, syntax-highlighted, read-only SQL with `Copy SQL` and `Download .sql`.
- Show the number of scopes/tables selected as text, but **do not pretend to know actual row counts** since the tool does not connect to Databricks.
- Contextual instructions: copy SQL → Power BI Desktop → Get Data → Azure Databricks → advanced/native SQL query if allowed → use approved sign-in → load metadata to a temporary *test* model/table. Or use another organization-approved SQL execution surface. Explain that the utility itself executes no SQL.

SQL generation rules:
- Query only UC `information_schema.tables` (alias `t`) and `information_schema.columns` (alias `c`) using a **LEFT JOIN from tables to columns** on `(table_catalog,table_schema,table_name)`.
- Output columns, **in exact fixed order**, so the Step-2 parser can recognize them:
  `table_catalog`, `table_schema`, `table_name`, `table_type`, `table_description`, `column_name`, `ordinal_position`, `data_type`, `column_description`.
- Select `t.comment AS table_description`, `c.comment AS column_description`, `t.table_type`, `c.ordinal_position`, and `c.data_type`.
- Scope:
  - **Catalog**: `FROM <quoted_catalog>.information_schema.tables t`, join its `.columns`, no schema/table filter; optionally exclude the `information_schema` system schema if appropriate, but explain that setting. It means *all visible schemas and tables in ONE specified catalog*, not every catalog in the metastore.
  - **Schema**: same FROM/JOIN, with `WHERE t.table_schema = '<escaped_schema>'`.
  - **Table**: same FROM/JOIN, with `WHERE t.table_schema = '<escaped_schema>' AND t.table_name = '<escaped_table>'`.
- Default sorting `ORDER BY t.table_catalog, t.table_schema, t.table_name, c.ordinal_position`.
- Use correct **Databricks SQL escaping**. Catalog identifiers can contain hyphens (e.g. `d3002-eastus2-aaaz-az4`) and must be enclosed in backticks; escape backticks inside identifiers by doubling them. Escape single quotes in schema/table filter string literals by doubling them. Validate required fields and do not permit unrelated SQL fragments / statements through user input. No shell execution or direct database access.
- The output must be read-only `SELECT`. Never generate `ALTER`, `GRANT`, `CREATE`, `DROP`, INSERT/UPDATE, or data-table SELECTs.
- Keep table descriptions even when all column comments are NULL. For view filtering, use only documented table type values and test them; if uncertain, omit the filter rather than generating invalid SQL.

**Reference query for the proven POC / table scope** (using actual example identifiers):

```sql
SELECT
    t.table_catalog,
    t.table_schema,
    t.table_name,
    t.table_type,
    t.comment AS table_description,
    c.column_name,
    c.ordinal_position,
    c.data_type,
    c.comment AS column_description
FROM `d3002-eastus2-aaaz-az4`.information_schema.tables AS t
LEFT JOIN `d3002-eastus2-aaaz-az4`.information_schema.columns AS c
    ON t.table_catalog = c.table_catalog
   AND t.table_schema = c.table_schema
   AND t.table_name = c.table_name
WHERE t.table_schema = 'powerbi_training'
  AND t.table_name = 'sales_product_summary'
ORDER BY t.table_catalog, t.table_schema, t.table_name, c.ordinal_position;
```

**Schema-scoped difference:** retain the same query but omit the `AND t.table_name = ...` clause.

**Catalog-scoped difference:** retain the same query but omit both the `WHERE t.table_schema = ...` and table filter. The fully qualified FROM remains `<catalog>.information_schema.tables`.

Databricks references for verifying SQL columns and behavior:
- https://learn.microsoft.com/en-us/azure/databricks/sql/language-manual/sql-ref-information-schema
- https://learn.microsoft.com/en-us/azure/databricks/sql/language-manual/information-schema/columns
- https://learn.microsoft.com/en-us/azure/databricks/sql/language-manual/information-schema/tables

### STEP 2: Import UC metadata result

Accept either:
- CSV upload or paste (including UTF-8 BOM, quoted commas, escaped double-quotes and quoted multiline fields), or
- TSV pasted from Power BI Desktop **DAX query view** results, including column headers. The user can query the temporary loaded metadata table using `EVALUATE 'NameOfLoadedMetadataTable'` and copy the result grid. Header names may be qualified (such as `'Metadata'[column_name]` or `Metadata[column_name]`) — normalize safely.

Auto-detect CSV vs TSV; if ambiguous, ask the user to select. Support file import via `<input type="file">`; no file/network upload. Treat empty, `NULL`, and blank description values as 'no UC description', never as an instruction to remove a PBI description. Preserve Unicode and multiline comments.

Required fields: `table_catalog`, `table_schema`, `table_name`, `column_name`, `column_description`. Support `table_description` and the other extra fields as recommended; report missing fields with human-friendly instructions. Table description is repeated on many column rows; deduplicate at `(catalog,schema,table)` and detect/report conflicting duplicate comments. Every column is uniquely keyed by full `(catalog,schema,table,column)`; do not merge similarly named columns from different tables/schemas.

UI: preview grid with the 9 fields, search/filter, summary KPIs (tables, columns, table descriptions present, column descriptions present, missing, duplicates/conflicts). Support a 'Load sample POC data' demo button with clearly labeled dummy values; don't misrepresent them as real user data.

### STEP 3: Map UC objects to Power BI Desktop model / TMDL

Require users to paste or upload **an existing TMDL `createOrReplace` script** obtained by right-clicking the relevant original semantic model table in Power BI Desktop **TMDL view** → `Script TMDL` (or dragging table into TMDL view), not the TMDL definition of the temporary metadata table.

Why: the UC metadata table loaded in step 1 is only a staging table, not the semantic model column properties. The tool's output must target the **original business model** (`sales_product_summary` / renamed model table), and metadata staging table can be removed from the real production model.

Allow an explicit mapping: dropdown/select UC full table name -> Power BI model table name parsed from TMDL. Default to a matching same-name table only if unique and exact (case-insensitive), and ask for explicit confirmation if different. Do **not** infer provenance of renamed model tables from a single name match across multiple UC schemas. No hardcoded catalog or table in production code.

Column matching **inside the confirmed table**:
1. Prefer exact case-insensitive match between UC `column_name` and the TMDL column object's `sourceColumn` property (only for genuine source-backed columns).
2. If absent, consider exact case-insensitive match with the model column name. The table must have been mapped first.
3. For renamed/transformed columns that cannot be safely matched, offer manual one-to-one mapping (UC source column -> Power BI model column) with a confidence badge, conflict detection and explicit user approval.
4. Do not apply descriptions to calculated columns or measures without an approved explicit design change (MVP defaults skip).
5. Never apply a UC description to more than one destination column unless an explicit opt-in design allows it; reject ambiguous 1:N and N:1 matches.

Comparison table: `[ ] Apply`, `UC catalog.schema.table`, `UC column`, `Power BI table`, `Power BI column`, `Current description`, `UC description`, `Match method`, `Status (new / unchanged / conflict / skipped / overwriting)`. Include per-table summary. By default check only unambiguous **new descriptions for currently undocumented Power BI fields**. Existing descriptions may be overwritten only by intentional toggle/action with a warning and a visible count. Skipped fields must be shown, not dropped silently.

### STEP 4: Generate TMDL, review, export

Generate a modified copy of the **input original TMDL**, changing *only* intended table/column descriptions, and preserve everything else byte-for-byte as far as possible (line-ending format, indentation, data types, `sourceColumn`, lineage tags, annotations, measures, partitions, DAX, M expressions, RLS, relationships if they are in the provided input, etc.). Do not synthesize a new incomplete TMDL table definition. If the parser cannot confidently identify an object and its description boundary, skip that object with an error. **Never perform a global regex that could accidentally insert into measure expressions.**

**Important actual TMDL syntax:** Descriptions are documented as `///` documentation comments **directly above object declarations**, not as a generic `description:` property. Multi-line descriptions should be encoded as consecutive `///` lines, with correct indentation. Support existing `///` blocks: preserve them by default; when overwrite is specifically approved, replace only the block that belongs to the targeted object.

Illustration (not a standalone executable full TMDL script):

```tmdl
createOrReplace

    /// This is a sample Product Sales data.
    table sales_product_summary

        /// Partition Date
        column partition_date
            dataType: dateTime
            sourceColumn: partition_date

        /// this is Date Column
        column date
            dataType: dateTime
            sourceColumn: date
```

NOTE: Actual TMDL emitted by Power BI may include many other required properties and nested contents; this example intentionally omits them for readability. **Never replace the full script with this simplified sample**.

TMDL parsing / safety:
- Recognize `createOrReplace`, `table`, `column`, `sourceColumn`, `///` description lines, optional quoted identifiers, table object indentation, multiple tables, and existing properties. Support Unicode, CRLF and LF, comments with punctuation/multiple lines, and appropriate TMDL name escapes. If unsupported syntax exists, fail closed for that object and report it.
- Prefer a syntax-aware editor/line scanner with object nesting + indentation + string block awareness, or an approved parser library where available. Do NOT naively edit DAX or Power Query contents using regex.
- A table-level description is inserted only once above the `table` declaration, even though input metadata repeats it on each column row.
- A column-level description is inserted only above its correct `column` declaration.
- If the exact description is already there, don't create duplicate `///` lines.
- Do not automatically delete any existing description due to a blank UC comment.
- Strict allowlist of changes: only `///` description lines. Produce unified diff / side-by-side before-after preview and object-level audit summary. If the generated diff contains any non-description change (apart from an unavoidable strictly whitespace-local edit adjacent to those comments), flag/block generation.
- Buttons: `Generate`, `Copy Updated TMDL`, `Download .tmdl`, `Download Change Report (.csv/.json)`, `Reset`.
- Show guidance: make a backup of PBIX/PBIP, open Power BI Desktop's **TMDL view**, paste generated **complete table script**, click **Preview** to inspect differences, then **Apply** only when it is clearly description-only. Test in a disposable file first. Continue existing Bitbucket/Ops-controlled UAT→Prod publication process; tool does not publish.

Official Power BI reference: https://learn.microsoft.com/en-us/power-bi/transform-model/desktop-tmdl-view
Microsoft TMDL authoring guidance: https://github.com/microsoft/skills-for-fabric/blob/main/skills/semantic-model-authoring/references/tmdl-guidelines.md

## 4. MVP technical stack / implementation

Optimize for enterprise Windows VDI where installers and npm package downloads might be blocked. **Preferred initial deliverable: a self-contained, zero-dependency, offline `index.html`** (embedded CSS/JavaScript, no backend or server) that the user can open locally in Microsoft Edge. This is a real usable app, not a screenshot or interactive design prototype.

If a single HTML file becomes unwieldy, structure readable source under `src/` and provide a documented, deterministic local build step that produces one standalone distributable HTML file, *but only if dependencies are already available and policy permits*. Do not require a build server, Node at runtime, browser extension, VS Code after packaging, or network access.

Required repository outputs:
- `index.html` (standalone working app)
- `README.md` (setup and exact instructions in Power BI Desktop)
- `tests/` or a built-in deterministic test harness for SQL generation, CSV/TSV parser, mapper, and TMDL patcher
- `fixtures/` containing entirely **synthetic** UC metadata and realistic sample TMDL input; include 1 table and 2-table/2-schema fixtures
- `SECURITY.md` (explicitly state zero network access, no credentials, no API, no auto-publishing; offline privacy limitations)
- `KNOWN_LIMITATIONS.md` (for example, complex TMDL scripts/unsupported multiline structures skip instead of unsafe edits)

Use robust parsing functions separated from UI business logic (even when bundled in one file). UI should have accessible labeled inputs, validation, keyboard controls, error banners, and actionable warnings. UI should scale to thousands of UC columns with search/pagination or virtualized/limited rendering. Display pasted metadata via `textContent`, not `innerHTML`, to avoid script injection. No hardcoded user identifiers, secret URLs, tokens or enterprise names in stored runtime state.

## 5. Required acceptance tests (must be runnable)

### SQL generator
- Catalog-only query includes all visible schema objects in the specified catalog and no schema filter.
- Schema query includes `WHERE t.table_schema = '...'` and no table filter.
- Table query includes both schema and table filters.
- Backtick-containing identifier is escaped correctly; hyphenated catalog remains valid.
- Apostrophes in schema/table inputs become escaped SQL literals.
- Missing required inputs produce errors, not malformed queries.
- Generated output is always SELECT-only, no REST requests, no authentication.
- Default includes all columns, including those without column descriptions.

### Import/parser
- CSV and TSV with headers, quoted commas, Unicode, tabs inside quoted cells, double quotes, multiline description, BOM and Windows line endings.
- Headers may be `column_description`, `[column_description]`, `'Metadata'[column_description]`, or `Metadata[column_description]`.
- NULL/empty comment remains missing; table-level description deduplicated safely.
- No accidental cross-table collisions with same `column_name` in other schemas.
- Duplicate contradictory records flagged, not silently resolved.

### Matching
- `sourceColumn` matches UC raw name even when semantic model column is renamed.
- Exact destination name fallback works when safe.
- Different UC tables with same column name cannot cross-match.
- Manual mappings are respected; ambiguous mappings do not auto-apply.
- Calculated columns and measures do not get modified.

### TMDL patching
- One table description plus 3 new column descriptions inserted correctly; null UC comments ignored.
- Existing PBI description preserved by default; guarded overwrites work only when explicitly selected.
- Newline in UC comments produces multiple `///` lines.
- Existing annotation, DAX measure, partition, and Power Query expression have zero changes.
- Idempotent: running generated output through sync again with same UC metadata changes nothing.
- Output is **full original TMDL with minimal comments inserted**, not a reconstructed model.
- An intentionally complex/unsupported script fails closed with explainable per-object warnings.
- Identical source vs generated TMDL (after stripping ONLY targeted `///` lines) remains byte-for-byte identical.

## 6. Operational / user-facing README instructions

1. In app Step 1 choose Catalog, Schema or Table and enter UC identifiers. Copy generated SQL.
2. In Power BI Desktop, use the existing approved Azure Databricks SQL connection, run that **read-only** query, and load the metadata result to a *temporary staging* table, ideally in a throwaway PBIX.
3. Use DAX query view: `EVALUATE 'Your_Metadata_Staging_Table'` and copy result grid with headers; or export to CSV/TSV by an approved method. Paste/import in app Step 2.
4. In the original business-model PBIX, open Power BI Desktop **TMDL view** and script the actual model table(s) (not the staging metadata table). Paste/upload the full script in Step 3.
5. Explicitly map source UC table(s) to model table(s), validate proposed column matches, check intended updates.
6. Step 4: Generate TMDL; review the diff. Save a backup and apply via Desktop TMDL view's **Preview** then **Apply**. Validate descriptions are populated in Model view.
7. Save changes and follow standard enterprise deployment governance. Nothing is automatically published or overwritten by the tool.

## 7. Scope boundaries and roadmap

MVP does **not** establish a persistent connection or continuous automatic synchronization with Unity Catalog. UC changes require the developer to re-run generated SQL through their approved Desktop query and import the new metadata. This is deliberate to comply with bank policy.

Future optional phases **only if approved**:
- Batch import multiple catalog/schema query results and TMDL files.
- Source-to-model mapping profiles exported/imported locally by the user, with no credentials and no automatic network sync.
- Enhanced diff, approval/reporting and integration into an approved Power BI development workflow.
- No APIs, workspace writes or live model editing unless policies explicitly change and written approvals exist.

## 8. Agent delivery / working instructions

**Do the work now in this VS Code workspace.** First inspect existing files. If none exist, create this tool from scratch. Briefly state an implementation plan, then write working files and tests; do not ask unnecessary questions or deliver only architectural advice. Run whatever local tests are possible; report accurately what passed and what was not validated because this environment cannot connect to the user's UC or Power BI Desktop. Avoid fictitious integrations. Build Step 1 SQL generator and Step 2 parser before tackling the safety-critical TMDL patcher. For any unsupported TMDL syntax, fail closed and present reviewable suggestions, not risky replacements. Deliver a real offline `index.html`, README, fixtures, and tests. Provide exact click-by-click validation steps for the sample `sales_product_summary` POC.

**Definition of Done:** A non-developer can open the offline tool in Edge, choose one of three scopes, copy Databricks metadata SQL, paste/import the returned rows, paste the original Desktop-generated TMDL, explicitly confirm matches, preview a description-only diff and copy a valid updated TMDL script for application in Desktop. Tests cover security guardrails and edge cases. No prohibited APIs or publishing methods are involved.
