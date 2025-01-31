+++
title = 'Entra ID - Add permissions to Managed Identity'
date = 2025-01-31T15:36:17+02:00
[params.cover]
  image = 'permissionsManagedIdentity/ManagedIdentityDemo.png'
  alt = 'Searching for permissions'
+++

Adding permissions to Managed Identities in Entra ID is annoyingly complicated. To make the job easier, I created a short script that allows me to search through all possible permissions in a simple GUI.

The script outputs the command needed to add the permission to the managed identity. If you feel confident, you can modify the script to run it directly.

```powershell
Connect-MgGraph -Scope Directory.Read.All, AppRoleAssignment.ReadWrite.All, Application.Read.All

# Object ID of the managed identity, NOT application ID!
$servicePrincipalId = "00000000-0000-0000-0000-000000000000"

# Get all service principals in tenant. Lazy first version. This can be filtered down into the actual useful applications.
$allServicePrincipal = Get-MgServicePrincipal -All

$roles = $allServicePrincipal | ForEach-Object {
    foreach($r in $_.AppRoles) {
        [PSCustomObject]@{
            AppId       = $_.Id
            DisplayName = $_.DisplayName
            Id          = $r.Id
            RoleName    = $r.DisplayName
            Value       = $r.Value
        }
    }
}

# List all roles and let the user select one (hit enter to continue in the gridview)
$roles | Out-GridView -PassThru -Title "Select the approle you want to assign to the managed identity" | ForEach-Object {
    $chosenRole = $_
}

@"
To grant $($chosenRole.RoleName) ($($chosenRole.Value)) from $($chosenRole.DisplayName) to the managed identity with object ID $servicePrincipalId, run the following:

`$params = @{
	principalId = $servicePrincipalId
	resourceId = $($chosenRole.AppId)
	appRoleId =  $($chosenRole.Id)
}
New-MgServicePrincipalAppRoleAssignment -ServicePrincipalId $servicePrincipalId -BodyParameter `$params
"@
```

Link to script on GitHub: [Add permissions to Managed Identity](https://github.com/JrndD/Nifty-scripts-for-Entra-ID-and-Microsoft-365/blob/main/AddPermissionsToManagedIdentity.ps1)