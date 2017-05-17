---
author: Aby George A
comments: true
date: 2016-08-01 02:12:23+00:00
layout: post
link: /blog/2016/08/01/specflow-sharing-data-between-steps/
slug: specflow-sharing-data-between-steps
title: Specflow - Sharing data between steps
wordpress_id: 133
categories:
- Specflow
tags:
- BDD
- Data Driven Testing Framework
- Selenium WebDriver
---

In Specflow, Step definitions are global. So a scenario can have multiple step definitions which can be present in different classes.  Sometimes, there arise a need to share the data between steps residing in different classes. How do we do it??

There are multiple ways to do it



	
  1. Context Injection

	
  2. Feature Context

	
  3. Scenario Context


Let us look into more details about how to store and retrieve data using Scenario Context .

**ScenarioContext.Current**

How do we add a key value pair to Scenario Context ? It is as simple as below

```
[Given(@"I have entered (.*) and (.*) into the Login Page")]
public void GivenIHaveEnteredAndIntoTheLoginPage(string p0, string p1)
 {
 ScenarioContext.Current.Add("username", p0);
 ScenarioContext.Current.Add("password", p1);
 }
```

How do we retrieve the value from ScenarioContext ?

```
When(@"I press retrieve data")]
public void WhenIRetrieveData()
{
string username = (string)ScenarioContext.Current["username"];
string password = (string)ScenarioContext.Current["password"];

}
```
Note: While retrieving , scenarioContext.Current always return an object . Hence we need use explicit casting while retrieving data from scenario context.

In Nut Shell,


<blockquote>**Set a value for a key ( Store data ) **
ScenarioContext.Current.Add(string key, object value);

**Get a value of the key ( Retrieve data) **
var value =(Type) ScenarioContext.Current.[string Key];

var value = ScenarioContext.Current.Get(string Key);</blockquote>


We can use this for storing and passing objects as well


<blockquote>Example of storing webdriver object is as below.(where "browser" is current webdriver object )

ScenarioContext.Current.Add("driver1", browser);

IWebDriver driver2 = (IWebDriver)ScenarioContext.Current["driver1"];</blockquote>
