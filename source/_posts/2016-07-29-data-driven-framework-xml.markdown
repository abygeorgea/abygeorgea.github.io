---
author: Aby George A
comments: true
date: 2016-07-29 22:10:02+00:00
layout: post
link: /blog/2016/07/29/data-driven-framework-xml/
slug: data-driven-framework-xml
title: Data Driven Framework - XML
wordpress_id: 30
tags:
- Data Driven Testing Framework
- Selenium WebDriver
- XML
categories:
- Data Driven Testing Framework
- Selenium WebDriver
- XML
keywords: Selenium , Specflow , Data Driven Testing , C# , XML
description: How to read from XML data sheet in a data driven framework using selenium webdriver and C#
---

I am not going to explain what is data driven framework or what is its benefits.  All of them are pretty well-known . If not, just google it.

Here I am going to explain a sample code which can be used to read from xml data files. This will be helpful to implement a data driven frame work for BDD testing , using Specflow or Cucumber


## Pre - requiste


Code is written in Csharp . We need to add below reference to visual studio solution



	
  1. Add reference to System.xml

	
  2. Add reference to System.Xml.Linq




## **XML Format**


[code language="xml" ]




[/code]

Node names in above example are Scenario1, Scenario2 and Scenario3

Element or Attribute name are username, password, Email .

You can add any number of nodes and attributes depending on the test scenario


## **Read specific Value from XML**


Below code provides a solution to read values of existing Key/Attribute from a specific node.

```
public static string ReadDataFromXML(string FileName, string NodeName, string KeyName)
{
 string _basePath = AppDomain.CurrentDomain.BaseDirectory.ToString();
string _datafilePath = _basePath + @"..\..\Data\" + FileName;
XDocument xmlDoc = XDocument.Load(_datafilePath);
data = xmlDoc.Root.Element(NodeName).Attribute(KeyName).Value;
return data;
}
```


 




## Read All values for a Scenario from XML


This code gives a solution for reading details of all attributes/key from a node. This comes very handy for reading all data required for a scenario and adding them to scenario context , so that data can be shared across specflow/cucumber step definitions

```
public static void ReadAllDataFromXml(string xFileName, string xNodeName)
{
 string data = string.Empty;
// Load path of xml file . Data Folder in below is folder name
string _basePath = AppDomain.CurrentDomain.BaseDirectory.ToString();
string _datafilePath = _basePath + @"..\..\Data\" + xFileName;
XDocument xmlDoc = XDocument.Load(_datafilePath);

var cols = xmlDoc.Descendants(xNodeName).First();

foreach (XAttribute xAtt in cols.Attributes())
{
Console.WriteLine("{0},{1}", xAtt.Name, xAtt.Value); // --printing values --
string temp = xAtt.Name.ToString();
ScenarioContext.Current.Add(temp, xAtt.Value);
// The values are added to scenario context as key value pair.can be modified
}
```





## Write Data into XML




  Below function gives a solution to update value on an existing key/attribute in data sheet . Sometime the data sheet will be copied over to bin folder when we build the solution. That makes it necessary to update both original data sheet and the one in bin folder  so that modified data can be used in same test without another rebuild.

```
public void WriteIntoXML(string xData, string xElement, string xAttribute, string xFileName)
{
 // This solution updates value of an existing key.
 // Xdoc refers to the xmlsheet in the Solution explorer
 // xDoc_2 refers to the xmlsheet inside bin folder while running

 // This solution write them separately in below code.
 // Below is the path to xml file after building solution
 XDocument xDoc_2 = XDocument.Load(Path.Combine(Environment.CurrentDirectory, "Data Folder", xFileName));

 //Below should be the path of xml file
 XDocument xDoc = XDocument.Load(Path.Combine(Path.GetFullPath(@"../../Data Folder"), xFileName));
 xDoc_2.Root.Element(xElement).Attribute(xAttribute).Value = data.ToString();
 xDoc.Root.Element(xElement).Attribute(xAttribute).Value = data.ToString();
 xDoc_2.Save(Path.Combine(Environment.CurrentDirectory, "Data Folder", xFileName));
 xDoc.Save(Path.Combine(Path.GetFullPath(@"../../Data Folder"), xFileName));
}
```





