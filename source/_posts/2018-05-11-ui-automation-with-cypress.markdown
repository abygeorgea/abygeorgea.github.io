---
layout: post
title: "UI Automation with cypress"
date: 2018-05-11 07:31:39 +1000
comments: true
categories: Cypress
keywords: UI Automation using cypress
description: How to install and configure cypress
---

When we talk about UI automation for browsers, the default tool which comes to mind is Selenium. There are different wrappers around selenium like protractor, Nightwatch , selenium webdriver etc. All of them are build on top of selenium and have all advantages /disadvantages of selenium. All of the control browser by executing remote commands through Network. We will most probably need additional libraries, framework etc to make full use of selenium.

Cypress.io is an open source UI automation tool which can be used for UI testing . Unlike others, this is not build on top of selenium . Instead is a complete new architecture and run in same run loop as browser. So it is running inside browser and have access to almost everything happening inside and outside browser. It is a complete set of tools that you will require to create and run E2E UI automation test cases. Team who developed cypress has made few design trade off which causes some disadvantages to cypress. There is no right tool for automation . It will depend on multiple factors.


## Installing Cypress ##

We can install cypress using npm. Run below command inside project folder to install cypress and all dependencies.

```
npm install cypress --save-dev
```

Another way of using cypress is to download zip file from [here](https://www.cypress.io/features/) . Just extract the file and start using it.

## Opening Cypress ##

Cypress can be opened by running `node_modules/.bin/cypress open` command in terminal under {{project_location}}/cypress.

If you have downloaded the zip file, you can open cypress by double clicking on the cypress executable. 

## Write your first test ##

Cypress already come with predefined example of KitchenSink application which will help you to identify various commands which can be used. It can be found under `cypress\integration\example_spec.js`.


Let us look at how to write a new test .

Create a new test script file called `demotest.js` under `{project_location}\cypress\integration`. Open up the file and write below code into it.

This code will open browser, load google and search for cypress.io and open up the first link.

```
describe('My first test for cypress', function() {
    it('Visits google home page ', function() {
      cy.visit('https://google.com');
    })
    it('should load the Google Homepage', () => {
        cy.title().should('eql', 'Google');
    })
    it('should search and open cypress home page', () => {
        cy.get('#lst-ib').type('cypress.io');
        cy.get('[value="I\'m Feeling Lucky"]').focus().click();
    })
   
   
  })
```


Note: If cross origin policy error is shown, flow the workarounds mentioned.



## How does cypress.io compare with Selenium ##

As mentioned earlier, there is no right or wrong tool for automation. It all depends on suitability for the task on hand. Let us compare few features where cypress.io and selenium have differences.

* Cross browser support - At this point selenium have more cross browser support that cypress. Cypress supports only chrome variants. You can read about them [here](https://github.com/cypress-io/cypress/issues/310)
* Debugging capability - This is high in cypress. I found that error message are more details and infact provide some more details about how to fix it. Also you have full access to chrome dev tools.
* Keypress - As of now , cypress doesnt support pressing Tab key . You can read about it [here](https://github.com/cypress-io/cypress/issues/299).
* Since cypress is build on node.js, we can chain commands together
* Cypress.io have built in support for test framework and assertion libraries like mocha, chai etc
* Cypress.io test cases can be written in Javascript
* Cypress.io handles wait times better than selenium
* Cypress.io have hot reloading of test cases. When we make changes to test cases and save it , the test rerun by itself. This is very effective to reduce time spend on building and rerunning selenium based test cases.
* Cypress.io have built in time travel and screenshots which will help us to go back to failure points and debug. It also capture before and after state for all actions

I will keep adding to this when I play more with cypress.

