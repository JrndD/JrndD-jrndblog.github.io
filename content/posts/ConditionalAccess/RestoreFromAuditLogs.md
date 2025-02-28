+++
title = 'Restore condtitional access policies from audit log'
date = 2025-02-28T21:02:21+02:00
[params.cover]
  image = 'ConditionalAccess/CAPolicyChangesDiff.png'
  alt = 'Conditional Access policy change diff'
+++
+++

From time to time I need to restore Conditional Access policies that have been deleted or changed. If you don't have a backup of the policies, you can luckily use the audit logs to help you restore.

Up until now I have just crawled my way through KQL queries to find the information I need, but a recent Entra ID preview feature allowing you to view each change as a colored diff gave me an idea. What if I could create a script to compare the policy at any point in time as far back as I store my audit logs?

```PowerShell
$tenantId = <tenantId>
$subscription = <subscription>
$resourceGroupName = <resourceGroupName>
$logAnalyticsWorkspace = <logAnalyticsWorkspace>

Connect-AzAccount -Tenant $tenantId
set-AzContext $subscription
$workspace = Get-AzOperationalInsightsWorkspace -ResourceGroupName $resourceGroupName -Name $logAnalyticsWorkspace

# Query audit logs for conditional access policy changes
$kqlQuery = '
AuditLogs
| where TimeGenerated > ago(365d)
| where OperationName in ("Update conditional access policy","Delete conditional access policy","Add conditional access policy")
'
$CAlogs = (Invoke-AzOperationalInsightsQuery -Workspace $Workspace -Query $kqlQuery).results

# Create a somewhat readable list of changes
$policyList = @()
foreach ($log in $CAlogs) {
    $policy = [PSCustomObject]@{
        TimeGenerated = $log.TimeGenerated
        OperationName = $log.OperationName
        Id = ($log.TargetResources | ConvertFrom-Json).id
        DisplayName = ($log.TargetResources | ConvertFrom-Json).displayName
    }
    $policyList += $policy
}

$policyList
```

The variable $policyList now contains all changes to Conditional Access policies over the time specified in the query. Once you have decided which policy you are interested in, fetch the Id and enter into the next part of the script.

```PowerShell
# Select a policy to analyze
$id = <policyId>
$selectedPolicies = $CAlogs | Where-Object { ($_.TargetResources | ConvertFrom-Json).id -eq $id }

# Per timegenerated, id and displayname, get the old and the new configuration
$selectedPolicy = [PSCustomObject]@{
    Id = $id
    DisplayName = ($selectedPolicies[0].TargetResources | ConvertFrom-Json).displayName
    Changes = @()
}

foreach ($log in $selectedPolicies) {
    $change = [PSCustomObject]@{
        TimeGenerated = $log.TimeGenerated
        Operation = $log.OperationName
        oldValue = ($log.TargetResources | ConvertFrom-Json).modifiedProperties.oldValue
        newValue = ($log.TargetResources | ConvertFrom-Json).modifiedProperties.newValue
    }
    $selectedPolicy.Changes += $change
}

# List out TimeGenerated, Operation and whether oldValue and newValue are present
$selectedPolicy.Changes | Select-Object TimeGenerated, Operation, @{Name="OldValue"; Expression={if ($_.oldValue) { "Present" } else { "Not Present" } }}, @{Name="NewValue"; Expression={if ($_.newValue) { "Present" } else { "Not Present" } }}
```

Now we have a list of changes, and when the changes occured.

![List of changes](/ConditionalAccess/CAPolicyChanges.png)

Copy the TimeGenerated and whether you are interested in the old or the new value into the next part.

```PowerShell
# First comparison
$timeGenerated1 = '2025-01-14T06:44:58.8148868Z'
$beforeOrAfterChange1 = 'newValue'

# Second comparison
$timeGenerated2 = '2025-02-20T11:36:51.2522837Z'
$beforeOrAfterChange2 = 'oldValue'

$comparison1 = ($selectedPolicy.Changes | Where-Object { $_.TimeGenerated -eq $timeGenerated1 } | Select-Object $beforeOrAfterChange1).$beforeOrAfterChange1 | ConvertFrom-Json | ConvertTo-Json -Depth 10
$comparison2 = ($selectedPolicy.Changes | Where-Object { $_.TimeGenerated -eq $timeGenerated2 } | Select-Object $beforeOrAfterChange2).$beforeOrAfterChange2 | ConvertFrom-Json | ConvertTo-Json -Depth 10

$f1 = New-TemporaryFile
$f2 = New-TemporaryFile

$comparison1 | Out-File $f1.FullName
$comparison2 | Out-File $f2.FullName

# Check for VS Code in common installation paths
$vsCodePaths = @(
    "C:\Program Files\Microsoft VS Code\Code.exe",
    "${env:LOCALAPPDATA}\Programs\Microsoft VS Code\Code.exe",
    "${env:ProgramFiles}\Microsoft VS Code\Code.exe"
)

$vsCodePath = $vsCodePaths | Where-Object { Test-Path $_ } | Select-Object -First 1

if ($vsCodePath) {
    & $vsCodePath --diff $f1.FullName $f2.FullName
} else {
    Write-Host "VS Code not found." -ForegroundColor Yellow
}

Write-Host "Removing $($f1.FullName) and $($f2.FullName)"

start-sleep -Seconds 10

Remove-Item $f1.FullName
Remove-Item $f2.FullName
```

And voilà, you now have a diff of the changes made to the policy.

![List of changes](/ConditionalAccess/CAPolicyChangesDiff.png)

This script can be modified in several ways to give other insights. Comparing two different policies to see the differences comes to mind as one example.

It should also be a relatively small effort to modify the script to automatically restore a policy to a point in time, but thats a story for another day.