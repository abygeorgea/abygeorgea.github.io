---
layout: post
title: "How to read browser cookies in Cypress"
slug: "how-to-read-browser-cookies-in-cypress"
date: 2020-11-19T19:42:58+11:00
comments: true
categories:
  - Cypress
keywords: Cypress, Cookies
description: "How to read browser cookies in Cypress"
---
Cypress has inbuild support to read browser cookies. There are two commands for this - GetCookie and GetCookies. Refer [documentation](https://docs.cypress.io/api/commands/getcookie.html#Session-id) for more details

###GetCookie

This command get a cookie by its name. 

```
cy.getCookie(name)

cy.getCookie(name, options)

Note: name is the name of cookie . Options can be used to change default behaviour like logging , timeout

```

Examples usage is as below

``` javascript

//Below line is added to get intellisense while writing code in visual studio code

/// <reference types="cypress" />


it('Read Cookies In Cypress',() =>  {
    
    cy.visit("www.commbank.com.au");

    //GetCookie returns an object with properties like name, domain, httpOnly, path, secure, value, expiry ( if provided), sameSite(if provided)
    
    //Checking for individual cookie property value
    cy.getCookie('s_cc').should('have.property','value','true');
    cy.getCookie('s_cc').should('have.property','domain','.commbank.com.au');


   // Checking multiple properties of a cookie. *cy.getCookie* will get an object. *Then* helps to work with object yielded from previous
    cy.getCookie('s_cc').then((cookie) => {

        cy.log(cookie);
        cy.log(cookie.name);
        expect(cookie.domain).to.equal('.commbank.com.au');
        expect(cookie.name).to.equal('s_cc');
        expect(cookie.httpOnly).to.equal(false);
        expect(cookie.path).to.equal('/');

        expect(cookie).to.not.have.property('expiry');
    })

   
})

```


Results from test run will look like below

![result](/images/2020/11/19/Cypress_GetCookie.png)
