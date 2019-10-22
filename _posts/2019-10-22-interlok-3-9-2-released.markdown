---
title:             "Interlok 3.9.2"
description:       "Interlok 3.9.2 has been released and is now available for download."
published:         true
categories:        [interlok, releases]
tags:              [interlok, releases]
author:            higgyfella
excerpt_separator: <!-- more -->
---

[Interlok 3.9.2](https://development.adaptris.net/installers/Interlok/3.9.2/) has reached GA. It is now available for [download](https://development.adaptris.net/installers/Interlok/3.9.2/).

<!-- more -->

The key highlights are :
* The [UI Config page](http://interlok.adaptris.net/interlok-docs/ui-config.html) has had many improvements including:
    * The validation process now produces warnings for deprecated usage.
    * The settings editor now has a 3 way toggle for viewing different levels of settings (normal, advanced and rare settings) making is easier to configure components.
    * The service testing features now support testing with 'MIME' encoded messages.
* The [UI Component Search](http://interlok.adaptris.net/interlok-docs/ui-interlok-component-search.html) has been improved to return more accurate results.
* A new exciting UI Page, the [Interlok Profiler](http://interlok.adaptris.net/interlok-docs/ui-profiler-monitor.html) is now available (you need to enable technical preview features), allowing you analysis runtime performance metrics. 
* New services to provide PGP encryption, decryption, signing, and verification (interlok-pgp).
* FtpConsumers now support a range of FileFilters, including filters that aren't just 'name based'.
* Interlok now supports JMS 2.0 asynchronous producers with XA transactions.
* New simplified cache services that are simpler to configure and avoid XML bloat.
* HikariCP is now supported as an implementation for Pooled JDBC Connections.
* New alterative authentication schemes for AWS.
* In the interlok-aws-s3 component, the S3 List Operation now supports styles.    

The formal [change log](https://interlok.adaptris.net/interlok-docs/changelog.html) can be found [here](https://interlok.adaptris.net/interlok-docs/changelog.html). 
Or you can check the usual [sway presentation](https://sway.office.com/wnC9gAv83jPKb6Z6)

# Screenshots

Config projects - deprecated validation:
![deprecated validation]({{ site.baseurl }}/images/posts/release-392/validation.png)

Config projects - view normal, advanced and rare settings:
![three way view]({{ site.baseurl }}/images/posts/release-392/three-way-view.png)

Profiler UI:
![profiler]({{ site.baseurl }}/images/posts/release-392/profiler.png)
![profiler 2]({{ site.baseurl }}/images/posts/release-392/profiler2.png)
![profiler 3]({{ site.baseurl }}/images/posts/release-392/profiler3.png)


