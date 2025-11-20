- 👋 Hi, I’m Manohar Chekka
- 👀 I’m Data Engineer
- 📫 Reach me at : manoharch0698@gmail.com


<!---
MC140/MC140 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
Patch(
    'Power BI Copilot Attestations',
    Defaults('Power BI Copilot Attestations'),
    {
        Title: txtUserName.Text,
        Email: txtUserEmail.Text,
        LineOfBusiness: Dropdown1.Selected.Value,
        TrainingCompleted: Checkbox1.Value,
        AgreementAccepted: Checkbox2.Value,
        SubmittedOn: Now()
    }
);

Notify(
    "Your attestation has been successfully submitted.",
    NotificationType.Success,
    3000
);