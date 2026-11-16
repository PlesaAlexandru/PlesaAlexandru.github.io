---
title: Windows Threat Detection 3 Walkthrough
description: Investigate Command and Control (C2) and persistence mechanisms on compromised Windows systems.
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

In this final room of the Windows Security Monitoring series, I investigated Command and Control (C2) channels and common persistence mechanisms on compromised Windows hosts. First, I analyzed `Sysmon.evtx` in `Task 2` to trace a malicious archive download (`Event ID 15`), identify the dropped C2 binary (`C:\Users\Administrator\AppData\Roaming\update.exe`), and uncover its Cloudflare Workers C2 domain (`Event ID 22`). Next, I examined `Security.evtx` to track an attacker brute-forcing the Administrator account (`Event ID 4625`), creating a local backdoor user named `support` (`Event ID 4720`), and adding it to the `Administrators` group (`Event ID 4732`). After that, I hunted down persistence via Windows Services (`Event ID 4697` - `Data Protection Service`), Scheduled Tasks (`Event ID 4698` - `AmazonSync`), Startup folder scripts (`odin.cmd`), and Registry Run keys (`Event ID 1` / `13` - `Basket` launching `kitten.exe`), before wrapping up with a review of Active Directory ransomware impact and early detection at the Initial Access stage.

---

**Learning Objectives**

- Understand the concept of Command and Control (C2)
- Learn why and how threat actors maintain control over their victims
- Use Windows event logs to uncover various persistence methods
- Observe and analyze how these persistence and C2 techniques function in a hands-on environment

---
## Task 1.1
From which URL did the user download the suspicious archive?

**Answer**: `http://10.14.97.15/URGENT!.zip`

**Why?**
Opening the Sysmon log in `Task 2` and checking `Event ID 15` (`FileCreateStreamHash`) at `8:13:55 PM` shows Chrome (`chrome.exe`) creating the `Zone.Identifier` stream for `C:\Users\Administrator\Downloads\URGENT!.zip`. In the **Contents** field, the `HostUrl` shows `http://10.14.97.15/URGENT%21.zip` (which decodes to `http://10.14.97.15/URGENT!.zip`).

![](../../assets/Pasted%20image%2020261109145853.png)

## Task 1.2
What is the full path to the C2 malware that the user accidentally launched?

**Answer**: `C:\Users\Administrator\AppData\Roaming\update.exe`

**Why?**
Looking at the Sysmon events after the archive was extracted and launched, `Event ID 22` (`DnsQuery`) and `Event ID 1` (`ProcessCreate`) show the C2 binary running from the **Image** path `C:\Users\Administrator\AppData\Roaming\update.exe`.

![](../../assets/Pasted%20image%2020261109150005.png)

## Task 1.3
Which C2 domain did the malware communicate with?

**Answer**: `route.m365officesync.workers.dev`

**Why?**
Checking `Event ID 22` (`DnsQuery`) at `8:14:30 PM` for `C:\Users\Administrator\AppData\Roaming\update.exe` (`ProcessId: 4340`), the **QueryName** field shows the malware communicating with the Cloudflare Workers C2 domain `route.m365officesync.workers.dev`.

![](../../assets/Pasted%20image%2020261109145950.png)

## Task 2.1
How many failed login attempts were made on the Administrator account?

**Answer**: `6`

**Why?**
Opening `C:\Users\Administrator\Desktop\Practice\Task 3\Security.evtx` in Event Viewer and filtering for `Event ID 4625` (An account failed to log on) returns a total of `6` failed login events targeting the `Administrator` account.

![](../../assets/Pasted%20image%2020261109150311.png)

## Task 2.2
What is the username of the backdoor account created by the attacker?

**Answer**: `support`

**Why?**
Filtering `Security.evtx` for `Event ID 4720` (A user account was created) reveals that at `9:01:38 PM`, the `Administrator` account created a new local user with the **Account Name** `support`.

![](../../assets/Pasted%20image%2020261109150607.png)

## Task 2.3
To which privileged group was the backdoor account added?

**Answer**: `Administrators`

**Why?**
Filtering `Security.evtx` for `Event ID 4732` (A member was added to a security-enabled local group) at `9:02:29 PM` shows the `THM-DFIR-VM-3\support` account being added to the `BUILTIN\Administrators` group (**Group Name**: `Administrators`).

![](../../assets/Pasted%20image%2020261109150646.png)

## Task 3.1
What is the name of the malicious service created to persist the "Nessie" malware?

**Answer**: `Data Protection Service`

**Why?**
Opening `C:\Users\Administrator\Desktop\Practice\Task 4\Security (All Time).evtx` and filtering for `Event ID 4697` (A service was installed in the system) at `7:09:16 PM` reveals a service executing `Service File Name: C:\Windows\Help\nessie.exe`. Above that, the **Service Name** is `Data Protection Service`.

![](../../assets/Pasted%20image%2020261109151637.png)

## Task 3.2
What is the name of the scheduled task created to persist the "Troy" malware?

**Answer**: `AmazonSync`

**Why?**
Filtering `Security (All Time).evtx` for `Event ID 4698` (A scheduled task was created) at `7:10:09 PM` shows a scheduled task configured to run `"C:\Program Files\Common Files\troy.exe"`. Under **Task Information**, the **Task Name** is `\AmazonSync` (`AmazonSync`).

![](../../assets/Pasted%20image%2020261109151824.png)

![](../../assets/Pasted%20image%2020261109151810.png)

## Task 3.3
Disable the scheduled task and run the "Troy" malware. What is the flag?

**Answer**: `THM{c2_is_on_schedule!}`

**Why?**
After checking the `AmazonSync` task in Task Scheduler (`"C:\Program Files\Common Files\troy.exe" -d`) and inspecting `Sysmon (After Reboot).evtx` (`Event ID 1`) for `troy.exe`, we find its **ParentCommandLine** is `C:\Windows\system32\svchost.exe -k netsvcs -p -s Schedule`. Running `C:\Program Files\Common Files\troy.exe` and entering that parent command line prints the flag `THM{c2_is_on_schedule!}`.

![](../../assets/Pasted%20image%2020261109152559.png)

![](../../assets/Pasted%20image%2020261109153340.png)

![](../../assets/Pasted%20image%2020261109153418.png)

## Task 4.1
Looking at the Sysmon logs, what is the ParentImage (or ParentCommandLine) of the "Odin" malware?

**Answer**: `C:\Windows\Explorer.EXE`

**Why?**
Opening `Sysmon (After Reboot).evtx` in `Task 5` and finding the `Event ID 1` (`ProcessCreate`) event where `cmd.exe` executes `C:\Users\Administrator\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup\odin.cmd`, the **ParentCommandLine** (and **ParentImage**) is `C:\Windows\Explorer.EXE` because Windows Explorer automatically launches items placed in the user's Startup folder upon logon.

![](../../assets/Pasted%20image%2020261109154522.png)

## Task 4.2
What is the last line that the "Odin" malware outputs?

**Answer**: `Done doing bad stuff!`

**Why?**
Inspecting the child `powershell.exe` process (`Event ID 1`) spawned by `odin.cmd` at `10:16:49 PM` shows the PowerShell one-liner command in `CommandLine`. At the end of the loop, the script runs `Write-Output 'Done doing bad stuff!'`.

![](../../assets/Pasted%20image%2020261109154629.png)

## Task 4.3
Remove the Run key and run the "Kitten" malware. What is the flag?

**Answer**: `THM{persisting_in_basket!}`

**Why?**
Checking `Sysmon (Before Reboot).evtx` (`Event ID 1`) shows `reg.exe` adding a Run key value named `Basket` under `HKCU\Software\Microsoft\Windows\CurrentVersion\Run` pointing to `"C:\Users\Public\kitten.exe"`. After deleting the `Basket` value in Registry Editor (`regedit`) and running `C:\Users\Public\kitten.exe` with the answer `Basket`, the binary outputs the flag `THM{persisting_in_basket!}`.

![](../../assets/Pasted%20image%2020261109154156.png)

![](../../assets/Pasted%20image%2020261109154317.png)

![](../../assets/Pasted%20image%2020261109154302.png)

## Task 5.1
What is the biggest threat to most corporate Windows networks?

**Answer**: `Ransomware`

**Why?**
As highlighted in the **Active Directory and Ransomware** section of Task 6, a Windows corporate network usually centers around Active Directory, where the main threat that can bring entire companies to a halt is `Ransomware`.

![](../../assets/Pasted%20image%2020261109154703.png)

## Task 5.2
At which stage is it best to detect and stop the attack (e.g. Exfiltration)?

**Answer**: `Initial Access`

**Why?**
As stated in the **Threat Detection Recap** section, complex attacks all start from a single breach, so it is best to detect and stop the attack before ransomware causes a disastrous impact, preferably right after `Initial Access`.

![](../../assets/Pasted%20image%2020261109155321.png)