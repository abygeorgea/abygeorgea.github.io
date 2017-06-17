---
layout: post
title: "UI Testing in isolation"
date: 2017-02-12 07:41:43 +1000
comments: true
categories: 
keywords: 
description: 
---
Traditional approach for automating UI test cases is to create selenium web driver based ( or any UI testing tools)  scripts for exceresing complete end to end flow. However it comes with its own challenges.

A typical web application architecture will have one or more front end application , which will talk to multiple back end services , api's etc. They will in turn talk to other back end services or to different databases.
On High level , it looks like below
![]({{site.images_dir}}/2017/02/02/UIinIsolation_1.png)

 On a enterprise world, all these will be developed and maintained by different teams. All of them will be working in parallel and will push in their code changes ( including occasional broken code) frequently . This will result in breakages test automation scripts written for heavily depending on UI. Even if there is no broken code, test can still fail due to multiple environmental issues for any of the backend services and other components.  Hence it will become increasingly difficult for achieving a green build . This will also make test cases less robust 
due different reasons like

1. Re - running of test cases may pass (if failure is caused by environmental issues)

2. Test depends on external factors which are outside of our control and not part of scope of testing

3. Failing test may not pin point exact location of failure since it is trying to test too many things.

4. There are chances that all components will not be ready when we want to test UI. Hence testing it pushed to the end , which will increase cost of fixing defects.

Solution for above is adopt more unit test like structure for UI testing . We should be testing UI in isolation to other back end services and their dependency. This allows to test as much as possible early in lifecycle without any dependency on other streams. we should replace all backend service calls with stubs


![]({{site.images_dir}}/2017/02/02/UIinIsolation_2.png)