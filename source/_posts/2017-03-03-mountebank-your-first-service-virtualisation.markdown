---
author: Aby George A
comments: true
date: 2017-03-03 11:46:48+00:00
layout: post
link: /blog/2017/03/03/mountebank-your-first-service-virtualisation/
slug: mountebank-your-first-service-virtualisation
title: Mountebank - Your first service Virtualisation
wordpress_id: 351
categories:
- Mountebank
- Service Virtualisation
---

In current development world, there will be scenarios were both API and its consumers are developed in parallel. Inorder to decouple their dependencies, we can mock an api response using mountebank. In this example, I will explain how to get started with your first service virtualisation using mountebank. After installing mountebank as mentioned in [here (Install Mountebank)](/blog/2017/02/13/service-virtualisation-using-mountebank/), we will proceed with configuring mountebank. It can be done in few ways. The method which I explain below is by using file based configuration. This involve setting up an imposter file and a stub response


## How to Create a Stub





	
  1. Navigate to mountebank installation path

	
  2. Create a folder and name it as "StubResponse". ( You can name it whatever you want)

	
  3. Create two json file using notepad and save it as "MockResponeForApiOne.json" and "MockResponeForApiTwo.json"( Or what ever you want).

	
  4. Copy paste below code to "MockResponeForApiOne.json" . Sample example only. Update the response and predicates to suite your need ( if required)

``` plain MockResponseForApiOne.json

"responses": [
 {
 "is": {
 "statusCode": 200,
 "body": {
 "Text":"Response ONE ","token":"username","expires_in":90
 }
 }
 }
 ],
 "predicates": [
 {
 "exists": {
 "body" :
 {
 "username": true,"password" : true
 },
 "method" : "POST",
 "path" : "/Apitesting/v1/test?type=ResponseOne"
 }
 }
 ]
```

-

5. Copy paste below code to "MockResponeForApiTwo.json" . Sample example only. Update the response and predicates to suite your need ( if required)

``` plain MockResponseForApiTwo.json
 "responses": [
 {
 "is": {
 "statusCode": 200,
 "body": {
 "Text":"Response TWO ","token":"emailAddress","expires_in":90
 }
 }
 }
 ],
 "predicates": [
 {
 "exists": {
 "body" :
 {
 "email": true,"password" : true
 },
 "method" : "POST",
 "path" : "/Apitesting/v1/test?type=ResponseTwo"
 }
 }
 ]
```


## How to create an Imposter





	
  1. Create another file called test.json in same path as above

	
  2. copy and paste below contents to it


``` plain test.json

{
"imposters": [
{
"port": 4547,
"protocol": "http",
"stubs": [
{
<% include MockResponseForApiOne.json %>
},
{
<% include MockResponseForApiTwo.json %>
}
]
}
]
}

```


## Let us have a close look into Imposter and stubs


Responses – Contains an array of responses expected to return for the defined stub. In the above scenario the response will include status code as 200 and response body. For more info, http://www.mbtest.org/docs/api/contracts
Predicates – is an array of predicates which will be used during matching process. Predicate object can be quite complex, it supports lots of different matching techniques.
For more info, http://www.mbtest.org/docs/api/predicates


## Let's Mock it


Once all required files are created and saved, mountebank can be started by following command in command prompt , after navigating to installation folder of mountebank

``` 

mb --configfile StubResponse/test.json

```

![cmd.jpg](https://seleniumtestingtips.files.wordpress.com/2017/03/cmd.jpg)



Once mountebank is started, we can verify it by navigating to path http://localhost:2525/imposters

It will list out all active ports and a list of stubs available

![imposter](https://seleniumtestingtips.files.wordpress.com/2017/03/imposter.jpg)




## Test It


Once we complete above steps, mountebank is ready with stubs. Now comes the part to test it and use. You can use any api testing tool ( Postman, soapUi etc ) for testing this. Just send the request matching the predicates and look for the responses

Below are the screenshot of Postman request

**Requesting for First API.**

Predicate of response One says that , request has to be of type POST, body of request should have "username" and "password" . Path of the request should have /Apitesting/v1/test?type=ResponseOne"

Now construct a postman request matching above and fire it

![bgone](https://seleniumtestingtips.files.wordpress.com/2017/03/bgone.jpg)





**Request for second API**

Predicate of response One says that , request has to be of type POST, body of request should have "**email**" and "password" . Path of the request should have /Apitesting/v1/test?type=Response**Two**"

Now construct a postman request matching above and fire it

![bgtwo](http://automationtestingtips.files.wordpress.com/2017/03/bgtwo.jpg)



As you can see, both request has succesfully received expected response message

For actual development usage, just point your application to this localhost URL and start consuming virtualised API








