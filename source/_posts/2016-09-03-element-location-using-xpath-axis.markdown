---
author: Aby George A
comments: true
date: 2016-09-03 22:33:04+00:00
layout: post
link: /blog/2016/09/03/element-location-using-xpath-axis/
slug: element-location-using-xpath-axis
title: Element location using XPath Axis
wordpress_id: 281
categories:
- Element Identification
- Selenium
- Specflow
tags:
- Element Locator Strategy
- Selenium WebDriver
- XPath

keywords: Selenium , Specflow , Element Locator Strategy , C# , XPath
description: How to take Identify web elements in selenium using relative XPath
---

During testing we will sometimes come up to situations where developers are not following best practises for testability . We will frequently come up situations where elements doesn't have any unique identifiable property. XPath axis comes to help in those situations. We can identify elements using various XPath Properties

List of various XPath Axis are available in https://developer.mozilla.org/en-US/docs/Web/XPath/Axes
If you have well-defined properties to identify the element, use them as your locator. Please read  locator strategy  [Using XPath](/blog/2016/08/30/element-location-using-xpath/) and [Other Parameters](/blog/2016/08/17/identifying-elements-using-locators-in-selenium/)

Below are major one's which we will frequently use


### 1. ancestor


This selects all ancestors of current node. That will include parent, grand parents etc
Eg :
```
 //td[text()='Product Desc']/ancestor::tr
```

### 2. descendant


This selects all children of current node. That will include child, grand child etc
Eg: 
```
/table/descendant::td/input
```

### 3. followingis


Th selects everything after the closing tag of current node Eg: 
```
//td[text()='Product Desc']/following::tr
```

### 4. following-sibling


This selects all siblings after the closing tah of current node.
Eg: 
```
//td[text()='Product Desc']/followingsibling::td
```

### 5. preceding


This selects everything prior to current node
Eg:
```
 //td[text()='Add to cart']/preceding::tr
```

6. preceding-sibling

This selects all siblings prior to current node
Eg: 
```
//td[text()='Add to cart']/precedingsibling::td
```

### 7. child


This selects all children of current node


### 8. parent


This select parent of current node

As usual , you can always use combinations of above in your test. Statements can be constructed in the same way as we traverse the XPath axis

Last , but not least... we can also use regular expression in XPath.
