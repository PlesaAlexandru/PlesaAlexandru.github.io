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

## Network Diagrams

We are given the following scenario  

Consider the scenario where you are investigating a chain of related alerts based on firewall logs and want to give some meaning to the IPs you see:  
1. **08:00**: An IP **103.61.240.174** is repeatedly connecting to a corporate firewall via port **TCP/10443**
2. **08:23**: Firewall logs show that the IP 103.61.240.174 was translated to an internal **10.10.0.53** IP
3. **08:25**: The IP 10.10.0.53 is scanning the **172.16.15.0/24** network range but does not find open ports
4. **08:32**: The same IP is now scanning the **172.16.23.0/24** network range, and the attack seems to be ongoing  

To investigate the case above, you will have to find out what service is running at the 10443 port and why anyone would connect there. Then, identify the subnet the 10.10.0.53 IP belongs to and why it would ever try to connect to other subnets. A **network diagram**, a visual schema presenting existing locations, subnets, and their connections, is an answer to your questions:  

![](../../assets/Pasted%20image%2020261012155125.png)

Depending on a company's size and structure, you may see more complex diagrams, but their use for SOC analysts remains the same - to help understand suspicious network activity. In our scenario, you can refer to the network diagram and reconstruct the attack path as follows:  

1. Threat actor behind the 103.61.240.174 IP performed **VPN brute force**, targeting vpn.tryhatme.thm
2. After a successful brute force and VPN login, the threat actor was assigned an IP from the **VPN Subnet**
3. Then, the adversary tried to scan the **Database Subnet**, but was likely blocked by the firewall rules
4. Seeing no success, the threat actor switches to the **Office Subnet**, looking for their next target  

## QUESTIONS:

**According to the network diagram, which service is exposed on the TCP/10443 port?**  
Answer: VPN, mentioned in the task.  

**Now, which subnet would the server behind 172.16.15.99 IP belong to?**  
![](../../assets/Pasted%20image%2020261012155428.png)  
172.16.15.99 is corresponding to database subnet.  
Answer: Database subnet

**Finally, does the scenario look like a True Positive (TP) or False Positive (FP)?**
Answer: TP, typical movement through the network to find the next step or target. 

## Workbooks Theory

With asset inventory and network diagrams, you can get enough context about the user, host, or IP address.  

## SOC Workbooks

A SOC workbook (or playbook/runbook) standardizes threat investigation and response. Designed by senior engineers to guide entry-level L1 analysts through complex scenarios, these step-by-step procedures minimize errors, streamline triage, and ensure consistent handling across all incoming alerts.

	## Workbook Example
**Unusual Login Location Workbook**  
![](../../assets/Pasted%20image%2020261012160138.png) 
The diagram above is a typical example of an investigation workbook aimed to help L1 analysts triage alerts about atypical email, web, or corporate VPN login. Most workbook diagrams are supplemented with a detailed textual guide and links to the mentioned resources. Also, note how the workbook is divided into three logical groups. By following the steps in the correct order, you can guarantee high-quality alert triage and eliminate cases where the verdict is made without enough evidence:  

1. **Enrichment**: Use Threat Intelligence and identity inventory to get information about the affected user
2. **Investigation**: Using the gathered data and SIEM logs, make your verdict if the login is expected
3. **Escalation**: Escalate the alert to L2 or communicate the login with the user if necessary  

## QUESTIONS:

**Which SOC role would use workbooks the most (e.g. SOC Manager)?**  
Answer: SOC L1 Analyst  

**What is the process of gathering user, host, or IP context using TI and lookups?**  
Answer: Enrichment

**Looking at the workbook example, what platform is used as an identity inventory source?**  
Answer: BambooHR

## Practice

Different teams have different approaches to workbook building. Some teams may have hundreds of complex workbooks for every possible detection rule, more like a SOAR automation playbook than human guides. Other teams may prepare just a few high-level workbooks for the most common attack vectors and rely more on the experience and decision-making of L1 analysts. In any case, you as an L1 analyst should know how to divide your investigation into modular blocks and build simple workbooks around it.

## QUESTIONS:

**What flag did you receive after completing the first workbook?**  
![](../../assets/Pasted%20image%2020261012161048.png)
Answer: THM{the_most_common_soc_workbook}  

**What flag did you receive after completing the second workbook?**  
![](../../assets/Pasted%20image%2020261012161318.png)
Answer: THM{be_vigilant_with_powershell}  

**What flag did you receive after completing the third workbook?**  
![](../../assets/Pasted%20image%2020261012161645.png)
Answer: THM{asset_inventory_is_essential}  