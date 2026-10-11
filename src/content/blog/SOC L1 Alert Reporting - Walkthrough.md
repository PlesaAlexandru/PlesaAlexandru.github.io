---
title: SOC L1 Alert Reporting - Walkthrough
description: "SOC L1 investigation walkthrough covering alert reporting, Basic lab with a simulated SIEM and 5 alerts with severities: High, Medium, Low"
pubDate: 2026-11-09
categories:
  - SOC
  - Walkthrough
tags:
  - TryHackMe
  - Walkthrough
  - Alert_Reporting
---
---
## Alert Funnel

Analysts receive alerts via SIEM, EDR or different Ticket Management solutions. Most of them are closed as False Positive or handled by an L1 Analyst, but the more important ones are sent to the L2 or higher based on the workflow. 

![](../../assets/Pasted%20image%2020261011151554.png)

## Alert Reporting

Before closing or passing the alert to L2, you might have to report it, based on the standard of the team, company etc. This is really important and especially for the true positive ones.

## Alert Escalation

When escalating, L2 reads the report which is a huge time saviour if done right.

## Communication

You may also need to communicate with other departments during or after the analysis. For example, ask the IT team if they confirm granting administrative privileges to some users or contact HR to get more information about the newly hired employee.


## QUESTIONS:

**What is the process of passing suspicious alerts to an L2 analyst for review?**
Answer: Alert Escalation

**What is the process of formally describing alert details and findings?**
Answer: Alert Reporting

## REPORTING GUIDE

Importance of the L1 Analyst reports are as following, not to be underestimated.

| **Alert Report Purpose**       | **Explanation**                                                                                                                                                   |
| ------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Provide context for escalation | - A well-written report saves lots of time for L2 analysts<br>- Also, it helps them quickly understand what happened                                              |
| Save findings for the records  | - Raw SIEM logs are stored for 3-12 months, but alerts are kept indefinitely<br>- As a result, it's better to keep all the context inside the alert, just in case |
| Improve investigation skills   | - If you can't explain it simply, you don't understand it well enough<br>- Report writing is a great way to boost L1 skills by summarising alerts                 |

## Report Format

Format typically uses the 5 Ws theory. An well structured report should answer the followings:
- **Who**: Which user logs in, runs the command, or downloads the file
- **What**: What exact action or event sequence was performed
- **When**: When exactly did the suspicious activity start and ended
- **Where**: Which device, IP, or website was involved in the alert
- **Why**: The most important W, the reasoning for your final verdict

Report sample:
![](../../assets/Pasted%20image%2020261011153245.png)

## QUESTIONS:

**According to the SOC dashboard, which user email leaked the sensitive document?**
![](../../assets/Pasted%20image%2020261011154600.png)
Answer: m.boslan@tryhackme[.]thm

This mail is really foundational due to the fact that the Data Leak is of the most important aspects in company, Data leaks are against CONFIDENCIALITY, first term in the foundational structure of the cybersecurity, CIA Triad. 

**Looking at the new alerts, who is the "sender" of the suspicious, likely phishing email?**
![](../../assets/Pasted%20image%2020261011154701.png)
Answer: support@microsoft[.]com

Huge red flags: 
- Microsoft doesn't use support@microsoft[.]com. They use an internal, custom service for managing tickets.
- Urgency, Huge numbers, step-by-step indications are also huge indicators of phishing, they are used to lure the user.
- 600% increase is unrealistic and never possible
- SPF / FAILED, DKIM / FAILED
- REPORT.rar, unprofessional name and not a common practice for support teams to send files in .rar extensions. They typically sent files as they are without grouping it into archived document.
- Report with no date, no notice



**Open the phishing alert, read its details, and try to understand the activity.**  
**Using the Five Ws template, what flag did you receive after writing a good report?**  

Answer: THM{nice_attempt_faking_microsoft_support}

## ESCALATION GUIDE:

You should escalate the alerts if:

1. The alert is an indicator of a major cyberattack requiring deeper investigation or DFIR
2. Remediation actions like malware removal, host isolation, or password reset are required
3. Communication with customers, partners, management, or law enforcement agencies is required
4. You just do not fully understand the alert and need some help from more senior analysts

To escalate the alert, in most cases, all you have to do is to **reassign the alert to the L2 on shift** and ping them in corporate chat or in person
![](../../assets/Pasted%20image%2020261011180235.png)

L2 analyst decides if it to an higher importance or not and decide the further actions.
It is generally fine for L1 to request senior support if something is unclear. Especially in your first months

![](../../assets/Pasted%20image%2020261011180543.png)


## QUESTIONS:

Who is your current L2 in the SOC dashboard that you can assign (escalate) the alerts to?
![](../../assets/Pasted%20image%2020261011182714.png)
**Answer**: E.Fleming

What flag did you receive after correctly escalating the alert from the previous task to L2?  
**Note:** Set the right assignee and add an intermediate verdict (TP or FP)

**Answer**: THM{good_job_escalating_your_first_alert}

Now, investigate the second new alert and provide a detailed alert comment.  
What flag did you get after escalating this alert according to the workflow?
![](../../assets/Pasted%20image%2020261011182842.png)
Process lineage analysis reveals that cmd.exe was spawned by C:\Users\Public\revshell.exe, which originated directly from the IIS worker process (C:\Windows\System32\inetsrv\w3wp.exe). This execution chain strongly suggests a web application compromise. Indicates post-exploitation reconnaissance targeting domain architecture and privileged groups.
Required escalation asap.

**Answer**: THM{looks_like_webshell_via_old_exchange}

## SOC Communication
 In the best scenario, the SOC team has its own **Crisis Communication** procedures - the guides and processes to help you and your teammates resolve the issues.
## Communication Cases
- **You need to escalate an urgent, critical alert, but L2 is unavailable and does not respond for 30 minutes.**  
    Ensure you know where to find emergency contacts. First, try to call L2, then L3, and finally your manager.
    
- **The alert about Slack/Teams account compromise requires you to validate the login with the affected user.**  
    Do not contact the user through the breached chat - use alternative contact methods like a phone call.
    
- **You receive an overwhelming number of alerts during a short period of time, some of which are critical.**  
    Prioritise the alerts according to the workflow, but inform your L2 on shift about the situation.
    
- **After a few days, you realise that you misclassified the alert and likely missed a malicious action.**  
    Immediately reach out to your L2 explaining your concerns. Threat actors can be silent for weeks before impact.
    
- **You can not complete the alert triage since the SIEM logs are not parsed correctly or are not searchable.**  
    Do not skip the alert - investigate what you can and report the issue to your L2 on shift or SOC engineer.
## Communication By L2

![](../../assets/Pasted%20image%2020261011183330.png)

## QUESTIONS:

S**hould you first try to contact your manager in case of a critical threat (Yea/Nay)?**
Answer: Nay, first L2->L3->Manager if not crisis procedure present

**Should you immediately contact your L2 if you think you missed the attack (Yea/Nay)?**
Answer: Yea, the sooner the better