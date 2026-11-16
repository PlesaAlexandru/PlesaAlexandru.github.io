---
title: Man-in-the-Middle Detection Walkthrough
description: Learn what MITM attack is, and how to identify the footprints of this attack in the network traffic.
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

In this room we looked at a Man-in-the-Middle (MITM) attack in Wireshark (`network-traffic.pcap`) where the attacker chained three techniques together: ARP Spoofing to get in the middle of the network, DNS Spoofing to redirect `corp-login.acme-corp.local` to their own IP, and SSL Stripping to downgrade HTTPS to HTTP and grab the victim's login password in cleartext.

---

**Learning Objectives**

- Understand common MITM attack vectors and techniques
- Learn to identify indicators of compromise related to MITM attacks
- Master network monitoring tools for detecting suspicious traffic patterns
- Practice incident response procedures for MITM scenarios


---
## Task 1.1

**How many ARP packets from the gateway MAC Address were observed?**  
**Answer**: `10`

**Why?**
To see the normal ARP traffic coming from the gateway (`192.168.10.1` with MAC `02:aa:bb:cc:00:01`), we use the filter:
`arp && arp.src.proto_ipv4 == 192.168.10.1 && eth.src == 02:aa:bb:cc:00:01`
Looking at the bottom right (`Displayed: 10`), there are `10` ARP packets from the real gateway MAC.
![](../../assets/Pasted%20image%2020261104142935.png)




## Task 1.2

**What MAC address was used by the attacker to impersonate the gateway?**  
**Answer**: `02:fe:fe:fe:55:55`

**Why?**
To spot IP-to-MAC conflicts from ARP spoofing, we can use Wireshark's duplicate address filter:
`arp.duplicate-address-detected || arp.duplicate-address-frame`
Clicking on packet `258` shows a yellow warning at the bottom: `[Duplicate IP address detected for 192.168.10.1 (02:fe:fe:fe:55:55) - also in use by 02:aa:bb:cc:00:01]`. Also, under `Ethernet II` -> `Source`, the `LG bit` is set to `1` (`Locally administered address, this is NOT the factory default`), which tells us `02:fe:fe:fe:55:55` is a spoofed MAC address pretending to be `192.168.10.1`.
![](../../assets/Pasted%20image%2020261104143410.png)



## Task 1.3

**How many Gratuitous ARP replies were observed for 192.168.10.1?**  
**Answer**: `2`

**Why?**
We can filter for Gratuitous ARP packets for `192.168.10.1` with:
`arp.isgratuitous && arp.src.proto_ipv4 == 192.168.10.1`
It returns `2` broadcast packets (packets `1` and `116`).
![](../../assets/Pasted%20image%2020261104143708.png)



## Task 1.4

**How many unique MAC addresses claimed the same IP (192.168.10.1)?**  
**Answer**: `2`

**Why?**
Filtering for all ARP replies (`arp.opcode == 2`) that claim the IP `192.168.10.1`:
`arp.opcode == 2 && arp.src.proto_ipv4 == 192.168.10.1`
gives `24` packets. Looking at the `Source` column, we can see `2` different MAC addresses claiming `192.168.10.1`: the real gateway (`02:aa:bb:cc:00:01`) and the attacker (`02:fe:fe:fe:55:55`).
![](../../assets/Pasted%20image%2020261104144007.png)


## Task 1.5

**How many ARP spoofing packets were observed in total from the attacker?**  
**Answer**: `14`

**Why?**
Using `arp.duplicate-address-detected || arp.duplicate-address-frame` (or `arp.opcode == 2 && arp.src.proto_ipv4 == 192.168.10.1 && eth.src == 02:fe:fe:fe:55:55`) filters only the fake ARP replies sent by `02:fe:fe:fe:55:55`. At the bottom right we can see `Displayed: 14`.
![](../../assets/Pasted%20image%2020261104144640.png)
## Task 2.1

**How many DNS responses were observed for the domain corp-login.acme-corp.local?**  
**Answer**: `211`

**Why?**  
We filter for DNS responses (`dns.flags.response == 1`) for the domain `corp-login.acme-corp.local`:
`dns.flags.response == 1 && dns.qry.name == "corp-login.acme-corp.local"`
The bottom bar shows `211` displayed packets.
![](../../assets/Pasted%20image%2020261104145230.png)
## Task 2.2

**How many DNS requests were observed from the IPs other than 8.8.8.8?**  
**Answer**: `2`

**Why?**
Normal DNS replies come from `8.8.8.8`, so we exclude `8.8.8.8` from the source IP:
`dns.flags.response == 1 && ip.src != 8.8.8.8 && dns.qry.name == "corp-login.acme-corp.local"`
This leaves `2` packets (packets `1124` and `2239`) coming from `192.168.10.55`. If we check the `Ethernet II` tab, the source MAC is `02:fe:fe:fe:55:55`, which is the same attacker from the ARP spoofing tasks.
![](../../assets/Pasted%20image%2020261104145618.png)

## Task 2.3

**What IP did the attacker's forged DNS response return for the domain?**  
**Answer**: `192.168.10.55`

**Why?**
Looking at the `Info` column for those two packets (`1124` and `2239`), the response says `A corp-login.acme-corp.local A 192.168.10.55`, sending the victim to the attacker's own IP `192.168.10.55`.
![](../../assets/Pasted%20image%2020261104145817.png)


## Task 3.1

**How many POST requests were observed for our domain corp-login.acme-corp.local?**  
**Answer**: `1`

**Why?**
After redirecting the domain to `192.168.10.55`, the attacker used SSL Stripping so the victim logged in over plain HTTP instead of HTTPS. Filtering for HTTP `POST` requests to that host:
`(http.host == "corp-login.acme-corp.local") && (http.request.method == "POST")`
returns `1` packet (packet `2241`, `POST /login HTTP/1.1`).
![](../../assets/Pasted%20image%2020261104150705.png)






## Task 3.2

**What's the password of the victim found in the plaintext after successful ssl stripping attack?**  
**Answer**: `Secret123!`


**Why?**
Right-click packet `2241` (`POST /login HTTP/1.1`) and click `Follow -> HTTP Stream`.
![](../../assets/Pasted%20image%2020261104150748.png)

In the stream window, we can see the login data in cleartext: `username=alice&password=Secret123!`.
![](../../assets/Pasted%20image%2020261104150739.png)

---

## Attack Indicators (TryHackMe Notes)

Based on the TryHackMe lesson and what we saw in the PCAP, these are the main indicators that show traffic is being spoofed or intercepted:

### 1. ARP Spoofing
- **Two MAC addresses claiming the same IP**: Seeing two different MAC addresses send `ARP Reply` packets for the same IP (especially the default gateway `192.168.10.1`).
- **Wireshark Duplicate IP warning**: Wireshark flags this in the packet details with `[Duplicate IP address detected for 192.168.10.1 (02:fe:fe:fe:55:55) - also in use by 02:aa:bb:cc:00:01]`.
- **Locally Administered MAC bit (`LG bit == 1`)**: Under `Ethernet II`, a real factory MAC has the `LG` bit set to `0`. Here, `02:fe:fe:fe:55:55` has `LG bit: 1 (Locally administered address)`, meaning it was manually set/spoofed.
- **Repeated unsolicited ARP replies**: A flood of `ARP Reply` (`arp.opcode == 2`) or Gratuitous ARP packets sent without a matching `Who has` request to keep the victim's ARP cache poisoned.

### 2. DNS Spoofing
- **DNS replies from an unexpected IP**: Normal DNS responses come from the configured DNS server (`8.8.8.8`). Getting a DNS reply (`dns.flags.response == 1`) from a local host IP (`192.168.10.55`) is a clear indicator of spoofing.
- **Different `A` records for the same domain**: Legitimate replies resolved `corp-login.acme-corp.local` to `93.184.216.34` or `192.168.10.200`, while the fake reply pointed it to `192.168.10.55`.
- **Same attacker MAC in Ethernet II**: The rogue DNS server `192.168.10.55` has the exact same MAC (`02:fe:fe:fe:55:55`) used in the ARP spoofing attack.

### 3. SSL Stripping
- **Plaintext HTTP on a login page**: Login pages (`corp-login.acme-corp.local/login`) should use HTTPS (port `443`). Seeing unencrypted `HTTP` traffic (port `80`) with `POST /login` and form data (`application/x-www-form-urlencoded`) shows the connection was downgraded.
- **Traffic going straight to the attacker's IP**: The victim (`192.168.10.10`) sends the HTTP `POST` request directly to `192.168.10.55`, exposing `username=alice&password=Secret123!` in cleartext.

---

## Personal Note: MITM Attack Types & Wireshark Detection Filters

Quick summary of the three MITM attacks and the Wireshark filters to check for them:

| Attack Type | How It Works | What to Look For | Wireshark Display Filters |
| :--- | :--- | :--- | :--- |
| **ARP Spoofing** | Attacker sends fake ARP replies linking the Gateway IP to the attacker's MAC so LAN traffic passes through them. | - Duplicate IP warning in Expert Info<br>- 2 MACs claiming the gateway IP<br>- `LG bit = 1` in Ethernet II<br>- Unsolicited ARP replies | - **Duplicate IP warnings:**<br>`arp.duplicate-address-detected \|\| arp.duplicate-address-frame`<br>- **ARP replies for Gateway IP:**<br>`arp.opcode == 2 && arp.src.proto_ipv4 == 192.168.10.1`<br>- **Gratuitous ARPs:**<br>`arp.isgratuitous && arp.src.proto_ipv4 == 192.168.10.1`<br>- **Filter by specific MAC:**<br>`arp && arp.src.proto_ipv4 == 192.168.10.1 && eth.src == <MAC>` |
| **DNS Spoofing** | Attacker intercepts DNS queries and replies with a fake `A` record pointing the domain to the attacker's IP. | - DNS replies coming from an IP other than the real DNS server (`8.8.8.8`)<br>- Domain resolving to a local/attacker IP | - **All DNS replies for a domain:**<br>`dns.flags.response == 1 && dns.qry.name == "corp-login.acme-corp.local"`<br>- **Rogue DNS replies (not from `8.8.8.8`):**<br>`dns.flags.response == 1 && ip.src != 8.8.8.8 && dns.qry.name == "corp-login.acme-corp.local"` |
| **SSL Stripping** | Attacker downgrades `HTTPS` to `HTTP` so the victim logs in over an unencrypted connection. | - Plain `HTTP` (port `80`) on login pages instead of `TLS`<br>- Cleartext credentials in `Follow -> HTTP Stream` | - **HTTP POST logins to domain:**<br>`(http.host == "corp-login.acme-corp.local") && (http.request.method == "POST")`<br>- **All form submissions:**<br>`http.request.method == "POST" && http.content_type contains "urlencoded"` |


