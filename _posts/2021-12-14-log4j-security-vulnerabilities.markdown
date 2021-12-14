---
title:             "Apache Log4j Security Vulnerabilities"
description:       "Log4J2 Vulnerability and Interlok"
published:         true
categories:        [interlok, log4j, security]
tags:              [interlok, log4j, security]
author:            higgyfella
billboard: /billboards/general-bg7.png
excerpt_separator: <!-- more -->
---

You may have heard that there has been a recently discovered vulnerability in Log4j 2. Are you affected? How does it work? and how do you fix it?

<!-- more -->

The vulnerability, named '[CVE-2021-44228](https://nvd.nist.gov/vuln/detail/CVE-2021-44228)', has the highest possible severity rating of 10. You can refer to [this page](https://logging.apache.org/log4j/2.x/security.html) on the apache.org site for full details concerning the Apache Log4j 2 security vulnerabilities.

## Are you affected?

All versions of Log4j2 prior to 2.15.0 are affected. To test your version of Log4j2, you'll need to decompress (with a standard zip tool) all jar files starting with log4j.* and view the /META-INF/MANIFEST.MF file.

You will probably see a line something like this:

`Log4jReleaseVersion: 2.14.1`

## How does it work?

An attacker would need to have Log4j2 log a particularly formatted string. Log4j2 would then perform a lookup on this object allowing a remote code execution exploit.

## How to fix it?

For Interlok users, we re-released Interlok 4.3.0 with the latest patched Log4j2 binaries. If you downloaded Interlok 4.3.0 before the 13th of Decemember 2021, you should [re-download](https://development.adaptris.net/installers/Interlok/4.3.0/). All 4.x users should upgrade to the latest release.

Note, if you're currently a 3.x user, then there are configuration changes to upgrade to 4.x.

If you're using the [interlok-build-parent](https://github.com/adaptris/interlok-build-parent) both the `v3` and `v4`  `build.gradle` have been upgraded to use the latest patched Log4j2 binaries. You should rebuild and redeploy.

If you cannot upgrade, then the best solution is to manually patch all java libraries that start "log4j" in the "./interlok/lib/" and download the latest versions from maven, replacing the older libraries.

For Interlok, these java libraries would be (including download link for the latest patched versions);

* log4j-api.jar - [2.15.0 version](https://repo1.maven.org/maven2/org/apache/logging/log4j/log4j-api/2.15.0/log4j-api-2.15.0.jar)
* log4j-core.jar - [2.15.0 version](https://repo1.maven.org/maven2/org/apache/logging/log4j/log4j-core/2.15.0/log4j-core-2.15.0.jar)
* log4j-slf4j-impl.jar - [2.15.0 version](https://repo1.maven.org/maven2/org/apache/logging/log4j/log4j-slf4j-impl/2.15.0/log4j-slf4j-impl-2.15.0.jar)
* log4j-1.2-api - [2.15.0 version](https://repo1.maven.org/maven2/org/apache/logging/log4j/log4j-1.2-api/2.15.0/log4j-1.2-api-2.15.0.jar)

Alternatively, if you're running log4j versions 2.10.0 - 2.14.x, then you can disable these lookups by adding the following JVM switch;

`log4j2.formatMsgNoLookups=true`

For Interlok users that would mean modifying the start script to include the new switch:

`java -Dlog4j2.formatMsgNoLookups=true -jar ./lib/interlok-boot.jar`
