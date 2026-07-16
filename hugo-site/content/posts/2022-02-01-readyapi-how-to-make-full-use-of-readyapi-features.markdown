---
layout: post
title: "How to make full use of readyAPI features"
slug: "readyapi-how-to-make-full-use-of-readyapi-features"
date: 2022-02-01T06:46:12+10:00
comments: true
categories:
  - ReadyAPI
  - SoapUI
keywords:
  - ReadyAPI
  - SoapUI
  - DataSource
  - Property Transfer
  - Conditional GoTo
description: How to use Datasource loop and property Transfer in ReadyAPI
---

In my previous post [here](/blog/2022/01/19/readyapi-getting-started-with-readyapi/), I mentioned how to do a basic functional test. Sometimes, we might have to do complex flows where we need to call multiple APIs and also iterate the tests with various sets of data.

Let us look at one scenario, which involves the below steps.

1) Get details of pet based on PetID

2) If the pet details are retrieved, place an order

3) Iterate the above scenario for different Pet ID

Let us look at how we can implement this

#### Step 1:

Get Details of Pet based on Pet ID. A glance of API functions in ready API or at the [swagger link ](https://petstore.swagger.io/) shows that there is an API endpoint to retrieve details of Pets based on pet ID. So let us get started

Create a new test case in ready API and add the API request to it. This can be done by first navigating to APIs >>SwaggerPetStore>>/pet/{petID}>>getPetByID and right-click on the request and add to test case.

![Result](/images/2022/02/01/1_add_to_testcase.png)

Now add the second API request to make an order. The API can be found at APIs >>SwaggerPetStore>>/store/order>>placeorder, right-click on the request and add to the test case.

![Result](/images/2022/02/01/2_add_to_testcase.png)

#### Step 2:

Look at the test case now and rename the tests to reflect what each request is doing. This can be done by right-clicking on the request and selecting rename option

![Result](/images/2022/02/01/3_rename.png)


#### Step 3:

Test out already added requests. For the first request, enter a pet ID and test. Repeat the same for the Place order. Place order is a POST call and it needs a few inputs like PetID, quantity etc. Manually fill in this step for now and test it.
We will look at how to transfer the details from one API to another later. 

#### Step 4:

To make an order for a pet, we need to pass details from one step to another. This can be achieved by using the property Transfer step. Right-click on test case name and select Add Steps and select Property Transfer

![Result](/images/2022/02/01/4_add_propertyTransfer.png)

By default, it adds a new test step at the end. Drag it to between the previous steps.
Select the property transfer step and click on the + button and give a name for the value we need to pass between API requests. Let us start with PetId. ReadyAPI will automatically select the source and target steps. In this example, we need to transfer the pet id from the response of get details By PetID to the request to Place order
Since the request and response is JSON, we can select JSONPath as the path language. We can then either manual key in JsonPath or use the icon to select the values

![Result](/images/2022/02/01/5_transfer.png)

If we run the test now, it will both test and do the property transfer


#### Step 5:
 
To iterate the test across various data sources, DataSource & DataSourceLoop test steps should be used. Right-click on the test case name and select Add Steps and select DataSource and DataSourceLoop. Rearrange the test steps in such a way that Datasource is the first step and Datasource loop is the last step

Select Datasource and select source type as Grid. It also supports multiple other options like JSON, Excel, XML, JDBC connection, and even a data generator. For simplicity let us use Grid

![Result](/images/2022/02/01/6_datasource.png)


Select Datasource loop . It will ask to select the data source and target step. Make sure to unselect the check box so that details of passed test cases are also saved

![Result](/images/2022/02/01/7_datasourceloop.png)


#### Step 6:

Run the test case by selecting the test case and clicking on the green Run button. This will trigger the test using all specified data sources. Details of the test can be found in Transaction log

![Result](/images/2022/02/01/8_transactionlog.png)


#### Step 7:

If we need to make an order only if pets are available, it can be done by using the Conditional Go to Test step. Add a conditional Go To Test step and select details like below.
Make sure to manually add the conditions as highlighted below

![Result](/images/2022/02/01/9_conditionalGoTo.png)


#### Step 8:

From the results, we can see that the property transfer is not executed.


![Result](/images/2022/02/01/10_Results.png)