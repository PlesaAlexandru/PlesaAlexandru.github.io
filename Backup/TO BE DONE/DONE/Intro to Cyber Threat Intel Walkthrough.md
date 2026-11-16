---
title: Intro to Cyber Threat Intel Walkthrough
description: Learn the foundations of Cyber Threat Intelligence (CTI), the intelligence lifecycle, and threat profiling.
pubDate: 2026-11-03
categories:
  - SOC
  - Walkthrough
  - TryHackMe
  - Network
tags:
  - Lessons
image: /image/networksecess.png
---
---
## Summary

In this room, we explore the foundations of **Cyber Threat Intelligence (CTI)**, including the four threat intelligence classifications (Strategic, Tactical, Operational, and Technical Intel), the six phases of the Threat Intelligence Lifecycle (Direction, Collection, Processing, Analysis, Dissemination, and Feedback), and industry sharing standards (`STIX` and `TAXII`). At the end, we use a simulated SIEM dashboard to correlate email, endpoint, and network alerts and build a complete threat profile.

---

**Learning Objectives**

- Understand the foundations of Cyber Threat Intelligence (CTI) and its classifications
- Learn the phases of the Threat Intelligence Lifecycle
- Explore standards and frameworks used to distribute and analyze threat intelligence

---
## Task 1.1
*What does CTI stand for?*

**Answer**: `Cyber Threat Intelligence`

**Why?**
As shown in the glossary popup, **CTI** stands for **Cyber Threat Intelligence**, which is evidence-based knowledge about adversaries, including their indicators, tactics, motivations, and actionable advice against them.

![](../../assets/Pasted%20image%2020261111160107.png)

## Task 1.2
*IP addresses, hashes and other threat artefacts would be found under which Threat Intelligence classification?*

**Answer**: `Technical Intel`

**Why?**
In the **Threat Intelligence Classifications** breakdown, atomic indicators and artifacts related to an attack, such as malicious IP addresses and file hashes, fall under **Technical intel**.

![](../../assets/Pasted%20image%2020261111171730.png)

## Task 2.1
*At which phase of the CTI lifecycle is data converted into usable formats through sorting, organising, correlation and presentation?*

**Answer**: `Processing`

**Why?**
In **Step 3: Processing - Normalising and Correlating the Data**, raw threat feeds from different sources are normalized into a consistent format, deduplicated, correlated, tagged with TLP labels, and converted into actionable files like firewall blocklists or YARA rules.

![](../../assets/Pasted%20image%2020261111201342.png)

## Task 2.2
*During which phase do security analysts get the chance to define the questions to investigate incidents?*

**Answer**: `Direction`

**Why?**
In **Step 1: Direction - Defining the Mission**, the CTI team defines the primary assets, business risks, available controls, and measurable intelligence questions (`Q1`, `Q2`) that will steer subsequent data collection and analysis.

![](../../assets/Pasted%20image%2020261111201448.png)

## Task 3.1
*What sharing models are supported by TAXII?*

**Answer**: `Collection and Channel`

**Why?**
In the **Sharing and Processing Intel** section, `TAXII` (Trusted Automated eXchange of Indicator Information) supports two sharing models: **Collection** (where threat intel is collected and hosted by a producer for pull requests) and **Channel** (where threat intel is published to subscribers from a central server).

![](../../assets/Pasted%20image%2020261111210758.png)

## Task 3.2
*When an adversary has obtained access to a network and is extracting data, what phase of the kill chain are they on?*

**Answer**: `Actions on Objectives`

**Why?**
Looking at the Cyber Kill Chain table, fulfilling the intended goals of the attack, such as financial gain, corporate espionage, ransomware encryption, and data exfiltration, takes place during the final **Actions on Objectives** phase.

![](../../assets/Pasted%20image%2020261111210904.png)

## Task 4.1
*What was the source email address?*

**Answer**: `vipivillain@badbank.com`

**Why?**
Opening the SIEM dashboard in **The Scenario** (`https://siem.internal`) and reviewing the **Alerts** table at `Jun 12th 2025 at 08:40`, we see the event `Email received by John Doe from vipivillain@badbank.com`.

![](../../assets/Pasted%20image%2020261111211003.png)

![](../../assets/Pasted%20image%2020261111210956.png)

## Task 4.2
*What was the name of the file downloaded?*

**Answer**: `flbpfuh.exe`

**Why?**
One minute after the phishing email arrived (`Jun 12th 2025 at 08:41`), the SIEM recorded `File download initiated by John Doe. File name: flbpfuh.exe`.

![](../../assets/Pasted%20image%2020261111211019.png)

## Task 4.3
*After building the threat profile, what message do you receive?*

**Answer**: `THM{NOW_I_CAN_CTI}`

**Why?**
Filling in each field of the threat intelligence flowchart at the bottom of the scenario based on the SIEM alerts:
- **Threat Actor Extraction IP Address?**: `91.185.23.222`
- **Threat Actor Email Address?**: `vipivillain@badbank.com`
- **Malware Tool?**: `flbpfuh.exe`
- **User Victim Logged Account?**: `Administrator`
- **Victim Email Recipient?**: `John Doe`

Once all 5 answers are submitted, the flowchart turns green and reveals the flag `THM{NOW_I_CAN_CTI}`.

![](../../assets/Pasted%20image%2020261111211637.png)
![](../../assets/Pasted%20image%2020261111211648.png)
