## Phase 7: Write unit tests for your code.

[Home](../../README.md) | Previous: [Phase 7](phase7.md) | Next: [Phase 9](phase9.md)

##### Prerequisites

 * You should have read the [Test Guidelines](https://prima.corp.telenor.no/confluence/display/BHUB/Test+Guidelines) (You **dont** have to read about the **SoapUI** parts for now).
 * Basic understanding of [Mockito](https://www.baeldung.com/mockito-series).
 * Basic understanding of [AssertJ](https://www.baeldung.com/introduction-to-assertj).
 * Basic understanding of [JUnit 5](https://www.baeldung.com/junit-5).

### Writing your Unit tests

How to write good unit tests is a **huge** topic and is not the target of this tutorial. However, some tips and
tricks is provided to get you started. And also some bulletpoints on what you should ask yourself writing
tests like this.

As you see from the **Test Guidelines** in the **Prerequisites** section, BSS comes with some
recommended tools and frameworks to use when writing tests. You are free to use other tools, but again try to
consider the cost of having tens of different tools in different projects on the platform. Selecting any of the
tools as suggested will also be supported by the platform team.

### Some examples of Mockito, AssertJ and JUnit 5

Given the recommended tools to use, the source code of this tutorial comes with some examples of some of
the vast features of the different tools. Have a look at the examples and read comments (or method signatures)
describing what is achieved (and feel free to contribute on the examples):

 * [MockitoFeaturesTest.java](../../src/test/java/com/telenor/bss/star/MockitoFeaturesTest.java)
 * [AssertJFeaturesTest.java](../../src/test/java/com/telenor/bss/star/AssertJFeaturesTest.java)
 * [JUnit5FeaturesTest.java](../../src/test/java/com/telenor/bss/star/JUnit5FeaturesTest.java)

Try adding some tests in your code and test some of the features.

### Writing unit tests for your code

Lets write some unit tests for our code. We are using Mockito, AssertJ and JUnit 5 for this. We limit the tests
to our services. You may write unit tests for other classes as well. Remember the following:

 * A unit test should be isolated! Test only one class and mock/stub all of its surroundings. Only changes to the class you are testing should affect your unit test; not changes to other classes!
 * It should be **fast**!
 * A good Java package structure will help you in writing tests for the code that is important:
   * Testing packages **service** and **config** is more important than for instance **model** package.
   * Will also help in focusing on **code coverage**. Having 100% test coverage on the **model** package does not say much.

#### TempConverterServiceImplTest.java
With that in mind, lets have a look at a unit test implementation for our SOAP webservice ( [TempConverterServiceImpl.java](../../src/main/java/com/telenor/bss/star/service/ws/TempConverterServiceImpl.java)):

```java
package com.telenor.bss.star.service.ws;

import org.junit.jupiter.api.BeforeAll;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.Arguments;
import org.junit.jupiter.params.provider.CsvSource;
import org.junit.jupiter.params.provider.MethodSource;

import java.util.stream.Stream;

import static org.assertj.core.api.Assertions.assertThat;

class TempConverterServiceImplTest {

    // The service class we want to test
    private static TempConverterServiceImpl service;

    /**
     * Testdata paramters for the test fahrenheitToCelcius_withMethodSource_verifyExpected
     */
    static Stream<Arguments> fahrenheitToCelciusData() {
        //@formatter:off
        return Stream.of(
                Arguments.of(212, 100),
                Arguments.of(50, 10),
                Arguments.of(-148, -100));
        //@formatter:on
    }

    @BeforeAll
    public static void setup() {
        service = new TempConverterServiceImpl();
    }

    /**
     * Simple test method testing that 100 celcius is 212 fahrenheit.
     */
    @Test
    void celciusToFahrenheit_with100_returns212() {
        assertThat(service.celciusToFahrenheit(100).getFahrenheit()).isEqualTo(212);
    }

    /**
     * Parameterized test method testing that several celcius to fahrenheit results. Method uses
     * CsvSource for feed the test with input.
     */
    @ParameterizedTest
    @CsvSource(value = {"100, 212", "10, 50", "-100, -148"})
    void celciusToFahrenheit_withCsvSource_verifyExpected(double celcius, double fahrenheit) {
        assertThat(service.celciusToFahrenheit(celcius).getFahrenheit()).isEqualTo(fahrenheit);
    }

    /**
     * Simple test method testing that 212 fahrenheit is 100 celcius.
     */
    @Test
    void fahrenheitToCelcius_with212_returns100() {
        assertThat(service.fahrenheitToCelcius(212).getCelcius()).isEqualTo(100);
    }

    /**
     * Parameterized test method testing that several fahrenheit to celcius results. Method uses
     * MethodSource for feed the test with input (using static method fahrenheitToCelciusData as input).
     */
    @ParameterizedTest
    @MethodSource("fahrenheitToCelciusData")
    void fahrenheitToCelcius_withMethodSource_verifyExpected(double fahrenheit, double celcius) {
        assertThat(service.fahrenheitToCelcius(fahrenheit).getCelcius()).isEqualTo(celcius);
    }
}
```

This is the simplest of the two services we have since it only contains mathematical calculations and has no
downstream services or any other injected dependencies. Both methods are tested in a simple way and one using
the feature of Parameterized Tests from JUnit 5. Mathematical calculations like this is a good example where
Parameterized Tests makes sense.

Implement the test yourself. Try not to only copy paste the test class, but write the test using your
preferred IDE.

#### TempConversionControllerTest.java

The REST service adds some complexity since it calls our injected SOAP service. And even though it would work
perfectly fine to test the REST service by injecting the SOAP service before running the test, it violates the
concept of unit testing. Because you will in fact test both services. Which is redundant since you have already
testet the SOAP service.

So to write a correct unit test for our REST service means mocking the SOAP service which leaves us with the
scenarioes: *Test REST service given that the SOAP service returns the following*.

**!!! Because we do not care how the SOAP service is implemented, only what it returns !!!**

There are several ways to mock the SOAP service using Mockito:

 * You may add a contructor or setter for the SOAP service so you can initialize the test with a Mock of the SOAP service. **This is kind of old school. Mockito provides better ways!**
 * You may refactor your REST service to move the actual calls to SOAP service into protected methods and use Mockito Spy feature to mock only the calls to SOAP service (partial mocking). Will require some refactoring and is more suited if you for instance called the "SOAP" service via a static method. To achieve a valid unit test in that scenario you would have to introduce something like PowerMock to mock the SOAP service call. PowerMock is slow and the powers of Mockito makes the need for such frameworks obsolete!!
 * You may use Mockitos powerful mock injection features.

In the following example we use the latter: The mock injection features in Mockito:

```java
package com.telenor.bss.star.service.rest;

import no.telenor.bss.tempconverterservice.ws.TempConverterServicePortImpl;
import no.telenor.bss.tempconverterservice.ws.TempResult;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.MockitoAnnotations;

import static org.assertj.core.api.Assertions.assertThat;
import static org.mockito.Mockito.doReturn;

class TempConversionControllerTest {

    /**
     * Have Mockito creating a Mock for the class TempConverterServicePortImpl
     */
    @Mock
    private TempConverterServicePortImpl soapService;

    /**
     * Have Mockito create an instance of the class we want to test and also inject whatever mocks/spies available.
     */
    @InjectMocks
    private TempConversionController restService;

    /**
     * Wire it all together by handing this testclass over to Mockito.
     */
    @BeforeEach
    public void initMocks() {
        MockitoAnnotations.initMocks(this);
    }

    @Test
    void celciusToFahrenheit() {
        // Stub call to SOAP service: Return defaultTempResult() when SOAP service celciusToFahrenheit is called with
        // 255
        doReturn(defaultTempResult()).when(soapService).celciusToFahrenheit(255);

        assertThat(restService.celciusToFahrenheit(255).getFahrenheit()).isEqualTo(100);
    }

    @Test
    void fahrenheitToCelcius() {
        // Stub call to SOAP service: Return defaultTempResult() when SOAP service fahrenheitToCelcius is called with
        // 300
        doReturn(defaultTempResult()).when(soapService).fahrenheitToCelcius(300);

        assertThat(restService.fahrenheitToCelcius(300).getCelcius()).isEqualTo(100);
    }

    /**
     * Default SOAP service response: returns hardcoded 100 for both celcius and fahrenheit
     */
    private TempResult defaultTempResult() {
        TempResult result = new TempResult();
        result.setFahrenheit(100);
        result.setCelcius(100);
        return result;
    }
}
```

To understand how this test works read the different comments. We have 2 simple tests, but prior to running the
actual test we stub the response of the SOAP service.

Implement the test yourself. Try not to only copy paste the test class, but write the test using your
preferred IDE.

#### A different approach

Lets have a look at a different approach for the REST service test. As stated earlier, a unit test should be
totally isolated. That is only partly true in the REST service test above. We have made sure we are not
affected by the SOAP service **implementation**. But we still reference the SOAP service class. Which means:

 * If the SOAP service changes in context of signature (changed method signature, package change etc), we will have to update both our REST service class and its corresponding test.

This is because we reference the SOAP service class (but not its implementation). It is possible to get closer
to a true isolated unit test by removing the reference to SOAP service class entirely. In this case it will
require some refactoring in our REST service implementation. And it might be that it is something you dont
want to do. The alternative implementation is just showed here to illustrate how you **could** get a more
isolated unit test.

##### TempConversionControllerRefactored.java

Below is a refactored version of our REST service. Read the comments about the changes as oppose to the original. You can add this class in your **test** folder, not **main** folder:

```java
package com.telenor.bss.star.service.rest;

import com.telenor.bss.star.model.Celcius;
import com.telenor.bss.star.model.Fahrenheit;
import no.telenor.bss.tempconverterservice.ws.TempConverterService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

public class TempConversionControllerRefactored {

    private TempConverterService tempConverterService;

    public Fahrenheit celciusToFahrenheit(@PathVariable("celcius") double celcius) {
        // Call an internal method to call the actual SOAP service
        double fahrenheit = getFahrenheitFromSoapService(celcius);
        return new Fahrenheit(fahrenheit, celcius + " Celcius is " + fahrenheit + " in Fahrenheit");
    }

    public Celcius fahrenheitToCelcius(@PathVariable("fahrenheit") double fahrenheit) {
        // Call an internal method to call the actual SOAP service
        double celcius = getCelciusFromSoapService(fahrenheit);
        return new Celcius(celcius, fahrenheit + " Fahrenheit is " + celcius + " in Celcius");
    }

    // Method that can be stubbed in unit test
    double getFahrenheitFromSoapService(double celcius) {
        return tempConverterService.celciusToFahrenheit(celcius).getFahrenheit();
    }

    // Method that can be stubbed in unit test
    double getCelciusFromSoapService(double fahrenheit) {
        return tempConverterService.fahrenheitToCelcius(fahrenheit).getCelcius();
    }
}
```

Now, we can write a unit test for this class that no longer has any reference to the SOAP service class. So
if the SOAP service is changed, only our REST service will be affected, not our corresponding unit test. The
unit test now looks like this:

```java
package com.telenor.bss.star.service.rest;

import org.junit.jupiter.api.Test;

import static org.assertj.core.api.Assertions.assertThat;
import static org.mockito.ArgumentMatchers.anyDouble;
import static org.mockito.Mockito.doReturn;
import static org.mockito.Mockito.spy;

class TempConversionControllerRefactoredTest {

    @Test
    void celciusToFahrenheit() {
        // Get the test class spy with stubbed values for SOAP service
        TempConversionControllerRefactored restService = getClassToTest();

        assertThat(restService.celciusToFahrenheit(255).getFahrenheit()).isEqualTo(100);
    }

    @Test
    void fahrenheitToCelcius() {
        // Get the test class spy with stubbed values for SOAP service
        TempConversionControllerRefactored restService = getClassToTest();

        assertThat(restService.fahrenheitToCelcius(300).getCelcius()).isEqualTo(100);
    }

    private TempConversionControllerRefactored getClassToTest() {
        // Instead of using a clean instance of TempConversionControllerRefactored, we create a spy around it.
        // The spy makes it possible to stub specific methods in the class we want to test. The rest will be as-is.
        // We stub both calls to SOAP service (Return 100 always, regardless of input (anyDouble())
        TempConversionControllerRefactored result = spy(new TempConversionControllerRefactored());
        doReturn(100D).when(result).getCelciusFromSoapService(anyDouble());
        doReturn(100D).when(result).getFahrenheitFromSoapService(anyDouble());
        return result;
    }
}
```

Again, read the comments to see how the test is setup. The unit tests are identical in both cases. Except for
the latter has completely removed any reference to the downstream SOAP service.

This is just an alternative approach. It might not fit your needs. It might be that you do not want to do
the refactoring required to achieve this implementation. But: If you want to stub static method call, this
is the preferred way. For the love of God, do not include PowerMock or any of his friends!

**Commit and push your code!**

##### At this point you should be able to:

 * What is the main features of Mockito? What does it provide and not?
 * How do you mock/stub external calls from your unit test in Mockito? Consider the different approaches and the differences between them.
 * What should a unit test actually test and **not**?
 * What is the difference between Mockitos **mock** and **spy** concepts?
 * What is test coverage? What is good test coverage? How can I easily see my test coverage?
 * How do I mock/stub a static method call?
 * What is the difference between a Unit test and a Integration test?
 * Consider the following statement: "Do not use Mockitos **@Mock** annotations and/or JUnits common **Before\*/After\*** annotations in your Unit tests". It is just a statement. Can you think of arguments that supports this statement?
 * What is parameterized unit test? How can that be implemented?
 * What are assertions and what is good assertions?
 * How can you debug your code during test execution?
 * Have you tested Mockitos **ArgumentCaptor**? Test it in one of your tests and understand what it does.
 * How do you isolate your unit test to the extent that your test is only dependent on the actual class you are testing and only that?

[Home](../../README.md) | Previous: [Phase 7](phase7.md) | Next: [Phase 9](phase9.md)

## Phase 8: Write integration tests for your code.

[Home](../../README.md) | Previous: [Phase 8](phase8.md) | Next: [Review](../review.md)

##### Prerequisites

 * The same as for the previous phase about unit testing

### Writing your Integration tests

So lets extend the scope of the tests a little. While **unit tests** focuses on one class and its business logic,
**integration tests** focuses more on your code as a whole. Meaning: you will run tests to check that the different
components in your application interacts in the way you intended them to do.

In practise this means that you move the mocking/stubbing further out. Instead of mocking one class' surroundings,
you now mock more at an application boundary level: *Write tests that test all the way from an entry point
of your application up until (but not including) downstream dependencies* You basically want to test the entire
internal call chain for a entry point, but have mocks towards all external calls. To be able to control
the response from those external calls. And the fact that you do not care how the externals are implemented or if
they are up an running. Only what they return. Isolation, but at the application level.

One important thing before writing integration tests:

 * Make sure you know what is already tested in the unit tests. Do not make the mistake of basically
 duplicating the unit test implementation in your integration tests. This will leave you with double
 the maintenance. And no added value. Integration tests will more often focus on the wiring and making
 sure a certain call chain works. And not the actual values. But this will differ. Just dont repeat yourself
 (see [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself)).

#### Writing integration test using Mockito

We will write an integration test for our REST service. Which again will call our SOAP service, which will also
be included in the test (we do not mock it away like we did in the unit test for the REST service). We do not
write a similar test for the SOAP service because it has no dependencies and we already have a unit test for
it.

To achieve a good Integration test using Mockito we can reuse the feature of **mock injections**. But where the
unit test for the REST service added a mock for the downstream SOAP service, we will instead use a Spy. This
will make sure that Mockito creates an instance of the SOAP service and then injects that instance into our
REST service. Since default behaviour of a Spy is that all operations towards the spy will be passsed to the
underlying object, we will test both our REST service and SOAP service. It is kind of cheating, but it works
well. We simply use Mockito to wire our dependencies (just like Spring does which we will use in the next
example):

##### TempConversionControllerIT.java

```java
package com.telenor.bss.star.service.rest;


import com.telenor.bss.star.service.ws.TempConverterServiceImpl;
import org.junit.jupiter.api.Tag;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Spy;
import org.mockito.junit.jupiter.MockitoExtension;

import static org.assertj.core.api.Assertions.assertThat;

// MockitoExtension: Wire it all together by handing this testclass over to Mockito.
@ExtendWith(MockitoExtension.class)
// JUnit 5 feature: tag this test as "integration". Not required.
@Tag("integration")
public class TempConversionControllerIT {

    /**
     * Have Mockito create a Mock for the class TempConverterServiceImpl
     */
    @Spy
    private TempConverterServiceImpl soapService;

    /**
     * Have Mockito create an instance of the class we want to test and also inject whatever mocks/spies available.
     */
    @InjectMocks
    private TempConversionController restService;

    @Test
    void celciusToFahrenheit() {
        assertThat(restService.celciusToFahrenheit(100)).isNotNull();
    }

    @Test
    void fahrenheitToCelcius() {
        assertThat(restService.fahrenheitToCelcius(212)).isNotNull();
    }
}
```

Running this test will execute both our REST and SOAP service. And to make a point of the concept of **DRY**,
we are not testing the logic of converting from one scale to the other (that is already tested in unit tests),
just that we get a result. Of course it would have been ok to check the values here as well. But the point is
that it should not test the same as in unit test.

The good thing about using this approach using Mockito is:

 * It is very **fast**, keeping our test execution time down.
 * It is, as long as the call hierarcy is quite simple, also very readable (a more complex call hierarchy will leave you with several Spies and Mocks etc which might clutter your test a little)
 * We use the same framework as for unit tests which should be familiar.

Try writing the test yourself!

##### TempConversionControllerITWithSpring.java

Dependent on what you actually want to achieve by running the integration test:

 * In the case of Mockito, we *creatively* wire dependencies and *creatively* uses Mockitos' Spy feature.
 * It might be good arguments for testing the wiring in the application through Spring itself. But this comes
 at the cost of slower tests since you will run the test in Spring context.

So lets write the same test using Springs test features and running the test using SpringBoot.

```java
package com.telenor.bss.star.service.rest;


import org.junit.jupiter.api.Tag;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.remoting.jaxws.JaxWsPortProxyFactoryBean;

import static org.assertj.core.api.Assertions.assertThat;

// Run test in SpringBoot and run it in a full env using the defined port.
@SpringBootTest
@Tag("integration")
public class TempConversionControllerITWithSpring {

    @Autowired
    private TempConversionController restService;

    @MockBean
    private JaxWsPortProxyFactoryBean tempConverterService;

    @Test
    void celciusToFahrenheit() {
        assertThat(restService.celciusToFahrenheit(100)).isNotNull();
    }

    @Test
    void fahrenheitToCelcius() {
        assertThat(restService.fahrenheitToCelcius(212)).isNotNull();
    }
}
```

First: there are several ways to setup an Integration test like this in a Spring context. This only shows one
way. This test also becomes a little strange because of how our application is designed. Remember that we have
a REST service calling a SOAP service as if the SOAP service were a downstream service not in our application
(see the **WebServiceClientConfiguration.java** class). But the SOAP service is both "internal" and "external"
which complicates things a little. But with the power of SpringBoots test library, all can be handled.

Our setup ends with us, calling ourselves through http://localhost:8080/ws/TempConverterService). Default when
annotating with **@SpringBootTest**, the **webEnvironment** is set to be **MOCK**. Which basically means that
we do not start an actual servlet container on a specific port, but rather a mocked servlet container. In that
case http://localhost:8080/ would not be available.

So what we need to do is to mock our downstream service. There are, again, several ways of doing this, but one
way is to simply mock the **JaxWsPortProxyFactoryBean** created in **WebServiceClientConfiguration**. This
will prevent Spring from setting up a JAX-WS client proxy towards http://localhost:8080/ws/TempConverterService
(which will not be avaiable anyhow because we do not start a servlet container at port 8080. By mocking
this, we are still going to have the SOAP service wired because it is also an internal bean. And that
bean will be injected by Spring into our REST service.

The result being that the integration test is identical as the **TempConversionControllerIT.java** test. The
only difference being the wiring: Mockito vs. Spring.

Try writing the test yourself!

**Commit and push your code!**

#### SpringBoots test library Mockito support

In the last example we had a downstream service; but again not, because we are calling ourselves. So we do not
mock the downstream service (stubbing its responses). In a normal scenario a downstream service, or a
database call or whatever, would have been mocked with stubbed values.

One of the many features of SpringBoots test library is its Mockito support! The library has the following
two annotations: 1) MockBean and 2) SpyBean. Corresponding to the Mock and Spy class in Mockito. Now consider the
following fictive service:

```java
@Service
public class MyService {

    @Autowired
    private SomeDatabaseCall database;

    @Test
    private int oneHundredPlusDatabase() {
        return 100 + database.getCount();
    }
}
```

We now want to write an integration test for this. But we would like to stub the result of calling **database**.
With the Mockito support, we can do this easily. Just like we did in the [TempConversionControllerTest.java](../../src/test/java/com/telenor/bss/star/service/rest/TempConversionControllerTest.java)
The only difference being the annotations we use:

```java
@SpringBootTest()
@Tag("integration")
public class MyServiceIT {

    @Autowired
    private MyService service;

    @MockBean
    private SomeDatabaseCall database;

    @Test
    private void myTest() {
        doReturn(100).when(database).getCount();

        assertThat(service.oneHundredPlusDatabase()).isEqualTo(200);
    }
}
```

We get **MyService** injected by Spring into our test. Then (with the **@MockBean** annotation), Spring will
also create an instance of **SomeDatabaseCall**. This instance will be of type Mockito Mock (we could also have
annotated it with **@SpyBean** and get a Mockito Spy instead). When Spring then discovers that MyService is
wired with a SomeDatabaseCall, it will inject the mock we have setup; bypassing the real wiring.

Since the SomeDatabaseCall mock is injected into our service class and it is of type Mocktito Mock we can
do things like:

```java
doReturn(100).when(database).getCount();
```

This kind of seamless integration between Spring and Mockito is quite powerful and your can create quite
complex integration test with somewhat small effort.

##### At this point you should be able to:

 * How can you write Integration tests using only Mockito?
 * How can you write Integration tests to have it running in a Spring context?
 * How can you write Integration tests to have it running in a Spring context and mocking downstream services using the glue between SpringBoots test library and Mockito.
 * What is the difference between the two approaches? Benefits? Downsides?
 * Are there other approaches to achieve a good Integration test without using Mockito **mock/spy** concepts or running the tests in a Spring context? Meaning: how can your setup your test to wire up all classes you want to test?
 * What can you use **Wiremock** for?
 * Consider the following statement: "Your integration test should mainly test one scenario/have only one test method". It is just a statement. Can you think of arguments that supports this statement?

[Home](../../README.md) | Previous: [Phase 8](phase8.md) | Next: [Review](../review.md)