---
title:             "Interlok 3.12.0.3"
description:       "Interlok 3.12.0.3 has been released and is now available for download."
published:         true
categories:        [interlok, releases]
tags:              [interlok, releases]
author:            domule
billboard: /billboards/general-bg7.png
excerpt_separator: <!-- more -->
---

Interlok 3.12.0.3 has reached GA.

<!-- more -->

Version 3.12.0.3 is a mini release, which was published in December 2021. It includes a few bug fixes as well as added deprecation warnings. The latter eases migrating from the major version 3.x to 4.x. It is therefore strongly recommended to first upgrade to 3.12.0.3 before embarking on an upgrade to 4.x.

The formal change log can be found below for the respective versions released after 3.12.0, leading up to 3.12.0.3:

* [3.12.0.1](https://interlok.adaptris.net/interlok-docs/#/pages/overview/changelog?id=version-31201)
* [3.12.0.2](https://interlok.adaptris.net/interlok-docs/#/pages/overview/changelog?id=version-31202)
* [3.12.0.3](https://interlok.adaptris.net/interlok-docs/#/pages/overview/changelog?id=version-31203)

This post contains guidance on how to upgrade to 3.12.0.3.

The versions from 3.12.0.1 onwards don't have installers, and only affect the following artifact jars since they are built from the same multi-module repository: interlok-boot.jar, interlok-client.jar, interlok-client-jmx.jar, interlok-common.jar, interlok-core.jar, interlok-core-apt.jar, interlok-logging.jar.

How to upgrade to version 3.12.0.3 depends on the way you build and deploy Interlok. Options include:

* If using Gradle then simply build again using the [updated V3 parent](https://raw.githubusercontent.com/adaptris/interlok-build-parent/main/v3/build.gradle), which now refers to 3.12.0.3.
* Update your dependency tree to use `com.adaptris:interlok-core:3.12.0.3-RELEASE`.
* If installing manually, first [install 3.12.0](https://development.adaptris.net/installers/Interlok/3.12.0/) as usual and then replace all the respective 3.12.0 jars with the following respective 3.12.0.3 versions:
  * [interlok-boot-3.12.0.3-RELEASE.jar](https://nexus.adaptris.net/nexus/content/repositories/releases/com/adaptris/interlok-boot/3.12.0.3-RELEASE/interlok-boot-3.12.0.3-RELEASE.jar)
  * [interlok-client-3.12.0.3-RELEASE.jar](https://nexus.adaptris.net/nexus/content/repositories/releases/com/adaptris/interlok-client/3.12.0.3-RELEASE/interlok-client-3.12.0.3-RELEASE.jar)
  * [interlok-client-jmx-3.12.0.3-RELEASE.jar](https://nexus.adaptris.net/nexus/content/repositories/releases/com/adaptris/interlok-client-jmx/3.12.0.3-RELEASE/interlok-client-jmx-3.12.0.3-RELEASE.jar)
  * [interlok-common-3.12.0.3-RELEASE.jar](https://nexus.adaptris.net/nexus/content/repositories/releases/com/adaptris/interlok-common/3.12.0.3-RELEASE/interlok-common-3.12.0.3-RELEASE.jar)
  * [interlok-core-3.12.0.3-RELEASE.jar](https://nexus.adaptris.net/nexus/content/repositories/releases/com/adaptris/interlok-core/3.12.0.3-RELEASE/interlok-core-3.12.0.3-RELEASE.jar)
  * [interlok-core-apt-3.12.0.3-RELEASE.jar](https://nexus.adaptris.net/nexus/content/repositories/releases/com/adaptris/interlok-core-apt/3.12.0.3-RELEASE/interlok-core-apt-3.12.0.3-RELEASE.jar)
  * [interlok-logging-3.12.0.3-RELEASE.jar](https://nexus.adaptris.net/nexus/content/repositories/releases/com/adaptris/interlok-logging/3.12.0.3-RELEASE/interlok-logging-3.12.0.3-RELEASE.jar)

Make sure to also visit [Apache Log4j Security Vulnerabilities](https://interlok.adaptris.net/blog/2021/12/14/log4j-security-vulnerabilities.html), which will help verify that the late-2021 Log4J vulnerability is also fully addressed.
