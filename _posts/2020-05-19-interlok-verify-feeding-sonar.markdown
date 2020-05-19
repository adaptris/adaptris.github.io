---
title:             "Interlok verify feeding sonar"
description:       "How to use shared-components in service tester"
published:         true
categories:        [interlok, build-parent, sonar]
tags:              [interlok, build-parent, sonar]
author:            [mcwarman]
keywords:          "interlok, build-parent, gradle, sonar"
excerpt_separator: <!-- more -->
billboard:         /billboards/icon-solutions.png
---

Since starting on our journey of maintaining a [build-parent]({% post_url 2019-11-13-parent-gradle-project %}) we've been constantly improving it, and now using a private fork of it in our internal build pipeline.

One rabbit hole I've recently travelled (which has since made it as an [improvement](https://github.com/adaptris-labs/interlok-build-parent/pull/15)) was trying to feed information from `interlokVerify` into [sonarqube](https://www.sonarqube.org/).

<!-- more -->

## TLDR

The end result is this:

![Sonar Violation Report]({{ site.baseurl }}/images/posts/sonar-interlok-hello-world.png)

To use it in a project add the following config:

```groovy
sonarqube {
  properties {
    property "sonar.projectKey", "<projectKey>"
    property "sonar.organization", "<organization>"
    property "sonar.host.url", "<sonar_host>"
    property "sonar.sourceEncoding", "UTF-8"
    property "sonar.sources", "./src/main/interlok"
    property "sonar.tests", "./src/test/interlok"
    property "sonar.externalIssuesReportPaths", interlokVerifySonarReport
  }
}
```

There are working examples using [sonarcloud.io](https://sonarcloud.io), found at [interlok-hello-world](https://github.com/adaptris-labs/interlok-hello-world/) and [interlok-twilio-sms](https://github.com/adaptris-labs/interlok-twilio-sms).

## The Journey

There are three parts to this journey:

* Sonarqube and getting information into it
* Interlok Verify and getting information out of it
* Gluing the two together

### Sonarqube

Sonarqube is a code quality and security monitoring tool, it provides mechanisms of performing and reporting on static code analysis, as well as taking inputs from other analysis tools.

It support the ability to provide issues in its [generic issue import format](https://docs.sonarqube.org/latest/analysis/generic-issue/). This allowed us to create our own report and use it as input to sonar using the `sonar.externalIssuesReportPaths` property.

### Interlok Verify

One simple yet powerful step that runs as a part of the `check` tasks in the build parent, is `interlokVerify` under the covers it runs the gradle equivalent of the following:

```shell
java -jar ./lib/interlok-boot.jar -configtest
```

Config test attempts to unmarshal the config based on `bootstrap.properties` and executes the component `init()` and `prepare()` for the unmarshaled config.

An output taken from [interlok-hello-world](https://github.com/adaptris-labs/interlok-hello-world) looks something like this:

```text
Bootstrap of Interlok 3.10-SNAPSHOT(2020-05-17:04:02:24 UTC) complete
INFO  [main] [com.adaptris.core.management.UnifiedBootstrap}] Adapter created
WARN  [main] [com.adaptris.core.util.LoggingHelper}] [PayloadFromMetadataService(set-payload)] is a payload-from-metadata-service; use payload-from-template or metadata-to-payload instead.
WARN  [main] [com.adaptris.core.Adapter}] [Adapter(hello-world)] has a MessageErrorHandler with no behaviour; messages may be discarded upon exception
DEBUG [main] [com.adaptris.core.Adapter}] FailedMessageRetrier []
Config check only; terminating
```

A development team practise is adding logging warning during a components initialisation when its deprecated, or to configuration that may cause unexpected behavior, as seen above.

With a combination of redirecting the output from `interlokVerify`:

```groovy
def interlokVerify = tasks.register("interlokVerify", JavaExec) {
  // ...
  systemProperty "interlok.logging.url", verifyLog4j
  standardOutput = new ByteArrayOutputStream()
  doLast {
    def verifyOutput = standardOutput.toString()
    mkdir(reportFile.getParentFile())
    reportFile.withOutputStream { stream ->
      standardOutput.writeTo(stream)
    }
  }
}
```

 and `log4j2.xml` filtering, which is referred to using the `interlok.logging.url` system property:

 ```xml
<Loggers>
  <Logger name="org" level="OFF"/>
  <Logger name="io" level="OFF"/>
  <Logger name="com" level="OFF"/>
  <Logger name="net" level="OFF"/>
  <Logger name="jndi" level="OFF"/>
  <Logger name="java" level="OFF"/>
  <Logger name="com.adaptris" level="WARN"/>
  <Root level="OFF">
   <AppenderRef ref="Console"/>
  </Root>
</Loggers>
```

We end up with a `report.txt` with the warnings that config test reported:

```text
[PayloadFromMetadataService(set-payload)] is a payload-from-metadata-service; use payload-from-template or metadata-to-payload instead.
[Adapter(hello-world)] has a MessageErrorHandler with no behaviour; messages may be discarded upon exception
```

## Gluing the two together

After extracting the information from `interlokVerify` and knowing it was possible to import a "generic issue report".

The next step was creating the report this was done by writing a java command line app: [interlok-verify-report](https://github.com/adaptris-labs/interlok-verify-report).

Which takes the `report.txt` and outputs a the "generic issue format":

```json
{
  "issues" : [ {
    "engineId" : "interlokVerify",
    "ruleId" : "rule1",
    "severity" : "INFO",
    "type" : "CODE_SMELL",
    "primaryLocation" : {
      "message" : "[PayloadFromMetadataService(set-payload)] is a payload-from-metadata-service; use payload-from-template or metadata-to-payload instead."
    }
  }]
}
```

Executing is done as a part of the build parent:

```groovy
def interlokVerifyReport = tasks.register("interlokVerifyReport", JavaExec) {
    group = 'Test'
    description = 'Create Interlok verify test reports'
    onlyIf{
      interlokVerify.get().didWork
    }
    main = 'com.adaptris.labs.verify.CreateVerifyReport'
    classpath = configurations.interlokVerifyReport
    args "--reportFile"
    args interlokVerifyTextReport
    args "--outputFile"
    args interlokVerifySonarReport
}
```

## The Future

At the moment all of the violations are set to `INFO` this is partly due to the fact we're relying on logging output to produce the report. What we've now realised is the potential for further improvements of `--configtest` and making it more aware of annotations and outputting a report itself to move away from log manipulation. This would open up the potential for increasing the severity.
