+++
title = 'Entra ID application permission report with usage'
date = 2024-10-30T21:53:10+02:00
+++

Enterprise applications and app registrations are two of the most dangerous things in Entra ID if managed incorrectly. They grant access to your data and resources, and if compromised, can lead to data breaches and other security incidents. Do you have control over all permissions granted over the course of your tenants lifetime? When was the last time you deleted stale permissions?

### Application permission report

My collegue Sridhar linked me a PowerShell module called [MSIdentity Tools](https://azuread.github.io/MSIdentityTools/), which contains a command called [Export-MsIdAppConsentGrantReport](https://azuread.github.io/MSIdentityTools/commands/Export-MsIdAppConsentGrantReport/). This command exports a report of all application permissions granted in the tenant, together with some additional helpful information related to risk.

As I have previously created a script for extracting application usage information from the audit logs, I decided to combine the two to easily see which applications are stale.

Pre-requisites: You need to have set up export of audit logs to a Log Analytics workspace. You can follow the guide [here](https://learn.microsoft.com/en-us/entra/identity/monitoring-health/howto-integrate-activity-logs-with-azure-monitor-logs#send-logs-to-azure-monitor). Two main reasons are longer history, and the ability to query the logs using Kusto Query Language (KQL) which is so so much faster for large tenants. Most companies already export this data for usage in Sentinel or other monitoring tools, so check with your security team if you want access. In this example I have hardcoded 180d usage logs, but you can change this to whatever you want.

```PowerShell
$tenantId = <tenantId>
$subscription = <subscription>
$resourceGroupName = <resourceGroupName>
$logAnalyticsWorkspace = <logAnalyticsWorkspace>

Connect-AzAccount -Tenant $tenantId
set-AzContext $subscription
$workspace = Get-AzOperationalInsightsWorkspace -ResourceGroupName $resourceGroupName -Name $logAnalyticsWorkspace

$kqlQuery = '
AADServicePrincipalSignInLogs
| where TimeGenerated > ago(180d)
| where ResultType == 0
| summarize count() by AppId
'
$AADServicePrincipalSignInLogs = Invoke-AzOperationalInsightsQuery -Workspace $Workspace -Query $kqlQuery

$kqlQuery = '
AADNonInteractiveUserSignInLogs
| where TimeGenerated > ago(180d)
| where ResultType == 0
| summarize count() by AppId
'
$AADNonInteractiveUserSignInLogs = Invoke-AzOperationalInsightsQuery -Workspace $Workspace -Query $kqlQuery

$kqlQuery = '
AADManagedIdentitySignInLogs
| where TimeGenerated > ago(180d)
| where ResultType == 0
| summarize count() by AppId
'
$AADManagedIdentitySignInLogs = Invoke-AzOperationalInsightsQuery -Workspace $Workspace -Query $kqlQuery

$kqlQuery = '
SigninLogs
| where TimeGenerated > ago(180d)
| where ResultType == 0
| summarize count() by AppId
'
$SigninLogs = Invoke-AzOperationalInsightsQuery -Workspace $Workspace -Query $kqlQuery

$signinlogshash = @{}
$signinlogs.results | ForEach-Object {
    if($signinlogshash[$_."AppId"]) {
        $signinlogshash[$_."AppId"] += [int]$_."count_"
    }
    else {
        $signinlogshash.Add($_."AppId",[int]$_."count_")
    }
}

$AADServicePrincipalSignInLogs.results | ForEach-Object {
    if($signinlogshash[$_."AppId"]) {
        $signinlogshash[$_."AppId"] += [int]$_."count_"
    }
    else {
        $signinlogshash.Add($_."AppId",[int]$_."count_")
    }
}

$AADNonInteractiveUserSignInLogs.results | ForEach-Object {
    if($signinlogshash[$_."AppId"]) {
        $signinlogshash[$_."AppId"] += [int]$_."count_"
    }
    else {
        $signinlogshash.Add($_."AppId",[int]$_."count_")
    }
}

$AADManagedIdentitySignInLogs.results | ForEach-Object {
    if($signinlogshash[$_."AppId"]) {
        $signinlogshash[$_."AppId"] += [int]$_."count_"
    }
    else {
        $signinlogshash.Add($_."AppId",[int]$_."count_")
    }
}
```

The above code generates a hash table with the AppId as the key and the total number of successful sign-ins as the value.

Next, we will generate the app consent report using Export-MsIdAppConsentGrantReport and add the sign-in count to the report.

```PowerShell
# Pre-requisites:
# Install-Module -Name MSIdentityTools
# Install-Module ImportExcel

Connect-MgGraph -Scopes Directory.Read.All

$appConsent = Export-MsIdAppConsentGrantReport -ReportOutputType PowerShellObjects

$appConsent | ForEach-Object {
    if($signinlogshash[$_.AppId]) {
        $_ | Add-Member -MemberType NoteProperty -Name SignInCount -Value $signinlogshash[$_.AppId]
    }
    else {
        $_ | Add-Member -MemberType NoteProperty -Name SignInCount -Value 0
    }
}

$appConsent | Export-Excel '.\appConsentReportWithSignIns.xlsx'
```

This report is just an example of how you can combine different data sources to get a better understanding of your environment. If you need to understand who the users of the application are, you can either check the sign in logs in the GUI or export the full usage logs using KQL.

Example KQL query for getting the users of an application from the SigninLogs:

```KQL
SigninLogs
| where TimeGenerated > ago(180d)
| where UserPrincipalName has ""@""
| where ResultType == 0
| where AppId == '$_'
| summarize signInCount = count() by UserPrincipalName
| sort by signInCount desc
```

Full code can be found [here](https://github.com/JrndD/Nifty-scripts-for-Entra-ID-and-Microsoft-365/blob/main/applicationConsentWith180dSignIns.ps1).