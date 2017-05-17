---
author: Aby George A
comments: true
date: 2016-09-06 12:00:23+00:00
layout: post
link: /blog/2016/09/06/how-to-take-screenshots-with-selenium-in-c/
slug: how-to-take-screenshots-with-selenium-in-c
title: How to take screenshots with Selenium in C#
wordpress_id: 298
categories:
- Selenium
- Specflow
---

Very frequently testers will meet a situation where they need to take screenshot of webpage they are testing , either for base line or as a proof of test result. This is same with automated testing . Even though automated test cases have their own of way of publishing test results, it is always desirable to keep a proof of result.. Screenshot come to help in this regards. In this blog post , I will explain , how to take a screenshot with Selenium Web Driver with C#. In future I will add another couple of post to explain , how to consolidate the screenshots into a PDF document.

In .NET binding, we have an interface called ITakesScreenshot , which helps to capture screenshot of active window. Below code will help to take screenshot and save it in the path specified while calling the function

``` plain

// PathToFolder is the location where we need to save the screenshot
// FileName is another string where PathToFolder is appended with timestamp
public void TakeScreenShot(string PathToFolder)
{
  string fileName = PathToFolder + DateTime.Now.ToString("HHmmss") +".jpeg";
  Screenshot cp = ((ITakesScreenshot)driver).GetScreenshot();
  cp.SaveAsFile(fileName, System.Drawing.Imaging.ImageFormat.Jpeg);

}
```
