---
layout: post
title: "How to modify environment variables for a different not logged in user in windows"
date: 2020-10-14 06:09:00 +1100
comments: true
categories: 
keywords: 
description: How to modify environment variables for a different not logged in user in windows
---
Modifying environment variables for a logged in user is straight forward. However there are some instances when we need to modify environment variables for a different user. One frequent usage which I have come across is when we need to modify PATH variables for a service account in a CI box. Service account may not have local logon rights to the machine Or we may not know its password. Hence I always end up using below method to update path variables

## Identify Security Identifier(SID) 

Open command prompt and use below commands

Replace USERNAME with actual username

```
wmic useraccount where name="USERNAME" get sid

```

We can also find username based on SID
Replace SIDNUMBER with actual value

```
wmic useraccount where sid="SIDNUMBER" get name

```

Once we have the SID number of the next user, move on to next step

## Modify Registry

* Open registry editore (Regedit.exe) in windows. 
* Navigate to HKEY_USERS >> SID of USER>> ENVIRONMENT
* This should display all defined environment variables. We can modify all variables as we need