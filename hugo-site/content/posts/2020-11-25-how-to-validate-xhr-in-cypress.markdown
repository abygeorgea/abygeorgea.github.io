---
layout: post
title: "How to validate XHR  in Cypress"
slug: "how-to-validate-xhr-in-cypress"
date: 2020-11-25T14:19:34+11:00
comments: true
categories:
  - Cypress
  - XHR
keywords:
  - Cypress
  - XHR
description: How to validate XHR  in Cypress
---


In the previous [post](/blog/2020/11/19/how-to-read-browser-cookies-in-cypress/) we saw about working with Cookies in Cypress. Now let us look at how we can work with XHR request in Cypress.

XHR stands for XMLHttpRequest. It is an API in the form of an object whose methods transfer data between a web browser and a web server. Cypress provides inbuilt functionality to work with XHR request. It provides us with objects with information about request and response of these calls. It will help to do various assertions on header, url , body , status code etc as needed. It also help us to stub the response if needed.

In Cypress 5 , the XHR testing was done mainly using cy.server() and cy.route(). However they are deprecated in Cypress 6.0.0. In version 6, XHR testing can be done using cy.intercept(), which will help to manipulate behaviour of HTTP request made. 

Usage format as defined in cypress [documentation](https://docs.cypress.io/api/commands/intercept.html#Comparison-to-cy-route) is as below

```
cy.intercept(url, routeHandler?)
cy.intercept(method, url, routeHandler?)
cy.intercept(routeMatcher, routeHandler?)
```

As you can see in above, the last parameter is routeHandler , which defines what should happen if cypress is able to intercept a call matching initial parameters. We can specify the criteria as either a URL (either string or regular expression) ,  method (string) & url (string or regular exp) or various combinations using routeMatcher. RouteMatcher has a list of properties based on which we identify the network calls. Properties are like path, url, auth, headers etc. Cypress documentation has more details on this.

Now let us look at a practical example of asserting XHR. In below example, 

``` javascript
//below line is added to get intellisense in visual studio IDE
/// <reference types="cypress" />

it('Working with XHR In Cypress',() =>  {
    
    //I am using only one parameter for intercept , which is an object with property pathname. This is the third way above ( routeMatcher). Only possible chaining for intercept is an alias. Hence I assign an alias as 'weather' for the intercepted call
    
    cy.intercept({
        pathname: '/scripts/marketing.json'
    }).as('weather');
    
    //visit the webpage
    cy.visit("http://www.bom.gov.au/nsw/forecasts/sydney.shtml");
    
    //wait for the interception and once network calls are made, then proceed with remaining
    
    cy.wait('@weather').then((interception) => {
        // 'interception' is an object with properties 'id', 'request' and 'response'
        cy.log(interception.id);
        cy.log(interception.state);
        cy.log('Status code is ' + interception.response.statusCode);
        cy.log('response body is '+interception.response.body);

        expect(interception.response.statusCode).to.eq(200);
      })
})
```

Below is the results from running above test. As you can see the response body is an object.

![result](/images/2020/11/25/result1.png)


## Stubbing an XHR request ##
Now let us see how we can stub the response call. Add another parameter to intercept method , which matches the routeHandler.

``` javascript

//below line is added to get intellisense in visual studio IDE
/// <reference types="cypress" />

it('Working with XHR In Cypress',() =>  {
    
    //Add second parameter to stub the response 
    cy.intercept({
        pathname: '/scripts/marketing.json'
    },'{ body: "Response body is stubbed" }').as('weather');
    cy.visit("http://www.bom.gov.au/nsw/forecasts/sydney.shtml");
    cy.wait('@weather').then((interception) => {
        // 'interception' is an object with properties 'id', 'request' and 'response'
        cy.log(interception.id);
        cy.log(interception.state);
        cy.log('Status code is ' + interception.response.statusCode);
        cy.log('response body is '+interception.response.body);

        expect(interception.response.statusCode).to.eq(200);
      })

})

```

Below is the results from running above test. As you can see the respone body is now having stubbed value passed above

![result](/images/2020/11/25/stubbed_result.png)
