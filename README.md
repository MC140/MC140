- 👋 Hi, I’m Manohar Chekka
- 👀 I’m Data Engineer
- 📫 Reach me at : manoharch0698@gmail.com

<!---
MC140/MC140 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
-- Base helpers
Views Flag :=
VAR op = SELECTEDVALUE('Activity Logs'[Operation])
RETURN IF(op IN {"ViewReport","OpenReport","ViewDashboard","Explore","Read"}, 1, 0)

Publish Flag :=
IF('Activity Logs'[Operation] = "Publish", 1, 0)

Is Report :=
IF('Activity Logs'[Artifact Kind] = "Report", 1, 0)

Is Dataset Refresh Success :=
IF('Activity Logs'[Operation] IN {"RefreshDatasetCompleted","ScheduledRefreshCompleted"}, 1, 0)

Is Dataset Refresh Failed :=
IF('Activity Logs'[Operation] IN {"RefreshDatasetFailed","ScheduledRefreshFailed"}, 1, 0)

Copilot Request :=
IF('Activity Logs'[Operation] = "RequestCopilot", 1, 0)

-- KPIs
Total Views :=
SUMX('Activity Logs', [Views Flag])

Unique Users :=
DISTINCTCOUNT('Activity Logs'[User ID])

MAU (30d) :=
CALCULATE([Unique Users], DATESINPERIOD('Dim_Date'[Date], MAX('Dim_Date'[Date]), -30, DAY))

Reports Published (30d) :=
CALCULATE(
    SUMX('Activity Logs', [Publish Flag]),
    FILTER('Activity Logs', [Is Report] = 1),
    DATESINPERIOD('Dim_Date'[Date], MAX('Dim_Date'[Date]), -30, DAY)
)

Active Workspaces (30d) :=
CALCULATE(
    DISTINCTCOUNT('Activity Logs'[Workspace ID]),
    DATESINPERIOD('Dim_Date'[Date], MAX('Dim_Date'[Date]), -30, DAY)
)

Refresh Success (30d) :=
CALCULATE(SUMX('Activity Logs',[Is Dataset Refresh Success]),
    DATESINPERIOD('Dim_Date'[Date], MAX('Dim_Date'[Date]), -30, DAY)
)

Refresh Failed (30d) :=
CALCULATE(SUMX('Activity Logs',[Is Dataset Refresh Failed]),
    DATESINPERIOD('Dim_Date'[Date], MAX('Dim_Date'[Date]), -30, DAY)
)

Refresh Success Rate % :=
DIVIDE([Refresh Success (30d)], [Refresh Success (30d)] + [Refresh Failed (30d)])

Copilot Requests (30d) :=
CALCULATE(SUMX('Activity Logs', [Copilot Request]),
    DATESINPERIOD('Dim_Date'[Date], MAX('Dim_Date'[Date]), -30, DAY)
)

-- Data Source mix (model snapshot measures)
Datasets Count := DISTINCTCOUNT('Dim_Datasets'[Dataset Id])

Datasets by SQL :=
CALCULATE(
    DISTINCTCOUNT('Dim_DatasourcesID'[Dataset Id]),
    'Dim_Datasources'[Datasource Type] = "Sql"
)
Datasets by File :=
CALCULATE(
    DISTINCTCOUNT('Dim_DatasourcesID'[Dataset Id]),
    'Dim_Datasources'[Datasource Type] = "File"
)
Datasets by OneLake :=
CALCULATE(
    DISTINCTCOUNT('Dim_DatasourcesID'[Dataset Id]),
    'Dim_Datasources'[Datasource Type] IN {"OneLake","Dataverse","Fabric"}
)
Datasets by Other :=
[Datasets Count] - [Datasets by SQL] - [Datasets by File] - [Datasets by OneLake]

-- Gateway dependency (if gateway id/flag exists)
Datasets requiring Gateway :=
CALCULATE(
    DISTINCTCOUNT('Dim_DatasourcesID'[Dataset Id]),
    NOT ISEMPTY('Dim_DatasourcesID'[Gateway ID])
)
Cloud-native Datasets := [Datasets Count] - [Datasets requiring Gateway]

-- RLS coverage (proxy: dataset has roles in security table if available; else from ReportUsers/permissions)
Datasets with RLS :=
DISTINCTCOUNT( FILTERS_SECURITY_TABLE[Dataset Id] )  -- replace with your security/roles table if present
RLS Coverage % :=
DIVIDE([Datasets with RLS], [Datasets Count])

-- Top workspaces
Workspace Views :=
CALCULATE([Total Views], ALLEXCEPT('Activity Logs','Activity Logs'[Workspace ID],'Activity Logs'[Workspace Name]))
Workspace Consumers :=
CALCULATE(DISTINCTCOUNT('Activity Logs'[User ID]), ALLEXCEPT('Activity Logs','Activity Logs'[Workspace ID],'Activity Logs'[Workspace Name]))