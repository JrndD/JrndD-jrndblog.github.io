+++
title = 'Optimizing Microsoft 365 licensing. Part 1 - License basics'
date = 2024-05-25T21:02:16+03:00
ShowToc = true
[params.cover]
  image = 'Microsoft365Licensing/money_flying.webp'
  alt = 'Money flying out the window'
  caption = 'Time to pay the yearly EA invoice.'
+++

It's that time of the year. You have one month before the yearly license numbers are due to be delivered to Microsoft. The questions have started pouring in. Can we make any savings? Are we utilizing all the features we have purchased? Do we have enough licenses for all our workloads? Are we even compliant?

This guide focuses on the things I consider to be the most important when designing your license strategy. I have experience managing licenses for large global enterprises over several years, and can show to several million euros saved on Microsoft licenses for some companies in my career. Part 2 and 3 of this series is where the cool stuff happens, but I need to cover some of the license basics before we get there.

I am planning to divide this guide into three parts:
- Part 1 - [License basics](https://jorund.dahl.cloud/posts/microsoft365licensing/)
- Part 2 - [Data driven decisions. How to accurately measure license usage?](https://jorund.dahl.cloud/posts/microsoft365licenseoptimize/)
- Part 3 - Automation - Coming!

## Get your license profiles in order

License profiles are mandatory if you want to free yourself from micromanagement. Simply explained, the number of license profiles you need are the least common divisor of different license packages to cover all your employees needs without overspending on unecessary features.

A license profile usually corresponds with one of the core Microsoft license packages, but can be mixed and matched to suite your needs.

### License profile considerations

The things to consider when choosing a license package for a license profile is:

#### Tools the employee needs to do their job

Office 365 F3, E1, E3 and E5 contain features like email, Teams, OneDrive and M365 Apps for Enterprise at different capacities.
For a full feature comparison, check [Office 365 package comparison](https://m365maps.com/matrix.htm#111000100000000000000/).

The E5 package contains Power BI Pro and Teams Phone. I would generally not recommend going for the full E5 license unless you are already utilizing Purview, as most companies covers their needs with Office 365 E3 + EMS E3 + E5 Security + Windows device licenses.

**Note:** Upgrading from Office 365 E3 to Microsoft 365 E3 offers Windows device license per user and security licenses. Your license strategy needs to evaluate if you are purchasing per device licenses, or rely on all users interacting with devices having one of the M365 licenses.

#### Security and compliance requirements

Review this link showing all [security features](https://m365maps.com/files/Microsoft-365-Enterprise-All.htm). The main ones are Enterprise Mobility + Security E3 / E5. And the step up licenses in Microsoft 365 F5 / E5 Security and Microsoft 365 F5 / E5 Compliance.

#### Other premium tools

Microsoft is unfortunately no longer selling the Microsoft 365 E5 license as the one package which includes everything. Almost every workload within Microsoft 365 now has a premium license with extra features. If you need these extra features they should be negotiated as part of your enterprise agreement package.
Intune premium, SharePoint Syntex, Entra premium and the list goes on.

For a full overview of license packages I recommend getting familiar with [m365maps](https://m365maps.com/). It's the ultimate reference on Microsoft 365 licenses.

### Common license packages

You should define between two to four license profiles that covers your organization. Just remember not to confuse end users with a large selection of license profiles if you have enabled self-service. Part 3 of this series will cover how to introduce more hidden license profiles through automation.

- "Office" - Microsoft 365 E3 or E5 for office workers with their own Windows device
- "Light office" - Office 365 E1 + security license (EMS E3 or similar) for workers not requiring the office package and not requiring Windows OS license. For example consultants using their own company device, or Linux users.
- "Field" - Office 365 F3 + M365 F1 + Defender for office plan 1 for field workers only using mobile device
- "Unlicensed" - Microsoft 365 F1 for those who need MFA, signing in to enterprise applications. Important part is that this profile lacks a mailbox and OneDrive.

Honorable mention: The Microsoft 365 F3 license can given to field users, and allows some usage of shared windows desktop devices. The terms allow use of office apps on their primary device if the device has a screen size less than 10.9 inches. The terms and condition for this license is so wooly that I recommend discussing your use-case with your local Microsoft rep (I can almost guarantee they don't understand it either...)

### Putting it all together

> **Example:**
> Your organization consists of 10 000 employees.
> - 5000 office workers with their own laptop - "Office" profile
> - 3000 field workers using a mobile phone as their primary device - "Field" profile
> - 2000 factory workers using a mobile phone as their primary device, but also having access to shared devices running Windows. - "Field" profile using Microsoft 365 F3
> 
> Depending on the maturity of your organization, you could cover the license need in several ways.
> 
> **Easy:** One license profile for office and factory workers, one for field workers.
> **Medium:**  One license profile for office workers, one for factory workers, one for field workers.  
> **Advanced:** Max four / five license profiles, but automatically moving employees between them based on role, usage, and manual license requests.

## Group based licensing

Direct license assignments leads to users having different features and license mix, even though they should have the same license profile. It also enables you to turn on or off features for large amounts of users with a few clicks. If you haven't done so already, just make the transition. Hundreds of blog articles has already been written about this, so I'll just share the [Microsoft doc](https://learn.microsoft.com/en-us/entra/identity/users/licensing-groups-assign).

## Connect your license profiles to your HR / IAM / ITSM tool

When a new user joins a company, the license profile should automatically be given based on set critereas. These are company specific, and can be tied to roles and responsibilities, country and similar. At the very least you should ensure that license assignment is assigned automatically based on device ownership.

Once the automatic part is fixed, you need a way for users and managers to apply for license changes (usually upgrades). Problem is, noone has ever applied for a license downgrade. So if we do nothing our license mix will only ever get more expensive. Follow along for part 2, where we dive into the usage logs available to us to see how we can downgra... optimize our license mix!