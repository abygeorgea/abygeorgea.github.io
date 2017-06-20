---
layout: post
title: "Automated  testing of CSS for Responsive Web Design"
date: 2017-06-21 06:15:55 +1000
comments: true
categories: Galen
keywords: 
- Responsive web Design
- Galen framework
description: How to do automated responsive design testing
---

In a world where mobile first seems to be the norm, testing of look and feel of websites on various mobile/tablet devices are essential. More businesses are now adopting Responsive Web designs for developing their web applications and sites.


###What is Responsive Web Design###

According to [Wikipedia](https://en.wikipedia.org/wiki/Responsive_web_design), Responsive web design (RWD) is an approach to web design aimed at allowing desktop webpages to be viewed in response to the size of the screen or web browser one is viewing with. In addition, it's important to understand that Responsive Web Design tasks include offering the same support to a variety of devices for a single website. A site designed with RWD adapts the layout to the viewing environment by using fluid, proportion-based grids, flexible images, and CSS3 media queries, an extension of the @media rule, in the following ways:


- The fluid grid concept calls for page element sizing to be in relative units like percentages, rather than absolute units like pixels or points.
- Flexible images are also sized in relative units, so as to prevent them from displaying outside their containing element.
- Media queries allow the page to use different CSS style rules based on characteristics of the device the site is being displayed on, most commonly the width of the browser

### How do we test responsiveness ###

An ideal option for testing is to test on different physical devices of various screen size. However, it is impossible to get hold of all available mobile/tablet devices in the market. Even if we prioritize the devices using analytics, it is very expensive to buy enough number of devices. Along with this, we need to upgrade to newer version of devices frequently when apple/google/Samsung releases an upgraded version.

Next possible option is to use device emulators like device mode in Chrome dev tools. As pointed out in their [documentation](https://developers.google.com/web/tools/chrome-devtools/device-mode/), it is only a close approximation of how the website will look on a mobile device.It have its own limitations which are listed [here](https://developers.google.com/web/tools/chrome-devtools/device-mode/emulate-mobile-viewports)

Best approach will be to use emulators early in development cycle and once UX design is stabilized, then test it on physical device based on priority obtained by analytics.

###Challenges in testing Responsive Websites###

Testing of responsive websites has its own challenges. 

 - Number of Options to be tested or number of  breakpoints which needs to be validated are high
 - Distinctive UI designs for different device screen sizes makes testing time consuming. This adds complexity to testing since it will require testing of below in various screen sizes 
     - All UI elements like image, text, controls are aligned properly with each other and doesn't overflow from screen display area
     - Consistency in font size, color, shades , padding, display orientation etc
     - Resizing of controls which take inputs (like text) to cater for long content typed in by users.
     - Other CSS validation specific for mobile and tablet devices

 - It is hard to test all of the above on every iteration manually.
 - Comparing UI & UX & Visual design will require more efforts.
 - Hard to keep track of every feature that needs to be tested and will have testing fatigue which will result in Non-obvious changes to UI

### Automated Responsive Design testing - Galen Framework ###

As mentioned above, one of the pain points in responsive design testing is the user fatigue happening over multiple iterations of testing. This can be easily avoided by having an automated test framework. I recently came across galen framework which is an open source framework to test layouts of webpages. You can read about Galen framework [here.](http://galenframework.com/)
Galen framework can be used for automation of CSS testing easier. It has evolved over time and has its own Domain specific language and commands which can be used for CSS testing. I will go through galen framework in more details in next post
