---
title: Network Security Essentials Walkthrough
description: Learn about key aspects of network security essentials and how to monitor and protect against adversaries.
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

In this room we went over the basics of network perimeter security and how tools like firewalls, WAFs, VPNs, and IDS work together to protect an internal network. We checked raw log files first and then used Splunk to follow a full attack chain from port scanning and VPN brute-forcing to SMB lateral movement, C2 beaconing, and data exfiltration.

---

**Learning Objectives**

- Understand what a network is and identify its key components.
- Explore the concept of the network perimeter and its importance.
- Identify the key perimeter threats.
- Examine the firewall logs to monitor normal and suspicious logs.


---
## Task 1.1

**Examine the firewall logs. Which IP address is performing the port scan?**  
**Answer**: `203.0.113.10`

**Why?**
Looking at `firewall_logs.txt`, we can see a bunch of `BLOCK TCP` lines coming from `203.0.113.10` hitting `10.0.0.20` on different ports (`135`, `139`, `443`, `445`, `1433`, `3306`, `3389`, `5900`, `8080`), which shows a port scan.
![](../../assets/Pasted%20image%2020261103190004.png)

## Task 1.2

**In the WAF Logs, which single source IP is responsible for all the blocked web attacks?**  
**Answer**: `198.51.100.12`

**Why?**
If we search for `action=BLOCK` in `waf_logs.txt`, all the blocked web attacks (like XSS `<script>alert('XSS')</script>`) come from the same IP: `198.51.100.12`.
![](../../assets/Pasted%20image%2020261103190105.png)

## Task 1.3

**In the VPN logs, how many brute-force attempts failed?**  
**Answer**: `90`

**Why?**
In `vpn_logs.txt`, there are `135` lines in total and 2 out of every 3 lines are `FAILED_AUTH` (`135 - 135 / 3 = 90`). Searching for `FAILED_AUTH` in the file also gives `90` matches.
![](../../assets/Pasted%20image%2020261103190306.png)

## Task 1.4

**Which suspicious IP address was found attempting the brute-force attack against the VPN gateway?**  
**Answer**: `45.137.22.13`

**Why?**
Looking at the `FAILED_AUTH` lines in `vpn_logs.txt`, all the failed login attempts (`admin`, `administrator`, `root`, `test`, `toor`) come from `45.137.22.13`.
![](../../assets/Pasted%20image%2020261103190440.png)

## Task 1.5

**Examine the firewall logs. What external IP performed the most reconnaissance?**  
**Answer**: `203.0.113.45`

**Why?**
In Splunk, we filter for blocked firewall traffic with:
`index="network_logs" sourcetype=firewall_logs action=BLOCK`
Clicking on `src_ip` on the left panel shows `203.0.113.45` at the top with `279` blocked events (`79.487%`).
![](../../assets/Pasted%20image%2020261103191303.png)

## Task 2.1

**In the firewall log, which internal host was targeted by scans?**  
**Answer**: `10.0.0.20`

**Why?**  
Using the same search (`index="network_logs" sourcetype=firewall_logs action=BLOCK`), we click on `dst_ip` and see `10.0.0.20` is the main target with `145` events.
![](../../assets/Pasted%20image%2020261103191357.png)

## Task 2.2

**Which username was targeted in VPN logs?**  
**Answer**: `svc_backup`

**Why?**
We search the VPN logs in Splunk with:
`index="network_logs" sourcetype=vpn_logs`
![](../../assets/Pasted%20image%2020261103191501.png)

Checking the `username` field on the left shows `svc_backup` with `135` events (`61.927%`), way more than normal users like `jsmith` or `bob`.
![](../../assets/Pasted%20image%2020261103191457.png)

## Task 2.3

**What internal IP was assigned after successful VPN login?**  
**Answer**: `10.8.0.23`

**Why?**
We add `result=SUCCESS username=svc_backup` to the search (`index="network_logs" sourcetype=vpn_logs result=SUCCESS username=svc_backup`) and check `assigned_ip`, which gives us `10.8.0.23`.
![](../../assets/Pasted%20image%2020261103191725.png)

## Task 3.1

**Which port was used for lateral SMB attempts?**  
**Answer**: `445`

**Why?**
Searching the firewall logs for traffic coming from the VPN IP (`src_ip="10.8.0.23"`) shows `ALLOW` connections to internal hosts like `10.0.0.51` on `dst_port: 445` (SMB).
![](../../assets/Pasted%20image%2020261103192254.png)

## Task 3.2

**In the IDS logs, which host beaconed to the C2?**  
**Answer**: `10.0.0.60`

**Why?**
Filtering the IDS logs for the C2 alert (`index="network_logs" sourcetype=ids_logs alert="ET TROJAN Possible C2 Beaconing"`) gives `80` events, all coming from `src_ip: 10.0.0.60`.
![](../../assets/Pasted%20image%2020261103192504.png)

## Task 3.3

**During the investigation, which IP was observed to be associated with C2?**  
**Answer**: `198.51.100.77`

**Why?**
In the same `ET TROJAN Possible C2 Beaconing` events, looking at `dst_ip` shows the external C2 server IP is `198.51.100.77` on port `4444`.
![](../../assets/Pasted%20image%2020261103192643.png)

## Task 3.4

**Which host showed the exfiltration attempts?**  
**Answer**: `10.0.0.51`

**Why?**
Filtering the IDS logs with `index="network_logs" sourcetype=ids_logs classification="Potential Data Exfiltration"` returns `60` alerts (`ET INFO Possible HTTP POST Large Upload`) where `src_ip` is `10.0.0.51`.
![](../../assets/Pasted%20image%2020261103192745.png)
