- 👋 Hi, I’m Manohar Chekka
- 👀 I’m Data Engineer
- 📫 Reach me at : manoharch0698@gmail.com


<!---
MC140/MC140 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->

 Claude, I am handing you an existing, working repository for a Tableau XML parser. I have created a copy of this repo, and I want you to refactor it into a professional, high-end 'Migration Blueprint' tool. The Goal: Clean the existing logic, remove all non-essential metadata, and package it into a single, standalone .exe.
Please follow these 4 pillars for the refactor:
1. Universal Logic Refactor (The 'Clean Sweep'):
• Friendly Names Only: Tableau uses internal IDs like federated.123abc. Modify the parser to map every instance of these IDs to the caption attribute found in the <datasource> tag. Every output field must show the 'Friendly Name.'
• Comprehensive Extraction: Extract all connection attributes for every data source found in the XML. This includes, but is not limited to: server, dbname, port, filename, username, and schema. Do not prioritize one source over another.
• Eliminate Noise: Completely strip out the parsing of <thumbnails>, <style>, and <format> tags. They are unnecessary for this migration and clutter the output.
• Logic Extraction: Extract all calculated field formulas. If a formula contains FIXED, INCLUDE, or EXCLUDE, flag it as a 'Complex LOD' in the output.
2. Modernized GUI (CustomTkinter):
• Build a clean, dark-mode window.
• Include a file picker for .twb and .twbx (handle .twbx unzipping silently in the background).
• Add a 'Process' button and a progress bar.
3. Dual-Output System:
• JSON: A structured technical file for data mapping.
• HTML Migration Blueprint: Generate a beautifully styled, standalone HTML file. It must include a 'Source Connections' section for all identified sources and a searchable 'Calculations Gallery' table.
• Add 'Copy to Clipboard' buttons in the HTML next to all connection strings and SQL queries.
4. One-Click Distribution (PyInstaller):
• Ensure all libraries and HTML templates are bundled into a single .exe.
• Use a resource_path() function to ensure the tool can find its assets when running as a standalone executable.
• Provide a build.py script that I can run to generate this final .exe.

