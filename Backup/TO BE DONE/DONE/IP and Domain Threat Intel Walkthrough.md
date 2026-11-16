---
title: IP and Domain Threat Intel Walkthrough
description: Investigate suspicious IPs and domains using OSINT and threat intelligence tools.
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

In this room, we investigate suspicious IP addresses and domains using DNS lookups (`NsLookup.io`), WHOIS registration records, VirusTotal community intelligence, BGP/ASN lookups, and internet-wide service scanners like Censys. We analyze how threat actors hide behind CDNs (`Cloudflare`), bulletproof or privacy-oriented hosting (`1337 Services GmbH`), and VPN providers (`Mullvad VPN`), and we pivot across TLS certificates and reverse/forward DNS records to uncover C2 infrastructure (`Remcos`, `AsyncRAT`, `DcRat`, and `raytracingengine.com`).

---

**Learning Objectives**

- Understand IP and domain threat intelligence for a SOC
- Geolocate IPs and interpret their Autonomous System Numbers (ASNs)
- Detect red-flag infrastructure via Shodan/Censys service banners
- Assess reputation with various tools
- Enrich domains with WHOIS age, DNS records, and certificate transparency

---
## Task 1.1
*What is the AS name associated with the domain purematrixa.com?*

**Answer**: `Cloudflare`

**Why?**
Looking up `purematrixa.com` on `NsLookup.io` shows that its `A` records (`188.114.96.0` and `188.114.97.0`) resolve to `AS13335` with the `AS name` **`Cloudflare`**, meaning the domain is proxied behind Cloudflare's CDN.

![](../../assets/Pasted%20image%2020261112002517.png)

## Task 1.2
*According to the WHOIS record for purematrixa.com, how old is the domain?*

**Answer**: `1 days old` (or `1`)

**Why?**
Checking the WHOIS record for `Purematrixa.com` under **Domain Profile -> Dates** reveals that the domain is **`1 days old`** (`Domain Status: Never Registered Before`), which is a classic indicator of a newly registered domain (NRD) stood up specifically for a phishing or malware campaign.

![](../../assets/Pasted%20image%2020261112005232.png)

## Task 2.1
*How many security vendors on VirusTotal flagged the IP address 2.58.56.50 as malicious?*

**Answer**: `8`

**Why?**
Searching `2.58.56.50` on VirusTotal shows **`8/91`** security vendors flagging the IP address as malicious, along with a negative community score of `-39`.

![](../../assets/Pasted%20image%2020261112010612.png)

## Task 2.2
*According to the VirusTotal community comments for 2.58.56.50, which malware family is associated with this C2 server?*

**Answer**: `Remcos`

**Why?**
Switching to the **Community** tab on VirusTotal shows a comment from `drb_ra` noting `Remcos Found / C2: 2[.]58[.]56[.]50:2404` with the tags `#c2 #Remcos`.

![](../../assets/Pasted%20image%2020261112042454.png)

## Task 2.3
*Which organization / ASN owns the IP block for 2.58.56.50?*

**Answer**: `1337 Services GmbH` (or `AS210558`)

**Why?**
Both the VirusTotal header and the community comment identify the Autonomous System as `AS 210558` (**`1337 Services GmbH`**).

![](../../assets/Pasted%20image%2020261112042608.png)

## Task 2.4
*What tags are associated with this Autonomous System in the ASN lookup?*

**Answer**: `Server Hosting, Tor services`

**Why?**
Inspecting the ASN profile page under **Tags** shows two infrastructure tags assigned to this provider: **`Server Hosting`** and **`Tor services`**.

![](../../assets/Pasted%20image%2020261112042651.png)

## Task 3.1
*How many total services are exposed on the host 64.89.160.44?*

**Answer**: `9`

**Why?**
Opening the Censys host report for `64.89.160.44` shows **`9 Total Services`** listed under the **Summary** section.

![](../../assets/Pasted%20image%2020261112044824.png)

## Task 3.2
*Which non-standard or remote access ports stand out among the exposed services on 64.89.160.44?*

**Answer**: `1000, 7777` (and `3389 / RDP`)

**Why?**
Looking closely at the 9 exposed services (`135 / DCERPC`, `139 / NETBIOS`, `445 / SMB`, `1000 / UNKNOWN`, `3389 / RDP`, `7777 / UNKNOWN`, and three `VALVE` game ports), we see `3389 / RDP` (explaining the `REMOTE_ACCESS` tag) alongside two suspicious `UNKNOWN` TCP services on ports **`1000`** and **`7777`**.

![](../../assets/Pasted%20image%2020261112044844.png)

## Task 3.3
*Which RAT family is identified in the TLS certificate on port 1000/TCP?*

**Answer**: `AsyncRAT` (or `AsyncRAT Server`)

**Why?**
Scrolling down to the `UNKNOWN 1000 / TCP` service details in Censys and inspecting the self-signed TLS **Certificate** reveals `Subject DN: CN=AsyncRAT Server` and `Issuer DN: CN=AsyncRAT Server`.

![](../../assets/Pasted%20image%2020261112044951.png)

## Task 3.4
*Which second RAT family is identified in the TLS certificate on port 7777/TCP?*

**Answer**: `DcRat`

**Why?**
Inspecting the TLS certificate on the other unknown port (`7777 / TCP`) reveals `Subject DN: CN=DcRat` and `Issuer DN: CN=DcRat Server, OU=qwqdanchun, O=DcRat By qwqdanchun, L=SH, C=CN`.

![](../../assets/Pasted%20image%2020261112045013.png)

## Task 4.1
*A London-based user logs in from a London IP address. You can confirm the IP belongs to a Mullvad VPN provider. Would you raise the alarm and prioritize the alert (Yea/Nay)?*

**Answer**: `Yea`

**Why?**
Even though the geolocation city (`London`) matches the user's normal location, logging into corporate infrastructure from a commercial anonymizing VPN provider like **Mullvad VPN** (rather than a residential ISP or corporate VPN) is a strong indicator of an attacker attempting to blend in with the user's region while masking their real origin IP. As the room note highlights after answering, no single tool catches every VPN, so we always correlate ASN/VPN context with behavioral signals.

![](../../assets/Pasted%20image%2020261112045315.png)
![](../../assets/Pasted%20image%2020261112045432.png)

## Task 5.1
*What is the IP address associated with the domain raytracingengine.com?*

**Answer**: `35.188.105.97`

**Why?**
Searching `raytracingengine.com` in the **WHOIS Domain Lookup** tool immediately shows `IP Address: 35.188.105.97`.

![](../../assets/Pasted%20image%2020261112045556.png)

## Task 5.2
*Which cloud provider / organization owns the network block for 35.188.105.97?*

**Answer**: `Google LLC` (or `GOOGLE-CLOUD-PLATFORM` / `AS396982`)

**Why?**
Checking **Network and Geolocation** for `35.188.105.97` shows `WHOIS Network: Google LLC (GOOGLE-CLOUD)` and `Autonomous System: 35.188.96.0/20 GOOGLE-CLOUD-PLATFORM - Google LLC (396982), US`, located in `Council Bluffs, United States (US)`.

![](../../assets/Pasted%20image%2020261112045705.png)

## Task 5.3
*What city and state are listed in the WHOIS address record for raytracingengine.com?*

**Answer**: `San Mateo, California`

**Why?**
Looking at the **Address** field in the WHOIS record for `raytracingengine.com` shows **`San Mateo, California, US`**.

![](../../assets/Pasted%20image%2020261112045739.png)

## Task 5.4
*When was the domain raytracingengine.com created, updated, and set to expire?*

**Answer**: `Created: 2/21/2026` / `Updated: 6/1/2026` / `Expires: 2/21/2027`

**Why?**
In the **Important Dates** box of the WHOIS lookup, we find:
- **Created**: `2/21/2026`
- **Updated**: `6/1/2026`
- **Expires**: `2/21/2027`

![](../../assets/Pasted%20image%2020261112045810.png)

## Task 5.5
*Pivoting on 35.188.105.97 in Censys, what other domains are listed under Forward DNS, what OS is running, and how many total services are exposed?*

**Answer**: `prohomedevelopers.com, www.mycdn.moe, mycdn.moe, api.mycdn.moe` / `Canonical Linux` / `11`

**Why?**
Searching `35.188.105.97` in Censys reveals the full host profile:
- **Forward DNS**: `prohomedevelopers.com`, `www.mycdn.moe`, `mycdn.moe`, `api.mycdn.moe`
- **OS**: `Canonical Linux`
- **Total Services**: `11 Total Services` (including `22 / SSH`, `23 / UNKNOWN`, `25 / SMTP`, `80 / HTTP`, `81 / HTTP`, `82 / HTTP`, and `443 / HTTP`).

![](../../assets/Pasted%20image%2020261112045843.png)

---

## Personal Note: IP & Domain Threat Intel Pivoting Workflow

Quick reference of the tools and CLI commands used to investigate suspicious IPs, domains, and URLs:

| Indicator Type | Tool / Command | What It Reveals |
| :--- | :--- | :--- |
| **IP Reputation & Abuse** | **AbuseIPDB** | Confidence of abuse score, ISP/ASN, country, and community reports (e.g., SSH brute force, port scanning). |
| **Exposed Services & TLS Certs** | **Censys** / **Shodan** | Open ports, service banners (`OpenSSH`, `nginx`), OS fingerprint, TLS certificate subject/issuer (`slmgr.win` KMS emulator), and **Forward DNS** domains hosted on the same IP. |
| **Malicious URLs & Payloads** | **URLhaus** / **ThreatFox** | Tracks active malware distribution URLs, payload hashes, and tags (`Mozi`, `Mirai`, `AsyncRAT`). |
| **Live Web Page Inspection** | **urlscan.io** | Safely renders suspicious phishing domains (`payout-service.com`), capturing DOM snapshots, HTTP transactions, redirected URLs, and server IPs without visiting the site directly. |
| **Domain Registration & DNS** | - `whois <domain>`<br>- `dig <domain> ANY`<br>- `nslookup -type=TXT <domain>` | - **WHOIS**: Registrar, creation/expiration dates (newly registered domains are high-risk), registrant location.<br>- **DNS**: `A`/`AAAA` records, `MX` mail servers, `NS` nameservers, and `TXT` (`SPF`/`DMARC` or verification tokens). |