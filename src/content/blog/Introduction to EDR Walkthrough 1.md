---
title: Introduction to EDR Walkthrough TEST
description: Learning the fundamentals of EDR and explore its features and working.
pubDate: 2026-10-13
categories:
  - SOC
  - Walkthrough
  - TryHackMe
tags:
  - TryHackMe
  - Walkthrough
  - Rooms
image: /image/introd-to-edr.png
---
---

## Executive Summary

The growing complexity of modern cyber threats and the rapid proliferation of digital endpoints demand deep, resilient endpoint protection solutions[cite: 1]. **Endpoint Detection and Response (EDR)** provides continuous monitoring, deep visibility, and proactive containment mechanisms regardless of the physical location of the endpoint within or outside the corporate perimeter[cite: 1].

While market solutions—such as CrowdStrike Falcon—vary in feature sets and dashboards, their underlying architecture and operational principles remain largely consistent[cite: 1]. This writeup explores the operational pillars of an EDR solution, analyzes an evasion attack chain comparing traditional AV to EDR, and performs a triage on three simulated endpoint detections[cite: 1].

---

## 1. The Three Pillars of EDR

A comprehensive EDR solution relies on three core capabilities[cite: 1]:

![](../../assets/Pasted%20image%2020261013154157.png)

### A. Visibility
Visibility is a top priority for any SOC Analyst[cite: 1]. EDR consoles expose full process trees alongside complete execution timelines, eliminating blind spots[cite: 1]:

| Process Modifications | Registry Modifications | File And Folder Modifications | User Actions | Advanced Telemetry |
| :--- | :--- | :--- | :--- | :--- |

![](../../assets/Pasted%20image%2020261013171039.png)

### B. Detection
Beyond traditional signature checks, EDR platforms integrate behavioral analytics and machine learning engines to baseline normal host activity and flag anomalies[cite: 1]. This design detects in-memory attacks and fileless malware, while still allowing teams to inject custom Threat Intelligence indicators (IOCs)[cite: 1].

Every alert provides granular metadata: severity, timestamp, triggering process, target host, user context, and mapping to the **MITRE ATT&CK** framework[cite: 1].

![](../../assets/Pasted%20image%2020261013172016.png)

### C. Response
When an alert is validated, the analyst can execute response measures directly from the centralized management interface: host network isolation, process termination, file quarantine, or initiating remote shell sessions for deeper inspection[cite: 1].

![](../../assets/Pasted%20image%2020261013173018.png)

---

## 2. Beyond Traditional Antivirus: Why EDR Is Essential

A common question in enterprise security is why an Antivirus (AV) solution alone is insufficient[cite: 1]. Both tools protect the host, but the depth of coverage and operational philosophy differ significantly[cite: 1].

> **The Airport Analogy:**  
> Think of an endpoint as an airport facility[cite: 1].  
> * **Antivirus (AV)** acts like the **Immigration Checkpoint**: it screens incoming travelers against a database of known criminals (signatures)[cite: 1]. If an offender appears in the database, entry is blocked[cite: 1]. However, if an individual with no prior criminal record enters with malicious intent, immigration lets them pass[cite: 1].  
> * **EDR** functions like the **Security Officers and CCTV inside the terminal**: it continuously observes post-entry behavior (e.g., unauthorized access attempts, abandoned bags, suspicious movements)[cite: 1].

**Analyst Note:** Deploying both layers provides depth; when a threat lacks a prior hash signature or leverages legitimate system utilities, EDR behavioral detection covers the visibility gap[cite: 1].

### Attack Scenario Breakdown

![](../../assets/Pasted%20image%2020261013175955.png)

| Attack Step | Antivirus (AV) Response | EDR Response |
| :--- | :--- | :--- |
| **Step #1:** Malicious macro-embedded document received via phishing[cite: 1] | Inactive if the file hash is absent from the signature database[cite: 1]. | Logs file creation and records associated hash telemetry[cite: 1]. |
| **Step #2:** User executes document[cite: 1] | Inactive; `winword.exe` is a legitimate executable[cite: 1]. | Logs the process start of `winword.exe` and monitors children[cite: 1]. |
| **Step #3:** Macro spawns PowerShell silently[cite: 1] | Inactive if macro code lacks known signature patterns[cite: 1]. | **Flags Alert:** Suspicious parent-child relationship (`winword.exe` -> `powershell.exe`)[cite: 1]. |
| **Step #4:** Obfuscated PowerShell pulls stage-2 payload[cite: 1] | Script obfuscation frequently evades standard AV pattern matching[cite: 1]. | Flags suspicious command-line execution and staging behavior[cite: 1]. |
| **Step #5:** Payload injected into `svchost.exe`[cite: 1] | No visibility into runtime memory space injection[cite: 1]. | **Detects Process Injection** targeting legitimate system binaries[cite: 1]. |
| **Step #6:** Outbound connection established to C2 server[cite: 1] | Lacks socket-to-process correlation visibility[cite: 1]. | Flags abnormal outbound network activity originating from `svchost.exe`[cite: 1]. |
| **Final Outcome** | **May mark activity as clean**[cite: 1]. | **Generates alert with full attack chain context**[cite: 1]. |

---

## 3. Architecture & Operational Mechanisms

![](../../assets/Pasted%20image%2020261013185314.png)

* **Agents (Sensors):** Lightweight services running on the operating system to collect endpoint telemetry and execute basic local detections[cite: 1].  
* **EDR Console:** The aggregation layer where agent telemetry is evaluated against correlation logic, machine learning models, and threat intelligence[cite: 1].  
* **Ecosystem Integration:** While EDR provides granular endpoint visibility, enterprise environments forward EDR events into a central **SIEM** alongside firewalls, proxies, DLPs, and email gateways for holistic correlation[cite: 1].

### Key Telemetry Categories
![](../../assets/Pasted%20image%2020261013200222.png)

1. **Process Tracking:** Tracks executions and terminations, uncovering anomalous parent-child relationships[cite: 1].
2. **Network Connections:** Identifies outbound C2 sockets, unauthorized ports, and lateral movement attempts[cite: 1].
3. **Command-Line Activity:** Captures raw CLI arguments and parameters, exposing obfuscated scripting[cite: 1].
4. **File System Auditing:** Monitors staging directories for ransomware preparations or malware drops[cite: 1].
5. **Registry Modifications:** Detects changes to startup keys, services, and persistence mechanisms[cite: 1].

### Detection Strategies
![Detection Strategies](../../assets/Pasted%20image%2020261013230826.png)

* **Behavioral Detection:** Evaluates process actions dynamically (e.g., document editors spawning command shells)[cite: 1].
* **Anomaly Detection:** Flags deviations from baseline workstation activity[cite: 1].
* **IOC Matching:** Matches hashes, IPs, and domains against curated Threat Intel feeds[cite: 1].
* **MITRE ATT&CK Alignment:** Translates observed operations directly into standardized tactics and techniques[cite: 1].

### Response & Forensics
Beyond standard quarantine and process kills, consoles provide live containment options and interactive remote shells (e.g., CrowdStrike Falcon Real Time Response - RTR)[cite: 1].

![](../../assets/Pasted%20image%2020261013230932.png)

Analysts can collect volatile evidence remotely without physical access:
* Memory dumps[cite: 1]
* Windows Event logs[cite: 1]
* Specific file and folder paths[cite: 1]
* Registry hives[cite: 1]

---

## 4. Triage Scenario: Investigating Endpoint Detections

Assuming the role of a SOC Analyst at TECH THM, alert triage was conducted on multiple endpoint detections across the environment[cite: 1].

### Incident 1: Payload Retrieval (`DESKTOP-HR01`)
* **Trigger Process:** `cmd.exe` launched a download utility[cite: 1].
* **Tool Utilized:** `CURL.exe`[cite: 1]
* **Downloaded File Path:** `C:\Users\Public\install.exe`[cite: 1]
* **Analysis:** The command line reflects external binary retrieval directly into `C:\Users\Public`. Because this path provides write permissions to standard user accounts, it is routinely abused by threat actors for payload staging.

![](../../assets/Pasted%20image%2020261013235141.png)
![](../../assets/Pasted%20image%2020261013235154.png)

### Incident 2: Suspicious Process & Exfiltration Attempt (`WIN-ENG-LAPTOP03`)
* **Suspect Binary:** `syncsvc.exe`[cite: 1]
* **Binary Path:** `C:\Users\haris.khan\AppData\Local\Temp\syncsvc.exe`[cite: 1]
* **Target Destination:**  
  `https[:]//files-wetransfer[.]com/upload/session/ab12cd34ef56/dump_2025.dmp`[cite: 1]
* **Analysis:** A rogue executable staged under a user `AppData\Local\Temp` path attempted an outbound transmission of a `.dmp` file toward a public file-transfer service, suggesting credential or memory dumping followed by exfiltration.

![](../../assets/Pasted%20image%2020261013235612.png)
![](../../assets/Pasted%20image%2020261013235830.png)

### Incident 3: Administrative Activity Validation (`DESKTOP-DEV01`)
* **Binary Assessed:** `UpdateAgent.exe`[cite: 1]
* **Threat Intel Classification:** `Known internal IT utility tool`[cite: 1]
* **Verdict:** Validated internal administrative activity; marked as a **False Positive (FP)**[cite: 1].

![](../../assets/Pasted%20image%2020261013235904.png)

---

## Identified Indicators of Compromise (IOCs)

| Type | Indicator | Context / Source |
| :--- | :--- | :--- |
| **File Path** | `C:\Users\Public\install.exe`[cite: 1] | Staged payload downloaded via `curl.exe` (`DESKTOP-HR01`)[cite: 1] |
| **File Path** | `C:\Users\haris.khan\AppData\Local\Temp\syncsvc.exe`[cite: 1] | Unsigned/masqueraded service binary (`WIN-ENG-LAPTOP03`)[cite: 1] |
| **URL** | `https[:]//files-wetransfer[.]com/upload/session/ab12cd34ef56/dump_2025.dmp`[cite: 1] | Exfiltration destination URL (`WIN-ENG-LAPTOP03`)[cite: 1] |

---

## Room Solutions & Review

<details>
<summary>Click to view TryHackMe Room Q&A</summary>

* **Which feature of EDR provides a complete context for all the detections?**  
  *Answer:* `Visibility`[cite: 1]
* **Which process spawned sc.exe?**  
  *Answer:* `cmd.exe` (identified by tracing the process tree hierarchy)[cite: 1]
* **In the given analogy, what presents an AV?**  
  *Answer:* `Immigration Check`[cite: 1]
* **Which legitimate process was hijacked by the attacker in the scenario?**  
  *Answer:* `svchost.exe` (referenced in Step 6 of the attack progression)[cite: 1]
* **Which security solution might mark this activity as clean?**  
  *Answer:* `Antivirus`[cite: 1]
* **Which component of the EDR is responsible for collecting telemetry from the endpoints?**  
  *Answer:* `Agent`[cite: 1]
* **An EDR agent is also known as a?**  
  *Answer:* `Sensor`[cite: 1]
* **Which telemetry data helps in detecting C2 communications?**  
  *Answer:* `Network Connections`[cite: 1]
* **Where are the configuration settings of a Windows system primarily stored?**  
  *Answer:* `Registry`[cite: 1]
* **Which feature of the EDR helps you identify threats based on known malicious behaviours?**  
  *Answer:* `IOC matching`[cite: 1]
* **Which tool was launched by CMD.exe to download the payload on DESKTOP-HR01?**  
  *Answer:* `CURL.exe`[cite: 1]
* **What is the absolute path to the downloaded malware on the DESKTOP-HR01 machine?**  
  *Answer:* `C:\Users\Public\install.exe`[cite: 1]
* **What is the absolute path to the suspicious syncsvc.exe on the WIN-ENG-LAPTOP03 machine?**  
  *Answer:* `C:\Users\haris.khan\AppData\Local\Temp\syncsvc.exe`[cite: 1]
* **On which URL was the exfiltration attempt being made on WIN-ENG-LAPTOP03?**  
  *Answer:* `https[:]//files-wetransfer[.]com/upload/session/ab12cd34ef56/dump_2025.dmp`[cite: 1]
* **What was UpdateAgent.exe labelled by Threat Intel on DESKTOP-DEV01?**  
  *Answer:* `Known internal IT utility tool`[cite: 1]

</details>