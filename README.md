- 👋 Hi, I’m Manohar Chekka
- 👀 I’m Data Engineer
- 📫 Reach me at : manoharch0698@gmail.com


<!---
MC140/MC140 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
Patch(
    'Rahona Power BI Copilot Attestation',
    Defaults('Rahona Power BI Copilot Attestation'),
    {
        Title: txtUserName.Text,
        Email: txtUserEmail.Text,
        LineofBusiness: Dropdown1.Selected.Value,
        TrainingCompleted: Checkbox1.Value,
        AttestedtoAgreement: Checkbox2.Value,
        Date_x0020__x0026__x0020_Time: Now()
    }
);

Notify(
    "Your attestation has been successfully submitted.",
    NotificationType.Success,
    3000
);

