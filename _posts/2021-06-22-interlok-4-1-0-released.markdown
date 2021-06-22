---
title:             "Interlok 4.1.0"
description:       "Interlok 4.1.0 has been released and is now available for download."
published:         true
categories:        [interlok, releases]
tags:              [interlok, releases]
author:            higgyfella
billboard: /billboards/general-bg4.png
excerpt_separator: <!-- more -->
---

[Interlok 4.1.0](https://development.adaptris.net/installers/Interlok/4.1.0/) has reached GA. It is now available for [download](https://development.adaptris.net/installers/Interlok/4.1.0/).

<!-- more -->

* Interlok improvements include:
  * A [new interlok-jdbc component](https://github.com/adaptris/interlok-ds/tree/develop/interlok-jdbc) that aims to simplify the syntax when [writing SQL statements](https://nexus.adaptris.net/nexus/content/sites/javadocs/com/adaptris/interlok-jdbc/4.1.0B1-RELEASE/com/adaptris/interlok/jdbc/package-summary.html), part of its goal is to make statement-parameters redundant.
  * The [interlok-build-parent project](https://github.com/adaptris/interlok-build-parent) has been promoted from [adaptris-labs](https://github.com/adaptris-labs) to the [adaptris github organisation](https://github.com/adaptris).
  * [Interlok service tester](https://github.com/adaptris/interlok-service-tester) has been improved so each test you configure can have a default [globally defined 'MainSource'](https://github.com/adaptris/interlok-service-tester/pull/114), making it easier to setup and re-configure.
  * The [Interlok service tester UI page](https://interlok.adaptris.net/interlok-docs/#/pages/ui/ui-service-tester) now supports opening and publishing projects from VCS, like how the [UI Config Page](https://interlok.adaptris.net/interlok-docs/#/pages/ui/ui-version-control) works.
  * The Interlok UI now allows adding a new ([interlok-jolokia](https://github.com/adaptris/interlok-jolokia) management enabled) adapter on the dashboard page using a [http jolokia url](https://interlok.adaptris.net/interlok-docs/#/pages/advanced/advanced-jolokia).
  * [Added JSON path service](https://nexus.adaptris.net/nexus/content/sites/javadocs/com/adaptris/interlok-json-streaming/4.1.0B1-RELEASE/com/adaptris/core/json/streaming/JsonPathStreamingService.html) that uses [JSON Surfer](https://github.com/jsurfer/JsonSurfer) to parse [JSON as a stream](https://github.com/adaptris/interlok-json/tree/develop/interlok-json-streaming) and extract data using JSON path syntax.
  * A [new ftp-recursive-consumer](https://nexus.adaptris.net/nexus/content/sites/javadocs/com/adaptris/interlok-core/4.1.0B1-RELEASE/com/adaptris/core/ftp/FtpRecursiveConsumer.html) that will recursively descend into any sub directories it finds to consume files.
  * The UI Widgets that display static data have been removed from the [widgets page](https://interlok.adaptris.net/interlok-docs/#/pages/ui/ui-widgets), and their data has been relocated to the [UI dashboard](https://interlok.adaptris.net/interlok-docs/#/pages/ui/ui-dashboard) information modal.
  * We now offer the ability to publish metrics to [Datadog](https://interlok.adaptris.net/interlok-docs/#/pages/advanced/advanced-profiler-datadog) and [NewRelic](https://interlok.adaptris.net/interlok-docs/#/pages/advanced/advanced-new-relic-profiling_v4)

The formal change log can be found [here](https://interlok.adaptris.net/interlok-docs/#/pages/overview/changelog)
Or you can check the usual [sway presentation](https://sway.office.com/mz6zHpO0QAG756Zx)
