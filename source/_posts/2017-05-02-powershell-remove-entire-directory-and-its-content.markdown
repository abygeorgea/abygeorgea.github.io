---
layout: post
title: "Powershell - Remove entire directory and it's content"
date: 2017-05-02 06:27:27 +1000
comments: true
categories: Powershell
keywords: Powershell , Remove folders having more than 260 character Path , git
description: How to remove folders and contents having more than 260 character
---

The PowerShell command to remove an entire directory and its contents ( including sub folders and files) is below

``` plain cmd.exe
rm -Rf pathToDirectoryToBeRemoved/
```

R flag denotes to run “rm” command recursively . “f” flag denotes to run in forcefully. We can even replace “f” with “v” for verbose mode and “i” for interactive mode.

Note: Above command can also be used to delete files which have long path ( more than 260 characters)