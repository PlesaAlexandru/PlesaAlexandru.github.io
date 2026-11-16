---
title: NetworkMiner Walkthrough
description: Learn how to use NetworkMiner to analyse recorded traffic files and practice network forensics activities.
pubDate: 2026-11-03
categories:
  - SOC
  - Walkthrough
  - TryHackMe
  - Network
  - NetworkMiner
tags:
  - Lessons
image: /image/networkminer.png
---
---
## Summary

In this room we learned how to use NetworkMiner (both version 2.7.2 and 1.6.1) to quickly parse PCAP files and pull out useful info without digging through every single packet manually. We used the different tabs to check host details, extract transferred files and images, grab cleartext passwords and NTLMv2 hashes, and read emails. It is a great tool to use alongside Wireshark when doing quick triage on a packet capture.

---

**Learning Objectives**

- Discover what NetworkMiner is
- Learn the basics of NetworkMiner
- Explore the different information tabs and extract information efficiently


---
## Task 1.1

**What is the total number of frames?**  
**Answer**: `460`

**Why?**
After loading `mx-3.pcap` in NetworkMiner, we look at the **Case Panel** on the right side.
![](../../assets/Pasted%20image%2020261103163948.png)

Right-click on `mx-3.pcap` and select **Show Metadata**.
![](../../assets/Pasted%20image%2020261103164027.png)

Looking at the **Frames** row gives `460`.
![](../../assets/Pasted%20image%2020261103163934.png)

## Task 1.2

**How many IP addresses use the same MAC address with host 145.253.2.203?**  
**Answer**: `2`

**Why?**
In the **Hosts** tab, expand `145.253.2.203` and then expand `MAC: FEFF20000100`, which shows `2` other IP addresses using the same MAC (`65.208.228.223` and `216.239.59.99`).
![](../../assets/Pasted%20image%2020261103164142.png)

## Task 1.3

**How many packets were sent from host 65.208.228.223?**  
**Answer**: `72`

**Why?**
Expanding host `65.208.228.223` in the **Hosts** tab shows `Sent: 72 packets (76,368 Bytes)`.
![](../../assets/Pasted%20image%2020261103164217.png)

## Task 1.4

**What is the name of the webserver banner under host 65.208.228.223?**  
**Answer**: `Apache`

**Why?**
Still under `65.208.228.223`, expanding **Host Details** shows `Web Server Banner 1 : TCP 80 : Apache`.
![](../../assets/Pasted%20image%2020261103164252.png)

## Task 1.5

**What is the extracted username for the 02694W-WIN10 host?**  
**Answer**: `#B\Administrator`

**Why?**
After removing `mx-3.pcap` and loading `mx-4.pcap`, go to the **Credentials** tab. For `172.16.66.37 [02694W-WIN10]`, the **Username** column shows `#B\Administrator`.
![](../../assets/Pasted%20image%2020261103164410.png)

## Task 2.1

**What is the extracted password for the user logged into the 02694W-WIN10 host?**  
**Answer**: `$NETNTLMv2$#B$136B077D942D9A63$FBFF3C253926907AAAAD670A9037F2A5$01010000000000000094`

**Why?**  
In the same **Credentials** tab, right-clicking the second NTLMSSP row for `#B\Administrator` and clicking **Copy Password** gives us the full NetNTLMv2 hash.
![](../../assets/Pasted%20image%2020261103164528.png)

## Task 2.2

**What is the name of the Linux distro mentioned in the file associated with frame 63075?**  
**Answer**: `CentOS`

**Why?**
Load `mx-7.pcap`, go to the **Parameters** tab, and search for `63075` in the filter bar. The `Host` value is `mirrorlist.centos.org`, so the distro is `CentOS`.
![](../../assets/Pasted%20image%2020261103165350.png)

## Task 2.3

**What is the header of the page associated with frame 75942?**  
**Answer**: `Password-Ned AB`

**Why?**
In the **Files** tab, filtering for `75942` shows `index[1].html`. Opening the file in the browser shows the header `Password-Ned AB`.
![](../../assets/Pasted%20image%2020261103170623.png)

## Task 3.1

**What is the source address of the image "ads.bmp.2E5F0FD9.bmp"?**  
**Answer**: `80.239.178.187`

**Why?**
Still in the **Files** tab, searching for `ads.bmp.2E5F0FD9[1].bmp` shows the **Source host** is `80.239.178.187`.
![](../../assets/Pasted%20image%2020261103170900.png)

## Task 3.2

**What is the frame number of the possible TLS anomaly?**  
**Answer**: `36255`

**Why?**
Going to the **Anomalies** tab shows two TLS alerts, and the first one is on frame `36255`.
![](../../assets/Pasted%20image%2020261103172350.png)

## Task 3.3

**Look at the messages. Which platform sent an email with the subject starting with "You have more"?**  
**Answer**: `Facebook`

**Why?**
Load `mx-9.pcap`, go to the **Messages** tab, and search for `You have more`. Looking at the **From** field and details on the right shows it came from `Facebook`.
![](../../assets/Pasted%20image%2020261103172850.png)

## Task 3.4

**What is the email address of Branson Matheson?**  
**Answer**: `branson@sandsite.org`

**Why?**
Searching for `Branson Matheson` in the **Messages** tab gives the email `branson@sandsite.org`.
![](../../assets/Pasted%20image%2020261103172938.png)

## Task 3.5

**What is the OS name of the host 131.151.37.122?**  
**Answer**: `Windows - Windows NT 4`

**Why?**
For `case1.pcap`, expand host `131.151.37.122` in the **Hosts** tab and check the **OS** node, which shows `Windows - Windows NT 4`.
![](../../assets/Pasted%20image%2020261103173405.png)

## Task 4.1

**Investigate the hosts 131.151.37.122 and 131.151.32.91. How many data bytes were received from host 131.151.32.91 to host 131.151.37.122 through port 1065?**  
**Answer**: `192`

**Why?**
Expanding `131.151.32.91` -> **Outgoing sessions** -> `Server: 131.151.37.122 (Windows) TCP 1065` shows `Client: 131.151.32.91 TCP 3614 (192 data bytes sent)`.
![](../../assets/Pasted%20image%2020261103173732.png)

## Task 4.2

**Investigate the hosts 131.151.37.122 and 131.151.32.21. How many data bytes were received from host 131.151.37.122 to host 131.151.32.21 through port 143?**  
**Answer**: `20769`

**Why?**
Expanding `131.151.32.21` -> **Outgoing sessions** -> `Server: 131.151.37.122 (Windows) TCP 143` shows `Server: 131.151.37.122 (Windows) TCP 143 (20769 data bytes sent)`.
![](../../assets/Pasted%20image%2020261103173907.png)

## Task 4.3

**What is the sequence number of frame 9?**  
**Answer**: `2AD77400`

**Why?**
Since the **Frames** tab is only in NetworkMiner `1.6.1`, we open `case1.pcap` in `1.6.1`, go to **Frames**, expand **Frame 9** -> **TCP**, and see `Sequence Number = 2AD77400`.
![](../../assets/Pasted%20image%2020261103174052.png)

## Task 4.4

**What is the number of the detected "content types"?**  
**Answer**: `2`

**Why?**
In the **Parameters** tab, filtering for `Content-Type` shows only `2` types: `multipart/mixed` and `text/plain`.
![](../../assets/Pasted%20image%2020261103174347.png)

## Task 4.5

**What is the USB product's brand name?**  
**Answer**: `ASIX`

**Why?**
Load `case2.pcap` and filter for `usb` in the **Files** tab to find `USB2-LAN.zip`. Opening the zip and checking `Ax88772.inf` shows the brand name is `ASIX`.
![](../../assets/Pasted%20image%2020261103175122.png)

## Task 4.6

**What is the name of the phone model?**  
**Answer**: `Lumia 535`

**Why?**
In the **Images (612)** tab, scrolling through the extracted pictures shows a photo named `MMD_Lumia535...` of a `Lumia 535` phone.
![](../../assets/Pasted%20image%2020261103175930.png)

![](../../assets/Pasted%20image%2020261103175921.png)

## Task 4.7

**What is the source IP of the fish image?**  
**Answer**: `50.22.95.9`

**Why?**
In the **Files** tab, searching for `fish` brings up `Crazy-Fishing[1].jpg`, and its source host IP is `50.22.95.9`.
![](../../assets/Pasted%20image%2020261103175958.png)

## Task 4.8

**What is the password of the "homer.pwned.se@gmx.com"?**  
**Answer**: `spring2015`

**Why?**
In the **Credentials** tab for `case2.pcap`, looking at the `Pop3` login for `homer.pwned.se@gmx.com` gives the password `spring2015`.
![](../../assets/Pasted%20image%2020261103180107.png)

## Task 4.9

**What is the DNS Query of frame 62001?**  
**Answer**: `pop.gmx.com`

**Why?**
Checking frame `62001` in the **DNS** tab (or in the **Frames** tab in NetworkMiner `1.6.1`) shows it is querying `pop.gmx.com`.




