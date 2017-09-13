---
layout: post
title: "Postman - Using Data File"
date: 2017-08-13 20:56:01 +1000
comments: true
categories: [postman, newman]
keywords: postman tutorial, newman tutorial
description: Using data files in postman collection runner and Newman
---

One of the common requirement for automated testing is to run same test case against multiple test data. Luckily postman supports this by providing facility to use data files. This is available only when we run through postman collection runner or newman. 


For this example, let us take a free public API `http://services.groupkt.com/country/get/iso2code/AU` . This API will return the name of the country depending on the 2 digit code passed. Let us assume that, we need to test this API with multiple country codes. For eg: AU, IN, GB etc.  Let us take a look to see how this can be achieved using postman data files. 

#### Environment file ####
First, create an enviornment Manage Environment option at top right. Create an entry for endpoint as below.

![EnivironmentSetup]({{site.images_dir}}/2017/08/13/PostmanDatafile 1.png)

#### Create Collection ####
Next step is to create a collection with a GET request and write tests to verify the response. GET request used here is `{EndPoint}/country/get/iso2code/{countrycode}`

Endpoint is defined in environment file and countrycode will be in data file

Now write some tests to check the results. The data coming from data file will be available under "data" dictionary  ( similar to global/environment variable. It can be accessed as `data.VARIABLENAME` or ` data["VARIABLENAME"]` in both test and pre requisite scripts. Below screenshot shows the test which is for validating country name based on the data file.

![EnivironmentSetup]({{site.images_dir}}/2017/08/13/PostmanDatafile 2.png)

#### DateFile####
Postman supports both CSV and JSON format. For CSV files, the first row should be the variable names as the header. All subsequent rows are data row. JSON file should be an array of the keyvalue pair where the variable name is the key. 

Data file used in this example is below. It has 3 column, where the first column is test case ID and the second one is country code which is used in the request and the third one is the country name, which is used for asserting the response received. In this example, I am looking for 3 different country codes.

![datafile]({{site.images_dir}}/2017/08/13/PostmanDatafile 3.png)


#### Running Collections####

While running collections, we need to specify below inputs.

* Collection Name
* Environment file
* Data File

Depending on number of records in the data file, iterations will be auto populated. The results will also show the details for each iteration using the data. Details of response can be found by expanding response body

![collection]({{site.images_dir}}/2017/08/13/PostmanDatafile 4.png)

![result]({{site.images_dir}}/2017/08/13/PostmanDatafile 5.png)


#### Running through Newman ####

We can run same collection through Newman as well

```
newman run PathToCollectionsFile -e PathToEnvironmentFiles -d PathToDataFile

```
In this cases, I should run `newman run DataDriven.postman_collection.json -e DataDrivenEnvironment.postman_environment.json   -d data-article.csv`.

Results will be as below

![NewmanResults]({{site.images_dir}}/2017/08/13/PostmanDatafile 6.png)