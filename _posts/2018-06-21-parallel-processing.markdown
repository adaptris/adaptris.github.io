---
title:             "Parallel Processing"
description:       "Executing long running services in parallel"
published:         true
categories:        [interlok]
tags:              [interlok]
author:            mcwarman
billboard: /billboards/icon-inventory.png
excerpt_separator: <!-- more -->
---

One thing that has been difficult to solve in Interlok is the ability to execute two services in parallel against a message.

<!-- more -->

With the use of shared services and the `pooling-message-splitter-service` we can now do exactly that.

More specifically we can use `dynamic-shared-service`, the `split-by-metadata` splitter and a metadata expressions, to execute a list of services is parallel.

```xml
<shared-components>
  <services>
    <service-list>
      <unique-id>action1</unique-id>
      <services>
        <!-- ... -->
      </services>
    </service-list>
    <service-list>
      <unique-id>action1</unique-id>
      <services>
        <!-- ... -->
      </services>
    </service-list>
  </services>
</shared-component>
<!-- ... -->
<services>
  <add-metadata-service>
    <key-value-pair>
      <key>actions</key>
      <value>action1,action2</value>
    </key-value-pair>
  </add-metadata-service>
  <pooling-message-splitter-service>
    <max-threads>2</max-threads>
    <splitter class="split-by-metadata">
      <metadata-key>actions</metadata-key>
      <split-metadata-key>action</split-metadata-key>
      <separator>,</separator>
    </splitter>
    <service class="service-list">
      <services>
        <dynamic-shared-service>
          <lookup-name>%message{action}</lookup-name>
        </dynamic-shared-service>
      </services>
    </service>
  </pooling-message-splitter-service>
</services>
```

A real world example could be a using `pooling-split-join-service` where the shared services made calls to different APIs then using an aggregator like `json-array-aggregator` to produce the combined payload.
