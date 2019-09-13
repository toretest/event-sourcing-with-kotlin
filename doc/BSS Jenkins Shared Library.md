## BSS Jenkins Shared Library

In order to keep **Jenkinsfile**s content to a bare minimum, this shared library is developed and configured
on Jenkins as a Global Shared Library. It provides both complete **pipelines**, **scripts** and **classes**.
All are documented below. Please contribute to the Library instead of all projects on the platform coding
their own Jenkinsfiles with duplicate code in all projects! Share your knowledge and dont repeat yourself!

## Pipelines

The library contains some complete pipelines which can be included in your **Jenkinsfile**. These pipelines are
meant as default pipelines with default values for all supported features.

### ciPipeline
---

The default CI pipeline on the platform supports those projects that have developed their services according
to the recommendations established on the platform. Using this predefined pipeline, will do the following:

 * Performs a **Build** and **Test** using either **Gradle (default)** or **Maven**.
 * During test, the task **testRegression** is called (from the **BSS Base Gradle plugin**)
 * If project uses **BSS SoapUI plugin**, a local server can be started and SoapUI tests will be executed towards it.
 * If success, mail and slack notifications are sent (default to culprits on status FIXED)
 * If failure, mail and slack notifications are sent (default to culprits on status FAILED or STILL FAILING)
 * All junit reports are scanned.

**Jenkinsfile**: *Will run the CI pipeline with default setup*
```groovy
@Library('bss-jenkins-shared-libs') _

ciPipeline {
}
```

**Jenkinsfile**: *Showing the CI pipeline with all possible configuration options (and the default values)*
```groovy
@Library('bss-jenkins-shared-libs') import com.telenor.bss.jenkins.*

// General: Below shows all possible configuration options. You may toggle features as you see fit using Groovys named parameter feature.
ciPipeline {

    // Agent label to build with (used for whole pipeline)
    agent = new AgentConfig(label: 'jdk8')
    
    // Gradle tool config. Gradle is the default build tool.
    gradle = new GradleConfig(wrapper: true, buildTask: 'clean testClasses', buildArgs: '', testTask: 'testRegression', testArgs: '')
    
    // Maven tool config. To use maven as build tool you set this config.
    maven = new MavenConfig(wrapper: true, buildPhase: 'clean install', buildArgs: '-Dmaven.test.skip=true', testPhase: 'test', testArgs: '')
    
    // Test server config. Set this to have a server started during testing (default is null). Either because you need it in your tests or that you simply want to verify that server is started OK.
    // NOTE: Supported with the setup Gradle and Spring Boot as of now.
    server = new ServerConfig(protocol: 'http', host: 'localhost', port: 8080)
    
    // JUnit result config. Pipeline will default fail if no test results are found. You may override this behaviour and give a 'justification'.
    junit: new JUnitConfig(testResults: 'build/test-results/**/TEST-*.xml', allowEmptyResults: false, justification: null)
    
    // Email setup: Default email will be sent to culprits on status FIXED, FAILED, STILL FAILING. 'alwaysRecipients' is comma separated list of emails to always send mail to (regardless of build status)
    emailConfig = new EmailConfig(alwaysRecipients: null)
    
    // Slack setup: Default sent messages to slack on status FIXED, FAILED, STILL FAILING. Default channel is '#bsshub_ci'.
    slack = new SlackConfig(channel: '#bsshub_ci')
    
}
```

## Scripts

Scripts are provided to gather common operations needed in pipelines so that projects do not have to duplicate
Jenkinsfile content all around. These scripts are used in the **Pipelines** provided by this module. If for
some reason it is decided to implement a custom pipeline in Jenkinsfile, these scripts can (and should) be
used in those cases. You will recognize that these scripts corresponds to the *Config classes used in
**pipelines**. The Config classes are described below.

### gradle
---

All Gradle related operations is gathered in this script. The following operations are available:

#### build

Executes a build.

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| config | GradleConfig | new GradleConfig() | Execute a gradle build given the config object |

**Examples**
```groovy
gradle.build() // Default behaviour

gradle.build(new GradleConfig(wrapper: false)) // Run with installed gradle and not wrapper

gradle.build(new GradleConfig(buildTask: 'clean mytask build', buildArgs: '-Dsome=thing')) // Run with custom build task(s) and args
```

#### test

Executes tests.

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| config | GradleConfig | new GradleConfig() | Execute a gradle test given the config object |
| serverConfig | ServerConfig | null | Start a server prior to running tests |

**Examples**
```groovy
gradle.test() // Default behaviour (runs the 'testRegression' task from BSS Base plugin)

gradle.test(new GradleConfig(wrapper: false)) // Run with installed gradle and not wrapper

gradle.test(new GradleConfig(testTask: 'test', testArgs: '-Dsome=thing')) // Run with custom test task(s) and args

gradle.test(new GradleConfig(), new ServerConfig()) // Start a server prior to running tests

gradle.test(new GradleConfig(), new ServerConfig(protocol: 'https', host: 'localhost', port: 8090)) // Start a server prior to running tests with custom protocol, host and/or port.
```

#### bootRun

Starts SpringBoot. The SpringBoot PID is stored in the file **./application.pid**. The operation
will wait for SpringBoot to be up and running by calling **/actuator/health** resource. If startup
is not achieved within a certain amount of time, the operation will fail.

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| config | GradleConfig | new GradleConfig() | Start the server using this Gradle config |
| serverConfig | ServerConfig | null | Server configuration (null will have no effect) |

**Examples**
```groovy
gradle.bootRun(gradleConfig, new ServerConfig()) // Will start the server using the given gradleConfig and default server config

gradle.bootRun(gradleConfig, new ServerConfig(protocol: 'https', host: 'localhost', port: 8090)) // Will start the server using the given gradleConfig and the given server config
```

#### bootRunShutdown

Shuts down SpringBoot by killing the PID from **./application.pid**

**Examples**
```groovy
gradle.bootRunShutdown() // Kills the server PID
```

#### bootRunDumpLogs

Dumps server logs to console.

**Example**
```groovy
gradle.bootRunDumpLogs()
```

### maven
---

All Maven related operations is gathered in this script. The following operations are available:

#### build

Executes a build.

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| config | MavenConfig | new MavenConfig() | Execute a maven build given the config object |

**Examples**
```groovy
maven.build() // Default behaviour

maven.build(new MavenConfig(wrapper: false)) // Run with installed maven and not wrapper

maven.build(new MavenConfig(buildPhase: 'clean install', buildArgs: '-Dsome=thing')) // Run with custom build phase(s) and args
```

#### test

Executes tests.

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| config | MavenConfig | new MavenConfig() | Execute a maven test given the config object |

**Examples**
```groovy
maven.test() // Default behaviour

maven.test(new MavenConfig(wrapper: false)) // Run with installed maven and not wrapper

maven.test(new MavenConfig(testPhase: 'test', testArgs: '-Dsome=thing')) // Run with custom test phase(s) and args
```

### email
---

All Mail related operations is gathered in this script. The script uses the **emailext** Jenkins plugin.
The following **recipientProviders** (from the **emailext** plugin) is setup: **CulpritsRecipientProvider**, **RequesterRecipientProvider**.
The following operations are available:

#### onSuccess

Email on success. Default behaviour is send emails on FIXED build status. Not on every SUCCESS (to avoid spamming).

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| config | EmailConfig | new EmailConfig() | Send email to culprits on FIXED builds and with the given config|

**Examples**
```groovy
email.onSuccess() // Default behaviour

email.onSuccess(new EmailConfig(alwaysRecipients: 'some@body.com')) // Send email to culprits on FIXED builds and all statuses to 'some@body.com' 
```

#### onFailure

Email on failure.

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| config | EmailConfig | new EmailConfig() | Send email to culprits on FAILED and STILL FAILING builds and with the given config|

**Examples**
```groovy
email.onFailure() // Default behaviour

email.onFailure(new EmailConfig(alwaysRecipients: 'some@body.com')) // Send email to culprits on FAILED/STILL FAILING builds and all statuses to 'some@body.com'
```

### slack
---

All Slack related operations is gathered in this script. The script uses the **Slack Notification** Jenkins plugin.

#### onSuccess

Slack on success. Only FIXED build status is sent to avoid spamming.

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| config | SlackConfig | new SlackConfig() | Send slack message given the config |

**Examples**
```groovy
slack.onSuccess() // Default behaviour

slack.onSuccess(new SlackConfig(channel: '#our_own_channel')) // Send slack notification to specified channel
```

#### onFailure

Slack on failure.

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| config | SlackConfig | new SlackConfig() | Send slack message given the config |

**Examples**
```groovy
slack.onFailure() // Default behaviour

slack.onFailure(new SlackConfig(channel: '#our_own_channel')) // Send slack notification to specified channel
```

## Classes

Classes defined by library, used in both **pipelines** and **scripts**.

### Package: com.telenor.bss.jenkins

#### AgentConfig

Used in **pipelines** to define agent info.

| Constructor parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| label | String | 'jdk8' | Agent label to use in pipeline. Must be available as agent in the Jenkins env. |

#### EmailConfig

Used in **pipelines** and **scripts** to control email behaviour.

| Constructor parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| alwaysRecipients | String | null | Comma separated list of email addresses to always send email to |

#### GradleConfig

Used in **pipelines** and **scripts** to control gradle behaviour.

| Constructor parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| wrapper | boolean | true | Whether or not to use Gradle Wrapper or installed Gradle |
| buildTask | String | 'clean testClasses' | Gradle task(s) to run during build |
| buildArgs | String | '' | Arguments to use in during build operation |
| testTask | String | 'testRegression' | Gradle task(s) to run during test |
| testArgs | String | '' | Arguments to use in during test operation |

#### JUnitConfig

Used in **pipelines** to control junit reports behaviour.

| Constructor parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| testResults | String | 'build/test-results/**/TEST-*.xml' | Where to search for junit test results |
| allowEmptyResults | boolean | false | Whether or not to accept empty test results |
| justification | String | null | Of allowing empty results, use this to justify the reason (ref to JIRA issue for developing missing tests is a good justification! |

#### MavenConfig

Used in **pipelines** and **scripts** to control maven behaviour.

| Constructor parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| wrapper | boolean | true | Whether or not to use Maven Wrapper or installed Maven |
| buildPhase | String | 'clean install' | Maven phase(s) to run during build |
| buildArgs | String | '-Dmaven.test.skip=true' | Arguments to use in during build operation |
| testPhase | String | 'test' | Maven phase(s) to run during test |
| testArgs | String | '' | Arguments to use in during test operation |

#### ServerConfig

Used in **pipelines** and **scripts** to control test server behaviour.

| Constructor parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| protocol | String | 'http' | Test server protocol to use |
| host | String | 'localhost' | Test server host to use |
| port | int | 8080 | Test server port to use |

#### SlackConfig

Used in **pipelines** and **scripts** to control slack behaviour.

| Constructor parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| channel | String | '#bsshub_ci' | Channel to send slack notifications to. Null will disable notications. |