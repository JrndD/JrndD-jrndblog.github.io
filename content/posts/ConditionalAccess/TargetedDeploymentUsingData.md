+++
title = 'Conditional Access - Deploying new policies using data'
date = 2025-02-17T08:36:19+02:00
[params.cover]
  image = 'ConditionalAccess/Device-code-phishing-attack-chain.webp'
  alt = 'Getting lost in the version history'
+++
Image source: [(Storm-2372 conducts device code phishing campaign)](https://www.microsoft.com/en-us/security/blog/2025/02/13/storm-2372-conducts-device-code-phishing-campaign/)

Last weeks device code phishing campaign "Storm-2372" forced many organizations to quickly implement countermeasures to block the attack. While I hope most organizations has already had the time to implement the necessary changes, I wanted to share a few tips on how to use data to target your deployment. Both for this specific campaign, but also for other similar controls.

## Conditional Access

Conditional Access is a powerful tool to control access to your resources. It can be used to enforce MFA, block legacy authentication, or block access from certain locations. In this example, we want to block access to using the device code flow for all users. But what if you don't know if device code flow is used in your organization today? Let's dive into the data available to us to investigate.

![Block device code flow using conditional access](/ConditionalAccess/CABlockdevicecodeflow.png)

## Data sources

The easiest way to implement a new Conditional Access policy is by creating a new policy in report-only mode. This will allow you to see exactly who would be affected by the policy, but has the downside of only collecting data from the day you created the policy.

If you have created a policy called "CA019 Baseline: Block device code flow" in report-only mode, you can use the following query to see who would be affected by the policy.

```KQL
SigninLogs
| where TimeGenerated > ago (7d)
| where ResultType == 0
| mvexpand ConditionalAccessPolicies
| where ConditionalAccessPolicies["result"] == "reportOnlyFailure"
| where ConditionalAccessPolicies.displayName == "CA019 Baseline: Block device code flow"
```

Look for AppDisplayName to see which applications are using the device code flow, and the UserPrincipalName to see which users would be affected. The output of this result can then be used in an exclusion list for the policy, so that you safely can enable the policy in block mode for the rest of the organization while you work on the exclusions. Don't let perfect be the enemy of fast and good enough.

If you don't have a policy already in place, we can use similar queries to investigate the sign-in logs.

```KQL
SigninLogs
| where TimeGenerated > ago(30d)
| where AuthenticationProtocol=="deviceCode"
| where ResultType == 0
```

Again, look for AppDisplayName and UserPrincipalName to create your exclusion list for the policy. 

The example of device code flow is just one example, but the same principles can be applied to any other control you want to implement. Below I will share a few real life examples I myself have used.

## Other KQL examples

Do you want to restrict Mobile Apps and Desktop clients from unmanaged Windows devices, and want to send targeted communications to users who would be affected? Run this query and get all the data you need.

```KQL
SigninLogs
| where TimeGenerated > ago(30d)
| where ResultType == 0
| where DeviceDetail.trustType <> "Hybrid Azure AD joined"
| where ClientAppUsed == "Mobile Apps and Desktop clients"
| where ConditionalAccessStatus <> "notApplied"
| where DeviceDetail.isCompliant <> true
| where DeviceDetail.operatingSystem in ('Windows','Windows10','Windows7','Windows8Dot1')
//| summarize count() by AppDisplayName
//| summarize count() by UserPrincipalName
```

Want to proactively help users who has never had a single successful MFA sign-in before tightening the MFA requirements?

```KQL
SigninLogs
| where TimeGenerated > ago(90d)
| where AuthenticationRequirement == 'multiFactorAuthentication'
| summarize SignInSuccess = make_set(ResultType) by UserPrincipalName
| where SignInSuccess !has '0'
```