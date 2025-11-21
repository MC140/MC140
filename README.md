- 👋 Hi, I’m Manohar Chekka
- 👀 I’m Data Engineer
- 📫 Reach me at : manoharch0698@gmail.com


<!---
MC140/MC140 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->

Office365Outlook.SendEmailV2(
    User().Email,
    "Confirmation: Power BI Copilot Attestation Submitted",
    "
    <html>
    <body style='font-family:Segoe UI, Arial; font-size:14px;'>

    <p>Dear <b>" & txtUserName.Text & "</b>,</p>

    <p>
    This is a confirmation that you have successfully completed the 
    <b>Rahona Power BI Copilot User Agreement Attestation</b>.
    </p>

    <p><b>Your Submission Details:</b></p>

    <table style='font-family:Segoe UI; font-size:14px; border-collapse:collapse;'>
        <tr><td style='padding:4px 8px;'><b>Date Submitted:</b></td>
            <td style='padding:4px 8px;'>" & Text(Now(), "yyyy-mm-dd hh:mm AM/PM") & "</td></tr>

        <tr><td style='padding:4px 8px;'><b>Line of Business:</b></td>
            <td style='padding:4px 8px;'>" & Dropdown1.Selected.Value & "</td></tr>

        <tr><td style='padding:4px 8px;'><b>Workspace Name:</b></td>
            <td style='padding:4px 8px;'>" & TextInput3.Text & "</td></tr>

        <tr><td style='padding:4px 8px;'><b>Training Completed:</b></td>
            <td style='padding:4px 8px;'>" & If(Checkbox1.Value,"Yes","No") & "</td></tr>

        <tr><td style='padding:4px 8px;'><b>Agreement Accepted:</b></td>
            <td style='padding:4px 8px;'>" & If(Checkbox2.Value,"Yes","No") & "</td></tr>
    </table>

    <br>

    <p>
    Thank you,<br>
    <b>Rahona Power BI Center of Excellence</b><br>
    TD Bank
    </p>

    </body>
    </html>
    ",
    { IsHtml: true }
);