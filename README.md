- 👋 Hi, I’m Manohar Chekka
- 👀 I’m Data Engineer
- 📫 Reach me at : manoharch0698@gmail.com


<!---
MC140/MC140 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->

#
Checklist Item
Description / What Workspace Owner Must Verify
Status
1
Validate Workspace Roles
All users are assigned through Workspace AD Groups (Contributor, QA, Viewer). No direct workspace access.
⬜
2
Confirm User Attestation
Every user completed Copilot User Agreement, reviewed PUG, and Key Risks.
⬜
3
Confirm Workspace on Premium
Workspace is hosted on P1 Premium capacity required for Copilot.
⬜
4
Create Workspace Copilot AD Group
Create a new AD group using naming convention: <Capacity>_<WorkspaceName>_Copilot (e.g., DBIGW_Rahona_Copilot).
⬜
5
Validate Copilot AD Group Settings
Group is security-enabled, visible, and Workspace Owner has permission to nest groups.
⬜
6
Nest Contributor AD Group
Add the workspace’s Contributor AD Group inside the new Copilot AD Group. No direct users.
⬜
7
Nest QA AD Group
Add the workspace QA AD Group inside the Copilot AD Group.
⬜
8
Nest Viewer / Report Viewer AD Group
Add the workspace Viewer or Report Viewer AD Group inside the Copilot AD Group.
⬜
9
Verify Nesting Structure
Ensure only AD groups are nested; no direct user assignments present.
⬜
10
Submit RITM Request
Use the official form to request linking your Workspace Copilot AD Group to the Tenant-Level Copilot Group.
⬜
11
Tenant Team Validation
BusOps/Admin verifies governance compliance and connects your group to: O365_PowerBI_Copilot_<Capacity>.
⬜
12
Copilot Activation Confirmation
Workspace Owner receives confirmation that Copilot is active for their workspace.
⬜
13
Notify Users
Inform workspace members that Copilot is now enabled & provide CoE support inbox for questions.
⬜

