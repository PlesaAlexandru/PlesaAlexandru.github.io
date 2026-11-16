---
title: Detecting Web Attacks Walkthrough
description: Explore web attacks and detection methods through log and network traffic analysis.
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

In this room we looked at the difference between client-side and server-side web attacks and practiced catching them in both web server access logs and network packet captures (PCAPs). Using `access.log`, CyberChef, and Wireshark, we traced an attack chain involving `ffuf` directory fuzzing, a Hydra brute-force login on `/login.php`, and a `sqlmap` SQL injection dump on `/account/changeusername.php`, before looking at how WAF rules can block these requests.

---

**Learning Objectives**

- Learn common client-side and server-side attack types
- Understand the benefits and limitations of log-based detection
- Explore network traffic-based detection methods
- Understand how and why Web Application Firewalls are used
- Practice identifying common web attacks using the methods covered



---
## Task 1.1

**What class of attacks relies on exploiting the user's behavior or device?**  
**Answer**: `Client-Side`

**Why?**
As highlighted in the text, client-side attacks rely on abusing weaknesses in user behavior or on the user's device, such as browser vulnerabilities or tricking users into unsafe actions.
![](../../assets/Pasted%20image%2020261105195528.png)
## Task 1.2

**What is the most common client-side attack?**  
**Answer**: `XSS`

**Why?**
Under "Common Client-Side Attacks", Cross-Site Scripting (`XSS`) is listed as the most common client-side attack, where malicious scripts run inside a victim's browser.
![](../../assets/Pasted%20image%2020261105195543.png)
## Task 2.1

**What class of attacks relies on exploiting vulnerabilities within web servers?**  
**Answer**: `Server-Side`

**Why?**
Server-side attacks target the web application, server, or backend database directly, leaving a trail of evidence in server logs and network traffic.

![](../../assets/Pasted%20image%2020261105232256.png)
## Task 2.2

**Which server-side attack lets attackers abuse forms to dump database contents?**  
**Answer**: `SQLi`

**Why?**
SQL Injection (`SQLi`) targets the database behind a website when user input is concatenated directly into SQL queries, allowing attackers to alter the query and dump or modify data.

![](../../assets/Pasted%20image%2020261105232342.png)
## Task 3.1

**What is the attacker's User-Agent while performing the directory fuzz?**  
**Answer**: `FFUF v2.1.0`

**Why?**
Opening `access.log`, the first 13 requests (`GET /social`, `/images`, `/manage`, `/adminpanel`, etc.) all show `"FFUF v2.1.0"` as the User-Agent string.

![](../../assets/Pasted%20image%2020261105232626.png)

## Task 3.2

**What is the name of the page on which the attacker performs a brute-force attack?**  
**Answer**: `/login.php`

**Why?**
Right after the directory fuzzing in `access.log`, we see a flood of automated `GET` and `POST` requests targeting `/login.php` with the User-Agent `"Mozilla/5.0 (Hydra)"`.

![](../../assets/Pasted%20image%2020261105232729.png)
## Task 3.3

**What is the complete, decoded SQLi payload the attacker uses on the /changeusername.php form?**  
**Answer**: `%' OR '1'='1`

**Why?**
At line 38 of `access.log`, we see a `sqlmap/stable` request: `GET /account/changeusername.php?q=%25%27+OR+%271%27%3D%271`. Putting that URL-encoded parameter into CyberChef with `URL Decode` (and `Treat "+" as space` enabled) decodes it to `%' OR '1'='1`.
![](../../assets/Pasted%20image%2020261105232751.png)

![](../../assets/Pasted%20image%2020261105232956.png)
## Task 4.1

**What password does the attacker successfully identify in the brute-force attack?**  
**Answer**: `astrongpassword123`

**Why?**
In `access.log` (line 33), we saw the successful login returned an HTTP `302` redirect. Filtering `traffic.pcap` in Wireshark with `http && http.response.code == 302` gives us packet `327`, and following the HTTP stream (`tcp.stream eq 32`) shows the `POST /login.php` body: `username=admin&password=astrongpassword123`.

![](../../assets/Pasted%20image%2020261105234059.png)

![](../../assets/Pasted%20image%2020261105234035.png)


## Task 4.2

**What is the flag the attacker found in the database using SQLi?**  
**Answer**: `THM{dumped_the_db}`

**Why?**
Filtering by `http` in Wireshark and inspecting packet `442` (the `200 OK` response to `GET /account/changeusername.php?q=%25%27+OR+%271%27%3D%271` in frame `440`), we can read the dumped `users` table right in the HTML response body: `4 | flag | THM{dumped_the_db}`.

![](../../assets/Pasted%20image%2020261105235233.png)



## Task 5.1

**What do WAFs inspect and filter?**  
**Answer**: `Web Requests`

**Why?**
WAFs act as gatekeepers in front of web applications by inspecting and filtering full incoming web request packets before they reach the server.

![](../../assets/Pasted%20image%2020261106001747.png)

## Task 5.2

**Create a custom firewall rule to block any User-Agent that matches "BotTHM".**  
**Answer**: `If User-Agent contains "BotTHM" then BLOCK`

**Why?**
Following the WAF rule example from the task (`If User-Agent contains "sqlmap" then BLOCK`), we replace `"sqlmap"` with `"BotTHM"` to block requests with that User-Agent.

![](../../assets/Pasted%20image%2020261106002055.png)