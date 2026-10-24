---
title: Template Walkthrough
description: Template
pubDate: 2026-10-16
categories:
  - SOC
  - Walkthrough
  - TryHackMe
tags:
  - TryHackMe
  - Walkthrough
  - Rooms
image: /image/template.png
---
---
## Summary

-
-
-

---


STAGE 1 - Initial execusion

Tile: Execution of review-helper.ps1
Description: The user S.REYES executed the malicious powershell script from Downloads folder.
Timestamp: 9/10/26  12:29:48.000 PM
Tactic: Execution
Technique: Command and Scripting Interpreter
User: S.REYES
Asset: THM-SREYES
List of IOCs: 
- File path : C:\\Users\\s.reyes\\Downloads\\NeuralNexusReview\\review-helper.ps1\


STAGE 2 - Persistence

Tile: Registry Runkey persistence
Description: The script copied itself Local/Appdata/ path and created a registry runkey to maintain persistence across reboots.
Timestamp: 9/10/26  12:37:20.000 PM
Tactic: Persistence
Technique: Logon autostart execution : Registry runkeys / startup folder
User: S.REYES
Asset: THM-SREYES
List of IOCs: 
- C:\Users\s.reyes\AppData\Local\NeuralNexus\review-helper.ps1
- HKU\S-1-5-21-1966530601-3185510712-10604624-1115\Software\Microsoft\Windows\CurrentVersion\Run\NeuralNexusReview


STAGE 3 - Command and Control (C2)

Tile: C2 connection established
Description: The malware initiated an HTTPS connection to the attacker to poll for task
Timestamp: 2026-09-10 12:29:49
Tactic: Command and Control (c2)
Technique: Application layer protocol: Web Protocols
User: S.REYES
Asset: THM-SREYES
List of IOCs: 
- 45.147.230.131:8443
- 55a5bf42c9e35fca805d28061ab65ef6206909dac051d7e7b7f3e9b897c1913e (CertPin)
- fd0d499479b4dcbbbfb2b90bf40b3b2862a189c1c96e8a62d75a30595fb64cbd (AuthToken)



STAGE 4 - Exfiltration

Tile: Exfiltration of review-cache.zip
Description: The attacker used curl.exe to upload the staged NeuralNexus archive to the C2 Server
Timestamp: 9/10/26 4:27:54.000 PM
Tactic: Exfiltration
Technique: Exfiltration over C2 channel
User: S.REYES
Asset: THM-SREYES
List of IOCs: 
- curl.exe
- review-cache.zip
- 45.147.230.131:8443/ingest/review-cache.zip







# Threat Case Report

## Incident: NeuralNexus Malicious PowerShell Campaign — Host THM-SREYES

**Prepared by:** Senior Threat Hunter **Incident Date:** Thu, 10 Sep 2026 **Severity:** **Critical**

---

### Executive Summary

On **10 September 2026**, user **S.REYES** (`s.reyes@tryhatmestudios.thm`) on asset **THM-SREYES** executed a malicious PowerShell script, `review-helper.ps1`, sourced from a directory named `NeuralNexusReview` within the user's Downloads folder, consistent with a **trojanized software review package** as the initial access vector. The script execution maps to **MITRE ATT&CK Execution (TA0002), T1059 – Command and Scripting Interpreter**, and represents the entry point of a multi-stage intrusion. Within approximately **seven minutes** of initial execution, the malware established **persistence** by copying itself to `C:\Users\s.reyes\AppData\Local\NeuralNexus\review-helper.ps1` and creating a **Registry Run key** under `HKU\...\Software\Microsoft\Windows\CurrentVersion\Run\NeuralNexusReview`, ensuring survival across reboots (**TA0003, T1547**). Notably, a **Command and Control (C2) connection** was initiated to `45.147.230.131:8443` just **one second** after initial script execution at `12:29:49 UTC`, indicating the C2 beacon was embedded directly within the script's primary execution flow (**TA0011, T1071 – Application Layer Protocol over HTTPS**). The C2 infrastructure was further identified by a **certificate pin** (`55a5bf42c9e35fca805d28061ab65ef6206909dac051d7e7b7f3e9b897c1913e`) and an **authentication token** (`fd0d499479b4dcbbbfb2b90bf40b3b2862a189c1c96e8a62d75a30595fb64cbd`), suggesting a purpose-built, authenticated C2 framework. Approximately **four hours** after initial compromise, at `16:27:54 UTC`, the attacker leveraged the established C2 channel to execute **data exfiltration** using the native `curl.exe` binary, uploading a staged archive `review-cache.zip` to `45.147.230.131:8443/ingest/review-cache.zip` (**TA0010, T1041 – Exfiltration Over C2 Channel**). The use of `curl.exe` indicates **living-off-the-land (LotL)** tradecraft to minimize detection during exfiltration. The staged archive naming convention (`review-cache.zip`) is consistent with the NeuralNexus theme, suggesting deliberate data collection and staging prior to exfiltration. The four-hour dwell time between C2 establishment and exfiltration suggests the attacker was actively tasking the implant to enumerate, collect, and stage sensitive data. **Immediate containment actions** are recommended, including isolation of host **THM-SREYES**, disabling the account `s.reyes@tryhatmestudios.thm`, blocking `45.147.230.131` at the perimeter firewall and proxy, and removing the identified registry Run key. Forensic investigation should prioritize recovery and analysis of `review-cache.zip` contents to determine the full scope of data loss. All identified IOCs — including the C2 IP, certificate pin, authentication token, and file paths — should be disseminated to detection engineering for immediate rule creation across SIEM, EDR, and network monitoring platforms. This incident reflects a **targeted, multi-stage intrusion** with characteristics of a deliberate campaign, warranting escalation to threat intelligence teams for attribution and broader campaign analysis.