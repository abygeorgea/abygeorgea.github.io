---
author: Aby George A
comments: true
date: 2016-08-30 10:58:16+00:00
layout: post
link: /blog/2016/08/30/element-location-using-xpath/
slug: element-location-using-xpath
title: Element Location using XPath
wordpress_id: 229
categories:
- Element Identification
- Selenium
- Specflow
tags:
- Selenium Locator Strategy
- Selenium WebDriver
- XPath
---

XPath is XML query language which can be used for selecting nodes in XML. Hence it can be used to identify elements from DOM since they are represented as XHTML documents. Selenium WebDriver also supports XPath for locating elements. They also help to look for elements in both direction and hence it is generally slow compared to all other locator strategy. We can use XPath with both absolute path and relative path.


#### **Absolute XPath**


Absolute Path refers to specific location in DOM, by considering it's complete hierarchy. However this is not an ideal locator strategy since it makes your test very brittle. The absolute path will change if there is any change/realignment etc in UI.

Example of Xpath using absolute path is as below

```
WebElement userId = driver.FindElement(By.XPath(html/body/div[2]/div/form/input[2]));

```


#### **Relative XPath**


With relative Path , we can find element directly without entire structure. It helps to look out for any elements which matches with specified relative path . Example for a relative path based locator strategy is as below.

Note: Relative XPath starts with "//"

```
WebElement userId = driver.FindElement(By.XPath("//input"));
// This retrieve first element with input tag.
WebElement userId = driver.FindElement(By.XPath("//input[2]"));
// This retrieve second element with input tag.

```


#### **Relative XPath  - With Attributes**


If we need to further narrow down our location strategy, we can use Attributes along with relative XPath. There may be situations where we need to multiple attributes to uniquely identify an element. We can also specify locators to identify for ANY attribute

```
WebElement passwordField = driver.FindElement(By.XPath("//input[@id='password']"));
// Above will identify first element with input tag which also has id as "password".

WebElement LoginButton = driver.FindElement(By.XPath("//input[@type='submit'and @value='Login']"));
//Note you can use "or" as well.
WebElement someField = driver.FindElement(By.XPath("//input[@*='password']"));
// Above will identify first element with input tag which also has any attribute as "password".
```


#### **Relative XPath - Partial Match**


Sometimes there may be situations where element attributes like ID are dynamically generated. Those will generally have some unique part in attributes likeID and
remaining will be generated dynamically , which will keep on changing. This will need a locator strategy which will help us to identify elements using partial match. Main types are



	
  * starts-with()

	
  * ends-with()

	
  * contains()


```
WebElement passwordField1 = driver.FindElement(By.XPath("//input[starts-with(@id,'password')]"));
// Above will identify first element with input tag which also has id starting with "password".
WebElement passwordField2 = driver.FindElement(By.XPath("//input[ends-with(@id,'password')]"));
// Above will identify first element with input tag which also has id ending with "password".

WebElement passwordField3 = driver.FindElement(By.XPath("//input[contains(@id,'password')]"));
// Above will identify first element with input tag which also has id containing with "password".
```
