---
layout: post
title: "Extracting Metrics from TeamCity"
date: 2018-03-01 22:18:25 +1100
comments: true
categories: [Team City, Metrics]
keywords: Team City, Metrics
description: Ways of extracting metrics from Teamcity
---

TeamCity is a java based build management and continuous Integration server from JetBrains. Very often , we will have to extract various metrics from TeamCity for tracking and trend analysis.  TeamCity provides versatile api for extracting various metrics which can then be manipulated or interpreted as we need. 

Below are basic api calls which can be used for extracting mertics. Please note that TeamCity api is powerful enough to do much more than extraction of data. However, for this blog post, I am focussing on metrics extraction part alone. All of these are GET request to TeamCity api with a valid user credentials ( use any id/password which can access TeamCity) 

1. Get List of Projects - http://teamcityURL:9999/app/rest/projects
2. Get details of a project - http://teamcityURL:9999/app/rest/projects/(projectlocator)
 Project locator  can be either "id:projectID" or "name:projectName"
3. Get List of Build configuration - http://teamcityURL:9999/app/rest/buildTypes 
4. Get List of Build configuration for a project- http://teamcityURL:9999/app/rest/projects/(projectLocator)buildTypes 
5. Get List of Build  - http://teamcityURL:9999/app/rest/builds/?locator=(buildLocator)
6. Get details of a specific Build  - http://teamcityURL:9999/app/rest/builds/(buildLocator)
 Build locator can be "id:BuildId" or "number:buildNumber" Or a combination of these like "id:BuildId,number:buildNumber,dimension3:dimensionvalue". We can use various different values for these dimension. Details can be found in TeamCity documentation
7. Get List of tests in a build - http://teamcityURL:9999/app/rest/testOccurrences?locator=build:(buildLocator)
8. Get individual test history - http://teamcityURL:9999/app/rest/testOccurrences?locator=test:(testLocator)


Recently I created a Nodejs program to extract below metrics by chaining some of the above api calls.

* Number of builds between any two given dates and their status
* Details of number of test cases and their status , pass percentage, fail percentage etc for each build
* Details as above for entire period.
* Trend of test progress, build failures etc between those dates
* Create an output JSON with cumulative counts of passed/failed/ignored builds, passed/failed/ignored test cases , percentage of sucessful builds, frequency of pull request and their success rates etc. 

