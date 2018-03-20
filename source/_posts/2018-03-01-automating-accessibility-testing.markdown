---
layout: post
title: "Automate Accessibility Testing using aXe"
date: 2018-03-01 21:30:32 +1100
comments: true
categories: Accessibility testing
keywords: Accessibility testing
description: How to automate accessibility testing using aXe
---

### What is Accessibility testing ?

It is a kind of testing performed to ensure application under test is usable by people with disabilities. One of the most common accessibility testing for web applications is to ensure it is easily usable by people with vision impairment. They normally use screen readers to read the screen and use key board to navigate. 

Web Content Accessibility Guideline (WCAG) list down guidelines and rules for creating accessible website. There are various browser extensions and developer tools available for scanning web pages to find out obvious accessibility issues. aXe is one of the widely used extension. Details of aXe can be found [here](https://www.deque.com/products/axe/).  Once browser extension is installed, you can analyze any web page to find out accessibility issues. They also have a javascript API for aXe core .

I recently came across [axe-selenium-csharp](https://github.com/javnov/axe-selenium-csharp) , which is a .NET wrapper around aXe. It is relatively very easy to setup and use.  Below are the steps 

1. Install Globant.Selenium.Axe nuget package for solution. This will add reference to dll 
2. Import namespace *using Globant.Selenium.Axe*
3. Call "Analyze" method to run accessibility check on the current page.

``` csharp
using Globant.Selenium.Axe

public void PerformAccessbilityAudit(IWebDriver _driver) {

 private AxeResult _results;
 _results = _driver.Analyze();
 
  foreach (var xyz in _results.Violations)
            {
                log.Info(xyz.Impact.ToString());
                log.Info(xyz.Description.ToString());
                log.Info(xyz.Id.ToString());
            }
  Assert.True(_results.Violations.Length == 0, "There are accessibility violations. Please check log file");
}

```



Automated accessibility testing is NOT a completely foolproof solution. We will still require someone to scan the page using screen reader software later. But this will help to move accessibility testing to left and have more frequent runs and reduce the need for regression.