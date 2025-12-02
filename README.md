- 👋 Hi, I’m Manohar Chekka
- 👀 I’m Data Engineer
- 📫 Reach me at : manoharch0698@gmail.com


<!---
MC140/MC140 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->

TransactionReportLink = 
VAR BaseURL = "https://app.powerbi.com/groups/me/reports/xxxx-xxxx-xxxx/ReportSection"
VAR SelectedUser = SELECTEDVALUE('UserTable'[UserID])

RETURN
IF(
    ISBLANK(SelectedUser), 
    BaseURL,  -- If no user is selected, open the report without filters
    BaseURL & "?filter=TransactionsTable/UserID eq '" & SelectedUser & "'"
)

TransactionReportLink = 
VAR BaseURL = "[https://app.powerbi.com/groups/me/reports/xxxx-xxxx-xxxx/ReportSection](https://app.powerbi.com/groups/me/reports/xxxx-xxxx-xxxx/ReportSection)"
VAR SelectedUser = SELECTEDVALUE('UserTable'[UserID])

RETURN
IF(
    ISBLANK(SelectedUser), 
    
    -- Scenario 1: No user selected. Return base URL (or link to a landing page).
    BaseURL, 
    
    -- Scenario 2: User selected. Append query string parameter.
    -- Syntax: ?filter=TableName/ColumnName eq 'Value'
    BaseURL & "?filter=TransactionsTable/UserID eq '" & SelectedUser & "'"
)
