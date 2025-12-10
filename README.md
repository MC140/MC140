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
All users are assigned only through Workspace AD Groups (Contributor, QA, Viewer). No direct workspace access.
[ ]
2
Confirm User Attestation
All users completed the Copilot User Agreement, reviewed Permissible Use Guidelines (PUG) and Key Risks.
[ ]
3
Confirm Workspace on Premium
Workspace is hosted on P1 Premium capacity (required for Copilot).
[ ]
4
Create Workspace Copilot AD Group
Create AD group using naming convention: <Capacity>_<WorkspaceName>_Copilot
[ ]
5
Validate Copilot AD Group Settings
Group is security-enabled and Workspace Owner has permissions needed to nest groups.
[ ]
6
Nest Contributor AD Group
Add the workspace Contributor AD Group inside the Copilot AD Group (no direct users).
[ ]
7
Nest QA AD Group
Add the workspace QA AD Group inside the Copilot AD Group.
[ ]
8
Nest Viewer / Report Viewer AD Group
Add workspace Viewer or Report Viewer AD Group inside the Copilot AD Group.
[ ]
9
Verify Nesting Structure
Ensure only AD groups are nested; no direct user assignments.
[ ]
10
Submit RITM Request
Submit the request form to link your Workspace Copilot AD Group to the Tenant-Level Copilot Group.
[ ]
11
Tenant Team Validation
BusOps/Admin validates compliance and connects group to: O365_PowerBI_Copilot_<Capacity>
[ ]
12
Copilot Activation Confirmation
Workspace Owner receives confirmation that Copilot is active.
[ ]
13
Notify Users
Inform users that Copilot is enabled and provide CoE support inbox for questions.
[ ]


