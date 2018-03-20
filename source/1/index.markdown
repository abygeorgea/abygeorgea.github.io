---
layout: page
title: ""
date: 2017-07-23 10:15
comments: false
sharing: false
footer: true
---
#Interview Questions
1. Typical day in office - Time divided between various responsibility in a day
	* 	20% - Status Update /scrum meeting, client communication, meetings etc
	*  50% - Test activities - varies differently depending on test cycle. test plan, test strategy, prioritisation of defects & test cases, identifying the planned test cases for the day, defect retest, test execution 
	*  20% - review of work done by team mates
	*  10% - coordination between different teams, monitoring the progress
	*  
2. Normal test process in your current company
	* Agile methodology 
	* Ownership of stories on complete team . what that means is , while user stories are in progress, we don't raise bugs. Instead we move it back to dev and get if fixed. Defects are raised only if it needs to be carried over and will take time to fix . But most of the expected acceptance criteria is done
	* previous projects had onshore offshore models
	* all review , metrics etc are done within sprint
	* 
3. Key achievements
	* Successfully stubbed out all downstream dependency . not imapcted by recent S4 outage. mentioned in CIO email
	* leading lights nomination
	* Digital Kudos nomination
	* mentioned in segment scrum
	* stream lined reporting process
	* created new report format for status update
	* succesfuly managed end to end testing of multi million testing engagement
	* Improved automation ratio of the projects
	* managed to remove downstream dependency while testing
	*
4. How to do test estimation
	* Discuss with test team and dev team/architect to identify the solution
	* Identify scope of testing for which test estimation is needed. There are chances that other test phases like System testing or end to end testing , performance testing etc will be done by different team.
	* Discuss with test team to come up 
5. What to do when there is not enough time for testing
6. What to do when a critical bug is found one day prior to release
7. What will you do when you find defects
  	* Analyze logs to identify root cause
  	* Use debugger in Visual studio to identify what is causing issue

##Test Process##

##Test Management##

##Selenium##
- explain page objectg model
 POM is the most widely used design pattern by the Selenium community on which each web page (or significant ones) considered as a different class. On each of this page classes (i.e page objects), you may define the elements of that page and specific methods for that page. Each page object represents the page of the web page or application. It is a layer between the test scripts and UI and encapsulates the features of the page. problems with page objects are they dont follow SOLID principles . Single responsibility principle and Open to extension and closed to modification principle 
 - explain page factory
 Page Factory
Page Factory is an extension to page objects that is used to initialize the web elements that are defined on the page object. You can define the web elements by using annotation with the help of page factory.

In constructor , we call Pagefactory.init elements

- what all are element locators and preference to use them
1. ID Locator
2. NAME Locator
3. CSS Locator
4. XPATH Locato

- how to synchronise AUT and selenium- implcit and explicit wait
Implicit Wait - It instructs the web driver to wait for some time by poll the DOM. Once you declared implicit wait it will be available for the entire life of web driver instance. By default the value will be 0. If you set a longer default, then the behavior will poll the DOM on a periodic basis depending on the browser/driver implementation.

driver.manage().timeouts().implicitlyWait(TimeOut, TimeUnit.SECONDS);	


Explicit Wait + ExpectedConditions - It is the custom one. It will be used if we want the execution to wait for some time until some condition achieved.
Explicit waits are intelligent waits that are confined to a particular web element. Using explicit waits you are basically telling WebDriver at the max it is to wait for X units of time before it gives up

WebDriverWait wait = new WebDriverWait(WebDriverRefrence,TimeOut);
wait.until(ExpectedConditions.visibilityOfElementLocated(By.xpath( "/html/body/")));


Fluent Wait

The fluent wait is used to tell the web driver to wait for a condition, as well as the frequency with which we want to check the condition before throwing an "ElementNotVisibleException" exception.

Wait wait = new FluentWait(WebDriver reference)
.withTimeout(timeout, SECONDS)
.pollingEvery(timeout, SECONDS)
.ignoring(Exception.class);


- how to make code readable and maintanble
 1. Create base class and do proper inheritence  ( base class to have browser related specs, waits etc) 
 2. Follow Page object model and group together UI elements and their acions
 3. Follow principles of clean code
 4. Follow DRY ( dont repear yourself)
 2. functon name to be readable
 2. proper syntaxing for all variable , 
 3. constant togethr
 4. Define only needed variable and function as public
 5. Hide away implementation details by proper encapsulatiom


consulting
- factors involved in deciding automation
type of testing
business and technical awareness
technology used

- how to estimate test automation effort
- how to calculate ROI
##Automation##

## General ##
What is testing to you? Do you have a testing paradigm? Can you explain it?

 -	A testing paradigm is a mental framework of testing. It covers the way you think about testing, the way you approach deciding your testing methods, the methodologies you choose to use, and even the words you use to describe the things you are doing.
 - 


Why did you choose software testing as a career, and what motivates you to stick with it?

- ended up here for onsire
- love solving challenges and writing code

Tell me about the work you’ve been doing recently. What’s the most interesting bug that you’ve found, and why?

- statements
- bugs were assumption were wrong and we couldnt verify assumption due to data issue
- 

What kind of challenges does testing present? Can you tell me about some specific software testing challenges you’ve faced, and how you overcame them?

## Behavioural and charector traits ##
Have you encountered any challenges working with your colleagues? Tell me about a specific instance when you were in a difficult situation, and how you dealt with it.

How about situations where you have to make decisions… Have you ever made a bad decision? What contributed to you making that decision? How did you deal with the consequences?
- everyone makes bad decision
- talk about incident where TM asked to defer a defect to next release ( account origination) which created trouble since it was a valid defect. Gut feel was that is not right and I should have defiedit.  had to get CIO approval to put a hot fix one day prior to deployment. Had to attend all those high profile meetings with root cause analysis, preventive actions etc

How have you added value to the organisations you’ve worked with? Can you give me a specific example from your last or current position?
- mountebank
- service stubbing

What process are you using for testing currently? Can you describe how you might improve it?

Describe a major goal you have set to yourself.
- Improving my influencing skills and public speaking skills. 
- I have started blog for that
- started training people


what is your weekness??
- workaholic. work is the first wife. explain about incidednts when i went to office even when every one is sick. 
- mention i would like to have more detailed attention even when it can be delegated. Sometimes, I spend more time than necessary on a task or take on tasks personally that could easily be delegated to someone else. Although I've never missed a deadline, it is still an effort for me to know when to move on to the next task, and to be confident when assigning others work.
- - take about examples of reviewing test cases of offshore team and finding bugs in it. Spending extra hours at home for that

Why do you want to work with us??
- reputuation of ASX is definitely one things
- It does have better than average reviews about work life balance, culture and ethics
- I have a friends who works on other stock markets who talk about challenges and oppurutnities they have, which made a interested to work in another stockexchange


What type of decision do you have difficulty in making

What are the your major success ??


Where do you want to be in 5 years

Why should we hire you? What makes you best fit for this role
 - depends on when is this asked. If asked in begining , you can ask for more details of roles
 - From my understanding, this role require someone who needs to be fully autonomous , who can make informed decisions and own the testing area and also have experience in making automation frameworks. From my experience , you can see clearly see that I possess all required  qualities for this role, which makes myself as a formidable option. 
 - 
Tell me about a time you didn’t perform to your capabilities.
How do you manage stress in your daily work?
How do you regroup when things haven’t gone as planned?



## Testing Skills ##
What kind of tests have you been doing? What do you enjoy about them? How do you develop those tests?

When you perform a test, what steps do you take? What’s your process?

Have you ever written a test plan? What would you put in one?

How do you prioritise your testing? What factors might influence your decisions?

How do you know when it’s time to stop testing?

What’s the role of risk in your testing? How do you analyse and measure it?

Do you measure how effective (or not) your testing is? What metrics do you use?

If I left you testing for two hours, what would you have to show me when I returned?


## Automation SKills
Have you automated any of your tests? How so?

What’s your favourite testing tool? Why? If some technical constraint meant you were unable to use it, what would you do instead?


How do you know when you (or your automation) has found a bug? What makes it a bug? Are some bugs more important than others? How do you report them?


What do you do if the developers decide the bug is not a bug?

How do you decide which tests to automate? Which tests don’t you automate, and why?

what is page object model
 - seperate class for each webpage
how to create report

how to send emails
how to take screenshots

java:
- have element locator in property file - read to hash table and use
- define url , login credential etc and use property file
- base class, extend
- 

## Commitement to learning ##
Testing can be challenging. What keeps you motivated?

How do you stay at the top of your game? What self-learning do you do?

What have you been criticized for in the past? How did you respond to that criticism? What did you do about it?

Describe the characteristics of your ideal boss, and why.

## Object Oriented
class - A class is a blueprint or template or set of instructions to build a specific type of object. Every object is built from a class. 

Instance - An instance is a specific object built from a specific class. It is assigned to a reference variable that is used to access all of the instance's properties and methods. 

Abstract class - an abstract class is a generic class (or type of object) used as a basis for creating specific objects that conform to its protocol, or the set of operations it supports. Abstract classes are not instantiated directly. An abstract class has at least one abstract method. An abstract method will not have any code in the base class; the code will be added in its derived classes. The abstract method in the derived class should be implemented with the same access modifier, number and type of argument, and with the same return type as that of the base class. 


inheritence - Inheritance is one such concept where the properties of one class can be inherited by the other. It helps to reuse the code and establish a relationship between different classes.

Encapsulation - Encapsulation is a mechanism where you bind your data and code together as a single unit. It also means to hide your data in order to make it safe from any modification.
We can achieve encapsulation in Java by:
Declaring the variables of a class as private.
Providing public setter and getter methods to modify and view the variables values.

constructor - A constructor is responsible for preparing the object for action, and in particular establishing initial values for all its data,. Always called when an object is initialised. will have same name as of class with no return type

## Test Automation Estimation
- How to do Test automation estimation?
Answers
Factors affecting automation
- Scope of project ( break down project into small modules, analyse modules for suitability for automation , identify test candidate)
- Identify complexity of test cases ( Identify steps/ verification points in test cases, preconditions, effort to setup date and cleanse them, split them into various complexity ( small , medium, large - use fibanocci series ) . 
- Identify tools required ( IDE, loggers, build tool, reporting tool etc) 
- Identify features needed for automation framework. Estimate time required for features implementaion, add buffer etc
- Estimate time required for env setup
- Estimate effort needed for setting up/ validating pre conditions
- Estimate management overhead for coordination, reporting etc

## ROI Calculation
ROI = Gain from Automation / Investment
Gain from automation include time saved by running automation( difference between automated run and manua run) , savings on analysis time, etc
Investment include tool cost, script development time effort, additional cost for automated testers

Above this, ROI should consider factors like additional test run, increase coverage, frequent test execution , more chances of doing exploratory tetsing etc

## Things to ask back
- why is this role open
- Have you had track record of mentoring people
- where do u see this role leading me to 
- what type of person do you look for ?
- what all attributes should be there to be a cultural fit
- What exactly is my role
- Will I be working desktop application or web application
- how long is current project running for ? or am i  going to start a new one? who all are team members and what skill set do they have. 
- do we have enough forecast for long term
- How many projects will I have to work parallely
- how will be a normal day for me in office
- do you have flexible work cultureo