---
title:             "build.gradle parent for Interlok"
description:       "Import our build parent to simplify your build.gradle"
published:         true
categories:        [interlok]
tags:              [interlok]
author:            [mcwarman, quotidian-ennui]
billboard: /billboards/icon-iot.png
excerpt_separator: <!-- more -->
---

We've standardised on [gradle](https://gradle.org) as our build tool of choice when developing solutions with Interlok. Since it's possible to end up down a rabbit warren of scripting with gradle; maintaining the build script is one of key considerations, while also providing what we consider best practise when working with Interlok. As a result, we have an early preview release of our proposed [parent gradle file](https://github.com/adaptris-labs/interlok-build-parent) that you can start using when developing Interlok based projects.

<!-- more -->

## Drivers

The primary drivers for the parent gradle were:

* Keep the child simple; so that people who aren't developers can use it (within reason).
* Hosted publicly so that any changes can be pushed out 
* Incorporate service-tester as a first class project member.
* Being able to customise your build output depending on the environment.
* Platform neutral (windows/mac/linux should work the same).

## Usage

The usage itself is hopefully self-explanatory, your build.gradle file simply has to contain

```
ext {
  interlokVersion = '3.9.2-RELEASE'
  interlokUiVersion = interlokVersion
  interlokParentGradle = "https://raw.githubusercontent.com/adaptris-labs/interlok-build-parent/master/build.gradle"
}

configurations {
    interlokRuntime{}
    interlokTestRuntime{}
    interlokJavadocs{}
}


dependencies {
    interlokRuntime ("com.adaptris:interlok-json:$interlokVersion") { changing=true }
    interlokRuntime ("com.adaptris:interlok-filesystem:$interlokVersion") { changing=true }
    interlokRuntime ("com.adaptris:interlok-csv-json:$interlokVersion") { changing=true }
}

allprojects {
    apply from: "${interlokParentGradle}"
}
```

* `interlokRuntime` configuration is where we declare all the dependencies for running interlok (the minimum dependencies are declared in the parent)
* `interlokTestRuntime` configuration is where we declare all the dependencies for running service-test; generally you won't change this unless you want to have custom JSON assertions and the like.
* `interlokJavadocs` configuration is where we declare all the javadocs that we want in the distribution.

Because we have custom closures that resolve dependencies; you need to apply from the parent *after all your configurations / dependencies are declared* otherwise you get an error around attempting to change configurations after they have been resolved.


## It's a preview...

It's a preview so there are a few gotchas which we hope to be able to iron out (eventually); The build parent applies the distribution plugin since it doesn't really need anothing else. If you want a full example (including service-tester/custom jars) then have a look at [build-parent-json-csv](https://github.com/adaptris-labs/build-parent-json-csv).

### Customising your output

* _interlokDistDirectory_ will control where the files are emitted; this is initially set to `build/distribution`

The parent gradle also assumes that you essentially have 2 property files that contain all the variables required for your project:

* `variables.properties` - which will generally contain all the _shared_ properties that aren't environment specific (path locations for instance)
* `variables-local-{buildEnv}.properties` which contains specific environment entries (e.g. specific hostnames etc)

You can control the environment via a standard gradle project property: `./gradlew -PbuildEnv=dev build`. What this effectively does is to select `variables-local-dev.properties` and copies it to `variables-local.properties` inside your distribution config dir; if it exists. Since our build environment is _dev_ in this instance, it also copies all the service-tester jars into the lib directory so it is ready to use within the UI.

### Custom tasks

There are custom _JavaExec_ tasks defined that allow us to execute service-tester, a version report, and configtest. They simply build up the classpath and execute the appropriate _main_ class. Since Windows has a fixed limit on your commandline; it builds up the required classpath by using a temporary jar file that declares the classpath using the _Classpath_ manifest entry. 

Have a log4j2.xml inside your _src/test/resources_ directory so you can see the output from the service-tester execution. If you want to mimic the standardised output from running ant+junit then you can use something like this: 

```
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="warn" monitorInterval="60" shutdownHook="disable">
  <Appenders>

    <Console name="Console" target="SYSTEM_OUT">
      <PatternLayout>
        <Pattern>%m%n</Pattern>
      </PatternLayout>
    </Console>

  </Appenders>
  <Loggers>

    <Logger name="org" level="WARN"/>
    <Logger name="io" level="WARN"/>
    <Logger name="com" level="WARN"/>
    <Logger name="net" level="WARN"/>
    <Logger name="jndi" level="WARN"/>
    <Logger name="java.sql.DatabaseMetadata" level="WARN"/>
    <Logger name="org.eclipse.jetty" level="WARN"/>
    <Logger name="org.hibernate.cache" level="WARN"/>
    <Logger name="org.springframework.aop.framework.CglibAopProxy" level="ERROR"/>
    <Logger name="org.hibernate.cache.EhCacheProvider" level="ERROR"/>
    <Logger name="com.mchange.v2.resourcepool.BasicResourcePool" level="ERROR"/>
    <Logger name="com.sun.jersey" level="ERROR"/>
    <Logger name="com.adaptris" level="OFF"/>
    <Logger name="com.adaptris.adaptergui.util.ObjectConverter" level="OFF"/>
    <Logger name="com.adaptris.adaptergui.util.ClassUtils" level="OFF"/>
    <Logger name="com.adaptris.tester.runtime.Test" level="INFO" />
    <Root level="WARN">
      <AppenderRef ref="Console"/>
    </Root>
  </Loggers>
</Configuration>
```

### Service Tester may not execute

Service Tester may not execute if you have overriden `interlokDistDirectory` as a gradle property. This is because there are a couple of expectations around relative paths.

### build/assemble will delete stuff

Since assemble is considered a `sync` task; this will delete all files inside your targetted distribution directory. If you want to preserve your settings, then you should include a custom _src/main/interlok/ui-resources/interlokuidb.properties_ that specifies your database location (e.g. using PostgresSQL). __All changes you make to build/distribution will be deleted the next time you execute assemble__

### What's next

* A example that also has custom java components..
* Although it _should support_ multi-module projects this hasn't been tested extensively.
* An actual gradle plugin?

