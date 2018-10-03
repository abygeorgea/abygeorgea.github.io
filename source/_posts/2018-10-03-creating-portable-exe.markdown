---
layout: post
title: "Creating utility tool as portable EXE"
date: 2018-10-03 21:58:18 +1000
comments: true
categories: [EXE]
keywords: Create EXE, Combine dll
description: How to create portable EXE
---

Recently one of my colleague approached me asking to help on creating a utility tool using selenium web driver. The requirement was simple which includes accepting few arguments from the command line and then open a browser and complete some actions on browser based on inputs provided. Having worked on selenium web driver for a few years, I thought this is relatively simple and can be done quickly.

It is implemented as a C# console app which had reference to selenium web driver. It accepts few arguments from command line and based on the values it opens up chrome browser and completes the action. The initial version was already there on which I made some modifications. We gave a demo to the user and thought it is all done. 

As in any normal software projects, it was far from over. There were additional requirements to support multiple browsers, flexibility to provide arguments in any required order, the requirement to display detailed help text so that end user will know how to use the utility tool. All of them was done and we shared the build output, which included the exe file, all dlls used, drivers for various browsers and configuration files.  That's when I had my next requirement to make it as a portable EXE with the single exe file. That is not something which I had done before. Hence I spent quite some time to google and read through various approaches.

##Fody.Costura##

Costura is an addin for Fody. It helps to embed all assembly references into the output assembly/exe. Details documentation and source code can be found in [github](https://github.com/Fody/Costura).  Usage was pretty easy.  

* Install nuget package  `Install-Package Costura.Fody`.  
* Create a FodyWeavers.xml ( modify if it exists) in the root folder of project . Update contents as below

```
<?xml version="1.0" encoding="utf-8" ?>
<Weavers>
  <Costura/>
</Weavers>
```

* Make sure all required dlls are marked as "Copy Local".
* Build the project

This helped to combine all dlls like webdriver, newtonsoft dll etc into the utility exe file.  The build output had only an exe file, config and other resources which I marked to copy to output. 



## Embedding Resources##

Fody.Costura helped to combine exe files with required dlls and there by reducing number of files which needs to be distributed. However, I still had few text and json files which are resources for this tool. Initially, all required resources were copied to output and were accessed from there. 

There is an option to embed all required files. It is done by changing  BuildAction in properties to Embedded Resources. This will include files in output assembly which can be accessed in code. 

This [webpage](http://derpturkey.com/embedding-dlls-or-resources-in-c-executable/) has more details about how it can be done. 

Below code snippet will show how it can be accessed. This shows how to read a resource file called Help.Txt

```csharp
 var assembly = Assembly.GetExecutingAssembly();
 var resourceName = "NameSpaceName.SubFolderPathWhereResourceIsKept.Help.txt";
 using (Stream stream = assembly.GetManifestResourceStream(resourceName))
 using (StreamReader sr = new StreamReader(stream))
{
 var line = sr.ReadToEnd();
 Console.WriteLine(line);
}
 ```


After completing above two steps, I was able to combine all dlls and other required files into the Utility Exe. Now I just had to distribute exe file and the drivers for various browsers.