---
title:             "Portable Service Tester Configuration"
description:       "Making service tester configuration environment agnostic."
published:         true
categories:        [interlok, service-tester]
tags:              [interlok, service-tester]
author:            [mcwarman]
keywords:          "interlok, service-tester, gradle"
excerpt_separator: <!-- more -->
---

Sometimes it isn't obvious the changes that need to be made to service tester configuration so thats its portable across machines and build pipelines.
<!-- more -->

Firstly some basics: service tester gives developers the ability to unit test there Interlok config. A typical test case will have input message and expected output controlled by assertions. The test is executed against a "service to test" which is a piece of config prepared using preprocessors.

## Service to Test

A "service to test" can come from anywhere in your config.

### Config Paths

When referring to files (ex: adapter.xml) the service tester working directory property should be used, it's location is dynamic (and absolute) making the paths portable.

Example:  `file:///${service.tester.working.directory}/src/main/interlok/config/adapter.xml`

### Service Selection

Once the config is selected the easiest way to extract the service is to use the "Service Unique Id Preprocessor".

A service from in channel or a workflow:
```xml
<service-unique-id-preprocessor>
  <channel>channel</channel>
  <workflow>workflow</workflow>
  <service>my-service</service>
</service-unique-id-preprocessor>
```

A service from `shared-services`
```xml
<service-unique-id-preprocessor>
  <service>my-service-shared-service</service>
  <service>my-service-in-service-list</service>
</service-unique-id-preprocessor>
```

`<service>` can occur n times, for services inside service-lists inside service-lists...

### Applying Variables

Variables can be applied to the service, using a properties file or as key value pairs. The working directory variable is available to be used as the properties path, or as a part of the `value`:

```xml
<preprocessors>
  <variable-substitution-preprocessor>
    <property-file>file:///${service.tester.working.directory}/src/main/interlok/variables-shared.properties</property-file>
  </variable-substitution-preprocessor>
  <properties-variable-substitution-preprocessor>
    <properties>
      <key-value-pair>
        <key>adapter.transforms.dir.url</key>
        <value>file:///${service.tester.working.directory}/src/main/interlok/transforms</value>
        </key-value-pair>
    </properties>
  </properties-variable-substitution-preprocessor>
</preprocessors>
```

## Test Client

Test clients are the adapter runtime used to execute tests against.

Tests may work in the UI but not in else where (gradle), this is likely due to "Test Adapter Type", which are:

* **Embedded JMX Test client** will create a temporary adapter in the JVM to run the tests.
* **External JMX Test client** will use the specified adapter to run the tests.
* **Local JMX Test client** will use the UI local adapter to run the tests.

The UI by default selects "Local JMX", this should be swapped to use "Embedded JMX".

## Execution

Execution when using [interlok-build-parent]({% post_url 2019-11-13-parent-gradle-project %}) is simple:

```
gradle interlokServiceTest

> Task :interlokServiceTest
Running [test-list.test]
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 1.435 sec
```


