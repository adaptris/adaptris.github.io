---
layout: post
title: "Using mapped diagnostic contexts for better logging"
author: quotidian-ennui
# date: 2018-01-15 11:00
comments: false
tags: [interlok, log4j]
categories: [interlok, log4j]
published: true
description: "Adding the message-id to all the logging"
keywords: "interlok"
billboard: /billboards/icon-real-time-reporting.png
excerpt_separator: <!-- more -->
---

Even in production, it can be very useful to have the message unique-id displayed on every log entry that appears in the log file. If the adapter has many workflows, and is processing a large number of messages; it can be hard to see the wood for the trees. Log4j2 supports mapped diagnostic contexts; and Interlok can interact with that via services or a [logging-context-workflow-interceptor][].

<!-- more -->

Since diagnostic contexts are thread based, it is probably better to use the service based contexts rather than the interceptor.

If you want to see the message-id in all the log entries; then what you want to do is to insert the current message-id as a context value against a specific key[^1]; do your processing as normal; and then remove the context value afterwards:

```xml
<service-collection class="service-list">
  <services>
    <add-metadata-service>
      <metadata-element>
        <key>myUniqueId</key>
        <value>$UNIQUE_ID$</value>
      </metadata-element>
    </add-metadata-service>
    <add-logging-context-service>
      <key>myLoggingContext</key>
      <value>%message{myUniqueId}</value>
    </add-logging-context-service>
    ... Now do stuff
    <remove-logging-context-service>
      <key>myLoggingContext</key>
    </remove-logging-context-service>
  </services>
</service-collection>
```

Then in your log4j2.xml you just use the `%X` syntax to include your mapped diagnostic key as part of your logging:

```xml
<Console name="Console" target="SYSTEM_OUT">
  <PatternLayout>
    <Pattern>%d{ISO8601} %-5p [%t] [%c{1}.] [%X{myLoggingContext}] %m%n</Pattern>
  </PatternLayout>
</Console>
```

Which leads to the output

```
DEBUG [ProductLookup] [c.a.c.ServiceList] [] Executing doService on [AddLoggingContext]
DEBUG [ProductLookup] [c.a.c.ServiceList] [8a13eb5a-c3b3-4557-882b-f024fc253c10] Executing doService on [GenerateUniqueMetadataValueService(GenerateUniqueMetadata)]
TRACE [ProductLookup] [c.a.c.s.m.GenerateUniqueMetadataValueService] [8a13eb5a-c3b3-4557-882b-f024fc253c10] Adding [515072ee-4620-48dd-9008-2fd143597aa1] to metadata key [a80c33d9-58c1-4c9b-a384-55c52b533eda]
DEBUG [ProductLookup] [c.a.c.ServiceList] [8a13eb5a-c3b3-4557-882b-f024fc253c10] Executing doService on [LogMessageService(LogProductRequest)]
```

Of course, you can get exceedingly fancy in log4j2 with routing based on the mapped diagnostic context. You could have a separate file for each message-id if you really wanted, though that may just leave you with hundreds of thousands of files on the filesystem...

```xml
<Routing name="Routing">
  <Routes pattern="$${ctx:myLoggingContext}">
    <Script name="RoutingInit" language="JavaScript"><![CDATA[
      if (logEvent.getContextMap().containsKey('myLoggingContext')) {
        logEvent.getContextMap().get('myLoggingContext');
      } else {
        "STANDARD";
      }]]>
    </Script>
    <Route>
      <RollingFile name="${ctx:myLoggingContext}" fileName="logs/${ctx:myLoggingContext}.log">
        <PatternLayout>
          <Pattern>%d{ISO8601} %-5p [%t] [%c.%M()] %m%n</Pattern>
        </PatternLayout>
        <Policies>
          <OnStartupTriggeringPolicy />
          <SizeBasedTriggeringPolicy size="10 MB"/>
        </Policies>
        <DefaultRolloverStrategy max="9" fileIndex="min"/>
      </RollingFile>
    </Route>
    <Route ref="ROLLLING_FILE" key="STANDARD"/>
  </Routes>
</Routing>
```

[^1]: In 3.7.3 add-logging-context-service will support `$UNIQUE_ID$` directly; so you can skip the add-metadata-service.

[logging-context-workflow-interceptor]: https://development.adaptris.net/javadocs/latest/Interlok-API/com/adaptris/core/interceptor/LoggingContextWorkflowInterceptor.html