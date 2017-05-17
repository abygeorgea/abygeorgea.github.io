---
author: Aby George A
comments: true
date: 2016-08-17 20:30:54+00:00
layout: post
link: /blog/2016/08/17/identifying-elements-using-locators-in-selenium/
slug: identifying-elements-using-locators-in-selenium
title: Identifying elements using Locators in Selenium
wordpress_id: 184
categories:
- Element Identification
tags:
- Selenium Locator Strategy
- Selenium WebDriver
---

Locators are html properties of a web element , which can be considered as an address of the element. An element will have various html properties. We can use Firebug extension or Chrome dev tools to identify different locators of an element.

Selenium Web Driver provides two different methods for identifying html elements .



	
  * _**FindElement  **_for WebDriver and WebElement Class. When locating element matching specified criteria, it looks through DOM( Document Object Model) for matching element and return the first matching element. If there are no matching element, it will throw NoSuchElementFoundException

	
  * _**FindElements** _for WebDriver and WebElement Class. When locating element matching specified criteria, it looks through DOM( Document Object Model) for matching element and return a list of all matching element. If there are no matching elements, then it will return an empty list .


Note: Both of them doesn't support regular expression for finding element. Simple way to do that will be to get list of all elements and then iterate to find a matching regular expression

There are multiple criteria which we can use for looking for an element. FindElement and FindElements work exactly same way except for above difference. Different critieria are
	
  * driver.FindElement(By.Id(<elementID>))

	
  * driver.FindElement(By.Name(<element name>))

	
  * driver.FindElement(By.ClassName(<element class>))

	
  * driver.FindElement(By.TagName(<htmltagname>))

	
  * driver.FindElement(By.LinkText(<linktext >))

	
  * driver.FindElement(By.PartialLinkText(<linktext >))

	
  * driver.FindElement(By.CssSelector(<css selector >))

	
  * driver.FindElement(By.XPath(<xpath query expression>))




Example:

```

WebElement _firstElement = driver.findElement(By.id("div1"));
WebElement _secondElementInsideFirstOne =_firstElement .findElement(By.linkText("username"));

IList&lt;IWebElement&gt; elements = driverOne.FindElements(By.ClassName(&lt;span class="pl-s"&gt;&lt;span class="pl-pds"&gt;“&lt;/span&gt;green&lt;span class="pl-pds"&gt;“&lt;/span&gt;&lt;/span&gt;));

```



Using any attributes other than XPath and CssSelector are straight forward. More about using XPath and CssSelector in next blog.
















