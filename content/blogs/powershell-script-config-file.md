+++
author = "Kyle Parrish"
date = "2021-01-15"
description = "Creating a Config File for your PowerShell Scripts"
linktitle = ""
title = "Creating a Config File for your PowerShell Scripts"
type = "post"
categories = ["powershell"]
image = "/images/powershell.jpg"
+++

# Introduction

Recently, I was working on a PowerShell scripts that I wanted to share with others on my team but did not want to commit it to our Git repository with the customizable variables declared in the code (Always best practice. Keep the sensitives out of repos!). Instead, I wanted to implement a way to have a config file where these configurable variables can be set and the main script would load them when launched. A quick "DuckDuckGo" search led to many others that have had the same goal but I didn't find one that really worked like I had intended. Here are my steps that I used to accomplish what I had in mind.

## The config file

I simply wanted a config file that mimicked what we typically see in a Docker .env file. Each line consists of a variable name and value (key-value pair) in the format `<key>=<value>`.

Here is an example of what that file may look like.

```docker
LISTENIP=0.0.0.0
LISTENPORT=8443
APPID=1cb8ea0e-fa15-47b2-9bd5-957117c01678
OU=OU=Users,DC=Corp,DC=com
```

This aproach provides a very simple layout that is easily recognized and can be repeated without looking back and thinking "how did I set that up again?".

## The PowerShell Code

The code snippet to retieve the values and set the variables consists of the following:

* Foreach loop to process each line of the file
* Splits the pair at the first "=" character
* Sets a variable based on the Name and Value parsed out of each line

```powershell
Foreach ($i in $(Get-Content script.conf)){
    Set-Variable -Name $i.split("=")[0] -Value $i.split("=",2)[1]
}
```

The first line begins the loop with `Foreach ($i in $(Get-Content script.conf)){` which pulls in the content of the config file and sets the current line to the value `$i`. 

For each line we use the `split` method to pull apart the key/value pair and will use that information to declare the variable.

`$i.split("=")[0]` splits the line up into parts using the "=" as a delimiter and selects the first part in the list ([0]).

Typically, each line will only contain one "=" sign but in some cases this may not be true. One example that I came across was when working with Active Directory DistinguishedNames such as "OU=Users,DC=contoso,DC=com". This would allow the name to be parsed but would split the remaining content of the line into additional parts depending on the number of "=" it contained. "DISTINGUISHEDNAME=OU=Users,DC=contoso,DC=com" would become:

```
DISTINGUISHEDNAME
OU
Users
contoso
com
```
But, what we want are the two parts to be:

```
DISTINGUISHEDNAME
OU=Users=contoso=com
```

This is why the value is determined via `$i.split("=",2)[1]`. The addition of the "2" after specifiying our delimeter is telling PowerShell to return a maximium of two substrings. So regardless of the amount of "=" signs in the string it will only split on the first one, returning a name (before the first "=") and a value (after the first "=").

The `[1]` is used to select the second value in the list and set that as the value of the variable.

## Putting it to use

Now that we have the structure in place, lets put it into practice and see what the it looks like all tied together.

Our scenario will be I am writing a simple PowerShell script to call a webservice and it requires an API key and a special User-Agent in the HTTP header. Once the script is complete I want to share this with the community but I don't want my API key and User-Agent values included in the script. 

### Create the script

This script makes a GET request against `https://swapi.co`, a popular Star Wars API, to find intersting details about a specified starship seen in the films.

#### script.conf

We will only be working with two values in the config file. The `APIKEY` and the `USERAGENT`.

```
APIKEY=R2D2C3P0BB8
USERAGENT='Luke-SkyWalker'
```


#### Get-DeathStar.ps1

The main script uses the snippet shown above to pull in the content of the `script.conf` file and sets the values accordingly.

``` powershell
Foreach ($i in $(Get-Content script.conf)){
    Set-Variable -Name $i.split("=")[0] -Value $i.split("=",2)[1]
}

$Headers = @{
    'APIKey' = $APIKEY
    'User-Agent' = $USERAGENT
    'Content-Type' = 'application/json'
}

$Url = 'https://swapi.co/api/starships/9/'

Invoke-RestMethod -Method GET -Uri $Url -Headers $Headers
```

If we inspect the request with a tool such as `Fiddler4` we can see our custom values being passed in the header.

```
GET https://swapi.co/api/starships/9/ HTTP/1.1
Content-Type: application/json
APIKey: R2D2C3P0BB8
User-Agent: 'Luke-SkyWalker'
Host: swapi.co
Connection: Keep-Alive
```

## Using with Git

Now that we have the two components working together how do we address the concens with sharing via Git?

First, we need to initialize the directory as a Git repository and set the origin.

```
git init
git remote add origin https://github.com/arnydo/get-deathstar.git
```

Then, we need to create a new file named `.gitignore`. This is where we place regex patterns of anything we do not want to be included when we push this git repo.

Our `.gitignore` will include the following:

```
*.conf
```

This tells git to not include any files that have the `.conf` extension.

To be sure that others who may use this repo understand how to setup the config file for their use, we can create a new file named `script.cong.example` that only contains that variable names that are expected.

```
APIKEY=
USERAGENT=
```

The `README.md` could mention the need to copy the `script.conf.example` to a new file named `script.conf`.

```markdown
## Before running script

Be sure to copy the script.conf.example file to script.conf and add your custom values before running the get-deathstar.ps1 script.
```

Now, with everything setup we can push our new repository to Github and it will not include our sensitive data.

```
git add .
git commit -m "Initial commit"
git push -u origin master
```

And that is it! I hope you found this helpful. If you have any questions or ideas for improvement please leave a comment.

See you next time.