---
layout: post
title: "Upgrading EHCache configurations"
author: gerco
# date: 2017-12-28 16:00
comments: false
tags: [interlok, ehcache]
categories: [interlok, ehcache]
published: true
description: "Upgrading from deprecated EHCache configuration"
keywords: "interlok"
excerpt_separator: <!-- more -->
---

The EHCache configuration element &lt;cache> was deprecated in Interlok version 3.6.4 in favor of a 
&lt;cache-connection> element so you don't have to repeat your cache configuration in each service
and can share the definitions easily.

<!-- more -->

In previous versions of Interlok, a cache service would look like this:

```xml
<retrieve-from-cache>
  ...
  <cache class="default-ehcache">
    <shutdown-cache-manager-on-close>true</shutdown-cache-manager-on-close>
    <cache-name>Some Name</cache-name>
    <cache-cleanup-interval>
      <unit>MINUTES</unit>
      <interval>1</interval>
    </cache-cleanup-interval>
    <max-elements-in-memory>0</max-elements-in-memory>
    <time-to-live>
      <unit>MINUTES</unit>
      <interval>30</interval>
    </time-to-live>
    <time-to-idle>
      <unit>MINUTES</unit>
      <interval>30</interval>
    </time-to-idle>
    <eviction-policy>LRU</eviction-policy>
  </cache>
  ...
</retrieve-from-cache>
```

And you would have to repeat this exact same configuration in each service using the cache. This is tedious
and error prone. Starting with Interlok 3.6.4, there is a new <cache-connection> object that you can add as
a shared connection:

```xml
<shared-components>
  <connections>
    <cache-connection>
      <unique-id>shared-cache</unique-id>
      <cache-instance class="default-ehcache">
        <shutdown-cache-manager-on-close>true</shutdown-cache-manager-on-close>
        <cache-name>Some Name</cache-name>
        <cache-cleanup-interval>
          <unit>MINUTES</unit>
          <interval>1</interval>
        </cache-cleanup-interval>
        <max-elements-in-memory>0</max-elements-in-memory>
        <time-to-live>
          <unit>MINUTES</unit>
          <interval>30</interval>
        </time-to-live>
        <time-to-idle>
          <unit>MINUTES</unit>
          <interval>30</interval>
        </time-to-idle>
        <eviction-policy>LRU</eviction-policy>
      </cache-instance>
    </cache-connection>
  </connections>
  <services/>
</shared-components>
```

Then you can reference that shared cache in any cache service, like this:

```xml
<retrieve-from-cache>
  ...
  <connection class="shared-connection">
    <lookup-name>shared-cache</lookup-name>
  </connection>
  ...
</retrieve-from-cache>
```

This approach elimitates many classes of problems that may occur with repeated configuration in each cache using
service and simplifies configuration considerably, especially if there are many services and workflows using the
cache.

The existing approach will still work fine for the forseeable future but a warning will be logged on adapter startup:
```
[RetrieveFromCacheService] 'cache' is deprecated; use a connection instead
```
