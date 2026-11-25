- 👋 Hi, I’m Manohar Chekka
- 📫 Reach me at : manoharch0698@gmail.com


<!---
MC140/MC140 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
Build Backaroo, a lightweight Windows 11 desktop utility for employees across a bank.

Product name: Backaroo
Tagline: “Backaroo saves your spot when a call comes in.”
Language: Python for the desktop utility. Use JavaScript only if needed for an optional Microsoft Edge extension.

Create the project files, implement the feature, and run the tests available in this environment. Deliver a runnable prototype I can demonstrate on my own laptop.

THE EMPLOYEE EXPERIENCE

I’m reading a policy in Edge with Excel open beside it. A Teams call comes in. I answer, switch browser tabs, open PowerPoint, and move Excel to share my screen.

When the call ends, a small prompt appears in the bottom-right corner:

    Backaroo
    [Resume Edge + Excel]   [×]

If I click Resume, Backaroo brings my original Edge and Excel windows back to their previous positions, with my previously active window in front. If I dismiss the prompt or ignore it for 15 seconds, nothing moves. PowerPoint and anything else I opened during the call stay open.

There’s nothing to click before answering the call. Backaroo runs quietly in the system tray. Give it a small, recognizable kangaroo icon and keep the rest of the interface understated and accessible.

BUILD THE CORE FIRST

1. Remember the pre-call workspace.
   Keep a short, in-memory history of eligible foreground windows and their recent placement. When a call starts, freeze the last work context from before Teams took focus. Exclude Teams call controls, the Windows shell, tray menus, and Backaroo’s own UI. Opening the tray to simulate a call must not replace the work context.

   Start with the last active work window. Include a second window only when there is good evidence it was a visible companion, such as Edge and Excel arranged side by side. If uncertain, capture just the active window. Document the selection rule.

2. Deliver an end-to-end demo using simulated call events.
   Add “Simulate call start” and “Simulate call end” to the tray menu. Both simulated and automatic events must use the same capture, prompt, and restore logic. Keep the frozen snapshot intact while the employee switches apps during the call. Duplicate start or end events must not overwrite it or produce extra prompts.

3. Restore only when the employee clicks Resume.
   Remember window identity, owning process, placement, monitor, and normal or maximized state. If a captured window has closed, skip it. Guard against restoring an unrelated window that reused its handle. If a monitor was disconnected, restore surviving windows within the available screen area.

   Show the prompt without stealing keyboard focus. Use the actual app names, such as “Resume Excel” or “Resume Edge + Excel.” Dismissal and timeout must never move windows or change focus. Include Pause and Exit in the tray menu.

THEN ADD INTEGRATIONS WHERE FEASIBLE

4. Automatic Teams call detection.
   Investigate documented, standard-user signals available for the current Teams desktop client. Verify whether the signal reliably distinguishes call start and end from opening Teams, dismissing an incoming call, or using a microphone in another app. A Teams window title, microphone activity, calendar event, or presence change alone is insufficient. Avoid scraping Teams logs or relying on private endpoints.

   If reliable detection cannot be validated in this environment, finish the working simulated-call prototype and clearly identify what requires testing on a managed Windows laptop. Make the active mode visible in the tray menu so a presenter knows which behavior to expect.

5. Optional exact Edge tab restoration.
   Window restoration must work independently. If feasible, build a minimal Edge extension that remembers the original tab ID and reselects it on Resume if it is still open. Use minimal permissions and avoid collecting URLs, page titles, or page content. If this capability is unavailable, explain that Backaroo restores the Edge window and leaves its currently selected tab as it is.

LOCAL OPERATION

Run under a standard Windows user account. Keep captured work context in memory and discard it when the prompt is resolved. Do not capture screenshots, keystrokes, clipboard data, meeting audio, or document contents. No telemetry, cloud services, AI API keys, or runtime Copilot dependency. Keep dependencies minimal.

DELIVERABLES

- Working source code with a clear Python entry point.
- A README with exact PowerShell commands to install dependencies and launch Backaroo from VS Code.
- A 60-second demo script using Edge, Excel, and the simulated call controls. Include a second run where I ignore the prompt so I can show that Backaroo leaves my current work alone.
- Focused tests for selecting the pre-call workspace, preserving it during the call, handling duplicate events, and ensuring dismiss or timeout never triggers restoration.
- A verification summary listing what passed automated tests, what you actually observed on Windows, and what still requires a manual check.

Start implementing now. Prioritize a working “Resume Edge + Excel” demo before spending time on automatic Teams detection, browser integration, packaging, or visual polish.