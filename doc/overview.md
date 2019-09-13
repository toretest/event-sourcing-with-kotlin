## bss-webservice-star

This GIT module is meant as starting point for developers that are going to develop, test and deploy Java services at BSS Hub. This is not a
tutorial on the different technologies in play but rather guidelines of best practices, golden paths and links to resources
that can answer questions you might have on your path.

### About the module

The module is built up as a step-by-step tutorial on how to develop and run some Java microservices on the BSS Hub.
When done, you will have built the same application as in this module. The code in this project is merely for reference
and lookup.

It contains a SOAP service and REST service. The SOAP service uses the principle of [Contract First](https://docs.spring.io/spring-ws/sites/1.5/reference/html/why-contract-first.html). The REST service is setup with Swagger documentation.

The following principles, frameworks and technologies are in play:

 * [Gradle](https://gradle.org/guides/) > Building and running application
 * [SpringBoot](https://spring.io/guides/gs/spring-boot/) > Framework for running standalone Spring applications.
 * [CXF](http://cxf.apache.org/docs/a-simple-jax-ws-service.html) > For JAX-WS API/WebService API enablement
 * [OpenShift](https://www.openshift.com/) > Container platform on top of [Kubernetes](https://kubernetes.io/). For deployment of container based microservices.
 * [Occam](https://prima.corp.telenor.no/confluence/display/BHUB/Guideline+-+Using+OCCAM+Tool) > Inhouse tool for building, configure and deploy containers to OpenShift.

If you would like to contribute to the content in this module, create a pull request :)

### Learning by doing

In addition to being a resource for best practices and guidelines, this module is build as a step-by-step tutorial that every new developer
should go through to have a minimum fundament of knowledge before starting any real work on the BSS Hub platform. Having completed the tutorial,
the developer should have it reviewed by a members of the **[BSS Enablement team](https://prima.corp.telenor.no/confluence/display/MBSS/Enablement+squad)**. A member of the
platform team will review the code and ask some questions around the understanding of what the developer have created.

**BEFORE STARTING**

Each step in the tutorials have a list of **prerequisites**. These prerequisites are not listed there for fun.
It is assumed that you as a reader has basic understanding of the prerequisites. If you feel that you don't have
the knowledge, then use the resources listed or resources elsewhere to gain this knowledge. Having this knowledge
means you have the basic understanding of what you are doing when coding and why you are doing it :)

### Start the tutorial

#### Git

 1. **Phase 1: [Getting access to Git](docs/git/phase1.md)**.
 2. **Phase 2: [Understanding Bitbucket and Git](docs/git/phase2.md)**.
 3. **Phase 3: [Understanding branching and pull requests](docs/git/phase3.md)**.
 4. **Phase 4: [Understanding branching workflows](docs/git/phase4.md)**.

#### Basics

 1. **Phase 1: [Create your sandbox GIT repository](docs/basics/phase1.md)**.
 2. **Phase 2: [Setup Gradle for your project](docs/basics/phase2.md).**
 3. **Phase 3: [Import your Gradle project into an IDE](docs/basics/phase3.md)**.
 4. **Phase 4: [Setup your SpringBoot framework code](docs/basics/phase4.md)**.
 5. **Phase 5: [Code your webservice](docs/basics/phase5.md)**.
 6. **Phase 6: [Code your REST service](docs/basics/phase6.md)**.
 7. **Phase 7: [Turning on security](docs/basics/phase7.md)**.
 8. **Phase 8: [Write unit tests for your code](docs/basics/phase8.md)**.
 9. **Phase 9: [Write integration tests for your code](docs/basics/phase9.md)**.
 10. **Review: [Have your code reviewed :)](docs/review.md)**

#### Continuous Integration (CI)

 1. <sub><sup><sub><sup>Configuration</sup></sub></sup></sub> **Phase 1: [About Continuous Integration (CI)](docs/ci/phase1.md)**.
 2. <sub><sup><sub><sup>Configuration</sup></sub></sup></sub> **Phase 2: [BSS CI Pipeline setup](docs/ci/phase2.md)**.
 3. <sub><sup><sub><sup>Configuration</sup></sub></sup></sub> **Phase 3: [Before creating a Bitbucket Team/Project job](docs/ci/phase3.md)**.
 4. <sub><sup><sub><sup>Configuration</sup></sub></sup></sub> **Phase 4: [Creating a Bitbucket Team/Project job](docs/ci/phase4.md)**.
 5. <sub><sup><sub><sup>Developer</sup></sub></sup></sub> **Phase 5: [Your first Jenkinsfile](docs/ci/phase5.md)**.
 6. <sub><sup><sub><sup>Developer</sup></sub></sup></sub> **Phase 6: [When default pipeline is not enough](docs/ci/phase6.md)**.
 7. <sub><sup><sub><sup>Developer</sup></sub></sup></sub> **Review: [Have your code reviewed :)](docs/review.md)**

#### OpenShift

 1. **Phase 1: [Understanding the use of projects/namespaces in BSS Hub](docs/openshift/phase1.md)**.
 2. **Phase 2: [Understanding Occam](docs/openshift/phase2.md)**.
 2. Phase x: Configure, build and deploy your application to OpenShift.
 3. ...
 4. Phase 4: Have your code reviewed :)


#### Functional testing

 1. **Phase 1: [About Functional testing](docs/testing-functional/phase1.md)**.
 2. **Phase 2: [Writing a Webservice test with SoapUI](docs/testing-functional/phase2.md)**.
 3. **Phase 3: [Writing a REST test with SoapUI](docs/testing-functional/phase3.md)**.
 4. **Phase 4: [Running your SoapUI test from Gradle](docs/testing-functional/phase4.md)**.
 5. **Phase 5: [Parameterizing your tests](docs/testing-functional/phase5.md)**.
 6. **Phase 6: [Running test from Gradle with properties](docs/testing-functional/phase6.md)**.
 7. **Phase 7: [Move your tests to BSS Test Web](docs/testing-functional/phase7.md)**.
 8. **Review: [Have your code reviewed :)](docs/review.md)**