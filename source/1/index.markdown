---
layout: page
title: ""
date: 2017-07-23 10:15
comments: false
sharing: false
footer: true
---


#Test Transformation
- how to do you measure maturity of test process. what are the various metrics you can use
   Talk about TMM maturity models. Where can it be
   Talk about how it can improve to next level
   
   
- What all test metrics can you use

https://www.tricentis.com/blog/64-essential-testing-metrics-for-measuring-quality-assurance-success/
explain what could be some relevant one. also explain why few one cannot be generailsed

- How to do you bring change in human mind set to improve test automation
- how do you handle conflicts

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

## Test Manager Interview Questions - Woolies X

* How do you handle conflicts within the team members. what all actions can you take
* How do you handle mental health issues like anxiety, depression etc in the team
* When have you felt demotivated and how you handle that
* How do you upskill yourself
* Any experience in publisher - consumer models
* Experience in cloud based systems
* Experience with CI CD tools
* What is your career plan
* What will you do to avoid attirition
* 

## Test Transforamtion ##
- 



Describe your biggest accomplishment and failure (if you had one) in your most recent role
- take about efforts taken to shift left testing and quality as a team responsibility
 - not raising defects every now and then. working collaboreitvily with devs, reducing count of defects, increasing unit test code coverage
 - convincing higher managerment that defect leakage is not bad and just monitoting the number of defects in prod vs lower environemnt alone,doesnt give quality
- we also compare how many times a task moved between test/ dev to get an understanding pf rigoursly we tested and use that in metrics

Biggest process imprevement
- we did a survey every sprint and noticed that team mood is generally bad toward release readines smeeting
- during sprint retro we discussed about it and found that it is mainly due to few tough stakeholders questioning development process, tech debt, open defects etc and raising risk around it. There are few GM level peopke as welll
- we implemented a process to meet with tough stakeholders first and walk them through the outstanding task before
this improved reales readines process


. Tell me about a time you adapted to a big change at work. How did you adjust to your new environment?
Make up something when scrum master left and I had to take up additional responsibility there
Changes when banks deprioritized
Scrambling to keep teams in place, making sure knowledge is not lost, ensuring documentation is in place

Share an example of when you had a problem with employee behavior. How did you resolve the issue?
- Can complain about offshore team not being on time due to trafiinc , dance and other activities. Not getting responses on time, setting up status call even after hours to support

 Give me an example of a time you made a mistake at work and explain how you fixed the issue.
---- talk about account origination error .. Thought Tm will be right, but it was wromnh. Need to get information from actual enduser,s takeholder.. Came up with remedial action, identified who is business takeholder, who to approve etc

Tell me about a time when your team had to work on a tight deadline. How did you ensure everyone completed their work on time?

 Netbank regression team is always timehcetice. Plan test cases and assign indivisually based on their area of expertise, get direct feedback , remove blocker
Did risk based prioritization, handled external stakeholders


Tell me about a time you successfully delegated tasks.
I try to avoid micro managing . Normally give full responsibility to team members for their action
It has backfired sometimes
Talk about 3 banks happening at same time even meeting scheduled at same time.. Delegated most of the task to other teammembers

When have you had to persuade others to see a situation from your perspective? Can you give an example and show how you explained your point of view?

Talk about merchant statements. No test data and env.. Entire team was of opionion that we test locally with stusb and move past . They were confident in designde and roll out approach
But created issues where ids were recycled. Raised risk , have acceptance

• Tell me about a project where you’ve had to use your analytical skills. How did you demonstrate your skills and what was the outcome of the project?
This is used more like BA role. I had played BA role during merchant statements where I tried to analyse the impacts of all systems

How do you manage stress among your team members?
Give me an example of how you mediated a conflict between your employees or colleagues.
Talk about Harish and raji . Onshore and offshore counter part. We rotate them every now and then 
Created issue when harish went offshore
	• Talk about harish and raji
	• No of test cases run per day onsfore/offshroe etc
	• Tlak it out, work coperatively
Started to notice resentmens in 1 - 1 and finally helped to find him another proejct

• What have you done in the past to alleviate stress from your team members?
•   identify what is causing stress. Talk to them. See whether we can manage it by resolving it
• Redirect to origiantional policicies like conselling etc
Help for leav


Tell me about a situation where you took initiative without being asked.
Talk about current projcet where TM was sick for few days and had attended some meeting on her behalf. I took over all coordiantion activities and it came to aplace when everything ws done by me

 Provide an instance when you had to split your time between several projects. How did you prioritize?
This is very common in CBA days.i had t manage multiple projects
I assume, you were referring to how to prioritize tasks for multuple priority projects.. I normally define priority and impact and effort needed. Also look at timeline. Normally try to prioritze what ever has to be done first. If there are many  , I normally look at their priority / effort and try to do the least effort one to get it through. It is not a hard and fast rule. Also look at task which will unblock others first 


• What’s the most stressful or difficult situation you’ve faced at your previous job? How did you handle it?
		○ Credit card signage issue
		○ High level meetings with CIO, why it missed. How can we fix, actions involved. Heightned scruitny into next release



• Have you ever missed a deadline? What happened? What would you do differently next time?
We had instance where tetsing didn’t finish on time. It depends on multiple external interaces. Our cba deployment was delayed too much. Their test env

• Tell me about a time you had to deliver bad news to a manager or team member. How did you do it? What was the other person’s reaction?
	• Show stopper defect on last sign off day
	• Literally angry
	We made a fix and deployed on the day

Tell me about a time you had to deal with a difficult colleague. How did you communicate with the colleague effectively?
- prepare well - what needs ot be highlighted, how are they going to response, what outcome wa sI looking at , what can motivte them
- take step back - make sure feedback is received from everyone whom he has worked
- make sure feedback is constrcutive which have way to imprpve , rather than just feedback
- manage emotions and make sure i remain calm
- put conversation on record

What is your management style
- doenst prefer to micro manage
- i make sure team understand what need to be fone and give highlevel task . I normally leave the team to select teh lowlevel details by them.
 I prefer to remain hands-off when it comes to individual tasks, but at the same time, I’m always available for help, guidance and assistance when needed.
then do informal checkin to make sure everyone can deliver the task and overall that will match up tp whats needed

Tell me about a time you led by example
- take about MSO defect fix and found that application has changed too much . amount worst case scenarios were too many 
- team complained that testing stresfull and their way of selecting test cases is not working and it has too many random  failure
I took lead to look into the code in stash and idenfitied how the logic works and realised that there were too many combinatiosn. I stook initiative to  identiy scenarios , remove duplicates, walkthrough with dev & ba and even execute it and order piza for staying late

 How do you motivate people?



.  Give an example of a tough decision you had to make.



 
2. Describe a situation where you had to collaborate with someone with a different working style.
Effective managers need to be able to work well with all kinds of personalities and working styles. This question gives candidates the opportunity to share their strategies for promoting teamwork among employees who take opposing approaches to completing projects.
 
3. Give an example of a strategy you have used to motivate others.
Asking candidates to think about times when they motivated other people can help you understand how they would support positive company culture and teamwork.
 -- find out what they need.. Find challenging roles .. Provide them , provide suport
Upksill
Career aspiration
Presentation, certification 
  
7. How do you make new employees at your workplace feel like a part of the team? Give an example.
Managers should know how to welcome new employees and incorporate them into the workplace culture. Their answer to this question can give you insight into how much they invest into their team’s happiness and productivity.
	- Employee onboarding ysstem
	- Buddy system or self study
	- Conservastion - make sure twe ask opinion
	- Incldde
	
	
 
8. When was a time when you had to make a difficult choice in the workplace? How did you make your decision and what was the outcome?
This question can help you understand the way each candidate makes decisions. Asking them to talk about a challenging choice gives them the opportunity to show that they have strong situational awareness and don’t need to rely on others to make hard choices for their team.
 
 
 
3. Describe how you used your problem-solving skills to benefit a team or company.
Answering Tip: Demonstrate how you look for solutions for the greater good of the company. Not just solutions for your own good and your team’s good.
4. Tell me about a time when you used creativity to overcome a dilemma.
Answering Tip: Think about a way that you surprised yourself with an unexpected idea. Did you follow a ‘creative process’? Or was your creativity more spontaneous in the situation?
5. What’s the best idea you’ve come up with on a team-based project?
Answering Tip: Brainstorm at least three different ideas and be prepared to discuss one during your interview. Focus on the ones that had the biggest impact.
6. How do you approach problems? What’s your process?
Answering Tip: Focus on the approach you use to solve problems. How do you break them down into steps in order to solve them? What tools and techniques do you use to work through a problem?
7.  Name three improvements you made in your most recent position.
Answering Tip: Make a list so that you’re not stumbling over your words during the interview. Focus more on the result you achieved for this question, and have the ‘3 things’ ready to discuss.
Bonus Example: Think of a time when you’ve used an approach like Design Thinking Or Lean Startup methodology to come up with solutions to a problem. This demonstrates that you’re well versed in typical problem solving approaches used by advanced tech folks out there. Kevin Lee, Founder at PMHQ recommends asking clarifying questions and focusing on the end user, especially for Product Management Interviews at companies like Google. Who is it that you made the improvements for and why? What difference did your improvements make to your end users lives?
Working in a Team
28. Describe a time when you were able to motivate unmotivated team members.
Answering Tip: Focus on your team-building skill set. What do you do to inspire those around you?
32. Have you ever had to counsel a difficult team member? Tell me about that time.
Answering Tip: Pick a time when you had to deliver uncomfortable counsel to a team member.
Bonus Example: Here, you need relevant examples where you stood out for the right reasons as a leader. You need to demonstrate that you know when to lead, when to follow and how to pick the right reasons to strive for. Talk about a time when you’ve communicated a vision, led a team, fought for the right reasons, done something in service of others, or motivated and developed others.
Personal Stress
33. Tell me about a time when you worked well under pressure.
Answering Tip: Make a list of three times and choose the best one. The more important thing is to talk about your mindset when you’re in a pressure situation. Are you mindful about the pressure you’re facing? Or do you just crumble under pressure?
38. Tell about a conflict at your job.
Answering Tip: Keep it focused on a work-related conflict, and what you learned from it.