---
layout: post
title: "How to work with XML in groovy Scripts"
slug: "readyapi-how-to-work-with-json-in-groovy-scripts"
date: 2022-02-24T06:42:20+10:00
comments: true
categories:
  - ReadyAPI
keywords:
  - ReadyAPI
  - SoapUI
  - JSON
  - XML
  - Groovy Scripting
description: How to parse XML in ReadyAPI groovy scripts
---

ReadyAPI can be used for testing both SOAP and REST services. Output format of them is mainly XML / JSON. Hence it is important to know how to parse them into corresponding objects.

### Parsing XML

Consider a scenario where output for a SOAP service or JDBC call is returning a XML containing list of person information, which we need to convert to objects. 

XML format is like below, which available as a response content of a ReadyAPI Step

``` XML
<Results>
   <ResultSet fetchSize="10">
      <Row rowNumber="1">
         <FIRSTNAME>Tom</FIRSTNAME>
         <LASTNAME>CITIZEN</LASTNAME>
         <AGE>20</AGE>
      </Row>
      <Row rowNumber="2">
         <FIRSTNAME>Jerry</FIRSTNAME>
         <LASTNAME>CITIZEN</LASTNAME>
         <AGE>15</AGE>
      </Row>
</ResultSet>
</Results>


```

Steps to parse them is as below.

* Create a person object
* User XMLSlurper to parse the response content to XML document
* Iterate over the rows and create an object and add to a list



``` groovy
//Create an object
class Person
{
String FirstName
String LastName
Integer Age
}

//Parse response content string to XML object
def Results = new XmlSlurper().parseText(messageExchange.responseContent)
log.info "Number of records :" + Results.ResultSet.Row.size();

// Iterate over each row and convert to obhect
def  DBRecordList = new ArrayList<Person>();
for(record in Results.ResultSet.Row ){
def obj = new Person();
obj.FirstName  = "${record.FIRSTNAME}" ;
obj.LastName = "${record.LastName}" ;
obj.Age =  ${record.Age};
DBRecordList.add(obj);
}
```




