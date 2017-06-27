---
layout: post
title: "Galen Framework - Getting started"
date: 2017-06-25 05:39:02 +1000
comments: true
categories: Galen
keywords: 
- Responsive web Design
- Galen framework
description: How to automate responsive design testing using Galen
---

In previous [post]({site.root}}blog/2017/06/21/automated-testing-of-responsive-web-design), I mentioned that we can use Galen for automated lay out testing. Galen offers a simple solution to test location of objects relative to each other on the page. Galen is implemented using Selenium Web driver. Hence we can use it for normal functional automation testing as well. 

###  Documentation of Galen ###

- Galen has its own domain specific language to define Specs. Detailed documentation can be found [here](http://galenframework.com/docs/reference-galen-spec-language-guide/).
- Galen has its own javascript API which provides a list of functions which make writing test cases easier. Detailed documentation can be found [here](http://galenframework.com/docs/reference-galen-javascript-api/).
- Galen pages javascript API is light weight javascript test framework. Details are available [here](http://galenframework.com/docs/reference-galenpages-javascript-api/). 
- Details of galen test suite syntax are [here](http://galenframework.com/docs/reference-galen-test-suite-syntax/). 
- Galen framework has a detailed documentation of its usage and functions [here](http://galenframework.com/docs/all/).

###Installation###

Below are high-level steps to help you get started.

1. Ensure Java is installed. Galen needs Java version above 1.8
2. Download binary from http://galenframework.com/download/
2. Extract the zip file
3. Add the location of extracted files to PATH  environment variables. A detailed guide for older versions of Windows is available [here](http://mindengine.net/post/2014-01-08-configuring-galen-framework-for-windows).
4. Alternatively, on Windows , you can create a bat file to run Galen by changing Path on the fly. Details are in below steps.


###Setting up Galen Framework###
There are different framework available for testing responsive design based on Galen. [Galen bootstrap](https://github.com/galenframework/galen-bootstrap) is one of such framework which can be reused. 

- Download and extract the project from Github. Keep relevant files only. You can remove 
- Create an `init.js` file to load 
 `galen-bootstrap/galen-bootstrap.js `script and configure all devices and a website URL for testing. URL mentioned below is an example of responsive web design template.


``` javascript
load("galen-bootstrap/galen-bootstrap.js");
//$galen.settings.website = "https://alistapart.com/d/responsive-web-design/ex/ex-site-FINAL.html";
//$galen.registerDevice("mobile", inLocalBrowser("mobile emulation", "450x800", ["mobile"]));
//$galen.registerDevice("tablet", inLocalBrowser("tablet emulation", "600x800", ["tablet"]));
//$galen.registerDevice("desktop", inLocalBrowser("desktop emulation", "1024x768", ["desktop"]));
```
Note: Uncomment the lines above. Octopress blog engine was throwing error when it tries generate post.




- Run `galen config` from the command line with the project directory. This will create Galen config file in the location where the command is run.

![Create Galen Config]({{site.images_dir}}/2017/06/25/GalenframeworkGettingStarted01.png)




- Modify galen.config file to make chrome as default browser and add path to chrome driver. There are other useful configs like range approximation, screenshot, selenium grid etc in the config.

```
galen.default.browser=chrome
$.webdriver.chrome.driver=.\\..\\WebProject\\Driver\\chromedriver.exe
```




- Create a folder named `Test` for keeping test cases and create test files `example.test.js`. Copy below content to `example.test.js`. Make sure to update the relative location of the init.js file created in previous steps. Below content loads init.js file which lists out website URL, device sizes that need to be tested.It then calls a function to test on all devices. Check layout is one of the available javascript API function.


```
load (".\\..\\init.js")
testOnAllDevices("Welcome page test", "/", function (driver, device) {
    checkLayout(driver, "specs/homepage.gspec", device.tags, device.excludedTags);
});

```


- Create a folder named `specs` and create a spec file named `homepage.gspec`. We need to update the specs with layout checks . Below is the sample spec for checking image and section intro for the sample URL from init.js. First section defines the objects and its identifier. Second section says that on desktop, image will on left side of section intro and on mobile and tablet, it will be above section intro

```
@objects
	image        id         logo
	menu         css        #page > div > div.mast > ul
	sectionintro    css     #page > div > div.section.intro

= Main  Section =
	image:
		@on desktop
			left-of sectionintro
		@on mobile, tablet
			above sectionintro
```



- Now create a bat file in the main folder to run the galen test cases. Make sure to give relative paths to test file, configs, reports correctly. Modify Path variable to include path location to galen bin. This is not needed if we manually set pah while installing. However, I prefer to have galen bin files as well in source control and point the path to that location so that we don't have any specific dependency outside the project.

```
SET PATH=%PATH%;.\galen-bin
galen test .\\test\\example.test.js  --htmlreport .\reports   --jsonreport .\jsonreports --config .\galen.config
```


once all files are created, folder structure will look like below
![]({{site.images_dir}}/2017/06/25/GalenframeworkGettingStarted00.png)


- run the bat file created above. This will ideally run example.test.js file which invokes chrome driver, navigate to the URl, resizes the browser and then check for the specs.It will list out the results in command prompt. Once it completes are all test execution, it creates both HTML report and JSON report in corresponding folder location mentioned in bat file. Below is a sample HTML report, which is self-explanatory.


Main report
![]({{site.images_dir}}/2017/06/25/GalenframeworkGettingStarted02.png)

If we expand the result for desktop emulation, it will look like below.It will list down each assertion made and indicate whether it is passed or failed. 
![]({{site.images_dir}}/2017/06/25/GalenframeworkGettingStarted03.png)

If we click on the assertion point, it will show the screenshot taken for that assertion by highlighting the objects which will help for easier verification. Below screenshot shows that image is on left side of section intro as defined in spec file.
![]({{site.images_dir}}/2017/06/25/GalenframeworkGettingStarted04.png)