- 👋 Hi, I’m Manohar Chekka
- 👀 I’m Data Engineer
- 📫 Reach me at : manoharch0698@gmail.com


<!---
MC140/MC140 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
## CHAIN ADDITION — Extended SQL Extraction
## Add this to parse_datasources() without changing anything else.

The current parse_datasources() extracts custom_sql from 
<relation type='text'> only. Extend it to also extract these 
three additional SQL types from the same datasource connection.

---

### What to extract and where it lives in the XML

#### 1. Initial SQL
Location: attribute on the <connection> element
```xml
<connection class='sqlserver' server='...' 
            initial-sql='SET NOCOUNT ON; EXEC dbo.SetContext()' />
```
Extract: connection.get('initial-sql', '')
Field name: initial_sql
If empty string → store as '' (do not store None)

#### 2. Join Logic
Location: <relation type='join'> children, each has a <clause>
```xml
<relation type='join' join='inner'>
  <relation type='table' table='[orders]' name='orders' />
  <relation type='table' table='[customers]' name='customers' />
  <clause type='join'>
    <expression op='='>
      <expression op='[orders].[cust_id]' />
      <expression op='[customers].[id]' />
    </expression>
  </clause>
</relation>
```
Extract join type (inner/left/right/full) and the two table names.
Build a plain English summary string per join:
  "INNER JOIN customers ON orders.cust_id = customers.id"
Field name: joins → list of strings (one per join clause found)
If no joins found → empty list []

#### 3. Stored Procedure
Location: <relation type='stored-proc'>
```xml
<relation type='stored-proc' procedure='[dbo].[usp_GetSalesData]'>
  <column name='...' />
</relation>
```
Extract: relation.get('procedure', '')
Strip outer brackets if present.
Field name: stored_procedure
If absent → ''

---

### How to extend the datasource dict

Current structure:
```python
{
  'display_name': '',
  'internal_name': '',
  'connection_type': '',
  'server': '',
  'database': '',
  'schema': '',
  'is_extract': False,
  'custom_sql': ''
}
```

Add these three fields:
```python
{
  ...existing fields unchanged...,
  'custom_sql':       '',   # existing — keep as is
  'initial_sql':      '',   # NEW
  'joins':            [],   # NEW — list of plain English join strings
  'stored_procedure': ''    # NEW
}
```

---

### Migration flag rules to ADD for these new fields

Add to build_migration_flags() — do not remove existing flag rules:

  RED:
    - datasource has stored_procedure non-empty →
      category: 'Stored Procedure'
      description: 'Stored proc "{name}" cannot be called directly 
                    from Power BI. Recreate as a database view or 
                    native query in Power Query.'

  AMBER:
    - datasource has initial_sql non-empty →
      category: 'Initial SQL'
      description: 'Initial SQL detected on "{datasource}". Power BI 
                    has no equivalent. Move session-level logic to a 
                    database role, view, or connection default.'

    - datasource has joins non-empty →
      category: 'Source-Level Join'
      description: 'Joins defined at connection level in "{datasource}". 
                    Recreate as Merge Queries in Power Query or define 
                    model relationships instead.'

---

### HTML changes — Source Connections card only

In each datasource card, after the existing custom_sql block add:

  If initial_sql non-empty:
    Sub-label: "Initial SQL"
    Code block showing the initial_sql value
    Info box: "No Power BI equivalent — move to DB role or view"

  If joins non-empty:
    Sub-label: "Source-Level Joins"
    Each join as a code line
    Info box: "Recreate as Power Query Merge or model relationships"

  If stored_procedure non-empty:
    Sub-label: "Stored Procedure"
    Code block showing procedure name
    Info box (red tint): "Cannot call stored procs in Power BI — 
                          requires database view or Power Query workaround"

---

### JSON changes — datasources array only

Add the three new fields to each datasource object in generate_json():
  "initial_sql": "",
  "joins": [],
  "stored_procedure": ""

---

### Scope boundary — do not touch anything else

- Do not change any other part of parse_datasources()
- Do not change parse_worksheets, parse_dashboards, or parameters
- Do not change the ParseResult dataclass
- Do not change the public entry points parse_twb() or batch_runner()
- Only extend the datasource dict, migration flags, 
  HTML datasource cards, and JSON datasource objects

---

### How to proceed

Show me the extended parse_datasources() function only.
Then show the 3 new flag rules to add to build_migration_flags().
Then show the HTML card changes for Source Connections.

After each part say:
  "Part N of 3 complete. Confirm to proceed to Part N+1?"
