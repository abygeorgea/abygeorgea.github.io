---
layout: post
title: "Reading external files  in Cypress using readFile"
slug: "reading-external-files-in-cypress-using-readfile"
date: 2020-12-19T06:21:51+11:00
comments: true
categories:
  - Cypress
keywords:
  - Cypress
  - Data driven testing
  - readFile
description: How to read external files in Cypress
---

In the previous post [here](/blog/2020/12/09/writing-cypress-tests-in-bdd-format) and [here](/blog/2020/12/11/using-datatable-in-cypress-cucumber), we saw how to write BDD test cases using cucumber along with Cypress and to use datatables. As we saw in those examples, we are hard coding few data in feature files. This is not a best practise since we need to modify the feature file for every new set of data. Assuming we need to have different data in different environments, this will make it hard to run the tests across various test environment. Let us look at how we can make read these data from  a file outside of feature file. 

Cypress provides two options to read external files. They are readFile and Fixtures. In this blog post, let us look into readFile method and how to use it.

## readFile

According to [documentation](https://docs.cypress.io/api/commands/readfile.html#Syntax), the command syntax is as below

```
cy.readFile(filePath)
cy.readFile(filePath, encoding)
cy.readFile(filePath, options)
cy.readFile(filePath, encoding, options)

```

cy.readFile() command look for the file to be present in default project root folder .Hence filepath should be specified relative to the root folder . For any files other than JSON format, this command yields the content of the file. For JSON files, the content is parsed into Javascript and returned.



#### FeatureFile ####

Let us write a new scenario to read both text file and json file. We then assert the content of the files. 

![ReadFile-FeatureFileImage](/images/2020/12/19/ReadFile-FeatureFile.png)

#### Step Definition ####

Corresponding Step definition will look like below. Here we get information from datatable and assert the text file content as is.
For JSON files, cypress yields a JSON object . Hence we convert the expected text to json object and assert on its properties.

![ReadFile-StepDefinitionImage](/images/2020/12/19/ReadFile-StepDefinition.png)

#### Test Output ####

Now it is time to run above scenario. Open Cypress by running command 
*npx cypress open*

Run the scenario on cypress UI and result will look like below. We can clearly see that assertions are passed.


![ReadFile-ResultImage](/images/2020/12/19/ReadFIle-Result.png)
