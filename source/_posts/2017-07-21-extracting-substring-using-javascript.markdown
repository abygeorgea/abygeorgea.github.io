---
layout: post
title: "Extracting Substring using Javascript"
date: 2017-07-21 06:09:53 +1000
comments: true
categories: [codesnippets, javascript, mountebank]
keywords: Extracting substring using javascript, Extracting XML node using javascript
description: 
---
In previous blogs [here]({{site.root}}blog/2017/04/27/stubbing-xml-responses-using-mountebank/) , I have explained how we return a XML response using mountebank. However , most of the time, we will have to make some modification to the template response before returning a response. Say for example, we may have to replace details like timestamp, or use an input from request parameter and update that in response etc. 

One of the easiest way to do this without using other frameworks like xml2js etc is to extract the substring between the node values and replace it . Below is a code snippet which will help to achieve this

The sample xml which we need to return is 

```
<Status>Added</Status>
<GeneratedID>12345</GeneratedID>
```

In above example, assume that we need to replace the inserted record value every time based on the request coming through . We can do that by below

``` javascript

var xmldata = "<Status>Added</Status>\r\n<GeneratedID>12345</GeneratedID>"

var generatedId = xmldata.match(new RegExp("<GeneratedID>"+"(.*)"+"</GeneratedID>"));
console.log(generatedId);
// Output will be as below. from Array we can extract the substring, index of its location etc
/*
[ '<GeneratedID>12345</GeneratedID>',
  '12345',
  index: 24,
  input: '<Status>Added</Status>\r\n<GeneratedID>12345</GeneratedID>' ]
  */

//so extract data from first location to get substring  
generatedId = xmldata.match(new RegExp("<GeneratedID>"+"(.*)"+"</GeneratedID>"))[1];
console.log(generatedId);
//Above will print "12345" , which is the expected value
// This can be used for extracting value of xml nodes

//if we need to replace this with another value ( possibly coming from request parameter)
var result = xmldata.replace(generatedId, "99999");
console.log(result);


```


