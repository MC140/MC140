- 👋 Hi, I’m Manohar Chekka
- 👀 I’m Data Engineer
- 📫 Reach me at : manoharch0698@gmail.com


<!---
MC140/MC140 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
# MIGRATION BLUEPRINT — COMPLETE BUILD SPECIFICATION
# Paste this entire prompt into GitHub Copilot chat before writing any code.

---

## REPO CONTEXT — UNDERSTAND BEFORE TOUCHING ANYTHING

Existing files (DO NOT MODIFY these):
  migration_blueprint_gui.py      ← GUI layer, hands off completely
  build_blueprint_exe.py          ← build script, do not touch
  Build_Blueprint_Exe.bat         ← launcher, do not touch
  Launch_Blueprint.bat            ← launcher, do not touch
  MigrationBlueprint.spec         ← PyInstaller spec, do not touch

Files you WILL work on:
  migration_blueprint_parser.py   ← all parsing logic lives here
  html_blueprint_generator.py     ← all HTML + JSON generation lives here
  test_new_features.py            ← add tests here as you build

Public entry points the GUI already calls (keep these exact signatures):
  parse_twb(file_path: str) -> ParseResult          ← single file
  batch_runner(folder_path: str) -> list[ParseResult] ← batch mode

Do not rename, move, or refactor these. The GUI depends on them.

---

## INPUT — WHAT THE TOOL ACCEPTS

The GUI passes file paths. Parser must handle transparently:

1. .twb  → plain XML, parse with ElementTree directly
2. .twbx → ZIP archive containing a .twb inside. Open with zipfile,
           find the entry ending in .twb, parse from memory stream.
3. Batch → folder path. Discover all .twb and .twbx recursively.
           Parse each independently. Produce one HTML + one JSON per file.
           Also produce batch_summary.html as an overview index.

```python
def load_xml_root(file_path: str):
    """Returns ET root regardless of .twb or .twbx"""
    if file_path.endswith('.twbx'):
        with zipfile.ZipFile(file_path, 'r') as zf:
            twb_name = next(n for n in zf.namelist() if n.endswith('.twb'))
            with zf.open(twb_name) as f:
                return ET.parse(f).getroot()
    return ET.parse(file_path).getroot()
```

---

## CORE PARSING RULES — NEVER VIOLATE THESE

### Rule 1: Caption beats name — always, everywhere, for display.

Every element in Tableau XML has two identifiers:
  name    = internal wire identifier: [Calculation_8472910], [id], fedcba123
  caption = human label the user sees: "YTD Revenue", "Order ID", "Sales DB"

ALWAYS use caption for anything shown in HTML or JSON display fields.
If caption is absent, clean the internal name as fallback:
  - Strip outer brackets
  - Replace underscores with spaces  
  - Title case
  - NEVER show [Calculation_8472910] to the user anywhere

```python
def get_display(element, caption_attr='caption', name_attr='name') -> str:
    cap = (element.get(caption_attr) or '').strip()
    if cap:
        return cap
    return clean_internal(element.get(name_attr, ''))

def clean_internal(name: str) -> str:
    n = re.sub(r'^\[|\]$', '', (name or '').strip())
    n = re.sub(r'^.*\]\.\[', '', n)
    return n.replace('_', ' ').title()
```

### Rule 2: Normalize for matching — never for display.

Normalize only when building dictionary keys or comparing fields.
[id], id, [Orders].[id] must all resolve to the same key.

```python
def normalize(name: str) -> str:
    n = (name or '').strip()
    n = re.sub(r'^\[|\]$', '', n)       # strip brackets
    n = re.sub(r'^.*\]\.\[', '', n)     # strip table prefix
    n = n.lower().strip()
    return n
```

### Rule 3: Store formula_raw and formula_display as separate fields.

formula_raw     = exact string from XML, never modified, never shown in HTML
formula_display = formula_raw with internal refs substituted for captions

Examples:
  formula_raw:     SUM([Calculation_8472910]) / SUM([Calculation_1928374])
  formula_display: SUM([YTD Revenue]) / SUM([Total Cost])

The render_formula() function does the substitution using the registry.
NEVER modify formula_raw. NEVER show formula_raw in the HTML main view.
formula_raw is available in JSON and in the expandable raw view only.

### Rule 4: Build FieldRegistry FIRST before parsing anything else.

Scan ALL datasource columns before touching worksheets or dashboards.
Everything else resolves through the registry. If you parse worksheets
before the registry is complete, field references will fail to resolve.

### Rule 5: Parameters live in a special datasource.

Tableau stores parameters as columns in a datasource named '[Parameters]'.
Formula references look like [Parameters].[Selected Year].
Strip the prefix → resolve 'Selected Year' through the same registry.
Register parameters separately in ParseResult.parameters list.

### Rule 6: Flag — never crash — on unresolved references.

If a formula references [Calculation_999] not in registry:
  - Mark display as: ⚠️ [Calculation_999]
  - Add to registry.unresolved list
  - Never throw KeyError or stop parsing

### Rule 7: First-seen wins for duplicates.

Same internal name can appear multiple times across datasource and
worksheet XML. First registration wins. Do not create duplicate entries.

### Rule 8: Conflict detection.

If two different internal names resolve to the same display caption
across different datasources, both are valid — prefix display name
with datasource name: "Sales DW · Revenue" vs "HR DB · Revenue"

---

## migration_blueprint_parser.py — BUILD ORDER

Implement exactly in this order. One step at a time.

### Step 1: Helper functions

```python
def normalize(name: str) -> str
def get_display(element, caption_attr='caption', name_attr='name') -> str
def clean_internal(name: str) -> str
def load_xml_root(file_path: str)  # handles .twb and .twbx
def detect_calc_type(formula: str) -> str
    # Returns: 'LOD_FIXED' | 'LOD_INCLUDE' | 'LOD_EXCLUDE' |
    #          'TABLE_CALC' | 'AGGREGATE' | 'ROW_LEVEL' | 'UNKNOWN'
    # LOD: formula contains { FIXED | { INCLUDE | { EXCLUDE
    # Table calc: RUNNING_SUM | WINDOW_ | RANK | INDEX | PREVIOUS_VALUE
    #             FIRST() | LAST() | LOOKUP | TOTAL(
    # Aggregate: starts with or contains SUM( AVG( COUNT( MIN( MAX( COUNTD(
    # Row-level: anything else (string funcs, date funcs, IF/CASE on dims)

def dax_complexity(calc_type: str) -> str
    # LOD_* → 'High'
    # TABLE_CALC → 'Medium'
    # AGGREGATE → 'Low'
    # ROW_LEVEL → 'Low'
    # UNKNOWN → 'Medium'
```

### Step 2: FieldRegistry class

```python
@dataclass
class FieldEntry:
    internal_name: str      # exactly as in XML: [Calculation_123]
    display_name: str       # caption or clean_internal fallback
    datatype: str
    role: str               # 'dimension' | 'measure' | 'unknown'
    formula_raw: str        # untouched from XML, empty string if not calc
    formula_display: str    # populated after render_formula() pass
    calc_type: str          # from detect_calc_type()
    dax_complexity: str     # Low / Medium / High
    datasource: str         # display name of parent datasource
    dependencies: list[str] # display names of fields this formula refs
    is_parameter: bool

class FieldRegistry:
    by_internal: dict[str, FieldEntry]    # raw internal name → entry
    by_normalized: dict[str, FieldEntry]  # normalized key → entry
    conflicts: list[dict]
    unresolved: list[str]

    def register(self, internal, caption, datatype, role,
                 formula_raw, datasource, is_parameter=False)
        # First-seen wins. Detect conflicts. Add to both dicts.

    def resolve(self, any_reference: str) -> FieldEntry | None
        # Accepts: [id], id, [Table].[id], Calculation_123
        # Normalize and look up. Return None if not found.

    def display_name(self, any_reference: str) -> str
        # resolve() and return display_name, or clean_internal fallback
```

### Step 3: render_formula(raw: str, registry: FieldRegistry) -> str

Use regex to find all [FieldRef] patterns in the formula string.
For each match call registry.display_name() and substitute.
Do NOT substitute string literals in quotes like 'year' 'month' 'day'.
Do NOT substitute inside DATEPART/DATENAME string arguments.
Return the rendered string. Never modify the input string.

After building registry, iterate all FieldEntry objects and populate
formula_display and dependencies list using this function.

### Step 4: parse_datasources(root, registry) -> list[dict]

For each <datasource> (skip the Parameters datasource here):
  display_name:    get_display(ds)
  internal_name:   ds.get('name')
  connection_type: ds/connection @class or @dbclass attribute
  server:          connection @server
  database:        connection @dbname
  schema:          connection @schema (may be absent)
  is_extract:      True if <extract> child exists
  custom_sql:      text of <relation type='text'> if present

Register all <column> children into FieldRegistry here.

### Step 5: parse_parameters(root, registry) -> list[dict]

Find datasource where name='[Parameters]' or caption='Parameters'.
For each <column> child:
  name, datatype, allowable_values_type (list/range/all),
  values list (from <members> children),
  default_value (column @value),
  used_in_sheets: [] ← populate during parse_worksheets pass

Register each into registry with is_parameter=True.

### Step 6: parse_worksheets(root, registry) -> list[dict]

For each <worksheet>:
  internal_name:    ws.get('name')
  display_name:     get_display(ws)
  is_background:    True if ws has @hidden='true' or name starts with
                    common BG prefixes (BG_, bg_, Background)

  For shelf contents look in <view><datasources> and <table>:
    rows: fields in <rows> element text, split and resolved
    cols: fields in <cols> element text, split and resolved
    marks (color/size/label/detail/tooltip): from <encoding> children
    filters: from <filter> children — field name + condition summary
    parameters_used: any resolved field where is_parameter=True

  chart_type: infer from mark type attribute on <mark> element:
    bar, line, area, circle, square, text, map, pie, gantt,
    shape, polygon, automatic → map to readable label

  has_table_calc: True if any field in sheet has calc_type TABLE_CALC
  has_lod: True if any field in sheet has calc_type LOD_*

  pbi_visual_suggestion: map chart_type →
    bar → 'Clustered Bar / Stacked Bar Chart'
    line → 'Line Chart'
    area → 'Area Chart'
    circle/square → 'Scatter Plot'
    text → 'Table / Matrix Visual'
    map → 'ArcGIS Map / Filled Map'
    pie → 'Pie Chart (consider Bar Chart)'
    automatic → 'Inspect shelf — likely Bar or Line'

  When a parameter is found in filters, add this sheet name to
  that parameter's used_in_sheets list.

### Step 7: parse_dashboards(root, registry) -> list[dict]

For each <dashboard>:
  internal_name, display_name (get_display)
  width, height: from @width @height attributes
  layout_type: scan <zones>/<zone> children
    if any zone has @x @y @w @h absolute attrs → 'Floating'
    if mix → 'Mixed'
    else → 'Tiled'
  sheets_contained: zone @name values where zone type is worksheet
  has_device_layout: True if <device-layouts> child exists
  actions: from <actions><action> children
    type: 'filter' | 'highlight' | 'url' | 'goto-sheet'
    source_sheet, target_sheet, url (for url type)
    plain_english: generate a 1-sentence description

### Step 8: build_migration_flags(registry, worksheets, dashboards) -> list[dict]

Auto-generate flags. Each flag:
  severity: 'red' | 'amber' | 'green'
  category: string label
  affected_object: field name / sheet name / dashboard name
  description: plain English sentence

Rules:
  RED flags:
    - Any field with calc_type LOD_FIXED, LOD_INCLUDE, LOD_EXCLUDE
    - Any datasource where custom_sql is not empty
    - Any unresolved field reference in registry.unresolved
    - Any field with formula_display still containing [Calculation_NNNNN]

  AMBER flags:
    - Any sheet where has_table_calc is True
    - Any dashboard where layout_type is 'Floating' or 'Mixed'
    - Any parameter with allowable_values_type 'range' (What-If needed)
    - Any parameter used as a field selector (string type, list of field names)
    - Any datasource with is_extract False (DirectQuery considerations)
    - registry.conflicts entries

  GREEN flags:
    - Aggregate-only measures (LOW complexity, just count them as one flag)
    - Tiled dashboards (just note they are straightforward)

### Step 9: ParseResult dataclass + parse_twb() entry point

```python
@dataclass
class ParseResult:
    meta: dict              # workbook_name, source_file, file_type,
                            # parsed_at, complexity_score, flag_counts
    datasources: list[dict]
    fields: list[FieldEntry]
    parameters: list[dict]
    worksheets: list[dict]  # active + background, is_background flag
    dashboards: list[dict]
    migration_flags: list[dict]
    registry: FieldRegistry
    parse_issues: list[str] # from validate()

def parse_twb(file_path: str) -> ParseResult:
    root = load_xml_root(file_path)
    registry = FieldRegistry()

    # PHASE 1: registry first
    datasources = parse_datasources(root, registry)
    parameters  = parse_parameters(root, registry)

    # PHASE 2: render formulas now registry is complete
    for entry in registry.by_internal.values():
        if entry.formula_raw:
            entry.formula_display = render_formula(entry.formula_raw, registry)
            entry.dependencies = extract_dependencies(entry.formula_raw, registry)
        else:
            entry.formula_display = ''
            entry.dependencies = []

    # PHASE 3: parse everything else
    worksheets  = parse_worksheets(root, registry)
    dashboards  = parse_dashboards(root, registry)
    flags       = build_migration_flags(registry, worksheets, dashboards)
    issues      = validate(registry, worksheets, dashboards)

    complexity  = compute_complexity(registry, dashboards)
    flag_counts = count_flags(flags)

    meta = {
        'workbook_name': get_display(root) or Path(file_path).stem,
        'source_file':   str(file_path),
        'file_type':     'twbx' if file_path.endswith('.twbx') else 'twb',
        'parsed_at':     datetime.now().isoformat(timespec='seconds'),
        'complexity_score': complexity,
        'flag_counts':   flag_counts
    }

    return ParseResult(meta, datasources,
                       list(registry.by_internal.values()),
                       parameters, worksheets, dashboards, flags,
                       registry, issues)

def batch_runner(folder_path: str) -> list[ParseResult]:
    files = list(Path(folder_path).rglob('*.twb')) + \
            list(Path(folder_path).rglob('*.twbx'))
    results = []
    for f in files:
        try:
            results.append(parse_twb(str(f)))
        except Exception as e:
            # produce a minimal ParseResult with error flag, don't stop batch
            results.append(make_error_result(str(f), str(e)))
    return results
```

### Step 10: validate(registry, worksheets, dashboards) -> list[str]

Returns list of human-readable issue strings. Run after full parse.
Check:
  - Any display name still matches r'\[Calculation_\d+\]'
  - Any display name is blank or None
  - Duplicate normalized keys in registry
  - Empty formula_raw on a calculated field (formula element existed but empty)
  - registry.conflicts and registry.unresolved
  - Any worksheet with no rows and no cols (empty shelf — likely parse miss)

---

## html_blueprint_generator.py — BUILD ORDER

### Step 11: generate_json(result: ParseResult) -> str

Returns JSON string. Structure exactly:

```json
{
  "meta": {
    "workbook_name": "",
    "source_file": "",
    "file_type": "twb|twbx",
    "parsed_at": "",
    "complexity_score": "Low|Medium|High|Very High",
    "flag_counts": { "red": 0, "amber": 0, "green": 0 }
  },
  "datasources": [{
    "internal_name": "", "display_name": "",
    "connection_type": "", "server": "", "database": "",
    "schema": "", "is_extract": false, "custom_sql": ""
  }],
  "fields": [{
    "internal_name": "", "display_name": "",
    "datatype": "", "role": "", "datasource": "",
    "is_calculated": false, "is_parameter": false,
    "formula_raw": "", "formula_display": "",
    "calc_type": "LOD_FIXED|LOD_INCLUDE|LOD_EXCLUDE|TABLE_CALC|AGGREGATE|ROW_LEVEL|null",
    "dax_complexity": "Low|Medium|High",
    "dependencies": []
  }],
  "parameters": [{
    "internal_name": "", "display_name": "",
    "datatype": "", "allowable_values_type": "list|range|all",
    "values": [], "default_value": "", "used_in_sheets": []
  }],
  "worksheets": [{
    "internal_name": "", "display_name": "",
    "is_background": false,
    "chart_type": "", "pbi_visual_suggestion": "",
    "rows": [], "cols": [],
    "marks": { "color": [], "size": [], "label": [],
               "detail": [], "tooltip": [] },
    "filters": [{ "field": "", "condition_summary": "" }],
    "parameters_used": [],
    "has_table_calc": false, "has_lod": false
  }],
  "dashboards": [{
    "internal_name": "", "display_name": "",
    "width": 0, "height": 0,
    "layout_type": "Tiled|Floating|Mixed",
    "sheets_contained": [],
    "actions": [{ "type": "", "source": "",
                  "target": "", "plain_english": "" }],
    "has_device_layout": false
  }],
  "migration_flags": [{
    "severity": "red|amber|green",
    "category": "", "affected_object": "", "description": ""
  }],
  "parse_issues": []
}
```

### Step 12: generate_html(result: ParseResult) -> str

Single self-contained HTML file. All CSS and JS inline.
Zero external dependencies. Works offline.

SECTIONS — render in this exact order:

  [NAV] Sticky top navigation bar with anchor links to all sections.
  Links: Summary | Roadmap | Source Connections | Data Model |
         Fields Used | Calculations Gallery | Dependencies |
         Dep. Diagram | Metadata | Worksheets | BG Sheets |
         Dashboards | Parameters | ⚠️ Flags | Metrics Inventory

  [1] EXECUTIVE SUMMARY (default: EXPANDED)
      - Workbook name, file type, file size, parsed timestamp, Tableau build
      - Stat cards: # datasources, # worksheets, # dashboards,
                    # parameters, # calc fields, # LOD expressions,
                    # table calcs, # background sheets
      - Complexity badge: Low (green) / Medium (amber) /
                          High (red) / Very High (red pulse)
        Complexity logic:
          LOD count ≥ 5 OR table calc count ≥ 3 OR datasources ≥ 3
          OR floating dashboard exists → High
          Any LOD OR table calc → Medium
          Neither → Low
          LOD ≥ 10 → Very High
      - Flag summary bar: X 🔴 High Effort | X 🟡 Medium | X 🟢 Low
      - parse_issues list if non-empty (show as warning box)

  [2] MIGRATION ROADMAP (default: EXPANDED)
      Generate steps dynamically based on what was found:
      Always include: connect sources, build model, build visuals,
                      rebuild dashboards
      Conditionally include:
        If LOD count > 0 → step: "Translate N LOD expressions → DAX"
        If table calc > 0 → step: "Translate N table calcs → DAX"
        If parameters > 0 → step: "Map N parameters → Slicers/What-If"
        If floating dashboard → step: "Manual layout: N floating dashboard(s)"
        If custom SQL → step: "Replicate N custom SQL queries in Power Query"
      Each step has: number, title, description, effort badge

  [3] SOURCE CONNECTIONS (default: COLLAPSED)
      One collapsible card per datasource.
      Show: display name, internal name (as code), connection type,
            server, database, schema, Live/Extract badge,
            custom SQL block if present (in code block),
            Power BI action note (DirectQuery vs Import decision)

  [4] DATA MODEL — MIGRATION FIELD SUMMARY (default: COLLAPSED)
      Searchable table. Columns:
        # | Display Name | Internal Name | Data Type | Role |
        Datasource | Source Column | Notes (aliases, groups, etc.)

  [5] FIELDS USED IN WORKSHEETS (default: COLLAPSED)
      Searchable table. Columns:
        Field Name | Type | Role | Used In Sheets | Shelf Position

  [6] CALCULATIONS GALLERY (default: COLLAPSED)
      Searchable table. Columns:
        # | Display Name | Formula (display) | Type | DAX Complexity | Dependencies
      LOD rows: red background tint
      Table calc rows: amber background tint
      Clicking row name expands inline sub-row showing:
        formula_raw (labeled clearly as "Raw — for reference only")
        formula_display (human readable)
        full dependency chain
      Note at bottom: "Raw formulas preserved in JSON file"

  [7] DEPENDENCIES (default: COLLAPSED)
      Searchable table. Columns:
        Field | Type | Depends On | Used By (sheets)

  [8] DEPENDENCY DIAGRAM (default: COLLAPSED)
      Visual node layout (HTML/CSS only, no external libs):
      Three rows: Source Fields → Calculated Fields → Sheets
      LOD nodes styled red, Table Calc nodes styled amber,
      regular calcs styled purple, source fields styled default.

  [9] METADATA VIEW (default: COLLAPSED)
      Raw metadata table for debugging:
        Internal Name | Caption (raw) | Resolved Display |
        Datatype | Role | Datasource
      Show badge "Fallback" if caption was absent and clean_internal was used.
      This section helps verify parse correctness.

  [10] WORKSHEETS — ACTIVE (default: COLLAPSED)
       One collapsible card per worksheet where is_background=False.
       Each card shows:
         Sheet name (display), chart type badge, PBI visual suggestion badge
         Rows shelf: tags for each resolved field
         Cols shelf: tags for each resolved field
         Marks (color/size/label/detail/tooltip): tags
         Filters: field name + condition summary
         Parameters used: links to Parameters section
         ⚠️ badges if has_table_calc or has_lod
         Migration notes box (auto-generated based on flags)

  [11] BACKGROUND SHEETS (default: COLLAPSED)
       Same card format as worksheets.
       Info box explaining BG sheets may not need direct PBI equivalents.

  [12] DASHBOARDS (default: COLLAPSED)
       One collapsible card per dashboard.
       Each card shows:
         Display name, dimensions (W×H), layout type badge
         Sheets contained (listed)
         Actions (each action as a flag-item card in plain English)
         ⚠️ Floating layout warning box if applicable
         Device layout note if present

  [13] PARAMETERS (default: COLLAPSED)
       Table. Columns:
         # | Display Name | Data Type | Values Type | Values/Range |
         Default | Used In Sheets | PBI Equivalent
       PBI Equivalent logic:
         list type → Slicer
         range type → What-If Parameter
         string list of field names → Field Parameter (Preview feature)

  [14] ⚠️ MIGRATION FLAGS (default: EXPANDED — always open)
       This is the developer's action checklist. Must be prominent.
       Grouped sections: 🔴 High Effort first, 🟡 Medium, 🟢 Low
       Each flag item shows:
         severity icon, category, affected object, description
       Count header per group: "🔴 High Effort (N items)"

  [15] METRICS INVENTORY (default: COLLAPSED)
       Flat searchable table of ALL measures (role='measure').
       Columns:
         # | Measure Name | Formula (display) | LOD? | Table Calc? |
         DAX Complexity | Datasource

INTERACTIVITY (all inline JS, no frameworks):
  - Toggle collapse: click section header
  - Search/filter: input boxes on tables 4, 5, 6, 7, 9, 15
  - Inline row expand: calc gallery row click shows formula detail
  - Sticky nav: position:sticky with backdrop blur
  - Print: window.onbeforeprint expands all sections
  - Batch summary page (batch_summary.html):
      Table of all workbooks processed, columns:
        Workbook Name | File | Complexity | 🔴 | 🟡 | 🟢 | Report Link

DESIGN:
  - Dark theme. Background #0d1117, surface #161b22, border #30363d
  - Text #e6edf3, muted #8b949e, accent #58a6ff
  - Red #f85149, Amber #d29922, Green #3fb950
  - Font: IBM Plex Mono for code/formulas, IBM Plex Sans for UI
    (load from Google Fonts)
  - Border-radius 8px on cards, 6px on badges
  - Zero external JS/CSS dependencies beyond Google Fonts

### Step 13: write_outputs(result: ParseResult, output_dir: str)

```python
def write_outputs(result: ParseResult, output_dir: str):
    name = Path(result.meta['source_file']).stem
    folder = Path(output_dir) / name
    folder.mkdir(parents=True, exist_ok=True)

    html_path = folder / f"{name}.html"
    json_path = folder / f"{name}.json"

    html_path.write_text(generate_html(result),  encoding='utf-8')
    json_path.write_text(generate_json(result),  encoding='utf-8')

    return str(html_path), str(json_path)
```

For batch, also call write_batch_summary(results, output_dir) which
writes batch_summary.html directly into output_dir (not a subfolder).

---

## PYTHON ENVIRONMENT

Python 3.10+. Standard library only:
  xml.etree.ElementTree, zipfile, json, re, dataclasses,
  pathlib, datetime, collections, html

No pip installs. No pandas, no lxml, no BeautifulSoup.

---

## HOW TO PROCEED

Start with Step 1 only — the helper functions.

Before writing any code for each step, tell me:
  1. What functions/classes you are writing
  2. What each takes as input and returns
  3. Any assumptions you are making about the XML structure

Then write the code.

After each step say exactly:
  "Step N complete. Ready for Step N+1 on your confirmation."

Do not proceed until I reply "yes" or "proceed".
Do not combine steps.
Do not touch migration_blueprint_gui.py under any circumstances.
