---
author: Aby George A
comments: true
date: 2017-04-06 19:43:44+00:00
layout: post
link: /blog/2017/04/07/mountebank-creating-a-response-based-on-a-file-template-and-modifying-it-based-on-request-part-1/
slug: mountebank-creating-a-response-based-on-a-file-template-and-modifying-it-based-on-request-part-1
title: Mountebank - Creating a response based on a file template and modifying it
  based on request - PART 1
wordpress_id: 461
categories:
- Mountebank
- Service Virtualisation
---

In the previous two blog post, I have explained about how to setup mountebank ([here]({{site.root}}blog/2017/02/13/service-virtualisation-using-mountebank/)) and how to create a virtualised respone([here]({{site.root}}blog/2017/03/03/mountebank-your-first-service-virtualisation/)) . Now coming to more detailed use cases which we might encounter in daily life. In this blog post, I will explain how we can use mountebank to create a virtualised response based on a template response stored in a file and modifying certain fields in response based on the request coming through.

In below Step by Step example , I will have two mock responses for searching for a customer details. First response is when customer is not available in back end systems and second response is when customer details are found.

Before we start, below is folder structure which I have and in this blog post we are discussing about only one stubbed response, which is the NOT FOUND scenario.

![folderstructure]({{images_dir}}/2017/04/folderstructure.png)

Let us first create the imposter.ejs file

``` plain Imposter.ejs

{
"imposters": [

<% include 4547.json %>

]
}

```



Now let us create the file which specifies the port number where it should run and order of responses. Below code tells mountebank that port which it needs to listen for incoming request is 4547 and protocol is http. There are two set of mock responses planned.



``` plain 4545
{
"port": 4547,
"protocol": "http",
"stubs": [

{
<% include CustomerNotFound.json %>
},
{
<% include CustomerFound.json %>
}
]
} 

```



In this example, let us look at first mock response.

``` plain CustomerNotFOund.json
"responses": [
{
"inject": "<%- stringify(filename, 'ResponseInjection\\GetCustomerNotFound.js') %>"
}

],
"predicates": [
{
"matches": {
"method" : "GET",
"path" : "/Blog.Api/1[0-9]+/CustomerView"
}
}
]
```

From above response, we can infer below. When ever an http GET request come to port 4547 , with a path matching "/Blog.Api/1[0-9]+/CustomerView', then we will call the Javascript function "GetCustomerNotFound.js" which is kept inside a directory "ResponseInjection" in same location. It is also good to notice that , predicate is a regular expression ( hence use matches) and all request where 1 followed by any number of numeric will be returned with this response

The javascript function listed here is responsible for reading the sample template response and sending it back .

``` plain GetCustomerNotFound.js
function GetTemplateResponse (request, state, logger) {

response = JSON.parse("<%- stringify(filename, 'StubTemplate\\CustomerNotFoundView.json') %>");

return {
statusCode : 404,
headers: {
'Content-Type': 'application/json; charset=utf-8'
},
body: response

};
}
```





Above function reads a json response kept inside directory "StubTemplate" and convert it to json and return to mountebank. Since this is for a scenario where customer records are not found,we set the status code as 404. We can also set the headers if needed







The stub template is as below






``` plain CustomerNotFoundView.json
{
"status": "fail",
"code": "CUSTOMER_NOT_FOUND",
"message": "Customer details not found."

}

```









Now let us run mountebank







![mountebank]({{images_dir}}/2017/04/mountebank.png)







Request through postman




![notfound.png]({{images_dir}}/2017/04/notfound.png)







As you can see , the GET request matching with predicate is returning the stubbed response with status 404.




For real time usage for testing any web application which needs to get a 404 message from back end API calls, just point the end point to this local host end point and fire a request which matches the predicate.







Details of second response will be shared in next blog post
