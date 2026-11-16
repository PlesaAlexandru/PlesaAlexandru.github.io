---
title: Web Security Essentials Walkthrough
description: Learn how the web works, common website security risks, and protections for a safer internet.
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

In this room we went over the basics of how web applications work, the three main layers of web infrastructure (Web Application, Web Server, and Host Machine), and the security controls used to protect each layer, including WAFs, CDNs, input validation, and system hardening.

---

**Learning Objectives**

- Understand the shift from desktop applications to web applications
- Learn why web applications are common targets for attackers
- Explore web infrastructure and the tools we use to protect the web
- Practice applying security measures to harden a new web application




---
## Task 1.1

**Have applications shifted from desktop to web over the past couple of decades (Yea/Nay)?**  
**Answer**: `Yea`

**Why?**
As shown in the timeline, applications moved from standalone desktop installs in the 1990s to browser-based SaaS platforms over the 2000s and 2010s.

![](../../assets/Pasted%20image%2020261105161103.png)
## Task 1.2

**Who is ultimately responsible for ensuring the security of users' data within a web application?**  
**Answer**: `Web App Owner`

**Why?**
Under the "As a Web App Owner" section, the room states that when you run a web application, you are responsible for securing your users' data.
![](../../assets/Pasted%20image%2020261105161347.png)




## Task 2.1

**What does your web browser send to a server to receive a web page?**  
**Answer**: `Request`

**Why?**
When you visit a website, your browser sends an HTTP request to the web server, which processes it and sends back a response.

![](../../assets/Pasted%20image%2020261105161915.png)

## Task 2.2

**What web server is most commonly used to host WordPress websites?**  
**Answer**: `Apache`

**Why?**
Looking at the web servers list, Apache is described as the most popular web server for hosting simple websites and blogs, most commonly WordPress.
![](../../assets/Pasted%20image%2020261105162013.png)


## Task 2.3

**What do we call the OS and environment that runs the web server and application?**  
**Answer**: `Host Machine`

**Why?**
The three main components of web infrastructure are the Application, the Web Server, and the Host Machine (the underlying Linux or Windows OS that runs both).

![](../../assets/Pasted%20image%2020261105162053.png)

## Task 3.1

**What cyber security concept involves stopping or limiting damage from threats?**  
**Answer**: `Mitigation`

**Why?**
In the Best Practices section, actively stopping or limiting an attack to reduce its impact is defined as mitigation.
![](../../assets/Pasted%20image%2020261105163309.png)
## Task 3.2

**What security control involves ensuring all software and components are up to date?**  
**Answer**: `Patch Management`

**Why?**
Under "Security Tips for All Three Components", keeping your app dependencies, web server, and host machine up to date is covered under Patch Management.
![](../../assets/Pasted%20image%2020261105165637.png)
## Task 4.1

**Which type of Web Application Firewall operates by running on the same system as the application itself?**  
**Answer**: `Host-Based`

**Why?**
A Host-based WAF runs directly on the web server as a software module or plugin, meaning it shares the same system resources as the application.

![](../../assets/Pasted%20image%2020261105185738.png)
## Task 4.2

**Which common WAF detection technique works by matching incoming requests against known malicious patterns?**  
**Answer**: `Signature-Based`

**Why?**
Looking at the WAF features table, Signature-Based Detection checks incoming traffic against a database of known attack patterns (like common SQLi or XSS strings).
![](../../assets/Pasted%20image%2020261105185906.png)
## Task 5.1

**What flag did you receive for securing the Web Application?**  
**Answer**: `THM{web_app_secured!}`

**Why?**
In the first stage of the interactive game ("Secure Your Web App"), we select the three controls that belong at the application layer: `Secure Coding`, `Input Validation & Sanitization`, and `Access Control`.

![](../../assets/Pasted%20image%2020261105185944.png)

![](../../assets/Pasted%20image%2020261105190442.png)

![](../../assets/Pasted%20image%2020261105190521.png)





## Task 5.2

**What flag did you receive for securing the Web Server?**  
**Answer**: `THM{server_security_expert!}`

**Why?**
In the second stage ("Secure Your Web Server"), we pick the three controls that protect and monitor the web server layer: `Web Application Firewall`, `Content Delivery Network`, and `Access Logging`.

![](../../assets/Pasted%20image%2020261105190544.png)

![](../../assets/Pasted%20image%2020261105190559.png)

![](../../assets/Pasted%20image%2020261105190615.png)




## Task 5.3

**What flag did you receive for securing the Host Machine?**  
**Answer**: `THM{the_final_security_layer!}`

**Why?**
In the final stage ("Secure Your Host Machine"), we apply the OS-level protections: `System Hardening`, `Antivirus`, and `Least Privilege` to get the last flag.

![](../../assets/Pasted%20image%2020261105191100.png)

![](../../assets/Pasted%20image%2020261105191121.png)

![](../../assets/Pasted%20image%2020261105191147.png)
