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
    "Dear <b>" & User().FullName & "</b>,<br><br>" &
    "This is a confirmation that you have successfully completed the Rahona Power BI Copilot Attestation.<br><br>" &
    "<b>Date Submitted:</b> " & Text(Now(), "yyyy-mm-dd") & "<br>" &
    "<b>Line of Business:</b> " & Dropdown1.Selected.Value & "<br>" & 
    "<b>Workspace Name:</b> " & TextInput3.Text & "<br><br>" &
    "Thank you,<br>" &
    "<b>Rahona Power BI Center of Excellence</b>",
    {IsHtml: true}
)
