---
title:             "How to change the jetty work directory"
description:       "The Interlok UI starts too slowly; I hate it!"
published:         true
categories:        [interlok]
tags:              [interlok]
author:            quotidian-ennui
billboard: /billboards/icon-brokerage.png
excerpt_separator: <!-- more -->
---

Do you remember when you were at school; someone did something a little bit naughty and the teacher, rather than pinpointing the culprit, punishes the entire class/year/school. Corporate IT and antivirus programs are a bit like that sometimes. By default jetty will extract all web applications into a temporary folder (usually the system temporary folder). This can lead to long start times as your antivirus of course on-demand scans all the jar files, and sometimes (this is true of MS Defender on some of my test boxen) asks you to upload various jar files for analysis.

<!-- more -->

As a java developer; this amuses me; compile times on aggressive antivirus systems are extended because `.class` files need to be scanned as they're written; resolving jars from your dependency manager needs to be scanned. Everything needs to be scanned. When running `gradle check` means that 90% of your CPU is being used by your antivirus rather than java+javac then...

![compiling](https://imgs.xkcd.com/comics/compiling.png )

If you're in that situation, then you have a couple of options, however, you have to be able to exclude a certain directory from real time scanning. Usually I choose the Interlok installation directory.

## Create a work directory

Just create a work directory inside your interlok installation e.g. C:\adaptris\interlok\work. Jetty will extract any war files into the `work` directory if its available, otherwise it will use the directory specified by `java.io.tmpdir` which leads us to the second option.

## Set JAVA_TOOL_OPTIONS environment variable

Later versions of Java 8 will take heed of the _JAVA_TOOL_OPTIONS_ environment variable and use any properties/arguments defined by that variable; this would affect all java processes that you start of course, and will create additional logging in your console. `JAVA_TOOL_OPTIONS=-Djava.io.tmpdir=C:/Adaptris/Interlok/TEMP` would set your temporary directory for all java processes to the specified directory.

This can have the benefit of putting all your Interlok temporary files into the same location as well (such as those created by a FileBackedMessageFactory or similar).

## Bonus Speedup

Switch to the in-memory database because you don't care about preserving your preferences, users, widgets across interlok JVM restarts. Your mileage may vary; sometimes it's nice to not have to keep resetting your preferences.

```
$ cat ui-resources/interlokuidb.properties
#Mon, 12 Aug 2019 19:26:43 +0100

dataSource.provider=derby
dataSource.driverClass=org.apache.derby.jdbc.EmbeddedDriver
dataSource.jdbcURL=jdbc:derby:memory:interlokuidb;create=true
dataSource.user=interlokuidb
dataSource.password=interlokuidb
```




