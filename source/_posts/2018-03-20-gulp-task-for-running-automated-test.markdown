---
layout: post
title: "Gulp Task For Running Automated Test"
date: 2018-03-20 11:46:30 +1100
comments: true
categories: Gulp
keywords: Gulp
description: Gulp task for running Automated Test
---

[Gulp](https://gulpjs.com/) is a toolkit for automating painful or time-consuming task in your development workflow, so you can stop messing around and build something. Gulp can be used for creating a simple task to run automated test cases.

Firstly, we will create package.json file for this project. This can be done by below command from project folder. It will prompt you to enter a list of information required for creating package.json file

	npm init
	
Once this is done, install gulp.  It can be done by below command. This will add gulp as a dev dependency.

	npm install --save-dev gulp-install
	
In order to run acceptance test cases, we will need to install nunit/xunit test runners. It can be done by below command from the root folder. 

	npm install --save-dev gulp-nunit-runner
			OR
	npm install --save-dev gulp-xunit-runner
	
Detailed usage of above test runners are available [here](https://www.npmjs.com/package/gulp-nunit-runner).

Once above are installed, we need to create `gulpfile.js` inside root folder. This file will have details of various gulp tasks


Sample Usage of test runner is below. Insert this code into gulpfile.js

```

var gulp = require('gulp'),
    nunit = require('gulp-nunit-runner');
 
gulp.task('unit-test', function () {
    return gulp.src(['**/*.Test.dll'], {read: false})
        .pipe(nunit({
            executable: 'C:/nunit/bin/nunit-console.exe',
            options : {
            	where : 'cat == test'
            }
        }));
});

```
* {read: false} means, it will read only file names and not the entire file. 
* Executable is the path to nunit console runner, which should be available.
* gulp.src is that path to acceptance test solution dll. Since we use wild character, we may have to modify this path to reflect the exact path of dll.( something like ./**/Debug/Project.acceptancetest.dll) 

Once we have above in gulpfile.js, it can be run by below command

	gulp unit-test
	
Out of above command will be something like

	C:/nunit/bin/nunit-console.exe "C:\full\path\to\Database.Test.dll" "C:\full\path\to\Services.Test.dll"


Note: If it complains about assembly missing, it means path to acceptance test solution is incorrect . Retry after fixing the path.

Gulp Nunit runner provide lot options to configure test run, like selecting test cases based on category, creating output files etc. Detailed options can be found [here](https://github.com/keithmorris/gulp-nunit-runner).

Below is an example with few options


```

var gulp = require('gulp'),
    nunit = require('gulp-nunit-runner');
 
gulp.task('unit-test', function () {
    return gulp.src(['**/*.Test.dll'], {read: false})
        .pipe(nunit({
            executable: 'C:/nunit/bin/nunit-console.exe',
            options : {
            	where : 'cat == test',
            	work : 'TestResultsFolder',
            	result : 'TestResults.xml',
            	config : 'Debug'
            }
        }));
});

```
* Where - Selects the category which needs to be run
* Work - Create a folder with specified path/name for output files
* result - create test results in xml 
* config - select the config which needs to be run

If we run ` gulp unit-test` now, it will execute only the test cases having category `test`. It will create a folder named `TestResultsFolder` and will have an xml report of the test run inside it .  The folder will be created in root where we have gulpfile.js. 