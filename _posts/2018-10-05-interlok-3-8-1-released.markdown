---
title:             "Interlok 3.8.1"
description:       "Interlok 3.8.1 has been released and is now available for download."
published:         true
categories:        [interlok, releases]
tags:              [interlok, releases]
author:            higgyfella
billboard: /billboards/general-bg1.png
excerpt_separator: <!-- more -->
---

[Interlok 3.8.1](https://development.adaptris.net/installers/interlok/) has reached GA. It is now available for [download](https://development.adaptris.net/installers/interlok/).

<!-- more -->

The key highlights are :

* The UI Config Page supports better [variable usages](http://interlok.adaptris.net/interlok-docs/ui-config-project.html#component-settings-modal), such as a mix of multiple variables and texts in a single setting
* The UI Config Page supports saving & loading of a [UI Project](http://interlok.adaptris.net/interlok-docs/ui-config-project.html) to & from a local file-system directory
* All the new UI projects will be saved with a [new structured format](http://interlok.adaptris.net/interlok-docs/ui-config-project.html#config-project-format) organised like Maven and Gradle projects
* First release of a reworked profiler; this will form the backbone of a new profiler component in the UI
* New optional component 'interlok-oauth-generic' which will give you a generic access token builder for OAUTH
* Changes to MongoDB for additional use-cases
* 'interlok-profiler-failover' has now been discontinued
    
The formal [change log](https://development.adaptris.net/docs/Interlok/changelog.html) can be found [here](https://development.adaptris.net/docs/Interlok/changelog.html). 
Or you can check the usual [sway presentation](https://sway.office.com/iLEeDZn6QtTCG2TS)


# Screenshots

Setting up a ui project with a local file system location:
![setting up ui project with local fs store]({{ site.baseurl }}/images/posts/release-381/381-fs-loc-1-setup.png)

Saving a ui project with a local file system location:
![Saving a ui project with local fs store]({{ site.baseurl }}/images/posts/release-381/381-fs-loc-2-saving.png)

The result of saving a ui project:
![result of saving a ui project]({{ site.baseurl }}/images/posts/release-381/381-fs-loc-3-result.png)

Loading a ui project from a local file system location: 
![loading a ui project from local fs store]({{ site.baseurl }}/images/posts/release-381/381-fs-loc-4-loading.png)

Loading a ui project from a local file system location (continued): 
![loading a ui project from local fs store - cont]({{ site.baseurl }}/images/posts/release-381/381-fs-loc-5-loading-2.png)

Setting up some variables in the ui project:
![Setting up some variables]({{ site.baseurl }}/images/posts/release-381/381-var-0-setup.png)

Opening a settings editor for a component:
![Opening a settings editor for a component]({{ site.baseurl }}/images/posts/release-381/381-var-1-settings-editor.png)

Using the variable input for a setting:
![Using the variable input for a setting]({{ site.baseurl }}/images/posts/release-381/381-var-2-open-var-input.png)

Having multiple variables in the one setting:
![Having multiple variables in the one setting]({{ site.baseurl }}/images/posts/release-381/381-var-3-multiple-vars.png)

Continuing to add strings and variables to the setting:
![loading a ui project from local fs store - cont]({{ site.baseurl }}/images/posts/release-381/381-var-4-added-strs.png)

An example of the new structured format for ui projects: 
![example dir structure]({{ site.baseurl }}/images/posts/release-381/dir-structure.PNG)
