---
title:             "Interlok 3.9.1"
description:       "Interlok 3.9.1 has been released and is now available for download."
published:         true
categories:        [interlok, releases]
tags:              [interlok, releases]
author:            higgyfella
billboard: /billboards/general-bg6.png
excerpt_separator: <!-- more -->
---

[Interlok 3.9.1](https://development.adaptris.net/installers/Interlok/3.9.1/) has reached GA. It is now available for [download](https://development.adaptris.net/installers/Interlok/3.9.1/).

<!-- more -->

The key highlights are : 
* The [UI Config page](https://interlok.adaptris.net/interlok-docs/#/pages/ui/ui-config) has had many improvements including:
    * Advanced settings view can be toggled on a per-component basis
    * When selecting a connection for a producer; we always try to display the most suitable connection first
    * The [UI Config Navigation Tree](https://interlok.adaptris.net/interlok-docs/#/pages/ui/ui-config-navigation-tree) now displays Services (experimental)
    * Improved validation when trying to import existing configuration into a project.
* Microsoft SQL Server is now supported as the [UI database](https://interlok.adaptris.net/interlok-docs/#/pages/ui/ui-switch-db?id=ms-sqlserver-configuration)
* A new management component that can cluster Interlok instances via jgroups (interlok-cluster-manager)
* New interlok-elastic-rest optional component that uses the Elastic RestHighLevelClient. This supersedes all existing elasticsearch components, which have been deprecated and will be removed in a later release.
* Amazon Kinesis is now supported as a produce target via interlok-aws-kinesis
* org.eclipse.jetty.security.JDBCLoginService is now a valid login service to use with Jetty
* EDI transforms may now cache definitions via an external cache provider
* MessageAggregators can now filter messages prior to performing the actual aggregation. 

The formal [change log](https://interlok.adaptris.net/interlok-docs/#/pages/overview/changelog) can be found [here](https://interlok.adaptris.net/interlok-docs/#/pages/overview/changelog). 
Or you can check the usual [sway presentation](https://sway.office.com/7qFLKMbRio7pAPXQ)
