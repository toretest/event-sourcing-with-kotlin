## Phase 1: About Functional testing

[Home](../../README.md) | Next: [Phase 2](phase2.md)

##### Prerequisites

 * You should have read the [Test Guidelines](https://prima.corp.telenor.no/confluence/display/BHUB/Test+Guidelines) (You can focus on functional testing and **SoapUI** parts for now).

 ### Before doing any coding

 Now that you have done the **Basic** tutorial and have had that reviewed and merged into your master branch,
 create a new feature branch (**feature/tutorial-functional-testing**) and commit and push your code to that branch for
 all code in this tutorial.

### The concepts of functional testing

Having read the **Test and Debug Guidelines** document you will know that we have defined 4 different test categories.
Unit and Integration tests are typically written alongside your code, most likely in the same language as the code itself.
These tests are totally or partially isolated and typically tests business logic or integration between different parts
of your code.

Moving further we would like to test the system at a higher level. There are several category/type/level terms
out there to describe all these. You have probably heard them all: System-, Acceptance-, Regression-, Functional-,
Smoke-, Performance tests and so on. Unit and Integration tests are somewhat given and clear defined. And to
keep it as simple as possible we would like to have as few categories as possible. Which means that all
tests that are not Unit, Integration or Performance tests are categorized as Functional tests. Indicating
that they are testing the functional requirements of the system. Which again is a good bag to put the rest in. It
could have been argued that API testing is better term given the context we are in, but the important thing
is the simplicity rather than the naming.

### Implementing functional tests

BSS platform does not require functional tests to be written given a specific tool or technology. However, **SoapUI**
have been used for these tests during establishment of the platform. The reason for this choice was somewhat
historical and somewhat based on the competency in the team at the time. There were also existing tests
in SoapUI in code that was moved to the new platform. But you are free to choose. Know however that, as shown in this
tutorial, there have been established routines, tools and so on that **supports SoapUI out of the box**.

It is important
that not every developer/tester in every team go their own way every time. We should respect the fact that shared
technology between the teams gives immediate upside for everyone. If your first thought is "I hate SoapUI, I would
like to use my preferred tool" make sure you know the cost of introducing it! And make at least sure that you check
if other teams have come to the same conclusion. And that you work in the same direction. Or you will set back everyone.

### The concepts of DRY

One of the main problems when many tests are written for the same code and by different team members is duplication
of tests. Code that is tested in detail in Unit tests are also tested in the same way in Integration- and/or in
Functional tests. It is important to focus on not repeating yourself. This is what **DRY (Don't repeat yourself)** is
all about!

There should be communication between developers writing Unit- and Integration tests
and for instance a test team writing Functional tests. Communication between the members on *which*
tests are responsible for testing *what* is important. Let the test team be aware of what is covered
by the Unit- and Integration tests. Don't duplicate the tests scenarios! It will be a waste of time
during implementation and test phase. And a waste of time during maintenance where you need to fix one
test case in several places!

##### At this point you should be able to:

 * Know the concepts of Functional testing on the BSS platform.
 * Understand that SoapUI is preferred tool for implementing tests.
 * Start the SoapUI GUI and be ready to implement tests.

[Home](../../README.md) | Next: [Phase 2](phase2.md)


## Phase 2: Writing a Webservice test with SoapUI

[Home](../../README.md) | Previous: [Phase 1](phase1.md) | Next: [Phase 3](phase3.md)

##### Prerequisites

 * Understand the core concepts of [SoapUI](https://www.soapui.org/getting-started/soap-test.html) testing.
 * [Download](https://www.soapui.org/downloads/soapui.html) and install SoapUI GUI locally (Open Source version is enough)
 * **[Turning on security in your application](../howtos/phase1.md)**.

### Creating your test project

Before creating your first test project you should be aware that BSS platform provides what is known as the
[BSS Test Web](https://source.telenor.no/stash/projects/BPT/repos/bss-test-web/browse). This is a web application that can be used for:

 * Storing your tests and test properties.
 * Test automation purposes.
 * A common place where non-technical people can execute tests from, towards all supported environments. Verifying that all is good.

This will be described later in this tutorial. For now we will implement and save the tests in our project in
Git. Later we will move it to BSS Test Web and execute the test from there.

Make sure your application is running locally and start the SoapUI GUI. When starting SoapUI your screen will look like this:

<br/><img src="img/phase2-1.png" width="1000" /><br/>

Now click on the SOAP icon in the upper left (as marked by "1")

<br/><img src="img/phase2-2.png" width="600" /><br/>

Paste **http://localhost:8080/ws/TempConverterService?wsdl** into *Initial WSDL* (Project Name will we populated). Then remove the "**?wsdl**"
part from *Project name* and toggle off *Create sample request for all operations* (you dont need it). Then click **OK**.

Your **Navigator** window will now look this:

<br/><img src="img/phase2-3.png" width="300" /><br/>

It will display your Project name (TempConverterService). Below your project you will see the result of the imported WSDL
(TempConverterServiceSoapBind). And below you will see a list of all available operations the Webservice provides. Right-click
on your project and select *Save Project As*. Call your file the same as your GIT repo name (**bss-webservice-star.xml** in this example)
and save it into your application project under the path **src/test/resources/soapui/**.

### Implementing our first tests

We will now write one test for each operation. First we need to create a *Test Suite* and *Test Case* in our SoapUI project.
Right-click on your project (TempConverterService) and select *New TestSuite* and call it for instance
"TempConverterService" also. Now right-click on your created TestSuite and choose *New TestCase*. Name it the same.
Your navigator window should now look like this:

<br/><img src="img/phase2-4.png" width="300" /><br/>

Under *Test Steps (0)*, right-click and add a new *Step* -> *SOAP Request*. Call the step **celciusToFahrenheit** and click **OK**.
In the next window you select the corresponding operation (**celciusToFahrenheit**) from the list and click **OK**. Your next window
will look like this:

<br/><img src="img/phase2-5.png" width="500" /><br/>

Toggle on all the checkboxes. This will add **Assertions** to the test (you can add more later). **IMPORTANT** Creating a test without a
single assertions is considered to have execution status *Unknown*. Dropping assertions in the test will only indicate on success
that the service is up and running. But not what is the expected behaviour. So add your assertions! Now click **OK**. A test step window will
popup, looking like this:

<br/><img src="img/phase2-6.png" width="800" /><br/>

Your will see the following:

 * A **request** window in the upper left with a *XML* and *Raw* tab.
 * A **response** window in the upper right with a *XML* and *Raw* tab.
 * A **result** window in the lower part. Click on the *Assertions* tab to see the assertions.

Before running the test, change the argument (arg0) of the request from **?** to **50**. (The name **arg0** is only because we have not given
the parameter a good name in our WSDL). Now run the test by clicking at the *Play* icon. The test will fail. It will be marked as red both in
your *Navigator* window and in your assertions. If you click on the *Raw* tab in your response window you will see that the test fails with
**HTTP status 401**. This is because the service requires authentication:

<br/><img src="img/phase2-7.png" width="800" /><br/>

To fix this we need to add a *Basic authentication* config to the test. You will see some tabs just above the result window
*Auth* - *Headers* - *Attachments* .... Click on *Auth*. You will see that you have *No Authorization* selected. Add a new of type *Basic*.
Then you will have a window to add *Username* and *Password*. Simply add **user** as username and **user** as password (as you configured
in your application). Now run the test again. It will now turn green:

<br/><img src="img/phase2-8.png" width="800" /><br/>

You will notice the following:

 * In the *response* window, looking at the *XML* tab you will see the expected result.
 * The test is marked as green in your *Navigator* window.
 * All the *assertions* are green. The response is:
   * a valid SOAP response
   * in compliance with the Schema
   * does not contain a SOAP fault

You may add as many assertions as you see fit. We will add an assertion that verifies that the response contains the correct result. This is
probably covered by Unit tests and breaks the concepts of *DRY*, but we do it here for educational reasons.

Click the *plus* icon above the Assertions window to add a new assertions. There are several assertions to pick from. For simplicity we will
add a *Contains* assertion. The correct here would probably be to add a *XQuery/XPath Match*, but you can play around with this. We add a
*Contains* assertion and assert that the response contains the text **&lt;fahrenheit&gt;122.0&lt;/fahrenheit&gt;**. The assertion will be
added in the assertion list and immediately turn green. The reason is that a test step response always remembers the latest execution. So the
assertion is run towards the last response.

Now Save your test and then to the same for the operation **fahrenheitToCelcius**. Finally save and commit and push your code to Git.

##### At this point you should be able to:

 * Create a new project in SoapUI GUI
 * Import a WSDL into your test project
 * Create Test Suites, Test Cases and Test Steps.
 * Add authentication details to your request
 * Add assertions to the test reponse

[Home](../../README.md) | Previous: [Phase 1](phase1.md) | Next: [Phase 3](phase3.md)