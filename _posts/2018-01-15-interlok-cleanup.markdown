---
layout: post
title: "Interlok temporary file cleanup"
author: quotidian-ennui
# date: 2018-01-15 11:00
comments: false
tags: [interlok]
categories: [interlok]
published: true
description: "Cleaning up temporary files processing; triggering System.gc() periodically"
keywords: "interlok"
excerpt_separator: <!-- more -->
---

When you're working with messages that are backed off to eh filesystem, then you may see find that there are a bunch of files that will be written to your tempdir (either configured explicitly or the value of `java.io.tmpdir`). They will be auto-deleted when messages go out of scope, and garbage collection happens. This will be particularly apparent if your configuration contains a large-message-workflow with a large-fs-consumer; you will find that the original file is left on the filesystem until garbage collection occurs which may be long after any processing has completed.

<!-- more -->

Since you can't guarantee when garbage collection happens, if you want to periodically force the adapter to suggest a `System.gc()`, and cleanup any temporary files, we can do this with a custom channel. Essentially your custom channel needs to have a file-system consumer on your temporary directory (denoted here as `${adapter.temp.dir}` or `${adapter.temp.dir.url}`) and a jmx connection to itself so we end up with something like this.

```xml
<channel>
  <unique-id>Cleanup</unique-id>
  <auto-start>true</auto-start>
  <workflow-list>
    <!-- Clean up temp folder -->
    <large-message-workflow>
      <unique-id>CleanupTempFiles(large)</unique-id>
      <consumer class="large-fs-consumer">
        <message-factory class="file-backed-message-factory">
          <temp-directory>${adapter.temp.dir}</temp-directory>
          <create-temp-dir>true</create-temp-dir>
        </message-factory>
        <destination class="configured-consume-destination">
          <destination>${adapter.temp.dir.url}</destination>
          <filter-expression>OlderThan=-P1D__@@__SizeGTE=1024000</filter-expression>
          <configured-thread-name>CleanupTempFiles(large)</configured-thread-name>
        </destination>
        <create-dirs>true</create-dirs>
        <file-filter-imp>com.adaptris.core.fs.CompositeFileFilter</file-filter-imp>
        <reacquire-lock-between-messages>true</reacquire-lock-between-messages>
        <poller class="quartz-cron-poller">
          <cron-expression>0 */15 * * * ?</cron-expression>
        </poller>
        <wip-suffix>_safe_to_delete</wip-suffix>
        <reset-wip-files>true</reset-wip-files>
      </consumer>
      <service-collection class="service-list"/>
      <send-events>false</send-events>
    </large-message-workflow>
    <!-- Clean up temp folder -->
    <standard-workflow>
      <unique-id>CleanupTempFiles(small)</unique-id>
      <consumer class="fs-consumer">
        <destination class="configured-consume-destination">
          <destination>${adapter.temp.dir.url}</destination>
          <filter-expression>OlderThan=-P1D__@@__SizeLT=1024000</filter-expression>
          <configured-thread-name>CleanupTempFiles(small)</configured-thread-name>
        </destination>
        <create-dirs>true</create-dirs>
        <file-filter-imp>com.adaptris.core.fs.CompositeFileFilter</file-filter-imp>
        <reacquire-lock-between-messages>true</reacquire-lock-between-messages>
        <poller class="quartz-cron-poller">
          <cron-expression>0 */15 * * * ?</cron-expression>
        </poller>
        <wip-suffix>_safe_to_delete</wip-suffix>
      </consumer>
      <service-collection class="service-list"/>
      <send-events>false</send-events>
    </standard-workflow>
    <!-- Do some Garbage Collection -->
    <standard-workflow>
      <unique-id>GarbageCollection</unique-id>
      <consumer class="polling-trigger">
        <destination class="configured-consume-destination">
          <configured-thread-name>GarbageCollection</configured-thread-name>
          <destination>GarbageCollection</destination>
        </destination>
        <poller class="quartz-cron-poller">
          <cron-expression>0 */12 * * * ?</cron-expression>
        </poller>
      </consumer>
      <service-collection class="service-list">
        <services>
          <jmx-operation-call-service>
            <connection class="jmx-connection">
              <jmx-service-url>service:jmx:jmxmp://localhost:5555</jmx-service-url>
            </connection>
            <object-name>java.lang:type=Memory</object-name>
            <operation-name>gc</operation-name>
          </jmx-operation-call-service>
        </services>
      </service-collection>
      <send-events>false</send-events>
    </standard-workflow>

  </workflow-list>
</channel>
```

Once this channel is started; the following things happen (the scheduling is arbitrary, as is the size filter):

* Every 15 minutes we consume files smaller than ~10Mb from your temp directory where the last modified time is at least 1 day ago.
* Every 15 minutes we consume files larger than (or equal to) ~10Mb from your temp directory where the last modified time is at least 1 day ago.
    * If garbage collection happens then they will be deleted; if not, then we will request a gc in the next workflow.
* Every 12 minutes we connect to ourselves via JMX and issue a garbage collection request.
    * Note that you can issue the JMX call manually via the UI, or using your preferred JMX application
