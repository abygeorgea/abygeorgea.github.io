---
layout: post
title: "Git - How to solve filename too long error"
date: 2016-09-23 06:31:03 +1000
comments: true
categories: Git
keywords: Git
description: How to solve filename too long error while using Git on Windows
---

Git for windows is normally shipped with long path support disabled due to mysys not supporting file path/name greater than 260 character. While cloning repository with large nested directory structute may cause error "file name too long". This can be fixed by below command. It can be executed using powershell or cmd directly in project ( or anywhere if git variable is available)

``` plain cmd.exe
git config --system core.longpaths true
```