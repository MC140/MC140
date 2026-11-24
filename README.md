- 👋 Hi, I’m Manohar Chekka
- 📫 Reach me at : manoharch0698@gmail.com


<!---
MC140/MC140 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
Project: Work Continuity — Intelligent Windows Workspace Recovery

1. Product vision

Build a production-oriented, lightweight Windows desktop application called Work Continuity.

The fundamental idea is to extend the concept of autosave from individual documents to an employee’s entire working environment.

Employees in large organizations frequently work across multiple applications, documents, browser windows, development environments and monitors simultaneously.

When their laptop restarts, Windows updates, applications crash or the working environment is interrupted, employees spend unnecessary time reconstructing their workspace.

Work Continuity should automatically preserve supported workspace information and allow employees to resume their previous working environment with minimal interaction.

The product philosophy is:

Install once. Work normally. Resume effortlessly.

This must not become another application that employees have to remember to use every day. It should operate quietly in the background and provide value without introducing additional steps into existing workflows.

2. Enterprise environment and constraints

The application is intended for employees working within a large, highly regulated financial services organization.

Available development tools:

* Visual Studio Code.
* GitHub Copilot.
* Microsoft 365 Copilot.
* Standard Windows operating system capabilities.

Mandatory constraints:

* Run entirely on local Windows laptops.
* Windows 11 compatibility.
* No VDI dependency.
* No external API calls.
* No Azure or cloud infrastructure.
* No backend server.
* No external database.
* No additional AI subscription.
* No Microsoft Graph API dependency.
* No Microsoft 365 Copilot API dependency.
* No administrator privileges for normal operation.
* No telemetry or external data transmission.
* No external authentication.
* No dependency on employees having VS Code installed.
* No dependency on employees having GitHub Copilot installed.
* Minimal ongoing maintenance.
* Support enterprise security review and approved internal distribution.

GitHub Copilot is a development tool, not a runtime dependency.

Do not introduce AI into the application simply for marketing purposes. Use deterministic local operating system capabilities wherever possible.

3. Technology architecture

Use the following preferred technology stack:

* C#.
* An organization-approved supported .NET LTS version, preferably .NET 10.
* WPF for the desktop user interface.
* Windows system tray integration.
* Native Windows Win32 APIs through P/Invoke.
* System.Text.Json for serialization.
* Windows Data Protection API (DPAPI) for local encryption.
* Local application data storage.
* MVVM architecture.
* Asynchronous background processing.
* Minimal third-party dependencies.

Build a self-contained Windows executable where practical, so employees do not need to manually install development runtimes.

Use native Windows capabilities before introducing additional NuGet packages.

4. Core product capabilities

A. Automatic workspace detection

Detect supported applications currently open on the user’s Windows desktop.

Capture only the minimum information necessary for restoration:

* Application identity.
* Approved executable path or application identifier.
* Window position.
* Window dimensions.
* Window maximized or minimized state.
* Monitor association.
* Monitor resolution and scaling information where necessary.
* Supported document or project paths, when explicitly available through approved application-specific integration.

Use appropriate Windows APIs such as:

* EnumWindows.
* IsWindowVisible.
* GetWindowThreadProcessId.
* GetWindowPlacement.
* SetWindowPlacement.
* MonitorFromWindow.
* GetMonitorInfo.
* QueryFullProcessImageName where appropriate.

Do not assume all Windows applications expose their internal state.

Do not persist temporary window handles or process IDs as permanent restoration identifiers.

Exclude system processes, credential windows, security tools, background services, hidden windows and unsupported applications.

B. Automatic workspace checkpointing

Automatically capture the supported workspace state at configurable intervals.

Default interval: 60 seconds.

Additional configurable intervals:

* 30 seconds.
* 2 minutes.
* 5 minutes.
* 10 minutes.

Capture a final checkpoint during supported Windows shutdown or session-ending events on a best-effort basis.

Implement:

* Change detection to avoid unnecessary disk writes.
* Atomic file replacement.
* Recovery from corrupted checkpoint files.
* Checkpoint retention limits.
* Thread-safe storage.
* Cancellation support.
* Graceful exception handling.

Do not overwrite a valid previous-session checkpoint with an empty or incomplete workspace immediately after Windows startup.

Maintain separate active-session and previous-session checkpoints.

C. Workspace restoration

After Windows restarts, the application should detect whether a recoverable previous workspace exists.

Provide a small, nonintrusive notification:

“Your previous workspace is ready to resume.”

Provide actions:

* Restore workspace.
* Dismiss.

Support optional automatic restoration if the user explicitly enables it in settings.

Workspace restoration should:

1. Read the encrypted workspace checkpoint.
2. Validate stored application identifiers and supported file paths.
3. Detect applications that are already running.
4. Avoid duplicate application instances where possible.
5. Launch supported applications using approved launch mechanisms.
6. Wait asynchronously for relevant application windows to become available.
7. Restore window positions and dimensions.
8. Restore supported maximized states.
9. Reposition windows safely if monitor configurations have changed.
10. Report applications that could not be restored without interrupting the entire restoration process.

Never execute arbitrary commands stored in a workspace file.

Do not restore raw command-line arguments, scripts or untrusted executable paths.

Use an explicit application allowlist and application-specific launch handlers.

Do not claim to restore unsaved document content, application memory, browser authentication sessions or exact internal application state.

D. Multi-monitor intelligence

Support employees using:

* Single laptop screens.
* Dual monitors.
* Triple monitors.
* Docking stations.
* Different monitor resolutions.
* Different Windows display scaling settings.

Example:

An employee works with Excel on the left monitor, VS Code on the right monitor and Edge on the laptop screen.

When the workspace is restored, attempt to position applications on their previous monitors.

If the employee disconnects from the docking station, automatically reposition windows within the available screen boundaries.

Ensure no restored window becomes permanently inaccessible outside the visible desktop area.

Account for Windows DPI awareness.

E. Supported applications

Design an extensible application adapter architecture.

Prioritize these applications:

* Microsoft Edge.
* Visual Studio Code.
* File Explorer.
* Microsoft Excel.
* Microsoft Word.
* Microsoft PowerPoint.
* Notepad.
* Power BI Desktop.
* Microsoft Outlook.
* Microsoft Teams.

For the initial version, support application-level recovery and window-layout restoration wherever possible.

Deeper document-level recovery must use application-specific integrations where supported and approved.

Do not pretend to support document-level recovery when a reliable mechanism is unavailable.

Create capability levels for each application:

* Application restoration supported.
* Window-layout restoration supported.
* Document restoration supported.
* Internal session restoration not supported.

Implement generic application restoration first.

Then implement reliable adapters incrementally.

Use File Explorer folder restoration as an early application-specific enhancement if feasible.

Microsoft Office integration may be added using approved, read-only application-specific mechanisms. Do not read document contents, modify documents, trigger macros or introduce unnecessary COM dependencies into the core engine.

Microsoft Edge browser tab recovery should be treated as an optional future enhancement requiring an approved browser extension or another officially supported mechanism.

Do not access private browser databases, cookies, credentials or browser debugging interfaces.

5. User experience

The application must feel like a native Windows productivity feature rather than a traditional enterprise application.

It should remain unobtrusive.

System tray experience

Create a Windows system tray icon.

The tray menu should include:

* Work Continuity status.
* Last checkpoint time.
* Restore previous workspace.
* Workspace history.
* Pause automatic checkpointing.
* Resume automatic checkpointing.
* Settings.
* Exit.

Use a simple status indicator for:

* Active.
* Paused.
* Restoring.
* Error.

Do not continuously display notifications.

Main application interface

Design a clean, premium, modern Windows 11-inspired interface.

Support both light and dark modes.

The main window should contain:

* Overview.
* Previous workspace.
* Workspace history.
* Supported applications.
* Settings.
* Privacy and security.

Dashboard indicators:

* Protection status.
* Last checkpoint.
* Number of supported applications detected.
* Number of restorable windows.
* Restore availability.
* Last restoration result.

Display detected application icons and application names.

Avoid collecting or displaying document titles unless explicitly needed and enabled.

Use clean typography, subtle animations, rounded corners and consistent spacing.

Avoid excessive gradients, decorative elements or a generic AI-generated dashboard appearance.

First-launch experience

On initial launch, explain clearly:

“Work Continuity automatically remembers your supported open applications and window layouts, helping you resume work after interruptions.”

Allow the user to:

* Enable automatic checkpointing.
* Choose supported applications.
* Enable or disable starting with Windows.
* Choose whether supported document paths may be stored.
* Review privacy settings.

Startup registration must be explicitly enabled by the user or configured through an approved enterprise deployment mechanism.

Once configured, the application should require almost no daily interaction.

6. Workspace history

Maintain a limited history of workspace checkpoints.

Default retention: seven days, with a configurable storage and checkpoint limit.

Allow the employee to view:

* Checkpoint date and time.
* Applications included.
* Number of supported windows.
* Restoration availability.

Allow restoration of a selected historical workspace.

Allow users to delete individual checkpoints or clear all stored data.

Do not create unlimited historical records.

Do not turn this feature into employee activity tracking.

7. Privacy and security

Security is a primary product requirement.

The application must be suitable for assessment in a regulated enterprise environment.

Never collect:

* Keystrokes.
* Screenshots.
* Clipboard contents.
* Document contents.
* Email contents.
* Teams messages.
* Browser cookies.
* Passwords.
* Access tokens.
* Network traffic.
* Employee productivity metrics.
* Time spent in individual applications.
* Raw process command lines.
* Customer information.
* Sensitive window titles by default.

Store only the minimum workspace metadata required for supported functionality.

Store local data under the current user’s LocalAppData directory.

Encrypt workspace checkpoints using Windows DPAPI with current-user scope.

Implement:

* Secure local storage.
* Input validation.
* Versioned checkpoint schemas.
* Application allowlisting.
* Safe launch behavior.
* Corrupted data handling.
* Configurable retention.
* Complete local data deletion.
* No network communication.
* No telemetry.
* No remote control.
* No elevated privileges.
* No hidden persistence mechanisms.
* No security control bypasses.

Treat application paths, document paths and workspace metadata as potentially confidential.

Diagnostic logs must not contain sensitive file paths, document names or internal organizational information by default.

Do not use real corporate information in development samples, tests, documentation or Copilot prompts. Use synthetic examples.

8. Performance requirements

The application should be lightweight enough to run continuously without noticeably affecting employee productivity.

Target:

* Near-zero CPU usage while idle.
* Less than 1% average background CPU usage during normal operation on a typical corporate laptop.
* Memory usage below 100 MB where practical.
* Minimal disk usage.
* Fast startup.
* Nonblocking checkpoint operations.
* No unnecessary process scanning.
* No polling more frequently than necessary.
* No interference with foreground applications.
* No keyboard or mouse focus stealing during background checkpointing.

Measure performance rather than simply claiming these targets have been achieved.

Use background services, timers and cancellation tokens appropriately.

9. Architecture and repository structure

Create a maintainable modular solution similar to:

WorkContinuity/
  src/
    WorkContinuity.App/
    WorkContinuity.Core/
    WorkContinuity.Windows/
    WorkContinuity.Storage/
    WorkContinuity.Adapters/
  tests/
    WorkContinuity.UnitTests/
    WorkContinuity.IntegrationTests/
  docs/
    Architecture.md
    Security.md
    Deployment.md
    UserGuide.md
    SupportedApplications.md
    TestResults.md
  scripts/
    build.ps1
    publish.ps1
  README.md
  WorkContinuity.sln
  .gitignore

Separate responsibilities into:

* Workspace capture service.
* Window discovery service.
* Monitor discovery service.
* Workspace restoration engine.
* Application adapter registry.
* Secure checkpoint storage.
* Checkpoint scheduling service.
* Startup configuration service.
* Notification service.
* Settings service.
* Diagnostic logging.
* User interface.

Create clear interfaces so that individual application adapters can be added independently without rewriting the core architecture.

Avoid unnecessary overengineering.

10. Development roadmap

Phase 1 — Functional MVP

Implement a complete working application with:

* Native Windows desktop interface.
* System tray functionality.
* Automatic workspace capture.
* Supported application detection.
* Window size and position capture.
* Encrypted local checkpoint storage.
* Previous workspace restoration.
* Basic multi-monitor support.
* Application allowlist.
* Pause and resume.
* Settings.
* Optional startup with Windows.
* Checkpoint deletion.
* Working executable publishing process.

Complete and validate this phase before introducing advanced integrations.

Phase 2 — Application intelligence

Add:

* File Explorer folder restoration.
* VS Code project restoration where reliably supported.
* Approved Office document restoration.
* Improved application-specific launch handlers.
* Better window matching.
* Monitor configuration intelligence.
* Enhanced workspace history.

Phase 3 — Optional enhancements

Evaluate:

* Approved Microsoft Edge extension for tab recovery.
* Automatic project workspace grouping.
* Named workspaces.
* Intelligent recovery after individual application crashes.
* Enterprise configuration policies.
* Enterprise-approved software signing and packaging.
* Accessibility improvements.

Do not introduce cloud services or AI APIs in these phases unless the product requirements are explicitly changed later.

11. Testing requirements

Create automated unit tests and appropriate Windows integration tests.

Test scenarios:

* Multiple supported applications open simultaneously.
* Workspace capture with no supported applications.
* Duplicate application windows.
* Applications closed during checkpoint capture.
* Applications already running during restoration.
* Applications uninstalled after checkpoint capture.
* Missing documents.
* Changed monitor resolutions.
* Removed external monitors.
* Different DPI configurations.
* Corrupted checkpoint files.
* Invalid executable paths.
* Malicious or modified checkpoint data.
* Interrupted checkpoint writes.
* Windows restart.
* Unexpected application termination.
* Startup before the desktop is fully initialized.
* Repeated restoration requests.
* Encryption and decryption failures.
* Paused checkpointing.
* Complete workspace history deletion.

Verify that the application never opens duplicate windows unnecessarily, executes arbitrary commands or overwrites valuable recovery checkpoints prematurely.

Separate tests that can be automated from tests requiring a real Windows desktop environment.

Do not report tests as passed unless they have actually been executed successfully.

12. Product demonstration mode

Include an optional demonstration mode that uses synthetic workspace data.

The demonstration should clearly show:

1. Multiple applications being detected.
2. An automatic workspace checkpoint being created.
3. A previous workspace being available.
4. The restoration process.
5. Window-layout restoration results.
6. Local privacy and security settings.

Clearly identify simulated data and simulated results.

The purpose is to demonstrate the product to technology leadership, innovation teams and enterprise security reviewers without exposing confidential information.

13. Enterprise value proposition

The application should be positioned around:

* Business continuity at the individual workspace level.
* Reduction in workspace reconstruction time.
* Improved employee experience.
* Reduced disruption following Windows restarts.
* Minimal adoption friction.
* No incremental cloud infrastructure costs.
* Local-first privacy architecture.
* Potential organization-wide applicability.

Do not invent productivity savings or ROI figures.

Provide a methodology for measuring time saved during a future pilot without implementing employee surveillance.

14. Final deliverables

Produce:

1. Complete functional source code.
2. Working Visual Studio solution.
3. Modern Windows user interface.
4. Self-contained Windows executable or approved equivalent packaging.
5. Build and publish scripts.
6. Automated tests.
7. Architecture documentation.
8. Security and privacy documentation.
9. Enterprise deployment guidance.
10. User guide.
11. Supported application capability matrix.
12. Known limitations.
13. Demonstration instructions.
14. Software dependency inventory.

15. Instructions for GitHub Copilot Agent

Act as a senior Windows desktop software engineer, enterprise solution architect, cybersecurity-conscious developer and product designer.

You are responsible for building this application, not merely explaining how it could be built.

Start by inspecting the current repository and available development environment.

Then:

1. Verify the installed .NET SDK and Windows development capabilities.
2. Create the architecture and implementation plan.
3. Initialize the solution and project structure.
4. Implement the core Windows integration.
5. Implement secure local storage.
6. Implement workspace capture and restoration.
7. Build the system tray functionality.
8. Develop the user interface.
9. Implement settings and checkpoint history.
10. Add automated tests.
11. Build the application.
12. Resolve compilation errors.
13. Run available tests.
14. Publish the executable.
15. Generate the documentation.

Continue implementing the Phase 1 MVP rather than stopping after generating a plan, skeleton project or README.

When encountering technical problems, investigate the supported Windows mechanisms and resolve them.

Do not use placeholder implementations for core functionality.

Do not use mock data in the real application workflow.

Do not silently bypass security requirements to make a feature work.

If a feature is technically unsupported or requires an additional enterprise permission, document the limitation and implement the closest safe supported alternative.

Do not install unapproved tools, modify enterprise security policies or request elevated permissions to overcome restrictions.

Do not deploy to other employees’ devices automatically.

Make sensible engineering decisions independently within the specified constraints.

The final product should feel like a genuine native Windows utility that employees would naturally want enabled on their laptops.

Begin implementation now.