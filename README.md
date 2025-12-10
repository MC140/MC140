- 👋 Hi, I’m Manohar Chekka
- 👀 I’m Data Engineer
- 📫 Reach me at : manoharch0698@gmail.com


<!---
MC140/MC140 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->

Power BI Copilot Enablement – Pre-Work Checklist for Workspace Owners

Before submitting the final Copilot enablement request, Workspace Owners must complete all steps below.
This ensures proper governance, attestation, AD group structure, and approval routing.

⸻

✅ 1. Identify Eligible Workspaces
	•	Review all your team’s workspaces in Staging and Live.
	•	Confirm which workspaces need Copilot access.

⸻

✅ 2. Create Workspace-Level Copilot AD Group
	•	Create a new AD group following the enterprise naming convention:
PBI-CPLT-<LOB>-<WorkspaceName>
	•	This group will control Copilot access for your workspace.

⸻

✅ 3. Nest Functional AD Groups Into the Workspace Copilot Group

For each workspace where Copilot is required:
	•	Nest the following groups (as applicable):
	•	Contributor AD Groups
	•	Viewer AD Groups
	•	Report-Level AD Groups
	•	Nest groups from:
	•	Live workspace
	•	Staging workspace, if Copilot access is required there as well

📌 Each nesting request triggers an approval flow → Reporting Manager → AD Group Owner. Workspace Owners must monitor approval completion.

⸻

✅ 4. Complete Mandatory Attestations

Workspace Owners must ensure:

4.1 Workspace Owner Attestation
	•	Must be attested to the Foundational Power BI User Agreement.

4.2 All AD Group Members Attestation
	•	Every member inside the nested AD groups must:
	•	Attest to the Copilot User Agreement
	•	Read and acknowledge the Copilot DOs & DON’Ts Guide

4.3 Ongoing Compliance Check
	•	If new users are added to functional AD groups in future:
	•	Workspace Owner must verify their attestation status.
	•	Must maintain a local log of attested members.

⸻

✅ 5. Nest Workspace Copilot Group Into the Tenant-Level Copilot Group
	•	Nest your workspace’s Copilot AD group (created in Step 2) into:
PBI-CPLT-O365-TENANT
	•	This allows the workspace to inherit tenant-level Copilot permissions.

📌 Approval routing again goes through Manager → AD Group Owner. Workspace Owner must track this.

⸻

✅ 6. Submit Copilot Enablement Intake Form

Once all steps above are complete and the tenant-level nesting ticket number is available:
	•	Submit the final enablement request using the Intake Form.
	•	Enter the ticket number from Step 5 in the required field.

📎 This is the only ticket the CoE team accepts — pre-work tickets will not be processed.

⸻

Important Note for Workspace Owners

Many users are skipping the pre-work and submitting the form prematurely.
This checklist ensures:
	•	Proper governance
	•	Proper attestation
	•	Correct AD group hierarchy
	•	Successful approval workflow

Only after completing all tasks in this checklist should the final intake form be submitted.

⸻

📨 Copilot Enablement Intake Form

👉 Submit here: [Insert Intake Form Link]

