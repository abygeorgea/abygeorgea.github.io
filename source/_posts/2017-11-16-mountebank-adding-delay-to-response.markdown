---
layout: post
title: "Mountebank - Adding delay to response"
date: 2017-11-16 09:59:17 +1100
comments: true
categories: [Mountebank, Service virtualization]
keywords: Mountebank, response
description: How to add delay to api response in mountebank
---

In previous blog post, I have explained about how to create a json response in mountebank. You can read about that [here]({{site.root}}blog/2017/04/07/mountebank-creating-a-response-based-on-a-file-template-and-modifying-it-based-on-request-part-1/) and [here]({{site.root}}blog/2017/04/07/mountebank-creating-a-response-based-on-a-file-template-and-modifying-it-based-on-request-part-2/). Recently , I had to test a scenario about what will happen to application if downstream API response is delayed for some time . Let us have a look about how we can use mountebank to simulate this scenario.

Mountebank supports adding latency to response by adding a behaviour. You can read about that [here](http://www.mbtest.org/docs/api/behaviors) . Let us try to implement the wait behaviour in one of the previous examples . This is a slight modification of the files used as part of examples mentioned [here]({{site.root}}blog/2017/04/07/mountebank-creating-a-response-based-on-a-file-template-and-modifying-it-based-on-request-part-1/) and [here]({{site.root}}blog/2017/04/07/mountebank-creating-a-response-based-on-a-file-template-and-modifying-it-based-on-request-part-2/).  You can clone my github repo and look at "ExamplesForWaitBehaviour" for the files.

The only change which we need is to add a behavior to the response. This is added in "CustomerFound.json" file. After injecting file, we need to add behavior for waiting 5000 milliseconds. 

```

"responses": [
{
"inject": "<%-stringify(filename, 'ResponseInjection\\GetCustomerFound.js') %>",
"_behaviors": {
    "wait": 5000
  }
}
],
"predicates": [
{
"matches": {
"method" : "GET",
"path" : "/Blog.Api/[0-9]+/CustomerView"
}
}
]

```

Now run Mountebank. If you are using the GitHub repo, you can do this by running RunMounteBankStubsWithExampleForWait.bat file. Else run below command inside the directory where mountebank is available. If needed, modify the path to Imposter.ejs as required.

```
mb --configfile ExamplesForWaitBehaviour/Imposter.ejs --allowInjection
```

When we trigger a request via postman, we will get a response after specified delay + time for getting a response.  Have a look at response time in below screenshot. Response time is more than 5000 ms.


![PostManRequest]({{site.images_dir}}/2017/11/16/Mountebank_Adding_Delay_PostmanRequest1.png)