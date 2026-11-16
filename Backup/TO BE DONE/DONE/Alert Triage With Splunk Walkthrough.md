---
title: Alert Triage With Splunk Walkthrough
description: Practice investigating realistic SOC alerts and attack scenarios using Splunk.
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

Alert triage is one of the primary responsibilities of a SOC analyst: taking an initial detection trigger and querying the underlying telemetry to determine whether an attack succeeded and how far it progressed. In this walkthrough, we use Splunk to investigate three realistic alerts covering a Linux SSH brute-force and privilege escalation chain, a Windows scheduled task persistence mechanism, and a WordPress Hydra brute-force attack followed by `b374k.php` web shell execution.

---

**Learning Objectives**

- Learn how to properly investigate alerts in a SOC environment
- Understand how to investigate brute-force attacks on Linux systems
- Discover persistence mechanisms on Windows systems
- Analyse a web shell on a vulnerable web server
- Learn how to investigate alerts for three given scenarios using Splunk

---
## Task 1.1
**How many failed login attempts were made on the user `john.smith`?**

**Answer**: `500`

**Why?**
In the Linux Initial Access investigation (`index=linux-alert`), we extract the targeted usernames and actions from the `sshd` logs coming from `10.10.242.248` and aggregate them by `username`:

```spl
index=linux-alert src_ip="10.10.242.248"
| rex field=_raw "^\\d{4}-\\d{2}-\\d{2}T[^\\s]+\\s+(?<log_hostname>\\S+)"
| rex field=_raw "sshd\\[\\d+\\]:\\s*(?<action>Failed)\\s+\\S+\\s+for(?: invalid user)? (?<username>\\S+) from (?<src_ip>\\d{1,3}(?:\\.\\d{1,3}){3})"
| eval process="sshd"
| stats count values(action) values(src_ip) as src_ip values(log_hostname) as hostname values(process) as process by username
```

The statistics table shows `500` failed SSH login attempts against `john.smith`.

![](../../assets/Pasted%20image%2020261114140251.png)

## Task 1.2
**What was the duration of the brute-force attack in minutes?**

**Answer**: `5`

**Why?**
Grouping the failed password attempts for `john.smith` into 1-minute buckets (`bin span=1m _time`) shows the brute-force traffic starting at `09:01:00` and ending at `09:06:00`, spanning `5` minutes.

![](../../assets/Pasted%20image%2020261114142206.png)

## Task 1.3
**Which user did `john.smith` escalate their privileges to?**

**Answer**: `root`

**Why?**
Querying `index="linux-alert" ("sudo:" OR "COMMAND=")` and tabulating the original and target users reveals `john.smith` running `COMMAND=/usr/bin/su` as `USER=root` at `09:11:28`, opening a `root(uid=0)` session.

![](../../assets/Pasted%20image%2020261114142838.png)

## Task 1.4
**Which user account was created by the attacker for persistence?**

**Answer**: `system-utm`

**Why?**
Both the sudo command table above and a direct filter for `index="linux-alert" COMMAND=*adduser*` show the attacker executing `/usr/sbin/adduser system-utm` at `09:12:10` to establish a backdoor account.

![](../../assets/Pasted%20image%2020261114142922.png)

## Task 2.1
**What is the `ProcessId` of the process that created the malicious scheduled task?**

**Answer**: `5816`

**Why?**
Moving to the Windows persistence alert (`index=win-alert`), we filter for Sysmon process creation events (`EventCode=1`) referencing the scheduled task `AssessmentTaskOne`. Expanding the event fields shows `schtasks.exe` running under `ProcessId` `5816`.

![](../../assets/Pasted%20image%2020261114143233.png)

## Task 2.2
**What is the name of the parent process that spawned `schtasks.exe`?**

**Answer**: `cmd.exe`

**Why?**
Scrolling down to the parent process fields of the same Sysmon Event ID 1 record shows `ParentImage: C:\Windows\System32\cmd.exe` (`ParentProcessId: 4128`).

![](../../assets/Pasted%20image%2020261114143304.png)

## Task 2.3
**Which local group was enumerated on the system?**

**Answer**: `Administrators`

**Why?**
Tabulating `_time ComputerName user_name Group_Name` across `index=win-alert` shows local group membership enumeration targeting the `Administrators` group alongside `oliver.thompson`.

![](../../assets/Pasted%20image%2020261114144056.png)

## Task 2.4
**What is the workstation name from which the attacker logged in?**

**Answer**: `DEV-QA-SERVER`

**Why?**
First, checking failed logons (`EventCode=4625`) on `host=WIN-H015` shows failed network logon attempts (`Logon Type: 3`) for `oliver.thompson` with `Account Domain: DEV-QA-SERVER`.

![](../../assets/Pasted%20image%2020261114145032.png)

Since Event ID 4625 only proves a failed attempt, we verify whether the login was subsequently accepted by querying `EventCode=4624` with `"DEV-QA-SERVER"`. This returns two successful logon events showing `Workstation Name: DEV-QA-SERVER` and `Source Network Address: 10.10.157.155`.

![](../../assets/Pasted%20image%2020261114145127.png)

## Task 3.1
**What time did the brute-force activity using Hydra begin?**

**Answer**: `2025-09-14 21:20:27`

**Why?**
In the web server investigation (`index=web-alert`), we filter for requests from `171.251.232.40` with `useragent="Mozilla/5.0 (Hydra)"` and sort in ascending chronological order (`sort + _time`). The earliest request targeting `/wp-login.php` occurs at `2025-09-14 21:20:27`.

![](../../assets/Pasted%20image%2020261114145456.png)

## Task 3.2
**How many `POST` login requests were made using Hydra?**

**Answer**: `158`

**Why?**
Adding `method=POST` to the Hydra query (`index=web-alert 171.251.232.40 useragent="Mozilla/5.0 (Hydra)" method=POST | sort + _time`) isolates the actual credential submission requests, returning `158` events out of the `316` total Hydra requests.

![](../../assets/Pasted%20image%2020261114145735.png)

## Task 3.3
**Which User-Agent was used by the attacker when interacting with the web shell?**

**Answer**: `Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/138.0.0.0 Safari/537.36`

**Why?**
Querying all traffic from `171.251.232.40` (`index=web-alert 171.251.232.40`) reveals `340` total events, including requests referencing the `b374k.php` web shell inside the WordPress theme editor.

![](../../assets/Pasted%20image%2020261114145834.png)

Inspecting the `useragent` field summary shows only two values: `Mozilla/5.0 (Hydra)` (`316` events) and the attacker's real browser User-Agent (`24` events): `Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/138.0.0.0 Safari/537.36`.

![](../../assets/Pasted%20image%2020261114145821.png)

Tabulating those `24` browser events confirms this User-Agent was used to log in to WordPress and interact with the uploaded web shell.

![](../../assets/Pasted%20image%2020261114150145.png)

## Task 3.4
**How many requests were made to the web shell (or with the login referer)?**

**Answer**: `4`

**Why?**
Filtering the attacker's browser traffic reveals `4` matching requests in the session.

![](../../assets/Pasted%20image%2020261114150214.png)