---
layout: post
title: "Setting Right expectation about benefits of Test automation"
date: 2017-09-07 22:42:15 +1000
comments: true
categories: Test automation
keywords: 
description: Managing stakeholder expectation on value delivered from test automation
---

Today I had a discussion with a project manager about stake holder expectations about value delivered from regression test automation and how to manage stake holder expectation. The discussion soon spanned on to challenges in automating manual test cases and candidates for automation testing.


### Expectation Vs Reality ###
Management Stakeholders always visualize test automation as a silver bullet for fixing all pain points.They envision automation tests to be quicker, cheaper and effective in identifying all defects. Automation test cases are expected to be run on a button click and with 100% pass rate (except for valid bugs). Needless to say, that expectation is about having complete test coverage for automation scripts. Thinking is always geared towards reducing manual testers based on automation progress rather than having focus on improved quality of final product, faster time to market etc.

The ground reality is different from above expectation. Automated test cases are only as good as how you script it to be. Automated checks will alert tester about problems that checks have been programmed to detect.It ignores all other problems outside of it. Cost, speed, and ROI will depend on the tool used and complexity of tests implemented.   Having an automated test is not a replacement for doing exploratory test manually. We need to cater for manual exploratory testing since automated scripts can only do verification of already known check points( for which the coding is done) and miss out check points which are not automated. In other words, test automation frees up tester's time to focus more on exploratory testing which adds value.



Challenge in this specific case is to automate E2E manual regression test cases which are not existing. The testers are supposed to identify the regression test cases first by going to through existing application and then automate them. The expectation is that testers will identify all possible error scenarios and incorporate corresponding checks in automated scripts. This is going to be time consuming and expensive. It depends on domain knowledge of the person who creates automation test cases. There are chances that all existing bugs will be considered as an expected behaviour. More over the end to end test cases done at UI level is generally time consuming to develop, slow to execute and heavily depended on UI which makes it brittle. 

### Testing Pyramid###
Solution to improve quality of a product is to follow the [testing pyramid](https://martinfowler.com/bliki/TestPyramid.html) and try to automate more at lower levels instead of focussing at E2E level.  This also has to be done while the product/software is developed. 

Below is a modified version of testing pyramid.

![]({{site.images_dir}}/2017/09/07/TestingPyramid.jpg)

As you can see above, more emphasis is given to have automated test at Unit test level, followed by component level, integration test level, and finally E2E level through UI.  It is relatively cheaper to implement automated test at the base of the pyramid and will get more expensive as we go up. Similarly, unit tests are faster to run, it can isolate issues immediately and are more stable. These characteristics will change adversely as we go up in test pyramid. 


As obvious, it is not feasible to achieve this for an already existing system without having a significant investment in people, time and tools. This will impact ROI. Depending on situations, there is no right or wrong way to do test automation. Having something is always better than nothing. Hence when there is a need to automate regression test cases, it normally starts from the top.  Significant investment is needed upfront to identify all critical regression test cases and corresponding validation that should be performed by automated tests. It is not feasible to automate all test or to have 100% coverage. Success rates of automation script run will depend on various factors like test data, environment stability etc. Everyone should understand that we are automating check point verifications and hence it does trigger alerts only for the checks which it is programmed to do. E2E regression through UI should be only a minimal subset of what is covered through other levels. We should be ready to invest in maintaining the automation assets over a period of time. 

In this case, stakeholder expectation needs to be carefully managed. It is important to set right expectation about benefits offered by test automation for a successful project delivery. Automation testing can deliver benefits over a long period of time , provided proper planning was done upfront to automate at different levels of testing. Instead of considering it as solution for all pain points, we need to clearly articulate /set expectation about its limitations and long term benefits.

