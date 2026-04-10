- 👋 Hi, I’m Manohar Chekka
- 👀 I’m Data Engineer
- 📫 Reach me at : manoharch0698@gmail.com


<!---
MC140/MC140 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
## CHAIN FIX — Critical Parsing Corrections
## These are targeted fixes only. Do not rewrite anything not listed here.
## Fix each issue exactly as described. Nothing else changes.

---

## FIX 1: Filter Tableau Special/System Fields

Problem: Fields like ':Measure Names', ':Measure Values', ':Row ID',
':Latitude', ':Longitude' are Tableau built-ins appearing as real fields.

In parse_datasources() inside the column registration loop, add this
filter BEFORE calling registry.register():

```python
TABLEAU_SYSTEM_FIELDS = {
    ':measure names', ':measure values', ':row id',
    ':latitude (generated)', ':longitude (generated)',
    'measure names', 'measure values', 'number of records',
    ':name', ':path'
}

def is_system_field(name: str, caption: str) -> bool:
    check = (caption or name or '').strip().lower()
    if check.startswith(':'):
        return True
    if check in TABLEAU_SYSTEM_FIELDS:
        return True
    return False
```

Skip registration entirely if is_system_field() returns True.
Also skip these in generate_html() field tables and JSON fields array.

---

## FIX 2: Worksheet Count — Only Count Real User Worksheets

Problem: 226 worksheets because parser counts ALL <worksheet> elements
including tooltip sheets, system sheets, and story point containers.

In parse_worksheets(), filter out non-user sheets:

```python
def is_user_worksheet(ws_element) -> bool:
    name = ws_element.get('name', '')
    
    # Tooltip sheets: Tableau auto-names these
    if name.startswith('__Tooltip'):
        return False
    if name.startswith('(Tooltip)'):
        return False
    
    # Empty name
    if not name.strip():
        return False
    
    # Check for hidden attribute — but do NOT exclude based on hidden alone
    # because user may have intentionally hidden a sheet (Background sheet)
    # Hidden sheets go into is_background=True bucket, not excluded entirely
    
    return True
```

Apply this filter: only process ws elements where is_user_worksheet() is True.
Hidden user worksheets → set is_background=True but still parse them.

Background sheet detection (separate from hidden):
```python
def is_background_sheet(ws_element, dashboard_sheet_names: set) -> bool:
    name = ws_element.get('name', '')
    caption = ws_element.get('caption', name)
    
    # Explicitly hidden
    if ws_element.get('hidden') == 'true':
        return True
    
    # Not used in any dashboard
    if name not in dashboard_sheet_names:
        return True
    
    return False
```

Note: collect dashboard_sheet_names by scanning <dashboard><zones> FIRST,
before parsing worksheets, so you know which sheets appear in dashboards.

---

## FIX 3: get_display() Must Never Return "Unknown"

Problem: When caption is absent, code falls through to returning "Unknown"
instead of cleaning the internal name properly.

Replace get_display() and clean_internal() with these exact versions:

```python
def clean_internal(name: str) -> str:
    """Convert any internal name to a readable label. Never returns empty."""
    if not name:
        return 'Unnamed'
    n = name.strip()
    # Strip outer brackets: [field] → field
    n = re.sub(r'^\[|\]$', '', n)
    # Strip table prefix: [Table].[field] → field
    if '].[' in n:
        n = n.split('].[')[-1].rstrip(']')
    # Replace underscores and hyphens with spaces
    n = n.replace('_', ' ').replace('-', ' ')
    # Title case
    n = n.title()
    # If it still looks like Calculation 123456789 — keep it but flag
    # Do NOT return 'Unknown'
    return n.strip() if n.strip() else 'Unnamed'

def get_display(element, caption_attr='caption', name_attr='name') -> str:
    """Always returns a non-empty human-readable string."""
    if element is None:
        return 'Unnamed'
    cap = (element.get(caption_attr) or '').strip()
    if cap:
        return cap
    nam = (element.get(name_attr) or '').strip()
    if nam:
        return clean_internal(nam)
    return 'Unnamed'
```

Apply this same fix anywhere a display name is resolved in the codebase.
Search for any place that returns 'Unknown' as a name fallback and
replace with clean_internal() or 'Unnamed'.

---

## FIX 4: Formula Extraction — Correct XML Path

Problem: Formulas are empty because the code is reading from the wrong
XML path. The formula is in a <calculation> child element's attribute.

The correct XML structure is:
```xml
<column name='[Calculation_123]' caption='YTD Revenue' 
        role='measure' datatype='real'>
  <calculation class='tableau' formula='SUM([Revenue])' />
</column>
```

Fix the formula extraction in the column registration loop:

```python
# WRONG — these don't work:
formula = col.get('formula', '')           # formula is NOT on column
formula = col.findtext('formula')          # no formula text node

# CORRECT:
calc_el = col.find('calculation')
formula_raw = ''
if calc_el is not None:
    formula_raw = calc_el.get('formula', '').strip()
```

After this fix, is_calculated should be set to True when formula_raw is
non-empty, False otherwise.

---

## FIX 5: Shelf Parsing — Correct Field Extraction Per Worksheet

Problem: Field Count = 0 because shelf parsing isn't finding the fields.

Tableau stores shelf contents in the <worksheet><table> section.
The rows and cols are encoded as VizQL expression strings, not clean lists.

Correct parsing approach:

```python
def parse_shelf_expression(expr_string: str, registry) -> list[str]:
    """
    Extract and resolve all field references from a shelf expression string.
    Shelf strings look like: [field1]+[field2] or YEAR([Order Date])
    """
    if not expr_string:
        return []
    # Find all [FieldRef] patterns
    raw_refs = re.findall(r'\[([^\]]+)\]', expr_string)
    resolved = []
    for ref in raw_refs:
        # Skip date part strings
        if ref.lower() in ('year','quarter','month','week','day',
                           'hour','minute','second'):
            continue
        display = registry.display_name(ref)
        if display and display != 'Unnamed':
            resolved.append(display)
    return resolved

def parse_worksheet_fields(ws_element, registry) -> dict:
    table = ws_element.find('.//table')
    if table is None:
        return {'rows': [], 'cols': [], 'marks': {}, 'filters': []}
    
    # Rows and Cols shelf
    rows_el = ws_element.find('.//table/view/rows')
    cols_el = ws_element.find('.//table/view/cols')
    
    rows = parse_shelf_expression(
        rows_el.text if rows_el is not None else '', registry)
    cols = parse_shelf_expression(
        cols_el.text if cols_el is not None else '', registry)
    
    # Marks (color, size, label, detail, tooltip)
    marks = {'color': [], 'size': [], 'label': [], 
             'detail': [], 'tooltip': []}
    for enc in ws_element.findall('.//table/panes/pane/mark/encoding'):
        enc_type = enc.get('type', '')
        if enc_type in marks:
            field_ref = enc.get('field', '')
            if field_ref:
                marks[enc_type].append(registry.display_name(field_ref))
    
    # Filters
    filters = []
    for f in ws_element.findall('.//table/filter'):
        field_ref = f.get('column', '')
        if field_ref:
            filters.append({
                'field': registry.display_name(field_ref),
                'condition_summary': f.get('type', 'filter')
            })
    
    return {
        'rows': rows,
        'cols': cols,
        'marks': marks,
        'filters': filters,
        'field_count': len(set(rows + cols + 
                               [f for ml in marks.values() for f in ml]))
    }
```

---

## FIX 6: Usage Count Cross-Reference

Problem: Usage Count = 0 because the cross-reference pass between
fields and worksheets was never implemented or ran before registry
was complete.

Add this function and call it at the END of parse_twb(), after
worksheets are parsed but before building flags:

```python
def build_usage_index(worksheets: list[dict], 
                      registry: FieldRegistry) -> None:
    """
    Populates each FieldEntry with which worksheets use it.
    Call AFTER parse_worksheets() completes.
    Modifies registry entries in place.
    """
    # Add used_in_sheets list to each FieldEntry if not present
    for entry in registry.by_internal.values():
        if not hasattr(entry, 'used_in_sheets'):
            entry.used_in_sheets = []
        if not hasattr(entry, 'usage_count'):
            entry.usage_count = 0

    for ws in worksheets:
        ws_name = ws.get('display_name', '')
        
        # Collect all field display names used in this worksheet
        all_fields_in_sheet = set()
        all_fields_in_sheet.update(ws.get('rows', []))
        all_fields_in_sheet.update(ws.get('cols', []))
        for mark_fields in ws.get('marks', {}).values():
            all_fields_in_sheet.update(mark_fields)
        for f in ws.get('filters', []):
            all_fields_in_sheet.add(f.get('field', ''))
        
        # Match back to registry entries by display_name
        for entry in registry.by_internal.values():
            if entry.display_name in all_fields_in_sheet:
                entry.used_in_sheets.append(ws_name)
                entry.usage_count += 1

def get_unused_fields(registry: FieldRegistry) -> list[FieldEntry]:
    """Fields defined but never used on any shelf."""
    return [e for e in registry.by_internal.values() 
            if hasattr(e, 'usage_count') and e.usage_count == 0
            and not e.is_parameter
            and not is_system_field(e.internal_name, e.display_name)]
```

Call order in parse_twb():
```python
# After parse_worksheets() and render_formula() pass
build_usage_index(worksheets, registry)   # ← ADD THIS
dashboards  = parse_dashboards(root, registry)
flags       = build_migration_flags(registry, worksheets, dashboards)
```

---

## FIX 7: Dependency Diagram — Provide Real Data

Problem: Diagram is blank because the HTML renderer received empty lists.

In generate_html(), the dependency diagram section must pull from the
actual registry data. Use this data-building logic:

```python
def build_dependency_data(registry: FieldRegistry) -> dict:
    """Build node/edge data for the dependency diagram."""
    source_fields = []
    calc_fields = []
    
    for entry in registry.by_internal.values():
        if is_system_field(entry.internal_name, entry.display_name):
            continue
        if entry.is_parameter:
            continue
        if entry.is_calculated:
            calc_fields.append({
                'name': entry.display_name,
                'calc_type': entry.calc_type,
                'deps': entry.dependencies,
                'complexity': entry.dax_complexity
            })
        else:
            source_fields.append(entry.display_name)
    
    return {
        'source_fields': source_fields[:50],  # cap at 50 for rendering
        'calc_fields': calc_fields[:100],
    }
```

In the HTML diagram section, iterate this data to render nodes.
Cap source_fields at 50 and calc_fields at 100 to prevent
the diagram from becoming unrenderable on large workbooks like this one
(95 fields would otherwise flood the layout).

---

## FIX 8: Field Usage Section — Add "Never Used" Detection

Image 5 shows "0 fields never used" which is suspicious given
Usage Count = 0 for everything.

After build_usage_index() runs, the Field Usage Analysis section
in HTML must show:

  X total fields defined
  Y fields actively used (Y/X %)      ← usage_count > 0
  Z fields never used — can skip migration (Z/X %)  ← usage_count == 0

This is valuable — it tells the developer which fields to skip.
Only count non-system, non-parameter fields in these totals.

---

## SCOPE BOUNDARY

Only change:
  migration_blueprint_parser.py  — Fixes 1,2,3,4,5,6,7,8
  html_blueprint_generator.py    — Fix 7 diagram rendering, Fix 8 display

Do NOT change:
  migration_blueprint_gui.py
  build_blueprint_exe.py
  parse_twb() signature
  batch_runner() signature
  ParseResult field names already working

---

## HOW TO PROCEED

Fix 1 first. Show me the updated filter function and where it's applied.
Then say: "Fix 1 done. Confirm Fix 2?"
Do not combine fixes. Each fix is reviewable independently.
