---
title:             "Interlok 4.0.0"
description:       "Interlok 4.0.0 has been released and is now available for download."
published:         true
categories:        [interlok, releases]
tags:              [interlok, releases]
author:            higgyfella
billboard: /billboards/general-bg3.png
excerpt_separator: <!-- more -->
---

[Interlok 4.0.0](https://development.adaptris.net/installers/Interlok/4.0.0/) has reached GA. It is now available for [download](https://development.adaptris.net/installers/Interlok/4.0.0/).

<!-- more -->

* Interlok Runtime improvements include:
  * This Interlok version has been compiled for Java 11.
  * The deprecated ProduceDestination & ConsumeDestination components have been removed and entirely replaced with simplified configuration elements.
  * The [Nashorn script engine is deprecated](https://interlok.adaptris.net/interlok-docs/#/pages/advanced/advanced-scripting?id=nashorn-alternative) and you will recieve warnings if the EmbeddedScriptingService language is set to 'nashorn'.
  * Lots of deprecated things have been removed; please execute --configtest (or use the parent gradle) with an existing 3.12 instance to see what might have been removed
  * To mitigate against [XXE](https://github.com/adaptris/interlok/pull/648), XML Services now use a restricted instance if you leave it null, which means the default underlying document factory will disable things like doctype and entity handling. This change will not break config directly, but it will break currently-working functionality if you are relying on those featured being defaulted on in some DocumentBuilderFactory implementations.
  * interlok-xinclude has been marked as deprecated and due to be removed in Interlok 5.0.0; One of the reasons for this, is the way the Interlok UI handles relative paths; it is not always compatible with the Interlok xincludes pre-processors.
  * Several optional components have been removed, either because there is an improved replacement or they are not compatible with Java 11 as they are too old, or they are unloved components. Check the [changelog](https://interlok.adaptris.net/interlok-docs/#/pages/overview/changelog) for more details.

The formal change log can be found [here](https://interlok.adaptris.net/interlok-docs/#/pages/overview/changelog)
Or you can check the usual [sway presentation](https://sway.office.com/1xz34z0YdqOy64Y2)
