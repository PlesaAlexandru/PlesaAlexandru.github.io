---
title: Windows Threat Detection 1 Walkthrough
description: Learn to detect Initial Access techniques on Windows hosts using Event Viewer and Sysmon logs.
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

In this room, I practiced detecting common Initial Access techniques on Windows endpoints using Event Viewer and Sysmon logs. Starting with the MITRE ATT&CK Initial Access tactic, I analyzed `RDP-Security.evtx` to investigate an RDP brute-force campaign (`Event ID 4625`) and identify the threat actor's IP address and workstation name after a successful logon (`Event ID 4624`). After that, I examined several phishing scenarios involving misleading `.com` files, malicious `.lnk` shortcuts, and double extensions (`best-cat.jpg.exe`), tracing the full attack chain in `Phishing-Sysmon.evtx` from browser download (`Event ID 15`) and archive extraction (`Event ID 11`) to C2 DNS resolution (`Event ID 22`). Finally, I inspected `USB-Sysmon.evtx` to track removable media malware launching from a USB drive, dropping a payload locally, and propagating across drives.

---

**Learning Objectives**

- Explore how threat actors access and breach Windows machines
- Learn common Initial Access techniques via real-world examples
- Practice detecting every technique using Windows event logs

---
## Task 1.1
Which MITRE technique ID describes Initial Access via a vulnerable mail server?

**Answer**: `T1190`

**Why?**
Looking at the MITRE ATT&CK page for **Exploit Public-Facing Application**, adversaries can gain initial access by exploiting weaknesses in internet-facing systems such as web servers, databases, or mail servers. In the screenshot, the technique ID listed on the right panel is `T1190`.

![](../../assets/Pasted%20image%2020261108162056.png)

## Task 1.2
Which Initial Access method relies on a user opening a malicious email attachment?

**Answer**: `Phishing`

**Why?**
Checking the MITRE ATT&CK framework under the Initial Access tactic (`T1566`), **Phishing** is the technique where adversaries send deceptive messages with malicious attachments or links to trick users into executing malicious code on their system.

![](../../assets/Pasted%20image%2020261108162336.png)

## Task 2.1
Which user seems to be most actively targeted by botnets?

**Answer**: `Administrator`

**Why?**
Opening `RDP-Security.evtx` in Event Viewer and inspecting the `Event ID 4625` (An account failed to log on) events shows repeated brute-force login attempts coming from external IP addresses. Under the **Account For Which Logon Failed** section, the targeted **Account Name** is `ADMINISTRATOR`.

![](../../assets/Pasted%20image%2020261108163807.png)

## Task 2.2
Which IP managed to breach the host and enter the RDP session?

**Answer**: `203.205.34.107`

**Why?**
Filtering `RDP-Security.evtx` for `Event ID 4624` (An account was successfully logged on) and looking for **Logon Type: 10** (RemoteInteractive / RDP) reveals a successful Administrator login at `9:51:47 AM`. Checking the **Network Information** section at the bottom shows the **Source Network Address** as `203.205.34.107`.

![](../../assets/Pasted%20image%2020261108164133.png)

## Task 2.3
Can you get the real Workstation Name (hostname) of the threat actor?

**Answer**: `DESKTOP-QNBC4UU`

**Why?**
Checking the `Event ID 4624` network logon event (`Logon Type: 3`) recorded two seconds earlier at `9:51:45 AM` from the same IP (`203.205.34.107`), NTLM authentication captures the connecting client's hostname. Under **Network Information**, the **Workstation Name** field shows `DESKTOP-QNBC4UU`.

![](../../assets/Pasted%20image%2020261108164755.png)

## Task 3.1
Run the file from the Phishing Case 1 folder, which flag do you get?

**Answer**: `THM{misleading_extension}`

**Why?**
In the `Phishing Case 1` folder, the file is named `www.skype.com`. While `.com` looks like a top-level domain, Windows treats `.com` as an MS-DOS executable application extension. Running `C:\Users\Administrator\Desktop\Practice\Phishing Case 1\www.skype.com` in Command Prompt executes the binary and prints the flag `THM{misleading_extension}`.

![](../../assets/Pasted%20image%2020261108164855.png)

## Task 3.2
From which URL does the malicious LNK download the next-stage malware?

**Answer**: `http://wp16.hqywlqpa.thm:8000/cgi-bin/f`

**Why?**
Right-clicking the `Official Website` shortcut in `Phishing Case 2` and opening **Properties -> Shortcut** reveals that the **Target** field runs PowerShell with `Invoke-WebRequest` (`iwr`). Looking at the highlighted command string inside the target box, it downloads the next stage from `http://wp16.hqywlqpa.thm:8000/cgi-bin/f`.

![](../../assets/Pasted%20image%2020261108165040.png)

## Task 3.3
What is the name of the double-extension file you see there?

**Answer**: `best-cat.jpg.exe`

**Why?**
After enabling **File name extensions** in the File Explorer **View** tab and opening the `Phishing Case 3` folder, we can see that the file disguised as an image actually has a double extension: `best-cat.jpg.exe` (listed as `Type: Application`).

![](../../assets/Pasted%20image%2020261108165114.png)

## Task 4.1
Which file did the user download from the browser? (Provide the full path)

**Answer**: `C:\Users\Administrator\Downloads\top-cats.zip`

**Why?**
Opening `Phishing-Sysmon.evtx` and checking `Event ID 15` (`FileCreateStreamHash`) at `6:58:28 PM` shows Microsoft Edge (`msedge.exe`) downloading an archive from the internet. The **TargetFilename** field shows the full path `C:\Users\Administrator\Downloads\top-cats.zip`.

![](../../assets/Pasted%20image%2020261108165402.png)

## Task 4.2
To which folder did the user extract the archive contents? (Provide the full path)

**Answer**: `C:\Users\Administrator\Pictures`

**Why?**
Looking at `Event ID 11` (`FileCreate`) at `6:58:43 PM`, `Explorer.EXE` extracted the double-extension file to `TargetFilename: C:\Users\Administrator\Pictures\best-cat.jpg.exe`, which means the archive contents were extracted into `C:\Users\Administrator\Pictures`.

![](../../assets/Pasted%20image%2020261108165455.png)

## Task 4.3
What is the Process ID (PID) of the launched phishing malware?

**Answer**: `5484`

**Why?**
I found the terminated PID by checking `Event ID 5` (`ProcessTerminate`) at `6:59:37 PM` (as well as the `ProcessCreate` event for `C:\Users\Administrator\Pictures\best-cat.jpg.exe`). In the event details, `best-cat.jpg.exe` had the **ProcessId** `5484`.

![](../../assets/Pasted%20image%2020261108165721.png)

## Task 4.4
Which C2 domain did the malware try to connect to?

**Answer**: `rjj.store`

**Why?**
Inspecting `Event ID 22` (`DnsQuery`) at `6:59:18 PM` for `ProcessId: 5484` (`C:\Users\Administrator\Pictures\best-cat.jpg.exe`) shows the malware performing a DNS lookup for **QueryName**: `rjj.store`.

![](../../assets/Pasted%20image%2020261108165818.png)

## Task 5.1
Which file on the USB did the user launch? (Provide the full path)

**Answer**: `E:\Open Sandisk 4GB USB.exe`

**Why?**
Opening `USB-Sysmon.evtx` and checking `Event ID 1` (`ProcessCreate`) at `7:34:06 PM` shows the user executing an executable masquerading as a USB drive shortcut directly from the `E:\` volume: `E:\Open Sandisk 4GB USB.exe`.

![](../../assets/Pasted%20image%2020261108170101.png)

## Task 5.2
Which suspicious file did the USB malware drop on the disk? (Provide the full path)

**Answer**: `C:\Users\Public\Documents\winupdate.exe`

**Why?**
Looking at the next event at `7:34:06 PM` (`Event ID 11` - `FileCreate`), `E:\Open Sandisk 4GB USB.exe` (`ProcessId: 1108`) created a local executable on the system at **TargetFilename**: `C:\Users\Public\Documents\winupdate.exe`.

![](../../assets/Pasted%20image%2020261108170127.png)

## Task 5.3
To which drive letter did the malware propagate itself? (Format: X:)

**Answer**: `F:`

**Why?**
Checking the following `Event ID 11` (`FileCreate`) at `7:34:06 PM`, `E:\Open Sandisk 4GB USB.exe` copied itself onto another removable drive at **TargetFilename**: `F:\Open Data Traveler 32 GB USB.exe`, meaning it propagated to the `F:` drive.

![](../../assets/Pasted%20image%2020261108170208.png)
