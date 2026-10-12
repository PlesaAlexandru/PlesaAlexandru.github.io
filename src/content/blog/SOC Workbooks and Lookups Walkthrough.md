---
title: SOC Workbooks and Lookups Walkthrough
description: This room from TryHackMe describes how exactly a SOC team operates, the workflow and other practices. What "arsenal" do they have and so on.
pubDate: 2026-10-11
categories:
  - SOC
  - Walkthrough
  - TryHackMe
tags:
  - TryHackMe
  - Walkthrough
  - Rooms
---
---

## Assets & Identities

We are given an scenario about a file named: "Financial Report US 2024.xlsx" sent by G.Baker, logged into HQ-FINFS-02 server to R.Lund. To correctly triage it, you will need to answer to the following:   
- Who is G.Baker? What are their working hours and role in the company?
- What is the purpose and location of HQ-FINFS-02? Who can access it?
- Why could R.Lund need access to the corporate financial records?  

## Identity Inventory
![](../../assets/Pasted%20image%2020261012034927.png)

Identity inventory is a catalogue of corporate employees (user accounts), services (machine accounts), and their details like privileges, contacts, and roles within the company. For the scenario above, identity inventory would help you get context about G.Baker and R.Lund, and make it simpler to decide if the activity was expected or not.  

Exemple:  
![](../../assets/Pasted%20image%2020261012035232.png)

**Sources of Identities**

|Solution|Examples|Description|
|---|---|---|
|Active Directory|On-prem AD, Entra ID|AD itself is an identity database, and it is commonly used by SOC|
|SSO Providers|Okta, Google Workspace|Cloud alternative for AD, an easy way to manage and search the users|
|HR Systems|BambooHR, SAP, HiBob|Limited to employees only, but usually provides full employee data|
|Custom Solution|CSV or Excel Sheets|It is common for IT or security teams to maintain their own solutions|

## Asset Inventory

![](../../assets/Pasted%20image%2020261012035737.png)

Assets can be hardware, software, employees and others. It can be also names asset lookup. For the scenario above, asset inventory would help you get context about the HQ-FINFS-02 server.

![](../../assets/Pasted%20image%2020261012040106.png)

**Sources of Assets**

| Solution         | Examples             | Description                                                           |
| ---------------- | -------------------- | --------------------------------------------------------------------- |
| Active Directory | On-prem AD, Entra ID | AD is not only an identity but also a solid asset inventory database  |
| SIEM or EDR      | Elastic, CrowdStrike | Some SIEM or EDR agents collect information about the monitored hosts |
| MDM Solution     | MS Intune, Jamf MDM  | A dedicated class of solutions created to list and manage assets      |
| Custom Solution  | CSV or Excel Sheets  | Same as with the identity inventory, custom solutions are common      |

## QUESTIONS:

**Looking at the identity inventory, what is the role of R.Lund at the company?**
![](../../assets/Pasted%20image%2020261012040324.png)
Answer: US Financial Adviser

**Checking the asset inventory, what data does the HQ-FINFS-02 server store?**
![](../../assets/Pasted%20image%2020261012040511.png)
Answer: Financial Records

**Finally, does the file sharing from the scenario look legitimate and expected? (Yea/Nay)**
Answer: Yea.  
The role, person and server matches the scenario, with this little information, we can decide as being legit. 

---

**Work In Progress**