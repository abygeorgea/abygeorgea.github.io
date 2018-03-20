---
layout: post
title: "Running Specflow Test from command line using Nunit"
date: 2017-03-04 20:39:59 +1100
comments: true
categories: [Specflow, Nunit]
keywords: Specflow, Nunit
description: How to run specflow test cases from command line
---
### How to run specflow test cases from command line ###

We can use nunit console runner for running specflow test cases from command line. Running specflow test cases through nunit console runner will help to create test results in xml file, which can then be used for creating html reports.  

Procedure for command line test execution are

1. Define Nunit as the test runner.  This is done in config file

``` xml 
    <specFlow>
          <unitTestProvider name="NUnit"/>
    </specFlow>
    
```

2. Include Nunit.Console.Runner package to solution via nuget package manager 
3.  Run specflow test cases using below command. We can create a bat file with below command and execute them as required. 

```

pathToNunitConsoleRunner\nunit3-console.exe  PathToProject.dll

 example will be
 .\..\..\..\packages\NUnit.ConsoleRunner.3.8.0\tools\nunit3-console.exe   .\BDDFramework.dll
```




