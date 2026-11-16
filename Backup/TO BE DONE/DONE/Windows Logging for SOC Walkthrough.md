---
title: Windows Logging for SOC Walkthrough
description: Understand how Windows event logs work and how to use them to identify threats.
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

In this room we explored the core Windows telemetry sources used by SOC analysts: Windows Security Event Logs, Sysmon, and PowerShell command history. Using Event Viewer on `Practice-Security.evtx` and `Practice-Sysmon.evtx`, we investigated an RDP brute-force attack (`4625` / `4624`), backdoor user creation and group assignment (`4720` / `4732`), a malicious browser download (`ckjg.exe`) via `Zone.Identifier` alternate data streams (Sysmon Event ID `15`), Startup folder persistence (`11`), C2 network and DNS activity (`3` / `22`), and `PSReadLine` command history files (`ConsoleHost_history.txt`).

---

**Learning Objectives**

- Understand how to find and interpret important Windows event logs
- Learn invaluable monitoring log sources like Sysmon and PowerShell
- Prepare for using the mentioned logs in SOC-SIM and subsequent rooms
- Practice your log analysis skills on multiple event log datasets

---
## Task 1.1

**Looking at the last screenshot, which event ID describes a successful login? (Answer format: LogSource / ID)**  
**Answer**: `Security / 4624`

**Why?**
In the Event Viewer screenshot under `Windows Logs > Security`, Event ID `4624` (`Logon`) shows the message `"An account was successfully logged on."`
![](../../assets/Pasted%20image%2020261108145137.png)

## Task 2.1

**Which IP performed a brute force of the THM-PC?**  
**Answer**: `10.10.53.248`

**Why?**
Opening `Practice-Security.evtx` from the Desktop and filtering by Event ID `4625` (Failed Logon) returns `14` events targeting `Administrator`, all showing `Source Network Address: 10.10.53.248`.

![](../../assets/Pasted%20image%2020261108145847.png)
## Task 2.2

**Which user has been breached as a result of the attack?**  
**Answer**: `Administrator`

**Why?**
Filtering `Practice-Security.evtx` for Event ID `4624` (Successful Logon) right after the failed attempts (`5/17/2025 10:53:41 PM`) shows a successful logon from `10.10.53.248` into `Account Name: Administrator`.

![](../../assets/Pasted%20image%2020261108151525.png)
## Task 2.3

**What was the Logon ID of the malicious RDP login? (Note: The login you are looking for has a Logon Type 10)**  
**Answer**: `0x183C36D`

**Why?**
In the same `4624` event from `10.10.53.248` (which has `Logon Type: 10` for RemoteInteractive / RDP), looking under `New Logon` gives us `Logon ID: 0x183C36D`.

![](../../assets/Pasted%20image%2020261108151637.png)
## Task 3.1

**Which user was created by the attacker soon after the RDP login?**  
**Answer**: `svc_sysrestore`

**Why?**
Filtering `Practice-Security.evtx` by Event ID `4720` (A user account was created) shows that at `10:54:58 PM`, session `0x183C36D` created `Account Name: svc_sysrestore`.
![](../../assets/Pasted%20image%2020261108151803.png)

## Task 3.2

**Which two privileged groups was the backdoor user added to? (Answer in alphabetical order, e.g. "Administrators, Power Users")**  
**Answer**: `Backup Operators, Remote Desktop Users`

**Why?**
Filtering for Event ID `4732` (A member was added to a security-enabled local group) shows `3` events where the new user (`S-1-5-21-...-1013`) was added to `Users`, `Backup Operators`, and `Remote Desktop Users`. Putting the two privileged groups in alphabetical order gives `Backup Operators, Remote Desktop Users`.
![](../../assets/Pasted%20image%2020261108152122.png)

![](../../assets/Pasted%20image%2020261108152109.png)

![](../../assets/Pasted%20image%2020261108152052.png)
## Task 3.3

**Does the Logon ID field match what you saw in the previous task (Yea/Nay)?**  
**Answer**: `Yea`

**Why?**
Under `Subject` in both the `4720` and `4732` events, `Logon ID` is `0x183C36D`, matching the attacker's RDP session from the previous task.
![](../../assets/Pasted%20image%2020261108152229.png)

## Task 4.1

**Which web browser does Sarah use to browse the web?**  
**Answer**: `Google Chrome`

**Why?**
Opening `Practice-Sysmon.evtx` and filtering by Event ID `1` (`Process Create`) shows `Description: Google Chrome` (`C:\Program Files (x86)\Google\Chrome\Application\...`).

![](../../assets/Pasted%20image%2020261108152601.png)
## Task 4.2

**Which file did Sarah download from the browser?**  
**Answer**: `C:\Users\sarah.miller\Downloads\ckjg.exe`

**Why?**
Checking the latest Event ID `1` (`Process Create`) at `4:08:43 PM`, we see `sarah.miller` executing `Image: C:\Users\sarah.miller\Downloads\ckjg.exe`.

![](../../assets/Pasted%20image%2020261108152651.png)

## Task 4.3

**Which URL was the file downloaded from? Note: Use other Sysmon events to find out!**  
**Answer**: `http://gettsveriff.com/bgj3/ckjg.exe`

**Why?**
When a browser downloads a file from the internet, it writes a `Zone.Identifier` Alternate Data Stream (https://isc.sans.edu/diary/Sysmon+and+Alternate+Data+Streams/26292). Filtering `Practice-Sysmon.evtx` by Sysmon Event ID `15` (`FileCreateStreamHash`) shows `TargetFilename: C:\Users\sarah.miller\Downloads\ckjg.exe:Zone.Identifier` with `Contents: [ZoneTransfer] ZoneId=3 HostUrl=http://gettsveriff.com/bgj3/ckjg.exe`.

![](../../assets/Pasted%20image%2020261108153449.png)

![](../../assets/Pasted%20image%2020261108154310.png)
## Task 5.1

**Which file was created by the downloaded malware to persist on the host?**  
**Answer**: `C:\Users\sarah.miller\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup\DeleteApp.url`

**Why?**
Filtering `Practice-Sysmon.evtx` by Event ID `11` (`FileCreate`) at `4:08:46 PM` shows `ckjg.exe` creating `TargetFilename: C:\Users\sarah.miller\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup\DeleteApp.url` (`RuleName: T1023`).

![](../../assets/Pasted%20image%2020261108154626.png)
## Task 5.2

**What is the Command & Control server malware connected to? (Answer in format IP:Port, e.g. 1.1.1.1:80)**  
**Answer**: `193.46.217.4:7777`

**Why?**
Filtering by Sysmon Event ID `3` (`NetworkConnect`) shows `ckjg.exe` initiating a TCP connection to `DestinationIp: 193.46.217.4` on `DestinationPort: 7777`.

![](../../assets/Pasted%20image%2020261108154705.png)


## Task 5.3

**Finally, which domain does the malicious IP correspond to?**  
**Answer**: `hkfasfsafg.click`

**Why?**
Filtering by Sysmon Event ID `22` (`DnsQuery`) at `4:08:58 PM` shows `ckjg.exe` querying `QueryName: hkfasfsafg.click`, which resolves to `::ffff:193.46.217.4`.
![](../../assets/Pasted%20image%2020261108154947.png)

## Task 6.1

**Which PowerShell command was executed first?**  
**Answer**: `Get-ComputerInfo`

**Why?**
Navigating to `C:\Users\Administrator\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine` and opening `ConsoleHost_history.txt` in Notepad shows `Get-ComputerInfo` on the very first line.

![](../../assets/Pasted%20image%2020261108160015.png)

![](../../assets/Pasted%20image%2020261108160032.png)
## Task 6.2

**When did the Administrator run the first PS command? (Format: Month Day, Year)**  
**Answer**: `May 18, 2025`

**Why?**
The whole room dataset takes place on the same date (`5/18/2025`), and right-clicking `ConsoleHost_history.txt` -> `Properties` shows `Created: Sunday, May 18, 2025, 8:49:26 PM` (which is when the first command in the history file was recorded).

![](../../assets/Pasted%20image%2020261108160118.png)

![](../../assets/Pasted%20image%2020261108160149.png)

## Task 6.3

**Can you find the flag stored in the PowerShell history? (Format: THM{...})**  
**Answer**: `THM{it_was_me!}`

**Why?**
Checking the other user profile on the machine (`C:\Users\thm.bob\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt`), we find the command `echo "THM{it_was_me!}" > flag.txt`.

![](../../assets/Pasted%20image%2020261108160642.png)
