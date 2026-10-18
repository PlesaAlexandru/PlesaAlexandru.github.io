---
title: Unified Kill Chain Walkthrough
description: The Unified Kill Chain is a framework which establishes the phases of an attack, and a means of identifying and mitigating risk to IT assets.
pubDate: 2026-10-17
categories:
  - SOC
  - Walkthrough
  - TryHackMe
tags:
  - TryHackMe
  - Walkthrough
  - Rooms
image: /image/chainunif.png
---
---
## Summary

The room provided a modern framework based on the kill chain concept, called Unified Kill Chain, and explained each step with examples to better put into perspective the view of an attacker on all 18 steps. Also, the questions were more adaptable this time with different scenario to make use of the concepts below, I would love this type of questions to appear more in theoretical rooms at least.
Fundamental room for everybody who wants to understand the logic behind the attack.

---

**Learning Objectives:**

- Understanding why frameworks such as the UKC are important and helpful in establishing a good cybersecurity posture
- Using the UKC to understand an attacker's motivation, methodologies and tactics
- Understanding the various phases of the UKC
- Discover that the UKC is a framework that is used to complement other frameworks such as MITRE.

## What is a "Kill Chain"

Originating from the military, a “Kill Chain” is a term used to explain the various stages of an attack.
## What is "Threat modelling"

Threat modelling, is a series of steps to improve security of a system. It is about identifying risks, such as:

- Knowing what the system is intended to do, what sensitive data is caring, so basically, understanding of the system is a big priority.
- What is the system vulnerable at. (Weaknesses and vulnerability)
- Creating a plan to patch the vulnerabilities highlighted.
- Putting in policies to prevent these vulnerabilities from occurring again

![](../../assets/Pasted%20image%2020261016201944.png)


It is meant to be create a high level big picture of the assets and procedures to resolve vulnerabilities.
UKC framework helps identify potential attack surfaces and how these systems may be exploited.
STRIDE, DREAD and CVSS (to name a few) are all frameworks specifically used in threat modelling.
## Introducing the Unified Kill Chain

![](../../assets/Pasted%20image%2020261016203631.png)

The UTK (Unified Kill Chain) is reffering at 18 stages of the attack, from recon until objectives. They are explained below:
![](../../assets/Pasted%20image%2020261016203657.png)

| **Benefits of the Unified Kill Chain (UKC) Framework**                                                                                                                                                | **How do Other Frameworks Compare?**                                                                                              |
| ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| Modern (released in 2017, updated in 2022).                                                                                                                                                           | Some frameworks, such as MITRE’s were released in 2013, when the cybersecurity landscape was very different.                      |
| The UKC is extremely detailed (18 phases).                                                                                                                                                            | Other frameworks often have a small handful of phases.                                                                            |
| The UKC covers an entire attack - from reconnaissance, exploitation, post-exploitation and includes identifying an attacker's motivation.                                                             | Other frameworks cover a limited amount of phases.                                                                                |
| The UKC highlights a much more realistic attack scenario. Various stages will often re-occur. For example, after exploiting a machine, an attacker will begin reconnaissance to pivot another system. | Other frameworks do not account for the fact that an attacker will go back and forth between the various phases during an attack. |



## Goal: In (Initial Foothold)
![](../../assets/Pasted%20image%2020261016205303.png)

Main Goal: **to get access to the system or network environment.**
Recon is one of the tactics used to gather information, for example, on a system/target of the attacker.
Also in this stage, it is a common practice for the bad actor to establish an persistence to access the machine at any given time. 
![](../../assets/Pasted%20image%2020261016210039.png)
**Reconnaissance (MITRE Tactic TA0043)**

Recon at this stage, could be passive or active.
Information gathered from this phase can include:
- Discovering what systems and services are running on the target, this is beneficial information in the weaponisation and exploitation phases of this section.
- Finding contact lists or lists of employees that can be impersonated or used in either a social engineering or phishing attack.
- Looking for potential credentials that may be of use in later stages,  such as pivoting or initial access. 
- Understanding the network topology and other networked systems can be used to pivot too.

**Weaponization (MITRE Tactic TA0001)**

This phase of the UKC describes the adversary setting up the necessary infrastructure to perform the attack. For example, this could be setting up a command and control server, or a system capable of catching reverse shells and delivering payloads to the system.

**Social Engineering (MITRE Tactic TA0001)**

The tactic in this stage is describing the technique used to manipulate employees to perform actions that will aid in the adversaries attack
- Getting a user to open a malicious attachment in a phishing email.
- Impersonating a web page and having the user enter their credentials.
- Calling or visiting the target and impersonating a user (for example, requesting a password reset) or being able to gain access to areas of a site that the attacker would not previously be capable of (for example, impersonating a utility engineer).

**Exploitation (MITRE Tactic TA0002)**

This step is where the attacker takes advantage of the weaknesses or vulnerabilities.
- Uploading and executing a reverse shell to a web application.
- Interfering with an automated script on the system to execute code.
- Abusing a web application vulnerability to execute code on the system it is running on.

**Persistence (MITRE Tactic TA0003)**

This phase of the UKC describes the techniques an adversary uses to maintain access to a system they have gained an initial foothold on.
- Creating a service on the target system that will allow the attacker to regain access.
- Adding the target system to a Command & Control server where commands can be executed remotely at any time.
- Leaving other forms of backdoors that execute when a certain action occurs on the system (i.e. a reverse shell will execute when a system administrator logs in).

**Defence Evasion (MITRE Tactic TA0005)**

It is a really valuable step, where the adversary uses different techniques to evade the defensive solutions.
- Web application firewalls.
- Network firewalls.
- Anti-virus systems on the lab machine.
- Intrusion detection systems.
This phase is valuable when analysing an attack as it helps form a response and better yet - gives the defensive team information on how they can improve their defence systems in the future.

**Command & Control (MITRE Tactic TA0011)**

An adversary can establish command and control of a target system to achieve its action on objectives. For example, the adversary can:
- Execute commands.
- Steal data, credentials and other information.
- Use the controlled server to pivot to other systems on the network.


**Pivoting (MITRE Tactic TA0008)**

"Pivoting" is the technique an adversary uses to reach other systems within a network that are not otherwise accessible (for example, they are not exposed to the internet). There are often many systems in a network that are not directly reachable and often contain valuable data or have weaker security.
For example, an adversary can gain access to a web server that is publically accessible to attack other systems that are within the same network (but are not accessible via the internet).



## Goal: Through (Network Propagation)
![](../../assets/Pasted%20image%2020261016213539.png)

The "**Through**" goal follows a successful foothold being established on the target network. If defensive controls prevent the attacker from achieving their objective, they will seek to gain additional access and privileges to systems. The attacker would set up a base on one of the systems to act as their pivot point and use it to gather information about the internal network. The diagram below represents the individual attack phases of the "**Through**":
![](../../assets/Pasted%20image%2020261016213635.png)

**Pivoting** (MITRE Tactic TA0008)

Tunnel between their command operations and the victim’s network.

**Discovery** (MITRE Tactic TA0007)

The adversary would uncover information about the system and the network it is connected to.

**Privilege Escalation** (MITRE Tactic TA0004)

Following their knowledge-gathering, the adversary would try to gain more prominent permissions within the pivot system. They would leverage the information on the accounts present with vulnerabilities and misconfigurations found to elevate their access to one of the following superior levels:
- SYSTEM/ ROOT
- Local Administrator
- A user account with Admin-like access
- A user account with specific access or functions

**Execution** (MITRE Tactic TA0002)

This is where they deploy their malicious code using the pivot system as their host.

**Credential Access** (MITRE Tactic TA0006)

Working hand in hand with the Privilege Escalation stage, the adversary would attempt to steal account names and passwords through various methods, including keylogging and credential dumping. This makes them harder to detect during their attack as they would be using legitimate credentials.

**Lateral Movement** (MITRE Tactic TA0008)

With the credentials and elevated privileges, the adversary would seek to move through the network and jump onto other targeted systems to achieve their primary objective. The stealthier the technique used, the better.

## Goal: Out (Action on Objectives)

![](../../assets/Pasted%20image%2020261016230918.png)

 These goals are usually geared toward compromising the confidentiality, integrity and availability (CIA) triad. The tactics to be deployed by an attacker would include:

![](../../assets/Pasted%20image%2020261016230934.png)


**Collection** (MITRE Tactic TA0009)

This, in turn, compromises the confidentiality of the data and would lead to the next attack stage, Exfiltration. The main target sources include drives, browsers, audio, video and email.

**Exfiltration** (MITRE Tactic TA0010)

The adversary would seek to steal data, which would be packaged using encryption measures and compression to avoid any detection. The C2 channel and tunnel deployed in the earlier phases will come in handy during this process.

**Impact** (MITRE Tactic TA0040)

If the adversary seeks to compromise the integrity and availability of the data assets, they would manipulate, interrupt or destroy these assets. The goal would be to disrupt business and operational processes and may involve removing account access, disk wipes, and data encryption such as ransomware, defacement and denial of service (DoS) attacks.

#### **Objectives**

With all the power and access to the systems and network, the adversary would seek to achieve their strategic goal for the attack. For example, if the attack was financially motivated, they may seek to encrypt files and systems with ransomware and ask for payment to release the data. In other instances, the attacker may seek to damage the reputation of the business, and they would release private and confidential information to the public.
## Practical

![](../../assets/Pasted%20image%2020261016234408.png)
Gather information means Reconnaissance. Which is the first stage of any attacker.
![](../../assets/Pasted%20image%2020261016234913.png)
Persistence, because the script allows remote access at a later date.

![](../../assets/Pasted%20image%2020261016235000.png)
Command and Control, as mentioned in the article.

![](../../assets/Pasted%20image%2020261016235030.png)
Pivoting is the right answer in this case.
![](../../assets/Pasted%20image%2020261016235222.png)
Action and Objectives, based on the theory provided by Tryhackme. Objectives tab says (if the attack was financially motivated, they may seek to encrypt files and systems with ransomware and ask for payment to release the data) but in that case, somebody buys the data provided.
## Conclusion 
As mentioned in this room, the UKC is a modern extension of other frameworks, such as Lockheed Martin's "Cyber Kill Chain" framework.

## Q&A
<details>
<summary>Click to view TryHackMe Room Q&A</summary>

* **Where does the term "Kill Chain" originate from?**  
  *Answer:* Military   

* **What is the technical term for a piece of software or hardware in IT (Information Technology?)**  
  *Answer:* Asset

* **In what year was the Unified Kill Chain framework released?**  
  *Answer:* 2017

* **According to the Unified Kill Chain, how many phases are there to an attack?**  
  *Answer:* 18

* **What is the name of the attack phase where an attacker employs techniques to evade detection?**  
  *Answer:* Defense Evasion

* **What is the name of the attack phase where an attacker employs techniques to remove data from a network?**  
  *Answer:* Exfiltration

* **What is the name of the attack phase where an attacker achieves their objectives?**  
  *Answer:* Objectives

* **What is an example of a tactic to gain a foothold using emails?**  
  *Answer:* Phishing

* **Impersonating an employee to request a password reset is a form of what?**  
  *Answer:* Social Engineering

* **An adversary setting up the Command and Control server is what phase of the Unified Kill Chain?**  
  *Answer:* Weaponization

* **Exploiting a vulnerability present on a system is what phase of the Unified Kill Chain?**  
  *Answer:* Exploitation

* **Moving from one system to another is an example of?**  
  *Answer:* Pivoting

* **Leaving behind a malicious service that allows the adversary to log back into the target is what?**  
  *Answer:* Persistence

* **As a SOC analyst, you pick up an alert pointing to failed logins from an administrator account.  
What phase of the Unified Kill Chain would an attacker be seeking to achieve?**  
  *Answer:* Privilege Escalation

* **Mimikatz, a known post-exploitation tool, was detected on the IT Manager's workstation.  
The Security logs show that the tool was attempting to dump OS and user secrets.  
Which Unified Kill Chain phase does this activity correspond to?**  
  *Answer:* Credential Access

* **While monitoring the network as a SOC analyst, you observe a big traffic spike.  
Most of the network traffic is sent to an unknown, suspicious IP address.  
What Unified Kill Chain phase could describe this activity?**  
  *Answer:* Exfiltration

* **Personally identifiable information (PII) has been released to the public by an adversary.  
Your organisation is facing reputational losses and scrutiny for the breach.  
What part of the CIA triad would be affected by this action?**  
  *Answer:* Confidentiality

* **Match the scenario prompt to the correct phase of the Unified Kill Chain to reveal the flag at the end. What is the flag?**  
  *Answer:* `THM{UKC_SCENARIO}`

</details>
