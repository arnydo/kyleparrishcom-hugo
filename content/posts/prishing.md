---
title: "Don't Say It...Prishing?"
date: 2025-07-23T10:00:00-05:00
slug: prishing-printer-phishing
draft: false
tags: ["cybersecurity", "printers", "phishing", "tech", "pentesting", "red team"]
categories: ["Security"]
description: "A red team assessment tale of turning multifunction printers into phishing delivery systems, and how I coined the term 'prishing' for printer-based social engineering attacks."
author: "Kyle Parrish"
cover:
  image: images/printer-touchscreen.png
  hiddenInList: false
---

# The Attack

During a recent internal red team assessment for a client, I found myself in familiar territory—established inside their network through a Citrix instance, carefully proxying commands through a covert tunnel to avoid detection.
What started as routine reconnaissance would lead me to discover an attack vector I'd never seen exploited in the wild: turning multifunction printers into phishing delivery systems.
I jokingly called it "prishing"—printer phishing, a play on other phishing techniques like "quishing" (QR code phishing) and "vishing" (voice phishing).

The assessment began like many others. I had established a tunnel from the client's Citrix environment back to my remote machine, allowing me to proxy commands through the Citrix instance to stay under the radar. Rather than perform noisy network scans that might trigger alerts, I decided to gather intelligence about network hosts through DNS enumeration.

With hundreds of host records collected, the hosts containing `-MFP-` (CONTOSO-MFP-01) immediately caught my attention.
These were clearly multifunction printers—devices that in my experience often run with default credentials and minimal security hardening.

## The Printer Vulnerability

![Printer Web Interface Login](/images/printer-login.png)

I looked up the model number for the MFPs and quickly found the factory [default credentials](https://www.thehacker.recipes/web/config/default-credentials): `administrator` and `password`. 
Browsing to the first printer's web interface at `http://10.120.2.105`, I was greeted with a login prompt.
The default credentials worked without any resistance.

Once inside the printer's administrative interface, I discovered two critical features that would make this attack possible:

1. **Print Logs**: A detailed history of everyone who had printed to the device over the past seven days, complete with usernames, document names, and timestamps.
2. **Print Upload**: A feature allowing users to upload documents via the UI for printing without needing to configure the printer on their local machine

Note: The printer UI images represent a fictional printer interface, but the features are real.

![Print Logs](/images/printer-logs.png)

![Printer Upload Feature](/images/printer-upload.png)

The pieces of a social engineering attack were falling into place.

## Crafting the Prishing Campaign

During my earlier reconnaissance, I had observed that this organization used [KnowBe4](https://www.knowbe4.com/) for security awareness training—a platform I had administered in previous roles. This gave me insight into the psychology of their users and their familiarity with security training certificates.

I decided to exploit this familiarity by creating fake KnowBe4 certificates congratulating users for "6 Months with Zero Phish Failures!." Each certificate would include the target's name and a QR code that supposedly allowed the recipient to redeem a $10 gift card to the company store.

![knowbe4-certificate](/images/knowbe4-certificate.png)

The technical implementation was more sophisticated than it might appear. I created a custom landing page hosted on [Google Cloud Run](https://developers.google.com/learn/pathways/cloud-run-serverless-computing) at a lookalike domain: `knowbe4.contosofinancial.com` (similar to their legitimate domain `contosolending.com`).

This site:

- Generated [Microsoft Device Codes](https://learn.microsoft.com/en-us/entra/identity-platform/v2-oauth2-device-code) to simulate verification codes using [Token Tactics V2](https://github.com/f-bader/TokenTacticsV2)
- Redirected users to legitimate [Microsoft Device Code login](https://microsoft.com/devicelogin) pages
- Captured successful authentications and displayed a congratulatory message
- Sent real-time notifications via [ntfy.sh](https://ntfy.sh/) when QR codes were scanned or logins completed

Each certificate was personalized using names extracted from the printer logs, and each QR code contained a unique URL with the user's email address as a query parameter for tracking purposes.

## The Print and Deploy Strategy

Rather than trying to add the printers to my Citrix instance (which I didn't have permission to do anyway), I used the printers' upload feature to deploy the phishing documents. I uploaded personalized certificates for each user who had printed to the MFPs recently.

![Printer Upload](/images/printer-upload.png)

The beauty of this attack vector was its physical nature. The certificates were printed and left in a stack on each printer, appearing to be legitimate company communications. Users would naturally grab their "certificate" when collecting other print jobs.

## The Results: Success and Lessons Learned

The notifications started rolling in over the next couple days.
Users began scanning the QR codes, and I could track their progress:

- **QR Code Scans**: Multiple users scanned the codes
- **Landing Page Visits**: Several users visited the phishing site multiple times
- **Complete Compromise**: One user completed the entire authentication flow

![ntfy notification](/images/ntfy-qr.png)

I was in the middle of a conversation with my wife when ntfy dinged on my phone (along with a random vibrate pattern and notification sound...so I would definitely notice it) and she saw my eyes light up.
She quietly smiled and walked away, knowing I had just scored a successful compromise.

![ntfy token notification](/images/ntfy-token.png)

This single access token allowed me to:

- Extract detailed information about the Azure environment for [BloodHound analysis](https://specterops.io/bloodhound-community-edition/)
- Download over 4,000 emails from the user's inbox
- Send emails on behalf of the compromised user
- Access Teams messages, OneDrive files, and other Microsoft 365 resources

## The Second Attack: Screen Hijacking

![Printer Touchscreen with Fake Message](/images/printer-touchscreen.png)

Energized by the first campaign's success, I launched a second attack using another printer feature I'd discovered: the ability to display custom messages on the device's 8-inch color touchscreen.

I configured a message that read: "Authentication Required to Release Print." This message appeared whenever users initiated print jobs or encountered errors, complete with a QR code leading to a similar Device Code phishing site.

While this second attack generated several QR code scans and landing page visits, no users completed the full authentication process. The context of "print held" may have seemed less urgent or legitimate than a congratulatory certificate.

## The Broader Implications of Prishing

This attack demonstrates several concerning realities about modern office environments:

### Physical Access Through Digital Means
Even in security-conscious organizations, printers often remain overlooked attack vectors. Their physical presence in common areas makes them perfect platforms for social engineering.

### Trust in Printed Materials
There's an inherent trust in physical documents that doesn't exist with digital communications. Users who might scrutinize an email-based phishing attempt may readily accept a printed certificate.

### Default Credential Persistence
Despite years of security awareness training, network devices—especially "utility" devices like printers—frequently retain default credentials.

### Multi-Vector Attack Potential
The combination of digital exploitation (compromising the printer) and physical social engineering (printed documents) creates a powerful attack vector that's difficult to defend against with traditional security measures.

## Defending Against Prishing Attacks

### For Organizations:
- **Change Default Credentials**: Immediately change default passwords on all network devices
- **Network Segmentation**: Isolate printers and other IoT devices from critical network segments
- **Physical Security**: Monitor printer areas and remove unclaimed documents promptly
- **User Education**: Train employees to verify QR codes and URLs before scanning or visiting

### For Security Teams:
- **Asset Discovery**: Regularly audit network devices, including printers and other "utility" devices
- **Configuration Management**: Implement centralized management for printer security settings
- **Monitoring**: Log and monitor printer access and configuration changes

## Conclusion

The term "prishing" might sound humorous, but the attack vector it represents is no joke. By combining traditional network exploitation with physical social engineering, attackers can bypass many digital security controls and leverage the inherent trust users place in printed materials.

This assessment reinforced an important lesson: in cybersecurity, we must think beyond the traditional attack vectors. Sometimes the most effective compromise comes not through sophisticated malware or zero-day exploits, but through creative combinations of old-school social engineering and overlooked network devices.

The next time you're conducting a security assessment, don't overlook the humble office printer. It might just be your gateway to the crown jewels—or in this case, the pathway to comprehensive domain compromise through a stack of fake certificates and a few QR codes.