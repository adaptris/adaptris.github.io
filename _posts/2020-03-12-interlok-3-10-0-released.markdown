---
title:             "Interlok 3.10.0"
description:       "Interlok 3.10.0 has been released and is now available for download."
published:         true
categories:        [interlok, releases]
tags:              [interlok, releases]
author:            higgyfella
billboard: /billboards/general-bg9.png
excerpt_separator: <!-- more -->
---

[Interlok 3.10.0](https://development.adaptris.net/installers/Interlok/3.10.0/) has reached GA. It is now available for [download](https://development.adaptris.net/installers/Interlok/3.10.0/).

<!-- more -->

* The [UI Optional Component Page](https://interlok.adaptris.net/interlok-docs/#/pages/ui/ui-optional-component-discovery) has had many improvements, including:
    * a reworked details modal that has had lots of new help text added to make things clearer
    * integration of project readme documents into the details window
    * more filtering options added, for better filtering of deprecated and licensed components
* The [UI Service Tester Page](https://interlok.adaptris.net/interlok-docs/#/pages/ui/ui-service-tester) has had many improvements, including:
    * A run option has been added to the test item that will run all its test cases.
    * When adding a new Test the Default Config File Source is now selected by default.
    * Improved the naming of tests when importing from existing config
* The [UI User Preference](https://interlok.adaptris.net/interlok-docs/#/pages/ui/ui-user-preferences) 'Always attempt to load the active adapter' has now been set to false by default as we continue to promote the use of project based configuration.  
* Interlok Runtime improvements include:
    * Added support for the Interlok to use the [Prometheus pushgateway](https://github.com/adaptris/interlok-profiler-prometheus)
    * By default, the standard [docker](https://hub.docker.com/r/adaptris/interlok/tags) images will run Interlok as an unprivileged user (You’ve always been able to run interlok as a unprivileged user, It just wasn’t done by default).
    * [interlok-jmx-jms](https://github.com/adaptris/interlok-jmx-jms) has been separated into individual provider projects
    * Initial support for Asynchronous messaging
    * The Default [JSON](https://github.com/adaptris/interlok-json) transformation driver is now "simple-json"
    * [apache-geode](https://github.com/adaptris/interlok-cache/tree/develop/interlok-apache-geode) is now supported as a caching provider
    * Prototype support for [multi-payload messages](https://interlok.adaptris.net/interlok-docs/#/pages/advanced/advanced-multi-payload-messages)
    * The default Saxon [transformation engine](https://interlok.adaptris.net/interlok-docs/#/pages/cookbook/cookbook-xml-transform) has been upgraded from 9.8 to 9.9.x

The formal change log can be found [here](https://interlok.adaptris.net/interlok-docs/#/pages/overview/changelog)
Or you can check the usual [sway presentation](https://sway.office.com/kUvj2NZRqflnEY6W)

# Screenshots

Optional Components Page :
![Optional Components 01]({{ site.baseurl }}/images/posts/release3100/OptionalComponents01.png)
![Optional Components 02]({{ site.baseurl }}/images/posts/release3100/OptionalComponents02.png)
![Optional Components 03]({{ site.baseurl }}/images/posts/release3100/OptionalComponents03.png)
![Optional Components 04]({{ site.baseurl }}/images/posts/release3100/OptionalComponents04.png)
![Optional Components 05]({{ site.baseurl }}/images/posts/release3100/OptionalComponents05.png)
![Optional Components 06]({{ site.baseurl }}/images/posts/release3100/OptionalComponents06.png)
![Optional Components 07]({{ site.baseurl }}/images/posts/release3100/OptionalComponents07.png)
