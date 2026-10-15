---
title: Cyber Kill Chain Walkthrough
description: The Cyber Kill Chain framework is designed for identification and prevention of the network intrusions. You will learn what the adversaries need to do in order to achieve their goals.
pubDate: 2026-10-15
categories:
  - SOC
  - Walkthrough
  - TryHackMe
tags:
  - TryHackMe
  - Walkthrough
  - Rooms
image: /image/chain_thumb.png
---
---
## Summary

This room is based on the 2011 Cyber Kill Chain, which is a framework that help you understand and protect against ransomware attacks, security breaches as well as Advanced Persistent Threats (APTs). Although outdated, the foundation of it is still relevant for the following rooms. The room is built on an fictional attacker ("Megatron") and it's practices to better put into perspective the whole idea of a cyber kill chain.

---
## Reconnaissance
![](../../assets/Pasted%20image%2020261015172206.png)
Is the first phase and the goal is to gather as much intel as possible before the attack.
Poor recon typically leads to sloppy attacks while the more informed ones can be create highly believable payloads for example.
A valuable piece of recon is **OSINT** (Open-Source Intelligence). With OSINT, adversaries gather insights about their target through publicly available information. Some public sources where OSINT data can be collected from include:
- Search engines
- Print and online media
- Social media accounts
- Online forums and blogs
- Online public record databases
- WHOIS and technical data

**Reconnaissance Types**

- **Passive Recon:** No interaction with the target (WHOIS, social media scraping...)
- **Active Recon:** Active interaction (social engineering, port scanning...) 

We are given an scenario from the attacker's perspective, who initially doesn't know what company he wants to attack.
"A malicious attacker who names himself "Megatron" decides to conduct a very sophisticated attack that he has been planning out for years; he has been studying and researching different tools and techniques that could help him get to the last phase of the Cyber Kill Chain. But first, he needs to start from the Reconnaissance phase.
In order to operate in this phase, the attacker would need to conduct OSINT"

So we are going to take a look at Email Harvesting, which is a process of obtaining, through free or paid services, the email addresses of certain individuals, services and other entities. This could lead to a phishing attack for example, which is a type of social-engineering attack used to steal sensitive data, including login credentials and credit card numbers.

The hacker in this scenario could use a couple of tools for this recon mission, for example:
- theHarvester: other than gathering emails, this tool is also capable of gathering names, subdomains, IPs, and URLs using multiple public data sources.
- Hunter.io: this is an email hunting tool that will let you obtain contact information associated with the domain.
- OSINT Framework: OSINT Framework provides the collection of OSINT tools based on various categories.
## Weaponization

![](../../assets/Pasted%20image%2020261015172220.png)

At this stage, "Megatron" would work on turning the raw information into actionable attack tools through crafting **malware** and **exploits** into a **payload**.
Most attackers usually use automated tools to generate the malware or refer to the [DarkWeb](https://www.kaspersky.com/resource-center/threats/deep-web) to purchase the malware. More sophisticated actors or nation-sponsored APT (Advanced Persistent Threat Groups) would write their custom malware to make the malware sample unique and evade detection on the target.

Key Terminology:

**Malware** is a program or software that is designed to damage, disrupt, or gain unauthorized access to a computer.

**Exploits** are programs or code that take advantage of the vulnerability or flaw in the application or system.

**Payload** is a malicious code that the attacker runs on the system.

Let's continue with the example of "Megatron", he chooses to buy an already written payload from someone else in the DarkWeb, so that he can spend more time on the other phases.

In the Weaponization phase, the attacker can adopt the following tactics:
- Create an infected Microsoft Office document containing a malicious macros or VBA (Visual Basic for Applications) scripts.
- Create a malicious payload or a very sophisticated worm, implant it on USB drives, and then distribute them in public.
- Set up Command and Control (C2) infrastructure for executing the commands on the victim's machine or deliver more payloads.
- Infect the victim's host with a backdoor, which would provide a way to access the computer system, and bypass the security mechanisms.
- Tailoring phishing templates or OAuth-consent apps to look legitimate and dupe the victim.

## Delivery
![](../../assets/Pasted%20image%2020261015172227.png)
Delivery is when Megatron decides to choose the method for transmitting the payload or the malware onto the target environment.
He has a couple of options:
- Phishing email: Actor could craft a malicious email. The email would contain a malicious link or email attachment that would result into a compromise.
    
- USB drops offer the attacker a physical delivery medium into public places like coffee shops, car parks, or on the street. An attacker might decide to conduct a sophisticated USB Drop Attack by printing the company's logo on the USB drives and mailing them to the company while pretending to be a customer sending the USB devices as a gift.
    
- Watering hole attacks are targeted and designed to aim at a specific group of people by compromising the website they are usually visiting. An example can be a malicious pop-up asking to download a fake Browser extension.

## Exploitation
![](../../assets/Pasted%20image%2020261015172241.png)
Is the phase where the victim executes the code, taking advantage of an known vulnerability.
Here, the attacker have a couple of options:

- Malicious macro execution: This may have been delivered through a phishing email, that would execute ransomware when the victim opens it.
- Zero-day exploits: These leverages on unknown and unpatched flaws in a system. These exploits leave no opportunity for detection at the beginning.
- Known CVEs: The attacker can choose to exploit unpatched public vulnerabilities found on the target environment.

After gaining access, the actor can exploit the system by escalatating the privileges or moving lateral through the network.
Some signs for that would be:

- Unexpected process spawns.
- Registry changes or new services created.
- Suspicious command-line arguments found in system logs.

## Installation
![](../../assets/Pasted%20image%2020261015172250.png)

This is where the attacker is looking for persistence opportunities such as backdoors, which are in essence an access point.
Once the attacker gets access to the system, he would want to reconnect back to the system if he loses the connection to it or if he got detected and got the initial access removed. Or if the system is later patched, they will no longer have access to it. That is when the attacker needs to install a persistent backdoor. A persistent backdoor will let the attacker access the system he compromised in the past.
The persistence can be achieved through:

- Installing a **web shell** on the webserver.
- Installing a backdoor on the victim's machine. (meterpreter for example)
- Creating or modifying Windows services. (This technique is known as T1543.003)
- Adding the entry to the "run keys" for the malicious payload in the Registry or the Startup Folder.

In this phase, the attacker can also use the **Timestomping** to mimic some legitimate files create/modify date to avoid detection by the forensic investigator.


## Command and Control

![](../../assets/Pasted%20image%2020261015172301.png)

After getting persistence Megatron opens up the C2 (Command and Control) channel through the malware to remotely control, known as **C&C or C2 Beaconing**.

The compromised endpoint would communicate with an external server set up by an attacker to establish a command & control channel. Until recently, IRC (Internet Relay Chat) was the traditional C2 channel used by attackers. This is no longer the case, as modern security solutions can easily detect malicious IRC traffic.

The most common C2 channels used by adversaries include:
- HTTP on port 80 and HTTPS on port 443, where this type of beaconing blends the malicious traffic with the legitimate traffic and can help the attacker evade firewalls.
    
- DNS (Domain Name Server), where the infected machine makes constant DNS requests to the DNS server that belongs to an attacker, this type of C2 communication is also known as DNS Tunneling

Note: An adversary or another compromised host can be the owner of the C2 infrastructure.

## Actions on Objectives (Exfiltration)

![](../../assets/Pasted%20image%2020261015172311.png)

After all these steps, the attacker can achieve, based on the end goal, the following:
- Collect the credentials from users.
- Perform privilege escalation (gaining elevated access like domain administrator access from a workstation by exploiting the misconfiguration).
- Internal reconnaissance (for example, an attacker gets to interact with internal software to find its vulnerabilities).
- Lateral movement through the company's environment.
- Collect and exfiltrate sensitive data.
- Deleting the backups and shadow copies. Shadow Copy is a Microsoft technology that can create backup copies, snapshots of computer files, or volumes. 
- Overwrite or corrupt data.




## Practice Analysis

![](../../assets/Pasted%20image%2020261015172319.png)

Tryhackme gives us the following scenario for a practice exercise:

_The infamous Target cyber-attack, which led to one of the largest data breaches in history took place on November 27, 2013._

On December 19th, 2013, Target released a statement confirming the breach, stating that approximately 40 million credit and debit card accounts were impacted between Nov. 27 and Dec. 15, 2013. Target had to pay the fine of $18.5 million under the terms of the multistate settlement agreement. This is considered to be the largest data-breach settlement in history.

We are tasked with:

Add each of the 6 items on the list in the correct Kill Chain entry form on the **Static Site** Lab. Note that although the Cyber Kill Chain framework consists of 7 stages, only 6 are relevant to this specific scenario:

- exploit public-facing application
- data from local system
- powershell
- dynamic linker hijacking
- spearphishing attachment
- fallback channels

With that being said, the solution is: 
![](../../assets/Pasted%20image%2020261015203449.png)




## Conclusion

![](../../assets/Pasted%20image%2020261015172332.png)
Cyber Kill Chain can be a great tool to improve network defence. Is it perfect and can it be the only tool to rely on? No.

Since the main focus of the framework is on malware delivery and network security, the traditional Cyber Kill Chain will not be able to identify **Insider Threats**. According to CISA, _"The Insider Threat is the potential for an insider to use their authorized access or understanding of an organization to harm that organization."_

We recommend not only relying on the traditional Cyber Kill Chain model but also referring to MITRE ATT&CK as well as Unified Kill Chain to apply a more comprehensive approach to your defence methodologies.

## Q&A
<details>
<summary>Click to view TryHackMe Room Q&A</summary>

* **What is the name of the Intel Gathering Tool that is a web-based interface to the common tools and resources for open-source intelligence?**  
  *Answer:* OSINT Framework  

* **What is the definition for the email gathering process during the stage of reconnaissance?**  
  *Answer:* Email Harvesting

* **What is the term for automated scripts embedded in Microsoft Office documents that can be used to perform tasks or exploited by attackers for malicious purposes?**  
  *Answer:* Macro

* **What do you call an attack targeting a specific group by infecting their frequently visited website?**  
  *Answer:* Watering Hole Attack

* **What is the term for a cyber attack that exploits a software vulnerability that is unknown by software vendors?**  
  *Answer:* Zero-Day

* **What technique is used to modify file time attributes to hide new or changes to existing files?**  
  *Answer:* Timestomping

* **What malicious script can be planted by an attacker on the web server to maintain access to the compromised system and enables the web server to be accessed remotely?**  
  *Answer:* web shell

* **What is the C2 communication where the victim makes regular DNS requests to a DNS server and domain which belong to an attacker.**  
  *Answer:* DNS TUNNELING

* **What technology is included in Microsoft Windows that can create backup copies or snapshots of files or volumes on the computer, even when they are in use?**  
  *Answer:* shadow copy

* **What is the flag after you complete the static site?**  
  *Answer:* `THM{7HR347_1N73L_12_4w35om3}`

</details>
