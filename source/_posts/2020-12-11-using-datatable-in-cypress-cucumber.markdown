---
layout: post
title: "Using Datatable in Cypress Cucumber"
date: 2020-12-11 14:03:27 +1100
comments: true
categories: [Cypress, Cucumber]
keywords: [Cypress, Cucumber, Datatable]
description: How to use Datatable in Cypress using Cucumber
---

In the previous post [here]({{site.root}}blog/2020/12/09/writing-cypress-tests-in-bdd-format), we saw how to use cucumber along with Cypress. The demo scenario we saw was a basic example. Now let us look into how we can use datartable in cypress cucumber. 

##Feature File##

``` Gherkin
Feature: Demo for BDD in Cypress

  I want to demo using BDD in Cypress
  
  @focus
  Scenario: Logging into ECommerce Site
    Given I open Demo shopping page
    When I login as 'standard_user' user
    Then I should see products listed
    Given I add below products to cart
    |ProductName                |Qty|
    |Sauce Labs Backpack        |1  |
    |Sauce Labs Fleece Jacket   |1  |
    |Sauce Labs Onesie          |1  |
    
```

The step definition file will be as below

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

//Use datatable to click on each element
//Identify the elements based on the product name and click corresponding add to cart button

  Given('I add below products to cart', (dataTable) => {
  
    cy.log('raw : ' + dataTable.raw());
    cy.log('rows : ' + dataTable.rows());
    cy.log('HASHES : ' );
    var propValue;
    dataTable.hashes().forEach(elem =>{
      for(var propName in elem) {
        propValue = elem[propName]

        cy.log(propName,propValue);
    }
    });
  
    dataTable.hashes().forEach(elem => {
      cy.log("Adding "+elem.ProductName);
      cy.get('.inventory_item_name').contains(elem.ProductName).parent().parent().next().find('.btn_primary').click();
      });
    
    
  });
  
  
```

datatable can be used in few different ways . Documentation of cucumberjs is [here](https://github.com/cucumber/cucumber-js/blob/master/docs/support_files/data_table_interface.md). I have used *datatable.Hashes* which return an array of hashes where column name is the key. Apart from datatable.Hashes, there are other methods like row( return a 2D array without first row) , raw( return table as 2D array) , rowsHash(where first column is key and second column is value) etc.


Running this in Cypress will result in below. we can see the values logged by cy.log

![Result]({{site.images_dir}}/2020/12/11/datatable.png)
![Result]({{site.images_dir}}/2020/12/11/DOM.png)



 