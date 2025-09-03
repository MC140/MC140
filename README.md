- 👋 Hi, I’m Manohar Chekka
- 👀 I’m Data Engineer
- 📫 Reach me at : manoharch0698@gmail.com

<!---
MC140/MC140 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
🚨 Risks and Consequences of Giving Build Access on Confidential Models

🔓 1. Data Exposure Outside VDI
	•	Build access allows users to connect from Power BI Desktop or Power BI Service, including outside the VDI if not restricted at the tenant level.
	•	If a developer opens Power BI Desktop on a personal laptop, they can connect to the shared dataset and download sensitive data — bypassing VDI security protocols.

🔥 Biggest risk: Once they connect, the entire data model can be queried, and RLS may or may not be enforced depending on configuration.

⸻

📤 2. Data Export and Download Risk
	•	With Build access:
	•	Users can export data to Excel or CSV.
	•	They can create composite models, which means importing part of your confidential data into their local PBIX file — which can be saved, emailed, or uploaded elsewhere.
	•	Even if export to Excel is blocked, composite models can be published from personal workspaces, leading to untraceable data leakage.

⸻

🧱 3. No Network or Location Restriction on Dataset Usage
	•	Power BI doesn’t restrict Build access by location (e.g., VDI-only) unless you configure strict conditional access policies or Azure AD restrictions.
	•	So even if the original semantic model was created inside VDI, anyone with Build access can pull it outside — unless blocked at the network or Azure tenant level.

⸻

👥 4. RLS Bypass via Composite Models (if not careful)
	•	Developers can create new calculated tables or relationships in their own model, potentially defeating RLS logic if:
	•	RLS is poorly configured
	•	Composite model is not RLS-aware
	•	They use a technique to infer restricted data (e.g., inference via slicers or joins)

⸻

🔍 5. Lack of Audit for Local Use
	•	Once data is connected in Power BI Desktop, usage is not logged (as discussed earlier).
	•	You won’t know:
	•	Who queried what
	•	Whether they exported data
	•	Where the PBIX was saved
