- 👋 Hi, I’m Manohar Chekka
- 📫 Reach me at : manoharch0698@gmail.com


<!---
MC140/MC140 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
Assess the Power BI semantic model under input/YourModel.

Do not modify the original model.

Generate the complete assessment package under output/YourModel, including:

- model inventory
- grain analysis
- table and column classifications
- modelling findings
- proposed star schema
- source-to-target mapping
- relationship plan
- measure impact analysis
- report impact analysis
- validation plan
- assumptions requiring business confirmation
- Mermaid model diagram


Update the current Power BI Star Schema Assessor project to support the final required output format described below.

Do not only explain the changes. Modify the agent instructions, workspace instructions, standards, templates, input README and root README.

The product should now be named:

Power BI Star Schema Converter

Its main purpose is to take a Power BI PBIP project from the input folder and create a complete parallel star-schema-converted PBIP candidate under the output folder.

The converted PBIP must have the same project structure as the input PBIP and must be directly openable by double-clicking its .pbip file.

The conversion output must also contain one self-contained HTML report. Do not generate multiple Markdown, CSV or Mermaid report files in the conversion output.

# 1. Required input structure

The agent must accept an input such as:

input/
└── SalesModel/
    ├── SalesModel.pbip
    ├── SalesModel.Report/
    ├── SalesModel.SemanticModel/
    └── .gitignore

The source folder may also contain additional valid PBIP project files.

Treat everything under input as read-only.

# 2. Required output structure

For an input folder named:

input/SalesModel/

create:

output/SalesModel_StarSchema/

The converted output must look like:

output/
└── SalesModel_StarSchema/
    ├── SalesModel.pbip
    ├── SalesModel.Report/
    ├── SalesModel.SemanticModel/
    ├── .gitignore
    └── conversion-report.html

Important:

- Add _StarSchema only to the outer output folder name.
- Preserve the original internal .pbip filename.
- Preserve the original .Report folder name.
- Preserve the original .SemanticModel folder name.
- Preserve all valid relative path relationships.
- Do not add _StarSchema to internal filenames unless doing so is explicitly required and every related reference is updated.
- The user must be able to double-click output/SalesModel_StarSchema/SalesModel.pbip and attempt to open the converted project in Power BI Desktop.

Do not create this structure:

output/SalesModel/converted-pbip/...

Instead, create the converted PBIP project directly as:

output/SalesModel_StarSchema/

# 3. Copy-first conversion

Before making any model changes:

1. Validate that the source contains a .pbip file.
2. Identify its referenced report folder.
3. Identify the report’s referenced semantic-model folder.
4. Copy the complete source PBIP project into the new output folder.
5. Perform all conversion work only inside the copied project.
6. Never edit files under input.
7. Never overwrite an existing output folder without explicit user approval.
8. If output/SalesModel_StarSchema already exists, stop and ask whether to:
   - replace it;
   - create SalesModel_StarSchema_2; or
   - cancel.

Copy all project definitions required to open the PBIP.

Known transient files such as these do not need to be copied:

- .pbi/cache.abf
- .pbi/localSettings.json
- temporary lock files
- operating-system temporary files

Do not omit valid report, semantic-model, query, resource, theme or project-definition files.

# 4. Supported automatic conversion

Automatic conversion should primarily support semantic models stored in TMDL format:

SalesModel.SemanticModel/
├── definition.pbism
└── definition/
    ├── model.tmdl
    ├── relationships.tmdl
    ├── tables/
    ├── roles/
    ├── cultures/
    └── other supported TMDL objects

A model.bim project may be assessed, but if safe automatic conversion cannot be completed, preserve the copied PBIP and report:

Conversion blocked pending TMDL conversion.

Do not create invalid model.bim or TMDL content merely to produce an output.

# 5. Conversion workflow

Perform the following workflow.

## Phase 1: Source inventory

Read and inventory:

- PBIP project file
- report definition
- semantic-model definition
- tables
- columns
- measures
- calculated columns
- calculated tables
- relationships
- hierarchies
- partitions
- Power Query expressions
- storage modes
- calculation groups
- perspectives
- translations
- roles and RLS
- format strings
- descriptions
- synonyms
- display folders
- sort-by-column settings
- hidden properties
- data categories
- aggregation metadata
- incremental-refresh metadata
- report pages
- visuals
- slicers
- filters
- tooltips
- bookmarks
- drill-through configurations
- conditional formatting
- report field references

## Phase 2: Grain analysis

For each possible transactional table:

- determine the likely grain;
- identify grain-defining columns;
- identify mixed-grain risks;
- assign High, Medium or Low confidence;
- mark unresolved grain as NEEDS BUSINESS CONFIRMATION.

Do not automatically convert a table when its grain cannot be determined with sufficient confidence.

## Phase 3: Proposed star schema

Identify:

- transaction facts;
- snapshot facts;
- dimensions;
- conformed dimensions;
- role-playing dimensions;
- bridge tables;
- factless facts;
- degenerate dimensions;
- aggregate tables;
- disconnected parameter tables;
- measures tables;
- dedicated date dimensions.

Prefer:

- one-to-many relationships;
- dimension-to-fact single-direction filtering;
- explicit measures;
- one dedicated conformed date dimension;
- active relationship for the primary date;
- inactive relationships for secondary date roles;
- descriptive attributes in dimensions;
- numeric events at the correct fact-table grain.

Do not automatically classify every text column as a dimension attribute or every numeric column as a fact.

## Phase 4: Convert only the copied semantic model

Within the copied semantic model:

- create the proposed fact and dimension tables;
- create relationships;
- preserve the determined business grain;
- preserve supported metadata;
- rewrite supported measures;
- preserve original DAX in the HTML conversion report;
- record every object created, changed, retained or blocked;
- never silently delete an object.

When using Power Query reference-query conversion:

- retain the original source connection;
- do not change credentials;
- do not expose credentials in output;
- preserve query folding where possible;
- do not use Table.Distinct blindly;
- validate business-key uniqueness before deduplication;
- do not silently remove duplicate business keys;
- do not change the fact grain;
- do not aggregate rows without confirmed business logic.

For large DirectQuery, composite or enterprise models, do not force an unsafe Power Query conversion. A copied PBIP may be created with the original model preserved, but the HTML status must state that upstream implementation is required.

# 6. Original flat-table treatment

The converted project must prioritize opening successfully over silently deleting source objects.

Use this sequence:

1. Create proposed fact and dimension tables.
2. Create and validate proposed relationships.
3. Remap supported measures.
4. Analyze report references.
5. Remap unambiguous report fields.
6. Check whether the original flat table still has dependencies.

If the original flat table has no remaining dependencies:

- hide it;
- disable load only when technically safe;
- or retain it as a staging/shared query when the model format supports this safely.

If unresolved dependencies remain:

- retain the original table as a hidden compatibility table;
- do not delete it;
- classify the result as Partially converted candidate generated;
- list the unresolved dependencies in the HTML report.

A model containing a retained compatibility table must not be described as a completed pure star schema.

# 7. Preserve semantic-model objects

Preserve or explicitly document the treatment of:

- measures;
- calculated columns;
- calculated tables;
- calculation groups;
- hierarchies;
- field parameters;
- RLS roles and expressions;
- perspectives;
- translations;
- partitions;
- incremental refresh;
- storage modes;
- aggregations;
- descriptions;
- synonyms;
- display folders;
- format strings;
- hidden settings;
- sort-by-column;
- data categories;
- summarization settings;
- source-column mappings.

Never silently drop unsupported metadata.

# 8. Report remapping

Inspect the copied report definition.

For every field reference affected by the conversion, classify it as:

- Automatically preserved
- Automatically remapped
- Manual remapping required
- Behaviour may change
- Unable to assess

Automatically remap a report field only when:

- there is exactly one source field;
- there is exactly one target field;
- the target has equivalent business meaning;
- the target data type is compatible;
- the mapping does not change grain;
- the relationship path supports the same filtering behaviour.

Track affected:

- pages;
- visuals;
- visual fields;
- slicers;
- page filters;
- report filters;
- visual filters;
- drill-through fields;
- tooltips;
- bookmarks;
- conditional formatting;
- sort fields;
- interactions.

Do not delete a visual because remapping is uncertain.

Preserve it and report the manual action required.

# 9. Direct-open PBIP validation

Before completing, validate the copied project structure.

Check:

1. The .pbip file exists in the output project root.
2. The .pbip file points to an existing copied report folder.
3. The report definition.pbir exists.
4. The report’s datasetReference points to the correct copied semantic-model folder.
5. The semantic-model definition.pbism exists.
6. The expected TMDL definition folder or model.bim exists.
7. All JSON files edited by the agent remain syntactically valid.
8. All relative paths resolve inside the copied output project.
9. No input path is referenced accidentally.
10. No report or semantic-model folder references the source under input.
11. No output definition contains temporary workspace paths.
12. Every modified TMDL object is structurally consistent with the surrounding project format.

Do not state that Power BI Desktop opening was validated unless Power BI Desktop was actually used successfully.

Use this language:

The converted PBIP candidate has been structurally generated and is ready for Power BI Desktop open, refresh and regression testing.

# 10. Single HTML conversion report

Create exactly one report file:

output/<SourceFolderName>_StarSchema/conversion-report.html

Do not create these conversion-report artifacts:

- executive-summary.md
- inventory.md
- grain-analysis.md
- findings.md
- proposed-star-schema.md
- relationship-plan.md
- measure-impact.md
- report-impact.md
- validation-plan.md
- mapping.csv
- assumptions.md
- diagram.mmd

All of that information must be consolidated into conversion-report.html.

The HTML must be fully self-contained:

- inline CSS;
- inline JavaScript;
- no CDN;
- no internet dependency;
- no external stylesheet;
- no external JavaScript;
- no external image requirement;
- no external font requirement.

It must open locally in a browser by double-clicking it.

# 11. HTML design

The HTML should look like a professional Power BI governance and conversion report.

Create a top header containing:

- source project name;
- source path;
- converted project path;
- conversion date and time;
- semantic-model format;
- conversion result status;
- overall confidence;
- number of tables before and after;
- number of measures;
- report-remapping summary;
- unresolved issue count.

Create clickable tabs for:

1. Executive Summary
2. Source Model
3. Grain Analysis
4. Proposed Star Schema
5. Conversion Changes
6. Table and Column Mapping
7. Relationships
8. Measures and DAX
9. Report Remapping
10. Validation
11. Warnings and Manual Actions
12. Complete Object Inventory

Each tab must work without reloading the page.

Include:

- status badges;
- summary cards;
- sortable or clearly structured tables;
- severity indicators;
- expandable details;
- original and proposed DAX blocks;
- source and target object names;
- confidence levels;
- reasons for changes;
- unresolved business confirmations;
- evidence used;
- validation checklist;
- Power BI Desktop testing checklist.

Where useful, include an inline SVG star-schema diagram.

Do not depend on Mermaid from a CDN.

The report should have:

- a Print / Save as PDF button;
- an Expand All button;
- a Collapse All button;
- a text search box;
- responsive layout;
- print styling that displays every tab as a continuous report.

When printing, all hidden tab content must become visible.

# 12. HTML security and privacy

Do not include:

- passwords;
- access tokens;
- credential values;
- complete sensitive connection strings;
- personally identifiable sample data;
- cached model data.

Connection information must be masked when displayed.

Escape model metadata before inserting it into HTML to prevent it from being interpreted as executable HTML.

# 13. Conversion statuses

Use only:

- Converted candidate generated
- Partially converted candidate generated
- Conversion blocked pending confirmation
- Conversion not supported for this model

Do not use:

- Successfully converted
- Fully converted
- Production ready
- Fully validated

unless actual Power BI Desktop opening, refresh and regression-test evidence has been supplied.

# 14. Agent response after conversion

After creating the converted candidate, respond with:

- source project assessed;
- converted output folder;
- exact .pbip file to open;
- exact HTML report to open;
- conversion status;
- major unresolved items;
- confirmation that input files were not modified.

Keep the chat response brief because the detailed information is in conversion-report.html.

# 15. Default behaviour

When the user asks to convert a model, generate:

1. the complete parallel PBIP candidate;
2. conversion-report.html.

Assessment-only mode may remain available only when the user explicitly requests assessment without conversion.

# 16. Tool configuration

Review the custom agent’s tools.

The agent needs sufficient workspace file capabilities to:

- read files;
- search files;
- create folders;
- copy the PBIP project;
- create and edit files under output;
- validate generated JSON and text definitions.

Do not give it:

- Power BI Service publishing;
- Fabric deployment;
- database write access;
- external API access;
- MCP access;
- credential access.

If a terminal tool is required solely to copy folders or validate local files, allow only local workspace operations and explicitly prohibit:

- network commands;
- package installation;
- deployment;
- publishing;
- credential commands;
- commands that modify input.

Use the exact valid built-in tool names supported by the installed VS Code version.

# 17. Documentation updates

Update README.md to show the final workflow:

1. Save a Power BI model as PBIP with TMDL.
2. Place a copy under input/<ModelName>/.
3. Select the Power BI Star Schema Converter agent.
4. Request conversion.
5. Open output/<ModelName>_StarSchema/<ModelName>.pbip.
6. Open output/<ModelName>_StarSchema/conversion-report.html.
7. Validate opening, refresh, measures, RLS and visuals in Power BI Desktop.

Update input/README.md with the same folder examples.

Remove documentation suggesting that the normal conversion output consists of multiple Markdown or CSV reports.

Templates may remain in the repository for internal guidance, but the runtime conversion output must contain only:

- the copied and converted PBIP project files;
- conversion-report.html.

# 18. Final verification

After making the changes:

1. Verify the custom agent file is valid.
2. Verify the new name is Power BI Star Schema Converter.
3. Verify input remains read-only.
4. Verify output structure matches the required format.
5. Verify conversion-report.html is the only consolidated report artifact.
6. Verify no instructions still require multiple Markdown output reports.
7. Verify the internal PBIP project names remain unchanged by default.
8. Verify the output parent folder receives the _StarSchema suffix.
9. Verify the README contains the exact run prompt below.
10. List every repository file modified.