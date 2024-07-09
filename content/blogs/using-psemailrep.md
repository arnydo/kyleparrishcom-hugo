+++
author = "Kyle Parrish"
date = "2020-02-08"
description = "Using the PSEmailRep PowerShell Module"
linktitle = ""
title = "PSEmailRep PowerShell Module"
type = "post"
categories = ["powershell"]
image = "/images/sublime-logo.png"
+++

# Introduction

Hey! Kyle here. I recently released a small update to one of the PowerShell modules I put together to interact with EmailRep's API and thought I would share a little about what it is and why I built it.

* [Github](https://github.com/arnydo/psemailrep)
* [PowerShellGallery](https://www.powershellgallery.com/packages/psemailrep)

![PSEmailRep](https://github.com/arnydo/PSEmailRep/raw/master/Media/screenshot.png)

First of all, if you work in Information Security or IT in general, be sure to check out [EmailRep.io](https://emailrep.io). This is an excellent service built by a great group of people to provide enrichment data about an email address. Nearly all of our online interactions these days link back to an email address and it is important that we observe the "reputation" of email addresses that are used to interact with our systems. 

Here is a brief description of EmailRep by the EmailRep team:

*EmailRep uses hundreds of factors like domain age, traffic rankings, presence on social media sites, professional networking sites, personal connections, public records, deliverability, data breaches, dark web credential leaks, phishing emails, threat actor emails, and more to answer these types of questions:*
- *Is this email risky?*
- *Is this a throwaway account?*
- *What kind of online presence does this email have?*
- *Is this a trustworthy sender?*

# PSEmailRep

## Goals

After using EmailRep for a short time I quickly realized the value that it would add to my daily routines of triaging phishing emails and maintaing the many security aspects of our environment. With PowerShell being my daily-driver for automation and security tools I decided to put together a module that would interact with the EmailRep API. Seeing that nobody else at the time had been working on one I figured it would be a good project to start and share with the community. Some of the things that I was looking to accomplish were:

 * Maintained in a Github repo
 * Published to PowerShellGallery
 * Helpful documentation on the available features
 * Deployment pipeline 

 Keep an eye out for a follow-up post on how I tackled these points. I learned a ton about building a module template with [Plaster](https://github.com/PowerShell/Plaster), running various checks with [Pester](https://github.com/pester/Pester), and preparing it to be pushed out to a central repository such as PowerShellGallery with [PSDeploy](https://github.com/RamblingCookieMonster/PSDeploy).

 ## The Module

 The goal of the PowerShell module isn't to recreate the wheel, rather, simply take advantage of what is already available. The only thing I needed to do was to find a way to streamline it into my daily routines. The obvious choice for me was a PowerShell module...because...PowerShell! The available functions mimick what is currently available in the [API Docs](https://docs.emailrep.io/) and as new endpoints are added more functions may be added to the module. Here are the current functions and what they are used for.

 ### PS> Get-EmailRep

```powershell
❯ Get-EmailRep bill@microsoft.com


email                      : bill@microsoft.com
reputation                 : high
suspicious                 : False
references                 : 85
blacklisted                : False
malicious_activity         : False
malicious_activity_recent  : False
credentials_leaked         : True
credentials_leaked_recent  : False
data_breach                : True
first_seen                 : 07/01/2008
last_seen                  : 10/16/2019
domain_exists              : True
domain_reputation          : high
new_domain                 : False
days_since_domain_creation : 10512
suspicious_tld             : False
spam                       : False
free_provider              : False
disposable                 : False
deliverable                : True
accept_all                 : True
valid_mx                   : True
spoofable                  : False
spf_strict                 : True
dmarc_enforced             : True
profiles                   : {twitter, vimeo, pinterest, angellist...}
```

This function does a GET query against the EmailRep API to collect what is already known, if anything, about the email address. In this case, bill@microsoft.com has several different peices of enrichment data that could help determine the reputation of the address. Now, most of this information is simply information and it is up to you to determine what you do with that information. In this case, EmailRep believes the reputation of this address to be "high", but that does not mean that it is "good" or "trusted". This simply means that there is enough information about address that it has a reputation.

 > How can I use this information?

Well, lets take a look at the following: 
```
blacklisted                : False
malicious_activity         : False
malicious_activity_recent  : False
credentials_leaked         : True
credentials_leaked_recent  : False
data_breach                : True
```
This information indicates that while there are no reports of malicious activity on the account there are reports that it has been listed in recent breach reports or credential leaks. This is important to consider when allowing this account to interact with users in your environment or with your services. It is possible that the account is in possession of a malicious actor and does not have the best of intentions. This may trigger you to look further into the account or require additional verification before proceeding.

These indicators are what I find very useful when triaging phishing emails:
```
domain_exists              : True
domain_reputation          : high
new_domain                 : False
days_since_domain_creation : 10512
```
A quick look at the domain details helps identify if this is a newly created domain and what the current reputation on that domain is. Often times, we will see that phishing emails are originating from domains that are only hours, or sometimes minutes, old. This typically an indicator of malicious activity.

You can take a look at the meaning of each attribute by heading over to the [API Docs](https://docs.emailrep.io/technical-reference/reference/email/getemail).

### PS> New-EmailRep

This function is used to report email addresses to the EmailRep API. It does require an API key and you can request one over at [EmailRep's site](https://emailrep.io/key). 

An example report would look like:

```powershell
New-EmailRep -Email bill@microsoft.com `
     -Tags bec,credential_phishing `
     -Description 'Compromised account sending Sharepoint login page clone' `
     -APIKey '4a534f4e20697320626574746572207468616e20584d4c21'
```
This will report to EmailRep that bill@microsoft.com has been involved in a [Business Email Compromise](https://www.trendmicro.com/vinfo/us/security/definition/business-email-compromise-(bec)) and is sending credential phishing via fake Sharepoint login pages. 

While the EmailRep API does not yet allow you to review these reports once they are submitted it is certainly on the road map. Having access to this data will greatly improve the accuracy of the determined 'reputation' of the email address.

### PS> Set-EmailRep

This function is used to save your API key to your user profile so you do not need to include it on every execution. Currently, this utilized PowerShell's [secure string](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.security/convertto-securestring?view=powershell-7) capabilities to encrypt the key in an XML file for later retrieval (each time Get-EmailRep or New-EmailRep is called). This key can only be read by the user that set it thanks to [Windows DPAPI](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.security/convertfrom-securestring?view=powershell-7#description). DPAPI is also a great topic to discuss. I learned a lot about how it is used and the risk it introduces...but that is for another day.

To use `set-emailrep` simply run the command and enter in your APIKey. If you have previously ran this command it will alert you and ask if you would like to overwrite the existing entry.

## Final Thoughts

This has been a learning process and opened up many new rabbit holes to follow. I plan on continueing to share these as I tackle them. Hopefully somebody out there will find it useful. I know I wouldn't be where I am at without the contributions that others have made to the community. It is by their encouragement that I want to share my experiences as well and return the favor. Let me know what you think and if you have any questions about PSEmailRep!


