---
layout: post
title: "Stubbing XML responses using Mountebank"
date: 2017-04-27 05:55:51 +1000
comments: true
categories: 
- Mountebank
- Service Virtualisation
---

Previous two blog post talked about how we can use mountebank for stubbing where responses are in json format . They can be accessed ([here]({{site.root}}blog/2017/04/07/mountebank-creating-a-response-based-on-a-file-template-and-modifying-it-based-on-request-part-1/)) and ([here]({{site.root}}blog/2017/04/07/mountebank-creating-a-response-based-on-a-file-template-and-modifying-it-based-on-request-part-2/)). We can use same approach for stubbing SOAP services using XML as well. In this post, I will explain how we can provide XML response using Mountebank .

Let us have a quick look into the files created. Before we begin, folder structure of various file as below

![folderstructure]({{site.images_dir}}/2017/04/27/Mountebank_XML_Response_Folder-Tree.jpg)

#### Imposter.ejs ####
The main Imposter file is 
``` plain Imposter.ejs
{
"imposters": [
<% include Port4547.json %>
]
}
```

####  Port4547.json ####

This file specifies which port number to use and what all stubs needs to be created is as below
``` plain Port4547.json
{
"port": 4547,
"protocol": "http",
"stubs": [
{
<% include XMLStubGET.json %>
},
{
<% include XMLStubPOST.json %>
}
]
}
```
####  XMLStubGET.json ####
This is the first stub for this example and it looks for any request coming with the method "GET" and path "/Blog.Api/[0-9]+/CustomerView" , where [0-9]+ is regular expression of any numeric

``` plain XMLStubGET.json
"responses": [
{
"inject": "<%-stringify(filename, 'ResponseInjection\\GetXMLStub.js') %>"
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
#### XMLStubPOST.json ####

This is the second stub for this example and it looks for any request coming with method "POST" and path "/Blog.Api/XMLexamplePOST/[0-9]+" , where [0-9]+ is regular expression of any numeric .It also needs a body as <Action>Insert</Action><Record>Customer1</Record>

Note: If you have body in multi-line, then make sure to enter "\n" for new line

``` plain XMLStubPOST.json
"responses": [
{
"inject": "<%-stringify(filename, 'ResponseInjection\\GetXMLStub-POST.js') %>"
}
],
"predicates": [
{
"matches": {
"body" : "<Action>Insert</Action><Record>Customer1</Record>",
"method" : "POST",
"path" : "/Blog.Api/XMLexamplePOST/[0-9]+"
}
}
]
```

#### GetXMLStub.js ####

Below js file create a response based on template mentioned and return the response with proper status. Please note that, we are not using "Json.Parse" here as we did for previous examples involving json.

``` plain GetXMLStub.js
function GetTemplateResponse (request, state, logger) {
response = "<%- stringify(filename, 'StubTemplate\\CustomerDetails.xml') %>"
return {
statusCode : 200,
headers: {
'Content-Type': 'application/xml; charset=utf-8'
},
body: response
};
}
```


#### GetXMLStub-POST.js ####

``` plain GetXMLStub-POST.js
function GetTemplateResponse (request, state, logger) {
response = "<%- stringify(filename, 'StubTemplate\\RecordAdded.xml') %>"
return {
statusCode : 200,
headers: {
'Content-Type': 'application/xml; charset=utf-8'
},
body: response
};
}
```


#### CustomerDetails.XML ####

This is the template for the first stub - GET example

``` plain XML
<customer>
  <FirstName>John</FirstName>
  <LastName>Citizen</LastName>
  <Address>Some St, Some State, Some Country</Address>
  <Email>Test@test.com</Email>
</customer>
```

#### RecordAdded.xml ####

This is the template for the second stub - POST example

``` plain XML
<Status>Added</Status>
<Record>Customer1</Record>
```

After creating above files and keeping them as per directory structure is shown above, it is time to start mountebank

> mb --configfile SOAP-XMLStubExample/Imposter.ejs --allowInjection


Note: Give the right path to Imposter.ejs . If you need to debug Mountebank, you can use below command at the end " --loglevel debug"

Now trigger a get request to http://localhost:4547/Blog.Api/3123/CustomerView.

This should match with our first predicate and should return the response mentioned

Mountebank_XML_Response_



Now trigger a POST request with a body . If predicates are matched, then it will respond with expected response as below

![PostManRequest]({{site.images_dir}}/2017/04/27/Mountebank_XML_Response_PostmanRequest1.png)

In Nut shell, creating a XML response is similar to creating json response. There are only minor differences in the js file which creates the response. The main difference is the omission of Json.Parse and also changing the response headers.

Above examples can be cloned from my GitHub repository [here](https://github.com/abygeorgea/MountebankExamples). After cloning the repository to local, just run RunMounteBankStubsWithSOAPXMLStubExampleData.bat file. Postman scripts can also be found inside PostmanCollections Folder to testing this