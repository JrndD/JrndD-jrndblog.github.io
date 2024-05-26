+++
title = 'Optimizing Microsoft 365 licensing, part 1'
date = 2024-05-25T21:02:16+03:00
draft = true
ShowToc = true
[params.cover]
  image = 'Microsoft365Licensing/money_flying.webp'
  alt = 'Money flying out the window'
  caption = 'Time to pay the yearly EA invoice.'
+++

It's that time of the year. You have one month before the yearly license numbers are due to be delivered to Microsoft. The questions have started pouring in. Can we make any savings? Are we utilizing all the features we have purchased? Do we have enough licenses for all our workloads? Are we even compliant?

You have already dedicated your time to other projects, and now this needs to be handled in the middle of all this. Where to begin?

Don't worry, take a deep breath. I understand licensing, so you don't have to.

This guide focuses on the things I consider to be important, managing licenses for large global enterprises over several years. I have saved several million euros on Microsoft licenses for some companies in my career.

I am planning to divide this guide into three parts:
- Part 1 - License governance
- Part 2 - Data driven decisions. How to accurately measure license usage.
- Part 3 - Automation.

## Get your license profiles in order

License profiles are mandatory if you want to free yourself from micromanagement. Simply explained, the number of license profiles you need are the least common divisor of different license packages to cover all your employees.

A license profile usually corresponds with one of the core Microsoft license packages, but can be mixed and matched to suite your needs.

### License profile considerations

The things to consider when choosing a license package for a license profile is:
- Tools the employee needs to do their job
    - Windows device license, Office package, email, OneDrive etc.
- Security tool requirements
    - MFA, Defender for Endpoint, Defender for Identity, Identity Protection etc.
- Compliance features in Purview
    - Retention, data loss prevention, legal holds
- Other administrative tools
    - Intune premium, Privileged Access Management etc.

### Common license packages

Although daunting at first sight, I will make it easier for you. Assuming your organization uses Entra ID for authentication.
1. All your employees need a license for **MFA and SSO** using enterprise applications. This license is called Entra ID plan 1, and is included in all Microsoft 365 packages, including the cheap Microsoft 365 F1 package.
2. 
3. Office workers signing in to Windows need a **Windows device license**. This can either be purchased per device, or per user through the Microsoft 365 E3 or E5 plan. Office workers signing in to Windows usually also needs the Office package (Word, Excel etc), which the E3 and E5 package contains. Win win.
4. Field workers will fill up their Office 365 F3 provided 2 GB Mailbox, so prepare to have a process for upgrading those that need a larger mailbox to **Exchange Online Plan 1**, instead of giving them the much more expensive E3 / E5 package.


And, bookmark this page: [m365maps](https://m365maps.com/). It's the ultimate reference on Microsoft 365 licenses.

> **Example:**
> Your organization consists of 10 000 employees.
> - 5000 office workers with their own laptop
> - 3000 field workers using a mobile phone as their primary device
> - 2000 factory workers using a mobile phone as their primary device, but also having access to shared devices running Windows.
> 
> Depending on the maturity of your organization, you could cover the license need in several ways. License packages 
> 
> **Easy:** One license profile for office and factory workers, one for field workers.
> **Medium:**  One license profile for office workers, one for factory workers, one for field workers.  
> **Advanced:** Max four / five license profiles, but automatically moving employees between them based on role, usage, and manual license requests.