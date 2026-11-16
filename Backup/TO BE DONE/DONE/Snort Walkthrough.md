---
title: Snort Walkthrough
description: Learn how to use Snort to detect real-time threats, analyse recorded traffic files and identify anomalies.
pubDate: 2026-11-03
categories:
  - SOC
  - Walkthrough
  - TryHackMe
  - Network
  - Snort
tags:
  - Lessons
image: /image/networksecess.png
---
---
## Summary

In this room we went through the basics of IDS/IPS systems and practiced using Snort across its different modes: Sniffer mode, Packet Logger mode, IDS/IPS mode, and reading PCAP files. At the end, we also wrote our own custom rules in `local.rules` to catch specific IP IDs, TCP flags, and packets with the same source and destination IP.

---

**Learning Objectives**

- Overview of IDS / IPS
- Snort Overview
- Writing IDS rules
- Detecting Intrusion with Snort


---
## Task 1.1

**Navigate to the Task-Exercises folder and run the command "sudo ./.easy.sh" and write the output.**  
**Answer**: `Too Easy!`

**Why?**
Go to `~/Desktop/Task-Exercises` in the terminal and run `sudo ./.easy.sh`, which prints `Too Easy!`.
![](../../assets/Pasted%20image%2020261104152054.png)

## Task 1.2

**Which IDS/IPS type can help you to stop the threats on a local machine?**  
**Answer**: `HIPS`

**Why?**
**Host-based Intrusion Prevention System (HIPS)** protects traffic on a single endpoint device and stops the connection if a signature is matched.
![](../../assets/Pasted%20image%2020261104154517.png)
## Task 1.3

**Which IDS/IPS type can help you to detect the threats on a local network?**  
**Answer**: `NIDS`

**Why?**
**Network Intrusion Detection System (NIDS)** monitors traffic across the whole network subnet and creates an alert when a signature is identified.
![](../../assets/Pasted%20image%2020261104154601.png)

## Task 1.4

**Which IDS/IPS type can help you to detect the threats on a local machine?**  
**Answer**: `HIDS`

**Why?**
**Host-based Intrusion Detection System (HIDS)** monitors traffic on a single endpoint device and generates an alert if a signature is matched.
![](../../assets/Pasted%20image%2020261104154629.png)
## Task 1.5

**Which IDS/IPS type can help you to stop the threats on a local network?**  
**Answer**: `NIPS`

**Why?**
**Network Intrusion Prevention System (NIPS)** monitors traffic across the whole subnet and terminates the connection if a signature is identified.
![](../../assets/Pasted%20image%2020261104154647.png)
## Task 2.1

**Which IDS/IPS type works by finding anomalies in the network?**  
**Answer**: `NBA`

**Why?**  
**Behaviour-based Intrusion Prevention System (Network Behaviour Analysis - NBA)** monitors traffic across the network and terminates the connection when an anomaly is found.
![](../../assets/Pasted%20image%2020261104154859.png)
## Task 2.2

**According to the official description, what kind of NIPS is it?**  
**Answer**: `full-blown`

**Why?**
From the official Snort quote in the lesson, Snort can be used as a packet sniffer, a packet logger, or a `full-blown` network intrusion prevention system.
![](../../assets/Pasted%20image%2020261104154930.png)
## Task 2.3

**NBA training period is also known as ...**  
**Answer**: `baselining`

**Why?**
Behaviour-based systems need a training period, also called `"baselining"`, to learn what normal traffic looks like so they can spot malicious anomalies.
![](../../assets/Pasted%20image%2020261104154953.png)
## Task 3.1

**Run the Snort instance and check the build number.**  
**Answer**: `149`

**Why?**
Running `snort -V` prints the version info: `Version 2.9.7.0 GRE (Build 149)`.
![](../../assets/Pasted%20image%2020261104155400.png)
## Task 3.2

**Test the current instance with "/etc/snort/snort.conf" file and check how many rules are loaded with the current build.**  
**Answer**: `4151`

**Why?**
We test the config file using `sudo snort -c /etc/snort/snort.conf -T`.
![](../../assets/Pasted%20image%2020261104155749.png)
Scrolling to the rule summary shows `4151 Snort rules read`.
![](../../assets/Pasted%20image%2020261104155708.png)
## Task 3.3

**Test the current instance with "/etc/snort/snortv2.conf" file and check how many rules are loaded with the current build.**  
**Answer**: `1`

**Why?**
We run `sudo snort -c /etc/snort/snortv2.conf -T` to test `snortv2.conf`.
![](../../assets/Pasted%20image%2020261104155833.png)

Under `Initializing rule chains...` it shows `1 Snort rules read`.
![](../../assets/Pasted%20image%2020261104155856.png)

## Task 4.1

**Navigate to the TASK-6 folder. Use the given commands to create a log file in ASCII mode and feed it with the "traffic-generator.sh" file. What is the source port used to connect port 53?**  
**Answer**: `3009`

**Why?**
After starting Snort in ASCII logger mode (`sudo snort -dev -K ASCII -l .`) inside `TASK-6` and running `sudo ./traffic-generator.sh`, we can see the DNS request `145.254.160.237.3009 > 145.253.2.203.53`, so the source port connecting to port `53` is `3009`.
![](../../assets/Pasted%20image%2020261104162212.png)
## Task 4.2

**Read the snort.log file with Snort; what is the IP ID of the 10th packet?**  
**Answer**: `49313`

**Why?**
In `TASK-6`, we read the first 10 packets from the log file with `snort -r snort.log.1640048004 -n 10`.
![](../../assets/Pasted%20image%2020261104162322.png)

Looking at the 10th packet (`65.208.228.223:80 -> 145.254.160.237:3372`) gives `ID:49313`.
![](../../assets/Pasted%20image%2020261104162418.png)

## Task 4.3

**Read the "snort.log.1640048004" file with Snort; what is the referer of the 4th packet?**  
**Answer**: `http://www.ethereal.com/development.html`

**Why?**
We run `snort -r snort.log.1640048004 -n 4 -X` to see the hex and ASCII dump of the 4th packet.
![](../../assets/Pasted%20image%2020261104162805.png)

At the bottom right of the ASCII output, we can see `Referer: http://www.ethereal.com/development.html`.
![](../../assets/Pasted%20image%2020261104162946.png)
## Task 4.4

**Read the "snort.log.1640048004" file with Snort; what is the Ack number of the 8th packet?**  
**Answer**: `0x38AFFFF3`

**Why?**
We read the first 8 packets with `snort -r snort.log.1640048004 -n 8`.
![](../../assets/Pasted%20image%2020261104163133.png)

In the 8th packet details, the Ack number is `Ack: 0x38AFFFF3`.
![](../../assets/Pasted%20image%2020261104163113.png)

## Task 4.5

**Read the "snort.log.1640048004" file with Snort; what is the number of the "TCP port 80" packets?**  
**Answer**: `41`

**Why?**
We filter the log file for TCP port 80 with `snort -r snort.log.1640048004 'tcp and port 80'`.
![](../../assets/Pasted%20image%2020261104163324.png)

At the bottom of the breakdown summary, `Total:` is `41`.
![](../../assets/Pasted%20image%2020261104163312.png)
## Task 5.1

**Navigate to the TASK-7 folder. Use the "/etc/snort/snort.conf" file and run Snort in full alert mode, then feed it with the "TASK-7 Exercise" in the "traffic-generator.sh" file. Wait until the traffic stops, then stop the Snort instance. Now analyse the output summary; what is the number of the detected HTTP GET methods?**  
**Answer**: `2`

**Why?**
In `TASK-7`, we start Snort in full alert mode: `sudo snort -c /etc/snort/snort.conf -A full -l .`.
![](../../assets/Pasted%20image%2020261105011537.png)

![](../../assets/Pasted%20image%2020261105011600.png)

Then in another terminal, we run `sudo ./traffic-generator.sh` and pick `4) TASK-7 Exercise`.

![](../../assets/Pasted%20image%2020261105012544.png)

![](../../assets/Pasted%20image%2020261105012555.png)

![](../../assets/Pasted%20image%2020261105012604.png)

After stopping Snort (`Ctrl+C`), looking under `HTTP Inspect - encodings` shows `GET methods: 2`.
![](../../assets/Pasted%20image%2020261105012505.png)


## Task 6.1

**Investigate the mx-1.pcap file with the default configuration file. What is the number of the generated alerts?**  
**Answer**: `170`

**Why?**
In `TASK-8`, we run `sudo snort -c /etc/snort/snort.conf -A full -l . -r mx-1.pcap`.
![](../../assets/Pasted%20image%2020261105012746.png)

Under `Action Stats`, it shows `Alerts: 170`.
![](../../assets/Pasted%20image%2020261105012848.png)

## Task 6.2

**Keep reading the output. How many TCP Segments are Queued?**  
**Answer**: `18`

**Why?**
Under `Stream statistics` in the `mx-1.pcap` output, we see `TCP Segments Queued: 18`.


![](../../assets/Pasted%20image%2020261105012953.png)

## Task 6.3

**Keep reading the output. How many "HTTP response headers" were extracted?**  
**Answer**: `3`

**Why?**
Under `HTTP Inspect - encodings` in the same output, `HTTP response Headers extracted:` is `3`.
![](../../assets/Pasted%20image%2020261105013030.png)

## Task 6.4

**Investigate the mx-1.pcap file with the second configuration file. What is the number of the generated alerts?**  
**Answer**: `68`

**Why?**
Running `sudo snort -c /etc/snort/snortv2.conf -A full -l . -r mx-1.pcap` with `snortv2.conf` gives `Alerts: 68` under `Action Stats`.
![](../../assets/Pasted%20image%2020261105013100.png)
## Task 6.5

**Investigate the mx-2.pcap file with the default configuration file. What is the number of the generated alerts?**  
**Answer**: `340`

**Why?**
Running `sudo snort -c /etc/snort/snort.conf -A full -l . -r mx-2.pcap` gives `Alerts: 340` under `Action Stats`.
![](../../assets/Pasted%20image%2020261105013153.png)


## Task 6.6

**Keep reading the output. What is the number of the detected TCP packets?**  
**Answer**: `82`

**Why?**
In the `Breakdown by protocol` section for `mx-2.pcap`, `TCP:` is `82`.
![](../../assets/Pasted%20image%2020261105013524.png)
## Task 6.7

**Investigate the mx-2.pcap and mx-3.pcap files with the default configuration file. What is the number of the generated alerts?**  
**Answer**: `1020`

**Why?**
Running both PCAPs with `--pcap-list="mx-2.pcap mx-3.pcap"` (`sudo snort -c /etc/snort/snort.conf -A full -l . --pcap-list="mx-2.pcap mx-3.pcap"`) gives `Alerts: 1020`.
![](../../assets/Pasted%20image%2020261105013606.png)

## Task 7.1

**Use "task9.pcap". Write a rule to filter IP ID "35369" and run it against the given pcap file. What is the request name of the detected packet?**  
**Answer**: `TIMESTAMP REQUEST`

**Why?**
In `TASK-9`, we open `local.rules`, add rules to check `id:35369;` for TCP, ICMP, and UDP, and run `snort -c local.rules -A full -l . -r task9.pcap`.
![](../../assets/Pasted%20image%2020261105014422.png)

![](../../assets/Pasted%20image%2020261105014437.png)

It generates `1` alert, and opening the `alert` file shows `TIMESTAMP REQUEST`.
![](../../assets/Pasted%20image%2020261105014448.png)

![](../../assets/Pasted%20image%2020261105014410.png)

## Task 7.2

**Create a rule to filter packets with Syn flag and run it against the given pcap file. What is the number of detected packets?**  
**Answer**: `1`

**Why?**
We comment out the old rules in `local.rules`, add `alert tcp any any <> any any (msg: "SYN Request"; flags:S; sid: 100001; rev:1;)`, and run `snort -c local.rules -A full -l . -r task9.pcap`. It returns `Alerts: 1`.
![](../../assets/Pasted%20image%2020261105015128.png)

![](../../assets/Pasted%20image%2020261105015110.png)

![](../../assets/Pasted%20image%2020261105015046.png)


## Task 7.3

**Clear the previous log and alarm files and deactivate/comment out the old rule. Write a rule to filter packets with Push-Ack flags and run it against the given pcap file. What is the number of detected packets?**  
**Answer**: `216`

**Why?**
We comment out the SYN rule, add `alert tcp any any <> any any (msg: "Push-Ack Request"; flags:P,A; sid: 100001; rev:1;)`, and run Snort on `task9.pcap`, which gives `Alerts: 216`.
![](../../assets/Pasted%20image%2020261105015754.png)

![](../../assets/Pasted%20image%2020261105015559.png)

## Task 7.4

**Clear the previous log and alarm files and deactivate/comment out the old rule. Create a rule to filter packets with the same source and destination IP and run it against the given pcap file. What is the number of detected packets?**  
**Answer**: `10`

**Why?**
Testing `sameip;` on UDP first (`alert udp any any <> any any (msg: "SAME-IP TEST"; sameip; sid: 100001; rev:1;)`) catches `7` UDP packets, and adding a rule for `tcp` (or `ip`) catches the other `3` TCP packets, giving `10` packets in total.
![](../../assets/Pasted%20image%2020261105020058.png)

![](../../assets/Pasted%20image%2020261105020051.png)

## Task 7.5

**Case Example - An analyst modified an existing rule successfully. Which rule option must the analyst change after the implementation?**  
**Answer**: `rev`

**Why?**
The `rev` option tracks how many times a rule was modified (`rev:1;`), so it should be updated whenever a rule is changed.
![](../../assets/Pasted%20image%2020261105020205.png)

---

## Personal Note: Snort & Snort2 Quick Reference

### 1. Snort vs. Snort2 Configs (`snort.conf` vs `local.rules`)
- **Snort 2 (`2.9.x`)** uses `/etc/snort/snort.conf` to load **preprocessors** (`Stream5` for TCP sessions, `HTTP Inspect` for HTTP traffic) and all enabled rule files (`community.rules`, `local.rules`).
- If we only want to test our own signatures on a `.pcap` without getting thousands of default alerts, we can pass `local.rules` directly with `-c local.rules` (or a minimal config like `snortv2.conf`).

### 2. Useful CLI Flags by Mode

| Mode | Flags | Example |
| :--- | :--- | :--- |
| **General / Test** | `-V` (version), `-T` (test config), `-q` (quiet banner) | `sudo snort -c /etc/snort/snort.conf -T` |
| **1. Sniffer Mode** | `-v` (verbose), `-d` (payload), `-e` (L2 MAC), `-X` (Hex+ASCII), `-i <if>` | `sudo snort -vde -i eth0` |
| **2. Packet Logger** | `-l <dir>` (log dir), `-K ASCII` (ASCII logs), `-r <file>` (read), `-n <count>` | `sudo snort -dev -K ASCII -l .`<br>`snort -r snort.log.xxx -n 10 -X 'tcp and port 80'` |
| **3. IDS / IPS Mode** | `-c <conf>`, `-A <mode>` (`console`, `fast`, `full`, `cmg`, `none`), `-Q` (inline IPS) | `sudo snort -c /etc/snort/snort.conf -A full -l .` |
| **4. PCAP Analysis** | `-r <file.pcap>`, `--pcap-list="a.pcap b.pcap"` | `sudo snort -c local.rules -A full -l . -r target.pcap` |

### 3. Snort2 Rule Structure & Options
```text
[Action] [Proto] [Src_IP] [Src_Port] [Dir] [Dst_IP] [Dst_Port] ( [Rule Options] )
alert    tcp     any      any        <>    any      80         (msg:"Test"; flags:S; sid:100001; rev:1;)
```
- **Actions**: `alert` (alert + log), `log` (log only), `pass` (ignore), `drop` (block + log), `reject` (block + log + reset).
- **Direction (`->` vs `<>`)**: `<>` checks traffic in both directions.
- **Common Rule Options**:
  - **General**: `msg:"...";` (alert name), `sid:1000001;` (custom rules start at `1000000`), `rev:1;` (revision number), `reference:cve,XXXX;`.
  - **Payload**: `content:"GET";` (text or hex `|90 90|`), `nocase;`, `offset:X;`, `depth:Y;`, `fast_pattern;`.
  - **Non-Payload**: `id:35369;` (IP ID), `ttl:64;`, `dsize:>100;` (payload size), `sameip;` (source IP equals destination IP), `flags:S;` (`S`=SYN, `A`=ACK, `P`=PSH, `F`=FIN, `R`=RST, `0`=NULL).


