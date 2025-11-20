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
        Line_x0020_of_x0020_Business: Dropdown1.Selected.Value,
        TrainingCompleted: Checkbox1.Value,
        AttestedToAgreement: Checkbox2.Value,
        SubmittedOn: Now()
    }
);

Notify(
    "Your attestation has been successfully submitted.",
    NotificationType.Success,
    3000
);

Patch(
    'Rahona Power BI Copilot Attestation',
    Defaults('Rahona Power BI Copilot Attestation'),
    {
        Title: User().FullName,
        Email: User().Email,
        'Line of Business': Dropdown1.Selected.Value,
        'Training Completed': Checkbox1.Value,
        'Attested to Agreement': Checkbox2.Value,
        'Date & Time': Now()
    }
);
Notify(
    "Your attestation has been successfully submitted.",
    NotificationType.Success,
    3000
);

// 1. Save to SharePoint
Patch(
    'Rahona Power BI Copilot Attestation',
    Defaults('Rahona Power BI Copilot Attestation'),
    {
        Title: User().FullName,
        Email: User().Email,
        'Line of Business': Dropdown1.Selected.Value,
        'Training Completed': Checkbox1.Value,
        'Attested to Agreement': Checkbox2.Value,
        'Date & Time': Now()
    }
);

// 2. Send Confirmation Email
Office365Outlook.SendEmailV2(
    User().Email,
    "Confirmation: Power BI Copilot Attestation Submitted",
    "Dear " & User().FullName & ",<br><br>" &
    "This is a confirmation that you have successfully completed the Rahona Power BI Copilot Attestation.<br><br>" &
    "<b>Date Submitted:</b> " & Text(Now(), ShortDate) & "<br>" &
    "<b>Line of Business:</b> " & Dropdown1.Selected.Value & "<br><br>" &
    "Thank you,<br>Power BI Center of Excellence"
);

// 3. Notify on Screen
Notify(
    "Success! You have attested and a confirmation email has been sent.",
    NotificationType.Success,
    4000
);
