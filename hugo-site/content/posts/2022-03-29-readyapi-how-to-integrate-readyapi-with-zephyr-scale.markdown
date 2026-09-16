---
layout: post
title: "How to integrate ReadyAPI with Zephyr Scale"
slug: "readyapi-how-to-integrate-readyapi-with-zephyr-scale"
date: 2022-03-29T06:41:00+10:00
comments: true
categories:
  - ReadyAPI
  - Zephyr
keywords:
  - ReadyAPI
  - SoapUI
  - Zephyr Scale
  - Integration
  - GroovyScripting
description: How to Integrate ReadyAPI with Zephyr Scale
---
ReadyAPI has inbuilt support for various test management tools. File >> Preferences will list down all integration with tools like Jira, Zephyr etc. However, at the time of writing this post,  Zephyr Integration is available only with Zephyr Squad and not Zephyr Scale.
If your team is using Zephyr Scale, there is no inbuilt integration. I hope this will change in future since both ReadyAPI and Zephyr Scale is owned by same company.

As of now, if you need to integrate the test execution result back to Zephyr Scale, then custom scripting has to be done. The main steps involved are below

1) Define ProjectID and TestCycleID at the project level. This can be done as Custom Project Properties. Select the Project folder and enter the details in custom project properties section

![Result](/images/2022/03/29/1_ZephyrIntegration.png)

2) Specify Jira Test Case ID for each test case. This can be done by custom test case properties. Create a new custom property for test case called **ID** and specify value as the Jira Key.


2) Add an event to run after every test case run. Click on event and select **TestSuiteRunListener.afterTestcase**. This will make sure that once we run a test suite, the code written in after test case will run after each test case. Please note that, it will run only if we execute test suite. Running a single test case will not trigger this.


![Result](/images/2022/03/29/2_ZephyrIntegration.png)

3) Enter the below code in after test case event. It does below actions.

* Retrieve test case , test suite and project object from test runner.
* Get details like test name, jira test case id, test cycle id and project code ( as defined in step 1 & 2) 
* Identify whether test case is pass or fail
* Post the result into Zephyr. This will need a token id for Zephyr , which you can create in Zephyr ( provided you have access)

Note: 

* Below code updates only one step of test case. Post body content has an element called **testScriptResults** which is an array. If there are multiple steps, it should have more number of elements in an array . The count should match number of steps.
* Zephyr Scale API currently doesnt support adding attachment. Hence if you need to have evidence of test execution in Zephyr, it has to be done either by **actualResult** or **comment** fields.


``` Groovy
       def tcobject = testCaseRunner.getTestCase()
       def tsobject = testCaseRunner.getTestCase().testSuite
       def projobject = testCaseRunner.getTestCase().testSuite.project
       def stepList = tcobject.getTestStepList();

       // Get all test cases’ names from the suite
       def testCaseName = tcobject.name;
       def testCaseID = tcobject.getPropertyValue("ID");
       def testCycleID = projobject.getPropertyValue("TestCycleID");
       def projectKey = projobject.getPropertyValue("projectID");

       log.info "projectKey : $projectKey , TestCaseName :  $testCaseName , TestCaseIID :   $testCaseID, TestCycleID:  $testCycleID  , ";
       if (testCaseID == null || testCaseID.length() == 0 || testCycleID == null || testCycleID.length() == 0 || projectKey == null || projectKey.length() == 0) {
         log.info "MANDATORY FIELDS NOT AVAILABLE. Please check Ready API script to confirm they have all fields defined"
         return 0;
       }

       def comment = "Comments For Jira Execution"

       // Check whether the case has failed
       if (testcaseStatus == 'FAIL') {
         // Log failed cases and test steps’ resulting messages
         log.info "$testCaseName has failed"
         for (testStepResult in testCaseRunner.getResults()) {
           testStepResult.messages.each() {
             msg -> log.info msg
           }
         }

         postToZephyrScale(projectKey, testCaseID, testCycleID, "Fail", comment)
       } else if (testcaseStatus == 'PASS') {
         postToZephyrScale(projectKey, testCaseID, testCycleID, "Pass", comment)
         log.info "$testCaseName  Test Passed";
       }

       log.info "Results updation to Jira is complete"

       def postToZephyrScale(String projectKey, String testcaseKey, String testcycleKey, String status, String comment) {
         def today = new Date()
         def formattedDate = today.format("yyyy-MM-dd'T'HH:mm:ssZ")

         def postmanPost = new URL('https://api.zephyrscale.smartbear.com/v2/testexecutions')
         def postConnection = postmanPost.openConnection()
         postConnection.setRequestProperty("Content-Type", "application/json")
         postConnection.setRequestProperty("Authorization", "ENTER YOUR API TOKEN HERE")
         postConnection.requestMethod = 'POST'

         def form = " {  " +
           " \"projectKey\": \"" + projectKey + "\",  " +
           " \"testCaseKey\": \"" + testcaseKey + "\",  " +
           " \"testCycleKey\": \"" + testcycleKey + "\",  " +
           " \"statusName\": \"" + status + "\",  " +
           " \"testScriptResults\": [  " +
           "   {    " +
           "     \"statusName\": \"" + status + "\",  " +
           "     \"actualEndDate\": \"$formattedDate\",  " +
           "     \"actualResult\": \"" + status + "\"   " +
           "} " +
           " ], " +
           " \"comment\": \"$comment\" " +
           " } "
         log.info form
         postConnection.doOutput = true
         def text
         postConnection.with {
           outputStream.withWriter {
             outputStreamWriter ->
               outputStreamWriter << form
           }
           text = content.text
         }

         if (postConnection.responseCode == 200 || postConnection.responseCode == 201) {

         } else {
           log.info "Posting to Jira failed" + postConnection.responseCode
         }
       }


```



