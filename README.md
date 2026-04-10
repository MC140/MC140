- 👋 Hi, I’m Manohar Chekka
- 👀 I’m Data Engineer
- 📫 Reach me at : manoharch0698@gmail.com


<!---
MC140/MC140 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->

# Tableau XML — Datasources Section Parser Instructions

## Key Rules

1. **Always iterate by `<datasource>`** — tag every output with `datasource_caption`
1. **Separate live vs extract** — check `parent-name` = `[sqlproxy]` vs `[Extract]`
1. **Dedup on `remote-name`** — same column can appear multiple times
1. **Map calc IDs to captions** — build a dict from top-level `<column>` tags first

-----

## What You Can Extract

|What                |Where                                                            |Key Attribute          |
|--------------------|-----------------------------------------------------------------|-----------------------|
|Live table name     |`connection > relation`                                          |`table`                |
|Live column names   |`connection > metadata-records[class='column']`                  |`remote-name`          |
|Calculated measures |`connection > metadata-records[class='measure']` + `calculations`|`formula`              |
|Calculated columns  |`datasource > column` (dimension type)                           |`caption`, `datatype`  |
|Extract table name  |`extract > connection > relation`                                |`table`                |
|Extract column names|`extract > connection > metadata-records[class='column']`        |`remote-name`, `family`|

-----

## Reference Code

```python
import xml.etree.ElementTree as ET
from collections import defaultdict

def parse_datasources(xml_path):
    tree = ET.parse(xml_path)
    root = tree.getroot()

    results = defaultdict(lambda: {
        'live_table': None,
        'live_columns': [],
        'calc_measures': [],
        'calc_columns': [],
        'extract_table': None,
        'extract_columns': []
    })

    for ds in root.findall('.//datasource'):
        ds_name = ds.get('caption') or ds.get('name', 'Unknown')

        # --- Step 1: Build caption map from top-level <column> tags ---
        caption_map = {}
        for col in ds.findall('column'):
            name = col.get('name', '')
            caption = col.get('caption', '')
            aggregation = col.get('aggregation', '')
            datatype = col.get('datatype', '')
            caption_map[name] = {
                'caption': caption,
                'aggregation': aggregation,
                'datatype': datatype
            }

        # --- Step 2: Live connection ---
        connection = ds.find('connection')
        if connection is not None:

            # Live table name
            relation = connection.find('relation')
            if relation is not None:
                results[ds_name]['live_table'] = relation.get('table')

            # Calculated field formulas
            calc_id_to_formula = {}
            for calc in connection.findall('.//calculation'):
                col_id = calc.get('column', '')
                formula = calc.get('formula', '')
                calc_id_to_formula[col_id] = formula

            # Metadata records — live
            seen_live = set()
            for record in connection.findall('metadata-records/metadata-record'):
                cls = record.get('class')
                remote_name = record.findtext('remote-name', '')
                local_name = record.findtext('local-name', '')
                local_type = record.findtext('local-type', '')
                parent = record.findtext('parent-name', '')

                # Skip extract records here
                if parent == '[Extract]':
                    continue

                if remote_name in seen_live:
                    continue
                seen_live.add(remote_name)

                info = {
                    'remote_name': remote_name,
                    'local_name': local_name,
                    'local_type': local_type,
                    'caption': caption_map.get(local_name, {}).get('caption', ''),
                    'formula': calc_id_to_formula.get(local_name, '')
                }

                if cls == 'column':
                    results[ds_name]['live_columns'].append(info)
                elif cls == 'measure':
                    results[ds_name]['calc_measures'].append(info)

        # --- Step 3: Extract connection ---
        extract = ds.find('extract')
        if extract is not None:
            ext_conn = extract.find('connection')
            if ext_conn is not None:

                # Extract table name
                ext_relation = ext_conn.find('relation')
                if ext_relation is not None:
                    results[ds_name]['extract_table'] = ext_relation.get('table')

                # Extract columns
                seen_extract = set()
                for record in ext_conn.findall('metadata-records/metadata-record'):
                    if record.get('class') != 'column':
                        continue
                    remote_name = record.findtext('remote-name', '')
                    if remote_name in seen_extract:
                        continue
                    seen_extract.add(remote_name)

                    results[ds_name]['extract_columns'].append({
                        'remote_name': remote_name,
                        'local_name': record.findtext('local-name', ''),
                        'local_type': record.findtext('local-type', ''),
                        'family': record.findtext('family', ''),
                        'approx_count': record.findtext('approx-count', '')
                    })

        # --- Step 4: Calculated columns (dimension type from caption_map) ---
        for name, info in caption_map.items():
            if name.startswith('[Calculation_'):
                if info['aggregation'] not in ('Sum', 'Count', 'Average', 'User'):
                    results[ds_name]['calc_columns'].append({
                        'name': name,
                        'caption': info['caption'],
                        'datatype': info['datatype']
                    })

    return dict(results)
```

-----

## Why This Works With Every File

|Rule                       |Reason                                     |
|---------------------------|-------------------------------------------|
|`.findall('.//datasource')`|Handles nested datasource structures       |
|`caption or name` fallback |Works even for unnamed datasources         |
|`seen` sets per section    |Prevents duplicate columns                 |
|`parent-name` check        |Correctly separates live vs extract records|
|Caption map built first    |Works even if calc IDs have no captions    |
|`findtext(tag, '')`        |Never throws error on missing tags         |