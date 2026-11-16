---
title: File and Hash Threat Intel Walkthrough
description: Learn to enrich file hashes and sandbox telemetry using threat intelligence platforms.
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

In this room, we practice enriching suspicious files and cryptographic hashes using local PowerShell utilities (`Get-FileHash`), public threat intelligence platforms (VirusTotal, MalwareBazaar, Hybrid Analysis), and the internal **TryDetectThis 2.0** portal. We inspect file naming tricks like double extensions (`payroll.pdf.exe`), analyze sandbox process trees and command lines for `bl0gger.exe` and `Morse-Code-Analyzer.exe`, and complete a full threat intelligence challenge on an Akira ransomware sample (`Challenge.bin.sample`).

---

**Learning Objectives**

- Interpret suspicious filepaths and filenames using heuristics
- Generate and validate file hashes
- Leverage VirusTotal and MalwareBazaar to enrich newly observed binaries
- Extract behavior from sandbox telemetry and map it to MITRE ATT&CK

---
## Task 1.1
*One file displays one of the indicators mentioned. Can you identify the file and the indicator?*

**Answer**: `payroll.pdf, Double extensions`

**Why?**
Opening the `CTI Files` directory on the Desktop, we see a file displayed as `payroll.pdf`, yet the `Type` column in File Explorer (and right-clicking **Properties**) shows `Application (.exe)`. Because Windows hides known file extensions by default, the attacker named the file `payroll.pdf.exe` using **Double extensions** to trick the user into thinking it is a harmless PDF document.

![](../../assets/Pasted%20image%2020261111224041.png)

![](../../assets/Pasted%20image%2020261111230206.png)

## Task 2.1
*What is the SHA256 hash of the file bl0gger?*

**Answer**: `2672B6688D7B32A90F9153D2FF607D6801E6CBDE61F509ED36D0450745998D58`

**Why?**
Opening PowerShell inside `C:\Users\matthew.collins\Desktop\CTI Files` and running `Get-FileHash -Algorithm SHA256 bl0gger.exe` outputs the SHA256 hash `2672B6688D7B32A90F9153D2FF607D6801E6CBDE61F509ED36D0450745998D58`.

![](../../assets/Pasted%20image%2020261111230341.png)

## Task 2.2
*On VirusTotal / TryDetectThis, what is the threat label used to identify the malicious file?*

**Answer**: `trojan.graftor/flystudio`

**Why?**
If we look up the hash on live VirusTotal, the popular threat label currently shows `trojan.blackmoon/flystudio`, but since the room uses the internal **TryDetectThis 2.0** platform to keep answers consistent, searching the hash there shows `Classification: trojan.graftor/flystudio`.

![](../../assets/Pasted%20image%2020261111230609.png)
![](../../assets/Pasted%20image%2020261111230620.png)

## Task 2.3
*When was the file first submitted for analysis? (Answer format: YYYY-MM-DD HH:MM:SS)*

**Answer**: `2025-05-15 12:03:49`

**Why?**
In the **TryDetectThis 2.0** report for `bl0gger.exe`, the top-right summary box and `Basic Properties` list `Submitted: 2025-05-15T12:03:49Z`, which formats to `2025-05-15 12:03:49`.

![](../../assets/Pasted%20image%2020261111230714.png)

## Task 2.4
*According to MalwareBazaar / TryDetectThis, which vendor classified the Morse-Code-Analyzer file as non-malicious?*

**Answer**: `CyberFortress`

**Why?**
First, we calculate the SHA256 hash of `Morse-Code-Analyzer.exe` in PowerShell (`Get-FileHash -Algorithm SHA256 .\Morse-Code-Analyzer.exe` -> `1F8806869616C18CBAE9FFCF581C0428915D32FB70119DF16D08078D92D1A5E3`). Querying that hash in **TryDetectThis 2.0** and checking **Vendor Analysis** shows **`CyberFortress`** flagging it as `Clean / Non-malicious` (`benign`).

![](../../assets/Pasted%20image%2020261111230824.png)
![](../../assets/Pasted%20image%2020261111230957.png)

## Task 2.5
*What MITRE technique has been flagged for persistence and privilege escalation for the Morse-Code-Analyzer file?*

**Answer**: `DLL Side-Loading`

**Why?**
Scrolling down to **Behavioural Analysis -> Technique Mapping** for `Morse-Code-Analyzer`, we see `T1574.002` (**`DLL Side-Loading`**) flagged under `Persistence, Privilege Escalation`.

![](../../assets/Pasted%20image%2020261111231118.png)

## Task 3.1
*What tags are used to identify the bl0gger.exe malicious file on Hybrid Analysis? (Answer format: Tag1, Tag2, Tag3)*

**Answer**: `BlackMoon, Discovery, windows-server-utility`

**Why?**
Searching the `bl0gger.exe` SHA256 hash on **Hybrid Analysis** and checking the **Behavior Tags** section shows the tags `BlackMoon`, `Discovery`, and `windows-server-utility` (alongside `agent`, `flystudio`, `trojan`).

![](../../assets/Pasted%20image%2020261111232614.png)

## Task 3.2
*What was the stealth command line executed from the file?*

**Answer**: `regsvr32 %WINDIR%\Media\ActiveX.ocx /s`

**Why?**
Looking at the **Process Tree** in the Hybrid Analysis report for `bl0gger.exe`, the main process (`PID: 7148`) spawns `regsvr32.exe (PID: 2876)` with the silent (`/s`) command line: `regsvr32 %WINDIR%\Media\ActiveX.ocx /s`.

![](../../assets/Pasted%20image%2020261111232753.png)

## Task 3.3
*Which other process was spawned according to the process tree?*

**Answer**: `werfault.exe`

**Why?**
Directly underneath `regsvr32.exe` in the same **Process Tree**, we see two instances of **`werfault.exe`** (`PID: 2400` and `PID: 7200`) spawned by the sample.

![](../../assets/Pasted%20image%2020261111232840.png)

## Task 3.4
*The payroll.pdf application seems to be masquerading as which known Windows file?*

**Answer**: `svchost.dll` (or `svchost.exe`)

**Why?**
Running `Get-FileHash -Algorithm SHA256 .\payroll.pdf.exe` gives `D202ED020ED8E36BD8A0F5B571A19D386C12ABECB2A28C989D50BBF92C78F54E`. Searching this hash in **TryDetectThis 2.0** and checking **Sandbox Analysis -> Impersonation Behaviour** reveals that the sample masquerades as **`svchost.dll`** (`svchost.exe` on VirusTotal).

![](../../assets/Pasted%20image%2020261111232908.png)
![](../../assets/Pasted%20image%2020261111233147.png)

![](../../assets/Pasted%20image%2020261111233241.png)

## Task 3.5
*What associated URL is linked to the file?*

**Answer**: `hxxp://121.182.174.27:3000/server.exe`

**Why?**
Switching to the **Network Indicators** tab under **Sandbox Analysis** for `payroll.pdf.exe` shows the defanged download URL **`hxxp://121.182.174.27:3000/server.exe`** over non-standard HTTP port `3000`.

![](../../assets/Pasted%20image%2020261111233344.png)

## Task 3.6
*How many extracted strings were identified from the sandbox analysis of the file?*

**Answer**: `454`

**Why?**
Clicking on the **Extracted Strings** tab in **Sandbox Analysis** shows a total count of **`454`** strings extracted from the binary.

![](../../assets/Pasted%20image%2020261111233424.png)

## Task 4.1
*What is the SHA256 hash of the file Challenge.bin.sample?*

**Answer**: `43B0AC119FF957BB209D86EC206EA1EC3C51DD87BEBF7B4A649C7E6C7F3756E7`

**Why?**
In PowerShell, running `Get-FileHash -Algorithm SHA256 .\Challenge.bin.sample` returns `43B0AC119FF957BB209D86EC206EA1EC3C51DD87BEBF7B4A649C7E6C7F3756E7`.

![](../../assets/Pasted%20image%2020261111233548.png)
![](../../assets/Pasted%20image%2020261111233640.png)

## Task 4.2
*What family labels are assigned to the file on VirusTotal / TryDetectThis?*

**Answer**: `akira, filecryptor`

**Why?**
Searching the `Challenge.bin.sample` hash in **TryDetectThis 2.0** shows `Classification: ransomware.akira/filecryptor` (and `Malicious: 61` / `61/72 flagged`), giving the two family labels **`akira, filecryptor`**.

![](../../assets/Pasted%20image%2020261111233631.png)

## Task 4.3
*When was the file known in the wild? (Answer format: YYYY-MM-DD HH:MM:SS)*

**Answer**: `2023-07-09 01:30:00`

**Why?**
In the top-right box of the **TryDetectThis 2.0** report for `Challenge.bin.sample`, the **ITW** (In The Wild) timestamp is `2023-07-09T01:30:00Z` (`2023-07-09 01:30:00`), with `Submitted: 2023-07-10T08:22:15Z`.

![](../../assets/Pasted%20image%2020261111233712.png)

## Task 4.4
*Name the text file dropped during the execution of the malicious file.*

**Answer**: `akira_readme.txt`

**Why?**
Under **Sandbox Analysis -> Dropped Files**, we see that the ransomware writes the ransom note **`akira_readme.txt`** into every encrypted directory.

![](../../assets/Pasted%20image%2020261111233748.png)

## Task 4.5
*What PowerShell script is observed to be executed?*

**Answer**: `Get-WmiObject Win32_Shadowcopy | Remove-WmiObject`

**Why?**
Switching to the **Command Execution** tab under **Sandbox Analysis** shows the PowerShell command **`Get-WmiObject Win32_Shadowcopy | Remove-WmiObject`**, which deletes all Volume Shadow Copies so the victim cannot restore encrypted files from local backups.

![](../../assets/Pasted%20image%2020261111233809.png)

## Task 4.6
*What is the MITRE ATT&CK ID associated with this execution?*

**Answer**: `T1490`

**Why?**
Under **Sandbox Analysis -> Technique Mapping**, deleting Volume Shadow Copies to prevent system recovery maps to MITRE ATT&CK technique **`T1490`** (**Inhibit System Recovery**).

![](../../assets/Pasted%20image%2020261111234018.png)

---

## Personal Note: File & Hash Threat Intel Lookup Cheat Sheet

Quick summary of the hash lookup operators and pivoting techniques used across this room:

| Platform / Tool | Search Syntax & Pivot Features | Best Used For |
| :--- | :--- | :--- |
| **PowerShell / Linux CLI** | - Windows: `Get-FileHash -Algorithm SHA256 <file>`<br>- Linux: `sha256sum <file>` / `md5sum <file>` | Generating hashes locally before querying external TI platforms. |
| **VirusTotal** | - Direct hash search (`MD5`, `SHA1`, `SHA256`)<br>- Icon pivot: `main_icon_hash:<dhash>`<br>- **Details** (`Names`, `History`), **Relations** (`Execution Parents`, `Dropped Files`), **Behavior** (`MITRE ATT&CK`, `DNS`, `Mutexes`) | Checking vendor detections, finding masquerading samples that reuse the same icon (`pdf` / `acrobat` icons), and reviewing sandbox behavior. |
| **MalwareBazaar (abuse.ch)** | - `sha256_hash:<hash>`<br>- `tag:<family>` or `signature:<MalwareFamily>` | Finding malware family attribution (`Lumma Stealer`, `Snake Keylogger`, `AgentTesla`), Intel reports, and **YARA rule hits**. |
| **ThreatFox (abuse.ch)** | - `ioc:<hash_or_ip_or_domain>`<br>- `malware:<family>` | Mapping file hashes directly to their associated C2 IPs, domains, and URLs. |
