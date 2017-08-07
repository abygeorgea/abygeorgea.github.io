---
layout: post
title: "Running Postman collection using Newman"
date: 2017-08-07 15:46:28 +1000
comments: true
categories: [postman, newman]
keywords: postman tutorial, newman tutorial
description: Running Postman Collection Using Newman
---
In previous [blog]({{site.root}}blog/2017/08/05/postman-tutorial/), I explained about how to create a GET request, analyze its response, write test cases for API and to save details to a collection for future use. In this blog, let me explain about how to run collections using Newman.

### What is Newman
Newman is a command line collection runner for postman. Newman also has feature parity with Postman and it runs collection in the same way how it is run through Postman. Newman also makes it easier to integrate API test case execution with other systems like Jenkins.

### Installing Newman
Newman is built on Node.js and hence it requires Node.js to be installed as prerequisite. Newman can be installed from npm with below command

```
$ npm install -g newman
```

### Running collection using Newman
Collections are executed by calling run command in Newman. Basic command for executing collections is

```
newman run PathToCollectionFile -e PathToEnvironmentFileIfAny
```

Below is an example of running collections created in previous blog post using newman.
Command will look like below
`newman run /Users/abygeorgea/Projects/Postman/Postman\ Tutorial.postman_collection.json -e /Users/abygeorgea/Projects/Postman/Test.postman_environment.json`

### Results
The result of API test case execution will look like below. It has a detailed report of number of iterations, number of request, test scripts, pre-requisites, assertions etc. As per standard, passed ones are shown in green and failed in red. The results look similar to details provided if collections are executed using postman.

![NewmanResult]({{site.images_dir}}/2017/08/07/Newman 1.png)

### Additional Options of run command
Newman has various options to customize run. Different options can be found by running with `-h` flag

```
newman run -h
```

Different options listed are below

```
Abys-MacBook-Pro:~ abygeorgea$ newman run -h
usage: newman run [-h] [-v VERSION] [--no-color] [--color]
                  [--timeout-request TIMEOUT_REQUEST] [--ignore-redirects]
                  [-k] [--ssl-client-cert SSL_CLIENT_CERT]
                  [--ssl-client-key SSL_CLIENT_KEY]
                  [--ssl-client-passphrase SSL_CLIENT_PASSPHRASE]
                  [-e ENVIRONMENT] [-g GLOBALS] [--folder FOLDER]
                  [-r REPORTERS] [-n ITERATION_COUNT] [-d ITERATION_DATA]
                  [--export-environment [EXPORT_ENVIRONMENT]]
                  [--export-globals [EXPORT_GLOBALS]]
                  [--export-collection [EXPORT_COLLECTION]]
                  [--delay-request DELAY_REQUEST] [--bail] [-x] [--silent]
                  [--disable-unicode] [--global-var GLOBAL_VAR]
                  collection

The "run" command can be used to run Postman Collections

Positional arguments:
  collection            URL or path to a Postman Collection

Optional arguments:
  -h, --help            Show this help message and exit.
  -v VERSION, --version VERSION
                        Display the newman version
  --no-color            Disable colored output
  --color               Force colored output (for use in CI environments)
  --timeout-request TIMEOUT_REQUEST
                        Specify a timeout for requests (in milliseconds)
  --ignore-redirects    If present, Newman will not follow HTTP Redirects
  -k, --insecure        Disables SSL validations.
  --ssl-client-cert SSL_CLIENT_CERT
                        Specify the path to the Client SSL certificate. 
                        Supports .cert and .pfx files.
  --ssl-client-key SSL_CLIENT_KEY
                        Specify the path to the Client SSL key (not needed 
                        for .pfx files).
  --ssl-client-passphrase SSL_CLIENT_PASSPHRASE
                        Specify the Client SSL passphrase (optional, needed 
                        for passphrase protected keys).
  -e ENVIRONMENT, --environment ENVIRONMENT
                        Specify a URL or Path to a Postman Environment
  -g GLOBALS, --globals GLOBALS
                        Specify a URL or Path to a file containing Postman 
                        Globals
  --folder FOLDER       Run a single folder from a collection
  -r REPORTERS, --reporters REPORTERS
                        Specify the reporters to use for this run.
  -n ITERATION_COUNT, --iteration-count ITERATION_COUNT
                        Define the number of iterations to run.
  -d ITERATION_DATA, --iteration-data ITERATION_DATA
                        Specify a data file to use for iterations (either 
                        json or csv)
  --export-environment [EXPORT_ENVIRONMENT]
                        Exports the environment to a file after completing 
                        the run
  --export-globals [EXPORT_GLOBALS]
                        Specify an output file to dump Globals before exiting
  --export-collection [EXPORT_COLLECTION]
                        Specify an output file to save the executed collection
  --delay-request DELAY_REQUEST
                        Specify the extent of delay between requests 
                        (milliseconds)
  --bail                Specify whether or not to gracefully stop a 
                        collection run on encountering the first error
  -x, --suppress-exit-code
                        Specify whether or not to override the default exit 
                        code for the current run
  --silent              Prevents newman from showing output to CLI
  --disable-unicode     Forces unicode compliant symbols to be replaced by 
                        their plain text equivalents
  --global-var GLOBAL_VAR
                        Allows the specification of global variables via the 
                        command line, in a key=value format
```

