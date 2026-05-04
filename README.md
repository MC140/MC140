- 👋 Hi, I’m Manohar Chekka
- 👀 I’m Data Engineer
- 📫 Reach me at : manoharch0698@gmail.com


<!---
MC140/MC140 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->

I want to build a Python-based .pbit file auditor tool. No LLMs or AI models — 
pure rule-based logic only. Here is the full spec:

## What is a .pbit file
A .pbit is a ZIP archive. It contains:
- `DataModelSchema` — UTF-16 LE encoded JSON (no file extension) with all 
  tables, columns, measures, relationships, partitions, hierarchies
- `Report/Layout` — UTF-8 JSON with pages, visuals, filters
- `[Content_Types].xml`

## Project Structure
Create this folder structure:
pbit_auditor/
├── main.py
├── extractor.py
├── model_detector.py
├── rule_engine.py
├── rules/
│   ├── base_rules.py
│   ├── import_rules.py
│   ├── directquery_rules.py
│   ├── composite_rules.py
│   └── report_rules.py
├── config/
│   └── rules.yaml
├── reporter.py
└── output/  (generated reports go here)

## RuleResult dataclass (use this exact shape everywhere)
@dataclass
class RuleResult:
    rule_id: str
    category: str
    severity: str        # "PASS" | "WARN" | "FAIL"
    title: str
    detail: str
    affected: list[str]
    applies_to: str      # "ALL" | "IMPORT" | "DIRECTQUERY" | "COMPOSITE"

## Step 1 — Build extractor.py first
- Open the .pbit as a zipfile
- Read `DataModelSchema` and decode as UTF-16 LE
- Parse as JSON and return the dict
- Read `Report/Layout` as UTF-8 JSON and return separately
- Handle missing files gracefully with clear error messages

## Step 2 — Build model_detector.py
Detect model type by inspecting partition modes in DataModelSchema:
- All partitions mode = "import" → ModelType.IMPORT
- Any partition mode = "directQuery" and others = "import" → ModelType.COMPOSITE
- All partitions mode = "directQuery" → ModelType.DIRECTQUERY
- No local tables → ModelType.LIVE
Return ModelType enum and a dict of {table_name: storage_mode}

## Step 3 — Build rule_engine.py
- Accepts parsed schema dict, layout dict, model type, and config from rules.yaml
- Runs base_rules always
- Runs import_rules only if IMPORT or COMPOSITE
- Runs directquery_rules only if DIRECTQUERY or COMPOSITE (DQ tables only)
- Runs report_rules always
- Returns list[RuleResult]

## Step 4 — Build rules (start with base_rules.py)
Implement these checks:
NM001 - Table names contain spaces → FAIL
NM002 - Column names contain spaces → WARN
NM003 - Measure names are generic (Measure1, Column1, Sum of *) → WARN
DC001 - Measures missing description field or description is empty → WARN
DC002 - Tables missing description → WARN
RL001 - Many-to-many relationships (fromCardinality and toCardinality both = "many") → WARN
RL002 - Bi-directional crossFilteringBehavior on relationships involving fact tables → FAIL
RL005 - Tables with no relationships at all → WARN
DT001 - No table has isDateTable = true → WARN
MS002 - Measures missing formatString → WARN
MS006 - Duplicate DAX expressions across measures (exact string match) → WARN

## Step 5 — Build reporter.py
Generate a single self-contained HTML file (no external CSS/JS):
- Header: filename, model type, scan timestamp
- Scorecard: total PASS / WARN / FAIL counts with color blocks
- Table-per-category: rule_id | severity | title | affected objects
- Color coding: FAIL=red, WARN=amber, PASS=green
- Save to output/audit_report.html

## Step 6 — Build main.py (CLI entry point)
Usage: python main.py --file report.pbit --config config/rules.yaml
- Parse args
- Call extractor → model_detector → rule_engine → reporter
- Print summary to console and save HTML report

## Rules config (rules.yaml)
Support these keys:
thresholds:
  max_visuals_per_page: 20
  max_columns_per_table: 30
disabled_rules: []
sensitive_column_keywords: [salary, sin, dob, ssn]
fact_prefixes: ["Fact_", "f_"]
dim_prefixes: ["Dim_", "d_"]

## Constraints
- Python stdlib only: zipfile, json, re, dataclasses, pathlib, datetime, argparse
- PyYAML is allowed for config parsing (pip install pyyaml)
- No pandas, no openpyxl, no LLMs, no API calls
- All output must work offline

Start by generating extractor.py completely, then ask me before moving to the next file.
