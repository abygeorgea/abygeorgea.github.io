---
layout: post
title: "Zip and Extract zip files using Csharp"
date: 2017-04-20 21:59:02 +1000
comments: true
categories: codesnippets
keywords: 
description: Zip and Extract zip files using Csharp
---
On corporate world, most of the times, the access required for installing applications and connecting to internet will be limited. There can be scenarios where access to install Mountebank using npm will not be available. In those circumstances, we can just unzip the zip file downloaded from self contained archive links in [mbtest.org](http://www.mbtest.org/docs/install)

Below code snippet can be used for extracting the zip files on the fly , so that it can be used for running test cases on any machine.

###Pre-Requisite###

- .Net 4.5 is needed
- Add reference to below dll to solution
	- System.IO.Compression.dll
	- System.IO.Compression.FileSystem.dll


###Example###

Below example is based on (copied from) [msdn](https://msdn.microsoft.com/en-us/library/hh485723(v=vs.110)

``` c#		
        Public void ZipFile()
        {
            string startPath = @"c:\example\start";
            string zipPath = @"c:\example\result.zip";
            string extractPath = @"c:\example\extract";

            ZipFile.CreateFromDirectory(startPath, zipPath);
        }

        Public void UnZipFile()
        {
            string startPath = @"c:\example\start";
            string zipPath = @"c:\example\result.zip";
            string extractPath = @"c:\example\extract";

            ZipFile.ExtractToDirectory(zipPath, extractPath);
        }
```

