---
title: Phishing Emails in Action Walkthrough
description: Different indicators of phishing attempts by examining actual phishing emails.
pubDate: 2026-10-20
categories:
  - SOC
  - Walkthrough
  - TryHackMe
tags:
  - TryHackMe
  - Walkthrough
  - Rooms
image: /image/phish-in-action.png
---
---
## Summary

Not much to note about this room, it has a lot of phishing indicators to better put into perspective the phishing attacks, but as a side note, I couldn't note any more details due to the fact that pictures are made to explain the indicators. It is a theory based room with a lot of phishing examples and an well structured summary. 

---

**Learning Objectives**
- Identify common social engineering tactics used in phishing
- Analyze red flags contained within phishing emails
- Detect link manipulation and tracking pixels
- Deconstruct credential harvesting and attachment manipulation

## Cancel Your Order

**Phishing Techniques Used**

- **Spoofed email address:** Mimicking a trusted service to gain immediate credibility
- **URL shortening:** Using redirection services to hide the true destination of a link
- **Branded HTML:** Impersonating legitimate corporate imagery to create a sense of authenticity

### First Observations

1. **Attention-grabbing subject line:** The subject line uses a fake transaction to create a sense of urgency, prompting you to react in haste
2. **From address:** This is an immediate red flag as the sender details `service@paypal.com` do not match the actual address `gibberish@sultanbogor.com`
3. **To address:** This is an unusual email recipient address and not a normal Yahoo domain

![](../../assets/Pasted%20image%2020261020160034.png)

### Email Body Analysis

The email is designed to appear as a legitimate email from PayPal. There aren't any attachments associated with this email, and the only interactive element in this email is the `Cancel the order` button.
![](../../assets/Pasted%20image%2020261020160426.png)

### Button Investigation

 The `Cancel the order` button leads to the shortened URL.
 As a rule of thumb, you should never interact with buttons or links without first confirming exactly where they lead.
 ![](../../assets/Pasted%20image%2020261020160801.png)

After checking WhereGoes:
![](../../assets/Pasted%20image%2020261020161046.png)

## Track Your Package

**Phishing Techniques Used**
- **Spoofed email address:** Mimicking a trusted distribution center to gain immediate credibility
- **Pixel tracking:** Embedding invisible images to notify the sender when the email is opened
- **Link manipulation:** Masking a malicious destination with a fraudulent tracking number
### First Observations

1. **Subject line:** The subject line uses a fake tracking number to create a sense of urgency, prompting the recipient to click to see their package status
2. **From address:** This is an immediate red flag, as the display name **Distribution Center** does not match the actual sender address `contact@beginpro.club`
3. **Hyperlink:** The link in the email body matches the subject line, although we do not know where it directs to just yet

![](../../assets/Pasted%20image%2020261020161424.png)

Note that in this email sample, Yahoo blocked the images from automatically loading. Any guesses as to why? Typically, you can hover the cursor over a link to see where the link is pointing to, but in this sample, that technique won't work because Yahoo disabled links in the email.  We can look at the raw source code for the email and find out.

### Hyperlink Tracking

Here is an image file named `Tracking.png`. These trackers send information back to the spammer's server. There are several reasons spammers embed tracking pixels (very small images) into their emails, and now we can understand why Yahoo automatically blocked the images in this email. Many email providers do the same.

![](../../assets/Pasted%20image%2020261020161708.png)

## Download Document Here

**Phishing Techniques Used**

- **Artificial urgency:** Creating a narrow window for action to create a sense of urgency
- **Brand impersonation:** Layering trusted brands, like Microsoft and Adobe, to build a false sense of security
- **Link redirection:** Using a chain of URLs to hide the final malicious destination from basic email filters
- **Credential harvesting:** Deploying a fake login portal to capture and exfiltrate usernames and passwords

### First Observation

1. **Send date:** The email was sent on Thursday, July 15th, 2021
2. **Expiration date:** A sense of urgency is introduced in this email. Notice that the link to download the fax document expires on the same day
3. **Download Document Here button:** There is an action to perform. In this case, a button to download the fax

![](../../assets/Pasted%20image%2020261020162308.png)

### Download Document Here
After clicking the `Download Document Here` button, the user is redirected to a landing page that mimics a legitimate OneDrive share. Interacting with the buttons on this page triggers a second redirection to a site impersonating Adobe, though several critical red flags are apparent. The URL is highly suspicious, and the provided directions are nonsensical. These inconsistencies lead to the final trap: a credential-harvesting portal that asks the victim to sign in with their email provider to view the document.


![](../../assets/Pasted%20image%2020261020162323.png)

### Logging In
 The victim attempts to log in using Outlook. Even if they had entered valid credentials, the result would be the same: a generic error message.

![](../../assets/Pasted%20image%2020261020162336.png)

## Your Account Is on Hold

**Phishing Techniques Used**

- **Spoofed email address:** The sender's display name is set to **Netllx billing** to appear legitimate
- **Sense of urgency:** Using a suspended account notification to pressure the victim into acting quickly
- **Brand impersonation:** Utilizing HTML templates and logos to mimic Netflix billing
- **Poor grammar and typos:** Noticeable misspellings of Netflix
- **Attachments:** Using a file attachment rather than a direct link to hide the malicious URL
### First Observations

1. **Email subject:** The receiver's ID was suspended and must act quickly
2. **From address:** The display name does not match the user and domain
3. **Brand impersonation:** The email utilizes rendered HTML to impersonate Netflix

![](../../assets/Pasted%20image%2020261020163033.png)

### Email Body and Attachment Analysis

The attachment contains an embedded link titled `Update Payment Account` which directs to a URL not associated with a legitimate Netflix domain.

1. The use of an atypical phone number format is a red flag
2. Using a legitimate Netflix help center domain to build a false sense of trust

![](../../assets/Pasted%20image%2020261020163140.png)

## Your Recent Purchase

**Phishing Techniques Used**

- **Spoofed email address:** The sender's display name is set to **Apple Support**
- **Recipient is BCCed:** The victim is not directly sent the email
- **Urgency:** Relies on the use of **Action Required** and a fake purchase notification
- **Poor grammar and typos:** Noticeable spelling errors within the email header
- **Attachments:** The email contains a dot file (Microsoft Word Template), which is an unusual format for a receipt
### First Observations

1. **Email subject:** The recipient is told they must act quickly to resolve an unauthorized purchase, creating a false sense of urgency
2. **From address:** The display name, **Apple Support**, does not match the user and domain. There are also typos in the From and To addresses
3. **Blind Carbon Copy:** The recipient was not directly emailed, instead Blind Carbon Copied (BCC)
![](../../assets/Pasted%20image%2020261020163635.png)
### Analyzing the Attachment

The only content present is an attachment in the form of a `.dot` file. When the user interacts with the large image embedded in the document, they are redirected to a phishing site. Although the URL includes familiar terms like **apps** and **ios** to appear legitimate, its excessive length and complexity are strong indicators of a malicious redirection attempt.
![](../../assets/Pasted%20image%2020261020163646.png)

## Scheduled Shipment

**Phishing Techniques Used**

- **Spoofed email address:** The sender's display name is set to `DHL Express`
- **Brand impersonation:** Utilizing HTML templates and logos to mimic DHL
- **Attachments:** An Excel document that triggers executable code upon opening

### First Observations

1. **Email subject:** Gives the impression that DHL will be shipping a package
2. **From address:** The display name, **DHL Express**, does not match the user and domain
3. **Brand impersonation:** The HTML in the email body is designed to look like it is sent from DHL
![](../../assets/Pasted%20image%2020261020163922.png)

### Email Body and Attachment

While there isn't much depth to the email body, the meat of this phishing attempt lies in the attached `.xlsx` file. Opening the document reveals several glaring inconsistencies. First, the sender uses a German domain, the invoice is addressed to a city in India, yet the document content itself contains Mandarin. The document contains a single clickable link designed to move the attack to the next stage (Trying to pretend it is an translation to the file).

![](../../assets/Pasted%20image%2020261020164015.png)

**The Executable**
When the link within the Excel document is clicked, it attempts to download and execute a malicious payload named `regasms.exe`. As seen in the screenshot, this execution results in a system error. While the error prevents the payload from running in this environment, it indicates the attacker’s intent to execute code directly on the victim’s machine.

If successfully executed, the attacker could:

- **Establish Persistence:** Create a backdoor or scheduled task to maintain access after a reboot
- **Exfiltrate Data:** Steal sensitive files, credentials, or browser-stored passwords
- **Deploy Ransomware:** Encrypt the system and demand payment for recovery
![](../../assets/Pasted%20image%2020261020164227.png)
## Conclusion

## Q&A
<details>
<summary>Click to view TryHackMe Room Q&A</summary>

* **Who is listed as the **Merchant** in the email body?**  
  *Answer:* Amazing Stuff   

* **What root domain does the hyperlink in the above example point to?  
Be sure to defang the URL.**  
  *Answer:* devret[.]xyz

* **The attacker deployed a fake portal to capture and exfiltrate user credentials.  
What is this type of attack called?**  
  *Answer:* credential harvesting

* **  What is the actual sender email address hidden behind the `Netllx billing` display name?**  
  *Answer:* z99@musacombi[.]online

* **What does the acronym BCC stand for?**  
  *Answer:* Blind Carbon Copy

* **What is the file extension of the attachment?**  
  *Answer:* .dot

* **What is the name of the executable that the Excel attachment attempts to run?**  
  *Answer:* regasms.exe

</details>
