---
title: Linux Logging for SOC Walkthrough
description: Explore authentication, runtime, and system logs on Linux for SOC investigations.
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

In this room, I explored the core log sources on Linux systems and how to query them during a SOC investigation. First, I worked with standard plain-text logs inside `/var/log/`, using `cat` and `grep` on `syslog` to check NTP time synchronization and kernel security module messages (`Yama`). Next, I analyzed `/var/log/auth.log` to uncover SSH brute-force attempts (`10.14.94.82`) and track the creation and privilege escalation of a backdoor user (`xerxes` added to the `sudo` group). After checking package installation records in `/var/log/dpkg.log` and inspecting `/root/.bash_history`, I reviewed how Linux system calls (`execve`) work and used `ausearch` to hunt through `auditd` logs (`/var/log/audit/audit.log`) for sensitive file access (`file_thmsecret`), tool downloads via `wget` (`proc_wget`), and internal network port scanning with `naabu`.

---

**Learning Objectives**

- Explore authentication, runtime, and system logs on Linux
- Learn the commands and pitfalls when working with logs
- Uncover how tools like auditd monitor and report events
- Practice every learned log source in the attached virtual machine

---
## Task 1.1
Which time server domain did the VM contact to sync its time?

**Answer**: `ntp.ubuntu.com`

**Why?**
Navigating to `/var/log` and filtering `syslog` for time synchronization messages with `cat syslog | grep -iE "syn"` shows `systemd-timesyncd` contacting the time server `185.125.190.58:123 (ntp.ubuntu.com)`.

![](../../assets/Pasted%20image%2020261109160427.png)
![](../../assets/Pasted%20image%2020261109160420.png)

## Task 1.2
What is the kernel message from Yama in `/var/log/syslog`?

**Answer**: `Yama: becoming mindful.`

**Why?**
Running `cat syslog | grep -iE "Yama"` in `/var/log` filters the kernel logs for the Yama Linux Security Module (LSM), showing the message `Yama: becoming mindful.`.

![](../../assets/Pasted%20image%2020261109160523.png)

## Task 2.1
Which IP address failed to log in on multiple users via SSH?

**Answer**: `10.14.94.82`

**Why?**
Filtering `/var/log/auth.log` for failed SSH logins using `cat auth.log | grep -iE "failed" | grep -iE "sshd"` reveals repeated `Failed password` events for `root`, `admin`, and `support` coming from `10.14.94.82`.

![](../../assets/Pasted%20image%2020261109161008.png)

## Task 2.2
Which user was created and added to the "sudo" group?

**Answer**: `xerxes`

**Why?**
Searching `/var/log/auth.log` for user creation and group modification events with `cat auth.log | grep -iE "useradd|usermod"` shows `useradd` creating a new user named `xerxes` (`UID=1001`) followed by `usermod` adding `xerxes` to the `sudo` group.

![](../../assets/Pasted%20image%2020261109161111.png)

## Task 3.1
According to the VM's package manager logs, which version of unzip was installed?

**Answer**: `6.0-28ubuntu4.1`

**Why?**
Checking `/var/log/dpkg.log` with `cat dpkg.log | grep -iE "unzip"` shows the `install` and `status installed` entries for `unzip:amd64` with version `6.0-28ubuntu4.1`.

![](../../assets/Pasted%20image%2020261109161612.png)

## Task 3.2
What is the flag you see in one of the users' bash history?

**Answer**: `THM{note_to_remember}`

**Why?**
Checking the `ubuntu` user's `.bash_history` shows `sudo su` commands, meaning commands were executed as `root`. Switching to `root` and running `history` (or checking `/root/.bash_history`) reveals line 4: `echo "THM{note_to_remember}" >> notes.txt`.

![](../../assets/Pasted%20image%2020261109161858.png)

![](../../assets/Pasted%20image%2020261109161954.png)

## Task 4.1
Which Linux system call is commonly used to execute a program?

**Answer**: `execve`

**Why?**
As shown in the **System Calls** diagram, when we run a program like `whoami` in the shell, the shell requests the Linux kernel to start `/usr/bin/whoami` by sending the `execve` system call.

![](../../assets/Pasted%20image%2020261109163917.png)

## Task 4.2
Can a typical program open a file or create a process bypassing system calls? (Yea/Nay)

**Answer**: `Nay`

**Why?**
As explained at the bottom of the **System Calls** section, user-space programs must go through the kernel via system calls to access hardware, open files, or create processes, so there is nearly no way for attackers to bypass system calls.

![](../../assets/Pasted%20image%2020261109163917.png)

## Task 5.1
When was the `secret.thm` file opened for the first time? (MM/DD/YY HH:MM:SS)

**Answer**: `08/13/25 18:36:54`

**Why?**
Checking `/etc/audit/rules.d/audit.rules` shows that access to `/secret.thm` is monitored under `-F key=file_thmsecret`. Running `ausearch -i -k file_thmsecret | grep -iE "open"` reveals the `openat` syscall (`proctitle=cat /secret.thm`) at `08/13/25 18:36:54`.

![](../../assets/Pasted%20image%2020261109164952.png)

![](../../assets/Pasted%20image%2020261109165541.png)

## Task 5.2
What is the original file name downloaded from GitHub via wget?

**Answer**: `naabu_2.3.5_linux_amd64.zip`

**Why?**
Querying the `proc_wget` audit rule key with `ausearch -i -k proc_wget | grep -iE "github"` shows the command `wget https://github.com/projectdiscovery/naabu/releases/download/v2.3.5/naabu_2.3.5_linux_amd64.zip -O /tmp/naabu.zip`, so the original file name on GitHub is `naabu_2.3.5_linux_amd64.zip`.

![](../../assets/Pasted%20image%2020261109165828.png)

## Task 5.3
Which network range was scanned using the downloaded tool?

**Answer**: `192.168.50.0/24`

**Why?**
Searching the audit logs for `naabu` with `ausearch -i | grep -iE "naabu"` shows the attacker unzipping `naabu.zip`, running `chmod +x naabu`, and executing `./naabu -host 192.168.50.0/24 -top-ports 100`, targeting the `192.168.50.0/24` subnet.

![](../../assets/Pasted%20image%2020261109170517.png)
