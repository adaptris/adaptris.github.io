---
title:             "Interlok 3.8.2"
description:       "Interlok 3.8.2 has been released and is now available for download."
published:         true
categories:        [interlok, releases]
tags:              [interlok, releases]
author:            higgyfella
excerpt_separator: <!-- more -->
---

[Interlok 3.8.2](https://development.adaptris.net/installers/Interlok/3.8.2/) has reached GA. It is now available for [download](https://development.adaptris.net/installers/Interlok/3.8.2/).

<!-- more -->

The key highlights are :

* [Config projects](http://interlok.adaptris.net/interlok-docs/ui-config-project.html) improvements to allow more customisation and easier to use with VCS.
* The UI [Widgets Page](http://interlok.adaptris.net/interlok-docs/ui-widgets.html) allows the ability to [create widgets that are backed by data from custom endpoints](http://interlok.adaptris.net/interlok-docs/ui-widgets.html#custom-widgets).
* The [Config Pages](http://interlok.adaptris.net/interlok-docs/ui-config.html) [Settings editor](http://interlok.adaptris.net/interlok-docs/ui-config.html#settings-editor-features) has improved so editing lists of objects is more manageable.
* New UI '[Component Search](http://interlok.adaptris.net/interlok-docs/ui-interlok-component-search.html)' page that allows you to search an Elasticsearch index containing all our Interlok components.
* New management component that can start and stop an external process on Interlok JVM start-up
* New optional component 'interlok-json-streaming' that lets you perform streaming operations on JSON
* Services JsonMapInsert and Upsert now support expression based table names
* New optional component 'interlok-kie' that is an upgraded replacement for 'interlok-drools'
* Improved Interceptor that publishes the last timeslice

The formal [change log](https://development.adaptris.net/docs/Interlok/changelog.html) can be found [here](https://development.adaptris.net/docs/Interlok/changelog.html). 
Or you can check the usual [sway presentation](https://sway.office.com/KYPk16t3bPqK2o9u)

# Screenshots

You can use variables properties inheritance directly within the UI:
![variables properties inheritance]({{ site.baseurl }}/images/posts/release-382/var-inheritance.png)

Configuration of x-includes directory structure:
![x-includes directory structure]({{ site.baseurl }}/images/posts/release-382/x-includes-dir.png)

Improved configuration of the variable properties naming:
![variable properties naming]({{ site.baseurl }}/images/posts/release-382/var-props-naming.png)

Add a custom widgets to the Widgets Page (part one):
![Add a custom widgets part-one]({{ site.baseurl }}/images/posts/release-382/custom-widgets-add-1.png)

Add a custom widgets to the Widgets Page (part two):
![Add a custom widgets part-two]({{ site.baseurl }}/images/posts/release-382/custom-widgets-add-2.png)

A custom widget on the Widgets Page:
![custom widget on the Widgets Page]({{ site.baseurl }}/images/posts/release-382/custom-widgets-view.png)

All the different custom widgets added to the Widgets Page:
![All the different custom widgets]({{ site.baseurl }}/images/posts/release-382/custom-widgets-all.png)

The Settings editor handling a list of objects:
![Settings editor handling a list of objects]({{ site.baseurl }}/images/posts/release-382/settings-1.png)

The Settings editor handling a list of objects (editing an individual object from a list):
![editing an individual object from a list]({{ site.baseurl }}/images/posts/release-382/settings-2.png)

The UI Component Search:
![UI Component Search]({{ site.baseurl }}/images/posts/release-382/search.png)
