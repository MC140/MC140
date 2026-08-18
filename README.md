- 👋 Hi, I’m Manohar Chekka
- 📫 Reach me at : manoharch0698@gmail.com


<!---
MC140/MC140 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->

1. Open Power Automate and sign in with your work account.
2. Select My flows → New flow → Scheduled cloud flow.
3. Enter:
    * Flow name: Friday Timesheet Reminder
    * Repeat every: 1 Week
    * Starting: Choose the next Friday at 1:00 PM
4. Select Create.
5. Open the Recurrence trigger and configure:
    * Time zone: (UTC-05:00) Eastern Time (US & Canada)
    * On these days: Friday
    * At these hours: 13
    * At these minutes: 00
6. Select + New step.
7. Search for Microsoft Teams.
8. Choose Post message in a chat or channel.
9. Configure:
    * Post as: Flow bot
    * Post in: Channel
    * Team: Select your Team
    * Channel: Select the appropriate channel, such as General
10. Enter this message:

⏰ Friday Timesheet Reminder
Please complete and submit your timesheet before the end of the day today. Thank you!