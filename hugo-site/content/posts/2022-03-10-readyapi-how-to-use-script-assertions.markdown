---
layout: post
title: "ReadyAPI_How to use Script Assertions"
slug: "readyapi-how-to-use-script-assertions"
date: 2022-03-10T06:40:30+10:00
comments: true
categories:
  - ReadyAPI
  - SoapUI
keywords:
  - ReadyAPI
  - SoapUI
  - Groovy Scripting
  - Script Assertion
description: How to use script assertion in ReadyAPI
---


In the previous blog post [here](/blog/2022/01/19/readyapi-getting-started-with-readyapi/) and [here](/blog/2022/02/01/readyapi-how-to-make-full-use-of-readyapi-features/),  we saw how to create a functional test case and add assertions to validate the result.
ReadyAPI provides many inbuilt assertion methods which will help to easily validate the output response without any coding. However, in real-world usage for test automation, it may not be enough. Consider the scenario where we need to validate the response content has proper values from an expected list.
In this example, let us make an assertion to validate, that the status field in the response for making the order is either placed or notplaced.

##### Step 1: 

Open up the assertion tab and click on +. Select the Script assertion. This will bring up a new window to write the code. ReadyAPI supports writing code in groovy or javascript. In this example, I am using groovy scripting. This can be defined at project level properties.

![Result](/images/2022/03/10/1_ScriptAssertion.png)

``` Groovy

//Check Valid values for Status is one of the below : placed , notplaced
def jsonResponse = messageExchange.getResponse().contentAsString
log.info "Recevied JSON String : " + jsonResponse
def jsonSlurper = new groovy.json.JsonSlurper();
def actualobject = jsonSlurper.parseText(jsonResponse)
log.info "Current Value of Status : " + actualobject.status;
assert actualobject.status == "placed" || actualobject.status == "notplaced"

```




Script assertion window provides access to some default objects like **log**, **context** and **message exchange**. They contain information about the request and response made.
Details of available methods can be found in Javadoc defined at [here](https://support.smartbear.com/readyapi/apidocs/soapui/index.html)  or in [here](https://support.smartbear.com/readyapi/apidocs/ready-api-core/index.html)

In the first line, we are reading the response of the current step by using the **getResponse()** method of **messageExchange** object. Once we have a string, which is in JSON format, we can use JsonSlurper to parse it into an object. JSON slurper parses text or reader content into a data structure of lists and maps
Once we have an object, we can easily assert whether the value belongs to the expected list

We can directly run the scripts from this editor and see output logs

![Result](/images/2022/03/10/2_assertionPassed.png)

##### Step2 :

We can expand above assertion to do additional validations. If there is a need to check response from current test step against previous steps, we can make use of context object.

``` Groovy
//Check Valid values for Status is one of the below : placed , notplaced
def jsonResponse = messageExchange.getResponse().contentAsString
log.info "Recevied JSON String : " + jsonResponse
def jsonSlurper = new groovy.json.JsonSlurper();
def currentResponseObject = jsonSlurper.parseText(jsonResponse)
log.info "Current Value of Status : " + currentResponseObject.status;
assert currentResponseObject.status == "placed" || currentResponseObject.status == "notplaced"

 def tcobject = context.getTestCase()
 //print name of test case
 log.info "Testcase name is : "+ tcobject.getName()
 //Get list of Test steps in current Test case
  def stepList = tcobject.getTestStepList();
 log.info "Number of test step is : "+ stepList.size()
stepList.each { steps ->
 if(steps.getLabel()== "REST Request- Get details by PetID"){
  log.info steps.getLabel()
    log.info steps.getPropertyValue("Endpoint")
  log.info steps.getPropertyValue("Response")
 def slurper = new groovy.json.JsonSlurper();
def previousResponseObject = slurper.parseText(steps.getPropertyValue("Response"))
assert currentResponseObject.id == previousResponseObject.id
 }      
}


```

In the above code, we are trying to compare the output of the current response with the response from the previous step. This can be achieved through Context objects. From the context object, get details of the test case object. Once we have access to the test case object, then it is easier to expand to the step list
We can easily identify previous steps with a specific name and then extract its response. Parse it again to an object and then make an assertion


![Result](/images/2022/03/10/3_Scriptassertion.png)








