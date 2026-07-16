---
layout: post
title: "Data driven testing in Cypress using Fixtures"
slug: "data-driven-testing-in-cypress-using-fixtures"
date: 2020-12-19T11:10:44+11:00
comments: true
categories:
  - Cypress
keywords:
  - Cypress
  - Data driven testing
  - Fixtures
description: How to do data driven testing in Cypress
---

In the previous post [here](/blog/2020/12/19/reading-external-files-in-cypress-using-readfile) we saw how to read external files in Cypress using cy.readFile. Cypress also provides another way to read files. In this post, I will show how to Fixtures to do data driven testing in Cypress. 

####Syntax####

According to [documentation](https://docs.cypress.io/api/commands/fixture.html#Syntax), syntax is as below.

```
cy.fixture(filePath)
cy.fixture(filePath, encoding)
cy.fixture(filePath, options)
cy.fixture(filePath, encoding, options)

```

####Comparison with cy.readFile####

Main difference between cy.readFile and cy.Fixture is that former one starts looking for the files from project root folder and later looks for files under *Fixture* folder. cy.Fixture supports a wide range of file types/extensions like json, txt, html,jpeg,gif,png [etc](https://docs.cypress.io/api/commands/fixture.html#JSON). If file name is not specified it looks for all supported filetypes in specific order starting with JSON. We can even assign alias to this , which will help to reuse this later on. 

####Example####

To demonstrate this, let's revisit old example of logging into SauceDemo website. This time, instead of using hardcoded values in feature file, I will move credentials into a separate JSON file and keep it inside *Fixtures\TestDataFiles* folder .  File should look like below. This has two set of login credentials.

``` javascript
[
    
    {
        "UserType" :"LockedOutUser",
        "UserName" : "locked_out_user",
        "Password" : "secret_sauce"
    },
    {
        "UserType" :"StandardUser",
        "UserName" : "standard_user",
        "Password" : "secret_sauce"
    }
]
```

####Feature File####

Create a new scenario to login to ECommerce site by using Fixtures. In below scenario we just specify type of User which we need to use for this scenario. I am specifying a user type here since there are different types of users and we can have different scenarios for them. 

``` Gherkin
@focus
  Scenario: Logging into ECommerce Site as Standard User
    Given I Login to Demo shopping page as 'StandardUser'
    Then I should see products listed
    
```

#### Step Definition ####

Step definition file for this will look like below. In Step definition files, below steps are performed.

* Reading test data file by using cy.fixture() by passing relative path from Fixture folder and then alias it into a variable *loginCredentials*. The hierarchy is separated by forward slashes.
* Get the JSON object ( which is an array of objects) from above step and identify the specific object which we need to use for this step based on the UserType specified in feature file. This is achieved by using *filter* method. This is mostly like LINQ in C# . More details can be found [here](https://sung.codes/blog/2018/02/24/approximate-equivalent-linq-methods-javascript/#where) . This filter will return an array of object which satisfy filter criteria specified. In this case, there will be only one object in array.
* Login to the demo website by using first user in the above array.

``` javascript

//Navigate to URl and login using credentials from Fixture
const url = 'https://www.saucedemo.com/index.html'
Given('I Login to Demo shopping page as {string}', (UserTypeValue) => {
  cy.visit(url);
  
  //Reading Json file and then alias it
  cy.fixture('TestDataFiles/LoginCredentials.json').as('loginCredentials');
  cy.log('Value passed in' +UserTypeValue);
  
  //Use alias and identify the object which matched to the information passed from feature file
  cy.get('@loginCredentials').then((user) => {
     
     // Find the object corresponding to UserType passed in
      var data = user.filter(item => (item.UserType == UserTypeValue));
      
      
      //printout details
      var propValue;
      cy.log('filtered data :'+data[0]);
      for(var propName in data[0]) {
        propValue = data[0][propName]
  
        cy.log(propName,propValue);
        }
        
       //Login
      cy.get('#user-name').type(data[0].UserName);
      cy.get('#password').type(data[0].Password,{log:false});
  });
  
    cy.get('#login-button').click();
});
```

#### Test Output ####

Now it is time to run above scenario. Open Cypress by running command 
*npx cypress open*

Run the scenario on cypress UI and result will look like below. We can clearly see that assertions are passed.


![Fixture-ResultImage](/images/2020/12/19/Fixture-Result.png)

