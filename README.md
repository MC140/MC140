- 👋 Hi, I’m Manohar Chekka
- 📫 Reach me at : manoharch0698@gmail.com


<!---
MC140/MC140 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
Technology Stack Revision — Python

Change the implementation technology from C#/.NET/WPF to Python. Retain all previously defined Work Continuity product requirements, features, security constraints, user experience and enterprise deployment objectives.

Use the following revised technology stack:

* Python 3.12 or a newer organization-approved supported version — Core application development.
* PySide6 (Qt) — Modern, native-style desktop interface and system tray integration.
* pywin32 and ctypes — Windows API integration, application detection, window management, multi-monitor support and workspace restoration.
* psutil — Process identification and resource monitoring where necessary.
* Windows DPAPI through win32crypt — Local workspace encryption.
* JSON — Workspace checkpoint serialization.
* pathlib — Local file management.
* threading and Qt timers — Nonblocking background checkpointing.
* pytest — Automated testing.
* PyInstaller — Standalone Windows executable packaging.

Additional implementation requirements

* Do not use .NET, C#, WPF, Electron, Flask or Streamlit.
* Build a genuine Windows desktop application, not a browser-based interface.
* Run silently in the Windows system tray.
* Use Windows-native APIs wherever possible.
* Ensure DPI awareness and multi-monitor compatibility.
* Implement single-instance execution to prevent duplicate background processes.
* Ensure graceful shutdown and thread-safe checkpoint storage.
* Minimize CPU and memory consumption.
* Package all required Python dependencies into the distributable application.
* Employees must not need Python, VS Code or GitHub Copilot installed to run the packaged application.
* Provide both a portable executable distribution and a single-file executable where practical.
* Review third-party dependency licenses and enterprise distribution requirements, particularly PySide6/Qt.
* Use only organization-approved dependencies and package repositories.
* Do not require administrator privileges.
* Do not introduce external APIs, AI services, cloud infrastructure, telemetry or network communication.

Development instructions

Inspect the existing repository before proceeding. If any C# or .NET implementation has already been generated, migrate the application architecture to Python while preserving the original functional requirements.

Create a modular Python project with a requirements file, virtual environment instructions, automated tests, build scripts and PyInstaller configuration.

Verify the installed Python version and available dependencies before implementation.

Build, test and package a fully functional Phase 1 MVP. Do not stop at generating architecture documentation, mock interfaces or placeholder functionality.

Preserve all original privacy, security, workspace recovery, application allowlisting and enterprise-readiness requirements.

The final application should feel like a premium native Windows utility, not a typical Python script.

Proceed with the Python implementation now.