---
layout: post
title: "Postman BDD"
date: 2018-04-28 07:46:43 +1000
comments: true
categories: Postman
keywords: How to write test cases using BDD style in postman
description: How to write test cases using BDD style in postman
---

In Previous blog post,we discussed about [how to use postman]({{site.root}}blog/2017/08/05/postman-tutorial/) and how to use [collections using newman ]({{site.root}}blog/2017/08/07/running-postman-collection-using-newman/)and [data file]({{site.root}}blog/2017/08/13/postman-using-data-file). If you haven't read that , please have a read through first .

In previous examples, we discussed about writing tests/assertions in postman. We followed normal Javascript syntax for writing test cases including asserting various factors of response ( like content , status code etc). Eventhough this is a straightforward way of writing, many people would like to use existing javascript test library like Mocha. They can use [postman - bdd](https://github.com/BigstickCarpet/postman-bdd/#installation) libraries.  

Let us take a deep dive into how to use setup postman bdd. 

Note: It is assumed that user already have postman and newman installed on their machine along with their dependencies. 

##Installing Postman BDD##

Installation is done triggering a Get request and setting the response as Global environment variable. 

*  Create a GET request to `http://bigstickcarpet.com/postman-bdd/dist/postman-bdd.js`
*  Set Global environment variable by using below command in test tab. `postman.setGlobalVariable('postmanBDD', responseBody);`


![PostManRequest]({{site.images_dir}}/2018/04/28/Installing Postman BDD.png)

Once we trigger above get request, postman bdd will be available for use.  We can make use of postman BDD features by below command 
`eval(globals.postmanBDD);`

##Writing Tests##
Postman bdd library provide us with flexibility to write tests and assertions using fluent asserts and have best features of Chai and Mocha. Inorder to demonstrate this, I am using sample Tutorial given with postman client. 

Open up the sample Request in Postman Tutorial folder under collections. It will already have some test predefined in Test tab. Remove them and add below test to it.  

```
eval(globals.postmanBDD)
//eval(postman.getGlobalVariable('postmanBDD'));
var jsonData = JSON.parse(responseBody);
describe('Testing Sample Request in Postman Tutorial', function () {
	it('CASE 1: Should respond with statusCode = 200', function () {
		response.should.have.status(200);
	});
	it('CASE 2: Should response time less than 500 ms', function () {
		pm.response.responseTime.should.be.below(500);
	});
	it('CASE 3: User ID should be 1', function () {
		jsonData.userId === 1;
	});

});

```

Note: You can find more details of various type of asserts in http://www.chaijs.com/api/bdd/


Once it is done, trigger the request 

![PostManRequest]({{site.images_dir}}/2018/04/28/Postman BDD example.png).


