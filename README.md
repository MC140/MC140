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
        Title: TextInput1.Text,             // Replace with your actual User Name input name
        Email: TextInput2.Text,             // Replace with your actual Email input name
        'Line of Business': Dropdown1.Selected.Value,
        'Training Completed': Checkbox1.Value,
        'Attested to Agreement': Checkbox2.Value,
        'Date & Time': Now()               // Ensure SharePoint column is named "Date & Time" or "Date"
    }
);
Notify("Your attestation has been successfully submitted.", NotificationType.Success, 3000);


