---
title: Introduction to SOAR Walkthrough
description: Learn the concepts and methodology surrounding security orchestration, automation, and response.
pubDate: 2026-10-14
categories:
  - SOC
  - Walkthrough
  - TryHackMe
tags:
  - TryHackMe
  - Walkthrough
  - Rooms
image: /image/introd-to-soar.png
---
---
## Summary:

SOAR is a modern solution that orchestrate and automate the processes of an SOC Analyst, this solution is capable of helping in a lot of scenarios and reduce TTR and overall make the job way more efficient. It is not a substitute of the analysts role, it is more like an helper that is capable of implementation through playbooks and also improves visibility by combining the tools used in a single space.
In this walkthrough we will see the importance of such a tool and some Phishing and CVE Patching examples.

---

## Traditional SOC and Challenges

SOC Capabilities:
- **Monitoring and Detection**: Scanning and flagging suspicious activities, mainly done through the SIEM
- **Recovery and Remediation**: Isolating or shutting down infected endpoints, removing malware, and stopping malicious processes. Done with EDR, firewalls, IAM (Identity and Access Management).
- **Threat Intelligence**: Being ready for the latest threats and updating everytime
- **Communication**: One of the most important aspect, due to the need of addressing incidents or verifying activities.

Challenges Faced by SOCs
- **Alert Fatigue:** Due to high number of false positive alerts
- **Too many Disconnected Tools:** Have to go through firewalls, endpoints, logs separated from eachother
- **Manual Processes:** Investigations that are not documented are slowing down the time of the response.
- **Talent Shortage:** SOC teams find recruiting and expanding their talent pool difficult to address the growing security landscape and sophisticated threats.

![](../../assets/Pasted%20image%2020261015125109.png)


## What is SOAR?

Security Orchestration, Automation, and Response (SOAR) is unifying all the solutions into one. So SIEM, EDR, firewalls or other tools can be monitored and accessed through the SOAR. Therefore reducing the challenges of a SOC Analyst.

![](../../assets/Pasted%20image%2020261015130822.png)

The core strength of a SOAR tool comes from the following three main capabilities:
## 1. Orchestration

We are given an VPN brute force scenario to understand the need of orchestration.
1. **SIEM** to check if the user usually uses the subject IP for logging
2. **Threat Intelligence (TI) platforms** to verify the IP's reputation
3. **IAM tool** to disable the user if there was any successful attempt
4. **Ticketing system** to open and track the incident

This manual switching between different tools slows down the process. So SOAR gather all the tools in one spot and define the workflow through playbooks (playbook = predefined steps to tell SOAR how to investigate an alert)

Let's continue the VPN brute force scenario, we would have the following playbook:

1. Received alert from SIEM
2. Query SIEM to check if the User normally uses the IP
3. Check TI platforms for the IP's reputation 
4. Query SIEM for any successful logins 
5. Escalate to containment actions

For example, If the IP is matching and the tries are minimal, the playbook may stop the alert early.
## 2. Automation

Multiple tools and predefined actions can be automated for the process to be easier and more efficient for the SOC analysts. Following the VPN brute force example:

1. SOAR receives the alert from SIEM
2. It automatically queries the SIEM for the user's historical logins
3. It automatically verifies the IP's reputation through TI platforms
4. If the IP is malicious, it automatically disables the user from the IAM
5. Lastly, it automatically opens a ticket in the ticketing system with all the details to initiate an investigation

## 3. Response
SOAR gives the ability to take actions using different tools from one unified interface.


The Orchestration, Automation, and Response capabilities of SOAR solve the major challenges a SOC team faces. With SOAR, there is no more alert fatigue, most of the processes are automated, and all the different tools are connected for coordination.

![](../../assets/Pasted%20image%2020261015131537.png)

## Do We Still Need SOC Analysts?

SOC Analysts are still critical, even in that context, for more important and crucial alerts. Also, a big note is that they are doing the playbook, maybe the process will filter the non important alerts, but in the end, the more important ones will need human assistance.

## Building SOAR playbooks

Let's see an example where we build an phishing playbook and CVE patching.

## Phishing Playbook

Phishing attacks are still one of the most common attack form and SOC Analysts lose a lot of time investigating the attacks. We are given the following scenario:

"Imagine you are a SOC analyst experienced in dealing with phishing alerts. You are training a junior analyst on how to respond to a phishing alert. For this, you make a flow diagram for them to take help from while investigating such alerts. What would you like to add to that flow diagram to assist your junior analyst?  Starting from receiving the alert "Suspicious email received", you will probably tell them the next steps they should take. They should create a ticket and check if the email contains a URL or an attachment. If it does not, just notify the users, but if it does, what would be the next steps? The next steps would further depend on whether it contains a URL or an attachment. So, a lot of "if this happens, do this; else, do this" instructions. This is how a plabook is made. The flow diagram below shows a playbook of phishing, telling what to do at what step."
![](../../assets/Pasted%20image%2020261015132853.png)


## CVE Patching Playbook

A CVE (Common Vulnerabilities and Exposures) is a vulnerability that is disclosed publicly and SOC Analysts have to always be ready for the newer ones, to check if their network is affected by the newer CVEs and patch the problems.

How would this playbook look like? This playbook will analyse the CVE details, assess its risk threshold, create a patching ticket, and test the patch before being pushed to the production environment. The following is an example of a playbook made for CVE patching.
![](../../assets/Pasted%20image%2020261015133250.png)
As you can see in both the playbooks' flow diagrams, most of the steps are automated, but a SOC analyst can be seen at a few points.

## Scenario

You are part of a SOC team that recently faced a large breach investigation that dragged on for ages due to a lack of automation. Your friend, McSkidy, recently advised adopting a SOAR and setting up automation workflows (also called playbooks) to help your security investigations. McSkidy sent you a checklist for a Threat Intelligence integration workflow, and your task is to figure out how it works.

In this scenario, your job is to determine and set which parts of the workflow should be `manual` or `automated` by clicking on the screens and toggling the switches. Click on the `RUN` command on the keyboard to check if your answers are correct.

![](../../assets/Pasted%20image%2020261015134404.png)
![](../../assets/Pasted%20image%2020261015134414.png)
![](../../assets/Pasted%20image%2020261015134423.png)
![](../../assets/Pasted%20image%2020261015134436.png)
![](../../assets/Pasted%20image%2020261015134502.png)


<details>
<summary>Click to view TryHackMe Room Q&A</summary>

* **How would you describe the experience of an overload of security events being triggered within a SOC?**  
  *Answer:* Alert Fatigue

* **The act of connecting and integrating security tools and systems into seamless workflows is known as?**  
  *Answer:* Orchestration

* **What do we call a predefined list of actions to handle an incident?**  
  *Answer:* Playbook

* **Is manual analysis vital within a SOAR workflow? Yay or Nay?**  
  *Answer:* Yay (human intervention still needed)

* **From where is the CVE Patching playbook fetching the new CVEs?**  
  *Answer:* Advisory Lists (seen in the diagram)

* **In the CVE Patching playbook, if the assets are found vulnerable even after the patch is deployed, what does the SOC develop next?**  
  *Answer:* Mitigation plan

* **What is the flag received?**  
  *Answer:* `THM{AUT0M@T1N6_S3CUR1T}¥`

</details>
