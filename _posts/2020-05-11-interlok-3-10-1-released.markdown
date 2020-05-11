---
title:             "Interlok 3.10.1"
description:       "Interlok 3.10.1 has been released and is now available for download."
published:         true
categories:        [interlok, releases]
tags:              [interlok, releases]
author:            higgyfella
billboard: /billboards/general-bg10.png
excerpt_separator: <!-- more -->
---

[Interlok 3.10.1](https://development.adaptris.net/installers/Interlok/3.10.1/) has reached GA. It is now available for [download](https://development.adaptris.net/installers/Interlok/3.10.1/).

<!-- more -->

* Use failed messages in the [UI Config page](http://interlok.adaptris.net/interlok-docs/ui-config.html) testing features
* [UI Config Projects](http://interlok.adaptris.net/interlok-docs/ui-config-project.html) new 'Additional Files' tab
* There is now a 2nd UI [Service Tester Page](http://interlok.adaptris.net/interlok-docs/ui-service-tester.html) available which is a  proof of concept page experimenting with a new style, aiming to make it easier to configure service tester config
* The [UI Dashboard Page](http://interlok.adaptris.net/interlok-docs/ui-dashboard.html) has new features that make it easier to add clustered Adapter instances
* [UI Optional Component](http://interlok.adaptris.net/interlok-docs/ui-optional-component-discovery.html) has been improved to make it easier to switch between release and snapshot components
* Interlok Runtime improvements include:
    * New methods to the Cache interface allowing a Per-item cache expiry where supported by the underlying cache provider
    * New [AWS](https://github.com/adaptris/interlok-aws) Key management services to work with Amazon KMS; sign/verify documents via KMS API
    * A new service get-and-cache-oauth-token that allows you to get an OAUTH Access Token and re-use it repeatedly
    * Added support for RefreshToken in the OAUTH AccessToken
    * Change of dependency from com.sun.mail:javax.mail to jakarta mail
    * Solace JCSMP early development as an alternative to JMS
    * Profiling to Prometheus now gives additional metrics; average speed of workflow processing and failed message counts.
    * Added support for a SMB consumer/producer
    * interlok-shell has been marked as Deprecated, since it isn't supported in Java 11
* A new exciting experimental installer based on JavaFX that allows you to pre-select optional components that you want to bundle into your installation

The formal change log can be found [here](https://interlok.adaptris.net/interlok-docs/changelog.html)
Or you can check the usual [sway presentation](https://sway.office.com/4uKpj2IBCi0hyM9V)

# Screenshots

Testing with Failed Messages  :
![Failed Messages 01]({{ site.baseurl }}/images/posts/\release-3101/f1.png)
![Failed Messages 02]({{ site.baseurl }}/images/posts/\release-3101/f2.png)
![Failed Messages 03]({{ site.baseurl }}/images/posts/\release-3101/f3.png)
![Failed Messages 04]({{ site.baseurl }}/images/posts/\release-3101/f4.png)
![Failed Messages 05]({{ site.baseurl }}/images/posts/\release-3101/f5.png)

UI Config Projects new 'Additional Files' tab  :
![Additional Files 01]({{ site.baseurl }}/images/posts/\release-3101/p1.png)
![Additional Files 02]({{ site.baseurl }}/images/posts/\release-3101/p2.png)
![Additional Files 03]({{ site.baseurl }}/images/posts/\release-3101/p3.png)
![Additional Files 04]({{ site.baseurl }}/images/posts/\release-3101/p4.png)
![Additional Files 05]({{ site.baseurl }}/images/posts/\release-3101/p5.png)