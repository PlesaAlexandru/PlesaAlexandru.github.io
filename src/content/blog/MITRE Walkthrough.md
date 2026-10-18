---
title: MITRE Walkthrough
description: In this room, we will focus on MITRE's cyber security frameworks and resources, including MITRE ATT&CK®, the CAR Knowledge Base, D3FEND, MITRE Engage™, and other relevant tools.
pubDate: 2026-10-16
categories:
  - SOC
  - Walkthrough
  - TryHackMe
tags:
  - TryHackMe
  - Walkthrough
  - Rooms
image: /image/mitre.png
---
---
## Summary

An introduction to MITRE and their frameworks, we used it to find different information about groups, attacks, defences. It was a really nice room that combined the theory from different MITRE frameworks with the task that asked you to provide valuable information, which is not only practical but also a step in getting used to these frameworks.
Overall, a foundational room with practical searching exercises in between lectures.

---

**Learning Objectives**
- Understand the purpose and structure of the MITRE ATT&CK® Framework
- Explore how security professionals apply ATT&CK in their work
- Use cyber threat intelligence (CTI) and the ATT&CK Matrix to profile threats
- Discover MITRE’s other frameworks, including CAR and D3FEND

## ATT&CK® Framework 

![](../../assets/Pasted%20image%2020261018143152.png)

The MITRE ATT&CK® framework is a knowledge base that stores tactics used in real world scenarios.
In 2013 MITRE recognized the need for such a database that stores the TTP of APTs and other malicious groups.
TTP stands for:
- Tactic: An adversary's goal or objective. The “why” of an attack.
- Technique: How an adversary achieves their goal or objective.
- Procedure: The implementation or how the technique is executed.

## ATT&CK Matrix
![](../../assets/Pasted%20image%2020261018141302.png)
1. **Tactic**: Let's say that an attacker wants to perform Reconnaissance on their target. This is the attacker's goal.
2. **Technique**: They may utilize the Active Scanning technique. This is how they achieve their Reconnaissance goal.
3. **Sub-technique**: Active Scanning comprises three specific methods: Scanning IP Blocks, Vulnerability Scanning, or Wordlist Scanning.

**Diving Deeper**

Let's take a closer look at the Active Scanning technique.

![](../../assets/Pasted%20image%2020261018141315.png)

Alongside the information discussed above, each tactic and technique page includes procedure examples (groups, software used, and campaigns), mitigations, detections, and references.

## ATT&CK in Operation 
ATT&CK provides cyber security professionals and organizations with a standard and consistent language for describing adversary behaviour.

**Threat Intelligence and Defence**
Beyond improving communication, ATT&CK also helps bridge the gap between threat intelligence and defensive operations. A threat report might describe what an attacker did, but not how to turn that information into usable detection measures. Through mapping threat activity to TTPs, defenders can translate intelligence into real detection logic, queries, and playbooks.

**Who Uses ATT&CK?**

| Who                                   | Their Goal                                                                           | How They Use ATT&CK                                                                                          |
| ------------------------------------- | ------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------ |
| Cyber Threat Intelligence (CTI) Teams | Collect and analyze threat information to improve an organization's security posture | Map observed threat actor behavior to ATT&CK TTPs to create profiles that are actionable across the industry |
| SOC Analysts                          | Investigate and triage security alerts                                               | Link activity to tactics and techniques to provide detailed context for alerts and prioritize incidents      |
| Detection Engineers                   | Design and improve detection systems                                                 | Map SIEM/EDR and other rules to ATT&CK to ensure better detection efforts                                    |
| Incident Responders                   | Respond to and investigate security incidents                                        | Map incident timelines to MITRE tactics and techniques to better visualize the attack.                       |
| Red & Purple Teams                    | Emulate adversary behavior to test and improve defenses                              | Build emulation plans and exercises aligned with techniques and known group operations                       |
## Mapping in Action

The group Mustang Panda (G0129) has been mapped to a variety of ATT&CK techniques based on its years of attacking government entities, non-profits, and NGOs. Below, you can see that Mustang Panda prefers phishing techniques for initial access, persists via scheduled tasks, obfuscates files to evade defenses, and uses an ingress tool transfer for command and control

![](../../assets/Pasted%20image%2020261018151555.png)
## ATT&CK for Threat Intelligence 

In the previous task, we explored how organizations and teams utilize Cyber Threat Intelligence (CTI) to understand adversary behavior and inform defense strategies by mapping intelligence to the ATT&CK framework. You also gained some experience using the Navigator to analyze a group's TTPs. Now, you'll use your newly gained knowledge to research threat groups that might target your organization.
![](../../assets/Pasted%20image%2020261018155331.png)
## Cyber Analytics Repository (CAR) 

CAR = a knowledge base of analytics developed by MITRE based on the MITRE ATT&CK adversary model. CAR defines a data model that is leveraged in its pseudocode representations, but also includes implementations directly targeted at specific tools (e.g., Splunk, EQL) in its analytics. With respect to coverage, CAR is focused on providing a set of validated and well-explained analytics, in particular with regard to their operating theory and rationale.

It is created for you, as a defender, to translate ATT&CK TTPs into real detections. We are given the following example:
![](../../assets/Pasted%20image%2020261018165738.png)
In the **Implementations** section, you are provided with Pseudocode, a Splunk query, and a LogPoint search as examples of how an analyst may filter for this technique using their organization's Security Information and Event Management (SIEM) platform. Pseudocode is a plain, human-readable way to describe a set of instructions or algorithms that a program or system will perform. Note that not every analytic within CAR will have the same implementation examples. Some analytics even include Unit Tests, which an analyst can use to validate whether the analytic works as intended.
![](../../assets/Pasted%20image%2020261018170025.png)


## MITRE D3FEND Framework 

With MITRE ATT&CK, you learn how attacks happen, but with MITRE D3FEND, you discover how to stop them.
**D3FEND** (Detection, Denial, and Disruption Framework Empowering Network Defense) is a structured framework that maps out defensive techniques

![](../../assets/Pasted%20image%2020261018170559.png)

For example, the Credential Rotation D3-CRO technique emphasizes the regular rotation of passwords to prevent attackers from reusing stolen credentials. D3FEND explains how this defense works,
![](../../assets/Pasted%20image%2020261018170727.png)

## Other MITRE Projects 

Here, we just find more MITRE Projects such as: 
![](../../assets/Pasted%20image%2020261018171728.png)
![](../../assets/Pasted%20image%2020261018171704.png)
![](../../assets/Pasted%20image%2020261018171711.png)

## Working with MITRE (Practical exercises)

On this lesson, we are tasked to browse through MITRE, and find the following:

**What Tactic does the Phishing technique belong to in the ATT&CK Matrix?**

To do that, we first need to go to the matrix and search for the Phishing with CTRL+F or find tool.
![](../../assets/Pasted%20image%2020261018144354.png)
Phishing has multiple entries, but in the end we can speculate that it is referring to phishing as an **Initial Access** method. 


After that we are tasked to search for the Create Account Technique ID, for that we can use the search bar top right.
![](../../assets/Pasted%20image%2020261018145617.png)
The ID is shown on the right side. For the sake of learning we can also see that "Create Account" is used as a Persistence method, which is part of the previous methods explained in articles such as: Cyber Kill Chain or Unified Kill Chain (UKC).
![](../../assets/Pasted%20image%2020261018145607.png)
The two requirements were coming from the first Task of the room.

For the second task, we have to look for: **In which country is Mustang Panda based?**
For that, we can search for the group page in MITRE Matrix
![](../../assets/Pasted%20image%2020261018151936.png)
In the first paragraph, we can see that is a China based group.

**Which ATT&CK technique ID maps to Mustang Panda’s Reconnaissance tactics?**

We can use the same group page, but also, we have to options, either search manually for the IDs or use the Navigator. I would choose the Navigator option
Scroll down till we reach Techniques Used, after we can select ATTCK Navigator and press view
![](../../assets/Pasted%20image%2020261018152308.png)

![](../../assets/Pasted%20image%2020261018152500.png)

We are shown the matrix, in which case we can see 2 blue zones highlighted in the recon category.
One is Search Open Websites/Domains (T1593) and the other is Spearphishing Link (T1598.003).
are tasked to write only one method. First we can try T1593 but it is the wrong answer, although, T1598.003 is a sub-technique of the T1598 method (Phishing for information).
Every technique have sub-techniques or different variants. With that being said, T1598 is the answer.

The third question is: **Which software is Mustang Panda known to use for Access Token Manipulation?**
![](../../assets/Pasted%20image%2020261018154411.png)
We can just CTRL+F on the group page to search for Access Token Manipulation to find the answer.

---
We are given the following scenario:
You are a security analyst in the aviation sector, and your organization is migrating its infrastructure to the cloud. Your task is to use ATT&CK to gather intelligence on APT groups known to target this sector, identify their tactics and techniques, and assess any potential gaps in your defensive coverage.

Let's go for it, first, they are tasking us to: 
**Which APT group has targeted the aviation sector and has been active since at least 2013?**

So, first of all, we are going to use the groups part of MITRE, we can search either for 2013 groups or aviation. Aviation returns 7 results and 2013 returns 13 results. Let's see what aviation brings us up to:

![](../../assets/Pasted%20image%2020261018160014.png)
![](../../assets/Pasted%20image%2020261018160021.png)
![](../../assets/Pasted%20image%2020261018160029.png)
and many more, but honestly, we can easily spot the fact that APT33 checks everybox.

**Which ATT&CK sub-technique used by this group is a key area of concern for companies using Office 365?**

Searching for the keyword "Office 365", 1 group shows up. When looking at the group, we can see the techniques used by Scattered Spider![](../../assets/Pasted%20image%2020261018160255.png)
Scrolling down we can see that Cloud Account, is one of the first techniques 
![](../../assets/Pasted%20image%2020261018161719.png)

**According to ATT&CK, what tool is linked to the APT group and the sub-technique you identified?**

From ![](../../assets/Pasted%20image%2020261018163858.png)
We can scroll down to find the answer:
![](../../assets/Pasted%20image%2020261018163924.png)
Answer would be Ruler.

**Which mitigation strategy advises removing inactive or unused accounts to reduce exposure to this sub-technique?**
For this, we are going to select Defenses -> Mitigations, here is the mitigation strategies. We can search again manually by looking for Users. Which returns 22 possible answers. But looking at the 22 returns, we can already spot one matching case

User Account Management
![](../../assets/Pasted%20image%2020261018164754.png)


**What Detection Strategy ID would you implement to detect abused or compromised cloud accounts?**
![](../../assets/Pasted%20image%2020261018165354.png)
Detection strategies -> Search for Cloud Accounts, you will find DET0546
![](../../assets/Pasted%20image%2020261018165326.png)


**Which ATT&CK Tactic is associated with CAR-2019-07-001?**
The link provided by TryHackMe, is getting us to the exact CAR. The highlighted part is what they are asking us to provide.
![](../../assets/Pasted%20image%2020261018170234.png)

**What is the Analytic Type for Access Permission Modification?

On the same page as before, Analytic type is shown in the right side of the CAR raport.
![](../../assets/Pasted%20image%2020261018170404.png)

**Which sub-technique of User Behavior Analysis would you use to analyze the geolocation data of user logon attempts?** 

From the link provided, we can scroll down to Technique Subclasses tab where we can find the answer.
![](../../assets/Pasted%20image%2020261018171104.png)

**Which digital artifact does this sub-technique rely on analyzing?**  
Clicking D3-UGLPA from that point will bring us to:
![](../../assets/Pasted%20image%2020261018171513.png)
We can scroll down to Artifact Relationships and note that Network Traffic is one of them.
![](../../assets/Pasted%20image%2020261018171451.png)

**What technique ID is associated with Scrape Blockchain Data in the AADAPT framework?**

Clicking the link for the Scrape Blockchain Data, we can see in the right side
![](../../assets/Pasted%20image%2020261018172048.png)

**Which tactic does LLM Prompt Obfuscation belong to in the ATLAS framework?**

Similar as before, get to the technique and on the right you have a little summary where you can see the Tactics.
![](../../assets/Pasted%20image%2020261018172101.png)

## Q&A
<details>
<summary>Click to view TryHackMe Room Q&A</summary>

* **What Tactic does the Phishing technique belong to in the ATTCK Matrix?**  
  *Answer:* Initial Access   

* **Which ID is associated with the Create Account technique?**  
  *Answer:* T1136

* **In which country is Mustang Panda based?**  
  *Answer:* China

* **Which ATTCK technique ID maps to Mustang Panda’s Reconnaissance tactics?**  
  *Answer:* T1598

* **Which software is Mustang Panda known to use for Access Token Manipulation?**  
  *Answer:* Cobalt Strike

* **Which APT group has targeted the aviation sector and has been active since at least 2013?**  
  *Answer:* APT33

* **Which ATTCK sub-technique used by this group is a key area of concern for companies using Office 365?**  
  *Answer:* Cloud Accounts

* **According to ATTCK, what tool is linked to the APT group and the sub-technique you identified?** 
  *Answer:* Ruler

* **Which mitigation strategy advises removing inactive or unused accounts to reduce exposure to this sub-technique?**  
  *Answer:* User Account Management

* **What Detection Strategy ID would you implement to detect abused or compromised cloud accounts?**  
  *Answer:* DET0546

* **Which ATTCK Tactic is associated with CAR-2019-07-001?**  
  *Answer:* Defense Evasion

* **What is the Analytic Type for Access Permission Modification?**  
  *Answer:* Situational Awareness

* **Which sub-technique of User Behavior Analysis would you use to analyze the geolocation data of user logon attempts?**  
  *Answer:* User Geolocation Logon Pattern Analysis

* **Which digital artifact does this sub-technique rely on analyzing?**  
  *Answer:* Network Traffic

* **What technique ID is associated with Scrape Blockchain Data in the AADAPT framework?**  
  *Answer:* ADT3025

* **Which tactic does LLM Prompt Obfuscation belong to in the ATLAS framework?**  
  *Answer:* Defense Evasion  

</details>
