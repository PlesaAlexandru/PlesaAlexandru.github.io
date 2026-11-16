---
title: Detecting Web DDoS Walkthrough
description: Learn denial-of-service recognition patterns and protection strategies for web applications.
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

In this room we covered how application-layer (Layer 7) DoS and DDoS attacks work, the motives behind them, and how to spot them in both raw web server logs and a SIEM. We first analyzed an `access.log` file to catch a single-IP `curl` flood against `/login` causing `503` errors, then used Splunk (`index="main"`) to investigate a 60-host botnet flooding `/search` at 207 requests per second, and finished with DDoS mitigation techniques like CAPTCHAs, rate limiting, WAFs, and CDN load-balancing.

---

**Learning Objectives**

- Learn how denial-of-service attacks function
- Understand attacker motives behind disruptive attacks
- See how web logs can help you reveal signs of web DoS and DDoS
- Get practice analyzing denial-of-service attacks through hands-on log analysis
- Discover detection and mitigation techniques that defenders can use

---
## Task 1.1

**What class of attack relies on disrupting the availability of a web service?**  
**Answer**: `Denial-of-Service`

**Why?**
Denial-of-Service (`DoS`) attacks are designed to overwhelm a website or application with excessive traffic so legitimate users cannot access it.

![](../../assets/Pasted%20image%2020261107143917.png)
## Task 1.2

**What do we call the network of compromised machines that attackers use to launch DDoS attacks?**  
**Answer**: `Botnet`

**Why?**
To scale up into a Distributed Denial-of-Service (`DDoS`) attack, attackers use a `botnet`, which is a network of malware-infected computers, IoT devices, and servers under their control.

![](../../assets/Pasted%20image%2020261107144017.png)
## Task 2.1

**Which attacker motive aims to make customers lose confidence in a company?**  
**Answer**: `Reputational Damage`

**Why?**
Looking at the motives table, `Reputational Damage` is defined as causing customers to lose trust in a company (for example, crashing game servers on launch day).

![](../../assets/Pasted%20image%2020261107144519.png)
## Task 2.2

**Which motive most likely drove the 2023 DDoS attack against Microsoft?**  
**Answer**: `Hacktivism`

**Why?**
In the "In the Wild" section, the 2023 Layer 7 DDoS attack that caused outages across Azure, OneDrive, and Outlook was claimed by the hacktivist group Anonymous Sudan, making the motive `Hacktivism` (disruption for social or political protest).

![](../../assets/Pasted%20image%2020261107144608.png)

![](../../assets/Pasted%20image%2020261107144617.png)
## Task 3.1

**What is the attacker's IP address?**  
**Answer**: `203.12.23.195`

**Why?**
Opening `access.log` on the Desktop, right after normal user requests from `192.168.1.5` and `192.168.1.10`, we see a massive flood of automated `curl/7.88.1` requests all coming from `203.12.23.195` at `01:59:08`.

![](../../assets/Pasted%20image%2020261107155720.png)
## Task 3.2

**Which page is repeatedly targeted by the attacker's requests?**  
**Answer**: `/login`

**Why?**
In the same `access.log` screenshot, every request sent by `203.12.23.195` is a `GET /login HTTP/1.1` request.

![](../../assets/Pasted%20image%2020261107155720.png)
## Task 3.3

**After the attack, what error code do legitimate users receive?**  
**Answer**: `503`

**Why?**
Scrolling down to the end of the flood in `access.log`, the server becomes overloaded and starts returning HTTP status code `503` (Service Unavailable) to both the attacker and legitimate internal users (`192.168.1.10` and `192.168.1.5`).

![](../../assets/Pasted%20image%2020261107155807.png)
## Task 4.1

**What was the most frequently requested uri?**  
**Answer**: `/search`

**Why?**
Querying `index="main"` in Splunk over `All time` and checking the `uri` field on the left panel shows `/search` at the top with `2,068` requests (`81.804%` of all events).

![](../../assets/Pasted%20image%2020261107160202.png)
## Task 4.2

**Which clientip made the most requests to the target uri?**  
**Answer**: `203.0.113.7`

**Why?**
Filtering by `index="main" uri="/search"` and clicking on `clientip` shows `203.0.113.7` as the top value with `50` requests.

![](../../assets/Pasted%20image%2020261107160318.png)
## Task 4.3

**How many IP addresses were part of the botnet that attacked your website?**  
**Answer**: `60`

**Why?**
Looking at the header of the `clientip` field summary for `index="main" uri="/search"`, Splunk reports `60 Values, 100% of events` across the `203.0.113.0/24` range.
![](../../assets/Pasted%20image%2020261107160516.png)

## Task 4.4

**Which useragent was most commonly used by the attacking traffic?**  
**Answer**: `Java/1.8.0_181`

**Why?**
Clicking on the `useragent` field for the `/search` traffic shows `Java/1.8.0_181` in first place with `135` events (`6.528%`), followed by `python-requests/2.25.1` and `CustomAgent/1.0`.

![](../../assets/Pasted%20image%2020261107160539.png)
## Task 4.5

**Use the timechart command to visualize the requests. What is the peak number of requests made per second during the attack?**  
**Answer**: `207`

**Why?**
We run the query `index="main" uri="/search" | timechart span=1sec count by uri limit=5` and switch to the `Visualization` tab, where the line chart peaks at `207` requests per second.

![](../../assets/Pasted%20image%2020261107160824.png)


## Task 4.6

**Which legitimate (non-attacking) clientip received the first 503 response status post-attack?**  
**Answer**: `10.10.0.27`

**Why?**
Searching `index="main" "503"` (or `status=503 clientip!="203.0.113.*"`) and looking at the earliest legitimate `10.10.0.x` event at `13:13:02` (`9/3/25 1:13:02.000 PM`) shows `10.10.0.27` receiving a `503` on `GET /contact HTTP/1.1`.

![](../../assets/Pasted%20image%2020261107161016.png)

![](../../assets/Pasted%20image%2020261107161011.png)
## Task 5.1

**What type of security challenge blocks bots by asking users to solve a simple puzzle?**  
**Answer**: `CAPTCHA`

**Why?**
Under the "Challenges" section, a `CAPTCHA` requires the user to solve a small puzzle (like clicking images or checking a box) before granting access, which blocks or slows down automated bots.

![](../../assets/Pasted%20image%2020261107161251.png)

## Task 5.2

**Which CDN feature spreads traffic across multiple servers to prevent overload?**  
**Answer**: `load-balancing`

**Why?**
As highlighted in the Content Delivery Network (CDN) section, CDNs provide `load-balancing` to distribute incoming traffic across multiple servers so no single server gets overloaded.

![](../../assets/Pasted%20image%2020261107161512.png)