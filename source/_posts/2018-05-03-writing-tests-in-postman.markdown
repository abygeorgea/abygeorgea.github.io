---
layout: post
title: "Writing Tests in Postman"
date: 2018-05-03 21:05:00 +1000
comments: true
categories: [postman, newman]
keywords: How to use Chai for writing postman tests
description: How to use Chai for writing postman tests
---

In previous blog [post]({{site.root}}blog/2018/04/28/postman-bdd/), we saw how to use BDD format for writing test cases in postman. Most important part of writing tests in postman is understanding various features available. Let us explore various options available . The examples specified in postman [documentation](https://documenter.getpostman.com/view/220187/postman-bdd-examples/6Z3uY71#30dfc9d2-5de4-b932-db3e-641c29fb0459), have lot of information about how to setup postman bdd, use chai http assertions, create custom assertions and use before and after hooks. Please import them into postman and try that by yourself to familiarise with postman BDD. Below is only few examples from them.

Postman BDD makes use of Chai Assertion Library and Chai-Http. We have access to both libray and postman scripting environment for writing test cases. Chai has two types of assertion styles.

* `Expect/should` for BDD
* `Assert` for TDD

Both styles support chainable language to construct assertions. We can use both of them to write postman test assertions. If you need details of all chainable constructs, please refer to their [documentation](http://www.chaijs.com/api/bdd/). Major ones which we may use in postman tests are

* Chains

		to
		be
		been
		is
		that
		which
		and
		has
		have
		with
		at
		of
		same
		but
		does

* Not - Negates all conditions 
* any - 
* all - 
* inlcude
* OK
* true
* false
* null
* undefined
* exist
* empty
* match(re[, msg])


Chai-Http module provide various assertions. Read through their documentation [here](https://github.com/chaijs/chai-http#assertions) to know details. Below are main commands at our disposal for validation .  

*  .status(code)
* 	.header (key[, value])
*  .headers
*  .ip
*  .json / .text / .html
*  .redirect
*  .param
*  .cookie



Postman bdd provide `response` object on which we do most of assertions. It will have all information like response.text, response.body, response.status, response.ok , response.error. Postman BDD will automatically parse JSON and XML responses and hence there is no need to call JSON.parse() or xml2json(). response.text will have unparsed content. It also have automatic error handling , which will allow to continue with other test even if something fails.

Examples for various assertions done on response object are below

```
\\Verifying Header information
expect(response).to.have.status(500);
expect(response).to.have.header('x-api-key');
expect(response).to.have.header('content-type', 'text/plain');
expect(request).to.have.header('content-type', /^text/);
expect(response).to.have.headers;
expect('127.0.0.1').to.be.an.ip;

\\Verifying Response body
expect(response).to.be.json;
expect(response).to.be.html;
expect(response).to.be.text;

response.should.have.status(200); 
response.body.should.not.be.empty;
response.ok.should.be.true;            // sucess with code 2XX
response.error.should.be.true; //failures

\\Verifying request
expect(req).to.have.param('orderby', 'date');
expect(req).to.not.have.param('orderby');
expect(req).to.have.cookie('session_id', '1234');
expect(req).to.not.have.cookie('PHPSESSID');



```


If we use above assertions in proper BDD format, it will look like below

```
eval(globals.postmanBDD);
describe('Example for Blog using SHOULD', function(){
   it("Tests using SHOULD", function() {
      response.should.have.status(200); 
      response.should.not.be.empty;
      response.should.have.header('content-type', 'application/json; charset=utf-8');
      response.type.should.equal('application/json');
      
      var user = response.body.results[0];
      user.name.should.be.an('object');
      user.name.should.have.property('first').and.not.empty;
      //user.name.should.have.property('first','david');
      user.should.have.property('gender','male');
   
   
   }) 
})
describe('Example for Blog using Expect', function(){
   it("Tests using EXPECT", function() {
      expect(response).to.have.status(200);
      expect(response).not.empty;
      expect(response).to.be.json;
      expect(response).to.have.header('content-type', 'application/json; charset=utf-8');
   }) 
})

it('should contain the un-parsed JSON text', () => {
    response.text.should.be.a('string').with.length.above(50);
    response.text.should.contain('"results":[');
});

```

![Postman]({{site.images_dir}}/2018/05/03/postman bdd.png)
