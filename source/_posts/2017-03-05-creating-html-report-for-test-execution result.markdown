---
layout: post
title: "Creating HTML report for test execution result"
date: 2017-03-05 20:58:13 +1100
comments: true
categories: [Specflow, Nunit]
keywords: Specflow, HTML report
description: How to create HTML report for test execution result
---

### How to create HTML report with details of test execution ###

Very often , we will be required to create a report with details of test execution , so that it can be presented to various stakeholders. Specflow provides a feature to create HTML reports. Let us look into more details about how is this done

* Read through and understand details of reporting from [specflow](https://github.com/techtalk/SpecFlow/wiki/Reporting).
* Ensure packages for Specflow, Nunit, Nunit console runner are already installed. 
* If you are using Nunit 3, install NUnit.Extension.NUnitV2ResultWriter package via nuget package manager. If this is not installed, we will get an error "Unknown result format: nunit2".
* Follow setups required for running specflow test cases from command line. Details can be found [here]({{site.root}}blog/2017/03/04/running-specflow-test-from-command-line-using-nunit). 
* Modify the bat file to create nunit2 reports.
```
PathToNunitConsolerunner\nunit3-console.exe --labels=All --out=TestResult.txt "--result=TestResult.xml;format=nunit2" PathTo\AcceptanceTests.dll
```
* Add below command into Bat file. This will create HTML Report called "MyResult.html"
```
PathToSpecfloPackage\specflow.exe nunitexecutionreport PathTo\AcceptanceTests.csproj /out:MyResult.html
```
* Final bat file will look like below.

```
REM bat file to run test cases from console and create xml result file
.\..\..\..\packages\NUnit.ConsoleRunner.3.8.0\tools\nunit3-console.exe  --labels=All --out=TestResult.txt "--result=TestResult.xml;format=nunit2" .\AcceptanceTest.dll

REM Generate html report from test output
.\..\..\..\packages\SpecFlow.2.1.0\tools\specflow.exe nunitexecutionreport .\..\..\AcceptanceTest.csproj /out:MyResult.html

```

EDITED: If it throws below error in newer version of Visual Studio  then ensure MS Build tool 2013 is installed. It can be downloaded from https://www.microsoft.com/en-US/download/details.aspx?id=40760

Error : "The tools version "12.0" is unrecognized. Available tools versions are "2.0", "3.5", "4.0". ",