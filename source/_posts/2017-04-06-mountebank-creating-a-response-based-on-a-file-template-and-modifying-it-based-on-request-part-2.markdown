---
author: Aby George A
comments: true
date: 2017-04-06 20:24:20+00:00
layout: post
link: /blog/2017/04/07/mountebank-creating-a-response-based-on-a-file-template-and-modifying-it-based-on-request-part-2/
slug: mountebank-creating-a-response-based-on-a-file-template-and-modifying-it-based-on-request-part-2
title: Mountebank - Creating a response based on a file template and modifying it
  based on request - PART 2
wordpress_id: 547
keywords: Mountebank , Stub , Service Virtualisation , Soap Response, XML response
description: How to stub a  response based on a sample template using Mountebank
categories:
- Mountebank
- Service Virtualisation
---

This is an extension to my previous blog about how we can use mountebank to create a stubbed response based on a template file . You can read about it [here]({{site.root}}blog/2017/04/07/mountebank-creating-a-response-based-on-a-file-template-and-modifying-it-based-on-request-part-1/).  In this step by step example, I will explain how we will use mountebank to modify the response based on the request . Before we start, please ensure you are familiar with [Part1 ]({{site.root}}blog/2017/04/07/mountebank-creating-a-response-based-on-a-file-template-and-modifying-it-based-on-request-part-1/)of the excercise. If you need to know more about mountebank and how to use mountebank , please read through [how to install mountebank ]({{site.root}}blog/2017/02/13/service-virtualisation-using-mountebank/) and [service virtualisation using mountebank]({{site.root}}blog/2017/03/03/mountebank-your-first-service-virtualisation/).

As in previous example, let us create Imposter.ejs and 4547.json . Contents of the Imposter.ejs is as below
``` plain Imposter.ejs

{
 "imposters": [

 <% include 4547.json %>

 ]
}

```

Contents of 4547.json is as below



**4547.json**

``` plain 4547.json

{
 "port": 4547,
 "protocol": "http",
 "stubs": [

{
 <% include  CustomerNotFound.json %>
 },
 {
  <% include  CustomerFound.json %>
 }
 ]
 }

```



Now create CustomerFound.json

``` plain CustomerFound.json

 "responses": [
 {
 "inject": "<%-stringify(filename, 'ResponseInjection\\GetCustomerFound.js') %>"
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



As we can see from above, if there is request which matches the predicates , then response will be dictated by the GetCustomerFound javascript file kept inside directory ResponseInjection. Predicate used here is a GET request which have a matching path of /Blog.Api/[0-9]+/CustomerView.

Contents of GetCustomerFound.js is

``` plain GetCustomerFound.js
function GetTemplateResponse (request, state, logger) {

response = JSON.parse("<%- stringify(filename, 'StubTemplate\\CustomerFoundView.json') %>");
 var ext =require('../../../StubResponse/ResponseInjection/extractrequest');

var reqdata = ext.extractor(request);

 response.data.customerID=reqdata.CustomerID;

 return {
 statusCode : 200,
 headers: {
 'Content-Type': 'application/json; charset=utf-8'
 },
 body: response

 };
}

```

The javascript file have a single function , which reads the stubbed response kept in template file . Then it calls another Javascript function to called "extractrequest". We will see the details of it soon. For now, it actually returns the customer number from the request . For eg, if request is "http://localhost:4547/Blog.Api/3123/CustomerView " then it return 3123 as customer ID. Once we extract the customer ID, then it will replace the customer ID in our template response with the value coming from request and return the response.

Let us take a close look at the extractrequest function.

``` plain extractrequest.js

module.exports = {extractor:function extractCIFAndPackageID (request) {

if(request && request.path) {
var req = request.path.split('/');
if(req.length >2 && req[1]) {
return { CustomerID: req[2] }
}
}

return null;
}}

```

This method will take the input parameter as the request and split it at "/" to get a an array . Then we will return the array[2] which is the customer ID from the request



Finally , the template response

**CustomerFoundView.json**

``` plain CustomerFoundView.json

{
"status": "success",
"code": 0,
"message": "",
"data":
{

"customerID": "123",
"firstName": "John",
"lastName": "Citizen",
"email": "John.Citizen@abcabacas.com"

}

}

```

Now let us fire up mountebank

![mountebank]({{site.images_dir_oldwordpress}}/2017/04/mountebank.png)

Make few request using postman, which have different request parameter

![customerFound1]({{site.images_dir_oldwordpress}}/2017/04/customerfound1.png)

Another request

![CustomerFound2]({{site.images_dir_oldwordpress}}/2017/04/customerfound2.png)

In above two examples,we  can see the CustomerID field is response is updated with number extracted from request.



Now let us try another example , where request is http://localhost:4547/Blog.Api/1234542323/CustomerView

![CustomerNotFound2]({{site.images_dir_oldwordpress}}/2017/04/customernotfound2.png)

As you can see, we are getting a customer Not found response. This is due to the order of predicates we use. In our 4547.json, the order of response are as below.



	
  1.  Customer Not found which has a predicate of "/Blog.Api/1[0-9]+/CustomerView"

	
  2. Customer found which has a predicate of "/Blog.Api/[0-9]+/CustomerView"


As you can see from above order, when a request comes through , mountebank will first match with predicate of first response and if it matches, it returns the response. If not, mountebank will keep trying with next one followed by all others. In this particular example, since our request have a customer ID of 1234542323, it matches with regular expression of first one ( 1[0-9]+)  and hence it return customer not found response.

In next blog post, I will provide more insights about how to extract request from different type of requests.




