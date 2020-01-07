---
title:             "Interlok 3.9.0"
description:       "Interlok 3.9.0 has been released and is now available for download."
published:         true
categories:        [interlok, releases]
tags:              [interlok, releases]
author:            higgyfella
billboard: /billboards/general-bg5.png
excerpt_separator: <!-- more -->
---

[Interlok 3.9.0](https://development.adaptris.net/installers/Interlok/3.9.0/) has reached GA. It is now available for [download](https://development.adaptris.net/installers/Interlok/3.9.0/).

<!-- more -->

The key highlights are :

* [Config projects](http://interlok.adaptris.net/interlok-docs/ui-config-project.html) continued improvements: We have improved the variable selector in the settings editor so it populates empty settings upon variable selection (making sure that element is outputted upon Apply Config); Our service testing features now support testing with variable sets.
* [UI Config Navigation Tree](http://interlok.adaptris.net/interlok-docs/ui-config-navigation-tree.html), there is now a new way to navigate the config page, which will make is easier to traverse large configuration
* The config-conditional components have been promoted into the core Interlok release
* New [Switch Service](https://nexus.adaptris.net/nexus/content/sites/javadocs/com/adaptris/interlok-core/3.9.0-RELEASE/com/adaptris/core/services/conditional/Switch.html) which replaces some use cases for [branching-service-collection](https://nexus.adaptris.net/nexus/content/sites/javadocs/com/adaptris/interlok-core/3.9.0-RELEASE/com/adaptris/core/BranchingServiceCollection.html)
* Our JMS/XA components have been split into their own specific modules, i.e. XA-ActiveMQ, XA-Atomikos, XA-Solace, XA-Tibco, XA-WebsphereMQ & (a non- provider specific) XA-JMS
* The Email Integration components have been moved into their own optional component (interlok-mail)
* Also, the Flat file integration components have been moved into their own optional component (interlok-flatfile)
* There is now a [docker image](https://hub.docker.com/r/adaptris/interlok/tags) for Amazon Corretto + Azul

Some additional notes:

* The new optional components, interlok-mail & interlok-flatfile are bundled into the 3.9.0 install. So this change has no immediate effect, once you install, they will still be in the 'lib' directory.
* Config Items that were marked for removal in 3.9.0 have been removed, so there will be config changes if moving from 3.8 to 3.9
* The [UI Config Navigation Tree](http://interlok.adaptris.net/interlok-docs/ui-config-navigation-tree.html) is a beta feature, to access it you have to switch on the 'User Preferences'>'Enable technical preview features'
* Due to improvements in how we load classes within the UI; it is possible to get some WARN messages in the logfile from ‘org.reflections’ concerning missing classes (‘could not get type for name X from any class loader’); these warnings can be safely ignored, and you should put &lt;Logger name='org.reflections' level='ERROR'/&gt; in your log4j2.xml configuration.


The formal [change log](https://development.adaptris.net/docs/Interlok/changelog.html) can be found [here](https://development.adaptris.net/docs/Interlok/changelog.html). 
Or you can check the usual [sway presentation](https://sway.office.com/agFPJJajFX0YzUfl)


# Screenshots

Config projects - improved variable selector:
![improved variable selector]({{ site.baseurl }}/images/posts/release-390/var-selection.png)

Config projects - service testing supporting variable sets:
![improved variable selector]({{ site.baseurl }}/images/posts/release-390/test-group.png)
![improved variable selector]({{ site.baseurl }}/images/posts/release-390/test-single.png)
![improved variable selector]({{ site.baseurl }}/images/posts/release-390/test-inset.png)

Config Navigation Tree:
![improved variable selector]({{ site.baseurl }}/images/posts/release-390/config-tree.png)
![improved variable selector]({{ site.baseurl }}/images/posts/release-390/config-tree-filtered.png)
