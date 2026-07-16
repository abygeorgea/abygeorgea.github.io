---
layout: post
title: "Sending Emails through SMTP in C#"
slug: "sending-emails-through-smtp-in-c-number"
date: 2019-04-09T22:03:26+10:00
comments: true
categories:
  - codesnippets
keywords:
  - Email
  - C#
  - codesnippets
description: How to send emails in C# using SMTP
---

Recently I was looking for some code for sending emails via SMTP in C#. Below are few links which I found with some reusable code. Overall it looks fine , but have to include multiple validations for error handling . 

* https://gist.github.com/robertgreiner/1529127
* https://gist.github.com/gzuri/2850914
* https://gist.github.com/pranavq212/1cbecac15abb229d40f1ad0765aa4dce
* https://gist.github.com/TrailCoder502/6254bdfcfe71c4000600

In Nutshell, flow is as below

* Define a function to send email which accepts an input Email object
* Validate the email object to ensure all mandatory fields are present and correct
* Create a new MailMessage object and SMTPClient Object and send email


Above gist links have some reusable code to achieve step 3 of above. 

