---
title: Invite Only
description: Extract insight from a set of flagged artefacts, and distil the information into usable threat intelligence.
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

In this challenge room, we support an L3 incident responder at TrySecureMe by investigating two flagged indicators (`101.99.76.120` and SHA256 `5d0509f68a9b7c415a726be75a078180e3f02e59866f193b0a99eee8e39c874f`) using the local threat intelligence platform `TryDetectThis2.0` and OSINT research. By pivoting across execution parents, dropped files, and community threat reports, we attribute the activity to an AsyncRAT and Skuld Stealer campaign delivered via hijacked Discord invite links and the ClickFix phishing technique.

---

![](../../assets/Pasted%20image%2020261112050257.png)

---
## Task 1.1
What is the name of the file identified with the flagged SHA256 hash?

**Answer**: `syshelpers.exe`

**Why?**
Searching the flagged SHA256 hash (`5d0509f68a9b7c415a726be75a078180e3f02e59866f193b0a99eee8e39c874f`) in `TryDetectThis2.0` (`http://127.0.0.1:8000`) shows `51/72 vendors flagged as malicious` and identifies the file name as `syshelpers.exe`.

![](../../assets/Pasted%20image%2020261112050405.png)

## Task 1.2
What is the file type associated with the flagged SHA256 hash?

**Answer**: `Win32 EXE`

**Why?**
Directly underneath `syshelpers.exe` in the File summary card, the file type is listed as `Win32 EXE` (`214.0 KB`).

![](../../assets/Pasted%20image%2020261112050437.png)

## Task 1.3
What are the execution parents of the flagged hash? List the names chronologically, using a comma as a separator. Note down the hashes for later use.

**Answer**: `361GJX7J,installer.exe`

**Why?**
Switching to the **Relations** tab and scrolling to **Execution Parents (2)** reveals two parent files ordered by date:
- `2025-05-08 18:20:56`: `361GJX7J` (`047c5eec0445746862710d20e50a5dd04510b7e625fa5c1f5d48ce078001c0de`)
- `2025-06-01 18:49:58`: `installer.exe` (`fa102d4e3cfbe85f5189da70a52c1d266925f3efd122091cdc8fe0fc39033942`)

![](../../assets/Pasted%20image%2020261112050515.png)
![](../../assets/Pasted%20image%2020261112050557.png)

## Task 1.4
What is the name of the file being dropped? Note down the hash value for later use.

**Answer**: `AClient.exe`

**Why?**
Scrolling down to the **Dropped Files (1)** section of the `syshelpers.exe` report shows that it drops `AClient.exe` (`dd02c105809e4ca41a5489e585ba025eddb89a91703b73a566c9903e6406a08c`).

![](../../assets/Pasted%20image%2020261112050644.png)

## Task 2.1
Research the second hash in question 3 and list the four malicious dropped files in the order they appear (from up to down), separated by commas.

**Answer**: `searchHost.exe,syshelpers.exe,nat1.vbs,runsys.vbs`

**Why?**
Searching the second execution parent hash (`fa102d4e3cfbe85f5189da70a52c1d266925f3efd122091cdc8fe0fc39033942`, `installer.exe`) in `TryDetectThis2.0` and checking **Dropped Files (20)** under Relations shows four files flagged as malicious with yellow warning icons from top to bottom: `searchHost.exe` (`37/73`), `syshelpers.exe` (`51/72`), `nat1.vbs` (`1/60`), and `runsys.vbs` (`2/62`).

![](../../assets/Pasted%20image%2020261112050928.png)

![](../../assets/Pasted%20image%2020261112050940.png)

## Task 2.2
Analyse the files related to the flagged IP. What is the malware family that links these files?

**Answer**: `AsyncRAT`

**Why?**
Searching the flagged IP `101.99.76.120` in `TryDetectThis2.0` (`12/94 vendors flagged as malicious`) and opening the **Community** tab reveals community threat intel comments identifying the IP as an `AsyncRAT C2` (`malpedia="AsyncRAT"`).

![](../../assets/Pasted%20image%2020261112051109.png)

![](../../assets/Pasted%20image%2020261112051059.png)

## Task 2.3
What is the title of the original report where these flagged indicators are mentioned? Use Google to find the report.

**Answer**: `From Trust to Threat: Hijacked Discord Invites Used for Multi-Stage Malware Delivery`

**Why?**
In the Community comment on `TryDetectThis2.0`, the Check Point Research report title and URL are listed directly: `Title: From Trust to Threat: Hijacked Discord Invites Used for Multi-Stage Malware Delivery` (`https://research.checkpoint.com/2025/from-trust-to-threat-hijacked-discord-invites-used-for-multi-stage-malware-delivery`).

![](../../assets/Pasted%20image%2020261112051635.png)

## Task 2.4
Which tool did the attackers use to steal cookies from the Google Chrome browser?

**Answer**: `ChromeKatz`

**Why?**
Opening the Check Point Research article (`From Trust to Threat: Hijacked Discord Invites Used for Multi-Stage Malware Delivery`) and reviewing the key findings shows that the threat actors bypassed Chrome's App Bound Encryption (ABE) using adapted tools like `ChromeKatz` to steal cookies from Chromium browsers.

![](../../assets/Pasted%20image%2020261112052118.png)

![](../../assets/Pasted%20image%2020261112052106.png)

## Task 3.1
Which phishing technique did the attackers use? Use the report to answer the question.

**Answer**: `ClickFix`

**Why?**
Both the Check Point Research summary bullet ("The attackers combined the ClickFix phishing technique, multi-stage loaders...") and the `campaigns` tag in the `TryDetectThis2.0` Community comment (`TA571, ClearFake, ClickFix`) confirm the phishing technique is `ClickFix`.

![](../../assets/Pasted%20image%2020261112052251.png)

## Task 3.2
What is the name of the platform that was used to redirect a user to malicious servers?

**Answer**: `Discord`

**Why?**
As detailed in both community comments (`Discord Invite Hijacking: How Fake Links Are Delivering Infostealers`) and the Check Point Research report, the attackers exploited expired or deleted invite links on `Discord` to redirect users to malicious servers.

![](../../assets/Pasted%20image%2020261112052427.png)
