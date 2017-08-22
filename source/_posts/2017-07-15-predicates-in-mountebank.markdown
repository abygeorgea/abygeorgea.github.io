---
layout: post
title: "Predicates In Mountebank"
date: 2017-07-15 21:07:40 +1000
comments: true
categories: Mountebank
keywords: Predicates In Mountebank
description: How to use predicates in Mountebank
---
Predicates in Mountebank imposter files is a pretty powerful way to configure stubs. It helps us to return different responses based on the request parameters like type, query string , headers, body etc. Let us have some quick look at extracting values from request

###  Based on Query String
Below is an example of extracting the records based on query string.
If the request is like `path?customerId=123&customerId=456&email=abc.com`
Note: This is slightly modified version of code in mbtest.org

```
{
  "port": 4547,
  "protocol": "http",
  "stubs": [
    {
      "predicates": [{
        "equals": {
          "query": { "customerId": ["123", "456"] }
        }
      }],
      "responses": [{
        "is": {
          "body": "Customer ID is either 123 or 456"
        }
      }]
    },
    {
      "predicates": [{
        "equals": {
          "query": { 
          	"customerId": "123",
          	"email" :"abc.com"
          	 }
        }
      }],
      "responses": [{
        "is": {
          "body": "Customer ID is 123 and email is abc.com"
        }
      }]
    }
  ]
}
```


### Based on Header Content

If input data is shared through values in header, that can be extracted. Below snippet is directly from mbtest.org

```
{
  "port": 4545,
  "protocol": "http",
  "stubs": [
    {
      "responses": [{ "is": { "statusCode": 400 } }],
      "predicates": [
        {
          "equals": {
            "method": "POST",
            "path": "/test",
            "query": {
              "first": "1",
              "second": "2"
            },
            "headers": {
              "Accept": "text/plain"
            }
          }
        },
        {
          "equals": { "body": "hello, world" },
          "caseSensitive": true,
          "except": "!$"
        }
      ]
    }
   ]
 }
```



