---
title:             "My HTTP Workflow is too busy"
description:       "Returning a 503 HTTP response might be better than succeeding slowly"
published:         true
categories:        [interlok]
tags:              [interlok]
author:            quotidian-ennui
billboard: /billboards/icon-orders.png
excerpt_separator: <!-- more -->
---

The Interlok configuration that you've deployed is part of of a high-volume, sustained load pipeline; it's awesome because you're using it to HTTP proxy some legacy backend system that is a critical part of the pipeline. However, you're now in a situation where your REST API calls to Interlok aren't timing out when the legacy system is under-stress; they're blocking for an unacceptable amount of time before finally succeeding. If it was enabled by an HTTP header on the request (naming `Expect: 102-Processing`) then you would have been getting periodic _HTTP 102 Processing_ responses back; what you really want is a configuration where you fail immediately if you can't immediately service the inbound request.

<!-- more -->

This is where you would use the [jetty-no-backlog-interceptor][]. Enabling it is quite easy, but there is a caveat that you need to be aware of: it only works with _pooling-workflow_, so if you want single-threaded behaviour then you need to set the maximum poolsize to be 1. After that it's simply a case of adding it to your workflow configuration

```
<pooling-workflow>
  <jetty-no-backlog-interceptor/>
  <jetty-pooling-workflow-interceptor/>
  <pool-size>10</pool-size>
  <min-idle>10</min-idle>
  <max-idle>10</max-idle>
  ... Rest of the workflow omitted.
</pooling-workflow>
```

What this means is that if the current request would cause the count of in flight messages to exceed the number of workers available (in this case 10) then a _503 Server Busy_ response is returned immediately to the caller. The message will still eventually be submitted to the workflow when a worker becomes available; but it will have been marked so that normal service and producer execution is skipped (check the javadocs), so it is effectively discarded.

We actually use it in many situations where we have an exceptionally long-running workflow (a batch job triggered on demand) that might take hours (it's a typical edge case that isn't that edgy); so what we do is to configure the workflow with _pool-size=1_ and a `jetty-no-backlog-interceptor`; this means that additional requests are just discarded (because who wants to queue up multiple batch jobs that might take 18 hours to run).

[jetty-no-backlog-interceptor]: https://nexus.adaptris.net/nexus/content/sites/javadocs/com/adaptris/interlok-core/3.8-SNAPSHOT/com/adaptris/core/http/jetty/JettyNoBacklogInterceptor.html
