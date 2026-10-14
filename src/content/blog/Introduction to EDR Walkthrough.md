---
title: Introduction to EDR Walkthrough
description: Learning the fundamentals of EDR and explore its features and working.
pubDate: 2026-10-13
categories:
  - SOC
  - Walkthrough
  - TryHackMe
tags:
  - TryHackMe
  - Walkthrough
  - Rooms
image: /image/introd-to-edr.png
---
---
## What is an EDR?

The increased number of threats and usage of digital devices demands an solution for all endpoint devices. Endpoint Detection and Response (EDR) is a security solution that offers deep-level protection for endpoints. No matter where the endpoints are, the EDR will make sure they are monitored constantly and threats are detected.  

Several other EDR solutions are available in the market. Their underlying architecture is mostly similar, but the features may vary. Let's take a look at the core features of an EDR. We will be using the screenshots from CrowdStrike Falcon EDR to demonstrate the core features of EDR.  
## Features of EDR
There are three main features of an EDR, which can also be referred to as the three pillars of an EDR solution.
![](../../assets/Pasted%20image%2020261013154157.png)

## Visibility

The level of visibility is a TOP priority for a SOC Analyst. And here comes the EDR, the level of visibility is amazing. The analyst can see the whole process tree with a complete activity timeline of the sequence of actions.

| Process Modifications | Registry Modifications | File And Folder Modifications | User Actions | And Much More |
| --------------------- | ---------------------- | ----------------------------- | ------------ | ------------- |

The following screenshot shows graphical representation of a process tree.
![](../../assets/Pasted%20image%2020261013171039.png)

## Detection

It incorporates signature-based detections as well as behavior-based detections, such as unexpected user activities. With modern machine learning capabilities, it identifies any deviation from the baseline behavior and instantly flags it. It can also detect fileless malware that resides in memory. It also allows us to feed custom IOCs for threat detections.

The following screenshot shows a dashboard of all the detections happening on the different endpoints. Each detection is represented by a row with different fields including the severity of the detection, time, triggering file, hostname, username, and more. The Tactic via Technique field maps the detection with MITRE. Any detection when clicked will show us rich details which helps a SOC analyst during the analysis.

![](../../assets/Pasted%20image%2020261013172016.png)

## Response

EDR also empowers analysts to take action on detected threats. These actions can be taken at any endpoint within the central EDR console. Imagine getting a detection on the EDR with full-fledged details on when, where, and what happened, and you have to opt for the best possible action for that detection. As an analyst, you may decide to isolate a complete endpoint, terminate a process, or quarantine some files. You can also connect to the host remotely and execute actions independently. You can do this all from within the EDR console.  

The following screenshot shows the actions available that can be taken on the host after connecting to it.
![](../../assets/Pasted%20image%2020261013173018.png)

There are several other actions that the analyst can take during the investigation.  

## QUESTIONS:

**Which feature of EDR provides a complete context for all the detections?**  
Answer: Visibility  

**Which process spawned sc.exe?**  
Answer: cmd.exe, just follow the process tree from the graphical representation of a process tree.  

## Beyond the Antivirus
Before we proceed to the **How** behind the EDR's working, let's answer a very common question that comes to mind when we hear about the EDR solution.  
## Why do we need an EDR when we already have an Antivirus (AV) on the endpoints?  

Both of these security solutions have the same motive of protecting the endpoint on which they are installed. However, both differ in the level of protection they provide. Let's assume an airport is an endpoint that needs protection. One layer of protection is to check the passports of people when they pass through immigration. The immigration check (AV) monitors incoming people and matches their passports with a list of known criminals in its database. If there is a match, the entry is blocked and caught.   

Sounds like a smart protection, right?  

But, what happens if somebody who has never been identified as a criminal in the past and has an innocent personality tries to come in? The immigration check will let them in. Now, what if this innocent person were a professional thief trained to evade basic security? The airport has an undetected threat inside!  

This is where an EDR comes in.  

As a summary of the scenario, having 2 solutions is the way to go, in cases similar to no records or no identification EDR shines.  

An EDR in this analogy would be the security officers stationed inside the airport (endpoint). These security officers would constantly monitor the security cameras and motion sensors in the airport (endpoint). Compared to the immigration check (AV), the security officers (EDR) enhance the protection of the airport (endpoint) through CCTV monitoring and motion sensors. Even if somebody manages to evade the immigration check, the security officers will constantly be monitoring their actions, such as:  

- Are they roaming close to restricted areas?
- Is their behaviour suspicious?
- Are they leaving their bags somewhere unattended?

You get the point. EDR is an added layer of protection and a more adaptable one. Which is a great tool in the arsenal of any SOC Analyst. Now, let's examine an advanced malicious activity step by step on an endpoint and compare the response of an AV and EDR at each stage.

## Scenario Breakdown

- **Step #1:** A user receives a phishing email with a Word document embedded with a malicious macro (VBA script)
- **Step #2:** The user downloads the document and opens it
- **Step #3:** The malicious macro is silently executed, and it spawns PowerShell
- **Step #4:** The malicious macro runs an obfuscated PowerShell command to download a sophisticated second-stage payload
- **Step #5:** The payload is injected into a legitimate svchost.exe
- **Step #6:** The attacker gains remote access to the system
![](../../assets/Pasted%20image%2020261013175955.png)

|   |   |   |
|---|---|---|
|**Attack Steps**|**AV's Response**|**EDR's Response**|
|Step #1|Does nothing if the downloaded file has no previous signature in the database|Logs the file download activity and monitors it|
|Step #2|Does nothing upon the opening of the document since winword.exe is a legitimate utility|Records the execution of winword.exe and keeps monitoring|
|Step #3|Does nothing if the executed macro has no previous signature|Detects and flags the macro execution due to the unusual parent-child relationship of winword.exe and PowerShell.exe processes|
|Step #4|Typically, AVs will not detect obfuscated PowerShell scripts|Flags the obfuscated script execution|
|Step #5|Will not flag malicious injection into svchost.exe since it does not monitor the memory injections|Detects Process Injection in svchost.exe|
|Step #6|Lacks Network Level Visibility|Flags the unexpected behaviour of svchost.exe, making an outbound connection|
|Final Action|May be marked as clean|Generates an alert with the full attack chain and enables the analyst to take actions from within the EDR|
**Note:** Some modern AVs may have more enhanced visibility and detection. However, an EDR is ahead as it levels up the detection and response in an endpoint.

## QUESTIONS:

**In the given analogy, what presents an AV?**  
Answer:  Immigration Check  

**Which legitimate process was hijacked by the attacker in the scenario?**  
Answer: svchost.exe, mention on Step 6 of the given scenario.  

**Which security solution might mark this activity as clean?**  
Answer: Antivirus  

## How an EDR works?

- How does an EDR manage to provide this much visibility of the endpoints?
- How can it detect advanced threats?
- How can a few clicks eradicate the threat from an endpoint?

![](../../assets/Pasted%20image%2020261013185314.png)

## Agents

Sometimes Agents are called sensors, are installed on the endpoint and can do some basic signature-based and behaviour-based detections by themselves and send them to the EDR console. But their role is to sit and collect data for the console from activities on the endpoint. 

## EDR Console

All the detailed data sent by the EDR agents is correlated and analyzed through complex logic and machine learning algorithms. The threat intelligence information is matched with the collected data. The EDR is just like the brain connecting all the dots. These dots connect to form a detection, often called an alert.
![](../../assets/Pasted%20image%2020261013191031.png)
## What happens after Detection?

When a detection comes, it's a SOC analyst's responsibility to acknowledge the alert and prioritize it. The prioritization is made easy by the EDR itself. It gives severities to all the alerts (Critical, High, Medium, Low, Informational). The alert with the highest severity is investigated as a priority. For the investigation, once the alert is clicked, the analyst can see all the details of the detection. This includes any files executed, processes executed, network connection attempts, registry modifications, and much more. Based on the available data, the analyst's job is first to use their expertise to determine if the alert is a false positive or a true positive. In case of a true positive, the analyst can take actions from within the EDR console.
## EDR with Other Tools

As a SOC Analyst, it is essential to understand that although a standalone EDR provides enough information to detect and respond to threats in an endpoint, it works alongside other security solutions to form a larger security ecosystem. Within a network, you will see Firewalls, DLPs, Email Security Gateways, IAMs, EDRs, and other security solutions protecting the different components of the network. To minimize the effort and maximize the efficiency, all these security solutions are integrated with a SIEM solution that becomes the central point of investigation for the analysts. We will also discuss the SIEM solution in detail in the upcoming rooms of this module.

## QUESTIONS:

**Which component of the EDR is responsible for collecting telemetry from the endpoints?**  
Answer: Agent  

**An EDR agent is also known as a?**  
Answer: Sensor  

## EDR Telemetry

Data collected by agents is called **Telemetry**. 

![](../../assets/Pasted%20image%2020261013200222.png)
![](../../assets/Pasted%20image%2020261013200222.png)

## Collected Telemetry

Some telemetry examples:

- **Process Executions and Terminations**  
    It keeps track of all the running and idle processes, which helps to identify suspicious child-parent process relationships, suspicious executables initiating a process, malware payload, etc.
- **Network Connections**  
    All the endpoints' network connections are monitored, which helps identify any connection to a C2 server, unusual port usage, data exfiltration, or lateral movement within the network.
- **Command Line Activity**  
    It captures all the commands executed on the endpoints in CMD, PowerShell, etc., which helps to identify malicious command execution, obfuscated PowerShell script executions, which are often missed by a traditional antivirus.
- **Files and Folders Modifications**  
    Threat actors modify different files and folders during data staging, ransomware executions, and malicious file dropping. The EDR tracks this.
- **Registry Modifications**  
    The registry is a goldmine of information about the configurations in a Windows system. There are many registry modifications that occur during a malicious activity, and most of these are monitored by the EDR.


## QUESTIONS:

**Which telemetry data helps in detecting C2 communications?**  
Answer: Network Connections  

**Where are the configuration settings of a Windows system primarily stored?**  
Answer: Registry  

## Detection

![278](../../assets/Pasted%20image%2020261013230826.png)

Based on the telemetry received from the endpoints, some advanced detection techniques are applied to this data. Some of these techniques include:

- **Behavioral Detection**  
    Instead of just matching the signatures with known threats, it observes the complete behavior of a file. Advanced threats craft their malware to look clean and use legitimate processes to carry out their attack. EDR catches this behavior.  
    **Example:** A process winword.exe spawning PowerShell.exe will be flagged by the EDR due to the behavior. A Word document spawning a PowerShell is an unusual parent-child relationship.
- **Anomaly Detection**  
    With time, EDR understands the baseline behavior of the endpoints. Any activity that deviates from this behavior will be flagged. During any malicious activity, the endpoint's behavior deviates from normal. EDR picks it up. Sometimes, this can generate false positives as well. However, with the full context it gives, the analyst can identify its legitimacy.  
    **Example:** On one of the endpoints, a process modifies an auto-start registry key, which is not a common behavior on the endpoint.
- **IOC matching**  
    EDRs have some strong threat intelligence field integrations. Except for zero-day attacks, most of the attacks have indicators published in the threat intelligence feeds. EDR flags any activity that matches any known IOC.  **Example:** A user downloads a file that drops an executable. The executable is often used in a specific attack. The hash of this executable will get matched with the threat intelligence feed and instantly flagged by the EDR.
- **MITRE ATT&CK Mapping**  
    Any activity flagged by the EDR is not only marked as malicious or suspicious but also mapped with the MITRE Tactic and Technique (attack stage) that the particular activity was on. This proves to be very helpful for the analysts.  
    **Example:** If the EDR flags the creation of a scheduled task for any reason, it will likely map this activity to the following:
    - Tactic: Persistence
    - Technique: Scheduled Task/Job

- **Machine Learning Algorithms**  
    Advanced threat actors try to evade defenses as much as possible, and their activities may sometimes bypass advanced detection techniques. Modern EDRs have machine learning models trained by a large dataset of normal and malicious behaviors. This can detect complex patterns of an attack.  
    **Example:** Attacks in which the individual actions are not inherently malicious, but the ML algorithm identifies the whole chain of activities as malicious. Fileless attacks and multi-staged intrusions are often detected through this.
## Response

The next step after any detection is the response. EDR offers both automated and manual responses. You can make policies to block known malicious behaviors automatically. However, manual response gives you a wide range of response capabilities. Let's discuss some of them.

- **Isolate Host**  
    During any malicious activity on an endpoint, you can isolate that endpoint from the network through EDR. This is a very effective function for containing malicious activity. Most attacks start from a single endpoint and move laterally to other endpoints to compromise the whole network. Isolating the infected endpoint on time can stop this from happening.
- **Terminate Process**  
    Not every malicious activity requires host isolation. Some hosts run the core business operations, and isolating them can cause more loss than the malicious activity. In such cases, terminating a process is enough to neutralize the malicious activity. The analysts get this option in the EDR. They can terminate any process at any time. This action should be taken consciously since terminating a legitimate process can disrupt the endpoint.
- **Quarantine**  
    If a malicious file comes into the endpoint, it can be quarantined. Quarantine ensures that the file is moved to an isolated location where it can not be executed. The analysts can then review the file to restore or permanently remove it. 
- **Remote Access**  
    Analysts can also remotely access the shell of any endpoint. This is often done when the EDR's built-in response is not enough to take action on a specific activity. Through remote access, analysts can gain deeper visibility into the system or take custom actions within the endpoints. The analysts can also run scripts or collect their desired data from the host through remote access.  
    Below is an example of CrowdStrike Falcon EDR's RTR (Real Time Response) console, which allows analysts to remotely access the shell of any endpoint and run commands and scripts.

![](../../assets/Pasted%20image%2020261013230932.png)

## Artefacts Collection  

Sometimes, the analysts may need to extract some data from the endpoints for detailed forensic investigation or reporting for legal actions. Analysts can extract important artefacts from the endpoints without physically accessing the device. The most commonly extracted artefacts include:  
    - Memory Dump
    - Event Logs
    - Specific Folder Contents
    - Registry Hives

The Detection and Response capabilities of EDR solutions differ from those of traditional endpoint protection solutions. Modern EDRs are introducing more advanced detection techniques.  

## QUESTIONS:

**Which feature of the EDR helps you identify threats based on known malicious behaviours?**  
Answer: IOC matching   

## Scenario

You are a SOC analyst at TECH THM with access to the EDR console, having multiple medium and high-severity detections. Your task is to perform triage on each detection using the available information in the EDR and answer a series of questions related to these detections.


## QUESTION:

**Which tool was launched by CMD.exe to download the payload on DESKTOP-HR01?**  
![](../../assets/Pasted%20image%2020261013235141.png)
Answer: CURL.exe  

**What is the absolute path to the downloaded malware on the DESKTOP-HR01 machine?**  
![](../../assets/Pasted%20image%2020261013235154.png)
Answer: C:\Users\Public\install.exe  

**What is the absolute path to the suspicious syncsvc.exe on the WIN-ENG-LAPTOP03 machine?**  
![](../../assets/Pasted%20image%2020261013235612.png)
Answer: C:\Users\haris.khan\AppData\Local\Temp\syncsvc.exe  

**On which URL was the exfiltration attempt being made on WIN-ENG-LAPTOP03?**  
![](../../assets/Pasted%20image%2020261013235830.png)
Answer: https[:]//files-wetransfer[.]com/upload/session/ab12cd34ef56/dump_2025.dmp  

**What was UpdateAgent.exe labelled by Threat Intel on DESKTOP-DEV01?**  
![](../../assets/Pasted%20image%2020261013235904.png)
Answer: Known internal IT utility tool  



