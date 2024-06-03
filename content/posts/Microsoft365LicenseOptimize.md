+++
title = 'Optimizing Microsoft 365 licensing. Part 2 - License optimization'
date = 2024-06-03T22:40:59+03:00
ShowToc = true
[params.cover]
  image = 'Microsoft365Licensing/License optimization.jpg'
  alt = 'Investigating all the data'
+++

This article will dive into license optimization. Which reports and measurements are available to us, and what are the important metrics to guide the decision on which license package a user needs? Ultimately, we want to use this information to reduce cost, but it can also help us to discover if we are uncompliant according to the license contract.

If you haven't already, I recommend reading through part 1 of this series to get a brief understanding of license profiles. I will be referring to some terms introduced there. If you are already familiar with Microsoft 365 licensing, then part 2 is the place to start.

This article is part of a series on Microsoft 365 license optimization:
- Part 1 - [License basics](https://jorund.dahl.cloud/posts/microsoft365licensing/)
- Part 2 - [Data driven decisions. How to accurately measure license usage?](https://jorund.dahl.cloud/posts/microsoft365licenseoptimize/)
- Part 3 - Automation - Coming!

*Disclaimer:* Before diving into any usage data. Please remember that you are dealing with personal identifiable information (PII) and depending on your organization and country you might be limited in what you can do. Please consult with your legal and data privacy team before taking any action!

## Useful data sources

Microsoft provides almost all the data we need to track usage, this has improved a lot over the last few years, and I expect the reports to get even better as time goes by.

### Get familiar with the built in reports

The first place to investigate is the [usage reports in the Microsoft 365 admin center](https://admin.microsoft.com/#/reportsUsage). If your company obfuscates names in these reports, you should discuss with your legal or data privacy team if you can get limited access to read out the data.

I always export the reports over the last 180 days, but if you want to be more aggressive on license reallocation you can use 90 days activity. I would not recommend going any lower than that.

The main reports we are interested in are:
- [getM365AppUserDetail](https://learn.microsoft.com/en-us/graph/api/reportroot-getm365appuserdetail?view=graph-rest-1.0&tabs=http)
    - Office package (M365 Apps for Enterprise) usage per user
- [getOneDriveUsageAccountDetail](https://learn.microsoft.com/en-us/graph/api/reportroot-getonedriveusageaccountdetail?view=graph-rest-1.0&tabs=http)
    - OneDrive storage used per user
- [getMailboxUsageDetail](https://learn.microsoft.com/en-us/graph/api/reportroot-getmailboxusagedetail?view=graph-rest-1.0&tabs=http)
    - Mailbox storage used per user
- [getOffice365ActiveUserDetail](https://learn.microsoft.com/en-us/graph/api/reportroot-getoffice365activeuserdetail?view=graph-rest-1.0&tabs=http)
    - General activity in office tools per user

### Entra ID and AD sign in logs

Our second stop is analyzing the Entra ID sign-ins. I recommend storing the data in a log analytics workspace for longer retention and fast programmatic access using KQL, but if you don't have access to that then you can download the 30-day logs. Every Entra ID user has a "last signed in" field that you can use, but unfortunately this gets updated on failed sign ins! So, you cannot trust that this is correct. The better way of doing it is to see when the last sign in was using KQL.

The good old AD lastSignIn field is also a good indication on when last sign in happened. Just note that this field is also not 100% reliable, but it is a good indicator of activity. Last password reset can also be used if you are trying to identify stale accounts.

### Defender for Endpoint reports

If your company uses Sentinel and Defender for Endpoint, you might already export defender for endpoint events to log analytics. If so, then you have a gold mine when it comes to finding which users are signing in to Windows devices. Write a KQL query to export all users who sign in to your organizations Windows devices for the last 180 days. If you want to get fancy you can count the number of sign-ins to devices to see if someone with low usage can utilize the Microsoft 365 F3 license.

## Other factors to evaluate

From my experience, here are some other factors to remember:
- Get data from your HR tool to see who is on leave. You might not want to downgrade the license of these users.
- Create a list of VIP users that you don't want to automatically change licenses of.
- Non-person accounts like service accounts, shared mailboxes etc needs to be evaluated per usecase. Some service accounts might have license requirements that do not trigger any activity.
- Easily granted, easily taken away. It is easier to downgrade licenses if you have a fully automated and fast way for users to get licenses back.

## License profiles

### Office users

For office users signing in to a Windows device, the following criteria are the most important:
- Have they signed in to a Windows device in the last 180 days? If so, how many times?
- Have they used Microsoft 365 Apps for Enterprise (the office package) in the last 180 days?
- Are they active within the office tools (Teams, SharePoint, Viva Engage, Email)?
- For stale accounts, do they have any sign ins in AD, Entra ID etc for the last 180 days?
- What is their OneDrive size? (Office 365 F3 and E1 has different capacity, and a license downgrade can block them from using it)
- What is their Mailbox size? (Exchange Online P1 (50 GB) or P2 (100 GB) can be used when downgrading users with large mailboxes from E3 / E5 to F3)

### Field users

For field users signing in to a mobile device, the following criteria are the most important:
- Are they active within the office tools?
- For stale accounts, do they have any sign ins in AD, Entra ID etc for the last 180 days?

### Other licenses

#### Visio and Project

Both Visio and Project now have very detailed usage reports in the Microsoft 365 admin center. Filter away every user with any activity in the tools for the last 180 days and remove the licenses for the rest.

#### Enterprise Applications

If you are using Entra ID enterprise applications with SSO, and your application owners are forcing all sign-ins to go through Entra ID. Then you can create full overviews of all sign ins for your application owners based on the sign in logs as long as your retention is. 

Example, your organization uses a CAD tool costing 150 € per user per month. The application is set up solely using Entra ID SSO. Noone will be able to sign in to this application without it generating a sign in log. Filter on the application and remove all licenses not in use.

## Putting it all together

How much effort you choose to put into this depends will directly correlate with savings achieved. I fully believe that any company with over 10 000 users can have someone hired to only optimize Microsoft 365 licenses as a software asset manager, and they would pay for themselves. From the easy way of manually exporting and combining the data in Excel or Power BI once every year, to fully automated daily exports, you should at least start somewhere to get familiar with the data. 

In my current company everything is automated and fed in to a SQL database daily. We visualize the data using Power BI reports and can automatically upgrade and downgrade without any manual intervention. The added benefit is that the SQL database is great for speeding up other scripts, as the size of our company makes any read operation using graph extremely slow.

## Informing vs not informing about license changes

Once all of the above is done. You have combined all the sources of data you could find and you are confident in the license downgrades (and upgrades!) that you can achieve, you have to ask the question on whether to inform the user or not.

On one hand it is good to have transparency so that licenses are not suddenly disappearing, but on the other hand people tend to hoard licenses like a dragon and absolutely refuse to let go of them even if the usage data shows otherwise. I tend to remove some licenses without informing, while others we send a short informational email with link to the license request form if they ever need it back.

In the next part of this series, we will have a look at how to automate as much of this as possible. There is nothing described in this article that I haven't already automated, but some sources are easier to work with than others. Stay tuned for part 3!