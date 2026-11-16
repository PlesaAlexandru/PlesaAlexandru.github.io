---
title: Data Exfiltration Detection Walkthrough
description: Learn how to detect data exfiltration attempts in various network channels.
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

In this room we went through different ways attackers try to sneak data out of a network, using protocols like DNS, FTP, HTTP, and ICMP. We used both Splunk and Wireshark to look at the logs and packets, spot unusual traffic spikes, and find the hidden flags inside the exfiltrated data. Overall, a really good practical room to see how data exfiltration looks from a SOC analyst perspective.

---

**Learning Objectives**

- Understand the common methods used for data exfiltration.
- Learn how to detect exfiltration attempts using network traffic analysis.
- Identify signs of exfiltration on endpoint devices.
- Correlate logs in a SIEM to uncover hidden exfiltration channels.


---
## Task 1.1

**Exfiltrating the data through HTTP comes under which technique?**  
**Answer**: `Network-based`

**Why?**
Looking at the **Techniques and Indicators** table from the lesson, methods like HTTP/HTTPS uploads, FTP transfers, DNS tunneling, and ICMP are all listed under the `Network-based` technique.
![](../../assets/Pasted%20image%2020261103205748.png)

## Task 1.2

**What is the suspicious domain receiving the DNS traffic?**  
**Answer**: `tunnelcorp.net`

**Why?**
In Splunk, we can start by searching the DNS logs in the `data_exfil` index with:
`index=data_exfil sourcetype=dns_logs`
![](../../assets/Pasted%20image%2020261103205847.png)

If we look at the `query` field on the left panel, next to normal domains like `microsoft.com` and `stackoverflow.com`, there are a lot of random-looking subdomains (like `036er95gzqeabe3ymiht.tunnelcorp.net`) that all point to `tunnelcorp.net`.
![](../../assets/Pasted%20image%2020261103205842.png)

## Task 1.3

**How many suspicious traffic/logs related to DNS tunneling were observed?**  
**Answer**: `315`

**Why?**
Looking at the fields on the left, there is a `label` field that marks the weird traffic as `suspicious`. Adding `label=suspicious` to our search (`index=data_exfil sourcetype=dns_logs label=suspicious`) gives us `315` events.
![](../../assets/Pasted%20image%2020261103210108.png)

## Task 1.4

**Which local IP sent the maximum number of suspicious requests?**  
**Answer**: `192.168.1.103`

**Why?**
Using the same search (`index=data_exfil sourcetype=dns_logs label=suspicious`), we can click on `src_ip` in the left panel. The top IP is `192.168.1.103` with `72` events (`22.857%`).
![](../../assets/Pasted%20image%2020261103210203.png)

## Task 1.5

**How many connections were observed from the guest account?**  
**Answer**: `5`

**Why?**
For the FTP part in Wireshark, we can filter for login requests that use the `guest` user with:
`ftp.request.arg == "guest"`
It returns `5` packets.
![](../../assets/Pasted%20image%2020261103211023.png)

## Task 2.1

**Apply the filter; what is the name of the customer-related file exfiltrated from the root account?**  
**Answer**: `customer_data.xlsx`

**Why?**  
Looking at the FTP packets (like packet `3520` going to `185.203.119.12`), if we expand the `File Transfer Protocol (FTP)` tab in the details pane, we can see the `STOR` command sending `customer_data.xlsx`.
![](../../assets/Pasted%20image%2020261103211117.png)

## Task 2.2

**Which internal IP was found to be sending the largest payload to an external IP?**  
**Answer**: `192.168.1.105`

**Why?**
We can sort the FTP packets by the `Length` column to get the biggest packet at the top. Packet `2853` has a length of `127` bytes, and looking at the `Source` column gives us `192.168.1.105`.
![](../../assets/Pasted%20image%2020261103211258.png)

## Task 2.3

**What is the flag hidden inside the ftp stream transferring the CSV file to the suspicious IP?**  
**Answer**: `THM{ftp_exfil_hidden_flag}`

**Why?**
Clicking on packet `2853` from the previous task and expanding the `File Transfer Protocol (FTP)` section shows `STOR internal_passwords.csv\r\n` and right below it the flag: `DATA: THM{ftp_exfil_hidden_flag}\r\n`.
![](../../assets/Pasted%20image%2020261103211309.png)

## Task 3.1

**Which internal compromised host was used to exfiltrate this sensitive data?**  
**Answer**: `192.168.1.103`

**Why?**
In Splunk, we check the HTTP logs and look at the `method` field, which has `GET` and `POST`. Since data exfiltration over HTTP usually uses `POST` to send data out, we filter by `method=POST`.
![](../../assets/Pasted%20image%2020261103212022.png)

Then we check the `uri` field and see `/v1/sync/upload`, which is clearly used for uploading files.
![](../../assets/Pasted%20image%2020261103212058.png)

Putting both together in the search bar (`index=data_exfil sourcetype=http_logs method=POST uri="/v1/sync/upload"`) gives `774` events.
![](../../assets/Pasted%20image%2020261103211951.png)

Clicking on `src_ip` shows `192.168.1.103` at the top with `163` requests.
![](../../assets/Pasted%20image%2020261103212131.png)

## Task 3.2

**What's the flag hidden inside the exfiltrated data?**  
**Answer**: `THM{http_raw_3xf1ltr4t10n_succ3ss}`

**Why?**
In Wireshark, we filter for HTTP `POST` requests coming from `192.168.1.103`:
`(ip.addr == 192.168.1.103) && (http.request.method == "POST")`
Sorting by `Length` puts packet `3589` (`782` bytes) at the top. Clicking on `Data (654 bytes)` highlights the ASCII text on the bottom right, where we can see `THM{http_raw_3xf1ltr4t10n_succ3ss}`.
![](../../assets/Pasted%20image%2020261103212417.png)

## Task 3.3

**What is the flag found in the exfiltrated data through ICMP?**  
**Answer**: `THM{1cmp_3ch0_3xf1ltr4t10n_succ3ss}`

**Why?**
To find data hidden inside ping packets, we filter for ICMP Echo Requests (`icmp.type == 8`) that have a frame length bigger than 100:
`icmp.type == 8 and frame.len > 100`
Selecting packet `8392` and clicking on `Data (120 bytes)` shows the ASCII text on the right side with the flag `THM{1cmp_3ch0_3xf1ltr4t10n_succ3ss}`.
![](../../assets/Pasted%20image%2020261103212952.png)


