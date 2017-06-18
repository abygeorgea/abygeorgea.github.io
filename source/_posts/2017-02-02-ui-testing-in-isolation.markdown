---
layout: post
title: "UI Testing- decoupling back end dependency"
date: 2017-02-12 07:41:43 +1000
comments: true
categories: 
keywords: 
description: 
---
The traditional approach for automating UI test cases is to create selenium web driver based ( or any UI testing tools) scripts for exercising complete end to end flow. However, it comes with its own challenges. It will have multiple steps as pre-requiste for reaching required UI page and hence it behaves as an E2E integration test rather than UI test.

A typical web application architecture will have one or more front-end application, which will talk to multiple back-end services, API's etc. They will, in turn, talk to other back-end services or to different databases. On High level , architecture looks like below 
![]({{site.images_dir}}/2017/02/02/UIinIsolation_1.png)


 On an enterprise world, all these will be developed and maintained by different teams. All of them will be working in parallel and will push in their code changes ( including occasional broken code) frequently. This will result in breakages since test automation scripts heavily depending on UI and its integration. Even if there is no broken code, a test can still fail due to multiple environmental issues for any of the backend services and other components.  Hence it will become increasingly difficult for achieving a green build. 

Hence UI based test cases are less robust due different reasons like


1. Test depends on external factors which are outside of our control and not part of scope of testing

2. Failing test may not pin point exact location of failure since it is trying to test too many things.

3. There are chances that all components will not be ready when we want to test UI. Hence testing it pushed to the end , which will increase cost of fixing defects.

4. Re - running of test cases may pass (if failure is caused by environmental issues)

5. UI test are brittle by nature since they will even fail  due to timing issues because it is depending on data from back end services.

The solution for above is to adopt more unit test like structure for UI testing. We should be testing UI in isolation to other back-end services and their dependency. This allows testing as much as possible early in lifecycle without any dependency on other streams. We should replace all backend service calls with stubs


![]({{site.images_dir}}/2017/02/02/UIinIsolation_2.png)

Mountebank is a tool which we can use for mocking the service calls. As per [mbtest.org](http://www.mbtest.org/), mountebank is the first open source tool to provide cross-platform, multi-protocol test doubles over the wire.  We can use mountebank for stubbing the back-end service calls and there by use it for decoupling UI from unpredictable back end. 