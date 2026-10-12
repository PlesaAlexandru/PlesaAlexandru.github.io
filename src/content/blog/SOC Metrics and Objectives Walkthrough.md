---
title: SOC Metrics and Objectives Walkthrough
description: This room from TryHackMe describes how exactly a SOC team operates, the workflow and other practices. What "arsenal" do they have and so on.
pubDate: 2026-10-11
categories:
  - SOC
  - Walkthrough
  - TryHackMe
tags:
  - TryHackMe
  - Walkthrough
  - Rooms
---
---

## Core Metrics:

Goal of SOC is to protect C.I.A of the the company. The L1 analysts' role in this process is to reliably report True Positives to the higher level, to L2. This leads us to the first four metrics:  

|Metric|Formula|Measures|
|---|---|---|
|Alerts Count|`AC = Total Count of Alerts Received`|Overall load of SOC analysts|
|False Positive Rate|`FPR = False Positives / Total Alerts`|Level of noise in the alerts|
|Alert Escalation Rate|`AER = Escalated Alerts / Total Alerts`|Experience of L1 analysts|
|Threat Detection Rate|`TDR = Detected Threats / Total Threats`|Reliability of the SOC team|

## Alerts Count

This metric is counting the overall number of alerts, if a SOC analyst have 80 alerts, can be overwhelming but also 0 alerts could mean a SIEM problem, which needs attention. **5 to 30 alerts per day per L1 analyst is a good metric**.  

## False Positive Rate
![444](Pasted%20image%2020261012164114.png)
  
If you have around 94% TP could be a sign of not enough tunning, with more noise, analysts tend to become less vigilant and more likely to miss the threat and treat all alerts just like "yet another spam". A False Positive rate of **0% is an unachievable ideal, but 80% or higher is a serious problem**, usually fixed by a tool and detection rules tuning, often called "False Positive Remediation".

## Alert Escalation Rate

L2 analysts rely on L1 to filter out the noise and escalate only the actionable, threatening alerts. At the same time, as L1, you don't want to be overconfident and triage alerts you do not fully understand without a senior support. The alert escalation rate comes in handy to evaluate how experienced and independent the L1 analysts are and how often they decide to escalate the alert. It is usually **aimed to be below 50%, or even better below 20%**.
  

## Threat Detection Rate
![444](../../assets/Pasted%20image%2020261012164725.png)

Imagine that out of six attacks for 2025, your SOC team detected and prevented four attacks, missed the fifth because of the broken detection rule, and the sixth because one of the L1 analysts misclassified the breach as False Positive. The resulting metric is TDR = 4 / 6 = 67%, and this is a very bad result. The threat detection rate **should always be at 100%** since every missed threat can have devastating consequences, such as ransomware infection and data exfiltration.

## QUESTIONS:

**Is zero alerts for one month a good sign for your SOC team? (Yea/Nay)**  
Answer: Nay, it is a huge red flag to have 0 alerts (TP or FP) for a whole month.  

**What is the False Positive Rate if only 10 out of 50 alerts appear to be real threats?**  
Answer: 80%, 10/50 = 0.2 * 100 = 20% , 100-20=80%

## Triage Metrics

We have to remember that an alert is not enough to stop an attack. The requirements to ensure a quick detection and remediation of the threat are commonly grouped into a **Service Level Agreement (SLA)** - a document signed between the internal SOC team and its company management, or by the managed SOC provider (MSSP) and its customers. The agreement usually requires quick threat detection (measured by **MTTD**), timely alert acknowledgement by L1 analysts (measured by **MTTA**), and finally, prompt response to the threat, like isolating the device or securing the breached account (measured by **MTTR**):  

![](../../assets/Pasted%20image%2020261012192431.png)

## Reference Table

Note that different teams might have different definitions or formulas for the SOC metrics, depending on what they want to measure. For this and the following tasks, please use the illustration above and the reference table below to answer the questions.  

|Metric|Common SLA|Description|
|---|---|---|
|SOC Team Availability|24/7|Working schedule of the SOC team, often Monday-Friday (8/5) or 24/7 mode|
|Mean Time to Detect (MTTD)|5 minutes|Average time between the attack and its detection by SOC tools|
|Mean Time to Acknowledge (MTTA)|10 minutes|Average time for L1 analysts to start triage of the new alert|
|Mean Time to Respond (MTTR)|60 minutes|Average time taken by SOC to actually stop the breach from spreading|

## QUESTIONS: 

**Imagine a scenario where the SOC team receives a critical alert on Saturday.**  
**If the team works 8/5, on which day of the week will they acknowledge the alert?**  
Answer: Monday

**Imagine a scenario where an employee was lured into running data stealer malware.**  
  1. **The SOC team received the "Connection to Redline Stealer C2" alert after 12 minutes.**  
  2. **One of the L1 analysts on shift moved the alert to In Progress 10 minutes later.**  
  3. **After 6 minutes, the alert was escalated to L2, who spent 35 minutes cleaning the malware.** 
**Provide the MTTD, MTTA, and MTTR via comma as your answer (e.g. 10,20,30).**    

MTTD (Mean Time to Detect): 12;  
MTTA (Mean Time to Acknowledge): 10;  
MTTR (Mean Time to Response): 10+35+6 = 51;  
Answer: 12,10,51

## Improving Metrics

Metrics were made so SOC workflow can be efficient and, therefore, make the attacks less successful. They are also used to monitor performance. So how can you improve the metrics?

![](../../assets/Pasted%20image%2020261012193401.png)

## Reference Table

| Issue                                     | Recommendations                                                                                                                                                                                                                                              |
| ----------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| False Positive Rate  <br>over 80%         | **Your team receives too much noise in the alerts. Try to:**  <br>  <br>1. Exclude trusted activities like system updates from your EDR or SIEM detection rules  <br>2. Consider automating alert triage for most common alerts using SOAR or custom scripts |
| Mean Time to Detect  <br>over 30 min      | **Your team detects a threat with a high delay. Try to:**  <br>  <br>1. Contact SOC engineers to make the detection rules run faster or with a higher rate  <br>2. Check if SIEM logs are collected in real-time, without a 10-minute delay                  |
| Mean Time to Acknowledge  <br>over 30 min | **L1 analysts start alert triage with a high delay. Try to:**  <br>  <br>1. Ensure the analysts are notified in real-time when a new alert appears  <br>2. Try to evenly distribute alerts in the queue between the analysts on shift                        |
| Mean Time to Respond  <br>over 4 hours    | **SOC team can't stop the breach in time. Try to:**  <br>  <br>1. As L1, make everything possible to quickly escalate the threats to L2  <br>2. Ensure your team has documented what to do during different attack scenarios                                 |
## QUESTIONS:

**What is the highest acceptable False Positive Rate for SOC teams?**  
Answer: 80%  

**Should all SOC roles work together to keep metrics improving? (Yea/Nay)**  
Answer: Yea  

## Practice
For this lab, imagine yourself as a SOC manager receiving different complaints related to the SOC team. Open the attached site by clicking the **View Site** button and try to improve SOC metrics across three scenarios by correctly assigning improvement tasks from the list. Once completed, claim the flags and answer the task questions!

## QUESTIONS:

**What flag did you get after completing the first scenario?**  
![](../../assets/Pasted%20image%2020261012195012.png)
Answer:THM{mttr:quick_start_but_slow_response}

**What flag did you get after completing the second scenario?**  
![](../../assets/Pasted%20image%2020261012194931.png)
Answer: THM{mttd:time_between_attack_and_alert}  

**What flag did you get after completing the third scenario?**  
![](../../assets/Pasted%20image%2020261012194800.png)
Answer: THM{fpr:the_main_cause_of_l1_burnout}  