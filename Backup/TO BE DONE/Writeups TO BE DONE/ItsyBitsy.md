---
title: ItsyBitsy
description: Put your ELK knowledge together and investigate an incident.
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

In this challenge room, we investigate a potential Command and Control (C2) communication alert inside Elastic SIEM (Kibana). By analyzing HTTP connection logs (`connection_logs`) from March 2022, we identify a compromised host (`192.166.65.54`) abusing the Windows `bitsadmin` utility to fetch a payload hosted on `pastebin.com/yTg0Ah6a`.

---

## Task 1.1
How many events were returned for the month of March 2022?

**Answer**: `1482` (or `1,482`)

**Why?**
Opening Kibana Discover on the `connection_logs` index and setting the time filter to cover March 2022 returns `1,482 hits`.

![](../../assets/Pasted%20image%2020261114193856.png)

## Task 1.2
What is the IP associated with the suspected user in the logs?

**Answer**: `192.166.65.54`

**Why?**
Checking the `source_ip` field values shows almost all normal browser traffic originating from `192.166.65.52`, while `192.166.65.54` only has `2 hits`. Filtering by `source_ip: 192.166.65.54` reveals two suspicious HTTP requests (`HEAD` and `GET`) made on `Mar 10, 2022 @ 13:23:11`.

![](../../assets/Pasted%20image%2020261114194413.png)

## Task 1.3
The user's machine used a legit windows binary to download a file from the C2 server. What is the name of the binary?

**Answer**: `bitsadmin`

**Why?**
Expanding the document details for `source_ip: 192.166.65.54` shows `user_agent: bitsadmin`, indicating that the Windows Background Intelligent Transfer Service utility (`bitsadmin`) was used to download the file.

![](../../assets/Pasted%20image%2020261114194520.png)

## Task 1.4
The infected machine connected with a famous filesharing site in this period, which also acts as a C2 server used by the malware authors to communicate. What is the name of the filesharing site?

**Answer**: `pastebin.com`

**Why?**
In the expanded connection log table, the `host` field shows `pastebin.com` (`destination_ip: 104.23.99.190` on `destination_port: 80`).

![](../../assets/Pasted%20image%2020261114194357.png)

## Task 2.1
What is the full URL of the C2 to which the infected host is connected?

**Answer**: `pastebin.com/yTg0Ah6a`

**Why?**
Combining the `host` field (`pastebin.com`) with the `uri` field (`/yTg0Ah6a`) gives the full URL `pastebin.com/yTg0Ah6a`.

![](../../assets/Pasted%20image%2020261114194547.png)

## Task 2.2
A file was accessed on the filesharing site. What is the name of the file accessed?

**Answer**: `secret.txt`

**Why?**
Navigating to `https://pastebin.com/yTg0Ah6a` in a browser shows the paste title at the top left: `secret.txt`.

![](../../assets/Pasted%20image%2020261114195324.png)

## Task 2.3
The file contains a secret code with the format THM{_____}.

**Answer**: `THM{SECRET__CODE}`

**Why?**
Reading line 1 of `secret.txt` on Pastebin reveals the flag `THM{SECRET__CODE}`.

![](../../assets/Pasted%20image%2020261114195228.png)
