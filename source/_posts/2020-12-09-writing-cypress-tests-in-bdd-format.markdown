---
layout: post
title: "Writing Cypress tests in BDD format"
date: 2020-12-09 13:46:17 +1100
comments: true
categories: [cypress, bdd, cucumber]	
keywords: [cypress, bdd, cucumber]	
description: How to write cucumber tests in Cypress
---

As discussed in previous posts [here]({{site.root}}blog/2020/11/25/how-to-validate-xhr-in-cypress) and [here]({{site.root}}blog/2020/11/19/how-to-read-browser-cookies-in-cypress), we use Cypress for writing tests to validate GUI and API tests. In many companies we use behaviour driven development practises where the requirements or acceptance criteria are specified in Gherkin format. As a results, test automation scenarios are also written in Gherkin format. Cucumber , Specflow etc are some of such framework used extensively . In this post, let us examine how we can write BDD test cases in Cypress.


First step is to identify , how we can run Gherkin sytanxed specs with Cypress. This is done with help of [cypress-cucumber-preprocessor](https://github.com/TheBrainFamily/cypress-cucumber-preprocessor). 

##Installation##
Installation of this package is straight forward

```

npm install --save-dev cypress-cucumber-preprocessor

```

##Configure##
Once NPM package is installed, then next step is to configure Cypress to use it. It consist of 3 main steps.

* Add below code to cypress/package/index.js  file.

```javascript

const cucumber = require('cypress-cucumber-preprocessor').default

module.exports = (on, config) => {
  on('file:preprocessor', cucumber())
}
```

* Modify package.json to add below line

``` javascript
"cypress-cucumber-preprocessor": {
  "nonGlobalStepDefinitions": true
}
```

nonGlobalStepDefinitions is set as true, which means Cypress Cucumber Preprocessor Style pattern will be used for Step definitions. Default value is false which means old cucumber format of everything global will be used.

There are some other configuration values which we can specify . Details are available in above project documentation link. 


* Add support for feature files in cypress.json.

``` javascript
{
  "testFiles": "**/*.{feature,features}"
}
```

Normally browser is relaunched for each feature files which will result in extended test execution time. cypress-cucumber-preprocessor provides a way to combine all files into a single *features* file. Above snippet shows that we can expected files named as .feature and .features.


## Feature Files ##
Now it is time to add a feature file. Add a *demo.feature* file inside Integration/Feature folder. Add below Gherkin into that file

``` Gherkin
Feature: Demo for BDD in Cypress

  I want to demo using BDD in Cypress
  
  @focus
  Scenario: Logging into ECommerce Site
    Given I open Demo shopping page
    When I login as "standard_user" user
    Then I should see products listed
    
```
  
  
## Step Definitions ##
 
   Recommended way is to create Step definiton files in a folder named as Feature and keep it inside where we have feature files. Let us create a *demosteps.js* file. 
  
 Location of feature file : Integration/Feature/demo.feature.
 
 Location of StepDefinition : Integration/Feature/demo/demosteps.js
 
 Note: Recommendation from team is to avoid using older way of having Global step definitions defined in cypress/support/step_definitions. More about the rationale for that is available in github documentation. We have specified to use latest style in package.json earlier.
  
``` javascript
// Import Given , When, then from cypress-cucumber-preprocessort steps
import { Given, When, Then } from "cypress-cucumber-preprocessor/steps";

//Navigate to URl
const url = 'https://www.saucedemo.com/index.html'
Given('I open Demo shopping page', () => {
  cy.visit(url)
});

//Type in user name and password. Username is passed from feature file. For demo purpose password is hard coded
// We specify what is the type of variable in step ..See {string}
//Password is not logged by providing the option
When("I login as {string} user", (username) => {
    cy.get('#user-name').type(username);
    cy.get('#password').type('secret_sauce',{log:false});
    cy.get('#login-button').click();
  });


//Get list of children and assert its length
Then('I should see products listed', () => {
  cy.get('div.inventory_list').children().should('have.length', 6);
  });

```

Now run above demo file. This can be done in Cypress UI.

![Result]({{site.images_dir}}/2020/12/09/cypress_cucumber.png)


Folder structure is as below
![FolderStructure]({{site.images_dir}}/2020/12/09/cypress_cucumber_folderstructure.png)
