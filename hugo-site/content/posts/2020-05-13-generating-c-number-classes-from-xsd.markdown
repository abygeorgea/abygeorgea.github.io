---
layout: post
title: "Generating C# classes from xsd"
slug: "generating-c-number-classes-from-xsd"
date: 2020-05-13T05:39:00+10:00
comments: true
categories:
  - C#
keywords: 
description: How to generate C# classes from xsd
---
XML Schema Definition tool will help to generate classes that conform to a schema. 
Steps are as follows.

*  Open VS Command prompt . ( Start Menu >> Visual Studio 2019 >> Developer command prompt for VS2019) 
*  Pass xml schema as an argument to xsd.exe . \c at the end denotes to generate classes

   ```  
   xsd.exe C:\Temp\sampleschema.xsd /c /o:C:\Temp 
   ```
   
*  There are other options as well. Main ones are below.

   ```  
   xsd.exe <schema.xsd> /classes|Dataset [/e:] [/l:] [/n:] [/o:] [/s:] 
   
   /classes | Dataset : denotes whether to generate class or dataset
   /e: Element from schema to process
   /l: Language to use for generated ode . Choose from 'CS','VB','JS','VJS','CPP'. Default is CS
   /n: Name os namespace
   /o: Output directory for generated classes
   
   ```
   
*    There are other options as well. Details can be found on help "xsd /?"


Location of xsd.exe tools is under C:\Program Files (x86)\Microsoft SDKs\Windows\<VERSION>\bin\NETFX <VERSION> Tools\xsd.exe. There are chances of having multiple versions of this tool . If you ever get "xsd is not recognised as n internal or external command" error, make sure the PATH variable is set to this. Else directly go to that location and run.
  
   