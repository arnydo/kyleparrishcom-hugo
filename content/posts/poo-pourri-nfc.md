---
title: "PooPourri’s NFC Tag Flaw: A Cruise Discovery with Security Risks"
date: 2025-05-12T10:00:00-05:00
slug: poopourri-nfc-tag-security-risks
draft: true
tags: ["cybersecurity", "NFC", "PooPourri", "phishing", "tech"]
categories: ["Security"]
description: "Discover how a writable NFC tag in PooPourri’s Pocket Sprayer, found during a cruise, could lead to phishing attacks, and learn how to mitigate the risks."
---

# Overview

During a cruise, I picked up a travel-sized Poo~Pourri Lavender Vanilla Pocket Sprayer to keep the cabin fresh. While aboard, a sticker on the bottle caught my eye, prompting a scan with my phone. Using the NFC Tools app, I found the bottle’s NFC tag linked to pourri.com. Out of curiosity, I tested if the tag was writable—and it was. I easily overwrote the URL, exposing a security flaw that could let attackers redirect users to malicious sites. In this post, I’ll share my finding, outline how this vulnerability could be exploited, detail the technical risks, and propose fixes to protect consumers.

## Step-by-Step: How to Abuse the NFC Tag

The writable NFC tag in Poo~Pourri’s Pocket Sprayer is a potential goldmine for scammers. Here’s how an attacker could exploit it, shared strictly for educational purposes. Do not attempt this—it’s illegal and unethical.

* **Get Tools**: Install the NFC Tools app on an NFC-enabled Android phone.
* **Find Products**: Visit a store (e.g., Target, Walgreens) with Poo~Pourri Pocket Sprayers on shelves.
* **Scan Tag**: Use NFC Tools to confirm the tag links to a pourri.com URL, like https://pourri.com/funk-factory.
* **Overwrite URL**: Set up a phishing site on a lookalike domain (e.g., pourri.io) and use NFC Tools to write this fake URL to the tag.
* **Leave for Buyers**: Place the tampered bottle back on the shelf. When purchased, the tag directs users to the malicious site.
* **Scale Up**: Repeat across multiple bottles to maximize impact.

This attack is fast, discreet, and requires only a smartphone.

## Technical Details and Risks

### NFC Tag Basics
The NFC tag in the spray is likely an NDEF-formatted tag, storing a URL for the Funk Factory App, which offers games and rewards. My cruise experiment confirmed the tag is writable, meaning it wasn’t locked during production—a common oversight in NFC-enabled products.

### The Vulnerability

* **Writable Tags**: No write protection allows anyone with an NFC writer to modify the tag.
* **No URL Validation**: The app or browser accepts any URL without checking its legitimacy.
* **Store Exposure**: Bottles are accessible on store shelves, enabling pre-purchase tampering.

### Risks

* **Phishing Attacks**: Fake URLs can lead to sites that steal credentials or push malware.
* **Data Theft**: Users might enter emails, phone numbers, or card details on scam pages.
* **Brand Damage**: Victims may lose trust in Poo~Pourri, blaming the brand for the breach.
* **Mass Exploitation**: Attackers can tamper with many bottles in one store visit.

### Real-World Impact

Imagine scanning the tag after buying the spray, expecting a fun app but landing on a phishing site. I caught the flaw on my cruise, but unsuspecting buyers might not, especially since rewriting tags in stores is so easy.

## Resolutions and Mitigation

### For Poo~Pourri

* **Lock Tags**: Make NFC tags read-only at the factory.
* **Use Secure Codes**: Store unique identifiers, not URLs, verified by the Funk Factory App.
* **Tamper-Evident Packaging**: Add seals to show if a bottle was opened.
* **Educate Users**: Instruct customers to verify URLs start with pourri.com.

### For Retailers

* **Secure Shelves**: Keep NFC products in monitored or locked areas.
* **Audit Tags**: Randomly scan bottles to ensure correct URLs.
* **Train Staff**: Teach employees to detect tampered products.

### For Consumers

* **Check Tags**: Use NFC Tools to read the tag’s URL before scanning.
* **Verify Domains**: Confirm the URL is pourri.com, not a lookalike like pourri.io.
* **Report Issues**: Alert retailers or Poo~Pourri (via pourri.com) about suspicious tags.

### Industry-Wide Solutions

* **NFC Standards**: Use read-only tags or cryptographically signed data.
* **Awareness Campaigns**: Educate consumers about NFC risks, akin to QR code warnings.

## Conclusion

Finding a writable NFC tag in my Poo~Pourri spray during a cruise was a surprising lesson in product security. A feature meant to delight customers could instead lead to phishing or worse. By locking tags, securing shelves, and raising awareness, Poo~Pourri, retailers, and users can close this gap. Next time you scan an NFC tag, double-check its destination—it could save you from a scam.
