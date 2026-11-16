---
title: Network Discovery Detection Walkthrough
description: Understand how attackers discover assets in a network, and how to detect that activity.
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

In this room we looked at how attackers scan a network to find active hosts, open ports, and services. We worked with CSV logs and Zeek connection logs in Kibana to tell the difference between internal and external scans, horizontal vs vertical scans, ping sweeps, and TCP SYN scans. Overall, a very useful lesson for understanding how recon looks in the logs before an actual attack happens.

---

**Learning Objectives**

- What is network discovery
- Why attackers perform network discovery
- What are the different types of network discovery
- How network discovery techniques work, and how we can detect them


---
## Task 1.1

**What do attackers scan, other than, IP addresses, ports, and OS version, in order to identify vulnerabilities in a network?**  
**Answer**: `Services`

**Why?**
In the **Attackers and Network Discovery** section, it mentions that besides IP addresses, ports, and OS versions, attackers also scan for the `Services` running on those hosts to find vulnerabilities.
![](../../assets/Pasted%20image%2020261103193912.png)

## Task 1.2

**Which file contains logs that showcase internal scanning activity?**  
**Answer**: `log-session-2.csv`

**Why?**
Opening the CSV files in LibreOffice Calc, we can see that `log-session-2.csv` has both `source.ip` (`192.168.230.127`) and `destination.ip` (`192.168.230.145` and `192.168.230.1`) in the same private `192.168.230.x` subnet.
![](../../assets/Pasted%20image%2020261103194509.png)

## Task 1.3

**How many log entries are present for the internal IP performing internal scanning activity?**  
**Answer**: `2276`

**Why?**
We can count how many times the internal source IP shows up in `log-session-2.csv` using the terminal:
`cat log-session-2.csv | cut -d "," -f3 | uniq -c`
Ignoring the header row, it gives us `2276` entries for `192.168.230.127`.
![](../../assets/Pasted%20image%2020261103194750.png)

## Task 1.4

**What is the external IP address that is performing external scanning activity?**  
**Answer**: `203.0.113.25`

**Why?**
Checking `log-session-1.csv` in LibreOffice Calc (or running `cat log-session-1.csv | cut -d "," -f3 | uniq -c` in the terminal) shows `2014` entries coming from the external IP `203.0.113.25` scanning `192.168.230.145`.
![](../../assets/Pasted%20image%2020261103195202.png)

![](../../assets/Pasted%20image%2020261103194919.png)

## Task 1.5

**One of the log files contains evidence of a horizontal scan. Which IP range was scanned? Format X.X.X.X/X**  
**Answer**: `203.0.113.0/24`

**Why?**
Scrolling through `log-session-2.csv`, we can see `192.168.230.127` pinging every IP across the `203.0.113.x` range from `.1` all the way to `.254`. Since it covers all 254 usable hosts in that subnet, the range is `203.0.113.0/24`.
![](../../assets/Pasted%20image%2020261103200123.png)

## Task 2.1

**In the same log file, there is one IP address on which a vertical scan is performed. Which IP address is this?**  
**Answer**: `192.168.230.145`

**Why?**  
A vertical scan hits a single IP across many different ports. In `log-session-2.csv`, we can see `192.168.230.145` being scanned on a bunch of different destination ports (`419`, `532`, `752`, `900`, `961`, `192`, `404`, `451`, etc.).
![](../../assets/Pasted%20image%2020261103200716.png)

## Task 2.2

**On one of the IP addresses, only a few ports are scanned which host common services. Which are the ports that are scanned on this IP address? Format: port1, port2, port3 in ascending order.**  
**Answer**: `80, 445, 3389`

**Why?**
Looking at the lines for `192.168.230.1` in `log-session-2.csv`, only three common ports are scanned: `80` (HTTP), `445` (SMB), and `3389` (RDP).
![](../../assets/Pasted%20image%2020261103201005.png)

## Task 2.3

**Which source IP performs a ping sweep attack across a whole subnet?**  
**Answer**: `192.168.230.127`

**Why?**
In Kibana Discover (and just like we saw earlier in `log-session-2.csv`), checking `source.ip` shows `192.168.230.127` sending all the ICMP echo requests across the `/24` subnet.
![](../../assets/Pasted%20image%2020261103202103.png)

## Task 3.1

**The zeek.conn.conn_state value shows the connection state. Using the information provided by this value, identify the type of scan being performed by 203.0.113.25 against 192.168.230.145**  
**Answer**: `TCP SYN scan`

**Why?**
From the lesson notes, a `TCP SYN scan` sends a `SYN` packet to check if a port is open without finishing the handshake.
![](../../assets/Pasted%20image%2020261103202534.png)

Filtering in Kibana for `destination.ip.keyword: 192.168.230.145`, `source.ip.keyword: 203.0.113.25`, and `zeek.conn.conn_state.keyword: S0` gives `2,003` hits where only a `SYN` packet was sent (`conn_state: S0`), which confirms a `TCP SYN scan`.
![](../../assets/Pasted%20image%2020261103202515.png)

## Task 3.2

**Is there any UDP scanning attempt in the logs? Y/N**  
**Answer**: `N`

**Why?**
Looking at the `network.protocol` field in Kibana shows `tcp` (`94.0%`), `icmp` (`6.0%`), and `udp` (`0.0%`).
![](../../assets/Pasted%20image%2020261103202719.png)

If we filter by `network.protocol.keyword: udp`, there are only `2` multicast packets going to `239.255.255.250`, so there is no UDP scan (`N`).
![](../../assets/Pasted%20image%2020261103202638.png)


