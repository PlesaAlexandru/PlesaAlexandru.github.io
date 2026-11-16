---
title: Alert Triage With Elastic Walkthrough
description: Investigate SOC alerts and correlate multi-source logs using Kibana and the Elastic Stack.
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

When investigating multi-stage intrusions, a single log source rarely tells the whole story. In this walkthrough, we use Kibana and the Elastic Stack to correlate web server access logs, Windows Security Event Logs, Sysmon process telemetry, and PowerShell Script Block logs (`Event ID 4104`) to trace an Exchange ProxyLogon compromise from initial web shell execution all the way through backdoor account creation and RAR archive staging.

---

**Learning Objectives**

- Use Kibana to analyze common security logs
- Learn how to identify key indicators of compromise (IoCs)
- Correlate events across multiple log sources
- Uncover the breach through a series of SOC alerts

---
## Task 1.1
**How many logs are available for analysis within the entire time range?**

**Answer**: `1467`

**Why?**
Opening the `Alert Triage With Elastic` data view in Kibana Discover across the full incident window (`Jul 20, 2025 @ 03:40:50.836 - Jul 20, 2025 @ 05:35:29.130`) displays `Documents (1,467)` at the top of the table.

![](../../assets/Pasted%20image%2020261114180528.png)

## Task 1.2
**What is the field value for the `client.ip` in the `weblogs` index?**

**Answer**: `203.0.113.55`

**Why?**
Filtering with KQL for `_index:weblogs` returns `65` records. Clicking on the `client.ip` field on the left sidebar reveals that `100%` of those web requests came from `203.0.113.55`.

![](../../assets/Pasted%20image%2020261114180601.png)

## Task 2.1
**What is the `url.path` targeted by the suspicious `POST` requests?**

**Answer**: `/ecp/proxyLogon.ecp`

**Why?**
Querying `_index:weblogs and client.ip:203.0.113.55 and http.request.method:POST` isolates `3` `POST` requests, all targeting `/ecp/proxyLogon.ecp` (indicative of CVE-2021-26855 / Microsoft Exchange ProxyLogon exploitation).

![](../../assets/Pasted%20image%2020261114181723.png)

## Task 2.2
**What is the `user.agent` associated with these `POST` requests?**

**Answer**: `python-requests/2.25.1`

**Why?**
Inspecting the `user.agent` field for those `3` `POST` records shows `python-requests/2.25.1` across `100%` of the events, confirming an automated Python exploit script was used.

![](../../assets/Pasted%20image%2020261114181908.png)

## Task 2.3
**What is the name of the web shell used by the attacker?**

**Answer**: `errorEE.aspx`

**Why?**
Filtering `_index:weblogs and client.ip:203.0.113.55 and cmd=` returns `20` command execution requests. Inspecting `url.path` shows every command being passed to `/errorEE.aspx?cmd=...`.

![](../../assets/Pasted%20image%2020261114182009.png)

## Task 2.4
**What was the first command executed by the attacker via the web shell?**

**Answer**: `hostname`

**Why?**
Checking the earliest web shell request at `Jul 20, 2025 @ 04:45:50.000` (`_index:weblogs and client.ip:203.0.113.55 and cmd=errorEE.aspx`) and expanding its `message` field shows `GET /errorEE.aspx?cmd=hostname`.

![](../../assets/Pasted%20image%2020261114182346.png)
![](../../assets/Pasted%20image%2020261114182415.png)

## Task 3.1
**What is the `winlog.record_id` of the Administrator `4624` logon event?**

**Answer**: `17166`

**Why?**
Pivoting to Windows Security logs at `05:11:22` with `@timestamp >= "2025-07-20T05:11:22" and winlog.event_id:4624 and host.name:winserv2019.some.corp and winlog.event_data.TargetUserName:"Administrator"` returns a single logon event whose `winlog.record_id` is `17166`.

![](../../assets/Pasted%20image%2020261114182613.png)

## Task 3.2
**What is the `process.pid` of the Sysmon Event ID 1 event that occurred on `Jul 20, 2025 @ 05:11:27.996`?**

**Answer**: `964`

**Why?**
Filtering for `@timestamp >= "2025-07-20T05:11:22" and winlog.event_id:1 and user.name:Administrator` at `Jul 20, 2025 @ 05:11:27.996` isolates the Sysmon process creation event.

![](../../assets/Pasted%20image%2020261114182657.png)

Inspecting the document fields shows `"process.pid": [ "964" ]` (with `"process.parent.pid": [ "1192" ]`).

![](../../assets/Pasted%20image%2020261114182749.png)

## Task 4.1
**Which Windows Security Event ID corresponds to the creation of a new user account?**

**Answer**: `4720`

**Why?**
Querying `winlog.channel:Security and winlog.task:User Account Management` returns `23` records across `3` accounts (`svc_backup`, `Administrator`, and `WINSERV2019$`).

![](../../assets/Pasted%20image%2020261114191024.png)

By filtering for the `Administrator` actor and `winlog.event_data.SamAccountName:svc_backup`, we isolate the exact `A user account was created` record (`Jul 20, 2025 @ 05:13:10.009`), which logs under `winlog.event_id` `4720`.

![](../../assets/Pasted%20image%2020261114191743.png)

## Task 4.2
**What is the username of the new backdoor account created by the attacker?**

**Answer**: `svc_backup`

**Why?**
Inspecting `winlog.event_data.TargetUserName` inside that Event ID `4720` record confirms the newly created account is `svc_backup`.

![](../../assets/Pasted%20image%2020261114191837.png)

## Task 4.3
**Which parent process spawned the `net.exe` commands that added `svc_backup` to privileged local groups?**

**Answer**: `cmd.exe`

**Why?**
Filtering `@timestamp >= "2025-07-20T05:13:15" and process.parent.name:cmd.exe and user.name:Administrator` reveals three `net.exe` commands (`net localgroup "Server Operators" svc_backup /add`, `net localgroup "Remote Desktop Users" svc_backup /add`, and `net localgroup Administrators svc_backup /add`), all spawned by `process.parent.name` `cmd.exe`.

![](../../assets/Pasted%20image%2020261114192149.png)

## Task 4.4
**What is the `winlog.record_id` of the Security event where `svc_backup` was added to the `Administrators` group?**

**Answer**: `17254`

**Why?**
Querying `@timestamp >= "2025-07-20T05:13:15" and winlog.event_id:4732` returns the three local group membership additions (`17245`, `17250`, and `17254`). Expanding the `05:13:28.091` document (`winlog.record_id: 17254`) confirms `Group Name: Administrators`.

![](../../assets/Pasted%20image%2020261114192655.png)

## Task 4.5
**What commands were executed in PowerShell Script Block logs (`event.code: 4104`) during the post-exploitation phase?**

**Answer**: `dir /s /b C:\*.xml` (and `net group "Domain Admins" /domain` / `net localgroup Administrators`)

**Why?**
Filtering `@timestamp >= "2025-07-20T05:13:15" and event.module:powershell and event.code:"4104"` and adding `powershell.file.script_block_text` as a column reveals the attacker enumerating `net localgroup Administrators` (`6794`), `net group "Domain Admins" /domain` (`6796`), and searching the drive for XML files via `dir /s /b C:\*.xml` (`6798`).

![](../../assets/Pasted%20image%2020261114192835.png)

## Task 4.6
**What is the name of the archive file created by the attacker using `Rar.exe`?**

**Answer**: `finance_it_archive.rar`

**Why?**
Querying `process.name: "Rar.exe"` returns two Sysmon events at `05:17:55.918`. Inspecting the `CommandLine` field shows the attacker staging sensitive documents into a password-protected RAR archive:

```text
"C:\Program Files\WinRAR\rar.exe" a -hpSpring2025! -m5 C:\Temp\finance_it_archive.rar C:\Users\asmith\Documents\* C:\IT\Admin\Scripts\*
```

![](../../assets/Pasted%20image%2020261114192958.png)
