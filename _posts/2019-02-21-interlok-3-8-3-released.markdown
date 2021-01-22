---
title:             "Interlok 3.8.3"
description:       "Interlok 3.8.3 has been released and is now available for download."
published:         true
categories:        [interlok, releases]
tags:              [interlok, releases]
author:            higgyfella
billboard: /billboards/general-bg2.png
excerpt_separator: <!-- more -->
---

[Interlok 3.8.3](https://development.adaptris.net/installers/interlok/) has reached GA. It is now available for [download](https://development.adaptris.net/installers/interlok/).

<!-- more -->

The key highlights are :

* [Config projects](https://interlok.adaptris.net/interlok-docs/#/pages/ui/ui-config-project) continued improvements: xpath validation for variables; variable token selectors now available on lists and KeyValuePairSets; outputted files now retain order where possible and properties retain users comments, making it easier to work with vcs.
* [Component Search](https://interlok.adaptris.net/interlok-docs/#/pages/ui/ui-interlok-component-search) improvements include added pagination to the results page and style improvements
* There is a new UI [user-preference](https://interlok.adaptris.net/interlok-docs/#/pages/ui/ui-user-preferences) flag "enable technical preview features" for users who want our UI beta features
* New ["rest" management component](https://interlok.adaptris.net/interlok-docs/#/pages/user-guide/adapter-hosting-rest) that replaces interlok-restful-services
* Added [JMS](https://interlok.adaptris.net/interlok-docs/#/pages/cookbook/cookbook-jms) 2.0 support
* Improved performance for very large [JSON->XML and XML->JSON transformations](https://interlok.adaptris.net/interlok-docs/#/pages/cookbook/cookbook-json-transform)
* Support added for interlok-aws to enable withEndpointConfiguration for use with custom endpoints
* JdbcDataQueryService improvements allow you to have dynamic column translators
* JdbcDataCaptureService will now give you the number of rows updated
* Actional Interceptor now supports nested services, such as splitters.

The formal [change log](https://development.adaptris.net/docs/Interlok/changelog.html) can be found [here](https://development.adaptris.net/docs/Interlok/changelog.html). 
Or you can check the usual [sway presentation](https://sway.office.com/Ean39QSCTT74mdjl)


# Screenshots

Config projects - xpath validation for variables:
![xpath validation for variables]({{ site.baseurl }}/images/posts/release-383/383-config-val.png)

Config projects - xpath validation for variables with failues:
![xpath validation for variables with failures]({{ site.baseurl }}/images/posts/release-383/383-config-val2.png)

Config projects - token selector on key value pair sets:
![token selector on key value pair sets]({{ site.baseurl }}/images/posts/release-383/383-config-keyvalpairs.png)

Component Search - examples:
![Component Search example 1]({{ site.baseurl }}/images/posts/release-383/383-search-example1.png)

![Component Search example 2]({{ site.baseurl }}/images/posts/release-383/383-search-example2.png)

![Component Search example 3]({{ site.baseurl }}/images/posts/release-383/383-search-example3.png)

![Component Search example 4]({{ site.baseurl }}/images/posts/release-383/383-search-example4.png)


New UI User Preference:
![New UI User Preference]({{ site.baseurl }}/images/posts/release-383/383-user-pref.png)
