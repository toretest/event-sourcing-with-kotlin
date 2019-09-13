## Phase 1: Understanding the use of projects/namespaces in BSS Hub

[Home](../../README.md) | Next: [Phase 2](phase2.md)

### Prerequisites

* GitBash [installed](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git) locally.
* OpenShift Client (oc) [installed](https://prima.corp.telenor.no/confluence/display/DCHUB/OpenShift+Command+Line+Tools+oc+client+tool) locally.

### Before doing any coding

Now that you have done the **Basic** tutorial and have had that reviewed and merged into your master branch,
create a new feature branch (**feature/tutorial-openshift**) and commit and push your code to that branch for
all code in this tutorial.

### Understand how namespaces are used on BSS Hub

A namespace in OpenShift/Kubernetes is a way to group related resources together, providing several important features:

* Access control to the resources. This will cover all of the resources contained in the namespace, from application definitions, route definitions, to administering Pods running within the namespace. Each squad is represented by a group in BSS Hub, and these groups are used to give squads read and write access to namespaces.
* Traffic control. Namespaces defines which resources in the cluster are allowed to communicate with each other on the IP-level. By default, all communication between different namespaces is disallowed. This can be opened as needed by changing the Netnamespace-definitions (requires cluster admin-access).
* Resource quotas and limitations. Through the use of Quotas, it is possible to limit resource usage within the namespace to prevent different application groups to consume too much resources and cause problems for other application groups. All namespace are set up with a default quota, which can be adjusted (requires cluter admin-access)

For each application group, there will exist several different namespaces in BSS Hub test and production environments. Namespaces are used to simulate separate test environments, which will all run in the same cluster BUT be separate due to the features described above. We also use a separate namespace for building the application artifacts (container image), to prepare the image for deployment to production, and of course a namespace in production to run the workload. Typical suffixes used for these environments are sit, test, nasse, brumm, tussi, with ci and prod-ready for handling building and promotion.

As an example, for an application group with the name "myapplication", with requirement for separate nasse, tussi and brumm test environments, the following namespaces will be created:

* In prod cluster
  * myapplication - for production deployment of applications
* In test cluster (with test environments nasse, tussi, brumm)
  * myapplication-ci - for building of container images, and logically storing the built image
  * myapplication-nasse
  * myapplication-tussi
  * myapplication-brumm
  * myapplication-prod-ready (for promotion of container images to production)

These namespaces are created and configured automatically by the project setup scripts.

### Take a look at the namespaces available to you in BSS Hub test

Due to some legacy from development of OpenShift, the resource types Namespace and Project is used interchangeably. They are kept in sync automatically, but there are still a few commands using Project both in the Admin GUI and command line interface.

To see all namespaces/projects visible to you:

````
oc login https://test-bsshub.corp.telenor.no:8443
oc projects
````

To set context of the OC-command to point to a given project or show current context:

````
oc project <project name>
oc project
````

All project related namespaces with owner, editors and viewers can be seen here:
<https://namespaces.test-bss.corp.telenor.no>

### Ordering application group namespaces in OpenShift (BSSHub test and prod)

You order new namespaces by registering av Jira-issue with the Enablement team, in project "BHUB". For setting up, the enablement team will need to know which test environments you need, as well as which team will be the owner of those namespaces.

##### At this point you should be able to:

* Explain what a namespace in OpenShift is, and how it is used in BSS Hub
* Explain in which namespace an application container image is built for an application
* Understand what different suffixes for namespace names mean
* Commands to login to OpenShift, show namespaces, select and show selected namespace
* Understand how to order a set of namespaces for an application group

[Home](../../README.md) | Next: [Phase 2](phase2.md)

## Phase 2: Understanding Occam

[Home](../../README.md) | Previous: [Phase 1](phase1.md) | Next: [Phase 3](phase3.md)

### Prerequisites

To fully understand the concepts below, a basic understanding of OpenShift/Kubernetes concepts from the following articles will help

* [OpenShift Builds and Image streams](https://docs.openshift.com/container-platform/3.11/architecture/core_concepts/builds_and_image_streams.html)
* [OpenShift Core Concepts](https://docs.openshift.com/container-platform/3.11/architecture/core_concepts/index.html)
* [OpenShift Routes](https://docs.openshift.com/container-platform/3.11/architecture/networking/routes.html)
* [BSS Naming standard](https://prima.corp.telenor.no/confluence/display/BHUB/Naming+Guidelines)

### Using occam to define, set up builds and deploying applications

Telenor has created a tool for helping developers to define, build and deploy their applications on DC Hub and BSS Hub. The tools was originally created for DC Hub, but has been adopted by BSS Hub as well to provide a common experience for developers.

Occam aims to hide much of the complexity and tedious work involved in defining how to build container images and writing the resource manifests needed to get the application up and running based on this image. It runs as a service in the cluster (one for each cluster), and has a simple CLI for interaction.

Occam tries to support the whole lifecycle of an application:

* Build container image. Occam provides multiple strategies for building applications, including OpenShift Source2Image and integration with Jenkins for defining build pipelines suitable for different kinds of applications.
* Set up deployment in various test environments. Occam provide an easy way to specify environment-specific values, and keep definitions specific to different environments separate.
* Promote and copy container image to production. Container image is tagged into -prod-ready-namespace in test for OCcam in production to know where to fetch it.
* Set up deployment in production. Deployment to production is done in same way as for test environments, except for a few more safety checks.

### Project file structure when using Occam

The input for each of these phases is a spec.json definition file, explained below.

When using Occam, the standard is to check in the spec.json-files as part of the application source repository in Git, using the following directory structure:

```sh
my-project-root/openshift/prod/spec.json
my-project-root/openshift/prod/init.sh
my-project-root/openshift/test/<environment>/spec.json
my-project-root/openshift/test/<environment>/init.sh
```

The Occam CLI will expect to find this structure, with openshift as its base directory. It also uses the presence of "test" and "prod" in the directory path below openshift to decide if the operations should be run against the test or prod instance of Occam. In addition, the init.sh script file is used to bootstrap the CLI, by validating the spec.json and downloading the actual CLI script.

You can have various structures for the openshift-hierarchy as long as you follow these base rules.
Examples:

```sh
# Prod specs for both DC and BSS Hub
my-project-root/openshift/prod/bss/spec.json
my-project-root/openshift/prod/dchub/spec.json

# Test specs for different namespaces in test
my-project-root/openshift/test/sit/spec.json
my-project-root/openshift/test/nasse/spec.json
my-project-root/openshift/test/brumm/spec.json

# Build only spec (we will look at this later)
my-project-root/openshift/test/ci/spec.json
```

### Understanding the spec.json

Occam provide a way to help you create your spec.json-file available here:

http://bsshub-occam.test-bss.corp.telenor.no/dchub/occam/specdoc

For the following descriptions, please refer to the example spec.json provided [here](../../openshift/test/occam-simple/spec.json).

Sections of the spec.json

* parameters
* parent?
* application
  * name
  * build
    * parent?
  * deployment
    * parent?
  * promotion

#### Parent specs

Parent specs can be defined for the application section as a whole, or for each of the build an deployment sections individually. When processing the spec.json, the first step Occam will do is populating its datamodel based on values from the parents specs. This helps keep the overall size of project specific specs low, as well as making it easy to provide default values for elements that normally should not be define. All values in spec.json itself can be seen as overriding the values already provided in base/parent specs.

To see the full spec after pre-processing, run the following command:
```sh
./init.sh
./occ showSpec
```

#### Parameters and spec-variables

A *variable* defined under parameters will be made available in the rest of the spec as ${variable}. This is useful for handling extra values that is useful when defining parameters of the spec. The standard fields themselves are made available in the same manner, and can be used across the whole spec.json. 

In the block below, the name of the config map will be constructed using the application name and a suffix. Other fields can be used in the same way.

```json
"configMaps": [
        {
          "id": "properties-files",
          "name": "${application.name}-config",
          "mountPath": "/app/config",
          "contentFromSource": [
            "src/main/resources/application.properties",
            "src/main/resources/application-sit.properties",
            "src/main/resources/logback-spring.xml"
          ]
        }
```

#### Application

This is the main section of the spec, defining values that should cover both the underlying build and deplyment sections.

Fields:

* name: this is the name of the application, and the value will be used for a lot of purposes both for the build process and when generating manifests for OpenShift. It is mandatory to follow the naming standard of BSS Hub when defining the name. The name must also correspond to the name of the Git repository for the application. The naming standard can be found here [Naming standard](https://prima.corp.telenor.no/confluence/display/BHUB/Naming+Guidelines).

#### Build

This section defines how the application is built. The type of build is defined through the use of parent spec and the value of the ciType-field. This will be described in [phase 3](phase3.md).

Fields:

* namespace. Defines which namespace should be used for the build and where the resulting container image is placed. There will be created an ImageStream named ${application.name} in this namespace.
* tag. Defines which tag should be used when tagging container images during build process
* builder. Defines extra parameters related to the build. The actual values needed will vary based on the type of build used.
  * environment. Arbitrary id/name/value-pairs that can be used in the build process. NOTE! Both name and id is mandatory if the id has not been defined in a parent spec, or the whole pair will be disregarded. Otherwise, name/value is sufficient.
* git. Defines where to get source code for the build
  * url. Full URL to the Git repository
  * ref. Name of the Git branch to use (master is most common)

#### Deployment

The Deployment section describes how Occam should produce resource manifest definitions for Kubernetes/OpenShift to deploy the application.

Fields:

* namespace. Defines which namespace should be used for running the application. All manifests will be generated in this namespace.
* tag. Defined which tagged version of the container image should be run.
* Memory / memoryMax: The request and limit for memory.
* Cpu / cpuMax: The request and limit for cpu. Note: Usually set way too high - please load-test application to find suitable values..

There are aleso several sub-sections providing important features for your application:

##### Autoscaler

The HorisontalPodAutoscaler in Kubernetes provide a way to scale up and down the number of running Pods (application instances) based on the actual resource usage (primarily cpu).

Here it is set to scale from min 1 to max 1, and perform up and downscale based on cpu usage percentage of 80%. This means using average 80% of the requested CPU for a 1 minute period will mean scaling up, while shrinking below
this average for the number of pods currently scaled will make it scale down.

##### Metrics

Settings for exposing a metrics endpoint for the application. This will result in the specified port and path being exposed as a service, and the service annotated with "prometheus.io/path", "prometheus.io/port", "prometheus.io/scrape:true". This allows the prometheus-instance running in the same namespace to automatically find and scrape metris for the application.

##### ConfigMaps

Specifies a set of files that should be put into a ConfigMap, and this ConfigMap added as a mounted volume in the application definition (deployment config). The files will be available in the application pods as separate files under the specified mount point.

Name is used for creating the name of the ConfigMap resource in OpenShift, and also used when referencing this
ConfigMap in the DeploymentConfig. If you change this name after doing a deployment, please ensure old resources
are cleaned up (manual steps).

##### Secrets

Specifies a set of predefined secrets in the namespace that should be mounted into the pods at runtime. The secrets must exist and will have to be managed outside Occam, this definition only causes them to be part of the deployment config.

##### Environment

The elements in environment will be added as environment variables in the running pod. Semantics of the entries are entirely up to the container/application, but some base images have predefined behaviour related to environment variables.

Note: Full pattern for entries are id/name/value. When overriding entries defined in parent-specs, name is omitted. If you define a new element not defined in parent specs, remember to use both id and name for it to be valid.

##### Endpoints

A endpoint definition will cause Occam to produce both Service and Route.

Based on the given list, there will be created one Service exposing each containerPort mentioned.
If a hostname is given for an endpoint, Occam will also generate a corresponding route.

For the values given in the example spec.json the following will be generated:

* a Service named "bss-webservice-star" exposing port 8080
* a Route with hostname "star-bsshub-demo-sit.test-bss.corp.telenor.no using https pointing to the service above

All routes using subdomains below _test-bss.corp.telenor.no_ and _bss.corp.telenor.no_ will be automtically resolved
through DNS, and will also have a valid certificate for https. This is handled through wildcard DNS entries and 
wildcard TLS certificates registered in the clusters.

Each spec.json can have several endpoints pointing to the same/different ports in the application.

##### LivenessProbe

Kubernetes can use liveness probes to check if a container is ok, and automatically restart it if needed. The
example config asks for this to be done on port 8080 and with a given http path. It will start probing after
60 seconds, do it every 20 seconds, and wait max 10 seconds before timing out.

Failing the liveness probe means the application is unhealthy, and Kubernetes will restart it.

##### ReadinessProbe

A readiness probe works the same way as a liveness probe, except it controls the flow of requests to the application pods through the corresponding Service. The Service will have a companion resource called Endpoints
used for looking up pods available to process requests. As long as a pod fails the readiness probe, it will be marked as unvailable in the Endpoints, and Service will not route traffic to it. When readiness probe is
successful, the Endoint is updated accordingly, and requests coming into the servive will be routed to the pod.

In this example, we use the same endpoint for both probes. This is ok, as long as it is really suitable for both. A more elegant solution will be to have a readiness-endpoint that actually monitors performance of the pod, and only accepts traffic when it will be able to fullfil SLA requirements. This is left to a future exercise.

#### Promotion

Promotion is used to tag an image for production deployment. This is done by coping the image to an image stream in a special namespace in test with suffix "-prod-ready". The values in the promotion-section specifies the name of the namespace to be used
to promote, and the tag to be used for the image when copied to that namespace.

In the example spec.json, the image specified under the Deployment-section with namespace, name and tag will be used as a source
when promoting. This image will be copied to namespace _bsshub-demo-prod-ready_ and tagged with _prod_.

When deploying to production, this step must be performed in test-environment first. The rest of the deployment process will be
done in prod-environment, and will expect the image to exist in this namespace to be copied to prod. If you forget to promote from test, the importProd command part of production deployment
will copy the previous image, and not the one you would actually like to deploy to production.

##### At this point you should be able to:

* What Occam is, and how it supports the development process
* Describe the directory structure expected by Occam
* Understand the structure of the Occam spec.json files
* Understand how images are promoted from test to production

[Home](../../README.md) | Previous: [Phase 1](phase1.md) | Next: [Phase 3](phase3.md)