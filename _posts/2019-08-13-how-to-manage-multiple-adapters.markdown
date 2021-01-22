---
title:             "How to manage multiple adapters"
description:       "Once you're in production; what then?"
published:         true
categories:        [interlok]
tags:              [interlok]
author:            quotidian-ennui
billboard: /billboards/icon-purchase-to-pay.png
excerpt_separator: <!-- more -->
---

Once you've developed your solution and deployed it then you'll want to monitor it in production. Since all the runtime capabilities of Interlok are JMX based; our UI only uses publicly accessible JMX (there is no special sauce); you can leverage your existing JMX tools to manage Interlok. Of course the Interlok UI is perfectly capable of monitoring multiple Interlok instances which means you can use it as a monitoring tool in the absence of anything else.

<!-- more -->

So, let's say you want to do just that; you want to use the Interlok UI to manage multiple Interlok instances. There are a few best practises that we suggest you follow:

## Secure the JMX interface

 By default, the JMX management component uses the reference JMXMP protocol which is passwordless; but that doesn't have to be the case. You can secure it using vanilla password protection, SSL, or any combination thereof; use firewalls to limit traffic to only known hosts on the JMX ports. You can of course use any JMX implementation including our [JMX+JMS](https://interlok.adaptris.net/interlok-docs/#/pages/advanced/advanced-jmx-jms). If you opt for the JMS route, then you can leverage the JMS providers's own security layer to secure the endpoints that the JMX implementation is attached to.

## Have a dedicated monitoring UI instance

Have a single Interlok instance that contains no configuration and only contains the UI as a deployed artifact. You can use this as your launch pad for connecting to other runtime Interlok instances. People often want us to bind the UI into active directory or similar; our reponse is invariably that this is fixing the wrong problem. The UI doesn't matter, after all there would be no difference between a UI you have deployed on your workstation, and one that's been deployed with AD integration; what matters are the runtime Interlok instances, if they aren't secured, then securing the UI won't make it more secure. If you are only using it to monitor; then the monitoring instance won't need any additional jars in its classpath; but if you're using dependency management, it will be easy enough to inject the right optional components into the classpath.

This particular Interlok instance really should be blank; don't be be encouraged to have more than

```
<adapter>
  <unique-id>monitoring-instance</unique-id>
</adapter>
```

## Disable the UI on all other instances

Interlok doesn't need the UI at runtime. If you delete the war file within the installation; then the UI won't be started. If this particular Interlok instance isn't handling HTTP requests then disable the jetty component; even if you want jetty, delete the war file. This is the preferred deployment scenario; if you don't need the UI, then having it there means you've increased the resource/memory footprint of Interlok for no reason.

## Secure the UI instance

While the UI does not have particularly fine grained permissions, you have roles which can limit users to particular facets of the UI. `monitor` users only have access to the dashbaord and cannot configure the UI. Change the default administrator password. We tend to do this as the final touch.



