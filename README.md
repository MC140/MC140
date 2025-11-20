- 👋 Hi, I’m Manohar Chekka
- 👀 I’m Data Engineer
- 📫 Reach me at : manoharch0698@gmail.com


<!---
MC140/MC140 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->

If(
    !Checkbox1.Value 
        || !Checkbox2.Value
        || Dropdown1.Selected.Value = "-- Select Line of Business --"
        || IsBlank(Dropdown1.Selected.Value),
    DisplayMode.Disabled,
    DisplayMode.Edit
)

Notify(
    "Your attestation has been successfully submitted.",
    NotificationType.Success,
    3000
);
