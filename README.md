# Tenant Level Logging

<h2>Purpose</h2>

- Enable logging for Azure Active Directory/Microsoft Entra ID.
- Analyze logs on LAW using KQL queries.
  
<img src="https://raw.githubusercontent.com/melisaaaaaaaaa-er/tenant-level-logging-images/main/Tenant%20Level%20Logging%20diagram.png"/>

<img src="https://raw.githubusercontent.com/melisaaaaaaaaa-er/tenant-level-logging-images/main/Tenant%20Level%20Logging%20diagram%202.png"/>

<img src="https://raw.githubusercontent.com/melisaaaaaaaaa-er/tenant-level-logging-images/main/Tenant%20Level%20Logging%20diagram%203.png"/>

#
<h2>Setup Logging for Microsoft Entra ID/Azure AD</h2>

<h3>Creating Diagnostic Setting to ingest Microsoft Entra ID Logs – Enable Audit Logs and Sign-in Logs</h3>

<img src="https://raw.githubusercontent.com/melisaaaaaaaaa-er/tenant-level-logging-images/main/1.png"/>

<img src="https://raw.githubusercontent.com/melisaaaaaaaaa-er/tenant-level-logging-images/main/2.png"/>

<img src="https://raw.githubusercontent.com/melisaaaaaaaaa-er/tenant-level-logging-images/main/3.png"/>

Go to Diagnostic Settings and select “+Add diagnostic setting”.

Give the Diagnostic Setting a name. Under Logs Categories select AuditLogs and SignInLogs. For the Destination details select “Send to Log Analytics Workspace” – choose the respective Subscription and LAW you want to send the logs to.

Select “Save”.

<img src="https://raw.githubusercontent.com/melisaaaaaaaaa-er/tenant-level-logging-images/main/4.png"/>

<img src="https://raw.githubusercontent.com/melisaaaaaaaaa-er/tenant-level-logging-images/main/4.5.png"/>

Go to the LAW blade and check Tables to make sure the AuditLogs and SignInLogs tables have been created.

#
<h2>Generating Logs for Analysis</h2>

<h3>Creating a fake user account to generate logs</h3>

<img src="https://raw.githubusercontent.com/melisaaaaaaaaa-er/tenant-level-logging-images/main/5.png"/>

<img src="https://raw.githubusercontent.com/melisaaaaaaaaa-er/tenant-level-logging-images/main/6.png"/>

Create a new user account (dummy_user) and sign into it in an incognito window.

<img src="https://raw.githubusercontent.com/melisaaaaaaaaa-er/tenant-level-logging-images/main/7.png"/>

<img src="https://raw.githubusercontent.com/melisaaaaaaaaa-er/tenant-level-logging-images/main/8.png"/>

<img src="https://raw.githubusercontent.com/melisaaaaaaaaa-er/tenant-level-logging-images/main/9.png"/>

Go back to the Azure Portal and assign dummy_user the Global Administrator role, then delete the account.

Each of these steps will have generated tenant level logs within Azure.

#
<h3>Log Analysis</h3>

<img src="https://raw.githubusercontent.com/melisaaaaaaaaa-er/tenant-level-logging-images/main/10.png"/>

Go to the LAW blade and go to Logs to view all the logs that have been generated from the previous activity using the query “AuditLogs”.

There is a record of every step from creation of the account, to sign-in, permission escalation, and deletion.

<img src="https://raw.githubusercontent.com/melisaaaaaaaaa-er/tenant-level-logging-images/main/12.png"/>

<img src="https://raw.githubusercontent.com/melisaaaaaaaaa-er/tenant-level-logging-images/main/13.png"/>

Expanding the logs provides more detail about them.

Here we can see the user account responsible for initiating the addition of a user account.

<img src="https://raw.githubusercontent.com/melisaaaaaaaaa-er/tenant-level-logging-images/main/15.png"/>

Here we can see a record of the privilege escalation of dummy_user.

<img src="https://raw.githubusercontent.com/melisaaaaaaaaa-er/tenant-level-logging-images/main/16.png"/>

Here we can see the record of when dummy_user was deleted.

#
<h3>A Deeper Analysis</h3>

<h4>AuditLogs</h4>

<img src="https://raw.githubusercontent.com/melisaaaaaaaaa-er/tenant-level-logging-images/main/17.png"/>

AuditLogs

| where OperationName == "Add member to role" and Result == "success"

| where TargetResources[0].modifiedProperties[1].newValue == '"Global Administrator"' or TargetResources[0].modifiedProperties[1].newValue == '"Company Administrator"'

| order by TimeGenerated desc

| project TimeGenerated, OperationName, AssignedRole = TargetResources[0].modifiedProperties[1].newValue, Status = Result, TargetResources

Here are the results from the KQL query written above. We’ll go over each line one by one to and break down  the meaning of this query.

#
<h4>AuditLogs</h4>

<img src="https://raw.githubusercontent.com/melisaaaaaaaaa-er/tenant-level-logging-images/main/18.png"/>

Separate the top query AuditLog to run it separately from the rest of the queries. 

Highlighting a single query line will also allow you to run them on their own and display the results of that one line.

The AuditLogs query brings up the whole AuditLogs table that we configured to do so earlier.

#
<h4>| where OperationName == "Add member to role" and Result == "success"</h4>

<img src="https://raw.githubusercontent.com/melisaaaaaaaaa-er/tenant-level-logging-images/main/19.png"/>

<img src="https://raw.githubusercontent.com/melisaaaaaaaaa-er/tenant-level-logging-images/main/20.png"/>

This query puts out only the records from the AuditLogs where the OperationName parameter is “Add member to role” and the Result parameter is “Success”. I.e. this query refers to successfully adding the dummy_user account to the Global Administrator role.

#
<h4>| where TargetResources[0].modifiedProperties[1].newValue == '"Global Administrator"' or TargetResources[0].modifiedProperties[1].newValue == '"CompanyAdministrator"'</h4>

<img src="https://raw.githubusercontent.com/melisaaaaaaaaa-er/tenant-level-logging-images/main/21.png"/>

In this query, we can bring up more detailed results for the prior clause when expanding the result, i.e where the target resource (dummer_user) was successfully assigned to the role of Global Administrator/Company Administrator (another term for Global Administrator on Azure).

#
<h4>| order by TimeGenerated desc</h4>

<img src="https://raw.githubusercontent.com/melisaaaaaaaaa-er/tenant-level-logging-images/main/22.png"/>

| project TimeGenerated, OperationName, AssignedRole = TargetResources[0].modifiedProperties[1].newValue, Status = Result, TargetResources

These last two lines have to do with organizing the results that come up and narrowing down the scope of the results.

The 4th line of the query displays the results by time in descending order (from earliest to latest). Because only 1 results is generated from this query, it doesn’t make a difference to this particular query, but is useful for when several results come up.

<img src="https://raw.githubusercontent.com/melisaaaaaaaaa-er/tenant-level-logging-images/main/22.5.png"/>

Here is what the results look like when that’s the case.

<img src="https://raw.githubusercontent.com/melisaaaaaaaaa-er/tenant-level-logging-images/main/23.png"/>

The 5th line tells the query to bring up only the specified parameters in the results. Queries typically bring up the values for dozens of parameters. Using the “project” keyword will bring up results for only the parameters we actually want to see.

#
<h4>SignInLogs</h4>

<img src="https://raw.githubusercontent.com/melisaaaaaaaaa-er/tenant-level-logging-images/main/24.png"/>

<img src="https://raw.githubusercontent.com/melisaaaaaaaaa-er/tenant-level-logging-images/main/25.png"/>

<img src="https://raw.githubusercontent.com/melisaaaaaaaaa-er/tenant-level-logging-images/main/26.png"/>

Using the attacker account, sign-in to Azure once successfully on an incognito browser session, then log off and try to sign in again using faulty credentials to generate failed logon attempt logs.

#
<img src="https://raw.githubusercontent.com/melisaaaaaaaaa-er/tenant-level-logging-images/main/27.png"/>

As with the AuditLogs, we will analyze the query and its results for SignInLogs, breaking them down line-by-line to gain an understanding of what each query line does.

Here are the results for the full query.

<img src="https://raw.githubusercontent.com/melisaaaaaaaaa-er/tenant-level-logging-images/main/28.png"/>

The SignInLogs keyword brings up the whole SignInLogs table (configured eariler in the lab).

#
<h4>| where ResultDescription == "Invalid username or password or Invalid on-premise username or password."</h4>

<img src="https://raw.githubusercontent.com/melisaaaaaaaaa-er/tenant-level-logging-images/main/29.png"/>

The second line of the query filters and puts out records for failed login attempts only.

#
<h4>| extend location = parse_json(LocationDetails)</h4>

<img src="https://raw.githubusercontent.com/melisaaaaaaaaa-er/tenant-level-logging-images/main/30.png"/>

The third line of the query (along with the previous lines) filters and puts out location detail records for failed login attempts. Here, we can see the failed login attempt came from Cambridge, Ontario, Canada. 

#
<h4>| extend City = location.city, State = location.state, Country = location.countryOrRegion, Latitude = location.geoCoordinates.latitude, Longitude =location.geoCoordinates.longitude</h4>

<img src="https://raw.githubusercontent.com/melisaaaaaaaaa-er/tenant-level-logging-images/main/31.png"/>

This query line puts out more detailed results for location of the activity as well as displays them in an easier to read format.

#
<h4>| project TimeGenerated, ResultDescription, UserPrincipalName, AppDisplayName, IPAddress, IPAddressFromResourceProvider, City, State, Country, Latitude, Longitude</h4>

<img src="https://raw.githubusercontent.com/melisaaaaaaaaa-er/tenant-level-logging-images/main/32.png"/>

<img src="https://raw.githubusercontent.com/melisaaaaaaaaa-er/tenant-level-logging-images/main/33.png"/>

The final query line displays results for the indicated parameters in the query. This puts out results that are more relevant for the data we would like to see from the logs.
