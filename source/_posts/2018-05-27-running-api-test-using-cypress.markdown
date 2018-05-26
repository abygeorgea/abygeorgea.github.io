---
layout: post
title: "Running API Test using Cypress"
date: 2018-05-27 08:03:28 +1000
comments: true
categories: [Postman, API Testing, Cypress]
keywords: Postman, API Testing, Cypress]
description: How to run API test using Cypress
---

Cypress is not just UI automation tool . It can be used for testing APIs as well . Even though we have other tools like Postman, Newman, Rest Assured, SOAP UI etc for testing APIs, I believe cypress is a good alternative for testing API. It will help to use same tool for both UI and API test automation. 

## Demo ##
Let us look at a sample API test case. In below example, we trigger a API call to `http://services.groupkt.com/country/get/iso2code/AU` and validate below in the response. 

* Status code of response is 200.
* Header include 'application/json'.
* Body contain "Country found matching code [AU]."

We can then extend this to do any further checks if needed.

Create a new file inside `Integration` folder of cypress and copy below code into that.

```
describe('API Testing with Cypress', () => {
    var result
    
    it('Validate the header', () => {
       result = cy.request('http://services.groupkt.com/country/get/iso2code/AU')
      
       result.its('headers')
             .its('content-type')
             .should('include', 'application/json')
        
    })

    it('Validate the status', () => {
        result = cy.request('http://services.groupkt.com/country/get/iso2code/AU')
       
        result.its('status')
              .should('equal',200);
     })

     it('Validate the body ', () => {
        result = cy.request('http://services.groupkt.com/country/get/iso2code/AU')
       
        result.its('body')
              .its('RestResponse.messages')
              .should('include', 'Country found matching code [AU].');
      
     })
}) 

```
Open Cypress by running `node_modules/.bin/cypress open` inside cypress root folder. This will open up Cypress.  

Run newly created test.

![APITestingWithCypress]({{site.images_dir}}/2018/05/27/01 Run Test Case.png)

Results of test execution will look like below.

![[APITestingWithCypress]]({{site.images_dir}}/2018/05/27/02 Check Result.png)

Expand each of them and right click on the asserts and inspect the element. This will open up chrome developer tool. Select the console tab , which will list down details of calls made, request received and assertions performed.  It will help to write additional assertions, investigate any failure etc. 

![[APITestingWithCypress]]({{site.images_dir}}/2018/05/27/03 Expand and Analyse result.png)


