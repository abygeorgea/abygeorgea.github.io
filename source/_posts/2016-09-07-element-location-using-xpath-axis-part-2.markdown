---
layout: post
title: "Element Location Using XPath Axis Part 2"
date: 2016-09-07 20:54:13 +1100
comments: true
categories: [element identification]
keywords: element identification
description: How to identify child element in selenium using XPath
---

In previous post, I have mentioned different ways of identifying web elements using XPath . Very often , we will have to identify child elements while automating using selenium. Let us consider below example . This is an HTML layout of table

``` html
<table id=table1 style="width:100%">
  <tr>
    <td>John</th>
    <td>Smith</th> 
    <td>50</th>
  </tr>
  <tr>
    <td>Jill</td>
    <td>Smith</td> 
    <td>50</td>
  </tr>
  <tr>
    <td>Eve</td>
    <td>Jackson</td> 
    <td>94</td>
  </tr>
</table>
```

Assuming we need to iterate across all the rows to identify which row have the name "Eve" and then do some action on that row . This can be achieved by below

``` csharp
 // Identify the table header using ID
 IWebElement e = driver.FindElement(By.Id("table1"));
 
 /* Identify all child nodes for the webelement e (table) . Note the "." in XPath which means to search within current node. */
 
 IList<IWebElement> rowlist = e.FindElements(By.XPath(".\\tr"));
 foreach(var row in rowlist){
 IList<IWebElement> collist = row.FindElements(By.XPath(".\\td"));
 
 // Now iterate over each value in collist to check whether it have "Eve".
 }
```
Important step here is to use "." in XPath so that selenium limit its search for current node rather than everywhere on document. This will help to identify elements with respective to another element.

Further details of how to use XPath can be found in https://www.w3schools.com/xml/xpath_syntax.asp

Main ones are 

* /    - Search from root 
* //   - Search anywhere in document which match selection
* .    - select current node
* ..   - select parent of current node
